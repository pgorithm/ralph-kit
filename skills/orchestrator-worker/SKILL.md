---
name: orchestrator-worker
description: Executes one claimed RALPH task end-to-end under orchestrator control, implements acceptance criteria, runs quality gates, and prepares review artifacts. Use when a worker agent is assigned a task from docs/tasks/tasks.json.
---

# Orchestrator Worker

## Role

Worker executes exactly one assigned task and prepares it for review.

## Required Inputs

1. `docs/new-agents.md` (full).
2. `.cursor/commands/run_ralph_orchestrator.md` (handoff, batch sync; do not self-approve `done` when review/TC required).
3. Assigned task in `docs/tasks/tasks.json`.

## Start Conditions

- `status = work in progress`
- `assignee` matches this worker
- Dependencies `done`

Otherwise stop and return control to dispatcher.

## Execution Scope

- One task only; all `acceptance_criteria`.
- No unrelated code changes.
- **Do not** edit `docs/tasks/tasks.json` or `docs/tasks/progress.md`.
- **Do not** commit; prepare diff + handoff for orchestrator.

## Mandatory Quality Gate

In repo root:

1. `{{LINT_CMD}}`
2. Narrow tests from `test_steps` or smallest safe subset for changed paths — **unless** task is **«без изменений кода»** (docs/legal/queue only, no edits to application source, `tests/`, or project scripts): then skip full test runner.

Then execute all manual `test_steps` and record outcomes.

Full-suite tests for `done` in parallel mode: **Test Coordinator** (`{{TEST_TWO_PHASE_CMD}}` preferred when stable, else `{{TEST_FULL_CMD}}` — see `docs/new-agents.md`). Non-code tasks: `lint_only`.

## Handoff Report

Orchestrator updates queue/progress. Include:

1. Task id and worker id.
2. Change summary.
3. Files changed for this task.
4. Artifacts: lint command/result, targeted tests, `test_steps` outcomes, note «awaiting test-coordinator full-suite verdict».
5. Blocker/conflict report with paths if applicable.

On gate failure: do not edit queue; report failure scope and proposed `status_reason`.

## Boundaries

- Do not set `done` when `review_required = true`.
- Do not re-prioritize queue or reassign other tasks.
- Do not set `needs_review` (orchestrator only).
- Do not revert unrelated parallel dirty-tree changes.
