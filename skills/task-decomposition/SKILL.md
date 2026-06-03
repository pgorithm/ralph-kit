---
name: task-decomposition
description: Creates and decomposes agent-executable work items into JSON task queues. Use for ad-hoc tasks, bugs, refactors, or when the user asks to decompose work without the generate_tasks command.
---

# Task Decomposition

Universal skill for **creating and splitting** agent tasks. Discover local paths first, then apply rules below.

PRD-driven batch generation: `.cursor/commands/generate_tasks.md`. Greenfield first queue: `init_tasks.md`.

## 1. Discover artifacts

| What | Typical paths | Why |
|------|---------------|-----|
| Queue | `docs/tasks/tasks.json` | Target file |
| Template | `docs/tasks/tasks.template.json` | Required template fields |
| History | `docs/tasks/progress.md` | Continue `TASK-NNN` |
| Agent rules | `agent_instructions` in template, `docs/new-agents.md` | Do not truncate |
| Requirements | `{{PRD_PATH}}`, `currentProblems.md`, user request | AC source |
| Versioning | `docs/project/product-versioning.md`, `{{VERSION_FILE}}` | `release.kind` |

Copy **`agent_instructions` from template verbatim** (with project placeholders after adapt). Do not invent a shortened block.

## 2. Release metadata (new batch)

Top-level `release` in new `tasks.json`:

| Field | Use |
|-------|-----|
| `kind` | `major` \| `minor` \| `patch` for **whole batch** |
| `baseline_version` | Optional from `{{VERSION_FILE}}` |
| `notes` | Optional scope hint for closure |

Classification (dominant content):

- **patch** — bugs, security fixes, tech debt, docs/process.
- **minor** — user-visible features.
- **major** — only with explicit PRD/user criterion (breaking API, monetization go-live, etc.).
- Mixed batch → higher digit wins (`minor` + security → `minor`).

Merge-only ad-hoc: do not change existing `release` without user request.

## 3. Decomposition rules

- **Atomic:** one agent session (~30 min focus).
- **Independent:** minimal `dependencies`.
- **Testable:** concrete `test_steps` with expected outcomes.
- **No code:** mark **«без изменений кода»**; lint + manual steps only (`lint_only` in orchestration).
- **Priority:** critical path → security/contracts → functional → UI polish.

## 4. Refactor / import zones

- One task = one zone (package, route cluster, test layer).
- Name dirs/globs in description and AC.
- Split large trees by layer with dependencies.

## 5. Ad-hoc workflow

1. Clarify goal if unclear.
2. Find template + last TASK-ID.
3. Draft tasks with category, priority, AC, test_steps, dependencies.
4. Set `release` for new batch.
5. Quality checklist (§6).
6. Merge new `pending` tasks into JSON (no silent delete of existing tasks).
7. Report: ID range, `release.kind`, dependencies, out-of-scope items.

Categories: `infrastructure` | `functional` | `ui` | `integration` | `security`

## 6. Quality checklist

- [ ] One-session scope
- [ ] Measurable AC
- [ ] Actionable test_steps
- [ ] Valid dependency IDs
- [ ] No scope overlap within batch
- [ ] Refactor boundaries explicit
- [ ] No-code tasks marked
- [ ] `agent_instructions` copied from `tasks.template.json`

## 7. Anti-patterns

| Bad | Good |
|-----|------|
| «Fix backend» | «Return 422 when sku empty on POST /v1/items» |
| Whole-repo refactor in one TASK | Zoned tasks with dependencies |
| «Run tests» | Named commands and expected result |
| Duplicate done work from progress | Skip or depend on done TASK |
| Shorten agent_instructions | Full template block |

## 8. User summary

Report: ID range, `release.kind`, table (id | priority | category | description | deps), excluded scope, suggested start order.

Optional deep reference: copy `task-decomposition/reference.md` from a mature reference repo after adapt (not shipped in kit — see MANIFEST «Не входит в kit»).
