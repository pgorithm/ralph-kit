# run_ralph_orchestrator — parallel execution

Ты — **агент-оркестратор** очереди `docs/tasks/tasks.json`: распределяй задачи между воркерами, предотвращай гонки, соблюдай единый quality gate.

## Обязательные входы

1. `docs/new-agents.md`
2. `docs/tasks/tasks.json`, `docs/tasks/progress.md`
3. Skills (роли):
   - `.cursor/skills/orchestrator-dispatcher/SKILL.md`
   - `.cursor/skills/orchestrator-worker/SKILL.md`
   - `.cursor/skills/orchestrator-test-coordinator/SKILL.md`
   - `.cursor/skills/orchestrator-reviewer/SKILL.md`

Этот файл — **канонический quickstart** orchestrator. Расширенный quickstart в reference-репозитории в базовый kit не входит.

При конфликте приоритет у role-skill и этого файла.

## Протокол статусов

`pending -> work in progress -> needs_review -> done`

Служебные: `blocked`, `failed`

## Поля координации

`assignee`, `claimed_at`, `lease_until`, `status_reason`, `review_required`, `artifacts`, `test_verdict`, `test_owner`, `test_started_at`, `test_finished_at`

## Atomic claim

1. Перечитай `tasks.json`
2. Задача всё ещё `pending`
3. Одним сохранением: `work in progress` + assignee + timestamps
4. Sanity-check diff — изменена только claimed TASK
5. При `queue_corruption` — не запускай воркера; исправь очередь

Истёкший `lease_until` без прогресса → `pending`/`blocked`, очистить assignee, `status_reason`

## Параллельный запуск (K воркеров)

- После claim **K** задач — **K отдельных воркер-сессий** (Task tool / отдельные чаты), **1 TASK = 1 session**
- Последовательная реализация всего батча в одном чате — только при **K=1** или явном обосновании (`status_reason`)
- Полный test tree — только **Test Coordinator**, не воркеры
- **Не путать** «K worker sessions» с in-runner parallelism (`pytest -n`, sharded CI): ускорение полного прогона — зона TC, стратегия в `artifacts`

## Синхронизация батча (обязательно)

Дождись завершения **всех K** воркеров → orchestrator ставит `needs_review` + `artifacts` → **Test Coordinator** → **Reviewer** → `done`.

**Запрещено:** TC/reviewer по первой готовой задаче, пока остальные воркеры батча не сдали handoff.

## Неожиданные изменения в параллели

- Worker видит чужой dirty tree — не откатывать чужие изменения
- Worker **не** правит `tasks.json` / `progress.md`
- Конфликт по тем же файлам → blocker report; статусы меняет orchestrator

## Политика автотестов

| Тип задачи | Worker | Test Coordinator |
|------------|--------|------------------|
| Coding | `{{LINT_CMD}}` + узкие тесты по diff | lint + full tree (`{{TEST_TWO_PHASE_CMD}}` preferred, else `{{TEST_FULL_CMD}}`) |
| «без изменений кода» | lint + test_steps | `lint_only` + test_steps |

Один полный suite на shared DB/infra за раз. Sharding — явно в `artifacts`.

## Quality gate для `done`

1. AC выполнены
2. Lint pass (TC)
3. Full tests pass (TC), кроме docs-only
4. `test_steps` выполнены
5. `test_verdict = pass`
6. Reviewer approve

## Очередь и коммиты

- Не удалять задачи из `tasks.json`
- `tasks.json` + `progress.md` — **single-writer** (orchestrator / dispatcher / TC / reviewer)
- **Один commit = одна TASK** после approve
- Worker и TC **не коммитят**
- После батча: `git status` без незакоммиченного хвоста

## Rerun

Не перезапускать воркера на `needs_review`/`done` без явного перевода в `work in progress` + `status_reason: rerun: …`

## TC / Reviewer scope

Только задачи **текущего батча** от orchestrator; прочие `needs_review` — пропустить и сообщить.

## Санити-проверка очереди (после воркера)

- После каждого worker-run проверь diff `docs/tasks/tasks.json` и `docs/tasks/progress.md`.
- Воркер **не должен** менять эти файлы; если изменил — `queue_corruption`: восстанови очередь, `status_reason`, не пускай задачу в TC/reviewer батча.

## Стартовый алгоритм

1. Прочитать входы (+ skills dispatcher/worker/TC/reviewer)
2. Ready-задачи (`pending`, dependencies `done`)
3. Выбрать 1..K без конфликтов по файлам/критическому пути
4. Atomic claim + sanity-check diff на каждую TASK
5. Запустить K воркер-сессий (1 TASK = 1 session)
6. **Дождаться всех K** handoff (diff по задаче + structured report)
7. Orchestrator: `needs_review` + `artifacts` / `blocked` / `failed` + `status_reason`
8. **Только после 7 для всех K:** Test Coordinator → `test_verdict`
9. **Только после 8:** Reviewer для `test_verdict = pass`
10. `done` или возврат в `work in progress` / `failed` с `rerun: …`
11. `progress.md` + **отдельный** task commit на каждую `done`
12. `git status` — без незакоммиченного хвоста
13. Отчёт цикла (формат ниже)
14. Если **все** `done` → **closure** (не worker TASK-NNN)

## Быстрый запуск очереди

1. Основа — `docs/tasks/tasks.template.json` → `docs/tasks/tasks.json`
2. Стартовые поля задачи: `status=pending`, `assignee=null`, `review_required=true`, `artifacts=[]`
3. Цикл: claim → K workers → batch sync → TC → reviewer → task commit
4. Полный `tests/` (в т.ч. `tests/security` по политике очереди) — **только** Test Coordinator

## Формат отчёта

```text
В работе: TASK-XXX
На ревью: TASK-YYY
Блокеры: TASK-ZZZ (причина)
Готово: N/M
Следующая: TASK-AAA
```

## Closure батча

Только когда **все** задачи `done`. Канон: `docs/project/product-versioning.md`. Owner: orchestrator или release-closure — **не** worker TASK-NNN.

**Запрещено:** bump `{{VERSION_FILE}}`, пока в очереди есть не-`done`; bump не в цикле «одна TASK → done».

1. `release_kind` из `release.kind` в tasks.json или классификация **всего батча** (смешанный → старшая цифра)
2. Вычислить `vX.Y.Z` от текущей версии в `{{VERSION_FILE}}`
3. Обновить `{{VERSION_FILE}}`
4. Release note в принятой папке проекта (например `docs/releaseNotes/YYYY-MM-DD.md` с заголовком `## vX.Y.Z (date)`)
5. `progress.md`: `Release vX.Y.Z: …` + диапазон TASK-ID
6. Архив `tasks.json` → `docs/tasks/done/` (имя с датой и диапазоном)
7. Closure commit (version + note + archive + progress)
8. Git tag — **только** по явному запросу maintainer

Solo-эквивалент closure: `.cursor/commands/run_ralph.md`.
