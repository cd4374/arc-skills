---
name: arc-04-03-resource-planning
description: Stage 11 (RESOURCE_PLANNING). Build execution schedule and resource estimates.
metadata:
  category: pipeline-stage
  trigger-keywords: "resource planning,schedule,gpu,time,stage 11"
  applicable-stages: "11"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 11 `RESOURCE_PLANNING`

## Inputs
From `artifacts/<run_id>/stage-09/`:
- `exp_plan.yaml`

## Outputs
Write to `artifacts/<run_id>/stage-11/`:
- `schedule.json`

## Definition of done
A feasible schedule with compute/time/resource estimates is produced.
