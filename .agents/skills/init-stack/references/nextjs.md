# Next.js (App Router)

Choose for public pages that need SEO or fast first paint, server rendering, or a React app with
server-side logic next to the UI. Dev server: `http://localhost:3000`. Output: a Node server
(`output: "standalone"` for a Docker image).

## Scaffold

```bash
pnpm --config.minimum-release-age=10080 create next-app@latest .scaffold-frontend --ts --tailwind --eslint --app --src-dir \
  --import-alias "@/*" --use-pnpm --skip-install
```

Check `pnpm create next-app@latest --help` first and adjust flags to what it offers. **create-next-app
writes its own `AGENTS.md` and `CLAUDE.md`** (a "this is not the Next.js you know" block telling agents
to read `node_modules/next/dist/docs/`). Keep that instruction — it is correct, because Next.js
changes faster than model training data — by copying it into the Framework section, then drop both
generated files. It also writes a `pnpm-workspace.yaml`: merge its keys into ours rather than
replacing ours. Then follow SKILL.md section 5.1.

**Keep `next`, `react` and `react-dom` on patched versions.** React Server Components and Next.js
had critical RCE and DoS fixes in 2025–2026 (CVE-2025-55182, CVE-2026-23864, CVE-2026-23870) and
several middleware-bypass fixes. A security release is the case the 7-day exemption exists for:
`minimumReleaseAgeExclude` with the CVE and a removal date.

Add `openapi-typescript` as a dev dependency. Add TanStack Query only when a client component needs
interactive server state (live lists, optimistic updates) — ask first.

## Scripts

```json
"dev": "next dev",
"build": "next build",
"start": "next start",
"lint": "eslint .",
"typecheck": "next typegen && tsc --noEmit",
"generate:api": "openapi-typescript ../backend/openapi.json -o src/lib/api-schema.d.ts"
```

Recent Next.js versions removed `next lint` (use ESLint directly). If `next typegen` does not exist in
the installed version, use `tsc --noEmit` alone.

## Environment — two readers, by design

Next.js inlines `process.env.NEXT_PUBLIC_*` into client JavaScript **only when written literally**:
`process.env.NEXT_PUBLIC_API_URL` works, `process.env[name]` is `undefined` in the browser. So the
public module spells each variable out.

`src/lib/env.ts` — public values, safe in any component:

```ts
function required(name: string, value: string | undefined): string {
  if (!value) throw new Error(`${name} is not set — copy frontend/.env.example to frontend/.env`);
  return value;
}

/** NEXT_PUBLIC_ values are inlined into browser JavaScript: never a secret. */
export const env = {
  apiUrl: required("NEXT_PUBLIC_API_URL", process.env.NEXT_PUBLIC_API_URL),
} as const;
```

`src/lib/env.server.ts` — server-only values (secrets, internal URLs), created only when one exists:

```ts
import "server-only";
```

`import "server-only"` makes any client component that imports this file a build error, which is
what keeps a secret out of the bundle. (`pnpm add server-only`.)

`.env.example`:

```bash
# Inlined into browser JavaScript — never a secret in a NEXT_PUBLIC_ variable.
NEXT_PUBLIC_API_URL=http://localhost:8000
```

`src/lib/api.ts`: `references/api-client.md`.

## Docker

Set `output: "standalone"` in `next.config.ts` and build a multi-stage image from `.next/standalone`
when the frontend needs to be containerised; add it to `docker-compose.yml` under the `app` profile.

## `frontend/AGENTS.md` section

```markdown
## Framework: Next.js (App Router)

- **Read `node_modules/next/dist/docs/` for the installed version before writing Next.js code.** APIs,
  conventions and file structure change between majors and may differ from your training data.
  Heed deprecation notices.
- App Router under `src/app/`. **Server Components by default**; add `"use client"` only for state,
  effects or browser APIs, as low in the tree as possible.
- Data: fetch in Server Components through `src/lib/api.ts`. TanStack Query only in client components
  that need interactive server state. No fetch in `useEffect`.
- Mutations go to the FastAPI backend. Route Handlers and Server Actions are not a second backend:
  no database access and no business rules in `src/app/`.
- **Styling is Tailwind v4.** No CSS-in-JS.
- Env: `src/lib/env.ts` (public, `NEXT_PUBLIC_*`, spelled out literally) and `src/lib/env.server.ts`
  (`import "server-only"`) are the only readers of `process.env`.
- Layout: `src/app/` (routes, layouts) · `src/lib/` (env, api, api-schema) · `src/components/`
  (shared UI) · `src/features/<name>/` (domain components and hooks; features do not import each
  other).

Review rules — also flag: a secret reachable from a client component; a Server Action or Route
Handler with no auth check; a client component that could be a Server Component and pulls a heavy
library into the bundle.
```
