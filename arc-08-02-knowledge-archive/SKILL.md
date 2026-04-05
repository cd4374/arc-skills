---
name: arc-08-02-knowledge-archive
description: Stage 21 (KNOWLEDGE_ARCHIVE). Archive retrospectives and reproducibility bundle index.
metadata:
  category: pipeline-stage
  trigger-keywords: "knowledge archive,retrospective,bundle,stage 21"
  applicable-stages: "21"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 21 `KNOWLEDGE_ARCHIVE`

## Inputs
Aggregate from run-root and stages:
- `stage_history.jsonl`
- `decision_history.json`
- `pipeline_summary.json` (if present)
- stage health and outputs from `stage-01`..`stage-23`
- final paper/export artifacts

## Outputs
Write to `artifacts/<run_id>/stage-21/`:
- `archive.md`
- `bundle_index.json`

## Definition of done
Retrospective and reproducibility index are generated and linked.
