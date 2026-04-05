---
name: arc-05-01-experiment-run
description: Stage 12 — Execute all scheduled experiment runs with real code execution. Requires an execution environment selected by Stage 0 (local GPU, local CPU, or SSH remote). Auto-diagnoses failures and repairs code.
metadata:
  category: pipeline-stage
  trigger-keywords: "experiment run,execute,runs,stage 12"
  applicable-stages: "12"
  priority: "1"
  version: "4.0"
  author: researchclaw
---

## Purpose

Execute the full experiment schedule and produce REAL metrics from ACTUAL code execution. This stage enforces that experiments must run — no simulated or fabricated data is allowed.

**CRITICAL**: This stage requires an execution backend. If `environment.json` shows `execution_capable: false`, this stage MUST fail immediately.

---

## Quality Contract

`experiment_summary.json` MUST satisfy:
- `completed_runs` = `total_runs` (all scheduled runs completed)
- Every condition has `metric_mean` and `metric_std` computed across ≥3 seeds
- All outputs use the required format: `condition=X metric_mean: M metric_std: S`
- No metric value is NaN or Inf
- **All data comes from ACTUAL code execution** — no fabricated metrics

---

## Inputs
`artifacts/<run_id>/stage-00/environment.json`
`artifacts/<run_id>/stage-10/experiment/`
`artifacts/<run_id>/stage-11/schedule.json`

---

## Outputs

### `runs/` directory
Per-run output directory: `runs/{run_id}/output.log`, `runs/{run_id}/metrics.json`

### `experiment_summary.json`
```json
{
  "total_runs": 47,
  "completed_runs": 47,
  "failed_runs": 0,
  "primary_metrics": {
    "h1_baseline": {"mean": 0.823, "std": 0.012, "n_seeds": 3},
    "h1_ablation": {"mean": 0.789, "std": 0.021, "n_seeds": 3}
  },
  "all_metrics": {},
  "execution_backend": "local",
  "execution_verified": true
}
```

### `execution_log.json`
```json
{
  "backend_used": "local",
  "gpu_info": {"type": "NVIDIA A100", "memory_gb": 80},
  "runs_executed": [
    {"run_id": "r001", "exit_code": 0, "elapsed_sec": 120.5, "condition": "baseline"}
  ],
  "repair_attempts": 0,
  "fabrication_detected": false
}
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Environment capable | `environment.json` shows `execution_capable: true` |
| All runs completed | `completed_runs == total_runs` |
| No NaN/Inf | All metric values are finite |
| Aggregated per condition | Each condition has `metric_mean` and `metric_std` |
| ≥3 seeds per condition | `n_seeds ≥ 3` for every condition |
| Execution verified | `execution_verified: true` in output |

**Failure**: 
- If `execution_capable: false` → `E12_NO_EXECUTION_ENV`
- If >50% runs fail after repair attempts → `E12`
- If fabrication detected → `E12_FABRICATION` (critical, non-retryable)

---

## Error Codes

| Code | Trigger | Retryable |
|------|---------|-----------|
| `E12_NO_EXECUTION_ENV` | No execution backend available | No |
| `E12` | >50% runs failed after repair | Yes (2 retries) |
| `E12_FABRICATION` | Fabricated/simulated data detected | No |
| `E12_TIMEOUT` | Execution exceeded time budget | Yes (1 retry with reduced scope) |

---

## Retry Policy
Max retries: **2** — retry re-executes only the failed runs, preserving completed ones.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 0 — Verify Execution Environment (MANDATORY)

Read `artifacts/<run_id>/stage-00/environment.json`.

**If `execution_capable: false`:**
1. Write `EXECUTION_ERROR.md`:
   ```markdown
   # Execution Error: No Backend Available
   
   The pipeline detected NO execution capability.
   
   ## How to Fix
   [Copy instructions from environment_error.md]
   
   ## What This Means
   Experiments CANNOT run without an execution backend.
   The pipeline will NOT fabricate data.
   ```
2. Fail immediately with `E12_NO_EXECUTION_ENV`
3. Do NOT proceed to Step 1

**If `execution_capable: true`:**
- Note the `primary_backend` and `fallback_backends`
- Proceed to Step 1

### Step 1 — Read Schedule and Experiment Code

Read `schedule.json` and verify the `experiment/` directory is complete:
- `main.py` or `run_experiment.sh` exists
- All required files present
- `requirements.txt` exists

### Step 2 — Prepare Execution Environment

**For Local GPU:**
```bash
# Verify GPU still available
nvidia-smi --query-gpu=index,memory.free --format=csv,noheader

