# run_ralph_prdrefactor — solo: рефакторинг PRD

Работа по **`docs/tasks/prd_refactor_tasks.json`** (или архиву в `docs/tasks/done/` после closure). Только документ **`{{PRD_PATH}}`**, без изменения application-кода.

## Solo-режим

- Статусы: `pending -> work in progress -> done`
- Не использовать orchestrator-only поля как обязательный этап
- Одна `PRD-REFACTOR-xxx` за раз

## Инициализация

1. Прочитай `docs/new-agents.md`
2. Прочитай `docs/tasks/prd_refactor_tasks.json`
3. Прочитай `{{PRD_PATH}}` (целиком или разделы из задачи)
4. `git log --oneline -20` в корне репозитория
5. Выбери `pending` с `dependencies = done`, наивысший приоритет

## Цикл задачи

1. Объяви `PRD-REFACTOR-XXX`
2. Переведи в `work in progress`
3. Выполни `acceptance_criteria`:
   - перенеси `sections_to_merge` в `target_section`;
   - сократи §3.5 до отсылок;
   - обнови перекрёстные ссылки и оглавление;
   - при расхождении с кодом — канон = реализация (если не помечено как roadmap).
4. Выполни все `test_steps`
5. `done` только после проверок
6. Запись в `docs/tasks/progress.md`
7. Отдельный commit (`PRD-REFACTOR-xxx: …`)

## Gotchas в new-agents

Если обнаружена recurring ловушка — **одна строка** в `docs/new-agents.md` по admission checklist; не дублируй README.

## Closure очереди

Когда все `PRD-REFACTOR-*` = `done`:

- архивируй `prd_refactor_tasks.json` в `docs/tasks/done/`;
- строка в `progress.md`;
- closure commit.

## Orchestrator

Для K>1 используй `.cursor/commands/run_ralph_orchestrator.md` с очередью `docs/tasks/prd_refactor_tasks.json` и skills orchestrator-*.
