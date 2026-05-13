# Project

Project ID for Plane: read from `.plane` file → `$(cat .plane)`

## Plane Rules
- Bug found → create a Plane task immediately
- Starting work → move task to In Progress
- Feature done → move task to Done
- Commits → always include sequence ID (e.g. PROJ-12)
- Never ask me to manage tasks manually

## Plane API
Full API reference is in `.plane-api.md` — read it before making any Plane API calls.

Env vars: $PLANE_URL, $PLANE_API_KEY, $PLANE_WORKSPACE (set in shell, never hardcode)
