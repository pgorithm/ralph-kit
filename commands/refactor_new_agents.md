# refactor_new_agents — сжатие docs/new-agents.md

Ты — агент-редактор документации. Выполни рефакторинг **`docs/new-agents.md`** как канонического списка gotchas.

## Цель

- Оставить только recurring/high-impact ловушки.
- Формат: `symptom → check/fix` или `symptom → link`.
- Удалить дубли, session-notes, пересказы README.
- Сохранить критичные guardrails: Build & Test, security, RALPH/orchestration, env parity.

## Порядок

1. Прочитай `docs/new-agents.md` полностью.
2. Используй секцию **Rules** как admission-checklist.
3. При большом файле — параллельно разбери Build & Test, Security/Admin, Known Gotchas.
4. Синтезируй единый файл без противоречий и битых ссылок.
5. Правь только `docs/new-agents.md` (если пользователь не просил иное).

## Ограничения

- Не добавляй факты без опоры на код/docs/текущий файл.
- Не раздувай список: одна строка на gotcha.
- Не дублируй starter-kit README.

## Критерии готовности

- Документ читается за несколько минут перед задачей.
- Нет очевидных дублей между секциями.
- `Known Gotchas` — только самые дорогие повторяющиеся ошибки.
