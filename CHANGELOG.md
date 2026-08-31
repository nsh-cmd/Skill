# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-08-31

### Added

- `dev-team` orchestrator skill: staged pipeline (plan → design → implement → test → review → ship) with approval gates, resumable checkpoints (`.claude/pipeline/<slug>/state.json`), task triage, and handoff discipline.
- 9 specialized subagents: planner, architect, implementer, test-engineer, code-reviewer, debugger, security-auditor, devops-engineer, docs-writer — each with identity, critical rules, workflow, deliverable schema, self-review checklist, success metrics, and memory duty.
- Project-memory convention (`.claude/memory/`): indexed system/concept/decision notes with staleness verification, read-before-explore and write-back discipline.
- Parallel fan-out recipe: N git-worktree-isolated implementation attempts, oracle-then-rubric judging, winner merge, mandatory cleanup.
- Slash commands: `/dev-team:pipeline`, `/dev-team:fanout`, `/dev-team:memory`.
- Plugin + marketplace manifests for installation via `/plugin marketplace add nsh-cmd/Skill`.
