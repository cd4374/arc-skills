---
name: arc-05-02-iterative-refine
description: Stage 13 — Perform edit-run-eval refinement cycles with auto-diagnosis and auto-repair. Detects experiment failures, diagnoses root causes, and applies targeted fixes.
metadata:
  category: pipeline-stage
  trigger-keywords: "iterative refine,repair,converge,stage 13"
  applicable-stages: "13"
  priority: "1"
  version: "4.0"
  author: researchclaw
---

## Purpose

Iteratively improve experiment configuration until hypotheses are supported with sufficient evidence or the refinement budget is exhausted. This stage includes AUTO-DIAGNOSIS of experiment failures and AUTO-REPAIR of common issues.

---

## Quality Contract

`experiment_final/` + `refinement_log.json` MUST satisfy:
- `refinement_log.json` documents every iteration: what changed, metrics before, metrics after
- `experiment_final/` is a complete snapshot of the best-performing experiment configuration
- Loop exits when: convergence (2 consecutive iterations with no improvement) OR budget ≥ 95% exhausted
- Final `experiment_summary.json` reflects the best configuration's metrics
- All deficiencies are diagnosed and repair attempts are logged

---

## Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `MAX_REPAIR_CYCLES` | 3 | Maximum auto-repair attempts per deficiency |
| `MIN_COMPLETION_RATE` | 0.5 | Minimum fraction of conditions that must complete |
| `CONVERGENCE_THRESHOLD` | 2 | Consecutive iterations with no improvement to exit |

---

## Inputs
`artifacts/<run_id>/stage-12/runs/`
`artifacts/<run_id>/stage-12/experiment_summary.json`
`artifacts/<run_id>/stage-12/execution_log.json`
`artifacts/<run_id>/stage-08/hypotheses.md`
`artifacts/<run_id>/stage-08/hypothesis_index.json`

---

## Outputs

### `refinement_log.json`
```json
{
  "iterations": [
    {
      "iteration": 1,
      "change_made": "[what was changed]",
      "metrics_before": {"h1_baseline": 0.78},
      "metrics_after": {"h1_baseline": 0.801},
      "improvement": true,
      "deficiencies_found": [],
      "repairs_applied": []
    }
  ],
  "best_iteration": 1,
  "converged": false,
  "budget_used_pct": 45.0,
  "total_repair_cycles": 0
}
```

### `diagnosis_report.json` (if deficiencies detected)
```json
{
  "deficiencies": [
    {
      "type": "gpu_oom",
      "severity": "major",
      "description": "GPU out of memory during training",
      "affected_conditions": ["h1_large_batch"],
      "suggested_fix": "Reduce batch_size by 50%",
      "repair_applied": true,
      "repair_result": "success"
    }
  ],
  "completion_rate": 0.6,
  "repairable": true
}
```

### `experiment_final/` directory
Complete snapshot of the best experiment configuration.

### `experiment_summary.json`
Updated summary reflecting the best configuration's results.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| `refinement_log.json` exists | File present |
| `iterations` array non-empty | At least 1 iteration was attempted |
| `experiment_final/` exists | Snapshot directory present |
| Loop termination valid | `converged: true` OR `budget_used_pct >= 95` |
| `experiment_summary.json` updated | Contains final best metrics |
| Completion rate sufficient | `completion_rate >= MIN_COMPLETION_RATE` |

**Failure**: 
- If `iterations` is empty → `E13`
- If `completion_rate < MIN_COMPLETION_RATE` after repair → `E13_LOW_COMPLETION`

---

## Error Codes

| Code | Trigger | Retryable |
|------|---------|-----------|
| `E13` | No refinement iterations completed | Yes (2 retries) |
| `E13_LOW_COMPLETION` | Completion rate below threshold after repair | Yes (1 retry) |
| `E13_REPAIR_EXHAUSTED` | Max repair cycles reached without improvement | No |

---

## Retry Policy
Max retries: **2** — retry on infrastructure failures.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Prior Outputs

Read `experiment_summary.json`, `execution_log.json`, `hypotheses.md`, `hypothesis_index.json`, and `runs/` directory.

### Step 2 — Diagnose Experiment Quality

**Run quality diagnosis:**

1. **Calculate completion rate:**
   ```
   completion_rate = completed_conditions / total_planned_conditions
   ```

2. **Check for NaN/Inf in metrics:**
   - Flag any condition with NaN or Inf values
   - Severity: CRITICAL

3. **Check for identical conditions:**
   - Compare metric distributions across conditions
   - If two conditions have identical mean AND std → flag as IDENTICAL_CONDITIONS
   - Severity: MAJOR

