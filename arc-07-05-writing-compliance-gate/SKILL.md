---
name: arc-07-05-writing-compliance-gate
description: Stage 17.5 (GATE) — Enforce writing structure, venue checklist, and anti-slop compliance before peer review proceeds.
metadata:
  category: pipeline-stage
  trigger-keywords: "writing compliance gate,anti ai writing,venue checklist,stage 17.5"
  applicable-stages: "17.5"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Catch structural writing failures before peer review and later quality gates waste effort on a non-compliant draft. This stage focuses on required sections, abstract range, lingering verification markers, venue checklist readiness, and obvious AI-slop patterns.

---

## Quality Contract

`writing_compliance_report.json` MUST:
- Confirm all required sections are present
- Confirm the abstract is within the allowed range for the current workflow
- Confirm no `TODO`, `FIXME`, `XXX`, or `[VERIFY]` markers remain
- Confirm a specific Limitations section exists
- Confirm venue checklist readiness using the resolved template contract
- Record any anti-AI writing pattern violations that remain after cleanup
- Set `writing_compliance_pass: true` only if all blocking checks pass

Hard constraints:
- Required sections MUST exist
- Limitations section MUST name at least one specific limitation
- No unresolved `TODO`/`FIXME`/`[VERIFY]` markers may remain
- Draft must be free of the most common forbidden filler and inflated-claim phrases recorded by the pipeline

---

## Inputs
`artifacts/<run_id>/stage-17/paper_draft.md`
`artifacts/<run_id>/template/template_manifest.json`
`artifacts/<run_id>/stage-15b/claim_scope_report.json`

---

## Outputs

### `artifacts/<run_id>/stage-17b/writing_compliance_report.json`
```json
{
  "writing_compliance_pass": true,
  "section_checks": {
    "abstract": true,
    "introduction": true,
    "related_work": true,
    "method": true,
    "experiments": true,
    "limitations": true,
    "conclusion": true
  },
  "marker_checks": {
    "todo_markers": 0,
    "verify_markers": 0
  },
  "anti_ai_patterns": {
    "forbidden_words_found": [],
    "inflated_claims_found": []
  },
  "blocking_issues": [],
  "warnings": []
}
```

### `artifacts/<run_id>/stage-17b/writing_compliance_checklist.md`
A concise checklist showing pass/fail state for section presence, limitations, unresolved markers, and venue readiness.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Required sections present | Abstract, Introduction, Related Work, Method, Experiments, Limitations, Conclusion all exist |
| Abstract in range | Abstract is within the configured word-count range |
| Limitations explicit | Specific limitation is present |
| No unresolved markers | `TODO`, `FIXME`, `XXX`, and `[VERIFY]` counts are all zero |
| Claim scope consumable | Draft does not visibly violate the claim-scope contract |
| Gate pass flag | `writing_compliance_pass == true` |

**Failure**: Missing required sections or unresolved markers → `E17B`

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `writing_compliance_pass=true` | `done`, advance to stage 18 |
| `approve` and `writing_compliance_pass=true` | `done`, advance to stage 18 |
| `reject` or `writing_compliance_pass=false` | `rejected`, rollback to stage 17 |

---

## Rollback Contract
On `reject` or `writing_compliance_pass=false`: rollback target is **stage 17** (`arc-07-02-paper-draft`).

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E17B` | Writing compliance report missing or blocking checks failed |
| `E17B_MISSING_SECTION` | One or more required sections are missing |
| `E17B_LIMITATIONS_MISSING` | Limitations section missing or too vague |
| `E17B_MARKERS_REMAIN` | `TODO`, `FIXME`, `XXX`, or `[VERIFY]` markers remain |
| `E17B_AI_PATTERNS` | Forbidden inflated or filler writing patterns remain |

---

## Retry Policy
Max retries: **0** — gate rejection uses rollback path.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`

---

## Procedure

### Step 1 — Read Draft and Contracts
Read `paper_draft.md`, `template_manifest.json`, and `claim_scope_report.json`.

### Step 2 — Check Structural Sections
Confirm the required sections exist and the Limitations section contains at least one specific limitation.

### Step 3 — Scan for Unresolved Markers
Search the draft for `TODO`, `FIXME`, `XXX`, and `[VERIFY]`. Count them and treat any remainder as blocking.

### Step 4 — Check Anti-Slop Patterns
Scan for known forbidden filler and inflated-claim patterns used elsewhere in the pipeline, including obvious AI-slop words or unsupported hype language.

### Step 5 — Check Venue Readiness
Use the template contract to verify that the draft is structurally ready for the target venue and does not visibly violate the claim-scope contract.

### Step 6 — Write Outputs
Write `writing_compliance_report.json` and `writing_compliance_checklist.md` under `stage-17b/`.

### Step 7 — Gate Decision
If required sections are missing, limitations are absent, unresolved markers remain, or major anti-slop patterns remain, fail with `E17B`. Otherwise block for approval and then advance to stage 18.
