# Backend — Agent Instructions

The API service. Read the root `AGENTS.md` first; this file is the backend slice of it.

## Stack

Python, uv, FastAPI, Pydantic + pydantic-settings, SQLAlchemy async + asyncpg, Alembic, redis-py
(asyncio). Dev: pytest + pytest-asyncio, ruff, mypy (strict), httpx.

- **The Python version is written only in `.python-version`.** The Docker image installs that
  interpreter; ruff and mypy infer it. Changing it: edit `.python-version` and the `requires-python`
  lower bound, then `uv lock`.
- **uv only.** Never `pip install`, never a `requirements.txt`. `uv add <pkg>` / `uv add --dev <pkg>`
  change `pyproject.toml` and `uv.lock` together — ask first (root: Agent boundaries).
- `uv.lock` is committed; CI and the Dockerfile install with `--locked`.

## Checks (run from `backend/`)

```bash
uv run ruff format . && uv run ruff check . && uv run mypy
uv run pytest -q                                   # unit tier; integration tests skip
INTEGRATION=1 uv run pytest -q                     # with `docker compose up -d` running
uv run python scripts/dump_openapi.py              # after ANY request/response model change
uv run alembic check                               # models and migrations agree (needs the DB)
```

## Layout

```text
backend/
├── app/
│   ├── main.py           # app factory, lifespan, error mapping, router mounting — no business logic
│   ├── config.py         # Settings — the only environment reader
│   ├── openapi.py        # renders openapi.json (no server needed)
│   ├── platform/         # infrastructure every module uses: db, redis_client, errors
│   ├── routers/          # routes that belong to no module (health)
│   ├── modules/          # one package per business domain — modules/README.md has the shape
│   ├── migrations/       # Alembic env + versions/
│   └── tests/            # mirrors app/; `integration` marker for tests that need Postgres/Redis
├── scripts/dump_openapi.py
├── openapi.json          # generated, committed: the frontend's type source
└── alembic.ini
```

An external API client (LLM provider, payments, email) goes in `app/platform/<name>.py` behind a small
interface the services call, so the provider can change without touching a module.

## Layering

`router → service → repository → database`

- **Router:** parses input, calls one service operation, shapes the typed output. No rules, no
  queries.
- **Service:** business rules and the transaction (`await session.commit()`). Raises
  `app.platform.errors` subclasses — never `HTTPException`; `main.py` maps them to statuses once.
- **Repository:** reads and writes its own module's tables. Returns models, never HTTP responses.

Modules import `app.platform`, never each other; a module that needs another's data calls that
module's service.

## Conventions

- Type every signature; mypy runs strict. A `# type: ignore[code]` carries a reason.
- `async def` for anything that does I/O. No `requests`, no sync DB driver, no `time.sleep` in a
  request path — use httpx, asyncpg, `asyncio.sleep`.
- Log with `logging.getLogger(__name__)`, never `print`. Never log a secret, token or full PII field.
- Declare a response as precisely as the write path constrains it: a `Literal` on the request and a
  bare `str` on the response makes the frontend re-narrow it by hand.
- Every JSON 2xx has a return annotation or `response_model`, so it has a schema in `openapi.json`.
- Operation ids are `<tag>.<function name>` (`main.operation_id`): renaming the function renames
  the frontend's operation; moving the path does not.

## Migrations

- Change the model → `uv run alembic revision --autogenerate -m "<what>"` → **read the file** → apply.
  Autogenerate misses CHECK constraints, some server defaults and renames (it sees a drop plus an add).
- A new module's `models.py` is found automatically (`migrations/env.py` walks `app/modules/`).
- Constraint names come from `NAMING_CONVENTION` in `platform/db.py` (PostgreSQL's own defaults). Name
  CHECK constraints explicitly.
- Never edit an applied revision; roll forward with a new one.
- Adding a NOT NULL column to a table with rows needs a server default or a backfill step.

## Anti-patterns (rejected)

- `os.getenv` / `load_dotenv` outside `app/config.py`.
- A query or a business rule in a router; `HTTPException` raised from a service.
- One `modules/*` package importing another.
- Blocking I/O in an `async def`.
- Tables created by startup code instead of a migration.
- Catching `Exception` only to log and re-raise, or wrapping responses in a custom envelope class.

## Code review rules

Extends the root rules for changes under `backend/`.

**Migrations are the highest-risk surface — a bad one is found in production.** Flag a revision that
drops or renames a column without preserving its data; adds NOT NULL with no default or backfill;
disagrees with its model; or orders statements so an earlier one breaks a later one.

**Concurrency:** a read-then-write with no constraint, lock or retry behind it (name the two requests
that interleave); a retry that catches `IntegrityError` broadly instead of the one constraint it can
resolve; user text in a `LIKE` pattern without escaping `%` and `_`; an `await` per row inside a loop
that one query could replace (N+1).

**Do not report:** a service function with no router yet; an import inside a function (usually breaks
a cycle); a `# type: ignore` that carries a reason.
