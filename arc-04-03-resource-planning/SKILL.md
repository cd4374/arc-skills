---
name: arc-04-03-resource-planning
description: Stage 11 — Build an executable schedule that maps all experiment runs to available resources and time budget.
metadata:
  category: pipeline-stage
  trigger-keywords: "resource planning,schedule,gpu,time,stage 11"
  applicable-stages: "11"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Convert the experiment plan into a time-resolved schedule that fits within the available compute budget and respects GPU memory constraints.

---

## Quality Contract

`schedule.json` MUST satisfy:
- `total_time_sec` ≤ the available time budget from `hardware_profile.json`
- All (condition, dataset, seed) combinations from `exp_plan.yaml` are scheduled
- `parallel_groups` are specified to maximize GPU utilization
- Intermediate checkpoints are defined at ≥2 points for early validation

---

## Inputs
`artifacts/<run_id>/stage-09/exp_plan.yaml`
`artifacts/<run_id>/stage-01/hardware_profile.json`

---

## Outputs

### `schedule.json`
```json
{
  "total_runs": 47,
  "total_time_sec": 14100,
  "time_budget_sec": 14400,
  "budget_utilization_pct": 97.9,
  "runs": [
    {
      "id": "run_001",
      "condition": "h1_baseline",
      "dataset": "cifar10",
      "seed": 0,
      "estimated_time_sec": 300,
      "parallel_group": 1
    }
  ],
  "parallel_groups": [[1,2,3],[4],[5,6]],
  "checkpoints": [
    {"at_run": 10, "action": "validate_intermediate"},
    {"at_run": 47, "action": "finalize"}
  ]
}
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| `total_time_sec` ≤ budget | Schedule fits within time budget |
| All conditions scheduled | Every condition × seed × dataset combination appears |
| `parallel_groups` defined | Array of run groups for parallel execution |
| ≥2 checkpoints | At least 2 intermediate checkpoints defined |

**Failure**: If schedule exceeds budget and cannot be reduced by dropping seeds/datasets → `E11`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E11` | Schedule exceeds time budget and cannot be feasibly reduced |

---

## Retry Policy
Max retries: **1** — retry with reduced scope (fewer seeds, tier-2 datasets dropped).

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read exp_plan.yaml and hardware_profile.json
Extract all conditions (baselines + ablations × seeds × datasets), `time_per_run_sec`, and hardware constraints.

### Step 2 — Calculate Total Runs
Count: sum over all conditions of (number of seeds × number of datasets).
Example: 5 conditions × 3 seeds × 2 datasets = 30 runs.

### Step 3 — Group Runs for Parallel Execution
Identify runs that can execute simultaneously on the available GPU(s):
- If `gpu_count >= 2` in `hardware_profile.json`: group runs that use different GPUs
- If single GPU: runs must be sequential
- Create `parallel_groups`: each group is a list of run IDs that share a time slot

### Step 4 — Schedule Runs with Checkpoints
Order runs: run tier-1 dataset experiments first (faster feedback), tier-2 last.
Insert checkpoints at ≥2 points:
- Early checkpoint (after ~30% of runs): validate intermediate results, check for divergence/NaN
- Final checkpoint (after ~80-90% of runs): assess convergence

### Step 5 — Validate Budget
Calculate `total_time_sec = sum(estimated_time_per_run × num_parallel_slots)`.
If `total_time_sec > time_budget_sec`:
- First reduction: drop to 1 seed per condition
- Second reduction: drop tier-2 datasets
- If still over budget → fail with `E11`

### Step 6 — Write schedule.json
Follow the Outputs template with `runs`, `parallel_groups`, `checkpoints`, and budget utilization.

### Step 7 — Validate
Run Validation checks. Fail → `E11`.
