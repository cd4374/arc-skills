---
name: arc-02-04-knowledge-extract
description: Stage 6 (KNOWLEDGE_EXTRACT). Extract structured cards from shortlisted papers.
metadata:
  category: pipeline-stage
  trigger-keywords: "knowledge extract,cards,stage 6"
  applicable-stages: "6"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 6 `KNOWLEDGE_EXTRACT`

## Inputs
From `artifacts/<run_id>/stage-05/`:
- `shortlist.jsonl`

## Outputs
Write to `artifacts/<run_id>/stage-06/`:
- `cards/`

## Definition of done
Each shortlisted paper has a structured knowledge card.
