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
From `artifacts/<run_id>/stage-12/` and `stage-13/`:
- `runs/`
- `refinement_log.json`
- `experiment_final/`

## Outputs
Write to `artifacts/<run_id>/stage-14/`:
- `analysis.md`

## Definition of done
Quantitative analysis and conclusions are documented with statistical rationale.

## Post-analysis hook (canonical parity)
After stage 14 is done, run experiment diagnosis (if available) and honor repair-needed signals before finalizing downstream decisions.
Expected run-root artifacts:
- `experiment_diagnosis.json`
- optional repair prompt/output artifacts
