---
description: Run the full dev-team lifecycle (plan → design → implement → test → review → ship) on a task, with approval gates and resumable checkpoints
argument-hint: <task description>
---

Run the dev-team pipeline on this task: $ARGUMENTS

Follow the dev-team skill's orchestrator playbook exactly (skill: `dev-team`; stage spec: its `references/pipeline.md`):

1. Read `.claude/memory/INDEX.md` and relevant notes first (offer `/dev-team:memory init` if memory doesn't exist).
2. Check `.claude/pipeline/*/state.json` for unfinished pipelines; if one matches this task, offer to resume it instead of starting over.
3. Ask the user one question before starting: gated mode (approve every stage) or fast mode (auto-advance implement/test/review when clean; plan, design, ship always gated).
4. Create `.claude/pipeline/<task-slug>/` with `state.json`, then run the stages: planner → architect → implementer → test-engineer → code-reviewer (add security-auditor in parallel if the change touches auth, input handling, network/file access, or dependencies) → devops-engineer + docs-writer.
5. At every stage: complete handoff prompt (checkpoint path, prior outputs, memory pointers, constraints), verify the deliverable against its schema, write the `NN-stage.md` file, update `state.json`, gate per mode.
6. Finish with the ship record and memory write-back.
