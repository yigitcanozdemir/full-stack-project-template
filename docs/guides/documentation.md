# Documentation guide

What goes in the code, and what goes in `docs/`.

## Comments and docstrings

A comment explains **why**: the constraint the code satisfies, the invariant it keeps, the failure it
prevents, or the alternative that looks right and isn't. It never restates **what** the next line
does.

It never narrates history either. "Used to", "before this", "was changed because" belong in the
commit message or an ADR. If the reason for today's shape is a past failure, state it as a present
risk: "a remount here replays the entrance animation", not "we removed the key because…".

Cross-cutting explanations live on the architecture pages; link rather than copy
(`# See docs/architecture/03-jobs.md`). Cite decisions as `ADR-0007 §2`.

**Python — Google-style docstrings.** A module docstring says what the module owns and the
invariants it keeps. A function docstring is a one-line summary, a blank line, then the contract and
its reason; add `Args:` / `Returns:` / `Raises:` only when the signature doesn't make them obvious.

**TypeScript — TSDoc on exports.** A component's header comment states what it is for and the
contract of its props. A non-obvious line gets a `//` comment explaining why.

**Size.** No more comment prose than code in a file, except in declaration-only modules (schemas,
registries). When a comment block outgrows its code, move it to the architecture page and link it.

## Documenting a new system

A **system** is anything with moving parts someone else will operate or change: a job, a pipeline,
a channel, an external integration, a token or credential, a module. Its docs ship in the same PR
(root `AGENTS.md`, Definition of done).

| You added | Write | Where |
|---|---|---|
| A cross-cutting mechanism | a system page | `docs/architecture/NN-*.md` |
| A package or module | a README — a local map | the package directory |
| Something an operator runs or repairs | a runbook | `docs/runbooks/` |
| A decision that is expensive to reverse | an ADR | `docs/adr/` (the `adr` skill) |
| An environment variable | a commented line | the service's `.env.example` |

Templates: [system page](../templates/system-page.md), [README](../templates/readme.md),
[runbook](../templates/runbook.md).

Checklist for the PR:

- [ ] The system page or README section exists and marks its status (Current / Target / Deprecated).
- [ ] It has the diagram its question needs.
- [ ] It says how the system fails and what an operator does about it.
- [ ] The owning ADR is linked, or written.
