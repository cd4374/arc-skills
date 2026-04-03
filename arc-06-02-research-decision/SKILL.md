---
name: arc-06-02-research-decision
description: Stage 15 (RESEARCH_DECISION). Decide PROCEED, REFINE, or PIVOT based on evidence.
metadata:
  category: pipeline-stage
  trigger-keywords: "research decision,proceed,refine,pivot,stage 15"
  applicable-stages: "15"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 15 `RESEARCH_DECISION`

## Inputs
From `artifacts/<run_id>/stage-14/`:
- `analysis.md`

## Outputs
Write to `artifacts/<run_id>/stage-15/`:
- `decision.md`

## Definition of done
Decision is evidence-grounded and explicitly one of: `proceed`, `refine`, `pivot`.

## Routing contract
- `proceed` -> next `arc-07-01-paper-outline`
- `refine` -> jump `arc-05-02-iterative-refine`
- `pivot` -> jump `arc-03-02-hypothesis-gen`
