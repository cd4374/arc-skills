---
name: arc-00-03-resume
description: Resume a pure skills ARC run from checkpoint and continue stage chaining.
metadata:
  category: orchestrator
  trigger-keywords: "arc resume,resume pipeline,continue run,checkpoint resume"
  applicable-stages: "1-23"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Continue execution from `artifacts/<run_id>/state/checkpoint.json`.

## Inputs
- `run_id` (required)
- `auto_approve_gates` (optional bool)

## Resume behavior
1. Load `checkpoint.json` and `pipeline_state.json`.
2. Determine next skill after `last_completed_skill`.
3. If status is `blocked_approval`, ask user to approve/reject first.
4. Re-enter `/arc-00-01-research-pipeline` with `from=<next_skill>`.

## Safety rules
- Do not overwrite previous stage outputs.
- Append all new events to `stage_history.jsonl`.
- Keep rollback and decision routing identical to `arc-00-01`.
