---
name: arc-10-02-figure-quality-gate
description: Stage 27 (GATE, CRITICAL) — Validate hard figure-count, readability, style, statistical consistency, data traceability, and figure authenticity requirements before final submission packaging.
metadata:
  category: pipeline-stage
  trigger-keywords: "figure quality,figure gate,stage 27"
  applicable-stages: "27"
  priority: "1"
  version: "2.0"
  author: researchclaw
---

## Purpose
Ensure the final paper satisfies venue-aware hard standards for figure quantity, readability, visual style, file quality, statistical consistency, traceability to real experiment outputs, and figure authenticity. This is a **critical gate**.

---

## Quality Contract

`figure_quality_report.json` MUST:
- Evaluate every figure in `charts/`
- Resolve venue-specific figure rules from `artifacts/<run_id>/stage-25/template_manifest.json`
- Record quantity checks against the resolved template's `required_figure_count_min` and `recommended_figure_count_range`
- Record readability checks (labels, units, legend, caption linkage, font legibility)
- Record visual-style checks (subfigure labels, color consistency, vector/raster suitability)
- Record statistical consistency checks against `experiment_summary.json`
- Record data traceability to generation artifacts
- Record figure authenticity checks proving each figure is evidence-bearing rather than fabricated or decorative

Hard constraints:
- `required_figure_count_pass == true`
- `failed_critical_checks == 0`
- `missing_traceability == 0`
- `authenticity_pass == true`
- `gate_pass == true`

---

## Inputs
`artifacts/<run_id>/stage-25/deliverables/charts/`
`artifacts/<run_id>/stage-25/main.tex`
`artifacts/<run_id>/stage-25/paper_polished.md`
`artifacts/<run_id>/stage-14/experiment_summary.json`
`artifacts/<run_id>/stage-25/template_manifest.json`
`artifacts/<run_id>/stage-25/deliverables/`
`artifacts/<run_id>/stage-25/manifest.json`

---

## Outputs

### `figure_quality_report.json`
```json
{
  "resolved_template_id": "neurips_2025",
  "required_figure_count_min": 4,
  "recommended_figure_count_range": [4, 8],
  "total_figures": 6,
  "required_figure_count_pass": true,
  "failed_critical_checks": 0,
  "missing_traceability": 0,
  "authenticity_pass": true,
  "gate_pass": true,
  "figures": [
    {
      "figure_file": "charts/fig1.pdf",
      "readability": {
        "axes_labeled": true,
        "units_present": true,
        "legend_clear": true,
        "caption_linked": true,
        "font_readable_min_pt_pass": true
      },
      "style": {
        "subfigure_label_style_pass": true,
        "color_consistent": true,
        "vector_or_high_dpi_pass": true
      },
      "statistical_consistency": {
        "matches_summary": true,
        "error_bars_present": true
      },
      "traceability": {
        "source_artifact": "stage-14/experiment_summary.json",
        "traceable": true
      },
      "authenticity": {
        "sidecar_present": true,
        "metric_found_in_summary": true,
        "visual_stats_consistent": true,
        "evidence_bearing": true,
        "decorative_only": false,
        "derived_from_experiment_artifacts": true
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
On `reject` or `gate_pass=false`: rollback target is **stage 22** (`arc-08-03-export-publish`), and stages 23–25 MUST be invalidated and re-run before returning to the remaining late-stage gates.

---

## Procedure

### Step 1 — Resolve Figure Rules
Read `artifacts/<run_id>/stage-25/template_manifest.json`. Resolve the effective figure policy for the selected template, including `required_figure_count_min`, `recommended_figure_count_range`, and any `figure_style_rules`.

### Step 2 — Enumerate Figures
List all figure files in `stage-25/deliverables/charts/` and cross-reference them with figure environments and captions in `stage-25/main.tex` and `stage-25/paper_polished.md`. If `stage-25/deliverables/charts/` is missing or diverges from the polished package manifest, fail rather than silently checking an older stage-22 figure set.

### Step 3 — Quantity Checks
Verify the polished package meets the resolved minimum figure count. If the template declares a recommended range, record whether the total falls inside or outside that range without downgrading a package that still meets the hard minimum.

### Step 4 — Readability and Style Checks
For each figure, verify labels, units, legend clarity, caption linkage, readable typography, subfigure-label conventions, color consistency, and vector or sufficiently high-DPI raster quality according to the resolved template rules.

### Step 5 — Statistical Consistency Checks
Cross-check key values and error-bar presence against `experiment_summary.json`.

### Step 6 — Traceability Checks
Verify each figure maps to a declared source artifact in `manifest.json` and experiment outputs.

### Step 7 — Authenticity Checks
For each figure, run all three authenticity sub-checks:
1. **Sidecar check** — locate `<figure>.provenance.json`; if missing, fail authenticity immediately
2. **Metric cross-check** — verify every declared metric key in the sidecar exists in `experiment_summary.json`; if not, fail authenticity
3. **Visual consistency check** — verify the plotted values/error bars are consistent with the declared experiment summary values; if materially inconsistent, fail with `E27_STATS_MISMATCH`

Only after those checks pass may the figure be marked evidence-bearing, non-decorative, and derived from real experiment artifacts.

### Step 8 — Write Report
Write `figure_quality_report.json` with resolved template metadata, aggregate quantity/style/authenticity pass fields, per-figure results, and blocking failures.

### Step 9 — Gate Decision
If the required figure count fails, any critical quality/style check fails, traceability is missing, or authenticity fails, set `gate_pass=false` and fail with `E27`. Rejecting this gate rolls back to stage 22 so figures can be regenerated and the late-stage package refreshed.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Report exists | `figure_quality_report.json` exists and is valid JSON |
| Template resolved | Report includes `resolved_template_id` and resolved figure rules |
| Coverage complete | Every figure in `stage-25/deliverables/charts/` has a report entry |
| Figure count passed | `required_figure_count_pass == true` |
| Readability passed | All critical readability checks pass |
| Style passed | All critical style/file-quality checks pass |
| Stats consistent | Figures align with `experiment_summary.json` |
| Traceable figures | Every figure has source artifact mapping |
| Authentic figures | `authenticity_pass == true` and figures are evidence-bearing |
| Gate pass flag | `gate_pass == true` |

**Failure**: Any failed check → `E27`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E27` | Figure quality gate failed |
| `E27_COUNT_FAIL` | Required minimum figure count is not met for the resolved template |
| `E27_READABILITY_FAIL` | Missing labels, units, captions, or readable typography |
| `E27_STYLE_FAIL` | Figure style, subfigure labeling, color consistency, or file-quality rules are violated |
| `E27_STATS_MISMATCH` | Figure data inconsistent with experiment summary |
| `E27_TRACEABILITY_FAIL` | Figure cannot be traced to source artifacts |
| `E27_AUTHENTICITY_FAIL` | Figure is not evidence-bearing or cannot be shown to derive from real experiment outputs |

---

## Retry Policy
Max retries: **1** — retry after figure regeneration or metadata repair.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`
