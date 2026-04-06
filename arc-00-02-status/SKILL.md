---
name: arc-00-02-status
description: Read-only inspector for pipeline run state. Reports current stage, completion, failures, and late-stage quality/authenticity observability signals.
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
| `stage-23/verification_report.json` | citation authenticity + citation sufficiency status |
| `stage-24/review_state.json` / `stage-24/AUTO_REVIEW.md` | whether review-loop artifacts were recorded honestly |
| `stage-25/polish_report.json` | citation thresholds + structural polish status |
| `stage-26-pre/numeric_truth_report.json` | numeric-truth pre-gate pass/fail and whether Stage 26 progress is still integrity-backed |
| `stage-27/figure_quality_report.json` | figure quality + authenticity status |
| `stage-28/submission_format_report.json` | final package, figure, citation, and reproducibility gate status |
| `stage-28/submission_bundle_manifest.json` | final bundle completeness + ready status |

---

## Status Report Format

```
=== Pipeline Status: rc-<run_id> ===

Current stage: 27 (arc-10-02-figure-quality-gate)
Status: blocked_approval
Pivot count: 1 / 2

Completed stages:
  ✅ 1 arc-01-01-topic-init
  ...
  ✅ 23 arc-08-04-citation-verify
  ✅ 24 arc-09-01-paper-review-loop
  ✅ 25 arc-09-02-paper-polish
  ✅ 26 arc-10-01-claim-evidence-trace-gate
  🔄 27 arc-10-02-figure-quality-gate (blocked)

Decision history:
  Attempt 1: refine → stage 13 (ITERATIVE_REFINE)

Late-stage signals:
  Citations: 34 verified | recent ratio 0.26 | threshold ✅
  Review loop: recorded ✅
  Polish: pass ✅
  Numeric truth: pass ✅
  Figures: 6 total | authenticity ✅ | gate ✅
  Reproducibility: present ✅ | semantic integrity ✅
  Submission gate: not yet run

Last heartbeat: 2026-04-05T15:30:22Z

=== Run artifacts ===
  checkpoint.json    ✅
  stage_history.jsonl ✅ (31 events)
  decision_history.json ✅
  stage-23/verification_report.json ✅
  stage-24/review_state.json ✅
  stage-24/AUTO_REVIEW.md ✅
  stage-25/polish_report.json ✅
  stage-26-pre/numeric_truth_report.json ✅
  stage-27/figure_quality_report.json ✅
```

---

## Validation
- `run_id` directory must exist
- If `pipeline_state.json` not found → report `run not found`
- If late-stage artifacts exist, reported citation / figure / reproducibility signals must reflect their current values

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

### Step 7 — Read late-stage observability artifacts (if they exist)
Inspect these files when present:
- `stage-23/verification_report.json` for citation count, recent-reference ratio, and blocking citation issues
- `stage-24/review_state.json` / `stage-24/AUTO_REVIEW.md` for whether review-loop artifacts were recorded
- `stage-25/polish_report.json` for citation-threshold and structural-polish status
- `stage-26-pre/numeric_truth_report.json` for numeric-truth pre-gate status and whether Stage 26 progress remains integrity-backed
- `stage-27/figure_quality_report.json` for figure count, quality, traceability, and authenticity status
- `stage-28/submission_format_report.json` for final compile/package/citation/figure/reproducibility gate status

### Step 8 — Format Status Report
Output the status report in the format defined in the Outputs section, including a concise late-stage signals block whenever those artifacts exist. If Stage 26 or later is claimed but `stage-26-pre/numeric_truth_report.json` is missing or failing, report that integrity break explicitly instead of implying late-stage readiness.
