# Architecture

How the system works and why — **one page per mechanism**, and the only place a cross-cutting
mechanism is explained. Package READMEs link here instead of re-explaining.

Start pages from [`../templates/system-page.md`](../templates/system-page.md). Number them in reading
order (`01-system-context.md`, `02-containers.md`, …) and list them below.

## Every section says which state it is in

- **Current** — built and running. Names real modules, classes, tables and channels, spelled the way
  the code spells them, so `grep` finds them.
- **Target** — decided but not built. Links the ADR or plan that decides it.
- **Deprecated** — being removed, or planned and never built. Says what replaces it.

A reader must never have to guess whether a sentence describes the code or a wish. Change the page in
the PR that changes the mechanism.

## Diagrams — one Mermaid type per question

| Question | Type |
|---|---|
| What exists and who talks to whom | `flowchart`, at one zoom level |
| What happens, in what order, across processes | `sequenceDiagram` |
| What states can this thing be in | `stateDiagram-v2` |
| What is stored and how it relates | `erDiagram` |

About 15 nodes at most; label every edge with what crosses it; use the code's names.

## Pages

_None yet._
