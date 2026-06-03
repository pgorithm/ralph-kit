# Product versioning — {{PROJECT_NAME}}

Канон semver для closure батча RALPH. Адаптируй путь к version-файлу в `docs/project/ai-assisted-config.md` (`VERSION_FILE`).

## Источник версии

- Один файл версии в репозитории (например `package.json`, `pyproject.toml`, `version.txt`).
- **Не bump'ить** версию, пока в `docs/tasks/tasks.json` есть задачи не в статусе `done`.

## release.kind (на батч)

| kind | Когда | Bump |
|------|--------|------|
| `patch` | багфиксы, security, infra/docs без заметных user-facing фич | Z+1 |
| `minor` | новые user-facing возможности | Y+1, Z→0 |
| `major` | breaking API/контракты, смена monetization model (явный критерий в PRD) | X+1, Y/Z→0 |

Смешанный батч → побеждает **старшая** цифра (`minor` + security fixes → `minor`).

## Поля в tasks.json

```json
"release": { "kind": "patch", "baseline_version": "0.1.0", "notes": "optional scope hint" }
```

`release.kind` — подсказка для closure; финальную классификацию подтверждает closure-агент по содержимому батча.

## Closure (после всех `done`)

1. Определить `release_kind`
2. Вычислить `vX.Y.Z` от текущей версии
3. Обновить `VERSION_FILE`
4. Release note (если принято в проекте): `docs/releaseNotes/YYYY-MM-DD.md`
5. Строка в `docs/tasks/progress.md`: `Release vX.Y.Z: …` + диапазон TASK-ID
6. Архив `tasks.json` → `docs/tasks/done/`
7. Closure commit; git tag — только по явному запросу maintainer

Подробности: `.cursor/commands/run_ralph.md`, `.cursor/commands/run_ralph_orchestrator.md`.
