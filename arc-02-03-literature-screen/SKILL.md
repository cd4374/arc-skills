---
name: arc-02-03-literature-screen
description: Stage 5 (LITERATURE_SCREEN, gate). Screen candidates and block for approval.
metadata:
  category: pipeline-stage
  trigger-keywords: "literature screen,gate,stage 5"
  applicable-stages: "5"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 5 `LITERATURE_SCREEN` (GATE)

## Inputs
From `artifacts/<run_id>/stage-04/`:
- `candidates.jsonl`

## Outputs
Write to `artifacts/<run_id>/stage-05/`:
- `shortlist.jsonl`

## Definition of done
Relevance and quality screening completed with approval decision.

## Gate behavior
- If `auto_approve_gates=true`: auto-approve and continue
- Approve: continue to `arc-02-04-knowledge-extract`
- Reject: rollback to `arc-02-02-literature-collect`
