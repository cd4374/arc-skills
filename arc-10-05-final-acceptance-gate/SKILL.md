---
name: arc-10-05-final-acceptance-gate
description: Stage 28.5 (GATE) — Aggregate all critical gate outcomes and final package completeness before declaring the paper pipeline accepted.
metadata:
  category: pipeline-stage
  trigger-keywords: "final acceptance gate,final package gate,paper acceptance gate,stage 28.5"
  applicable-stages: "28.5"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Provide a single final acceptance decision that aggregates the entire pipeline's hard guarantees. This stage does not generate new content. It only checks whether all critical upstream gates passed and whether the final paper package is complete enough to count as a real, submission-ready research artifact.

---

## Quality Contract

`final_acceptance_report.json` MUST:
- Confirm every critical blocking gate passed
- Confirm the final package includes at least `main.tex`, compiled `main.pdf`, bibliography, and submission manifest artifacts
- Confirm final citation verification, figure quality, numeric truth, reproducibility bundle, and academic integrity checks all passed
- Record any remaining non-blocking warnings
- Set `final_acceptance_pass: true` only if all blocking checks pass

Hard constraints:
- `main.tex` and compiled `main.pdf` MUST exist
- Bibliography and citation-verification outputs MUST exist and pass
- Figure-quality, numeric-truth, reproducibility-bundle, submission-format, and academic-integrity gates MUST all pass
- No blocking issue from any critical upstream gate may remain open

---

## Inputs
`artifacts/<run_id>/stage-22/manifest.json`
`artifacts/<run_id>/stage-23/verification_report.json`
`artifacts/<run_id>/stage-24/review_state.json`
`artifacts/<run_id>/stage-24b/academic_integrity_report.json`
`artifacts/<run_id>/stage-27/figure_quality_report.json`
`artifacts/<run_id>/stage-26-pre/numeric_truth_report.json`
`artifacts/<run_id>/stage-21b/reproducibility_bundle_report.json`
`artifacts/<run_id>/stage-28/submission_format_report.json`
`artifacts/<run_id>/stage-25/main.tex`
`artifacts/<run_id>/stage-25/main.pdf`
`artifacts/<run_id>/stage-25/references.bib`

---

## Outputs

### `artifacts/<run_id>/stage-28b/final_acceptance_report.json`
```json
{
  "final_acceptance_pass": true,
  "critical_gate_status": {
    "citation_verification": true,
    "submission_format": true,
    "academic_integrity": true,
    "figure_quality": true,
    "numeric_truth": true,
    "reproducibility_bundle": true
  },
  "final_package": {
    "main_tex_present": true,
    "main_pdf_present": true,
    "bibliography_present": true,
    "manifest_present": true
  },
  "warnings": [],
  "blocking_issues": []
}
```

### `artifacts/<run_id>/stage-28b/final_acceptance_summary.md`
A short human-readable acceptance summary showing whether the pipeline produced a real final paper bundle and which critical gates were required for acceptance.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Critical gates passed | All required upstream gate reports exist and pass |
| Final package complete | `main.tex`, `main.pdf`, bibliography, and manifest exist |
| Citation verification passed | Citation verification output exists and passes |
| Integrity passed | Academic integrity gate exists and passes |
| Figure and numeric truth passed | Figure quality and numeric truth gates both pass |
| Reproducibility bundle passed | Reproducibility bundle gate passes |
| Gate pass flag | `final_acceptance_pass == true` |

**Failure**: Missing critical artifacts or failed upstream gate → `E28B`

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `final_acceptance_pass=true` | `done`, pipeline complete |
| `approve` and `final_acceptance_pass=true` | `done`, pipeline complete |
| `reject` or `final_acceptance_pass=false` | `rejected`, rollback to stage 28 |

---

## Rollback Contract
On `reject` or `final_acceptance_pass=false`: rollback target is **stage 28** (`arc-10-03-submission-format-gate`).

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E28B` | Final acceptance report missing or blocking checks failed |
| `E28B_PACKAGE_INCOMPLETE` | Final paper package is incomplete |
| `E28B_CRITICAL_GATE_FAILED` | One or more critical upstream gates failed |
| `E28B_CITATION_VERIFY_MISSING` | Citation verification output missing or failed |
| `E28B_INTEGRITY_MISSING` | Academic integrity output missing or failed |

---

## Retry Policy
Max retries: **0** — gate rejection uses rollback path.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`

---

## Procedure

### Step 1 — Read Final Reports
Read the final export manifest and all critical upstream gate reports.

### Step 2 — Check Critical Gate Aggregation
Verify that citation verification, submission format, academic integrity, reproducibility bundle, figure quality, and numeric truth all passed.

### Step 3 — Check Final Package Completeness
Verify that the final package contains `main.tex`, compiled `main.pdf`, bibliography artifacts, and the final manifest.

### Step 4 — Write Outputs
Write `final_acceptance_report.json` and `final_acceptance_summary.md` under `stage-28b/`.

### Step 5 — Gate Decision
If any critical upstream gate failed or the final package is incomplete, fail with `E28B`. Otherwise block for approval and then advance to stage 29.
