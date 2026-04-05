---
name: arc-10-03-submission-format-gate
description: Stage 28 (GATE, CRITICAL) — Final submission gate for compile cleanliness, venue format compliance, and reproducibility-ready package completeness.
metadata:
  category: pipeline-stage
  trigger-keywords: "submission format,final gate,stage 28"
  applicable-stages: "28"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Block final output unless the package is submission-ready: clean compile, valid formatting, complete artifacts, and an explicit reproducibility artifact with semantic integrity. This is a **critical final gate**.

---

## Quality Contract

`submission_format_report.json` MUST confirm:
- Clean compile status from `compile_report.json`
- Required submission files are present
- Venue format checks pass (template + page/structure constraints)
- `reproducibility_report.json` is present and semantically complete in the final package

`submission_bundle_manifest.json` MUST:
- Enumerate required bundle files with paths
- Include checksum fields for core artifacts
- Mark package status `ready: true`

Hard constraints:
- `gate_pass: true`
- `ready: true`

---

## Inputs
`artifacts/<run_id>/stage-25/main.tex`
`artifacts/<run_id>/stage-25/main.pdf`
`artifacts/<run_id>/stage-25/references.bib`
`artifacts/<run_id>/stage-25/compile_report.json`
`artifacts/<run_id>/stage-25/manifest.json`
`artifacts/<run_id>/stage-25/reproducibility_report.json`
`artifacts/<run_id>/stage-25/deliverables/`

---

## Outputs

### `artifacts/<run_id>/stage-28/submission_format_report.json`
```json
{
  "compile_clean": true,
  "template_compliant": true,
  "package_complete": true,
  "reproducibility_minimum_present": true,
  "reproducibility_semantic_integrity": true,
  "blocking_issues": [],
  "gate_pass": true
}
```

### `artifacts/<run_id>/stage-28/submission_bundle_manifest.json`
```json
{
  "required_files": [
    "main.tex",
    "main.pdf",
    "references.bib",
    "paper_polished.md",
    "compile_report.json",
    "manifest.json",
    "reproducibility_report.json",
    "code/",
    "charts/"
  ],
  "checksums": {
    "main.tex": "sha256:...",
    "main.pdf": "sha256:...",
    "references.bib": "sha256:..."
  },
  "ready": true,
  "source_stage": 25
}
```

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `gate_pass=true` | `done`, pipeline complete |
| `approve` and `gate_pass=true` | `done`, pipeline complete |
| `reject` or `gate_pass=false` | `rejected`, rollback to stage 22 and invalidate stages 23–25 |

---

## Rollback Contract
On `reject` or `gate_pass=false`: rollback target is **stage 22** (`arc-08-03-export-publish`), and stages 23–25 MUST be invalidated and re-run before returning to trust gates.

---

## Procedure

### Step 1 — Compile Cleanliness Check
Read `artifacts/<run_id>/stage-25/compile_report.json` and verify final exit code is 0 with no undefined refs/cites.

### Step 2 — Template/Format Check
Check that `artifacts/<run_id>/stage-25/main.tex` and the polished package use the intended venue template and that structure/page constraints are satisfied.

### Step 3 — Package Completeness Check
Verify `artifacts/<run_id>/stage-25/deliverables/` contains required files (`main.tex`, `main.pdf`, `references.bib`, `paper_polished.md`, `code/`, `charts/`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`).

### Step 4 — Reproducibility Minimum Check
Read `artifacts/<run_id>/stage-25/reproducibility_report.json` and verify the polished package includes explicit reproducibility metadata:
- execution command summary
- dependency/environment summary
- seed summary
- provenance fields consistent with the stage-22 export

### Step 5 — Write Reports
Write `artifacts/<run_id>/stage-28/submission_format_report.json` and `artifacts/<run_id>/stage-28/submission_bundle_manifest.json` against the polished stage-25 package.

### Step 6 — Gate Decision
If any compile/format/package/reproducibility check fails, set `gate_pass=false` and fail with `E28`.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Format report valid | `artifacts/<run_id>/stage-28/submission_format_report.json` exists and valid JSON |
| Bundle manifest valid | `artifacts/<run_id>/stage-28/submission_bundle_manifest.json` exists and valid JSON |
| Compile clean | `compile_clean == true` |
| Template compliant | `template_compliant == true` |
| Package complete | `package_complete == true` |
| Reproducibility minimum | `reproducibility_minimum_present == true` |
| Reproducibility semantics preserved | `reproducibility_semantic_integrity == true` |
| Manifest references polished package | `source_stage == 25` and required files match the polished package |
| Gate pass flag | `gate_pass == true` and `ready == true` |

**Failure**: Any failed check → `E28`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E28` | Stage-28 submission gate found compile, format, package, or reproducibility noncompliance |
| `E28_TEMPLATE_NONCOMPLIANT` | The polished package violates template, page, or structure requirements |
| `E28_PACKAGE_INCOMPLETE` | Required files are missing from the polished submission bundle |
| `E28_COMPILE_FAIL` | `compile_report.json` indicates unresolved compile errors or references |
| `E28_REPRODUCIBILITY_FAIL` | `reproducibility_report.json` is missing or fails semantic integrity checks |

---

## Retry Policy
Max retries: **2** — retry after format/package/compile fixes.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`
