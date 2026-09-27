---
name: init-stack
description: Start a project from this template or change its stack — read the idea/PRD documents, recommend a stack from their requirements, refresh the template's version snapshot, scaffold the frontend (Vite + React, Next.js or Astro), add, drop or replace a backing service, and update every file the stack touches (AGENTS.md, compose, CI, Dependabot, env examples, README, ADR, first plan). Use when starting a new project from this template, when a PRD arrives, or when any stack slot changes.
---

# Start the project, or change its stack

This template separates **contracts** (root `AGENTS.md` → "Contracts that survive a stack change")
from **slots** (the Stack table). A slot can change because every other part depends only on the
contracts. Your job: turn what the project needs into slot choices, apply them, and leave every seam
consistent.

Invoking this skill is consent to scaffold the chosen framework and install its default packages.
Anything beyond that — extra libraries, a new service — still needs the human's yes.

## 1. Find out where you are

- `frontend/package.json` absent and "What this is" unfilled → **new project**. Otherwise → **change**.
- Read the root `AGENTS.md` Stack table, `frontend/AGENTS.md`'s Framework section, and the ADR index.
- Check tools: `node --version`, `pnpm --version`, `uv --version`, `docker compose version`.

## 2. Read the product documents

Look in `docs/product/` and at anything the human attached, pasted or linked — an idea note, a PRD,
research, meeting notes, a PDF. Read each one fully. If they arrived outside the repo, save them into
`docs/product/` (`idea.md`, `prd.md`, `research/…`) so the decisions can cite them.

Extract only what changes a stack decision, as a table you show the human:

| Requirement (cite the doc, and its ID if it has one) | Stack implication |
|---|---|
| Public pages must rank in search / share previews | Server or static rendering → Next.js or Astro, not a SPA |
| App behind a login, dashboard-heavy | Vite + React SPA is enough |
| Mostly content, few interactive parts | Astro |
| Live updates, collaboration, presence | SSE or WebSockets, Redis pub/sub |
| Emails, exports, anything slower than a request | A job queue → keep Redis, add a worker |
| Uploads, documents, images | Object storage (S3-compatible) — a new slot |
| Search over records | PostgreSQL full-text first; a search engine only past its limits |
| Customers with separate data | A tenancy decision → an ADR before any schema |
| AI features | A provider adapter in `backend/app/platform/`, cost and rate limits, the security-audit skill's LLM section |
| SSO, social login, MFA | An auth decision → an ADR |
| Mobile app, offline use | Out of this template's scope — say so |

Then list **what the documents leave open** — scale, hosting, auth, compliance, data residency,
budget — as questions. Never fill a gap by guessing. With no documents, go straight to section 3.

## 3. Recommend, then ask

Present one recommended stack with the requirement behind each choice, the main alternative for
each slot and why it lost, and the open questions — in one message. **Wait for the human's
confirmation** before scaffolding anything. If you cannot ask interactively, write the
recommendation into `docs/adr/0001-stack.md` as `Proposed` and stop.

Without documents, ask: what the project is (one or two sentences), which frontend (**Vite +
React**, **Next.js**, **Astro** or **none**), whether Redis is needed yet, and whether anything else
changes.

## 4. Refresh the version snapshot (new projects)

The template's pins are a snapshot of when the template was last updated, not a decision. Before
building on them, move each to the newest release **at least 7 days old**, reading each release's
notes for breaking changes:

- **Python:** newest stable minor the dependencies support → `backend/.python-version` and the
  `requires-python` lower bound; then `cd backend && uv lock --upgrade` (the 7-day window applies).
- **Postgres / Redis:** image tags in `docker-compose.yml` — the only place they are written; CI
  starts that file. Major-version notes matter here: the official `postgres` image changed its data
  directory layout at 18, so check the volume mount against the image's docs.
- **uv** in `backend/Dockerfile`; **pnpm** and **Node** come from step 5.
- After scaffolding, re-run `uv lock` / `pnpm install` and confirm nothing newer than 7 days slipped
  in (`uv lock` and pnpm refuse by themselves; scaffolders do not — see step 5.1).

## 5. The seams — every file a slot touches

| Slot | Files to change |
|---|---|
| Frontend | `frontend/**`; `frontend/AGENTS.md` Framework section; `frontend/README.md`; `CORS_ORIGINS` in `backend/.env.example`; the npm block in `.github/dependabot.yml`; CI `frontend` job (auto-runs when `frontend/package.json` exists) |
| Redis | `docker-compose.yml` `redis`; `backend/app/platform/redis_client.py`; `app/config.py` `redis_url`; `app/main.py` lifespan; `app/routers/health.py` readiness; `app/tests/conftest.py`; `backend/.env.example`; CI `REDIS_URL`; `pyproject.toml` dependency |
| Database | `docker-compose.yml` `postgres`; `app/platform/db.py`; `app/config.py`; `app/migrations/`; `alembic.ini`; readiness; conftest; `.env.example`; CI env; driver dependency |
| New backing service (object storage, mail catcher, search, worker) | a compose service with a healthcheck; a `platform/<name>.py` client behind a small interface; a setting in `config.py` + `.env.example`; a readiness check; a Stack table row; a seam row here; an ADR |
| Backend | `backend/**`; `backend/AGENTS.md`; compose `backend`; CI `backend` job; `.pre-commit-config.yaml` — see `references/backend.md` |
| Always | root `AGENTS.md` (What this is, Stack table, Layout if it moved); root `README.md`; an ADR |

Search for leftovers after a removal: `grep -rniE "redis|REDIS_URL" --exclude-dir=.venv --exclude=uv.lock .`
(adjust the term) must return nothing unexpected.

