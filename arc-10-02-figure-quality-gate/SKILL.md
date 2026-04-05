---
name: arc-10-02-figure-quality-gate
description: Stage 27 (GATE, CRITICAL) — Validate figure readability, statistical consistency, and data traceability before final submission packaging.
metadata:
  category: pipeline-stage
  trigger-keywords: "figure quality,figure gate,stage 27"
  applicable-stages: "27"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Ensure all figures meet publication-quality standards and remain traceable to real experiment outputs. This is a **critical gate**.

---

## Quality Contract

`figure_quality_report.json` MUST:
- Evaluate every figure in `charts/`
- Record readability checks (labels, units, legend, caption linkage)
- Record statistical consistency checks against `experiment_summary.json`
- Record data traceability to generation artifacts

Hard constraints:
- `failed_critical_checks == 0`
- `missing_traceability == 0`
- `gate_pass: true`

---

## Inputs
`artifacts/<run_id>/stage-22/charts/`
`artifacts/<run_id>/stage-25/main.tex`
`artifacts/<run_id>/stage-25/paper_polished.md`
`artifacts/<run_id>/stage-14/experiment_summary.json`
`artifacts/<run_id>/stage-25/deliverables/`

---

## Outputs

### `figure_quality_report.json`
```json
{
  "total_figures": 6,
  "failed_critical_checks": 0,
  "missing_traceability": 0,
  "gate_pass": true,
  "figures": [
    {
      "figure_file": "charts/fig1.pdf",
      "readability": {
        "axes_labeled": true,
        "units_present": true,
        "legend_clear": true,
        "caption_linked": true
      },
      "statistical_consistency": {
        "matches_summary": true,
        "error_bars_present": true
      },
      "traceability": {
        "source_artifact": "stage-14/experiment_summary.json",
        "traceable": true
      },
      "critical_failures": []
    }
  ]
}
```

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `gate_pass=true` | `done`, advance to stage 28 |
| `approve` and `gate_pass=true` | `done`, advance to stage 28 |
| `reject` or `gate_pass=false` | `rejected`, rollback to stage 22 and invalidate stages 23–25 |

---

## Rollback Contract
On `reject` or `gate_pass=false`: rollback target is **stage 22** (`arc-08-03-export-publish`), and stages 23–25 MUST be invalidated and re-run before returning to trust gates.

---

## Procedure

### Step 1 — Enumerate Figures
List all figure files in `stage-22/charts/`.

### Step 2 — Readability Checks
For each figure, verify labels, units, legend clarity, and caption linkage in `main.tex`/`paper_polished.md`.

### Step 3 — Statistical Consistency Checks
Cross-check key values and error-bar presence against `experiment_summary.json`.

### Step 4 — Traceability Checks
Verify each figure maps to a declared source artifact in `manifest.json` and experiment outputs.

### Step 5 — Write Report
Write `figure_quality_report.json` with per-figure results and aggregate pass/fail fields.

### Step 6 — Gate Decision
If any critical check fails or traceability is missing, set `gate_pass=false` and fail with `E27`.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Report exists | `figure_quality_report.json` exists and is valid JSON |
| Coverage complete | Every figure in `charts/` has a report entry |
| Readability passed | All critical readability checks pass |
| Stats consistent | Figures align with `experiment_summary.json` |
| Traceable figures | Every figure has source artifact mapping |
| Gate pass flag | `gate_pass == true` |

**Failure**: Any failed check → `E27`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E27` | Figure quality gate failed |
| `E27_READABILITY_FAIL` | Missing labels/units/legend/caption linkage |
| `E27_STATS_MISMATCH` | Figure data inconsistent with experiment summary |
| `E27_TRACEABILITY_FAIL` | Figure cannot be traced to source artifacts |

---

## Retry Policy
Max retries: **1** — retry after figure regeneration or metadata repair.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`
