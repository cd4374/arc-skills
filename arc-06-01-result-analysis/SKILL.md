---
name: arc-06-01-result-analysis
description: Stage 14 — Analyze experimental results with rigorous data validation and produce statistical conclusions for each hypothesis. Detects anomalies, validates data quality, and produces evidence-based conclusions.
metadata:
  category: pipeline-stage
  trigger-keywords: "result analysis,metrics,statistics,stage 14"
  applicable-stages: "14"
  priority: "1"
  version: "4.0"
  author: researchclaw
---

## Purpose

Transform raw experiment metrics into evidence-based conclusions about each hypothesis. This stage includes DATA VALIDATION to detect anomalies and ensure result integrity before analysis.

---

## Quality Contract

`analysis.md` MUST satisfy:
- Every hypothesis (H1, H2, ...) has a SUPPORTED / NOT_SUPPORTED / INCONCLUSIVE determination with explicit evidence
- `results_table.tex` is a valid LaTeX table with mean ± std for every condition
- Statistical claims are grounded in actual metric values from `experiment_summary.json`
- Ablation effects are quantified (not just described qualitatively)
- **Data validation passed** — no NaN, Inf, or anomalous patterns detected

---

## Inputs
`artifacts/<run_id>/stage-12/experiment_summary.json`
`artifacts/<run_id>/stage-13/refinement_log.json`
`artifacts/<run_id>/stage-13/diagnosis_report.json`
`artifacts/<run_id>/stage-13/experiment_final/`
`artifacts/<run_id>/stage-08/hypotheses.md`

---

## Outputs

### `data_validation.json`
```json
{
  "valid": true,
  "checks": {
    "no_nan_inf": {"passed": true, "details": ""},
    "sufficient_variance": {"passed": true, "details": "All conditions show seed-level variance"},
    "no_identical_conditions": {"passed": true, "details": ""},
    "reasonable_magnitude": {"passed": true, "details": "All metrics within expected ranges"},
    "baseline_sanity": {"passed": true, "details": "Baselines perform above random chance"}
  },
  "warnings": [],
  "anomalies": []
}
```

### `analysis.md`
```
# Experimental Analysis

## Data Validation Summary
[Pass/fail status for each validation check]

## Summary of Findings
[1-2 paragraph overview]

## Statistical Analysis
[Per-hypothesis analysis with actual metric values]

## Hypothesis Support Assessment
- H1: SUPPORTED — [evidence with numbers]
- H2: NOT_SUPPORTED — [evidence]
- H3: INCONCLUSIVE — [reason]

## Ablation Results
[Effect size for each ablation]

## Unexpected Findings
[Any surprising results not predicted by hypotheses]
```

### `results_table.tex`
Valid LaTeX table with `hline`, column headers, and one row per condition.

### `experiment_summary.json`
Promoted to `stage-14/` as the canonical summary for paper stages.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Data validation passed | `data_validation.json` shows `valid: true` |
| All hypotheses assessed | H1, H2, ... each have SUPPORTED/NOT_SUPPORTED/INCONCLUSIVE |
| Evidence is quantitative | Claims reference actual numbers from experiment_summary |
| `results_table.tex` valid | Valid LaTeX table syntax |
| Ablation effects quantified | Each ablation has a metric delta |

**Failure**: 
- If data validation fails with CRITICAL issues → `E14_DATA_INVALID`
- If any hypothesis lacks an assessment → `E14`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E14` | Missing hypothesis assessment, or results_table.tex invalid |
| `E14_DATA_INVALID` | Data validation detected critical anomalies |
| `E14_NO_EVIDENCE` | Evidence is purely qualitative without numbers |

---

## Retry Policy
Max retries: **1** — retry if analysis is purely qualitative.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read All Experiment Data

Read `experiment_summary.json`, `refinement_log.json`, `diagnosis_report.json`, `experiment_final/`, and `hypotheses.md`.

### Step 2 — Data Validation

Run comprehensive data validation checks:

**Check 2.1: No NaN/Inf**
```python
for metric in all_metrics:
    if isnan(metric) or isinf(metric):
        fail("NaN/Inf detected in {metric}")