### 5.1 Scaffold the frontend

Read `references/<vite-react|nextjs|astro>.md` first; it has the commands, file contents and the
`frontend/AGENTS.md` section for that framework. The procedure is the same for all three:

1. **Scaffold into a temporary directory at the repo root** (`.scaffold-frontend/`), never into
   `frontend/` directly — scaffolders refuse or prompt for a non-empty directory. **Always pass
   `--config.minimum-release-age=10080`** to `pnpm create`: it runs outside `frontend/`, so the
   project's 7-day rule does not apply to it (verified — without the flag it fetches releases days
   old). Check the scaffolder's `--help` first; flags drift between majors. If it insists on
   interactive prompts, give the human the exact command and continue after.
2. **Move everything into `frontend/`**, except: never overwrite `frontend/AGENTS.md`,
   `frontend/CLAUDE.md` or `frontend/pnpm-workspace.yaml`. If the scaffolder wrote its own
   `pnpm-workspace.yaml`, merge its keys into ours. If it wrote `AGENTS.md` or `CLAUDE.md`
   (create-next-app does), fold their content into the Framework section and drop the files. Drop the
   generated `README.md` — you rewrite it in step 8. Remove `.scaffold-frontend/`.
3. **Pin the toolchain:** `corepack use pnpm@<newest pnpm at least 7 days old>` (writes
   `packageManager`; `scripts/setup-machine.sh` prints that version), and write `frontend/.node-version`
   with the current Node LTS major. CI reads both.
4. **`package.json` scripts:** the five contract scripts (`dev`, `build`, `lint`, `typecheck`,
   `generate:api`) with the reference's commands; add `openapi-typescript` as a dev dependency.
   (openapi-typescript 7.x declares a `typescript ^5` peer; with TypeScript 6 pnpm warns but
   generation and type-checking work — verified. Don't downgrade TypeScript over the warning.)
5. **Install** with `pnpm install` from `frontend/`. If `minimumReleaseAge` refuses a version the
   scaffolder pinned, pin that package to the newest version older than 7 days
   (`pnpm view <pkg> time --json`). If `strictDepBuilds` stops on an install script, tell the human
   which package and why it wants one; approve it only with their yes (`pnpm approve-builds`). A
   `ERR_PNPM_TRUST_DOWNGRADE` is always on a version under 30 days old (`trustPolicyIgnoreAfter`
   exempts older ones): stop, show the human the package and its publish history, and add it to
   `trustPolicyExclude` only with their yes. Never lower a setting.
6. **Wire the contracts:** `pnpm generate:api` (writes `src/lib/api-schema.d.ts`), then create
   `src/lib/env.ts` and `src/lib/api.ts` from the references, and `.env.example` + `.env` with the
   API URL variable.
7. **Backend CORS:** keep only the chosen dev server's origin in `CORS_ORIGINS` (`backend/.env.example`
   and the local `.env`).
8. **Rewrite `frontend/README.md`** as a local map — what is here, where to start reading, how to run
   it, its checks, a link to `AGENTS.md` — and **replace the Framework section** of
   `frontend/AGENTS.md` with the reference's block, adjusted to what you installed.
9. **Uncomment the npm block** in `.github/dependabot.yml`.

### 5.2 Drop, add or replace a backing service

- **Drop:** remove every file or line in its seam row, then run the leftover grep. Keep the
  readiness endpoint honest — it checks exactly what the app uses.
- **Add or replace** (object storage, Redis → Valkey, another database): propose an ADR first (`adr`
  skill) and wait. Then change the seam row's files; the contracts do not change.

## 6. Record the decisions

- Root `AGENTS.md`: fill "What this is" (from the PRD when there is one); update the Stack table's
  choices — **no versions there**, they live in the files listed under it.
- **New project:** `docs/adr/0001-stack.md` with the `adr` skill's template. Context cites the PRD;
  each decision driver is a requirement ID; alternatives rejected are the frameworks and services not
  chosen. Status `Proposed`; the human accepts it. Decisions the PRD forces beyond the stack (tenancy,
  auth, storage) get their own ADRs, also `Proposed`.
- **Change:** a new ADR superseding the affected decision of the earlier one.
- Root `README.md`: retitle it for the project, replace the intro, delete "Starting a new project".
- **First plan (offer, don't impose):** if the PRD has scope or milestones, offer to draft
  `docs/plans/<first-milestone>.md` — checkboxes in build order, each tied to a requirement ID — and
  add its row to `docs/plans/README.md`. Flag any requirement the chosen stack cannot meet.

## 7. Verify

Run everything and fix what fails before reporting:

```bash
docker compose config --quiet && docker compose up -d --wait
(cd backend && uv run ruff check . && uv run mypy && INTEGRATION=1 uv run pytest -q)
(cd frontend && pnpm generate:api && pnpm lint && pnpm typecheck && pnpm build)
python3 .agents/skills/security-audit/scripts/find_hidden_unicode.py frontend docs
```

Then start both (`uv run uvicorn app.main:app`, `pnpm dev`) and confirm the frontend's example call
to `/health` succeeds in the browser without a CORS error.

## 8. Report

What was read (documents), recommended and confirmed; what was scaffolded and installed; each file
changed per slot; any version pinned back for the 7-day rule and any install script approved; the
open questions still unanswered; and what the human must decide (accept the ADRs, take the plan).
The human commits — suggest Conventional Commit messages, e.g.
`feat(frontend): scaffold Next.js on the template contracts` and `docs(adr): propose the stack`.
