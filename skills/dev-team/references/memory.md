# Project memory reference

A file-based convention (inspired by Graft / codebase-memory-mcp) that gives agents persistent knowledge of the project across sessions. No server, no index database — just disciplined markdown.

## Why

Without memory, every session re-pays the exploration tax: re-discovering where things live, how they connect, and which traps exist. With memory, agents read a few short notes instead of grepping the world. The convention only pays off if notes are READ before exploring and UPDATED when reality diverges.

## Layout

```
.claude/memory/
├── INDEX.md                    # the catalog — always read this first
├── system-<name>.md            # how a subsystem works (auth, billing, build...)
├── concept-<name>.md           # cross-cutting concept (error model, tenancy...)
└── decision-<date>-<slug>.md   # a significant decision and its rationale
```

## INDEX.md format

One line per note. Keep it under ~50 lines; if it grows past that, consolidate notes.

```
# Memory index
| Note | Scope | Updated |
|------|-------|---------|
| system-auth.md | Login, sessions, token refresh | 2026-08-31 (abc1234) |
| decision-2026-08-31-use-worktrees.md | Why parallel work uses worktrees | 2026-08-31 |
```

## Note template

```markdown
# <system/concept name>

## Purpose
<what this exists to do, one paragraph>

## Key files
- path/to/entry.py — <role>
- path/to/core.py — <role>

## How it works
<the mechanism in 5-15 lines: flow, invariants, dependency direction>

## Gotchas
- <trap that costs time, e.g. "X must be called before Y or the cache silently serves stale data">

## Last verified
<date> @ <commit short-hash>
```

Rules:
- Each note ≤150 lines. Memory is a map, not a mirror — link to files, don't duplicate code.
- Notes record VERIFIED facts, not plans or hopes.
- `decision-*` notes additionally record: options considered, chosen option, why, and what would trigger revisiting.

## Read discipline (all agents)

1. Read `INDEX.md` first — costs ~50 lines, saves whole exploration passes.
2. Read only notes whose scope touches the task.
3. Staleness check: if a note's `Last verified` commit is far behind HEAD (`git log --oneline <hash>..HEAD -- <key files>` shows changes to its key files), verify against code before trusting it — and flag it for update.

## Write discipline (orchestrator)

Subagents REPORT memory-worthy facts in their "Memory Duty" section; the orchestrator writes the notes (single writer avoids conflicting edits):

- New system explored in depth → create/extend `system-*.md`.
- Significant design decision approved → `decision-<date>-<slug>.md`.
- A gotcha cost an agent real time → add to the relevant note's Gotchas.
- Reality contradicted a note → fix the note NOW, update `Last verified`.
- Every write updates INDEX.md's `Updated` column.

What does NOT belong: session logs, TODO lists, anything derivable in seconds, secrets/credentials (never), opinions not grounded in code.

## Git

Default: COMMIT `.claude/memory/` — it is team-shareable project knowledge and versioning it gives history for free. If the user prefers it private, add `.claude/memory/` to `.gitignore` instead; ask once at init and respect the choice.

## Initialization (`/dev-team:memory init`)

1. Create `.claude/memory/INDEX.md`.
2. Survey the project (README, manifests, top-level structure, entry points).
3. Write 2-5 starter notes for the most load-bearing systems only — seed quality over coverage; the rest accretes as tasks touch them.
4. Ask the user: commit memory to git, or gitignore it?
