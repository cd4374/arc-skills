---
name: arc-04-01-experiment-design
description: Stage 9 (GATE) — Design experiments with baselines, ablations, metrics, and resource estimates. Pipeline blocks here until approved.
metadata:
  category: pipeline-stage
  trigger-keywords: "experiment design,plan,stage 9,gate"
  applicable-stages: "9"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Create a complete experiment blueprint that covers all hypotheses with baselines and ablative studies. This is a GATE stage — no code generation begins until the plan is explicitly approved.

---

## Quality Contract

`exp_plan.yaml` MUST satisfy:
- Each hypothesis (H1, H2, ...) has ≥1 associated baseline and ≥1 ablation
- All experiments use ≥3 seeds with aggregated `mean ± std` output format
- Compute budget (`time_per_run_sec` × total_runs) is estimated and feasible
- Every dataset referenced is assigned a tier (tier-1 for iteration, tier-2 for final validation)

---

## Inputs
`artifacts/<run_id>/stage-08/hypotheses.md`
`artifacts/<run_id>/stage-08/hypothesis_index.json`
`artifacts/<run_id>/stage-01/hardware_profile.json`

---

## Outputs

### `exp_plan.yaml`
```yaml
hypotheses:
  - id: H1
    statement: "..."
    primary_metric: accuracy
    success_threshold: 0.85
baselines:
  - name: random_baseline
    description: "..."
    paper: "Author et al. 2023"
  - name: existing_method
    description: "..."
    paper: "..."
ablations:
  - component: component_A
    description: "removing component A"
    expected_effect: "accuracy drop of 2-5%"
ablation_settings:
  num_seeds: 3
  min_runs_per_seed: 5
datasets:
  tier1:
    - name: cifar10
      expected_size_gb: 0.16
  tier2:
    - name: imagenet
      expected_size_gb: 150
compute_budget:
  time_per_run_sec: 600
  max_total_runs: 60
  stop_at_percent: 80
```

### `design_rationale.md`
```
# Design Rationale

## Why These Baselines
[Justification for each baseline choice]

## Why These Ablations
[Justification for each ablation component]

## Expected Outcomes
[What each hypothesis being supported/refuted would look like in the metrics]
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Each hypothesis has baseline | All H1, H2, ... have ≥1 baseline |
| Each hypothesis has ablation | All H1, H2, ... have ≥1 ablation |
| `num_seeds` ≥ 3 | Ablation settings specify ≥ 3 seeds |
| Budget feasible | `time_per_run_sec × max_total_runs` ≤ `hardware_profile.json` time budget |
| Dataset tiers assigned | All datasets have tier-1 or tier-2 label |

**Failure**: If any check fails → `E09`

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` | `done`, advance to stage 10 |
| `approve` | `done`, advance to stage 10 |
| `reject` | `rejected`, pipeline rolls back to stage 8 |

---

## Rollback Contract
On `reject`: rollback target is **stage 8** (`arc-03-02-hypothesis-gen`). Prior hypotheses may be revised before re-designing experiments.

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E09` | Missing baseline or ablation for any hypothesis, or seeds < 3, or infeasible budget |

---

## Retry Policy
Max retries: **0** — gate rejection is not a retry. Use the `reject` → `rollback` path.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`

---

## Procedure

### Step 1 — Read Hypotheses and Hardware Profile
Read `hypotheses.md`, `hypothesis_index.json`, and `hardware_profile.json`.

### Step 2 — Select Baselines
For each hypothesis, identify the strongest existing methods to compare against:
- Look at the literature cards from stage-06 — what methods were used as baselines?
- Prioritize: (a) most cited methods on the same task, (b) most recent top-tier results
- Document each baseline with: name, description, expected performance range

### Step 3 — Design Ablations
For each hypothesis, identify the key components to ablate:
- Which architectural choices are novel in the proposed method?
- Which hyperparameters might matter most?
- Design ablation experiments that isolate the contribution of each component

### Step 4 — Assign Datasets
- **Tier-1** (for rapid iteration): Small/medium datasets that fit GPU memory and run quickly (e.g., CIFAR-10, MNIST, small subsamples)
- **Tier-2** (for final validation): Large/production datasets used in the literature (e.g., ImageNet, GLUE)

### Step 5 — Estimate Compute Budget
Calculate:
- `time_per_run_sec`: estimated runtime for one experiment run on the available hardware
- `max_total_runs`: sum of (baselines + ablations) × seeds × datasets
- `stop_at_percent`: 80% — if budget is exceeded, reduce seeds first, then drop tier-2 datasets

### Step 6 — Write exp_plan.yaml and design_rationale.md
Follow the Outputs templates exactly.

### Step 7 — Validate
Run Validation checks. Fail → `E09`.

### Step 8 — Block at Gate
Set status to `blocked_approval`. Wait for approve/reject. On `approve`: advance to stage 10. On `reject`: initiate rollback to stage 8.
