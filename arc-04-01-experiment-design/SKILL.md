---
name: arc-04-01-experiment-design
description: Stage 9 (EXPERIMENT_DESIGN, gate). Build experiment plan with baselines, ablations, and metrics.
metadata:
  category: pipeline-stage
  trigger-keywords: "experiment design,plan,stage 9,gate"
  applicable-stages: "9"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 9 `EXPERIMENT_DESIGN` (GATE)

## Inputs
From `artifacts/<run_id>/stage-08/`:
- `hypotheses.md`

## Outputs
Write to `artifacts/<run_id>/stage-09/`:
- `exp_plan.yaml`

## Definition of done
Experiment plan includes baselines, ablations, metrics, and approval decision.

## Gate behavior
- Approve: continue to `arc-04-02-code-generation`
- Reject: rollback to `arc-03-02-hypothesis-gen`
