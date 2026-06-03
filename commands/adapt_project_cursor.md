# adapt_project_cursor — адаптация starter-kit под конкретный проект

Ты — инициализирующий агент. Пользователь уже создал (или существенно заполнил) **PRD**. Твоя задача — **подстроить** скопированный [ralph-kit](https://github.com/pgorithm/ralph-kit) под этот репозиторий: заменить плейсхолдеры, обновить команды и rules, дополнить `docs/new-agents.md` стартовыми секциями Build & Test и Admin.

## Когда вызывать

- После `init_prd` / первого `docs/PRD.md`.
- После смены стека (другой язык, другой test runner, другой путь к PRD).
- При форке starter-kit в новый репозиторий.

## Входные данные (прочитать обязательно)

1. **`docs/PRD.md`** (или путь из репозитория) — название продукта, платформа, стек, админка, интеграции.
2. **Корень репозитория** — `pyproject.toml` / `package.json` / `go.mod`, `docker-compose.yml`, CI workflow, README.
3. **Существующие `.cursor/`** — не затирай уникальные commands/skills проекта; **мержи** или спроси, если конфликт имён.

## Выход (что должно получиться)

### 1. Файл конфигурации проекта

Создай или обнови **`docs/project/ai-assisted-config.md`** (канон для человека и агентов):

```markdown
# AI-assisted project config

| Key | Value |
|-----|-------|
| PROJECT_NAME | … |
| PRD_PATH | docs/PRD.md |
| CURRENT_WORK_SECTION | §3.5 |
| LINT_CMD | … |
| TEST_FULL_CMD | … |
| TEST_TWO_PHASE_CMD | … or «не используется» |
| MIGRATE_CMD | … or «не используется» |
| ADMIN_BASE_URL | … or «нет админки» |
| PRIMARY_LANGUAGE | … |
| PACKAGE_MANAGER | … |
| TASK_ID_PREFIX | TASK |
| PROGRESS_PATH | docs/tasks/progress.md |
| TASKS_QUEUE_PATH | docs/tasks/tasks.json |
| VULN_CATALOG_PATH | docs/security/vulnerability-categories.md |
| SECURITY_TASKS_PATH | docs/tasks/security-tasks.json |
| TEST_CMD | pytest tests/security/ -v (или эквивалент) |
| VERSION_FILE | package.json / pyproject.toml / … |
| BENCHMARK_RUN_CMD | … or «не используется» |
| BENCHMARK_CHECK_CMD | … or «не используется» |
| BENCHMARK_BASE_URL | … or «не используется» |
```

Значения выведи из PRD и репозитория, не оставляй `{{...}}`.

### 2. Замена плейсхолдеров

Пройди по файлам starter-kit и замени все `{{...}}` (в т.ч. `VERSION_FILE`, `BENCHMARK_*`) в:

- `docs/new-agents.md`
- `docs/RALPH-ORCHESTRATION-QUICKSTART.md` — только если копируете из reference (в kit не входит; канон — `run_ralph_orchestrator.md`)
- `docs/tasks/progress.md`
- `docs/tasks/tasks.template.json` (в `agent_instructions`)
- `.cursor/rules/new-agents-gotchas.mdc`
- `.cursor/commands/*.md` с плейсхолдерами (run_ralph*, generate_tasks*, generate_tasks_security, run_ralph_security, run_benchmark, …)
- `docs/tasks/security-tasks.template.json` (пути `{{VULN_CATALOG_PATH}}`, `{{SECURITY_TASKS_PATH}}`, `{{PRD_PATH}}` в meta)
- `.cursor/skills/**/SKILL.md` с `{{LINT_CMD}}` и т.п.

### 3. Обновление `docs/new-agents.md`

Добавь **проектные** bullets (не дублируя README):

- как поднять проект локально (1–3 строки + ссылка на README/GettingStarted);
- команда миграций / install;
- как запустить админку (если есть);
- 2–5 **специфичных** рисков из PRD (auth, payments, external API, rate limits).

Не заполняй Known Gotchas выдуманными пунктами — только из PRD/README/очевидных инвариантов.

### 4. Адаптация commands

| Команда | Действие |
|---------|----------|
| `generate_tasks.md` | Пути к PRD, currentProblems, progress; формат TASK-ID |
| `update_prd.md` | `{{PRD_PATH}}`, `{{CURRENT_WORK_SECTION}}` |
| `run_ralph.md` / `run_ralph_orchestrator.md` | Точные lint/test команды; наличие/отсутствие two-phase |
| `generate_tasks_security.md` / `run_ralph_security.md` | Security-поток, пути к каталогу уязвимостей и progress |
| `generate_tasks_prdrefactor.md` / `run_ralph_prdrefactor.md` | PRD refactor queue |
| `run_benchmark.md` | Команды и пути `docs/benchmarks/` |
| `ralph.md` | Без изменений, если пути стандартные |

### 5. Hooks (опционально)

- Реализуй или скопируй `scripts/sync_env_parity.py`
- Активируй `.cursor/hooks.json` из `hooks.json.example`
- Добавь в `docs/new-agents.md` одну строку про env parity, если hook включён

### 6. Rules

- Обнови ссылки на `docs/new-agents.md` и имя проекта в `new-agents-gotchas.mdc`.
- `project-wide-orchestration.mdc` — оставь.

### 7. Skills

- Стартовые stubs уже в `.cursor/skills/` — обнови плейсхолдеры команд.
- Переименуй `senior-engineer` → `senior-python-engineer` (и т.д.), если нужна языковая специфика.
- Доменные skills (legal, marketing) — предложи копирование из reference-репозитория, не копируй без запроса.

## Правила

- **Не генерируй** полный PRD заново — только адаптация инфраструктуры агентов.
- **Не удаляй** пользовательские доработки в `.cursor/` без явного конфликта.
- **Не коммить** секреты; в примерах — placeholders.
- После работы выведи краткий отчёт: какие файлы изменены, итоговая таблица из `ai-assisted-config.md`, следующий шаг (`generate_tasks` или `init_tasks`).

## Критерии готовности

- Нигде в адаптированных файлах не осталось `{{PLACEHOLDER}}`.
- `docs/new-agents.md` читается без отсылок к чужому продукту, кроме optional «based on starter-kit».
- Команды `run_ralph` и `generate_tasks` ссылаются на реальные пути и команды тестов этого репозитория.
- Пользователь может выполнить `generate_tasks` сразу после adapt.

## Пример вывода в чат

```text
Adapt complete.
- Config: docs/project/ai-assisted-config.md
- Updated: 12 files
- LINT: {{LINT_CMD}}
```
