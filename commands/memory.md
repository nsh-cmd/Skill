---
description: Manage the project memory (.claude/memory/) — init scaffolds it, update refreshes stale notes, show prints the index
argument-hint: [init|update|show]
---

Manage dev-team project memory. Subcommand: $ARGUMENTS (default: `show`).

Follow the dev-team skill's `references/memory.md` convention exactly.

**init** — Create `.claude/memory/INDEX.md`. Survey the project (README, manifests, top-level layout, entry points) and write 2–5 starter `system-*.md` notes for the most load-bearing systems only, each with Purpose / Key files / How it works / Gotchas / Last verified (current commit hash). Ask the user once: commit `.claude/memory/` to git (recommended, team-shareable) or add it to `.gitignore`. If memory already exists, say so and suggest `update` instead.

**update** — Read `INDEX.md`. For each note, check staleness: `git log --oneline <last-verified-hash>..HEAD -- <key files>`. For notes whose key files changed, verify claims against current code, fix divergences, refresh `Last verified`. Report what changed. Do not rewrite fresh notes.

**show** — Print `INDEX.md` and a one-line staleness assessment per note (fresh / possibly stale / no hash recorded). If memory doesn't exist, say so and offer `init`.
