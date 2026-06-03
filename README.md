# ralph-kit

Универсальный шаблон **Cursor rules, commands и skills** для AI-assisted engineering: PRD → очередь задач JSON → исполнение solo или через RALPH-orchestrator (dispatcher, workers, test coordinator, reviewer).

Легко адаптируется под любую другую IDE.

Формулировки обобщены: стек, пути и команды lint/test подставляются командой **`adapt_project_cursor`**.

## Что внутри

```
ralph-kit/
├── README.md, MANIFEST.md
├── .cursor/
│   ├── rules/                 always-apply (new-agents, project-wide orchestration)
│   ├── commands/              PRD, tasks, RALPH, security, PRD-refactor, benchmark
│   └── skills/                оркестрация, декомпозиция …
└── docs/
    ├── new-agents.md
    ├── PRD-stub.md
    ├── project/
    │   ├── ai-assisted-config.md
    │   └── product-versioning.md
    ├── security/
    │   └── vulnerability-categories.md
    └── tasks/
        ├── tasks.template.json
        ├── security-tasks.template.json, prd_refactor_tasks.template.json
        ├── progress.md, security-progress.md, currentProblems.md
        └── (после старта) tasks.json, security-tasks.json, done/
```

Полный перечень файлов и плейсхолдеров — [MANIFEST.md](MANIFEST.md). Краткий цикл orchestrator — [.cursor/commands/run_ralph_orchestrator.md](.cursor/commands/run_ralph_orchestrator.md).

## Быстрый старт

| Шаг | Команда / действие | Результат |
|-----|-------------------|-----------|
| 1 | Скопировать kit в корень репозитория (см. [MANIFEST.md](MANIFEST.md)) | `.cursor/`, `docs/` шаблоны |
| 2 | Выполнить **`/init_prd`** | Будет создан `docs/PRD.md` |
| 3 | Выполнить **`/adapt_project_cursor`** | `docs/project/ai-assisted-config.md`, без `{{...}}`, проектные секции в `new-agents.md` |
| 4 | Выполнить **`/init_tasks`** *или* **`generate_tasks`** | Будет создан батч задач `docs/tasks/tasks.json` |
| 5 | Выполнять **`/**ralph`** → solo **`run_ralph`** или **`run_ralph_orchestrator`** до закрытия батча задач | Закрытие TASK с gates и progress |

### Что делать после

После первичной настройки работа идёт **итерациями**: накопили вход → обновили PRD → сгенерировали очередь → прогнали RALPH → повторили.

#### Основной цикл (фичи, баги, доработки)

| # | Команда / действие | Зачем |
|---|-------------------|--------|
| 1 | Дописать в [docs/tasks/currentProblems.md](docs/tasks/currentProblems.md) баги, идеи, полные ТЗ | Единый вход для всего, что ещё не в PRD |
| 2 | **`/update_prd`** | Перенести пункты из `currentProblems` в `docs/PRD.md` |
| 3 | **`/generate_tasks`** | Собрать новый батч `docs/tasks/tasks.json` из PRD + `currentProblems` |
| 4 | **`/ralph`** → **`run_ralph`** (solo) или **`run_ralph_orchestrator`** (параллель) | Закрыть все TASK в батче (lint, tests, reviewer, `progress.md`, `done/`) |
| 5 | Повторить с шага 1 | Следующая порция работы |

Solo vs orchestrator — см. [ralph.md](.cursor/commands/ralph.md). Версию продукта bump'айте только после **полного** закрытия батча — [product-versioning.md](docs/project/product-versioning.md).

#### Поиск и устранение уязвимостей

| # | Команда / действие | Зачем |
|---|-------------------|--------|
| 1 | Заполнить [vulnerability-categories.md](docs/security/vulnerability-categories.md), самостоятельно или с ИИ | Чеклист категорий для аудита |
| 2 | **`/generate_tasks_security`** | Очередь `docs/tasks/security-tasks.json` (SEC-*) |
| 3 | **`/run_ralph_security`** | Аудит по security-очереди |
| 4 | Подтверждённые находки попадают в **`currentProblems.md`** | Фиксация как продуктовых задач |
| 5 | **`/update_prd`** → **`/generate_tasks`** → **`/run_ralph`** (или orchestrator) | Исправления через product-батч |
| 6 | Повторить с шага 2 | Регулярный security-цикл |

