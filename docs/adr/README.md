# Architecture Decision Records

An ADR records one non-obvious decision and the reasoning behind it, so a choice that looks arbitrary
later — an unused field, a chosen library, a rejected approach — can be understood without re-running
the discussion. The architecture pages explain the standing system; an ADR captures a decision made
while building it.

## When to write one

When a choice is not already settled here or in `docs/architecture/`, and it is expensive to reverse
or easy to misread later: adding or rejecting a dependency, choosing or changing a stack slot, a
schema decision that constrains the future, reserving something for a capability not built yet.

The agent stops, proposes the ADR, and waits for a human decision before coding. Routine work that
follows existing rules needs none.

## How

The `adr` skill writes one (`/adr` in Claude Code, `$adr` in Codex); its `SKILL.md` has the template.
In short: `NNNN-short-title.md`, about a page — context, decision drivers, the options considered,
the decision, its consequences, how it is confirmed (a test, lint rule or CI job), and when to
revisit it. Status moves `Proposed → Accepted` or `Rejected` only when a human decides; rejected
proposals stay listed, because "we considered X" is worth keeping. Accepted ADRs are immutable — a
later ADR supersedes, and the old one's status line says so. An ADR cites PRD requirement IDs from
`docs/product/`, never a plan.

## Index

| ADR | Title | Status |
|---|---|---|

> **The next free number is 0001** — check this line, not the file listing. A gap in the table is a
> retired or reserved number, not a missing file.
