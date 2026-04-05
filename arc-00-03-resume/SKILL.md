---
name: arc-00-03-resume
description: Resume a pure skills ARC run from checkpoint and continue stage chaining.
metadata:
  category: orchestrator
  trigger-keywords: "arc resume,resume pipeline,continue run,checkpoint resume"
  applicable-stages: "1-23"
  priority: "1"
  version: "1.1"
  author: researchclaw
---

## Purpose
Continue execution from `artifacts/<run_id>/checkpoint.json`.

## Inputs
- `run_id` (required)
- `auto_approve_gates` (optional bool)

## Resume behavior
1. Load `checkpoint.json` and `pipeline_state.json` from run root.
2. Resolve next skill from `last_completed_stage` (+1 in stage sequence).
3. If status is `blocked_approval`, detect blocked gate and require explicit user decision:
   - approve -> resume from next stage
   - reject -> resume from rollback target:
     - gate stage 5 -> `arc-02-02-literature-collect`
     - gate stage 9 -> `arc-03-02-hypothesis-gen`
     - gate stage 20 -> `arc-07-01-paper-outline`
4. If status is `paused`, resume same stage.
5. Re-enter `/arc-00-01-research-pipeline` with `from=<resolved_skill>`.

## Safety rules
- Do not overwrite previous outputs without stage versioning.
- Append new entries to `stage_history.jsonl`.
- Keep rollback and decision routing identical to `arc-00-01`.
