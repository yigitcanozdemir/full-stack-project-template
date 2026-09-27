# Product

What we are building, for whom, and why — the documents that come before the code. Any format:
Markdown, PDF, exported HTML, a pasted chat.

| Document | Holds | Changes |
|---|---|---|
| `idea.md` | The problem, who has it, the bet — a page | Rarely; a new idea is a new file |
| `prd.md` (or `prd-<area>.md`) | Requirements with IDs (`R-12`), users, scope and non-goals, success measures | Versioned: a revision says what changed and when |
| `research/` | Market research, interviews, competitor notes, validation reports | Append only |

## How agents use these

- **Product truth.** Architecture pages, ADRs and plans derive from these. When one of them — or a
  task — disagrees with a PRD, the agent flags the difference rather than silently choosing.
- **The `init-stack` skill reads them first** when starting a project: it derives the stack
  recommendation from the requirements, cites the requirement behind each choice in ADR-0001, and
  lists what the documents leave open instead of guessing.
- **ADRs cite requirement IDs** (`Relates to: PRD R-12`), so a decision can be traced to the need
  that forced it. Give requirements stable IDs for that reason.
- These files describe intent, not the system as built. How it actually works lives in
  [`../architecture/`](../architecture/README.md).