```

**Check 2.2: Sufficient Variance**
```python
for condition in conditions:
    seed_values = get_seed_values(condition)
    if len(seed_values) >= 2:
        variance = std(seed_values)
        if variance == 0:
            warning("No variance across seeds for {condition} — possible bug")
```

**Check 2.3: No Identical Conditions**
```python
for cond1, cond2 in pairs(conditions):
    if mean(cond1) == mean(cond2) and std(cond1) == std(cond2):
        warning("Conditions {cond1} and {cond2} produce identical results")
```

**Check 2.4: Reasonable Magnitude**
```python
for metric, value in all_metrics:
    if metric contains "accuracy" and value < 0 or value > 1:
        fail("Accuracy {value} outside [0,1] range")
    if metric contains "loss" and value < 0:
        warning("Negative loss: {value}")
    if metric contains "loss" and value > 100:
        warning("Very high loss: {value} — possible divergence")
```

**Check 2.5: Baseline Sanity**
```python
for baseline in baselines:
    if baseline_accuracy < random_chance * 1.1:
        warning("Baseline {baseline} near random chance — possible bug")
```

**Write `data_validation.json`** with all check results.

### Step 3 — Handle Validation Failures

**If `valid: false` with CRITICAL issues:**
- Do NOT proceed to analysis
- Fail with `E14_DATA_INVALID`
- Write `DATA_ISSUE.md` describing the problem

**If `valid: true` with warnings:**
- Continue to analysis
- Note warnings in `analysis.md`

### Step 4 — Per-Hypothesis Analysis

For each hypothesis (H1, H2, ...):
1. Collect the metric values for the hypothesis condition vs. the baseline
2. Compute statistical significance (t-test if applicable)
3. Check: is `metric_mean >= success_threshold`? → SUPPORTED
4. Check: is `metric_mean < baseline_mean - 0.01`? → NOT_SUPPORTED
5. Otherwise → INCONCLUSIVE
6. Document with exact numbers: "H1 achieved {mean}±{std} vs. baseline {mean}±{std}, a {delta} improvement"

### Step 5 — Ablation Analysis

For each ablation:
1. Compare ablating component X vs. the full method
2. Quantify the effect: "Removing X changed accuracy by {delta}"
3. Compute significance if applicable
4. Note if the effect is statistically significant

### Step 6 — Unexpected Findings Analysis

Look for:
- Results that contradict the hypothesis direction
- Large variance that suggests instability
- Conditions that perform unexpectedly well or poorly
- Interactions between ablations

### Step 7 — Write analysis.md

Follow the Outputs template exactly. Include:
- Data validation summary
- All actual numbers from `experiment_summary.json`
- Every claim must cite a specific metric value

### Step 8 — Write results_table.tex

Create a valid LaTeX table:
- One row per condition (baseline, hypothesis, each ablation)
- Columns: Condition, Mean ± Std, Dataset
- Use `\pm` for the std symbol
- Wrap in `\begin{table}...\end{table}` with `\hline`

### Step 9 — Validate

Check:
- `data_validation.json` shows `valid: true` (or only warnings)
- All hypotheses have SUPPORTED/NOT_SUPPORTED/INCONCLUSIVE
- All evidence is quantitative
- `results_table.tex` is valid LaTeX

Fail → `E14`, `E14_DATA_INVALID`, or `E14_NO_EVIDENCE`.

---

## Data Validation Checks Summary

| Check | Severity on Failure |
|-------|---------------------|
| NaN/Inf in metrics | CRITICAL |
| Zero variance across seeds | WARNING |
| Identical condition results | WARNING |
| Metrics outside expected range | CRITICAL or WARNING |
| Baseline near random | WARNING |

---

## Key Rules

1. **Validate first**: Always run data validation before analysis
2. **No qualitative evidence**: Every claim must have a number
3. **Statistical rigor**: Use proper statistical tests where applicable
4. **Honest reporting**: Report anomalies and unexpected findings
5. **Reproducible**: All numbers must trace back to experiment_summary.json