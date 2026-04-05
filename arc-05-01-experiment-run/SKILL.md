---
name: arc-05-01-experiment-run
description: Stage 12 (EXPERIMENT_RUN). Execute experiment runs and collect artifacts.
metadata:
  category: pipeline-stage
  trigger-keywords: "experiment run,execute,runs,stage 12"
  applicable-stages: "12"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 12 `EXPERIMENT_RUN`

## Inputs
From `artifacts/<run_id>/stage-10/` and `stage-11/`:
- `experiment/`
- `schedule.json`

## Outputs
Write to `artifacts/<run_id>/stage-12/`:
- `runs/`

## Definition of done
Scheduled runs complete with metrics and run artifacts persisted.

## Retry policy
- Max retries: `2` on execution failures.
