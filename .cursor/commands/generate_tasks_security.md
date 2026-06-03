# generate_tasks_security — генерация очереди white-hat аудита

Ты — инициализирующий агент для цикла проверки безопасности **{{PROJECT_NAME}}**. Преобразуй каталог уязвимостей и накопленный прогресс в структурированный список задач для red-team исполнения (без исправления production-кода в этом цикле).

Режимы генерации:

- **Cycle 1 (первичная инициализация):** широкий охват `{{VULN_CATALOG_PATH}}`, первичные PoC по категориям.
- **Cycle 2+ (повторный аудит):** приоритет ретестов ранее найденных проблем и валидации качества security-автотестов; циклы **чередуют** профиль: regression-heavy и discovery-heavy (см. `docs/new-agents.md` — majority regression-focused для cycle ≥ 2).

## Контекст процесса

1. Security-цикл **ищет и документирует** уязвимости (PoC + регрессионные тесты, которые **FAIL**, пока дыра открыта).
2. Подтверждённые находки попадают в `docs/tasks/currentProblems.md` (шаблон записи — в `.cursor/commands/run_ralph_security.md`).
3. Исправления — **отдельный** продуктовый батч: `update_prd` → `generate_tasks` (см. `.cursor/commands/generate_tasks.md`) → `run_ralph` / `run_ralph_orchestrator`.
4. Исполнение security-очереди — `.cursor/commands/run_ralph_security.md`.

## Входные данные

1. **`{{VULN_CATALOG_PATH}}`** — каталог уязвимостей/векторов с приоритетами (в kit по умолчанию: `docs/security/vulnerability-categories.md`). **Источник строк для генерации задач** — этот каталог, не PRD целиком.
2. **`docs/tasks/currentProblems.md`** — список текущих проблем; исполняющий агент дописывает в **конец** выявленные уязвимости (`exploitable` / `partially`). Исправление — не в аудите.
3. **`docs/tasks/progress.md`** — журнал продуктовых задач (в т.ч. заявленные исправления security-находок). Используй как дополнительный контекст к security-progress:
   - выдели задачи, где заявлено закрытие SEC-* или security-фикс (по заголовкам/описаниям TASK-блоков);
   - трактуй их как **обязательные кандидаты на повторную проверку фикса** в новом цикле;
   - из текстов выполнения извлекай условия для ретеста (конфиги, middleware, миграции, compose/env, feature flags и т.п.).
4. **`docs/tasks/security-progress.md`** — лог предыдущих циклов (шаблон записи: `docs/tasks/security-progress.md` в kit). Если файл есть:
   - определи последний использованный SEC-ID (например SEC-012);
   - прочитай блок **«Предложения на следующий цикл»** (или эквивалент) — на его основе формулируй **углублённые** задачи;
   - по умолчанию не создавай новую задачу для пункта, уже покрытого записью со статусом `verified_fixed` / устойчивым `not_exploitable` с автотестом — **кроме** задач повторной проверки (см. «Повторные циклы»).
5. **`docs/new-agents.md`** — правила проекта, в т.ч.:
   - **Security tasks re-init:** при перегенерации `{{SECURITY_TASKS_PATH}}` учитывай **и** `security-progress.md`, **и** `progress.md` — сначала fix-validation, потом новые deep-check векторы.
   - **Security repeat cycles:** для cycle ≥ 2 очередь в основном regression-focused (retest fixes + test quality + PoC hardening); новые векторы — после обязательного слоя.
6. **`{{PRD_PATH}}`** (опционально, skim) — согласование смысла `vuln_ref`, приоритетов и формулировок с продуктовым бэклогом security-требований. **Не подменяет** шаг 1. Если в проекте есть отдельное security-приложение к PRD — используй его; иначе — релевантные разделы `{{PRD_PATH}}`.

## Выходной файл

- **Путь:** `{{SECURITY_TASKS_PATH}}` (создай или перезапиши; в kit: `docs/tasks/security-tasks.json`).
- **Каркас:** `docs/tasks/security-tasks.template.json` — сохрани структуру `meta` / `agent_instructions` / полей задачи; дополни значениями из разделов ниже (не удаляй обязательные ключи шаблона).
- Содержимое: только **новые** задачи текущего цикла; ID продолжают нумерацию из `security-progress.md` (SEC-001, SEC-002, …).
- Для `cycle >= 2` список **risk-based**, не «полный повтор каталога».

## Формат задач — строго JSON

