# Astro

Choose for content-heavy sites — marketing, docs, blogs — that are mostly static with a few
interactive parts (islands). Dev server: `http://localhost:4321`. Output: static files in `dist/`, or
a server with an adapter when pages must render per request.

## Scaffold

```bash
pnpm --config.minimum-release-age=10080 create astro@latest .scaffold-frontend --template minimal --no-install --no-git --skip-houston --yes
```

Check `pnpm create astro@latest --help` first. Then follow SKILL.md section 5.1. After installing, from
`frontend/`:

```bash
pnpm astro add tailwind          # Tailwind v4 through @tailwindcss/vite
pnpm add -D @astrojs/check typescript openapi-typescript
pnpm add -D eslint eslint-plugin-astro typescript-eslint @eslint/js
```

Add a UI framework (`pnpm astro add react`) only when an island needs one — ask first.

`eslint.config.js`:

```js
import js from "@eslint/js";
import astro from "eslint-plugin-astro";
import tseslint from "typescript-eslint";

export default [
  { ignores: ["dist/", ".astro/", "src/lib/api-schema.d.ts"] },
  js.configs.recommended,
  ...tseslint.configs.recommended,
  ...astro.configs.recommended,
];
```

## Scripts

```json
"dev": "astro dev",
"build": "astro check && astro build",
"preview": "astro preview",
"lint": "eslint .",
"typecheck": "astro check",
"generate:api": "openapi-typescript ../backend/openapi.json -o src/lib/api-schema.d.ts"
```

## Environment — `astro:env`

Declare the schema in `astro.config.mjs`, so a missing or mistyped value fails the build:

```js
import { defineConfig, envField } from "astro/config";

export default defineConfig({
  env: {
    schema: {
      PUBLIC_API_URL: envField.string({ context: "client", access: "public" }),
    },
  },
});
```

`src/lib/env.ts` — the only module that imports `astro:env`:

```ts
import { PUBLIC_API_URL } from "astro:env/client";

/** PUBLIC_ values are inlined into browser JavaScript: never a secret. */
export const env = { apiUrl: PUBLIC_API_URL } as const;
```

A secret is declared with `context: "server", access: "secret"` and imported from `astro:env/server`
in a server-only module.

`.env.example`:

```bash
# Inlined into browser JavaScript — never a secret in a PUBLIC_ variable.
PUBLIC_API_URL=http://localhost:8000
```

`src/lib/api.ts`: `references/api-client.md`.

## `frontend/AGENTS.md` section

```markdown
## Framework: Astro

- Astro, static by default. Pages render at build time; add an adapter and `export const prerender =
  false` only for pages that must render per request.
- **Ship zero JavaScript unless a part is interactive.** An island is a UI-framework component with a
  `client:*` directive; prefer `client:visible` or `client:idle` over `client:load`.
- Data: fetch in page or component frontmatter through `src/lib/api.ts`, at build or request time.
- **Content is Content Collections** (`src/content.config.ts`, schema-validated) — not ad hoc Markdown
  imports.
- **Styling is Tailwind v4** (via `@tailwindcss/vite`). Scoped `<style>` in `.astro` files is fine
  for one-offs.
- Env: `astro:env` schema in `astro.config.mjs`; `src/lib/env.ts` is the only importer of
  `astro:env/client`.
- Layout: `src/pages/` (routes) · `src/layouts/` · `src/components/` · `src/content/` · `src/lib/`
  (env, api, api-schema).

Review rules — also flag: `set:html` with data that is not sanitised; a `client:load` island that
could be static or `client:visible`.
```
