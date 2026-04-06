---
name: arc-09-03-academic-integrity-gate
description: Stage 24.5 (GATE) — Check anonymity, disclosure, citation ethics, and integrity declarations before final acceptance proceeds.
metadata:
  category: pipeline-stage
  trigger-keywords: "academic integrity gate,anonymity gate,coi disclosure,stage 24.5"
  applicable-stages: "24.5"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Block papers that are technically complete but not submission-safe due to integrity failures such as missing disclosures, broken anonymity, unbounded self-citation, or unresolved ethical declarations. This stage ensures the final package is not only factual, but ethically and procedurally publishable.

---

## Quality Contract

`academic_integrity_report.json` MUST:
- Confirm required disclosure fields are present for the current venue/workflow
- Confirm anonymity status is consistent with the target submission mode
- Confirm self-citation is disclosed and within configured policy bounds
- Confirm AI-use disclosure status is explicit
- Confirm no obvious plagiarism placeholders, copied-review text, or unresolved authorship/conflict markers remain
- Set `academic_integrity_pass: true` only if all blocking checks pass

Hard constraints:
- Required disclosure fields MUST be explicit
- Anonymous submissions MUST not expose author-identifying metadata in the paper body or package notes
- COI / funding / ethics / AI-use status MUST be explicit when applicable
- Unresolved disclosure markers (`TODO`, `[DISCLOSE]`, `[COI]`, `[AUTHOR]`) MUST not remain

---

## Inputs
`artifacts/<run_id>/stage-24/review_state.json`
`artifacts/<run_id>/stage-22/main.tex`
`artifacts/<run_id>/stage-22/main.pdf`
`artifacts/<run_id>/template/template_manifest.json`

---

## Outputs

### `artifacts/<run_id>/stage-24b/academic_integrity_report.json`
```json
{
  "academic_integrity_pass": true,
  "checks": {
    "anonymity_ok": true,
    "coi_disclosure_explicit": true,
    "funding_disclosure_explicit": true,
    "ai_use_disclosure_explicit": true,
    "self_citation_policy_ok": true,
    "unresolved_disclosure_markers": 0
  },
  "warnings": [],
  "blocking_issues": []
}
```

### `artifacts/<run_id>/stage-24b/academic_integrity_checklist.md`
A concise checklist for anonymity, COI/funding/AI-use disclosures, self-citation policy, and unresolved integrity markers.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Anonymity consistent | Anonymous workflows do not leak author identity |
| Disclosure explicit | COI/funding/AI-use status is explicit where applicable |
| Self-citation policy ok | Self-citation is bounded and disclosed |
| No unresolved markers | Disclosure placeholders and unresolved markers are zero |
| Gate pass flag | `academic_integrity_pass == true` |

**Failure**: Integrity/disclosure/anonymity checks fail → `E24B`

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `academic_integrity_pass=true` | `done`, advance to stage 25 |
| `approve` and `academic_integrity_pass=true` | `done`, advance to stage 25 |
| `reject` or `academic_integrity_pass=false` | `rejected`, rollback to stage 24 |

---

## Rollback Contract
On `reject` or `academic_integrity_pass=false`: rollback target is **stage 24** (`arc-09-01-paper-review-loop`).

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E24B` | Academic integrity report missing or blocking checks failed |
| `E24B_ANONYMITY_LEAK` | Anonymous submission leaks identifying information |
| `E24B_DISCLOSURE_MISSING` | Required disclosure field missing or unresolved |
| `E24B_SELF_CITATION_POLICY` | Self-citation exceeds configured policy or is undisclosed |
| `E24B_MARKERS_REMAIN` | Integrity/disclosure markers remain unresolved |

---

## Retry Policy
Max retries: **0** — gate rejection uses rollback path.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`

---

## Procedure

### Step 1 — Read Submission Artifacts
Read the review state, final LaTeX/PDF artifacts, and template manifest.

### Step 2 — Check Anonymity and Metadata
Verify whether the package matches the target anonymity mode and does not expose disallowed author-identifying information.

### Step 3 — Check Required Disclosures
Verify COI, funding, ethics, and AI-use disclosure fields are explicit for the current workflow.

### Step 4 — Check Citation Ethics
Check self-citation policy compliance and scan for unresolved disclosure/integrity markers.

### Step 5 — Write Outputs
Write `academic_integrity_report.json` and `academic_integrity_checklist.md` under `stage-24b/`.

### Step 6 — Gate Decision
If anonymity leaks, required disclosures are missing, or unresolved integrity markers remain, fail with `E24B`. Otherwise block for approval and then advance to stage 25.