```json
{
  "meta": {
    "cycle": 1,
    "generation_mode": "initial|repeat",
    "repeat_focus": "fix_validation|test_quality|poc_hardening|new_vectors",
    "generated_at": "ISO-дата",
    "source": "{{VULN_CATALOG_PATH}}",
    "prd_alignment": "{{PRD_PATH}}",
    "delivery_progress": "docs/tasks/progress.md",
    "previous_progress": "docs/tasks/security-progress.md"
  },
  "agent_instructions": {
    "before_start": [
      "Прочитай docs/new-agents.md (правила {{PROJECT_NAME}})",
      "При необходимости сверки security-требований: {{PRD_PATH}}",
      "Прочитай этот файл и git log --oneline -10",
      "Оркестратор (параллельная очередь): .cursor/commands/run_ralph_orchestrator.md — 1..K ready-задач, atomic claim, K воркер-сессий",
      "Соло-воркер: одна задача pending (или уже в work in progress); наивысший приоритет среди доступных",
      "Все dependencies — status done"
    ],
    "during_work": [
      "Работай только над выбранной SEC-задачей",
      "PoC и автотесты — в tests/security/",
      "НЕ модифицируй production-код приложения (см. docs/new-agents.md / README — API, workers, admin server code)",
      "НЕ выполняй деструктивных действий: DROP TABLE, DELETE без WHERE, rm -rf, перезапись .env реальными секретами",
      "Все скрипты — идемпотентные и безопасны для повторного запуска"
    ],
    "before_finish": [
      "Выполни ВСЕ test_steps из задачи",
      "При exploitable/partially — pytest в tests/security/, FAIL пока уязвимость открыта, PASS после фикса",
      "Обнови {{SECURITY_TASKS_PATH}}: status → done (не удаляй и не редактируй другие задачи)",
      "Запиши результат в docs/tasks/security-progress.md по шаблону kit",
      "При exploitable/partially — допиши блок в конец docs/tasks/currentProblems.md (шаблон: run_ralph_security.md)",
      "ЗАПРЕЩЕНО удалять или редактировать задачи в очереди — только status"
    ]
  },
  "tasks": [
    {
      "id": "SEC-001",
      "task_type": "exploit|retest_fix|test_quality|poc_hardening|new_vector",
      "source_sec_ids": ["SEC-014"],
      "vuln_ref": "1.1",
      "category": "auth|authz|injection|ssrf|files|crypto|http|csrf|rate-limit|data-leak|integration|infra|dependencies|workers|business-logic",
      "priority": "critical|high|medium|low",
      "title": "Краткое название проверки",
      "description": "Что проверяем и какой сценарий атаки моделируем",
      "attack_vector": "Пошаговое описание атаки для воспроизведения",
      "target_files": ["path/to/module.py"],
      "acceptance_criteria": [
        "Изучен код target_files, описан конкретный путь эксплуатации",
        "Создан tests/security/sec_001_exploit.py с PoC",
        "Скрипт запущен, вердикт: exploitable | not_exploitable | partially | config_dependent",
        "Для repeat: запущены старые PoC/pytest из source_sec_ids, подтверждена актуальность",
        "Для repeat: проверено качество автотеста (ловит уязвимость, не ложноположительный, не на хрупких моках)",
        "Для repeat: минимум одна усиленная bypass-попытка (из «Предложений на следующий цикл» или новый близкий вектор)",
        "При exploitable/partially: tests/security/test_sec_001.py",
        "Запись в security-progress.md",
        "При exploitable/partially: запись в конец currentProblems.md"
      ],
      "test_steps": [
        "Шаг 1: изучить target_files, найти точку входа",
        "Шаг 2: написать PoC",
        "Шаг 3: запустить PoC, зафиксировать stdout/stderr",
        "Шаг 4: при подтверждении — автотест",
        "Шаг 5: security-progress.md",
        "Шаг 6: при exploitable/partially — currentProblems.md",
        "Шаг 7: не было изменений в production-коде"
      ],
      "constraints": [
        "Не модифицировать production-код",
        "Не удалять данные из БД",
        "Без реальных внешних API — мокать при необходимости"
      ],
      "dependencies": [],
      "status": "pending"
    }
  ]
}
```

## Принципы декомпозиции

**Cycle 1:** одна строка/подпункт каталога = одна задача (допустима декомпозиция на 2–3 подзадачи для крупных пунктов).

**Cycle 2+:** не покрывай весь каталог заново. Risk-based отбор:

- сначала ретест / регрессия по найденным и **заявленно исправленным** проблемам;
- затем усиление старых PoC и ревизия качества тестов;
- затем новые векторы;
- в **каждом втором** repeat-цикле — discovery-heavy профиль (новые векторы и bypass — приоритет).

**Атомарность:** одна сессия агента (ориентир до 30 минут). Артефакт: PoC + запись в progress.

**Безопасная эксплуатация** (white-hat): скрипты **пытаются пролезть**, но **не ломают** проект:

- тестовые данные, mock-серверы, HTTP-клиент к **локальному** dev-серверу;
- без запросов к **продакшен** внешним API и без реальных секретов;
- без записи/удаления в прод-БД — тестовая БД, транзакции с rollback, изолированные фикстуры;
- для инфраструктуры — анализ конфигов и read-only inspect (Docker/K8s), без изменений кластера.

