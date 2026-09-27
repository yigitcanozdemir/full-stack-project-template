# Plans

One plan per body of work. A plan is the **working** document for what is being built now: phases or
batches, checkboxes, verification notes, and reasoning too situational for an ADR.

| Plan | Covers | State |
|---|---|---|

## The rule that keeps these readable

**A plan is closed, never extended.** When a body of work ships, its file stops taking new items and
the next body of work gets a new file:

1. create `docs/plans/<name>.md` and add a row above, state **live**;
2. mark the previous plan **closed** and leave it where it is;
3. never move items between plans — what was built under one stays recorded there.

A checklist that absorbs every later batch becomes a file nobody, human or agent, can read in one
sitting, and that cost is paid on every read. A closed plan is a record: where it disagrees with the
code, the code and the ADRs win.

## Plan or ADR?

- **A decision that is expensive to reverse goes in an [ADR](../adr/README.md), the same day.**
- **Everything else — sequencing, scope, what is left, how it was verified — goes in the plan.**

An ADR never cites a plan: the ADR outlives it. A plan that turns out to carry a real decision has
found an ADR that has not been written yet.
