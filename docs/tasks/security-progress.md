# Security audit progress

Журнал результатов security-циклов (`run_ralph_security`).

## Шаблон записи

```markdown
## [SEC-NNN] <title>
**Дата:** YYYY-MM-DD
**Vuln ref:** <vuln_ref из задачи>
**Статус:** exploitable | not_exploitable | partially | config_dependent
**Вердикт:** Кратко (1–2 предложения)

**Что проверено:**
- Файлы: ...
- Сценарий: ...

**Результат:**
- <Конкретные находки>

**PoC:** `tests/security/sec_NNN_exploit.py`
**Автотест:** `tests/security/test_sec_NNN.py` | нет (причина)

**Рекомендации по исправлению:**
- ...

**Предложения на следующий цикл:**
- ...

---
```

---