# Create virtual environment if needed
python -m venv .venv && source .venv/bin/activate
pip install -q -r requirements.txt
```

**For Local CPU:**
```bash
# Create virtual environment if needed
python -m venv .venv && source .venv/bin/activate
pip install -q -r requirements.txt
```

**For SSH Remote:**
```bash
# Sync code to remote
rsync -avz --exclude='*.pt' --exclude='data/' experiment/ user@host:/remote/path/

# Verify remote GPU
ssh user@host "nvidia-smi --query-gpu=index,memory.free --format=csv,noheader"
```


### Step 3 — Execute Runs (with Auto-Debug)

For each entry in `schedule.runs`, execute the experiment:

```bash
cd experiment/
python main.py --seed={seed} --config={condition} 2>&1 | tee runs/{run_id}/output.log
```

Track each run's completion. Store per-run metrics in `runs/{run_id}/metrics.json`.

**Auto-Debug Protocol (on failure):**

If a run fails (exit code ≠ 0):

1. **Parse the error:**
   ```
   ModuleNotFoundError: No module named 'X'
   → Missing dependency
   
   CUDA out of memory
   → GPU OOM
   
   FileNotFoundError: [Errno 2] No such file or directory: 'data/'
   → Missing data path
   
   RuntimeError: CUDA error: device-side assert triggered
   → Code bug or data issue
   
   loss: nan
   → Training divergence
   ```

2. **Classify and attempt repair:**
   
   | Error Type | Auto-Fix | Max Attempts |
   |------------|----------|--------------|
   | Missing dependency | Add to requirements.txt, pip install | 1 |
   | GPU OOM | Reduce batch_size by 50% | 2 |
   | Missing data path | Fix path or use cached dataset | 1 |
   | NaN loss | Reduce learning rate 10x, add gradient clipping | 2 |
   | Code bug | No auto-fix — log and continue | 0 |

3. **Re-run with fix applied:**
   - Apply the fix to experiment code
   - Re-run ONLY the failed run
   - Log the repair in `execution_log.json`

4. **If repair fails after max attempts:**
   - Mark the run as failed
   - Continue to next run
   - Track in `failed_runs`

### Step 4 — Aggregate Results

After all runs complete, for each unique condition:
1. Collect all seed-level metric values from `runs/*/metrics.json`
2. Compute `metric_mean = mean(values)`, `metric_std = std(values)`
3. Record `n_seeds = count of valid seeds`

### Step 5 — Validate Metrics

Check for NaN or Inf in any metric. If any found:
- Mark that condition's result as invalid
- Log the issue with the specific run ID
- If >50% of conditions have invalid metrics → fail with `E12`

### Step 6 — Anti-Fabrication Check

Verify that `experiment_summary.json` contains data from ACTUAL execution:

1. **Check run logs exist:** Each `runs/{run_id}/output.log` should exist and be non-empty
2. **Check timestamps:** Execution timestamps should be sequential and realistic
3. **Check metric consistency:** Metrics should vary between seeds (not all identical)
4. **Check GPU usage:** If using GPU, verify GPU was actually used (check logs for CUDA messages)

If fabrication is suspected (e.g., all metrics identical, no execution logs, timestamps unrealistic):
- Fail with `E12_FABRICATION`
- Write `FABRICATION_REPORT.md` with evidence

### Step 7 — Write experiment_summary.json

Write the aggregated summary with:
- `completed_runs`, `failed_runs`, `primary_metrics`, `all_metrics`
- `execution_backend`: which backend was used
- `execution_verified: true`

### Step 8 — Validate

Check: 
- `completed_runs == total_runs`
- All metrics finite
- ≥3 seeds per condition
- `execution_verified: true`

Fail → `E12` (after repair exhaustion) or `E12_FABRICATION` (if fabrication detected).

---

## Execution Backends

### Local GPU
- Use for: NVIDIA CUDA or Apple MPS
- Pros: No network latency, full control
- Cons: Limited by local hardware

### Local CPU
- Use for: CPU-only execution when no GPU backend is available
- Pros: Widely available, simplest setup
- Cons: Slow for large experiments

### SSH Remote
- Use for: Powerful remote GPU servers selected in Stage 0
- Pros: Access to better hardware
- Cons: Network latency, requires SSH config

---

## Key Rules

1. **NO FABRICATION**: Never generate metrics without actual code execution
2. **Fail fast**: If no execution backend, fail immediately
3. **Auto-repair**: Attempt to fix common errors automatically
4. **Preserve evidence**: Keep all logs for verification
5. **Seed variation**: Ensure metrics vary between seeds (real randomness)