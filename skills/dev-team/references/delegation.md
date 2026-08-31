# Delegation reference

How the orchestrator routes work to agents and writes handoff prompts. Only the main conversation delegates — subagents can never spawn subagents, so never instruct an agent to "have the reviewer check this"; it comes back to you.

## Routing table

| Situation | Agent | NOT |
|-----------|-------|-----|
| Vague/large request needs scoping | planner | architect (design ≠ scope) |
| Approved plan needs structure/contracts | architect | implementer guessing structure |
| Approved design needs code | implementer | — |
| Code exists, needs proof it works | test-engineer | implementer testing own work at ship gates |
| Test fails and the WHY is unclear | debugger | implementer patching symptoms |
| Test fails on a clear, named criteria miss | implementer | debugger (no mystery to solve) |
| Diff needs quality judgment | code-reviewer | — |
| Change touches auth/input/network/files/deps | security-auditor (alongside review) | skipping because "it's internal" |
| CI/build/release/version work | devops-engineer | implementer hacking CI config |
| Docs/README/changelog | docs-writer | implementer writing docs as an afterthought |

## Handoff prompt anatomy

Every delegation prompt MUST contain these five parts. Agents are stateless — they see nothing of this conversation except what you put in the prompt.

1. **Task** — what to do and the definition of done for THIS delegation.
2. **Checkpoint context** — task slug, checkpoint dir path, which stage this is, pointer to prior-stage output files to read.
3. **Memory pointers** — the specific `.claude/memory/` notes relevant to this task (not "read all memory").
4. **Constraints** — anything decided in conversation the agent must honor (user preferences, approved decisions, scope lines).
5. **Return format** — remind it of its deliverable schema when precision matters (it's in the agent definition, but reinforcement prevents drift on long tasks).

## Template

```
Task: <one paragraph: goal + definition of done>

Pipeline context: task slug `<slug>`, stage <N> (<stage name>).
Checkpoint dir: .claude/pipeline/<slug>/
Read first: <NN-prior-stage.md files>, .claude/memory/INDEX.md,
and memory notes: <specific notes>.

Constraints:
- <approved decision the agent must not reverse>
- <user preference from conversation>

Return your standard <agent> deliverable. Include your Memory Duty section.
```

For non-pipeline (single-agent) delegations, drop the pipeline context block but keep memory pointers and constraints.

## Anti-patterns

- **Delegating trivia.** A one-line fix, a rename, a quick question about the code — do it yourself in the main conversation. A delegation costs a full context spin-up; spend it where isolation or specialization pays.
- **Chaining without checkpointing.** Never feed agent B from agent A's output without first writing A's output to the checkpoint dir. If the session dies mid-chain, un-checkpointed work is gone.
- **Sending an agent in blind.** A delegation prompt without memory pointers and prior-stage outputs forces the agent to re-explore everything — the exact tax this system exists to avoid.
- **Wrong-direction escalation.** When an agent returns BLOCKED or reports a plan/design contradiction, the decision comes back to you and (if it changes approved decisions) to the user — never "just have the agent figure it out".
- **Re-litigating approved stages.** Don't let a later agent's preference reopen an approved decision without new facts. New facts → surface to the user at the gate.
- **Parallel agents sharing a working tree.** Two agents editing one checkout corrupt each other. Parallel implementation attempts require worktrees (see parallel.md); parallel READ-ONLY agents (e.g. reviewer + security-auditor on the same diff) are fine and encouraged — launch them in one message.
