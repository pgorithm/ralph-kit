# run_ralph — solo execution

Работа по `docs/tasks/tasks.json` в **solo-режиме**: один агент закрывает задачу end-to-end без orchestrator / TC / reviewer.

## Жёсткий режим SOLO

- Приоритет этого промпта над orchestrator-указаниями в `agent_instructions` внутри `tasks.json`
- Статусы: `pending -> work in progress -> done`
- **Не** использовать `needs_review` как обязательный этап
- Поля `assignee`, `lease_until`, `test_verdict` — не заполнять «для процесса»

## Перед стартом

1. `docs/new-agents.md` (полностью)
2. `docs/tasks/tasks.json`, `docs/tasks/progress.md`
3. `git log --oneline -20` в корне репозитория
4. Ready-задача: `pending`, dependencies = `done`, max priority

## Цикл задачи

1. Объяви TASK-XXX
2. `status` задачи = `work in progress`
3. Реализуй AC (сверяй с `{{PRD_PATH}}`)
4. Проверки **до** `done`:
   - `{{LINT_CMD}}`
   - Если **«без изменений кода»** в description/AC — full tests **не** нужны; lint + `test_steps`
   - Иначе: `{{TEST_TWO_PHASE_CMD}}` preferred, fallback `{{TEST_FULL_CMD}}`
   - Windows: при необходимости `py -m ...` или путь к venv (см. `docs/new-agents.md`)
   - Не помечай `done` и не коммить, пока lint не зелёный и (для coding) полный охват `tests/` не выполнен
5. `test_steps` (ручные шаги — опиши результат)
6. `done` только после зелёных проверок *(исключение: security-тесты, явно отложенные в pending задачах текущей очереди)*
7. Запись в `docs/tasks/progress.md` (1–2 строки)
8. Отдельный commit (`TASK-xxx: …`)

Если задача «просит review» — в solo это твоя проверка шага 4; сразу `done`, без `needs_review`.

## Ошибки

- Не брать вторую задачу до закрытия первой
- Блокер → `docs/tasks/currentProblems.md`, остановиться
- Регрессии в тестах — исправить до перехода дальше

## После задачи

```text
TASK-XXX выполнена (N/M). Следующая: TASK-YYY (…)
```

## Closure (все задачи `done`)

Только когда **все** задачи в `docs/tasks/tasks.json` — `done`. Канон: `docs/project/product-versioning.md`.

1. `release_kind` — из `release.kind` в tasks.json или классификация батча
2. Вычисли `vX.Y.Z`, bump **одну** цифру в `{{VERSION_FILE}}`
3. Release note в `docs/releaseNotes/` (если принято в проекте): `YYYY-MM-DD.md`, заголовок `## vX.Y.Z (YYYY-MM-DD)`
4. `progress.md`: строка `Release vX.Y.Z: …` + диапазон TASK-ID
5. Архив `tasks.json` → `docs/tasks/done/` (имя с датой и диапазоном TASK)
6. Closure commit; `git tag` — **только** по явному запросу maintainer

**Запрещено:** version bump в середине батча или в цикле «одна TASK → done».
