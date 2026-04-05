---
name: arc-10-01-claim-evidence-trace-gate
description: Stage 26 (GATE, CRITICAL) — Build a machine-checkable claim-evidence matrix and block unsupported or untraceable claims before figure/format finalization.
metadata:
  category: pipeline-stage
  trigger-keywords: "claim evidence,traceability,stage 26"
  applicable-stages: "26"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Guarantee that every major claim in the paper is traceable to explicit evidence (figure/table/experiment metric/citation). This is a **critical gate**.

---

## Quality Contract

`claim_evidence_matrix.json` MUST:
- Enumerate every major claim with stable `claim_id`
- Map each claim to at least one explicit evidence pointer
- Classify each claim as `supported`, `partial`, or `unsupported`
- Mark any untraceable claim with a blocking reason

Hard constraints:
- `unsupported_claims.length == 0`
- `untraceable_claims.length == 0`
- `gate_pass: true`

---

## Inputs
`artifacts/<run_id>/stage-26-pre/numeric_truth_report.json`
`artifacts/<run_id>/stage-25/paper_polished.md`
`artifacts/<run_id>/stage-14/experiment_summary.json`
`artifacts/<run_id>/stage-25/deliverables/`
`artifacts/<run_id>/stage-25/compile_report.json`

---

## Outputs

### `claim_evidence_matrix.json`
```json
{
  "total_claims": 14,
  "supported": 12,
  "partial": 2,
  "unsupported": 0,
  "untraceable": 0,
  "gate_pass": true,
  "claims": [
    {
      "claim_id": "C01",
      "claim_text": "Method X improves macro-F1 over baseline on dataset D.",
      "paper_location": "results.section.2",
      "status": "supported",
      "evidence": [
        {
          "type": "table",
          "artifact": "stage-14/results_table.tex",
          "pointer": "row=method_x,col=macro_f1"
        },
        {
          "type": "figure",
          "artifact": "stage-22/charts/fig1.pdf",
          "pointer": "caption: main comparison"
        }
      ]
    }
  ],
  "unsupported_claims": [],
  "untraceable_claims": []
}
```

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `gate_pass=true` | `done`, advance to stage 27 |
| `approve` and `gate_pass=true` | `done`, advance to stage 27 |
| `reject` or `gate_pass=false` | `rejected`, rollback to stage 19 |

---

## Rollback Contract
On `reject` or `gate_pass=false`: rollback target is **stage 19** (`arc-07-04-paper-revision`).

---

## Procedure

### Step 0 — Numeric Truth Precheck
Read `artifacts/<run_id>/stage-26-pre/numeric_truth_report.json`. If the file is missing or `numeric_truth_gate_pass != true`, fail with `E26_NUMERIC_PRECHECK_FAILED` and do not continue.

### Step 1 — Extract Major Claims
Read `paper_polished.md` and extract major result/method claims.

### Step 2 — Build Evidence Links
For each claim, attach explicit evidence pointers from `experiment_summary.json` and the polished package artifacts in `stage-25/`.

### Step 3 — Classify Claim Status
Assign one status per claim:
- `supported`: explicit, sufficient evidence
- `partial`: some evidence but incomplete
- `unsupported`: no valid evidence

### Step 4 — Detect Untraceable Claims
Mark claims as untraceable if no concrete pointer exists to table/figure/metric/citation artifacts.

### Step 5 — Write Matrix
Write `claim_evidence_matrix.json` using the schema above.

### Step 6 — Gate Decision
If any `unsupported` or `untraceable` claims exist, set `gate_pass=false` and fail with `E26`.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Numeric precheck passed | `numeric_truth_report.json` exists and `numeric_truth_gate_pass == true` |
| Matrix exists | `claim_evidence_matrix.json` present and valid JSON |
| Claims enumerated | Every major claim appears with a `claim_id` |
| Evidence pointers valid | Each claim has ≥1 explicit artifact pointer |
| No unsupported claims | `unsupported == 0` |
| No untraceable claims | `untraceable == 0` |
| Gate pass flag | `gate_pass == true` |

**Failure**: Any failed check → `E26`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E26` | Unsupported or untraceable claims detected |
| `E26_NUMERIC_PRECHECK_FAILED` | Numeric pre-gate report missing or `numeric_truth_gate_pass != true` |
| `E26_TRACE_MISSING` | Claim exists without explicit evidence pointer |
| `E26_CLAIM_UNSUPPORTED` | Claim contradicted or not supported by artifacts |

---

## Retry Policy
Max retries: **0** for content failures. Retry only on transient file/parse issues.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`
