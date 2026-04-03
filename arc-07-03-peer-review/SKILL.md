---
name: arc-07-03-peer-review
description: Stage 18 (PEER_REVIEW). Generate multi-perspective review feedback for the draft.
metadata:
  category: pipeline-stage
  trigger-keywords: "peer review,review draft,stage 18"
  applicable-stages: "18"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 18 `PEER_REVIEW`

## Inputs
From `artifacts/<run_id>/stage-17/`:
- `paper_draft.md`

## Outputs
Write to `artifacts/<run_id>/stage-18/`:
- `reviews.md`

## Definition of done
At least 2 review perspectives with actionable and evidence-aware feedback.
