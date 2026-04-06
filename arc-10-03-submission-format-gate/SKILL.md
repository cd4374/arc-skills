---
name: arc-10-03-submission-format-gate
description: Stage 28 (GATE, CRITICAL) — Final submission gate for compile cleanliness, schema-valid venue compliance, hard format checks, figure/citation trust checks, and reproducibility-ready package completeness.
metadata:
  category: pipeline-stage
  trigger-keywords: "submission format,final gate,stage 28"
  applicable-stages: "28"
  priority: "1"
  version: "2.0"
  author: researchclaw
---

## Purpose
Block final output unless the package is submission-ready: clean compile, schema-valid template resolution, venue-aware page/structure/anonymity/front-matter compliance, figure-gate consistency including authenticity, citation-threshold compliance, complete artifacts, and an explicit reproducibility artifact with semantic integrity. This is a **critical final gate**.

---

## Quality Contract

`submission_format_report.json` MUST confirm:
- Clean compile status from `compile_report.json`
- `template_manifest.json` is internally consistent with the template contract declared by the skills chain
- The submission report itself is internally consistent and complete
- Required submission files are present
- Venue format checks pass (template + page + anonymity + front-matter + figure consistency constraints)
- Citation thresholds pass (or justified topic constraints are recorded)
- `reproducibility_report.json` is present and semantically complete in the final package
- `stage-26-pre/numeric_truth_report.json` still exists with `numeric_truth_gate_pass == true`
- Stage-27 figure results still correspond to the canonical polished package rather than an older export snapshot
- Final figure provenance sidecars are present and consistent with both `figure_quality_report.json` and the polished package manifest

`submission_bundle_manifest.json` MUST:
- Enumerate required bundle files with paths
- Include checksum fields for core artifacts
- Mark package status `ready: true`

Hard constraints:
- `compile_clean == true`
- `template_compliant == true`
- `package_complete == true`
- `reproducibility_minimum_present == true`
- `reproducibility_semantic_integrity == true`
- `gate_pass == true`
- `submission_bundle_manifest.json` records `ready == true`

---

## Inputs
`artifacts/<run_id>/stage-26-pre/numeric_truth_report.json`
`artifacts/<run_id>/stage-25/main.tex`
`artifacts/<run_id>/stage-25/main.pdf`
`artifacts/<run_id>/stage-25/references.bib`
`artifacts/<run_id>/stage-25/compile_report.json`
`artifacts/<run_id>/stage-25/manifest.json`
`artifacts/<run_id>/stage-25/reproducibility_report.json`
`artifacts/<run_id>/stage-25/template_manifest.json`
`artifacts/<run_id>/stage-27/figure_quality_report.json`
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
  "resolved_template_id": "neurips_2025",
  "publication_type": "conference",
  "schema_validation": {
    "template_manifest_schema_pass": true,
    "submission_report_schema_pass": true
  },
  "page_checks": {
    "page_limit_policy": "main_body_only",
    "main_body_pages": 9,
    "page_limit_main_body": 9,
    "page_check_pass": true
  },
  "anonymity_checks": {
    "required": true,
    "author_block_present": false,
    "acknowledgement_hidden": true,
    "anonymity_pass": true
  },
  "front_matter_checks": {
    "keywords_present": null,
    "title_present": true,
    "abstract_present": true,
    "front_matter_pass": true
  },
  "figure_checks": {
    "figure_count": 6,
    "figure_gate_pass": true,
    "authenticity_pass": true
  },
  "citation_checks": {
    "reference_count": 34,
    "reference_count_min": 30,
    "reference_count_pass": true,
    "recent_reference_ratio": 0.26,
    "recent_reference_ratio_min": 0.20,
    "recent_reference_ratio_pass": true,
    "topic_constraint_justified": false
  },
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
    "template_manifest.json",
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
| `auto_approve_gates=true` and `gate_pass=true` | `done`, final late-stage acceptance complete |
| `approve` and `gate_pass=true` | `done`, final late-stage acceptance complete |
| `reject` or `gate_pass=false` | `rejected`, rollback to stage 22 and invalidate stages 23–25 |

---

## Rollback Contract
On `reject` or `gate_pass=false`: rollback target is **stage 22** (`arc-08-03-export-publish`), and stages 23–25 MUST be invalidated and re-run before returning to the remaining late-stage gates.

---

## Procedure

### Step 1 — Numeric-Truth Integrity Check
Read `artifacts/<run_id>/stage-26-pre/numeric_truth_report.json`. If it is missing or `numeric_truth_gate_pass != true`, fail immediately rather than treating the package as late-stage ready.

### Step 2 — Compile Cleanliness Check
Read `artifacts/<run_id>/stage-25/compile_report.json` and verify final exit code is 0 with no undefined refs/cites.

### Step 3 — Contract Consistency Check
Read `artifacts/<run_id>/stage-25/template_manifest.json`. Verify the resolved template manifest is internally consistent and complete, then prepare the stage-28 report so it is likewise internally consistent and complete.

### Step 4 — Template and Page-Format Check
Check that the manifest's `resolved_template_id`/`template_id` is supported by the skills chain's declared families, that the manifest records a real external template-asset contract for the execution workspace, and that `artifacts/<run_id>/stage-25/main.tex` satisfies the resolved page-limit policy and page cap where applicable.

