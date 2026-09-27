# `src/lib/api.ts` — shared by every framework

The only module that calls `fetch`. It speaks the backend's error envelope
(`{"error": {"code", "message", "field?"}}`, from `backend/app/platform/errors.py`), and every domain
type is an alias onto the generated schema. Only `env.ts`'s import differs per framework.

```ts
import type { components } from "./api-schema";
import { env } from "./env";

/** Every schema the backend declares, generated from backend/openapi.json. */
export type Schemas = components["schemas"];

/** A non-2xx response, carrying the backend's stable error `code` to branch on. */
export class ApiError extends Error {
  readonly status: number;
  readonly code: string;
  readonly field?: string;

  constructor(status: number, code: string, message: string, field?: string) {
    super(message);
    this.status = status;
    this.code = code;
    this.field = field;
  }
}

type ErrorBody = { error?: { code: string; message: string; field?: string } };

export async function request<T>(path: string, init?: RequestInit): Promise<T> {
  const response = await fetch(`${env.apiUrl}${path}`, {
    ...init,
    headers: { "Content-Type": "application/json", ...init?.headers },
  });
  if (!response.ok) {
    const body = (await response.json().catch(() => null)) as ErrorBody | null;
    throw new ApiError(
      response.status,
      body?.error?.code ?? "http_error",
      body?.error?.message ?? response.statusText,
      body?.error?.field,
    );
  }
  return (response.status === 204 ? undefined : await response.json()) as T;
}

// One typed function per operation. The type is an alias, never a restated interface.
export type Health = Schemas["Health"];
export const getHealth = () => request<Health>("/health");
```

Notes:

- Class fields are declared rather than written as constructor parameter properties, because Vite's
  TypeScript template enables `erasableSyntaxOnly`, which rejects parameter properties.
- The base URL is concatenated, not passed to `new URL(path, base)`: that form drops any path prefix
  on the base (`https://host/api` + `/health` → `https://host/health`).
