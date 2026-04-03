---
name: arc-05-02-iterative-refine
description: Stage 13 (ITERATIVE_REFINE). Perform edit-run-eval refinements until convergence or budget limit.
metadata:
  category: pipeline-stage
  trigger-keywords: "iterative refine,repair,converge,stage 13"
  applicable-stages: "13"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 13 `ITERATIVE_REFINE`

## Inputs
From `artifacts/<run_id>/stage-12/`:
- `runs/`

## Outputs
Write to `artifacts/<run_id>/stage-13/`:
- `refinement_log.json`
- `experiment_final/`

## Definition of done
Refinement loop converges or exits at max iteration with final experiment snapshot.
