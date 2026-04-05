---
name: arc-04-02-code-generation
description: Stage 10 — Generate a runnable multi-file experiment project that implements the approved experiment plan.
metadata:
  category: pipeline-stage
  trigger-keywords: "code generation,experiment code,stage 10"
  applicable-stages: "10"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Transform the experiment plan into a complete, runnable, reproducible experiment project. The Agent executing this skill decides how to generate the code — the skill only defines what the code must produce and how it must behave.

---

## Quality Contract

The generated `experiment/` project MUST satisfy all of the following:

**Functional completeness:**
- Implements all hypotheses, baselines, and ablations from `exp_plan.yaml`
- Uses ≥3 seeds per condition and outputs metrics in the required format
- Generates `results.json` via a `finalize()` call on every run

**Output format:**
```
condition={name} seed={N} {metric}: {value}
condition={name} metric_mean: {mean} metric_std: {std}
```

**Reproducibility:**
- `requirements.txt` lists all dependencies
- `README.md` documents how to run and expected runtime

**Validation:**
- All `.py` files pass Python syntax check (`python -m py_compile`)
- Entry point (`main.py` or `run_experiment.sh`) accepts `--seed` and `--config` arguments

---

## Inputs
`artifacts/<run_id>/stage-09/exp_plan.yaml`
`artifacts/<run_id>/stage-09/design_rationale.md`
`artifacts/<run_id>/stage-01/hardware_profile.json`

---

## Outputs

### `experiment/` directory
```
experiment/
├── main.py              # or run_experiment.sh
├── config.py
├── models/model.py
├── datasets/dataset.py
├── train.py
├── evaluate.py
├── requirements.txt
└── README.md
```

### `experiment_spec.md`
```
# Experiment Specification

## File Structure
[listing of all generated files]

## How to Run
[exact command to execute, including seed and config arguments]

## Expected Runtime
[time per run, total time for full schedule]

## Metrics Produced
[which metrics are logged and in what format]
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| All hypotheses implemented | `exp_plan.yaml` coverage matches experiment code |
| Seeds ≥ 3 | Output format check confirms ≥3 seeds per condition |
| Output format correct | `condition=X metric_mean: M metric_std: S` format present |
| `results.json` generated | File written by `finalize()` call |
| Python syntax valid | All `.py` files pass `python -m py_compile` |
| README exists | `README.md` present with run instructions |

**Failure**: If any check fails → `E10`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E10` | Generated code fails validation on any check above |

---

## Retry Policy
Max retries: **2** — each retry targets specific validation failures.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Experiment Plan
Read `exp_plan.yaml`, `design_rationale.md`, and `hardware_profile.json`. Note: hypotheses, baselines, ablations, datasets, compute budget.

### Step 2 — Design Project Structure
Design the experiment directory structure:
- `main.py` or `run_experiment.sh`: entry point accepting `--seed` and `--config`
- `config.py`: configuration dataclass for each experiment condition
- `models/`: model implementations
- `datasets/`: dataset loaders for tier-1 and tier-2 datasets
- `train.py`: training loop
- `evaluate.py`: evaluation and metrics computation
- `finalize()` function: writes `results.json` at the end of each run

### Step 3 — Implement Experiment Code
Generate all Python files:
- Implement each hypothesis condition as a named config variant
- Implement all baseline methods
- Implement all ablation variants
- Ensure `--seed` argument sets `torch.manual_seed()` and `numpy.random.seed()`
- Ensure `--config` argument selects the experiment condition
- Each run must output metrics in the required format: `condition={name} seed={N} {metric}: {value}`

### Step 4 — Implement Results Aggregation
After all seeds complete, the `finalize()` function must:
- Read all per-seed outputs
- Compute `metric_mean` and `metric_std` across seeds
- Write `results.json` with all condition results

### Step 5 — Create requirements.txt and README.md
- `requirements.txt`: all pip-installable dependencies
- `README.md`: exact commands to run, expected runtime, hardware requirements

### Step 6 — Syntax Validation
Run `python -m py_compile` on every `.py` file. Fix any syntax errors.

### Step 7 — Validate
Run all Validation checks. Fail → `E10`.

### Step 8 — Write experiment_spec.md
Document file structure, run commands, expected runtime, and metrics format.
