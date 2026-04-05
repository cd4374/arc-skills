---
name: arc-07-01-paper-outline
description: Stage 16 (PAPER_OUTLINE). Build structured paper outline from analysis and decision.
metadata:
  category: pipeline-stage
  trigger-keywords: "paper outline,stage 16"
  applicable-stages: "16"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 16 `PAPER_OUTLINE`

## Inputs
From `artifacts/<run_id>/stage-14/` and `stage-15/`:
- `analysis.md`
- `decision.md`

Prefer best-of-refine artifacts when present (run root):
- `analysis_best.md`
- `experiment_summary_best.json`

## Outputs
Write to `artifacts/<run_id>/stage-16/`:
- `outline.md`

## Definition of done
Complete paper structure with section-level objectives and evidence mapping.
