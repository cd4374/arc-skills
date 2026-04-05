---
name: arc-08-02-knowledge-archive
description: Stage 21 (NONCRITICAL) — Archive retrospectives and preserve reproducibility context. NONCRITICAL: failure does not block deliverables.
metadata:
  category: pipeline-stage
  trigger-keywords: "knowledge archive,retrospective,bundle,stage 21"
  applicable-stages: "21"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Capture the run's execution history and preserve reproducibility context for later inspection. The archive should preserve the same final experiment code snapshot used by the export package, but it does not own the canonical `reproducibility_report.json` written by Stage 22. This stage does not affect paper quality — it is purely archival.

---

## Quality Contract

`archive.md` + `bundle_index.json` MUST be produced. These files may be minimal if the pipeline ran with `skip_noncritical=true`.

---

## Inputs
`artifacts/<run_id>/stage_history.jsonl`
`artifacts/<run_id>/decision_history.json`
`artifacts/<run_id>/checkpoint.json`
`artifacts/<run_id>/pipeline_state.json`
`artifacts/<run_id>/stage-13/experiment_final/` (final experiment code snapshot)

---

## Outputs

### `archive.md`
```
# Knowledge Archive

## Run Summary
- Topic: [from goal.md]
- Stages completed: N
- Key decisions: [from decision_history.json]

## What Worked
## What Did Not Work
## Execution Trace
```

### `bundle_index.json`
```json
{
  "experiment_code": "experiment_final/",
  "configs": ["config files"],
  "checkpoints": [],
  "total_artifacts": 3
}
```

---

## Procedure

### Step 1 — Collect Run History
Read `stage_history.jsonl`, `decision_history.json`, `checkpoint.json`, and `pipeline_state.json`.

### Step 2 — Extract Key Decisions
From `decision_history.json`: list all pivot/refine decisions and their justifications.

### Step 3 — Document Execution Trace
Summarize what happened across the pipeline so far: which stages completed, which were skipped, what the key execution events were, and whether stage 26 pre-gate (`arc-10-04`) checks were triggered.

### Step 4 — Write "What Worked / Did Not Work"
From the stage history, identify:
- Which experimental configurations led to improvements
- Which changes had no effect or negative effect

### Step 5 — Bundle Reproducibility Artifacts
Copy from `stage-13/experiment_final/` to a bundle directory so the archive preserves the same final code snapshot used by stage 22 export:
- All source code
- Config files
- Any checkpoints

### Step 6 — Write archive.md and bundle_index.json
Follow the Outputs templates.

### Step 7 — Validate
Check both output files exist and are non-empty. Fail → `E21`.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| `archive.md` exists | File non-empty |
| `bundle_index.json` valid | JSON with required fields |
| `bundle_index.json` lists artifacts | All key artifacts are referenced |

**Failure**: Missing either required output → `E21`

---

## Noncritical Note
This stage is **NONCRITICAL**. If it fails and `skip_noncritical=true`, the pipeline continues without blocking.

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E21` | Archive or bundle index could not be produced |

---

## Retry Policy
Max retries: **1**

---

## State Transition
`pending` → `running` → `done` | `failed`
