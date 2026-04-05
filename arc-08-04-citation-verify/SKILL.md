---
name: arc-08-04-citation-verify
description: Stage 23 (CITATION_VERIFY). Verify citations and produce verification report.
metadata:
  category: pipeline-stage
  trigger-keywords: "citation verify,references,bib,stage 23"
  applicable-stages: "23"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 23 `CITATION_VERIFY`

## Inputs
From `artifacts/<run_id>/stage-22/`:
- `paper_final.md`

## Outputs
Write to `artifacts/<run_id>/stage-23/`:
- `verification_report.json`
- `references_verified.bib`

## Definition of done
All citations are validated against real sources and hallucinations are flagged.

## Blocking rule
If hallucinated or unverifiable citations remain, mark stage as failed (`E23_VERIFY_FAIL`) and block completion until fixed/removed.
