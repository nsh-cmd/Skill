# Parallel fan-out reference

Orca-style pattern: fan one implementation task across N isolated attempts in git worktrees, compare, merge the winner. Expensive — use it deliberately.

## When to fan out

- The approach is genuinely ambiguous: two+ credible designs and arguing abstractly is slower than building both.
- The task is risky and self-contained: a tricky algorithm, a gnarly refactor with a clear test oracle.
- The user asked for alternatives to compare.

When NOT to: routine tasks with one obvious approach (fan-out buys nothing), tasks whose winner can't be judged objectively (no test oracle and no crisp rubric → you'll pick by vibes at N× the cost), or huge diffs (comparison becomes its own project). N=2–3 is almost always enough.

## Constraint

Only the main conversation can run this: it creates worktrees, launches the parallel agents, and judges. Subagents cannot spawn subagents.

## Recipe

1. **Freeze the brief.** One identical brief for all attempts: goal, acceptance criteria, constraints, and the judging rubric. Differences in outcome must come from approach, not from prompt drift. If exploring two named strategies, the brief may assign one per attempt ("Attempt 1: event-driven; Attempt 2: polling") — but criteria stay identical.
2. **Create worktrees.** From the repo root, one per attempt:
   ```bash
   git worktree add ../<repo>-fanout-<slug>-1 -b fanout/<slug>-1
   git worktree add ../<repo>-fanout-<slug>-2 -b fanout/<slug>-2
   ```
3. **Launch in parallel.** One message, N implementer delegations. Each prompt = frozen brief + "Work ONLY inside <worktree path>. Do not touch any other checkout." + memory pointers.
4. **Collect.** Each attempt returns its implementation summary. Record per attempt: files changed, checks run, self-reported status.
5. **Judge objectively first.** Run the test oracle against each worktree (delegate to test-engineer per attempt, or run the suite yourself). An attempt that fails the oracle is out, regardless of elegance.
6. **Judge qualitatively second.** For survivors, compare with the rubric:
   - Tests pass (gate, from step 5)
   - Diff size / blast radius (smaller wins, all else equal)
   - Convention fit with the existing codebase
   - Complexity: which version will be cheaper to live with
   Optionally delegate code-reviewer on each survivor's diff (read-only, safe to parallelize).
7. **Present.** Show the user the comparison table and your recommendation. The user picks (or pre-authorized you to pick at fan-out start).
8. **Merge the winner.** From the main checkout: `git merge fanout/<slug>-<w>` (or rebase/squash per the repo's convention).
9. **Clean up. Always.**
   ```bash
   git worktree remove ../<repo>-fanout-<slug>-1 --force
   git branch -D fanout/<slug>-1
   # ...for every non-winning attempt; remove the winner's worktree too once merged
   git worktree prune
   ```

## Failure handling

- One attempt errors/blocks → continue with survivors; note the failure in the comparison.
- ALL attempts fail the oracle → the brief or the plan is wrong, not the agents. Stop, diagnose (often with debugger on the most promising attempt), re-gate with the user. Do not re-roll the same brief hoping for luck.
- Session dies mid-fan-out → worktrees and branches survive on disk; `git worktree list` recovers state. Record fan-out status in the checkpoint dir if inside a pipeline.

## Cost honesty

N attempts ≈ N× implementation tokens plus judging overhead. Say so when proposing a fan-out, and prefer N=2 unless the decision space genuinely has three shapes.
