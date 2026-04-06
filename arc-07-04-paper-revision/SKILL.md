---
name: arc-07-04-paper-revision
description: Stage 19 — Address all major peer review concerns and produce a revised paper with tracked changes.
metadata:
  category: pipeline-stage
  trigger-keywords: "paper revision,revise draft,stage 19"
  applicable-stages: "19"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Address the major concerns from peer reviews and produce a revised paper. Every major concern must be explicitly tracked with the action taken.

---

## Quality Contract

`paper_revised.md` MUST satisfy:
- All major concerns from `reviews.md` are addressed
- `revision_log.md` tracks every major concern with: the original concern, action taken, and location in the revised paper
- The revised paper does not introduce new unsubstantiated claims while addressing concerns
- All sections from `paper_draft.md` remain present and coherent

---

## Inputs
`artifacts/<run_id>/stage-17/paper_draft.md`
`artifacts/<run_id>/stage-18/reviews.md`
`artifacts/<run_id>/template/template_manifest.json`

---

## Outputs

### `paper_revised.md`
Revised paper with all review feedback addressed.

### `revision_log.md`
```
# Revision Log

## Major Concerns

### Reviewer 1: [Concern — copy of original]
- **Action**: Addressed / Partially addressed / Not addressed
- **Location**: [Section in revised paper]
- **Rationale**: [Why this action was taken]

## Reviewer 2: ...
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Major concerns addressed | Every `### Major Concerns` item from reviews.md has an entry in revision_log |
| `revision_log.md` exists | File present |
| All sections present | Revised paper contains all sections from draft |
| No new unsubstantiated claims | No metric claims not supported by experiment_summary |

**Failure**: If any major concern has no corresponding revision_log entry → `E19`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E19` | Major concern from reviews not tracked in revision_log |

---

## Retry Policy
Max retries: **1** — retry if revision_log doesn't cover all major concerns.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Draft, Reviews, and Template Contract
Read `paper_draft.md`, `reviews.md`, and `template_manifest.json`. For each major concern, note: the concern text, which reviewer raised it, and the section it references.

### Step 2 — Triage Concerns
For each major concern:
- **Address**: The concern is valid and can be fixed by adding evidence, clarifying text, or adding an experiment → plan the fix
- **Partially address**: The concern has merit but the fix is partial → plan partial fix + provide rationale
- **Reject**: The concern is based on a misunderstanding or is out of scope → plan a rebuttal in the revision log

### Step 3 — Revise Paper
Create `paper_revised.md` based on `paper_draft.md`, applying the planned fixes:
- Add missing evidence to support claims
- Rewrite unclear sections to address reviewer concerns
- Add additional experiments if feasible (within time budget)
- Preserve template-sensitive constraints from `template_manifest.json` (anonymous mode, required sections, page-policy assumptions)
- Do NOT introduce new unsubstantiated claims while addressing concerns

### Step 4 — Write revision_log.md
For every major concern from every reviewer, document:
- The original concern (copy the exact text from reviews.md)
- **Action taken**: Addressed / Partially addressed / Not addressed
- **Location**: Which section and paragraph was changed in `paper_revised.md`
- **Rationale**: Why this action was taken

### Step 5 — Validate
Check: all major concerns have a corresponding entry in `revision_log.md`, all original sections are still present in the revised paper, no new unsubstantiated claims introduced. Fail → `E19`.
