# Runbook — <situation>

## Symptoms

What the operator sees: the alert, the screen, the log event name (for example `readiness: database check failed`).

## Impact

Who is affected and how badly, and whether it gets worse with time.

## Diagnose

1. Numbered checks, each with the exact command or screen.
2. What each result means.

## Fix

1. Numbered steps. Commands are copy-paste ready and say where they run
   (`docker compose exec …`, `cd backend && uv run …`).
2. Irreversible steps are marked **irreversible** and say what to check first.

## Verify

How to confirm it is fixed.

## Afterwards

What to record (an incident note, a link to the logs) and what to change so it doesn't recur.
