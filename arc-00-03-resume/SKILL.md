---
name: arc-00-03-resume
description: Resume a paused, blocked, or failed pipeline from the last checkpoint.
metadata:
  category: orchestrator
  trigger-keywords: "arc resume,resume pipeline,continue run"
  applicable-stages: "1-28"
  priority: "1"
  version: "4.1"
  author: researchclaw
---

## Purpose
Continue execution from the last checkpoint without re-running completed stages.

---

## Inputs
- `run_id` (required)
- `auto_approve_gates` (optional, inherit from original)
- `skip_noncritical` (optional, inherit from original)

---

## Resume Logic

Read `pipeline_state.json` and `checkpoint.json`:

| `pipeline_state.status` | Resume Action |
|------------------------|---------------|
| `paused` | Re-run same stage |
| `blocked_approval` | Prompt for approve/reject, then route accordingly (including stage 26 pre-gate (`arc-10-04`, output: `stage-26-pre/numeric_truth_report.json`) and stages 26–28 gate rollbacks; stage 27/28 rollbacks must invalidate and re-run stages 23–25 after returning to stage 22) |
| `failed` | Retry if retries < cap, else check noncritical or abort |
| `running` | Treat as crash — retry same stage |

---

## Validation
- `checkpoint.json` must exist
- Stage history must confirm claimed completed stages actually produced their outputs
- Resume point must be unambiguous

---

## Safety Rules
- Never overwrite existing stage outputs without version suffix
- Append to `stage_history.jsonl`, never truncate
- Do not count resumed run as new pivot attempt
- Rollback/routing rules identical to `arc-00-01`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E_RESUME_01` | `checkpoint.json` missing from run directory |
| `E_RESUME_02` | `pipeline_state.json` is malformed or unreadable |
| `E_RESUME_03` | Stage history integrity check failed — claimed completed stages have missing outputs |
| `E_RESUME_04` | Resume point is ambiguous (gap between checkpoint and history) |

---

## Retry Policy
Max retries: **1** — retry if Step 1–3 (checkpoint/state validation) fails due to transient file system issue. If integrity check (Step 4) fails → `E_RESUME_03`, do not retry.

---

## State Transition
Resuming does not change `pipeline_state.status` until the resumed stage begins executing.

---

## Procedure

### Step 1 — Validate Checkpoint
Verify `artifacts/<run_id>/checkpoint.json` exists. If missing → fail. Read it.

### Step 2 — Read pipeline_state.json
Parse `pipeline_state.json`. Extract `current_stage`, `status`, `pivot_count`.

### Step 3 — Determine Resume Action
Route by `pipeline_state.status`:

| Status | Resume Action |
|--------|--------------|
| `paused` | Re-run same stage (set to `running`) |
| `blocked_approval` | Prompt for approve/reject, then route accordingly (including stage 26 pre-gate (`arc-10-04`, output: `stage-26-pre/numeric_truth_report.json`) and stages 26–28 gate rollbacks; stage 27/28 rollbacks must invalidate and re-run stages 23–25 after returning to stage 22) |
| `failed` | If retries < cap: retry; else check noncritical or abort |
| `running` | Treat as crash: retry same stage |

### Step 4 — Verify Stage History Integrity
Check `stage_history.jsonl` confirms claimed completed stages actually produced their outputs:
- For each completed stage, verify its output directory exists and is non-empty
- If any output is missing → fail with `E_RESUME_INVALID`

### Step 5 — Determine Resume Point
Let `resume_stage = checkpoint.last_completed_stage + 1`.
Do NOT skip to a later stage — resume from exactly after the last successful checkpoint.

### Step 6 — Prepare Resume State
- Do NOT overwrite existing `checkpoint.json`
- Append to `stage_history.jsonl` only (never truncate)
- Resume run does NOT count as a new pivot attempt
- Rollback/routing rules are identical to fresh start

### Step 7 — Execute Resume
Transition `pipeline_state.status` to `running` and hand off to the resumed stage skill.
