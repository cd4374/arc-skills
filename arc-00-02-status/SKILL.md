---
name: arc-00-02-status
description: Show current progress of a pure skills ARC run from artifacts state files.
metadata:
  category: orchestrator
  trigger-keywords: "arc status,pipeline status,run status,checkpoint"
  applicable-stages: "1-23"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Report current stage, state, and recent events for an ARC skills run.

## Inputs
- `run_id` (required)

## Read
- `artifacts/<run_id>/state/pipeline_state.json`
- `artifacts/<run_id>/state/checkpoint.json`
- `artifacts/<run_id>/state/stage_history.jsonl`

## Output format
- Current skill and canonical stage number
- Status (`pending|running|blocked_approval|failed|done`)
- Last completed skill
- Last 5 history events
- Next suggested command:
  - continue -> `/arc-00-03-resume`
  - inspect gate -> approve/reject instruction

If state files are missing, report that no run state was found and ask user to start with `/arc-00-01-research-pipeline`.
