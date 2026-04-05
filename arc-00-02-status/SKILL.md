---
name: arc-00-02-status
description: Show current progress of a pure skills ARC run from run-root state files.
metadata:
  category: orchestrator
  trigger-keywords: "arc status,pipeline status,run status,checkpoint"
  applicable-stages: "1-23"
  priority: "1"
  version: "1.1"
  author: researchclaw
---

## Purpose
Report current stage, state, and recent events for an ARC skills run.

## Inputs
- `run_id` (required)

## Read (run root)
- `artifacts/<run_id>/pipeline_state.json`
- `artifacts/<run_id>/checkpoint.json`
- `artifacts/<run_id>/stage_history.jsonl`
- `artifacts/<run_id>/pipeline_summary.json` (if exists)
- `artifacts/<run_id>/heartbeat.json` (if exists)

## Output format
- Current skill and canonical stage number
- Status (`pending|running|blocked_approval|approved|rejected|paused|retrying|failed|done`)
- Last completed stage from checkpoint
- Last 5 history events
- Heartbeat snapshot (if available)
- Final summary snapshot (if available)
- Next suggested command:
  - continue -> `/arc-00-03-resume`
  - inspect gate -> approve/reject instruction

If files are missing, report no run state found and ask user to start with `/arc-00-01-research-pipeline`.
