# RALPH router (modes)

Этот файл — короткий роутер: какой промпт/режим использовать для работы по очередям.

## Product queue: solo

- Используй `.cursor/commands/run_ralph.md`
- Когда нужен один агент end-to-end без ролей orchestrator/test-coordinator/reviewer
- Полный контур проверки выполняет сам агент (согласно правилам `run_ralph.md`)

## Product queue: orchestrator

- Используй `.cursor/commands/run_ralph_orchestrator.md`
- Когда нужен параллельный батч и role-based контроль качества:
  - Dispatcher / Orchestrator
  - K worker sessions
  - Test Coordinator
  - Reviewer
- `docs/tasks/tasks.json` и `docs/tasks/progress.md` — single-writer зона control-plane

## Security queue (поиск и проверка уязвимостей)

- Генерация security-очереди: `.cursor/commands/generate_tasks_security.md`
- Исполнение security-аудита: `.cursor/commands/run_ralph_security.md`
- Рекомендуемый поток:
  1) сформировать `docs/tasks/security-tasks.json`;
  2) прогнать аудит задачами SEC-*;
  3) подтвердившиеся уязвимости переносить в `docs/tasks/currentProblems.md`;
  4) исправление уязвимостей выполнять отдельным продуктовым батчем (`generate_tasks` + `run_ralph`/orchestrator).

## PRD refactor queue (документация требований)

- Генерация: `.cursor/commands/generate_tasks_prdrefactor.md` → `docs/tasks/prd_refactor_tasks.json`
- Solo: `.cursor/commands/run_ralph_prdrefactor.md`
- Orchestrator: тот же `run_ralph_orchestrator.md` с PRD-очередью
