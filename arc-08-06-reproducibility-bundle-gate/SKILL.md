---
name: arc-08-06-reproducibility-bundle-gate
description: Stage 21.5 (GATE) — Verify that the reproducibility bundle is complete and executable before final export packaging proceeds.
metadata:
  category: pipeline-stage
  trigger-keywords: "reproducibility bundle gate,repro bundle,artifact reproducibility,stage 21.5"
  applicable-stages: "21.5"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Catch missing or unusable reproducibility artifacts immediately after packaging logic exists, before submission-oriented export and citation verification hide the defect. This stage verifies that code, configs, environment locks, execution manifests, and run instructions are actually bundled and coherent.

---

## Quality Contract

`reproducibility_bundle_report.json` MUST:
- Confirm the bundle includes runnable code or scripts for the reported experiments
- Confirm the bundle includes configuration artifacts, environment/dependency lock artifacts, and execution instructions
- Confirm the bundle records seeds, hardware/software environment, and artifact manifest information
- Confirm every claimed core result has at least one corresponding artifact pointer in the bundle
- Set `reproducibility_bundle_pass: true` only if all blocking checks pass

Hard constraints:
- At least one environment/dependency lock artifact MUST exist (`requirements.txt`, `environment.yml`, `pyproject.toml`, or `Dockerfile`)
- A human-readable reproduction entrypoint MUST exist (`README`, `run.sh`, `Makefile`, or equivalent)
- Seed policy and runtime environment recording MUST be present in bundled artifacts
- Core result artifacts referenced by later paper/export stages MUST exist in the bundle manifest

---

## Inputs
`artifacts/<run_id>/stage-21/package_manifest.json`
`artifacts/<run_id>/stage-21/reproducibility/`
`artifacts/<run_id>/stage-09b/reproducibility_contract.md`
`artifacts/<run_id>/stage-15b/claim_scope_report.json`

---

## Outputs

### `artifacts/<run_id>/stage-21b/reproducibility_bundle_report.json`
```json
{
  "reproducibility_bundle_pass": true,
  "checks": {
    "entrypoint_present": true,
    "environment_lock_present": true,
    "seed_policy_present": true,
    "runtime_record_present": true,
    "artifact_manifest_present": true,
    "claim_artifact_links_complete": true
  },
  "bundle_files_checked": [
    "reproducibility/README.md",
    "reproducibility/requirements.txt",
    "reproducibility/run.sh"
  ],
  "warnings": [],
  "blocking_issues": []
}
```

### `artifacts/<run_id>/stage-21b/reproducibility_bundle_checklist.md`
A concise checklist showing whether the reproducibility bundle contains runnable entrypoints, dependency locks, environment records, and links from claims to artifacts.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Entrypoint present | A human-readable reproduction entrypoint exists |
| Environment lock present | At least one dependency/environment lock artifact exists |
| Seed policy present | Seed handling is documented in the bundle |
| Runtime record present | Hardware/software environment recording is present |
| Artifact manifest present | Bundle manifest enumerates expected artifacts |
| Claim links complete | Core claims can be traced to bundled artifacts |
| Gate pass flag | `reproducibility_bundle_pass == true` |

**Failure**: Missing required bundle artifacts or claim/artifact traceability → `E21B`

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `reproducibility_bundle_pass=true` | `done`, advance to stage 22 |
| `approve` and `reproducibility_bundle_pass=true` | `done`, advance to stage 22 |
| `reject` or `reproducibility_bundle_pass=false` | `rejected`, rollback to stage 20 |

---

## Rollback Contract
On `reject` or `reproducibility_bundle_pass=false`: rollback target is **stage 20** (`arc-08-01-quality-gate`).

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E21B` | Reproducibility bundle report missing or blocking checks failed |
| `E21B_ENTRYPOINT_MISSING` | No runnable reproduction entrypoint found |
| `E21B_ENV_LOCK_MISSING` | No dependency/environment lock artifact found |
| `E21B_RUNTIME_RECORD_MISSING` | Runtime environment recording missing |
| `E21B_CLAIM_LINK_MISSING` | One or more core claims cannot be traced to bundled artifacts |

---

## Retry Policy
Max retries: **0** — gate rejection uses rollback path.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`

---

## Procedure

### Step 1 — Read Bundle Inputs
Read the package manifest, bundled reproducibility directory, reproducibility contract, and claim-scope report.

### Step 2 — Check Bundle Structure
Verify the bundle contains executable or scriptable entrypoints, dependency/environment lock artifacts, and a manifest of included artifacts.

### Step 3 — Check Reproduction Metadata
Verify that seed policy, runtime environment recording, and run instructions are present and internally consistent.

### Step 4 — Check Claim-to-Artifact Traceability
Confirm that each core claim expected downstream can be traced to a concrete artifact or result file in the bundle.

### Step 5 — Write Outputs
Write `reproducibility_bundle_report.json` and `reproducibility_bundle_checklist.md` under `stage-21b/`.

### Step 6 — Gate Decision
If required bundle artifacts are missing or core claims cannot be traced to the bundle, fail with `E21B`. Otherwise block for approval and then advance to stage 22.
