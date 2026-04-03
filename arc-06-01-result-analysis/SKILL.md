---
name: arc-06-01-result-analysis
description: Stage 14 (RESULT_ANALYSIS). Analyze metrics and summarize evidence.
metadata:
  category: pipeline-stage
  trigger-keywords: "result analysis,metrics,statistics,stage 14"
  applicable-stages: "14"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 14 `RESULT_ANALYSIS`

## Inputs
From `artifacts/<run_id>/stage-12/`:
- `runs/`

## Outputs
Write to `artifacts/<run_id>/stage-14/`:
- `analysis.md`

## Definition of done
Quantitative analysis and conclusions are documented with statistical rationale.
