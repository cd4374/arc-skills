---
name: arc-00-01-research-pipeline
description: Pure skills orchestration for AutoResearchClaw 23-stage pipeline. Use when user asks to run end-to-end research in skills mode with arc-xx-xx stage chain.
metadata:
  category: orchestrator
  trigger-keywords: "arc,pipeline,skills-only,research workflow,run research"
  applicable-stages: "1-23"
  priority: "1"
  version: "1.1"
  author: researchclaw
---

## Purpose
Run the complete ARC skills pipeline by chaining `arc-01-01` through `arc-08-04` with canonical gate, rollback, retry, and decision-loop behavior.

## Inputs
- `topic` (required)
- `run_id` (optional; default `rc-YYYYMMDD-HHMMSS`)
- `from` (optional start skill, e.g. `arc-04-02-code-generation`)
- `auto_approve_gates` (optional bool, default false)
- `skip_noncritical` (optional bool, default true)

## Run-root state files (canonical paths)
Write/update under `artifacts/<run_id>/`:
- `pipeline_state.json` (current stage/skill/status)
- `checkpoint.json` (last completed stage)
- `stage_history.jsonl` (append-only events)
- `decision_history.json` (pivot/refine attempts)
- `heartbeat.json` (last done stage heartbeat)
- `pipeline_summary.json` (final summary)

## Stage chain
1. `arc-01-01-topic-init`
2. `arc-01-02-problem-decompose`
3. `arc-02-01-search-strategy`
4. `arc-02-02-literature-collect`
5. `arc-02-03-literature-screen` **[GATE]**
6. `arc-02-04-knowledge-extract`
7. `arc-03-01-synthesis`
8. `arc-03-02-hypothesis-gen`
9. `arc-04-01-experiment-design` **[GATE]**
10. `arc-04-02-code-generation`
11. `arc-04-03-resource-planning`
12. `arc-05-01-experiment-run`
13. `arc-05-02-iterative-refine`
14. `arc-06-01-result-analysis`
15. `arc-06-02-research-decision`
16. `arc-07-01-paper-outline`
17. `arc-07-02-paper-draft`
18. `arc-07-03-peer-review`
19. `arc-07-04-paper-revision`
20. `arc-08-01-quality-gate` **[GATE]**
21. `arc-08-02-knowledge-archive`
22. `arc-08-03-export-publish`
23. `arc-08-04-citation-verify`

## Gate and rollback policy (canonical)
If gate is rejected:
- `arc-02-03` -> rollback to `arc-02-02`
- `arc-04-01` -> rollback to `arc-03-02`
- `arc-08-01` -> rollback to `arc-07-01`

## Decision loop policy (stage 15 equivalent)
At `arc-06-02-research-decision`:
- `proceed` -> continue to `arc-07-01`
- `refine` -> jump to `arc-05-02`
- `pivot` -> jump to `arc-03-02`

Enforce canonical cap:
- Maximum pivot/refine loops: `2` (`MAX_DECISION_PIVOTS`)
- If cap reached, force `proceed`, write `quality_warning.txt`, and continue to paper stages.

Before each loop rollback:
- Version rollback range directories to avoid overwrite (e.g., `stage-08_v1`, `stage-09_v1`, ...).

## Retry and status handling
- Respect per-stage retry caps from contracts:
  - stage 4: 2, stage 10: 2, stage 12: 2, stage 13: 2 (others default as defined)
- Status handling:
  - `running` -> execute stage
  - `blocked_approval` -> wait for approve/reject
  - `retrying` -> retry same stage until cap
  - `paused` -> stop and wait for `/arc-00-03-resume`
  - `rejected` -> stop pipeline
  - `failed` -> stop unless noncritical skip applies
  - `done` -> checkpoint + heartbeat update

## Noncritical stage policy
If `skip_noncritical=true`, failures on these stages do not abort:
- stage 20 `arc-08-01-quality-gate`
- stage 21 `arc-08-02-knowledge-archive`

## History event schema
Each line in `stage_history.jsonl` should be JSON with:
- `timestamp` (ISO8601)
- `event_type` (`pipeline_start|stage_start|stage_end|stage_fail|pipeline_end`)
- `stage_number`
- `stage_skill`
- `status`
- `decision` (optional)
- `elapsed_sec` (optional)
- `error` (optional)

## Execution rules
- Before each stage: write `pipeline_state.json` status `running`; append `stage_start` event.
- On success: write `checkpoint.json`; append `stage_end`; update `heartbeat.json`.
- On failure: append `stage_fail`; apply retry/noncritical logic.
- After stage 14 done: run diagnosis/repair hook if available (`experiment_diagnosis.json`, optional repair pass).
- Always store stage outputs in `artifacts/<run_id>/stage-XX/`.
- At end: write `pipeline_summary.json` and append `pipeline_end` event.

## Related commands
- `/arc-00-02-status` to inspect progress
- `/arc-00-03-resume` to continue from checkpoint
