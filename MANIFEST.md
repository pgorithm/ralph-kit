# MANIFEST — ralph-kit

Список файлов шаблона для копирования в корень целевого репозитория. Пути относительны к корню [ralph-kit](https://github.com/pgorithm/ralph-kit) (каталог репозитория после clone).

## Корень

| Файл | Назначение |
|------|------------|
| `README.md` | Обзор, быстрый старт, контакты |
| `MANIFEST.md` | Этот список |

## `.cursor/rules/` (always-apply)

| Файл |
|------|
| `new-agents-gotchas.mdc` |
| `project-wide-orchestration.mdc` |

## `.cursor/commands/`

| Файл | Назначение |
|------|------------|
| `adapt_project_cursor.md` | Адаптация kit под репозиторий |
| `init_prd.md` | Диалог → `docs/PRD.md` |
| `init_tasks.md` | PRD → первая `docs/tasks/tasks.json` |
| `update_prd.md` | Обновление PRD по факту работ |
| `generate_tasks.md` | PRD → product-очередь |
| `generate_tasks_security.md` | Каталог уязвимостей → security-очередь |
| `generate_tasks_prdrefactor.md` | PRD hygiene → refactor-очередь |
| `ralph.md` | Роутер solo / orchestrator / security / PRD-refactor |
| `run_ralph.md` | Product queue, solo |
| `run_ralph_orchestrator.md` | Product queue, параллель + TC + reviewer (канон quickstart) |
| `run_ralph_security.md` | Security audit queue |
| `run_ralph_prdrefactor.md` | PRD refactor queue, solo |
| `run_benchmark.md` | Performance / benchmark TASK |
| `refactor_new_agents.md` | Чистка `docs/new-agents.md` |

## `.cursor/skills/`

| Skill | Роль |
|-------|------|
| `orchestrator-dispatcher` | Claim, lease, ready tasks |
| `orchestrator-worker` | Одна TASK end-to-end |
| `orchestrator-test-coordinator` | Lint + full tests на батч |
| `orchestrator-reviewer` | AC, gates, approve / return |
| `project-wide-orchestrator` | Обзор всего репозитория |
| `task-decomposition` | Ad-hoc очередь без `generate_tasks` |

## `docs/`

| Файл / каталог | Назначение |
|----------------|------------|
| `new-agents.md` | Долгосрочная память агента (gotchas, build, риски) |
| `PRD-stub.md` | Ориентир структуры PRD |
| `project/ai-assisted-config.md` | Канон путей и команд (после adapt) |
| `project/product-versioning.md` | Semver closure батча |
| `security/vulnerability-categories.md` | Каталог для security-очереди |
| `tasks/tasks.template.json` | Шаблон очереди TASK-* |
| `tasks/security-tasks.template.json` | Шаблон SEC-* |
| `tasks/prd_refactor_tasks.template.json` | Шаблон PRD-refactor |
| `tasks/progress.md` | Журнал выполненного |
| `tasks/security-progress.md` | Журнал security-волн |
| `tasks/currentProblems.md` | Подтверждённые находки → product, ТЗ, баги |

## После копирования (создаёт агент / команды, не в kit)

| Путь | Кто создаёт |
|------|-------------|
| `docs/PRD.md` | `init_prd` |
| `docs/tasks/tasks.json` | `init_tasks` / `generate_tasks` |
| `docs/tasks/security-tasks.json` | `generate_tasks_security` |
| `docs/tasks/prd_refactor_tasks.json` | `generate_tasks_prdrefactor` |
| `docs/tasks/done/` | Архив выполненных батчей задач |

## Плейсхолдеры `{{...}}`

Заменяются в **`adapt_project_cursor`**. Основные ключи — в `docs/project/ai-assisted-config.md`: `PROJECT_NAME`, `PRD_PATH`, `CURRENT_WORK_SECTION`, `LINT_CMD`, `TEST_FULL_CMD`, `TEST_TWO_PHASE_CMD`, `TEST_CMD`, `MIGRATE_CMD`, `VERSION_FILE`, `VULN_CATALOG_PATH`, `SECURITY_TASKS_PATH`, `BENCHMARK_*`, …
