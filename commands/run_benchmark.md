# run_benchmark — performance / load investigation

Ты — исполняющий агент. Задача: **воспроизводимо** измерить hot paths, найти узкие места и выдать **действенные рекомендации** с артефактами в репозитории.

> После `adapt_project_cursor` подставь реальные команды, пути и пороги из `docs/project/ai-assisted-config.md` и профильных docs проекта.

## Цели

1. Зафиксировать latency (p50/p95/p99), RPS где уместно.
2. Отделить узкое место: БД, сервис, HTTP, внешние API.
3. Предложить улучшения (индексы, кэш, пулы, конфиг).
4. Оценить запас до деградации под нагрузкой.

## Куда класть результаты

| Что | Куда |
|-----|------|
| Сырые JSON, manifest, EXPLAIN, дампы | `docs/benchmarks/runs/<run-id>/` |
| Имя run-id | UTC `YYYYMMDDTHHMMSSZ` или осмысленный суффикс |
| Нарративный отчёт | `docs/benchmarks/YYYY-MM-DD-<topic>.md` со ссылкой на `runs/.../` |

Не смешивай прогоны со служебными каталогами (`_templates`, `_last_manual`).

## Предусловия (адаптируй под проект)

1. Тестовая/стресс БД поднята и схема актуальна (`{{MIGRATE_CMD}}` на **той же** БД, что в benchmark URL).
2. API/сервис доступен по `{{BENCHMARK_BASE_URL}}` (если HTTP-нога нужна).
3. Есть baseline JSON с порогами (создай после первого представительного прогона).
4. Секреты только из `.env`, не в отчётах.

## Команды (плейсхолдеры)

Замени на скрипты проекта после adapt:

```bash
{{BENCHMARK_RUN_CMD}}
{{BENCHMARK_CHECK_CMD}}
```

Примеры для разных стеков: `npm run bench`, `go test -bench`, `k6 run scripts/load.js`, `py scripts/run_perf_smoke.py run`.

## Процесс агента

1. Прочитай `docs/new-agents.md` (Build & Test).
2. Уточни scope (какие endpoints/запросы/фоновые job).
3. Запусти `{{BENCHMARK_RUN_CMD}}`; сохрани артефакты в `docs/benchmarks/runs/<run-id>/`.
4. Сравни с baseline (`{{BENCHMARK_CHECK_CMD}}` или ручные пороги).
5. Напиши отчёт `.md`: контекст, метрики, bottlenecks, рекомендации, next steps.
6. Не коммить секреты; не логировать пароли.

## Windows

- Цепочка команд: `;` вместо `&&` в PowerShell 5.x.
- Интерпретатор: `py -m ...` или путь к venv из `ai-assisted-config.md`.

## Связь с RALPH

Benchmark обычно **не** входит в `tasks.json` автоматически. При необходимости добавь TASK вручную или через `generate_tasks` с AC «отчёт в docs/benchmarks/».
