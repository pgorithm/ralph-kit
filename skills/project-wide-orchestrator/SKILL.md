---
name: project-wide-orchestrator
description: Orchestrates repo-wide investigation by dispatching bounded parallel subagents, synthesizing findings into one report, and avoiding over-dispatch. Use for whole-repo code review, security review, architecture inventory, or full documentation planning.
---

# Project-Wide Orchestrator

## Purpose

Broad repository coverage with synthesis across subsystems. **Not** for local 1–3 file questions.

Triggered by `.cursor/rules/project-wide-orchestration.mdc`.

## Start Gate

**Global** if user asks for whole repo / full review, or answer needs most major areas.

**Local** otherwise — do not use this flow.

## Dispatch Budget

- **2–3 workers:** small–medium global review.
- **4–6 workers:** default for large repos.
- **7–8 max:** only for explicit deep audits.

Hard limits: max **8** workers; no nested orchestration beyond `orchestrator → worker` or `orchestrator → worker → specialist` (one level).

## Domain Partitioning

Non-overlapping slices (adapt paths to repo layout):

1. API/routes and contracts
2. Services / business logic
3. Data layer / migrations
4. Security / auth / secrets
5. Integrations / workers / jobs
6. Tests / CI / docs (merge with another slice if budget low)

## Worker Prompt Contract

Each worker gets:

1. Exact scope (dirs/files).
2. Question for that scope.
3. Evidence: paths/symbols; facts vs assumptions.
4. Output: findings, risks, unknowns, confidence.
5. **No edits** unless explicitly requested.

## Escalation

At most **one** specialist subagent per worker for a deep subdomain; worker merges summary to parent.

## Synthesis

After workers finish:

1. Merge into one map.
2. Deduplicate.
3. Resolve contradictions.
4. List unknowns + missing evidence.
5. Final report: coverage map, key findings (by severity), risks/gaps, recommended next actions.

## Stop

Stop when all slices covered or more workers would only duplicate. When unsure — synthesize now.