### Step 5 — Anonymity and Front-Matter Check
Using the resolved publication target and validation rules, verify `artifacts/<run_id>/stage-25/main.tex` has compliant author handling, acknowledgement visibility, title/abstract presence, and venue-specific front matter such as keywords for journal targets.

### Step 6 — Figure Consistency Check
Read `artifacts/<run_id>/stage-27/figure_quality_report.json` and verify the figure gate passed, the recorded figure count is consistent with `artifacts/<run_id>/stage-25/deliverables/charts/`, and no unresolved figure blocking issues remain. Treat figure authenticity as failed if the figure package is not traceable to real experiment outputs, if any required provenance sidecar is missing or inconsistent, or if stage-27 results appear to have been computed against an older export snapshot.

### Step 7 — Citation Threshold Check
Inspect the polished bibliography and verify the final package reaches at least 30 verified references and at least 20% recent references from the last 5 years, unless topic constraints are explicitly documented. Record both raw counts/ratios and pass/fail fields in `citation_checks`.

### Step 8 — Package Completeness Check
Verify `artifacts/<run_id>/stage-25/deliverables/` contains required files (`main.tex`, `main.pdf`, `references.bib`, `paper_polished.md`, `code/`, `charts/`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, `template_manifest.json`).

### Step 9 — Reproducibility Minimum Check
Read `artifacts/<run_id>/stage-25/reproducibility_report.json` and verify the polished package includes explicit reproducibility metadata:
- execution command summary
- dependency/environment summary
- seed summary
- provenance fields consistent with the stage-22 export

### Step 10 — Write Reports
Write `artifacts/<run_id>/stage-28/submission_format_report.json` and `artifacts/<run_id>/stage-28/submission_bundle_manifest.json` against the polished stage-25 package, recording contract-consistency results, the resolved template ID, publication type, numeric-truth integrity, page/anonymity/front-matter/figure/citation checks, and any blocking issues.

### Step 11 — Gate Decision
If any numeric-truth, compile, schema, format, page, anonymity, front-matter, figure, citation, package, or reproducibility check fails, set `gate_pass=false` and fail with `E28`. Rejecting this gate rolls back to stage 22 so stages 23–25 can be re-run against a refreshed export.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Format report valid | `artifacts/<run_id>/stage-28/submission_format_report.json` exists and valid JSON |
| Bundle manifest valid | `artifacts/<run_id>/stage-28/submission_bundle_manifest.json` exists and valid JSON |
| Template manifest schema valid | `schema_validation.template_manifest_schema_pass == true` |
| Submission report schema valid | `schema_validation.submission_report_schema_pass == true` |
| Numeric pre-gate intact | `stage-26-pre/numeric_truth_report.json` exists and `numeric_truth_gate_pass == true` |
| Compile clean | `compile_clean == true` |
| Template compliant | `template_compliant == true` and `resolved_template_id` is present |
| Page checks passed | `page_checks.page_check_pass == true` |
| Anonymity checks passed | `anonymity_checks.anonymity_pass == true` when anonymity is required |
| Front matter checks passed | `front_matter_checks.front_matter_pass == true` |
| Figure checks passed | `figure_checks.figure_gate_pass == true` |
| Figure authenticity passed | `figure_checks.authenticity_pass == true` |
| Citation checks passed | Citation thresholds pass or a justified topic constraint is recorded |
| Package complete | `package_complete == true` |
| Reproducibility minimum | `reproducibility_minimum_present == true` |
| Reproducibility semantics preserved | `reproducibility_semantic_integrity == true` |
| Manifest references polished package | `source_stage == 25` and required files match the polished package |
| Gate pass flag | `gate_pass == true` in `submission_format_report.json` and `ready == true` in `submission_bundle_manifest.json` |

**Failure**: Any failed check → `E28`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E28` | Stage-28 submission gate found numeric-truth, compile, schema, format, package, citation-threshold, figure-authenticity, or reproducibility noncompliance |
| `E28_SCHEMA_FAIL` | `template_manifest.json` or `submission_format_report.json` violates its schema contract |
| `E28_TEMPLATE_NONCOMPLIANT` | The polished package violates template, page, anonymity, front-matter, or structure requirements |
| `E28_NUMERIC_PRECHECK_FAIL` | `stage-26-pre/numeric_truth_report.json` is missing or no longer passing |
| `E28_FIGURE_GATE_FAIL` | Stage-27 figure quality gate failed or is inconsistent with the polished package |
| `E28_CITATION_THRESHOLD_FAIL` | Final bibliography fails the reference-count or recent-reference-ratio threshold without justified topic constraints |
| `E28_PACKAGE_INCOMPLETE` | Required files are missing from the polished submission bundle |
| `E28_COMPILE_FAIL` | `compile_report.json` indicates unresolved compile errors or references |
| `E28_REPRODUCIBILITY_FAIL` | `reproducibility_report.json` is missing or fails semantic integrity checks |

---

## Retry Policy
Max retries: **2** — retry after format/package/compile fixes.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`
