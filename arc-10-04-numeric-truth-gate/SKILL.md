---
name: arc-10-04-numeric-truth-gate
description: Stage 26 pre-gate (CRITICAL) — Verify that quantitative claims in the paper are traceable to experiment outputs before claim-evidence gating.
metadata:
  category: pipeline-stage
  trigger-keywords: "numeric truth,numeric verification,anti fabrication,stage 26 pre-gate"
  applicable-stages: "26"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Block the pipeline if quantitative claims in `paper_polished.md`/`main.tex` are numerically inconsistent with experiment artifacts. This is a **critical pre-gate** for stage 26.

---

## Quality Contract

`numeric_truth_report.json` MUST:
- Enumerate quantitative claims with stable `metric_id`
- Trace each metric to explicit experiment artifacts
- Classify each metric as `matched`, `mismatch`, or `unverifiable`
- Record tolerance policy used for comparison

Hard constraints:
- `mismatch == 0`
- `unverifiable == 0`
- `numeric_truth_gate_pass: true`

---

## Inputs
`artifacts/<run_id>/stage-25/paper_polished.md`
`artifacts/<run_id>/stage-25/main.tex`
`artifacts/<run_id>/stage-14/experiment_summary.json`
`artifacts/<run_id>/stage-25/compile_report.json`

---

## Outputs

### `artifacts/<run_id>/stage-26-pre/numeric_truth_report.json`
```json
{
  "total_metrics": 24,
  "matched": 24,
  "mismatch": 0,
  "unverifiable": 0,
  "tolerance_policy": {
    "relative_tolerance": 0.001,
    "absolute_tolerance": 0.000001,
    "rounding_rule": "round-half-up to reported precision"
  },
  "numeric_truth_gate_pass": true,
  "metrics": [
    {
      "metric_id": "M01",
      "claim_text": "Our method achieves 82.3% macro-F1.",
      "paper_location": "results.main",
      "reported_value": 82.3,
      "source_artifact": "stage-14/experiment_summary.json",
      "source_pointer": "primary_metrics.h1_main.mean",
      "source_value": 0.823,
      "comparison_mode": "percent_transform",
      "status": "matched",
      "notes": "0.823 -> 82.3%"
    }
  ],
  "mismatch_metrics": [],
  "unverifiable_metrics": []
}
```

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `numeric_truth_gate_pass=true` | `done`, continue to stage 26 main gate (`arc-10-01`) |
| `approve` and `numeric_truth_gate_pass=true` | `done`, continue to stage 26 main gate (`arc-10-01`) |
| `reject` or `numeric_truth_gate_pass=false` | `rejected`, rollback to stage 19 |

---

## Rollback Contract
On `reject` or `numeric_truth_gate_pass=false`: rollback target is **stage 19** (`arc-07-04-paper-revision`).

---

## Procedure

### Step 1 — Extract Quantitative Claims
Extract numeric performance claims from `paper_polished.md` and `main.tex` (results statements, table/figure summaries, and quantitative conclusions).

### Step 2 — Load Experiment Metrics
Parse `experiment_summary.json` and construct a metric registry with explicit pointers.

### Step 3 — Match Claims to Registry
For each claim, attempt direct or deterministic transformed match:
- direct value match
- percentage transform (`x` ↔ `x*100`)
- delta/improvement claims against baseline values

### Step 4 — Apply Tolerance Rules
Compare using:
- `relative_tolerance = 1e-3` for non-zero values
- `absolute_tolerance = 1e-6` for near-zero values
Allow rounding to reported precision but disallow direction-changing mismatches.

### Step 5 — Classify Metric Status
Classify each metric as:
- `matched`
- `mismatch`
- `unverifiable`

### Step 6 — Write Report
Write `artifacts/<run_id>/stage-26-pre/numeric_truth_report.json` with aggregate counts, per-metric trace data, and tolerance policy.

### Step 7 — Pre-gate Decision
If `mismatch > 0` or `unverifiable > 0`, set `numeric_truth_gate_pass=false` and fail with `E26_NUMERIC_MISMATCH` or `E26_NUMERIC_UNVERIFIABLE`.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Report exists | `artifacts/<run_id>/stage-26-pre/numeric_truth_report.json` exists and valid JSON |
| Metric coverage | Every quantitative claim has a metric record |
| Traceability | Every metric has `source_artifact` and `source_pointer` |
| No mismatch | `mismatch == 0` |
| No unverifiable | `unverifiable == 0` |
| Gate pass flag | `numeric_truth_gate_pass == true` |

**Failure**: Any failed check → `E26_NUMERIC_MISMATCH` or `E26_NUMERIC_UNVERIFIABLE`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E26_NUMERIC_MISMATCH` | At least one quantitative claim mismatches experiment metrics |
| `E26_NUMERIC_UNVERIFIABLE` | At least one quantitative claim cannot be traced to experiment artifacts |
| `E26_NUMERIC_ROUNDING_POLICY_VIOLATION` | Rounding/formatting changes alter claim direction or semantic meaning |

---

## Retry Policy
Max retries: **0** for content mismatches; **1** only for transient parse/read failures.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`
