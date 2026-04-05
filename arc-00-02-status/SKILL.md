---
name: arc-00-02-status
description: Read-only inspector for pipeline run state. Reports current stage, completion, and any failures.
metadata:
  category: orchestrator
  trigger-keywords: "arc status,run status,stage progress"
  applicable-stages: "1-28"
  priority: "1"
  version: "4.1"
  author: researchclaw
---

## Purpose
Inspect the current state of a running or completed pipeline without modifying any files.

---

## Inputs
- `run_id` (required): run identifier

---

## Outputs

Reports the following from `artifacts/<run_id>/`:

| File | What to Report |
|------|---------------|
| `pipeline_state.json` | current_stage, status, pivot_count |
| `checkpoint.json` | last_completed_stage, timestamp |
| `stage_history.jsonl` | all events, stages done/failed |
| `decision_history.json` | all pivot/refine decisions |
| `heartbeat.json` | last activity timestamp |

---

## Status Report Format

```
=== Pipeline Status: rc-<run_id> ===

Current stage: 5 (arc-02-03-literature-screen)
Status: running
Pivot count: 1 / 2

Completed stages:
  ✅ 1 arc-01-01-topic-init
  ✅ 2 arc-01-02-problem-decompose
  ✅ 3 arc-02-01-search-strategy
  ✅ 4 arc-02-02-literature-collect
  🔄 5 arc-02-03-literature-screen (running)

Decision history:
  Attempt 1: refine → stage 13 (ITERATIVE_REFINE)

Last heartbeat: 2026-04-05T15:30:22Z

=== Run artifacts ===
  checkpoint.json    ✅
  stage_history.jsonl ✅ (5 events)
  decision_history.json ✅
```

---

## Validation
- `run_id` directory must exist
- If `pipeline_state.json` not found → report `run not found`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E_STS_01` | Run directory `artifacts/<run_id>/` does not exist |
| `E_STS_02` | `pipeline_state.json` is malformed or unreadable |
| `E_STS_03` | `stage_history.jsonl` is malformed (corrupted append log) |

---

## Retry Policy
Max retries: **0** — this is a read-only inspector. Failures indicate the run does not exist or state files are corrupted.

---

## State Transition
Read-only — no state changes.

---

## Procedure

Note: Stage 26 may include a pre-gate check via `arc-10-04-numeric-truth-gate` before `arc-10-01`, with report output at `stage-26-pre/numeric_truth_report.json`.


### Step 1 — Validate Run Directory
Verify `artifacts/<run_id>/` exists. If not found, report `run not found`.

### Step 2 — Read pipeline_state.json
Parse `pipeline_state.json`. Extract:
- `current_stage`, `current_skill`, `status`, `pivot_count`

### Step 3 — Read checkpoint.json (if exists)
Parse `checkpoint.json`. Extract:
- `last_completed_stage`, `last_completed_name`, `timestamp`

### Step 4 — Read stage_history.jsonl (if exists)
Parse each line. Count:
- Total events
- Stages completed vs. failed
- Last event timestamp

### Step 5 — Read decision_history.json (if exists)
Parse array. List all pivot/refine decisions with attempt numbers and timestamps.

### Step 6 — Read heartbeat.json (if exists)
Parse `heartbeat.json`. Note last activity timestamp.

### Step 7 — Format Status Report
Output the status report in the format defined in the Outputs section.
