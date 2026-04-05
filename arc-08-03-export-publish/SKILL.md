---
name: arc-08-03-export-publish
description: Stage 22 (EXPORT_PUBLISH). Export final paper package and code bundle.
metadata:
  category: pipeline-stage
  trigger-keywords: "export publish,final package,stage 22"
  applicable-stages: "22"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 22 `EXPORT_PUBLISH`

## Inputs
From `artifacts/<run_id>/stage-19/`:
- `paper_revised.md`

## Outputs
Write to `artifacts/<run_id>/stage-22/` (and deliverables where applicable):
- `paper_final.md`
- `paper.tex`
- `references.bib`
- `code/`
- `charts/`
- `manifest.json`
- conference template/style files as needed

## Definition of done
Final paper export is complete in target format with code bundle attached.
