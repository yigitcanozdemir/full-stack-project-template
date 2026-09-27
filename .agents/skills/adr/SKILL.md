---
name: adr
description: Propose an Architecture Decision Record in docs/adr/ for a choice that is expensive to reverse or easy to misread later — a dependency, a stack slot, a data-model constraint, a security boundary, a rejected approach — with options, drivers and how the decision will be checked, then stop for a human decision before coding it. Use when the user asks for an ADR or decision record, or when a task hits such a choice that no PRD, architecture page or existing ADR already settles.
---

# Write an ADR

An ADR records one non-obvious decision and why, so that a choice which looks arbitrary later — an
unused field, a chosen library, a rejected approach — can be understood without re-running the
discussion. The format follows the useful parts of MADR 4: drivers, options side by side, and how
the decision is confirmed.

## Is an ADR needed?

Yes when the choice is **not already settled** — check the index in `docs/adr/README.md`,
`docs/architecture/` and `docs/product/` first — **and** it is expensive to reverse or easy to
misread. Typical triggers: adding or rejecting a dependency, choosing or changing a stack slot, a
schema decision that constrains the future, a security or trust boundary, generalising a mechanism,
reserving something for a capability not built yet.

No for routine work that follows existing rules — most tasks need none. Do not write one to look
thorough.

## Steps

1. **Read first:** the PRD requirements the decision serves (`docs/product/`), the ADRs it touches,
   and the code it affects. An ADR that contradicts an accepted one must supersede it explicitly.
2. **Number:** the next free number is stated at the bottom of the index in `docs/adr/README.md` —
   use it, not the file listing (numbers can be retired or reserved). Zero-pad to four digits.
3. **File:** `docs/adr/NNNN-short-kebab-title.md`, from the template below. About a page; if it needs
   more, it is probably two decisions.
4. **Status:** always `Proposed`. Only a human moves it on.
5. **Index:** add the row to `docs/adr/README.md` and bump the "next free number" line.
6. **Stop.** Show the human the decision, the strongest rejected option, and the main cost — then
   **wait for a decision before writing code that depends on it.**
7. **After the human decides**, set the status. `Accepted`, or `Rejected` — a rejected proposal
   stays in the index, because "we considered X and said no" is exactly what the next person asks.

## Statuses

`Proposed` → `Accepted` | `Rejected`. Later: `Superseded by ADR-NNNN` (a newer ADR replaces it) or
`Deprecated` (no longer relevant, nothing replaces it). The status line and index row are the only
edits an accepted ADR ever receives; the old ADR and its successor both link each other.

## Rules

- One decision per record. Two decisions are two ADRs.
- Accepted ADRs are immutable; revise by writing a new one that supersedes it.
- **An ADR never cites a plan** (`docs/plans/`). A plan closes; the ADR outlives it. Cite PRD
  requirement IDs, other ADRs, architecture pages or code.
- Name things as the code names them, so `grep` finds them.
- At least two real options, including "do nothing / keep what we have" when that is viable. A
  decision with no downside has not been thought through.
- **Confirmation is concrete** — a test, a lint rule, a CI job, a review rule in `AGENTS.md` — or it
  says honestly that the decision is checked only by review.

## Template

```markdown
# ADR-NNNN — <short title: the decision, not the topic>

- **Status:** Proposed
- **Date:** YYYY-MM-DD
- **Relates to:** <PRD requirement IDs, ADRs, architecture pages; "supersedes ADR-MMMM" if it does>

## Context

What forces the decision now: the problem, the constraints, and what is already fixed elsewhere.

## Decision drivers

- <the requirement, quality or constraint that decides between the options>

## Considered options

1. **<option>** — one line on what it is.
2. **<option>**
3. **Keep what we have** — if viable.

## Decision

What we are doing, stated plainly. Numbered if it has parts.

## Consequences

- **Good:** what this makes easy.
- **Bad:** what this makes hard, and what it commits us to.

## Confirmation

How we will know it is followed: the test, lint rule, CI job or review rule that enforces it.

## Options compared

| Driver | <option 1> | <option 2> | <option 3> |
|---|---|---|---|
| <driver> | … | … | … |

For each rejected option, one line on why it lost.

## Revisit when

The observable change that should reopen this (a scale threshold, a new requirement, a dependency's
end of life).
```