4. **Check for near-random performance:**
   - If accuracy-like metric < 15% (random chance) → flag as NEAR_RANDOM
   - Severity: MAJOR

5. **Check for insufficient seeds:**
   - If any condition has < 2 seeds → flag as INSUFFICIENT_SEEDS
   - Severity: MINOR

**Write `diagnosis_report.json`** with all findings.

### Step 3 — Auto-Repair (if deficiencies found)

For each CRITICAL or MAJOR deficiency:

| Deficiency Type | Auto-Fix |
|-----------------|----------|
| `gpu_oom` | Reduce batch_size by 50%, enable gradient_checkpointing |
| `nan_loss` | Reduce learning_rate by 10x, add gradient_clipping (max_norm=1.0) |
| `missing_dependency` | Add package to requirements.txt |
| `identical_conditions` | Verify condition parameter is wired into model/training |
| `near_random` | Check learning_rate, data preprocessing, loss function |
| `synthetic_data_fallback` | Fix dataset path, use cached dataset |
| `code_crash` | Parse traceback, fix the specific error |

**Apply the fix and re-run affected conditions.**

Log in `refinement_log.json`:
```json
{
  "iteration": 0,
  "type": "repair",
  "deficiency": "gpu_oom",
  "fix_applied": "batch_size: 64 → 32",
  "result": "success",
  "metrics_after": {...}
}
```

**If repair fails after MAX_REPAIR_CYCLES:**
- Mark deficiency as `repair_exhausted`
- Continue to Step 4 (may still achieve useful results)

### Step 4 — Assess Hypothesis Status

For each hypothesis, compare current metrics against the `success_threshold`:
- If `metric_mean >= success_threshold`: hypothesis is SUPPORTED
- If `metric_mean < baseline + 0.01`: hypothesis is NOT_SUPPORTED
- Otherwise: INCONCLUSIVE — needs more evidence

### Step 5 — Identify Improvement Targets

For each hypothesis that is NOT_SUPPORTED or INCONCLUSIVE:
- Diagnose the likely cause:
  - (a) hyperparameter mismatch
  - (b) model capacity insufficient
  - (c) baseline too strong
  - (d) data preprocessing issue
- Propose a specific change: adjust learning rate, increase model depth, modify loss function, etc.

### Step 6 — Refinement Iteration

For each proposed change:
1. Modify the experiment code/config
2. Re-run the affected conditions (minimum: the changed variant + baseline + 1 ablation)
3. Collect new metrics
4. Compare to previous iteration

Document in `refinement_log.json`:
- `iteration` number
- `change_made`: specific modification
- `metrics_before` / `metrics_after`: per hypothesis
- `improvement`: boolean

### Step 7 — Check Exit Conditions

After each iteration, check:
- **Convergence**: 2 consecutive iterations with no improvement on any hypothesis → exit loop
- **Budget**: refinement budget ≥ 95% exhausted → exit loop
- Otherwise → go to Step 5 for another iteration

### Step 8 — Snapshot Best Configuration

Copy the current `experiment/` to `experiment_final/` with the best settings applied (based on highest aggregate metric improvement).

### Step 9 — Write Final experiment_summary.json

Ensure the summary reflects the best configuration's metrics.

### Step 10 — Validate

Check:
- `refinement_log.json` has iterations
- `experiment_final/` exists
- Loop terminated validly (converged or budget exhausted)
- `completion_rate >= MIN_COMPLETION_RATE`

Fail → `E13`, `E13_LOW_COMPLETION`, or `E13_REPAIR_EXHAUSTED`.

---

## Deficiency Types

| Type | Severity | Auto-Repairable |
|------|----------|-----------------|
| `no_conditions_completed` | CRITICAL | No |
| `gpu_oom` | MAJOR | Yes |
| `nan_loss` | MAJOR | Yes |
| `missing_dependency` | CRITICAL | Yes |
| `identical_conditions` | MAJOR | Partially |
| `near_random` | MAJOR | Partially |
| `insufficient_seeds` | MINOR | Yes (run more seeds) |
| `synthetic_data_fallback` | CRITICAL | Yes |
| `code_crash` | MAJOR | Case-by-case |
| `time_guard_dominant` | MAJOR | Yes (reduce conditions) |
| `dataset_unavailable` | CRITICAL | Yes (use cached) |
| `permission_error` | CRITICAL | Partially |

---

## Key Rules

1. **Diagnose first**: Always run diagnosis before attempting refinement
2. **Repair automatically**: Fix common issues without human intervention
3. **Log everything**: Every repair attempt must be recorded
4. **Budget awareness**: Track compute budget throughout
5. **Graceful degradation**: If repair fails, still produce best-effort results
6. **No fabrication**: Never make up metrics — re-run actual experiments