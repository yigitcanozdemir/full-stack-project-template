# Replacing the backend

The Python backend is the default slot, not a requirement. Replacing it (Node/TypeScript, Go, …) is
an ADR first (`adr` skill) — then the new backend must keep every contract the rest of the repository
depends on:

| Contract | What the new backend must provide |
|---|---|
| API contract | `backend/openapi.json`, committed, generated from the code without a running server, deterministic (sorted keys), and a test or CI step that fails when it is stale. The frontend's `generate:api` reads this path. |
| Error envelope | Non-2xx bodies are `{"error": {"code", "message", "field?"}}` — `src/lib/api.ts` parses that shape. |
| Config | One settings module is the only environment reader; required values fail at startup; every variable is in `backend/.env.example`. |
| Migrations | Generated from the models or schema, reviewed, never edited once applied. |
| Health | `GET /health` touches nothing; `GET /health/ready` checks each backing service and answers 503 when one fails. |
| Image | `backend/Dockerfile`, non-root user, port 8000, configured only by environment variables. |
| Operation ids | Stable names derived from code, not from the path, so the generated client doesn't churn. |

Then update the seams:

1. `backend/AGENTS.md` — rewrite with the `agents-md` skill: stack, check commands, layering,
   conventions, anti-patterns, review rules for the new language.
2. `.github/workflows/ci.yml` — the `backend` job's setup, install and check steps.
3. `.pre-commit-config.yaml` — replace the ruff and mypy hooks with the new linters.
4. `docker-compose.yml` — the `backend` service's build and environment.
5. Root `AGENTS.md` Stack table and Layout; root `README.md` quick start; `backend/README.md`.
6. `.gitignore` — the new toolchain's build and cache directories.

Replacing only the web framework inside Python (e.g. FastAPI → Litestar) keeps `app/config.py`,
`app/platform/db.py`, migrations and tests' structure; rewrite `main.py`, the routers, `openapi.py`
and the error handler.