**Автотесты-регрессии** (при `exploitable` / `partially`):

- воспроизводят атаку программно (TestClient / httpx / моки);
- **FAIL** пока уязвимость открыта → **PASS** после фикса;
- имя: `test_sec_NNN.py`; маркер `@pytest.mark.security` (если проект использует маркеры);
- если автотест невозможен — явно `autotest: false` в progress с причиной.

**Приоритизация:** таблица приоритетов в `{{VULN_CATALOG_PATH}}` (P0→critical, P1→high, P2→medium, P3→low).

## Повторные циклы

Если `docs/tasks/security-progress.md` уже существует:

1. `generation_mode`: нет history → `initial`; иначе → `repeat`.
2. Для `repeat` выбери профиль (чередуй по номеру цикла):
   - **regression-heavy:**
     - не менее **60%** задач — `retest_fix` / `test_quality` / `poc_hardening`;
     - не менее **30%** — `retest_fix` по `verified_fixed`, `mitigated` или фиксам из `progress.md`;
     - `new_vector` — не более **40%** списка.
   - **discovery-heavy:**
     - не менее **50%** — `new_vector`;
     - не менее **30%** — `retest_fix` / `test_quality` / `poc_hardening`;
     - не менее **20%** — `retest_fix` по заявленным фиксам.
3. Для каждого релевантного старого SEC — **связанный пакет**:
   - повторный PoC;
   - повтор pytest security-теста (если есть);
   - усиленный bypass (из «Предложений…» + минимум один новый близкий вектор; для high/critical — **минимум три** независимые bypass-попытки).
4. **Quality-check автотестов:** хрупкий/узкий тест → задача `test_quality`.
5. `exploitable` / `partially` без устойчивого фикса → `poc_hardening` или `retest_fix` (если фикс заявлен в `progress.md`).
6. `not_exploitable` → `new_vector` только при осмысленном новом сценарии (предложения, свежие изменения кода/инфра).
7. Увеличь `cycle` в `meta`.

## Выполни (чеклист генерации)

1. Прочитай **`{{VULN_CATALOG_PATH}}`**.
2. Прочитай **`docs/tasks/progress.md`** — SEC/ security-фиксы, заявленные как выполненные.
3. Прочитай **`docs/tasks/security-progress.md`** (если есть) — последний SEC-ID, статусы, «Предложения на следующий цикл».
4. Прочитай **`docs/new-agents.md`**.
5. При необходимости — **`{{PRD_PATH}}`** (skim по затронутым темам); не подменяет шаг 1.
6. **Repeat-cycle:** сначала пул обязательных регрессий:
   a. ретест из security-progress (`verified_fixed`, `mitigated`);
   b. ретест фиксов из progress.md (recent + high/critical);
   c. ревизия quality security-тестов по тем же пунктам;
   d. усиление PoC (предложения + минимум один новый bypass).
7. Затем — новые направления (не проверявшиеся), с учётом профиля `regression-heavy` / `discovery-heavy`.
8. Для каждой задачи: `attack_vector`, `target_files`, `constraints`; для repeat — `task_type`, `source_sec_ids`.
9. ID SEC-NNN — продолжение нумерации.
10. Приоритеты — каталог + риск регрессии.
11. Для high/critical в `test_steps`: минимум три bypass-попытки; запрет финального `not_exploitable` без полноценного локального окружения (БД, кэш, миграции — по `docs/new-agents.md`) или явная пометка `config_dependent` с условием перепроверки.
12. Сохрани **`{{SECURITY_TASKS_PATH}}`** с заполненным `meta` (`prd_alignment`, `delivery_progress`, `previous_progress`).

## Связка с исполнением и продуктовой очередью

| Шаг | Команда / артефакт |
|-----|-------------------|
| Генерация очереди | этот файл → `{{SECURITY_TASKS_PATH}}` |
| Исполнение аудита | `.cursor/commands/run_ralph_security.md` |
| Очередь исправлений | `update_prd` → `.cursor/commands/generate_tasks.md` → `run_ralph` |
| Шаблон прогресса | `docs/tasks/security-progress.md` |
| Каталог векторов | `{{VULN_CATALOG_PATH}}` |

Продуктовая очередь **`docs/tasks/tasks.json`** и полный прогон тестов (**`{{TEST_CMD}}`**, two-phase при наличии) — вне scope `security-tasks.json`; не дублируй их в security-очереди.

## После генерации (отчёт агенту)

- номер `cycle` и `generation_mode` / `repeat_focus`;
- сколько SEC-задач создано;
- доля `retest_fix` / `test_quality` / `poc_hardening` / `new_vector` / `exploit`;
- следующий шаг: **`run_ralph_security`** (solo) или **`run_ralph_orchestrator`** (parallel SEC-задачи, если настроено).
