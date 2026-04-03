---
name: arc-03-02-hypothesis-gen
description: Stage 8 (HYPOTHESIS_GEN). Produce falsifiable hypotheses grounded in synthesis.
metadata:
  category: pipeline-stage
  trigger-keywords: "hypothesis gen,hypotheses,stage 8"
  applicable-stages: "8"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 8 `HYPOTHESIS_GEN`

## Inputs
From `artifacts/<run_id>/stage-07/`:
- `synthesis.md`

## Outputs
Write to `artifacts/<run_id>/stage-08/`:
- `hypotheses.md`

## Definition of done
At least 2 falsifiable hypotheses with measurable implications.
