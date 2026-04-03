---
name: arc-04-02-code-generation
description: Stage 10 (CODE_GENERATION). Generate runnable experiment project and spec.
metadata:
  category: pipeline-stage
  trigger-keywords: "code generation,experiment code,stage 10"
  applicable-stages: "10"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 10 `CODE_GENERATION`

## Inputs
From `artifacts/<run_id>/stage-09/`:
- `exp_plan.yaml`

## Outputs
Write to `artifacts/<run_id>/stage-10/`:
- `experiment/`
- `experiment_spec.md`

## Definition of done
Multi-file runnable experiment project and implementation spec are generated.
