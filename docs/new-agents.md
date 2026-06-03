# Common Context — {{PROJECT_NAME}}

## Purpose

Краткий ориентир для AI-агентов: только **неочевидные** ловушки, которые ломают локальный запуск, тесты, деплой или продуктовые инварианты. Подробности — в README и профильных docs; здесь — admission-фильтр и канонические gotchas.

## Rules

- Before starting a task, read this file **fully**.
- **Who adds entries:** the agent decides per admission checklist below; user approval not required. When unsure — **do not add**.
- **Admission checklist** — append only when **all** are true:
  1. Without this note, another agent would likely repeat a **costly mistake** (failed CI, broken local run, wrong DB, security foot-gun).
  2. The point is **not** already clear from README or linked docs.
  3. The issue is **recurring or high-impact**, not a one-off debug note.
- **Format:** one bullet per gotcha, `symptom → check/fix` or `symptom → link`.
- **Maintenance:** when code/docs change, update or remove stale bullets in the same change when reasonable.
- Do not duplicate README; link instead.

## General Section

- **User-visible copy vs traceability:** Do not put task IDs or internal backlog links in user-facing UI. In code comments, references to PRD/tasks are encouraged.
- **PRD path:** canonical requirements — `{{PRD_PATH}}`.
- **Security tasks re-init:** when regenerating `docs/tasks/security-tasks.json`, use both `docs/tasks/security-progress.md` and `docs/tasks/progress.md` — include explicit fix-validation tasks before new deep-check vectors.
- **Security repeat cycles:** for cycle ≥ 2, keep security queue majority regression-focused (retest fixes + test quality + PoC hardening); add new attack vectors only after that layer.
- **Legislative / compliance gotchas:** jurisdiction-specific legal findings that affect implementation — document in a dedicated subsection (add via `adapt_project_cursor` when the product has regulatory scope).

## Build & Test

<!-- Заполните после adapt_project_cursor -->

- Local run fails after pull → `{{MIGRATE_CMD}}` on the same DB the app/tests use.
- Lint: `{{LINT_CMD}}`
- Full tests: `{{TEST_FULL_CMD}}` (preferred two-phase: `{{TEST_TWO_PHASE_CMD}}` if applicable)
- Shell on Windows: use `;` instead of `&&` in PowerShell 5.x; prefer `py -m ...` if runtime not on PATH.

## Admin / Operations (если есть админка)

- Admin base URL: `{{ADMIN_BASE_URL}}`
- Login issues over HTTP → check cookie secure flags in env (dev often needs insecure cookies on HTTP).

## Security / Secrets

- No API keys or tokens in client-side templates or committed `.env`.
- Public API docs disabled by default in production unless explicitly enabled.
- New public mutating endpoints → explicit rate limit / auth design.
- Security PoC import errors for app package → run as module (`python -m tests.security.sec_NNN_exploit`) or set `PYTHONPATH` to repo root (see README / Build & Test).
- Full pytest red only on known security backlog → for feature work use project-documented ignore/exclude for `tests/security` until fixes land (do not weaken CI without explicit queue policy).

## Known Gotchas

- Solo RALPH must not follow orchestrator-only gates → use `pending → work in progress → done` without mandatory `needs_review`.
- `docs/tasks/tasks.json` may be runtime queue state — do not assume it is always committed.
- Orchestrator runs K workers in **K separate sessions**, not sequentially in one chat when K>1.
- Orchestrator must wait for **all** workers in batch before Test Coordinator / reviewer.
- Worker must not edit `docs/tasks/tasks.json` or `docs/tasks/progress.md` in orchestrator mode.
- `.env` / `.env.example` parity — optional; enable via `adapt_project_cursor` (hook + skill `env-dotenv-parity` из reference, не в базовом kit).
- `docs/tasks/progress.md` encoding corrupted after edit → restore from git or rewrite as UTF-8.
