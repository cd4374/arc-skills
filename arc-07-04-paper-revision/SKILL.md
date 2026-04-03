---
name: arc-07-04-paper-revision
description: Stage 19 (PAPER_REVISION). Revise paper by addressing review feedback.
metadata:
  category: pipeline-stage
  trigger-keywords: "paper revision,revise draft,stage 19"
  applicable-stages: "19"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 19 `PAPER_REVISION`

## Inputs
From `artifacts/<run_id>/stage-17/` and `stage-18/`:
- `paper_draft.md`
- `reviews.md`

## Outputs
Write to `artifacts/<run_id>/stage-19/`:
- `paper_revised.md`

## Definition of done
Review comments are addressed with coherent revisions and tracked rationale.
