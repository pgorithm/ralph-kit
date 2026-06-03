---
name: orchestrator-dispatcher
description: Coordinates task distribution for RALPH queues, selects ready tasks by dependency and priority, performs atomic task claims, and manages lease/recovery state. Use when running or supervising parallel execution of docs/tasks/tasks.json, or when the user asks to orchestrate workers.
---

# Orchestrator Dispatcher

## Role

Dispatcher is the control-plane agent for parallel execution of `docs/tasks/tasks.json`.
It does not implement product code unless explicitly asked for emergency takeover.

## Required Inputs

1. Read `docs/new-agents.md` fully.
2. Read `.cursor/skills/orchestrator-worker/SKILL.md`.
3. Read `.cursor/commands/run_ralph_orchestrator.md`.
4. Read current `docs/tasks/tasks.json`.

## Task Selection Rules

- Only `status = "pending"` tasks are eligible.
- All `dependencies` must be `done`.
- Priority: `critical > high > medium > low`.
- Prefer tasks with low file-overlap risk for parallel starts.

## Atomic Claim Protocol

Before launching a worker:

1. Re-read `docs/tasks/tasks.json`.
2. Re-check the target task is still `pending`.
3. In one update set:
   - `status = "work in progress"`
   - `assignee = <worker-id>`
   - `claimed_at = <ISO timestamp>`
   - `lease_until = <ISO timestamp>`
4. Persist file.
5. Sanity-check diff: only intended claimed `TASK-xxx` blocks may change.
6. On unrelated changes: do not launch workers; fix queue, record `queue_corruption`, retry claim.
7. Only then start worker execution.

If claim fails due to concurrent update, re-read queue and pick another task.

## Parallel Worker Launch (K>1)

- After claiming **K** tasks, start **K separate worker sessions** (e.g. Task tool with `run_in_background=true`, one task per session). Do not serialize the whole batch in one chat when K>1 is allowed.
- Each worker prompt: task id, `assignee`, pointers to `orchestrator-worker` + `run_ralph_orchestrator.md`.
- Full test tree ownership: **Test Coordinator** only. Workers: narrow tests. **«без изменений кода»** → `lint_only`.
- Wait for **every** worker in the batch before TC or Reviewer.
- After each worker-run: verify worker did **not** edit `docs/tasks/tasks.json` or `docs/tasks/progress.md`.

## Lease and Recovery

- Expired lease without progress → reclaim.
- Recovery: `pending` (transient) or `blocked` (real blocker); clear assignee/claim fields; write `status_reason`.

## Boundaries

- Dispatcher may change only orchestration metadata and statuses.
- Dispatcher must not mark `done` without reviewer path (unless project explicitly skips review).
- Dispatcher must not delete tasks or rewrite AC.
- Dispatcher schedules TC before reviewer `done` in parallel mode.
- Dispatcher writes queue/progress after worker handoff (not the worker).

## Status Pipeline

`pending -> work in progress -> needs_review -> done`

Service: `blocked`, `failed`.

## Output Format

```text
В работе: TASK-...
На ревью: TASK-...
Блокеры: TASK-... (reason)
Готово: N/M
Следующая к запуску: TASK-...
```
