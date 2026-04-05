---
name: arc-02-02-literature-collect
description: Stage 4 (LITERATURE_COLLECT). Retrieve candidate papers from planned sources.
metadata:
  category: pipeline-stage
  trigger-keywords: "literature collect,candidates,stage 4"
  applicable-stages: "4"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 4 `LITERATURE_COLLECT`

## Inputs
From `artifacts/<run_id>/stage-03/`:
- `search_plan.yaml`

## Outputs
Write to `artifacts/<run_id>/stage-04/`:
- `candidates.jsonl`

## Definition of done
Candidate paper set collected from specified sources.

## Retry policy
- Max retries: `2` on transient collection failures.
