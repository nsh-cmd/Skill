# Pipeline reference

The full specification for running a dev-team pipeline task. The orchestrator (main conversation) owns all of this — subagents only produce stage deliverables.

## Directory layout

Each pipeline task gets a checkpoint directory:

```
.claude/pipeline/<task-slug>/
├── state.json        # machine-readable pipeline state (schema below)
├── 01-plan.md        # planner output
├── 02-design.md      # architect output
├── 03-implementation.md  # implementer summary (code lives in the repo)
├── 04-tests.md       # test-engineer report
├── 05-review.md      # code-reviewer report (+ security-auditor report if run)
└── 06-ship.md        # devops/docs output + ship record
```

`<task-slug>`: lowercase, hyphenated, ≤40 chars, derived from the task title (e.g. `add-oauth-login`).

## state.json schema

```json
{
  "task": "Add OAuth login",
  "slug": "add-oauth-login",
  "created": "2026-08-31T12:00:00Z",
  "mode": "gated",
  "current_stage": "design",
  "stages": [
    {
      "name": "plan",
      "agent": "planner",
      "status": "approved",
      "output_file": "01-plan.md",
      "revisions": 0,
      "approved_at": "2026-08-31T12:10:00Z"
    }
  ]
}
```

- `mode`: `"gated"` (default) or `"fast"` (user opted in at pipeline start; see gates below).
- `status` values: `pending` → `in_progress` → `awaiting_approval` → `approved` | `rejected` → (revision loop) → `done`. Stages skipped by triage are `skipped` with a `skip_reason`.
- Update `state.json` immediately after every status change — the file is the source of truth for resuming.

## The six stages

| # | Stage | Agent | Entry criteria | Deliverable | Gate |
|---|-------|-------|----------------|-------------|------|
| 1 | plan | planner | Task accepted as pipeline-worthy | `01-plan.md`: tasks, acceptance criteria, risks, out-of-scope | **ALWAYS gated** |
| 2 | design | architect | Plan approved | `02-design.md`: decisions, contracts, data flow, plan coverage | **ALWAYS gated** |
| 3 | implement | implementer | Design approved | Code in repo + `03-implementation.md` summary | Gated unless `fast` |
| 4 | test | test-engineer | Implementation complete, checks green | `04-tests.md`: criteria coverage, results, gaps | Gated unless `fast` |
| 5 | review | code-reviewer (+ security-auditor when the change touches auth, input handling, network/file access, or dependencies) | Tests green | `05-review.md`: ranked findings + verdict | Gated unless `fast` |
| 6 | ship | devops-engineer + docs-writer | Review verdict APPROVE / APPROVE WITH CHANGES with changes addressed | `06-ship.md`: docs updated, release prep, ship record | **ALWAYS gated** |

Small plans may legitimately skip stages (see triage in SKILL.md): e.g. a doc-only task skips implement-as-code stages. Record skips in `state.json`.

## Approval-gate protocol

At a gated stage boundary:

1. Run the stage agent; receive its deliverable.
2. Check the agent's self-review section is present and clean. If the deliverable is structurally deficient (missing sections, unverified claims), send it back BEFORE bothering the user — that's a revision, not a gate.
3. Write the deliverable to its `NN-stage.md` file; set status `awaiting_approval`.
4. Present a concise summary to the user (key decisions, risks, anything surprising) plus the file path for full detail. Ask: approve / request changes / abort.
5. On approve: record `approved_at`, advance `current_stage`.
6. On request changes: status `rejected`, increment `revisions`, re-run the SAME agent with the original brief + the deliverable + the user's feedback. Cap: after 3 revisions of one stage, stop and escalate — summarize the disagreement and ask the user to redirect.
7. On abort: mark the pipeline `aborted` in state.json, leave all files for post-mortem.

NEVER auto-advance past plan, design, or ship, even in fast mode. Fast mode only auto-advances implement → test → review when each deliverable's own verdict is clean (checks green, review verdict APPROVE). Any failure or REQUEST CHANGES re-engages the gate.

## Rejection/failure loops

- Test stage finds implementation bugs → route to `debugger` (mystery failures) or back to `implementer` (clear criteria misses), then re-run test stage. This is a revision of stage 3/4, not a new pipeline.
- Review verdict REQUEST CHANGES → implementer addresses findings → re-run review on the delta. BLOCKER findings always re-gate.
- Security verdict FAIL → ship stage is blocked until CRITICAL/HIGH findings are remediated and re-audited.

## Resuming

On any dev-team invocation, glob `.claude/pipeline/*/state.json`. For each with a non-terminal status, offer: resume at `current_stage`, or close as abandoned. When resuming, re-read the approved stage outputs — they are the contract; don't re-litigate approved decisions without new information.

## Ship record

`06-ship.md` must end with:

```
## Ship record
Approved by user: <yes, timestamp>
Actions taken: <commit/push/tag/release, exact refs>
Actions explicitly NOT taken: <e.g. deploy — left to user>
Memory write-back: <notes updated>
```
