---
name: arc-02-01-search-strategy
description: Stage 3 (SEARCH_STRATEGY). Build literature search plan, sources, and queries.
metadata:
  category: pipeline-stage
  trigger-keywords: "search strategy,literature plan,stage 3"
  applicable-stages: "3"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 3 `SEARCH_STRATEGY`

## Inputs
From `artifacts/<run_id>/stage-02/`:
- `problem_tree.md`

## Outputs
Write to `artifacts/<run_id>/stage-03/`:
- `search_plan.yaml`
- `sources.json`
- `queries.json`

## Definition of done
At least 2 search strategies with verified sources and concrete queries.
