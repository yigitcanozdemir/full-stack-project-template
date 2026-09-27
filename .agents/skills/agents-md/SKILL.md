---
name: agents-md
description: Create, rewrite, prune or test AGENTS.md files (root and per-package) and their CLAUDE.md companions so any coding agent — Codex, Claude Code, Cursor, Copilot, Gemini CLI and others — gets only high-signal, project-specific rules. Use when asked to write, update, shorten or audit AGENTS.md or CLAUDE.md, when a new package or service directory is added, when an agent keeps making the same mistake, or after the stack changes.
---

# AGENTS.md author

Your goal is **signal density, not completeness**. An AGENTS.md is a minimal operational checklist
for coding agents. It holds only information that is:

1. project-specific,
2. non-obvious,
3. action-guiding,
4. likely to prevent a costly mistake.

**Why minimal, with evidence:** in a 2026 study across Claude Code, Codex and Qwen Code on real
GitHub issues, LLM-generated context files *lowered* task success by about 3% and raised inference
cost by more than 20%; developer-written files raised success by about 4% but cost just as much
(Gloaguen et al., "Evaluating AGENTS.md", ETH Zurich, arXiv 2602.11988). Every line is paid for on
every task, and unnecessary requirements make tasks harder. So never generate a file from a repo
scan and keep the output; write only what you verified matters.

## Where a rule should come from

- **A hard constraint** of the project: a trust boundary, a migration rule, a contract between
  packages, a command that must run.
- **An observed mistake**: an agent got something wrong in this repo, twice. Write the rule that
  would have prevented it, in the file nearest the mistake.
- Not from general knowledge the model already has. If a capable engineer new to the repo would do
  it right without being told, leave it out.

## How the files fit together

- **Root `AGENTS.md`** — rules for the whole repository that survive a stack change: what the
  project is, the stack table, layout, cross-package contracts, dependency and supply-chain policy,
  how docs are organised, definition of done, code review rules, git and agent boundaries.
- **`<package>/AGENTS.md`** — one per service or app directory: that package's stack slice, its
  check commands, layering, conventions, anti-patterns, its own review rules. It says "extends the
  root rules" and never restates them.
- **A rule goes in the shallowest file whose whole subtree it applies to.** A rule that crosses a
  package boundary (the API contract binds backend and frontend) belongs in the root.
- **Progressive disclosure.** A multi-step procedure belongs in a skill (`.agents/skills/`), loaded
  only when needed; an explanation belongs in `docs/`, linked in one line. AGENTS.md keeps the rule.

## What each tool reads

| Tool | Reads | Limits and traps |
|---|---|---|
| Codex | `AGENTS.md` from the repo root down to the working directory; `AGENTS.override.md` wins in its directory | Concatenated and truncated past `project_doc_max_bytes` (32 KiB default) — keep the root plus the deepest chain well under |
| Claude Code | `CLAUDE.md`; `@AGENTS.md` inside it imports the file. Recent versions read `AGENTS.md` directly **only when no `CLAUDE.md` exists in the working directory or above** | So a directory with an `AGENTS.md` either has a `CLAUDE.md` containing `@AGENTS.md`, or no directory in the chain has one — a root `CLAUDE.md` alone hides every nested `AGENTS.md`. Aim under 200 lines per file |
| Cursor, GitHub Copilot, OpenCode, Amp, most others | `AGENTS.md` | — |
| Gemini CLI | `GEMINI.md` by default | Point its context file setting at `AGENTS.md` (see its settings docs) |

In this repository: **every directory with an `AGENTS.md` has a `CLAUDE.md` containing exactly
`@AGENTS.md`**, which works on every Claude Code version. Put content in `CLAUDE.md` only if it is
Claude-only, below the import line.

Check sizes with `wc -lc AGENTS.md */AGENTS.md`.

## What an AGENTS.md should contain (only if applicable)

- Critical safety constraints: migrations, API contracts, secrets, trust boundaries, compatibility.
- The exact check commands to run before finishing — only commands that really exist.
- Non-obvious workflow constraints: package-manager-only rules, codegen order, service start order.
- Repository conventions agents routinely miss; file locations that are not obvious.
- Known gotchas that have caused repeated mistakes.
- A **Code review rules** section: "Always flag" (with what breaks), "Do not report", and "Verify
  before flagging". Reviews are where agents waste the most time on noise.

## What it must not contain

- README content, onboarding, or architecture deep-dives (link `docs/architecture/` instead).
- Generic advice ("write clean code", "handle errors", "add tests").
- Rules already enforced by lint, type-check, formatter or CI — unless there is a known exception.
- Versions (they live in manifests and go stale here), duplicated rules, aspirational rules nobody
  enforces, anything stale or uncertain, long examples.
- Secrets, internal URLs with credentials, or an instruction to fetch and follow remote content.

## Style

- "Must / never" rules over recommendations; bullets over paragraphs.
- Give a rule that looks arbitrary **one clause of reason** ("…because Vite inlines `VITE_` values
  into browser JS"). Agents follow rules they understand and generalise them correctly.
- Name things as the code names them (paths, commands, symbols) so `grep` finds them.
- Omit a section with nothing high-signal. If something is uncertain, omit it rather than invent it.

Preferred root structure (adapt): What this is · Stack · Layout · Contracts · Dependency policy ·
Code style · How we document · Definition of done · Code review rules · Git and commits · Agent
boundaries. Preferred package structure: Stack · Checks · Layout · Layering · Conventions ·
Anti-patterns · Code review rules.

## Modes

**Create** — read the manifests (`pyproject.toml`, `package.json`), CI workflows, compose file,
`docs/product/` and existing docs first. Write only what you verified.

**Rewrite** — remove low-value and generic content aggressively, deduplicate across the root and
nested files, turn vague language into explicit rules, keep every truly critical project-specific
constraint, shorten relentlessly without losing meaning.

**Sync after a stack change** — diff the Stack table and each nested file against the manifests.
Remove rules for tools no longer installed, add the new slot's rules (for a frontend, the
`init-stack` skill's `references/<framework>.md` has the block), update check commands.

**Fix a repeated mistake** — find the file nearest the code involved, add the one rule that would
have prevented it, and check no existing rule already said so (if one did, make it sharper instead).

## Test it

Instructions files are trusted like a system prompt, so treat an edit as code:

1. **Probe** — in a fresh agent session, ask two or three questions the file must answer ("What
   must you run before finishing a backend change?", "Where may environment variables be read?"). A
   wrong answer means the rule is missing, buried or ambiguous.
2. **Scan** — run the `security-audit` skill's `scripts/find_hidden_unicode.py` on the files you
   changed; instruction files must contain nothing a reviewer cannot see.

## Output

Write the files directly. In chat, report only what changed and why, in ten bullets or fewer — what
you cut, what you added, what the probe showed, and anything left out because you could not verify
it.

## Self-check before finishing

- Every bullet is project-specific or prevents a real mistake; no generic advice remains.
- No duplication across the root and nested files; no versions.
- Every command and path you wrote exists.
- Each `AGENTS.md` has its `CLAUDE.md` companion; the chain is under the size budgets.
- A coding agent could use the file immediately, mid-task, as a checklist.
