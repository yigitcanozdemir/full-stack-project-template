# Agent Instructions

The source of truth for every coding agent in this repository — Codex reads it directly, Claude Code
through `CLAUDE.md`. Read it, then the nearest `AGENTS.md` to the code you are changing
(`backend/AGENTS.md`, `frontend/AGENTS.md`). The nearer file wins where the two conflict.

## What this is

<!-- init-stack: one or two sentences — the product, who uses it, the current target. -->
_Not described yet. Run the `init-stack` skill when starting a project from this template._
Product documents (idea, PRD, research) are in `docs/product/`.

## Stack

| Slot | Choice | Rules |
|---|---|---|
| Backend | Python · FastAPI · Pydantic · SQLAlchemy (async) · Alembic · uv | `backend/AGENTS.md` |
| Database | PostgreSQL | `backend/AGENTS.md` |
| Cache / queue | Redis | `backend/AGENTS.md` |
| Frontend | _Not chosen_ — Vite + React, Next.js or Astro, via the `init-stack` skill | `frontend/AGENTS.md` |
| Local infra | Docker Compose | `docker-compose.yml` |

**Versions are written once, in the file that uses them** — never in this table, a README or CI:
`backend/.python-version` (Python, also used by the Docker image), `backend/uv.lock`,
`docker-compose.yml` (Postgres, Redis; CI starts the same file), `frontend/.node-version`,
`frontend/package.json` (`packageManager`, dependencies). Dependabot proposes updates.

A chosen stack is locked. Changing a slot takes an ADR plus the `init-stack` skill, which lists every
file a slot touches. Do not propose alternatives without a concrete reason.

## Layout

```text
.
├── AGENTS.md, CLAUDE.md   # agent rules; CLAUDE.md only imports this file
├── .agents/skills/        # project skills (Codex); .claude/skills is a symlink to it (Claude Code)
├── backend/               # API service — backend/AGENTS.md
├── frontend/              # web app — frontend/AGENTS.md
├── docs/                  # product/, adr/, architecture/, plans/, guides/, templates/ — docs/README.md
├── scripts/               # setup-machine.sh (per-machine supply-chain settings) and automation
└── docker-compose.yml     # local Postgres + Redis; the backend image under the `app` profile
```

## Contracts that survive a stack change

A new framework in any slot must keep these. They are why a slot can be swapped without the others
noticing.

1. **One settings module per service reads the environment** — `backend/app/config.py`,
   `frontend/src/lib/env.ts`. Nothing else calls `os.getenv`, `load_dotenv`, `process.env` or
   `import.meta.env`. Missing required config fails at startup. Test harnesses are exempt.
2. **The API contract is generated, never written twice.** The backend writes `backend/openapi.json`
   (committed); the frontend generates `src/lib/api-schema.d.ts` from it. A frontend domain type is an
   alias onto the generated schema, never a hand-written interface with the same fields — nothing
   compares the two, so the drift ships as a runtime `undefined`. An API change is not done until both
   files are regenerated; a backend test and CI fail otherwise.
3. **The backend serves generic resources** with filter, sort and pagination parameters — never an
   endpoint shaped for one screen or chart. A frontend redesign must not need a backend change.
4. **Schema changes are generated migrations.** Never hand-edit an applied revision; never create
   tables from startup code.
5. **Each service is one Docker image configured only by environment variables.** Every variable is
   in that service's `.env.example` with a comment.

## Dependency policy

Write it yourself unless a library avoids non-trivial, error-prone or standard reinvention — drivers,
crypto, migrations, parsers, SDKs. Not a wrapper around a few standard-library lines, a "nicer API"
over something already installed, or a second library for a job one already does.

Adding a runtime dependency: the commit body says what it does that can't be written clearly in under
30 lines, and what it pulls in.

### Supply chain

Compromised releases (LiteLLM, axios, the keyv worm — all 2026) were live for minutes to hours
before removal. Every layer below exists because one of them got through somewhere.

- **Nothing younger than 7 days is installed.** `exclude-newer` in `backend/pyproject.toml`;
  `minimumReleaseAge` in `frontend/pnpm-workspace.yaml` (pnpm 11+ ignores `.npmrc` for it). Never
  lower either.
- **One-off tools skip project config.** `uvx`, `pnpm dlx`, `pnpm create` and `npx` read only
  user-level settings: pass `uvx --exclude-newer "7 days"` / `pnpm --config.minimum-release-age=10080`,
  or run `scripts/setup-machine.sh --apply` once per machine.
