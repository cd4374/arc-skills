---
name: arc-00-01-research-pipeline
description: Pure skills orchestration for AutoResearchClaw 23-stage pipeline. Use when user asks to run end-to-end research in skills mode with arc-xx-xx stage chain.
metadata:
  category: orchestrator
  trigger-keywords: "arc,pipeline,skills-only,research workflow,run research"
  applicable-stages: "1-23"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Run the complete ARC skills pipeline by chaining `arc-01-01` through `arc-08-04` with canonical gate and rollback behavior.

## Inputs
- `topic` (required)
- `run_id` (optional; default `rc-YYYYMMDD-HHMMSS`)
- `from` (optional start skill, e.g. `arc-04-02-code-generation`)
- `auto_approve_gates` (optional bool, default false)

## State files
Write/update under `artifacts/<run_id>/state/`:
- `pipeline_state.json` (current stage/skill/status)
- `checkpoint.json` (last completed skill)
- `stage_history.jsonl` (append-only events)

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

## Execution rules
- Before each stage: mark status `running` in `pipeline_state.json` and append history event.
- On success: update checkpoint and append `done` event.
- On failure: append `failed` event and stop unless user asks to continue.
- Always store stage outputs in `artifacts/<run_id>/stage-XX/`.

## Related commands
- `/arc-00-02-status` to inspect progress
- `/arc-00-03-resume` to continue from checkpoint