Подробнее — раздел [Security-слой](#security-слой) ниже.

#### Гигиена и оптимизация PRD

Когда PRD разросся:

| # | Команда / действие | Зачем |
|---|-------------------|--------|
| 1 | **`/generate_tasks_prdrefactor`** | Очередь `docs/tasks/prd_refactor_tasks.json` |
| 2 | **`/run_ralph_prdrefactor`** *или* **`/run_ralph_orchestrator`** с PRD-очередью | Закрыть PRD-TASK |
| 3 | **`/update_prd`** / **`/generate_tasks`** → product RALPH | Вернуться к обычному циклу на обновлённом PRD |

#### Прочие потоки

| Цель | Генерация / вход | Исполнение |
|------|------------------|------------|
| Пересмотреть память агентов | — | **`/refactor_new_agents`** (по [new-agents.md](docs/new-agents.md)) |
| Производительность | TASK вручную или **`/generate_tasks`** | **`/run_benchmark`** |

Команда-роутер всех режимов и путей очередей — **`/ralph`**.

## Команды Cursor

### Жизненный цикл продукта

| Команда | Когда |
|---------|--------|
| `init_prd` | Нет PRD, greenfield |
| `adapt_project_cursor` | После PRD или смена стека / test runner |
| `init_tasks` | Первая очередь из PRD (без полного `generate_tasks`) |
| `update_prd` | Добавить в PRD доработки из `currentproblems.md` |
| `generate_tasks` | Новый батч product-TASK из PRD + `currentProblems` |
| `ralph` | Выбор режима исполнения |
| `run_ralph` | Одна сессия, полный контур качества |
| `run_ralph_orchestrator` | Параллель: claim → K workers → TC → reviewer |

### Специализированные очереди

| Поток | Генерация | Исполнение |
|-------|-----------|------------|
| Security | `generate_tasks_security` | `run_ralph_security` → fixes через product batch |
| PRD hygiene | `generate_tasks_prdrefactor` | `run_ralph_prdrefactor` или orchestrator с PRD-очередью |
| Gotchas | — | `refactor_new_agents` |
| Performance | TASK вручную или `generate_tasks` | `run_benchmark` |

Роутер режимов и путей очередей — **`ralph`** (`.cursor/commands/ralph.md`).

## Security-слой

1. Заполните [docs/security/vulnerability-categories.md](docs/security/vulnerability-categories.md).
2. **`generate_tasks_security`** → `docs/tasks/security-tasks.json`.
3. **`run_ralph_security`** (аудит SEC-*).
4. Подтверждённые находки → [docs/tasks/currentProblems.md](docs/tasks/currentProblems.md).
5. Исправления: **`update_prd`** → **`generate_tasks`** → product RALPH.

## Skills (в kit)

| Skill | Назначение |
|-------|------------|
| `orchestrator-dispatcher` | Ready tasks, atomic claim, lease |
| `orchestrator-worker` | Одна claimed TASK |
| `orchestrator-test-coordinator` | Lint + full tests на батч |
| `orchestrator-reviewer` | AC, `test_verdict`, done / return |
| `project-wide-orchestrator` | Сквозной обзор репозитория |
| `task-decomposition` | Ad-hoc очередь без `generate_tasks` |

## Принципы

| Принцип | Канон |
|---------|--------|
| Требования | `{{PRD_PATH}}` → обычно `docs/PRD.md` |
| Выполненное | `docs/tasks/progress.md` |
| Память агента | `docs/new-agents.md` |
| Очередь задач | `docs/tasks/tasks.json` |
| Quality gate | AC + lint + tests + reviewer → `done` |
| Версия продукта | Bump только после **всего** батча — [product-versioning.md](docs/project/product-versioning.md) |

Конфигурация после adapt — [docs/project/ai-assisted-config.md](docs/project/ai-assisted-config.md).

## Копирование в новый репозиторий

1. Скопируйте дерево kit (см. [MANIFEST.md](MANIFEST.md)); при конфликте имён в `.cursor/` — **мерж**, не затирайте уникальные commands/skills проекта.
2. Выполните шаги быстрого старта выше.
3. Держите один эталонный репозиторий (reference) для периодического сравнения доменных skills и hooks; в kit — минимальный универсальный набор.

## Плейсхолдеры

До **`adapt_project_cursor`** в шаблонах остаются `{{PROJECT_NAME}}`, `{{PRD_PATH}}`, `{{LINT_CMD}}`, `{{TEST_FULL_CMD}}`, … Полный список ключей — таблица в `ai-assisted-config.md` и [MANIFEST.md](MANIFEST.md).

## Автор и контакты

| | |
|---|---|
| **Автор** | Евгений Лучковский ([@pangorithm](https://t.me/pangorithm)) |
| **Репозиторий** | [github.com/pgorithm/ralph-kit](https://github.com/pgorithm/ralph-kit) |
| **Telegram** | [@pangorithm](https://t.me/pangorithm) |
