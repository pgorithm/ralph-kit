# init_tasks — первая очередь задач из PRD (greenfield)

Ты — инициализирующий агент. Преобразуй PRD в первую рабочую очередь `docs/tasks/tasks.json` для инкрементального исполнения агентами (solo или orchestrator).

## Входные данные

1. **`{{PRD_PATH}}`** (обычно `docs/PRD.md`)
2. `docs/new-agents.md`
3. `docs/tasks/progress.md` (если есть — для продолжения нумерации)

## Выход

- **`docs/tasks/tasks.json`** — строго по `docs/tasks/tasks.template.json`
- Скопируй блок **`agent_instructions`** из шаблона **целиком** (подставь `{{LINT_CMD}}`, `{{TEST_FULL_CMD}}`, `{{TEST_TWO_PHASE_CMD}}` после `adapt_project_cursor`)
- Каждая задача — все поля из шаблона (`assignee`, `claimed_at`, `lease_until`, `review_required`, `status_reason`, `artifacts`, `test_verdict`, …)
- Пустой **`docs/tasks/progress.md`**, если отсутствует
- Ориентир **15–50** атомарных TASK

## Нумерация

- Пустой progress → `TASK-001`
- Иначе → продолжай после max TASK в progress

## Декомпозиция

- Одна задача ≈ одна сессия агента (~30 мин)
- Минимум зависимостей; критический путь первым
- Проверяемые `acceptance_criteria` и практичные `test_steps`
- Приоритет: infrastructure/security → core → integrations → UI

### Задачи без изменения кода

Явно **«без изменений кода»**; gate — `{{LINT_CMD}}` + `test_steps`, без полного test runner.

### Пояснение: full tests vs orchestrator

Воркер — только узкие тесты. Полный охват `tests/` для `test_verdict`/`done` в parallel mode — **Test Coordinator** (`{{TEST_TWO_PHASE_CMD}}` preferred, `{{TEST_FULL_CMD}}` fallback). Non-code → `lint_only`.

## Обязательные категории в первой очереди

- infrastructure (bootstrap, deps, config)
- functional (core business)
- integration (API, external systems)
- ui (или эквивалент пользовательского контура)
- security (auth, validation, secrets handling)

## Статусы

`pending -> work in progress -> needs_review -> done` (orchestrator) или solo `pending -> work in progress -> done` без обязательного `needs_review`.

## После генерации

Сообщи: число задач, диапазон ID, рекомендацию `run_ralph` (первые 3–5) vs `run_ralph_orchestrator` (рост батча).
