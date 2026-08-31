---
description: Fan one implementation task across N parallel attempts in isolated git worktrees, compare results, merge the winner
argument-hint: <n> <task description>
---

Run a dev-team parallel fan-out: $ARGUMENTS

The first token of the arguments is N (number of attempts, default 2 if omitted or not a number; cap at 3 unless the user insists); the rest is the task.

Follow the dev-team skill's `references/parallel.md` recipe exactly:

1. Confirm the task suits fan-out (genuine ambiguity or high risk, objective way to judge). If not, say so and recommend the normal path — with the ~N× token cost stated either way.
2. Freeze ONE brief: goal, acceptance criteria, constraints, judging rubric (+ per-attempt strategy assignment only if exploring named alternatives).
3. Create N worktrees: `git worktree add ../<repo>-fanout-<slug>-<i> -b fanout/<slug>-<i>`.
4. Launch N implementer subagents in ONE message, each confined to its own worktree, each with the frozen brief and memory pointers.
5. Judge: test oracle first (failures are out), then rubric (diff size, convention fit, complexity). Optionally parallel code-reviewer passes on survivors.
6. Present the comparison table with a recommendation; the user picks.
7. Merge the winner into the working branch; remove ALL fan-out worktrees and branches; `git worktree prune`. Cleanup is not optional.
