---
name: arc-04-04-reproducibility-design-gate
description: Stage 9.5 (GATE) — Freeze the reproducibility contract before code generation begins.
metadata:
  category: pipeline-stage
  trigger-keywords: "reproducibility design gate,experiment reproducibility,stage 9.5"
  applicable-stages: "9.5"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Require that the experiment design explicitly declares how the work will be reproduced before any code is written. This stage blocks vague plans that postpone seeds, environment recording, statistical testing, or artifact packaging to the end of the pipeline.

---

## Quality Contract

`reproducibility_design_report.json` MUST:
- Confirm that the experiment plan declares seed policy, environment-recording policy, and dependency-lock policy
- Confirm that datasets, baselines, metrics, and statistical comparison method are explicit
- Confirm that checkpoint/save policy and hardware recording policy are explicit
- Confirm that a reproducibility bundle is planned as an output of later stages
- Set `reproducibility_design_pass: true` only if all required items are present

Hard constraints:
- `num_seeds` MUST remain ≥ 3 unless topic constraints are explicitly justified
- At least one dependency/environment lock artifact must be planned (`requirements.txt`, `environment.yml`, or `Dockerfile`)
- Statistical significance method must be declared for baseline comparisons unless explicitly not applicable
- Hardware/software version recording must be declared

---

## Inputs
`artifacts/<run_id>/stage-09/exp_plan.yaml`
`artifacts/<run_id>/stage-09/design_rationale.md`
`artifacts/<run_id>/stage-01/hardware_profile.json`

---

## Outputs

### `artifacts/<run_id>/stage-09b/reproducibility_design_report.json`
```json
{
  "reproducibility_design_pass": true,
  "checks": {
    "seed_policy_present": true,
    "dependency_lock_declared": true,
    "hardware_recording_declared": true,
    "checkpoint_policy_declared": true,
    "significance_test_declared": true,
    "bundle_outputs_declared": true
  },
  "warnings": [],
  "blocking_issues": []
}
```

### `artifacts/<run_id>/stage-09b/reproducibility_contract.md`
Short human-readable contract describing the required reproducibility outputs and execution logging obligations for stages 10–22.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Seed policy declared | Design includes explicit seed handling and `num_seeds >= 3` |
| Dependency lock declared | At least one of `requirements.txt`, `environment.yml`, or `Dockerfile` is planned |
| Hardware recording declared | Runtime environment recording is explicit |
| Checkpoint policy declared | Save/restart policy is explicit |
| Statistical method declared | Baseline comparison method is explicit or justified as not applicable |
| Bundle outputs declared | Later reproducibility bundle artifacts are explicitly planned |
| Gate pass flag | `reproducibility_design_pass == true` |

**Failure**: Missing reproducibility design fields → `E09B`

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `reproducibility_design_pass=true` | `done`, advance to stage 10 |
| `approve` and `reproducibility_design_pass=true` | `done`, advance to stage 10 |
| `reject` or `reproducibility_design_pass=false` | `rejected`, rollback to stage 9 |

---

## Rollback Contract
On `reject` or `reproducibility_design_pass=false`: rollback target is **stage 9** (`arc-04-01-experiment-design`).

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E09B` | Reproducibility design report missing required fields |
| `E09B_SEED_POLICY_MISSING` | Seed policy missing or `num_seeds < 3` without justification |
| `E09B_ENV_LOCK_MISSING` | No dependency/environment lock artifact declared |
| `E09B_STATS_METHOD_MISSING` | Statistical comparison method missing |

---

## Retry Policy
Max retries: **0** — gate rejection uses rollback path.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`

---

## Procedure

### Step 1 — Read Experiment Plan
Read `exp_plan.yaml`, `design_rationale.md`, and `hardware_profile.json`.

### Step 2 — Extract Reproducibility Requirements
Check whether the plan explicitly defines:
- seed policy
- dependency/environment lock artifact
- runtime environment recording
- checkpoint/save policy
- statistical comparison method
- later reproducibility bundle outputs

### Step 3 — Check Minimum Standards
Verify `num_seeds >= 3`, identify the planned lock artifact, and confirm the baseline comparison method is explicit.

### Step 4 — Write Reproducibility Contract
Write a short contract that later stages must follow when generating code, running experiments, and packaging outputs.

### Step 5 — Write Report
Write `reproducibility_design_report.json` and `reproducibility_contract.md` under `stage-09b/`.

### Step 6 — Gate Decision
If any required reproducibility element is missing, fail with `E09B`. Otherwise block for approval and then advance to stage 10.
