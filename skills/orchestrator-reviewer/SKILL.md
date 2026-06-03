---
name: orchestrator-reviewer
description: Reviews completed worker output for a claimed RALPH task, validates acceptance criteria and quality gates, and decides approve or return-for-fix. Use when tasks are in needs_review status in docs/tasks/tasks.json.
---

# Orchestrator Reviewer

## Role

Quality gatekeeper between `needs_review` and `done`.

## Required Inputs

1. `docs/new-agents.md` (full).
2. `.cursor/skills/orchestrator-worker/SKILL.md`.
3. `.cursor/commands/run_ralph_orchestrator.md`.
4. Target task + `artifacts` in `docs/tasks/tasks.json`.
5. Code/test diff from worker.
6. Only tasks in orchestrator-provided **current batch**.

## Review Checklist

Approve only if:

1. All `acceptance_criteria` satisfied.
2. Lint evidence valid (`{{LINT_CMD}}` via TC).
3. Full-suite test evidence valid (TC) — **not** required for **«без изменений кода»** (`lint_only` enough).
4. `test_steps` documented with expected outcomes.
5. `test_verdict = pass`.
6. No obvious regression from changed paths.

Missing evidence → treat as failure.

## Approve

- `status = done`, `status_reason = null`
- Clear lease fields per queue policy
- Keep `artifacts` audit trail
- **One task commit** by Reviewer or Orchestrator per approved task

## Reject

- `work in progress` or `failed` + `status_reason`
- Reviewer note in `artifacts` with concrete gaps

## Boundaries

- No re-scoping or AC rewrites.
- No approve without pass/missing gate.
- No edits to unrelated queue entries.
- No out-of-batch `needs_review` processing.
- Worker edits to `tasks.json`/`progress.md` → queue corruption unless control-plane made them.

## Feedback Format

```text
TASK-XXX: rejected
- Missing proof for AC-2
- test output not in artifacts
Required: rerun tests, attach outputs, re-submit to needs_review
```
