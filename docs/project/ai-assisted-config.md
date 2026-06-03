# AI-assisted project config — {{PROJECT_NAME}}

> Заполняется командой **`adapt_project_cursor`**. Канон для агентов и команд RALPH.

| Key | Value |
|-----|-------|
| PROJECT_NAME | {{PROJECT_NAME}} |
| PRD_PATH | {{PRD_PATH}} |
| CURRENT_WORK_SECTION | {{CURRENT_WORK_SECTION}} |
| LINT_CMD | {{LINT_CMD}} |
| TEST_FULL_CMD | {{TEST_FULL_CMD}} |
| TEST_TWO_PHASE_CMD | {{TEST_TWO_PHASE_CMD}} |
| MIGRATE_CMD | {{MIGRATE_CMD}} |
| ADMIN_BASE_URL | {{ADMIN_BASE_URL}} |
| PRIMARY_LANGUAGE | {{PRIMARY_LANGUAGE}} |
| PACKAGE_MANAGER | {{PACKAGE_MANAGER}} |
| TASK_ID_PREFIX | TASK |
| PROGRESS_PATH | docs/tasks/progress.md |
| TASKS_QUEUE_PATH | docs/tasks/tasks.json |
| VULN_CATALOG_PATH | docs/security/vulnerability-categories.md |
| SECURITY_TASKS_PATH | docs/tasks/security-tasks.json |
| TEST_CMD | {{TEST_CMD}} |
| BENCHMARK_RUN_CMD | {{BENCHMARK_RUN_CMD}} |
| BENCHMARK_CHECK_CMD | {{BENCHMARK_CHECK_CMD}} |
| BENCHMARK_BASE_URL | {{BENCHMARK_BASE_URL}} |
| VERSION_FILE | {{VERSION_FILE}} |

## Notes

- Replace all `{{...}}` placeholders; none should remain after adapt.
- Domain-specific skills and hooks (env parity, legal, …) — copy from a reference repo when needed; do not expect them in the base kit copy.
