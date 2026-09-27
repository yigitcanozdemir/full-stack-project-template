# Frontend — Agent Instructions

The web app. Read the root `AGENTS.md` first. The first half of this file holds for any framework;
the framework section below is written by the `init-stack` skill from
`.agents/skills/init-stack/references/<framework>.md`.

## The contract every frontend here keeps

CI and the other slots depend on these, whichever framework is installed.

- **pnpm only.** The lockfile is `pnpm-lock.yaml`; a `package-lock.json` or `yarn.lock` is a mistake —
  delete it. `package.json` pins `packageManager: "pnpm@<version>"` and `.node-version` pins Node;
  CI reads both, and Corepack would otherwise fetch the newest pnpm unvetted.
- **pnpm settings live in `pnpm-workspace.yaml`**, not `.npmrc` (pnpm 11+ reads only registry and
  auth there). It refuses releases younger than 7 days, trust downgrades, git/tarball transitive
  dependencies, and unapproved install scripts. A fresh version you genuinely need goes in
  `minimumReleaseAgeExclude` with the CVE and a removal date. Never lower `minimumReleaseAge`.
- **These `package.json` scripts exist, under these names.** CI calls nothing else:

  | Script | Does |
  |---|---|
  | `dev` | the dev server |
  | `build` | the production build; fails on a type error |
  | `lint` | ESLint (or the framework's linter) |
  | `typecheck` | type-check with no output |
  | `generate:api` | `openapi-typescript ../backend/openapi.json -o src/lib/api-schema.d.ts` |

- **`src/lib/api-schema.d.ts` is generated, committed, never hand-edited.** Regenerate after every
  backend API change; CI fails when it is stale.
- **Domain types alias the generated schema** —
  `type Order = components['schemas']['OrderOut']`, never a hand-written interface with the same
  fields. If the shape you need is missing, the backend is missing it: fix the backend and regenerate.
- **All HTTP goes through `src/lib/api.ts`** over native `fetch`. Components never call `fetch` or
  hardcode a URL. No axios or other HTTP library.
- **`src/lib/env.ts` is the only environment reader.** It fails fast on a missing value. A variable
  with the framework's public prefix (`VITE_`, `NEXT_PUBLIC_`, `PUBLIC_`) is inlined into browser
  JavaScript: it is never a secret.
- **One styling system and one server-state library**, named in the framework section. A second of
  either needs an ADR.

## Checks (run from `frontend/`)

```bash
pnpm lint && pnpm typecheck && pnpm build
pnpm generate:api      # after a backend API change
```

## Framework

<!-- init-stack: replace this section with the "frontend/AGENTS.md section" block from
     .agents/skills/init-stack/references/<vite-react|nextjs|astro>.md -->
_Not chosen yet._ Nothing is installed in this directory until the `init-stack` skill scaffolds one.

## Anti-patterns (rejected)

- Reading the environment outside `src/lib/env.ts`; a secret in a public-prefixed variable.
- `fetch` in a component, or a hardcoded API URL.
- A hand-written response type, or editing `api-schema.d.ts` by hand.
- `npm install` / `yarn`; a second lockfile.
- `any` to silence the type-checker — use `unknown` and narrow.

## Code review rules

Extends the root rules for changes under `frontend/`.

**Losing the user's work is the P0 here.** Flag a save that writes back a stale copy of fields it does
not own, a refetch that overwrites what someone is typing, or a navigation that drops an open editor
without asking. Name the value that disappears.

**Server state:** a query key missing a value the query depends on (a stale entry survives switching
record); a mutation that does not invalidate every list, detail and count it changed.

**Do not report:** styling or class-name choices, component-splitting preferences, a missing `useMemo`
with no measured problem.
