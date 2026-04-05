---
name: arc-08-01-quality-gate
description: Stage 20 (QUALITY_GATE, gate). Evaluate final draft quality and require approval.
metadata:
  category: pipeline-stage
  trigger-keywords: "quality gate,approval,stage 20"
  applicable-stages: "20"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 20 `QUALITY_GATE` (GATE)

## Inputs
From `artifacts/<run_id>/stage-19/`:
- `paper_revised.md`

## Outputs
Write to `artifacts/<run_id>/stage-20/`:
- `quality_report.json`

## Definition of done
Quality assessment completed and explicit approve/reject decision recorded.

## Gate behavior
- If `auto_approve_gates=true`: auto-approve and continue
- Approve: continue to `arc-08-02-knowledge-archive`
- Reject: rollback to `arc-07-01-paper-outline`
