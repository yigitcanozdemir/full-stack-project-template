# Vite + React (SPA)

Choose for dashboards, internal tools and apps behind a login, where SEO and server rendering don't
matter. Dev server: `http://localhost:5173`. Output: static files in `dist/`, served by any static
host or CDN.

## Scaffold

```bash
pnpm --config.minimum-release-age=10080 create vite@latest .scaffold-frontend --template react-ts --eslint --no-immediate --no-interactive
```

Check `pnpm create vite@latest --help` first. `--no-immediate` stops it installing and starting a
server (you install from `frontend/` after moving). React templates default to Oxlint; `--eslint`
keeps ESLint — if the human prefers Oxlint, drop the flag and point the `lint` script at `oxlint`.
Then follow SKILL.md section 5.1.

Libraries this slot uses by default (confirm with the human before adding any others):

```bash
pnpm add @tanstack/react-query @tanstack/react-router
pnpm add -D tailwindcss @tailwindcss/vite openapi-typescript
```

Tailwind v4 is CSS-first: add `tailwindcss()` from `@tailwindcss/vite` to `vite.config.ts` plugins
and `@import "tailwindcss";` at the top of `src/index.css`. There is no `tailwind.config.js`.

## Scripts

```json
"dev": "vite",
"build": "tsc -b && vite build",
"lint": "eslint .",
"typecheck": "tsc -b",
"generate:api": "openapi-typescript ../backend/openapi.json -o src/lib/api-schema.d.ts",
"preview": "vite preview"
```

## `src/lib/env.ts`

```ts
function required(name: string, value: string | undefined): string {
  if (!value) throw new Error(`${name} is not set — copy frontend/.env.example to frontend/.env`);
  return value;
}

/** The only reader of import.meta.env. VITE_ values are inlined into the bundle: never a secret. */
export const env = {
  apiUrl: required("VITE_API_URL", import.meta.env.VITE_API_URL),
} as const;
```

`.env.example`:

```bash
# Inlined into browser JavaScript at build time — never put a secret in a VITE_ variable.
VITE_API_URL=http://localhost:8000
```

`src/lib/api.ts`: `references/api-client.md`.

## `frontend/AGENTS.md` section

```markdown
## Framework: Vite + React (SPA)

- Vite + React + TypeScript (strict), a client-rendered SPA. No SSR, no Node server in front of it;
  `dist/` is static files.
- **Server state is TanStack Query** — no fetch in `useEffect`, no second data-fetching library.
  Query keys include every value the query depends on (record id, filters, the signed-in user).
- **Routing is TanStack Router**; routes are declared per feature and composed in `src/app/`.
- **Styling is Tailwind v4** (`@tailwindcss/vite`, CSS-first, no config file). No CSS modules,
  styled-components or Emotion.
- Client state: `useState` / `useReducer` / context first; a global store only for real cross-tree
  state, and only with an ADR.
- `import.meta.env` is read only in `src/lib/env.ts`. Every `VITE_` variable ships to the browser.
- Layout: `src/app/` (router, providers, shell) · `src/lib/` (env, api, api-schema) ·
  `src/components/` (shared presentational UI) · `src/features/<name>/` (routes, hooks, components
  for one domain; features do not import each other).
```
