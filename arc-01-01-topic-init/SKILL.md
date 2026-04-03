---
name: arc-01-01-topic-init
description: Stage 1 (TOPIC_INIT). Define SMART research goal and hardware profile for the run.
metadata:
  category: pipeline-stage
  trigger-keywords: "topic init,goal,stage 1"
  applicable-stages: "1"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 1 `TOPIC_INIT`

## Inputs
- User topic and constraints

## Outputs
Write to `artifacts/<run_id>/stage-01/`:
- `goal.md`
- `hardware_profile.json`

## Definition of done
SMART goal statement with scope/constraints and hardware profile captured.