- **A fix you need now is the one exception** — exempt that single package (`exclude-newer-package`
  / `minimumReleaseAgeExclude`) with a comment naming the CVE and a removal date. Dependabot security
  updates bypass the delay on their own.
- **Install scripts stay blocked** (`strictDepBuilds`); approving one (`pnpm approve-builds`) is a
  reviewed change.
- **CI actions and pre-commit hooks are pinned to commit SHAs**, the token is read-only, and CI audits
  both lockfiles for known CVEs weekly.
- **Agent config is code.** A change to `AGENTS.md`, `CLAUDE.md`, `.agents/skills/`, `.claude/`,
  `.codex/`, `.mcp.json` or `.vscode/` can make an agent run commands (the keyv worm persisted through
  Claude Code hooks and VS Code tasks). Review it like code.

## Code style

- Small, obvious functions and focused files. Extract on the third real caller, not before; no
  speculative abstractions, no feature flags nobody asked for.
- Validate at boundaries — HTTP input, external APIs, DB writes, untrusted parsing. Trust internal
  callers.
- Comments explain why, never what, and never narrate history ("used to…" belongs in the commit).
  `docs/guides/documentation.md`.
- No backwards-compatibility shims unless asked.

## How we document

Do not invent new standing documents. Each kind of page has one home (`docs/README.md`):

- **`docs/product/`** — what we are building and why: idea, PRD, research. Product truth: when a
  plan, ADR or task disagrees with it, flag the difference rather than silently choosing one.
- **`docs/architecture/`** — how a mechanism works and why. Every section is marked **Current**,
  **Target** or **Deprecated**, so a plan never reads as a feature. Change it in the PR that changes
  the mechanism.
- **`docs/adr/`** — one expensive-to-reverse decision per record. If a task hits a choice nothing
  already settles and it is expensive to reverse or easy to misread later, **stop, propose an ADR
  (the `adr` skill), and wait for a human decision before coding.** Routine work needs none.
- **`docs/plans/`** — working checklists, one file per body of work. A finished plan is closed, never
  extended. An ADR never cites a plan: the ADR outlives it.
- **Package `README.md`** — a local map: what is here, where to start reading, links up. Never a
  second explanation of an architecture page.
- **`docs/runbooks/`** — what to do when a running system is broken. Created on first need, from
  `docs/templates/runbook.md`.

## Definition of done

- **Checks pass** — each touched package's checks, listed in its `AGENTS.md`. CI runs the same.
- **The schema is migrated** — a model change ships its autogenerated, reviewed revision.
- **The API contract is current** — `backend/openapi.json` and the frontend's generated types.
- **The docs moved with the code** — a new system (job, integration, credential, pipeline, module)
  ships its architecture section or README; a decision that is expensive to reverse has its ADR.

`.github/pull_request_template.md` is this list as a checklist.

## Code review rules

Every review of this repository. Nested `AGENTS.md` files add rules for their paths.

**Report only what changes the outcome**, ranked by blast radius. Cite `file:line` and say what
breaks — a rule name alone is not a finding.

Always flag:

- **Secrets, tokens or full PII** in code, logs, errors, responses or a client bundle.
- **Silent data loss** — a write that erases or reverts values the user did not touch. Name the value.
- **A fallback that overwrites stored intent**, or that fires without a trace.
- **A contract broken** — a hand-written type mirroring a generated one, an API change without
  regenerated contract files, config read outside the settings module.
- **Blocking I/O in async code.**
- **A supply-chain weakening** — a release-age setting lowered or removed, an exemption with no CVE
  and removal date, an action or hook referenced by tag, an approved install script, or an agent
  config change that runs commands.
- **A new system with no doc page**, or a `docs/architecture/` claim the diff makes false.

Do not report: what lint, type-check or the formatter already fail on; style preferences these files
don't state; pre-existing issues the diff didn't make worse; missing tests unless tied to a path that
can silently break.

**Verify before flagging.** A behaviour claim needs a `file:line` you read, not an inference from a
name. Check `docs/adr/` first — a deliberate decision is not a bug.

## Git and commits

- **The human runs git** — branch, stage, commit, push, open PRs. Agents edit the working tree and say
  when a change is ready to commit. Workflow and naming: `CONTRIBUTING.md`.
- Conventional Commits; the body says **why**. One logical change per commit.
- No AI attribution trailers (`Co-authored-by:` naming an assistant, model or tool).

## Agent boundaries

- Run the checks the nested `AGENTS.md` files list, and `docker compose up` for local infra, freely.
- Ask first before: adding, upgrading or removing a dependency (lockfile changes); migrating any
  database that is not local; deleting data or volumes (`docker compose down -v`); anything that
  leaves this machine.
