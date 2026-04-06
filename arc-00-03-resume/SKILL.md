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

Resume must preserve the same hard output invariants as a fresh run: no bypass of blocking environment checks, no acceptance of hallucinated or insufficient citations, no silent downgrade of figure authenticity requirements, and no loss of reproducibility semantics during late-stage continuation.

---

## Inputs
- `run_id` (required)
- `target_venue` (optional, inherit from original unless explicitly changed before template resolution)
- `template_version` (optional, inherit from original unless explicitly changed before template resolution)
- `publication_type` (optional, inherit from original)
- `submission_mode` (optional, inherit from original)
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
- If resuming after late-stage export, citation / figure / reproducibility artifacts must still satisfy the expected invariants for the claimed completed stages

---

## Safety Rules
- Never overwrite existing stage outputs without version suffix
- Append to `stage_history.jsonl`, never truncate
- Do not count resumed run as new pivot attempt
- Rollback/routing rules identical to `arc-00-01`
- Resume must not skip or waive citation-threshold, figure-authenticity, or reproducibility checks that were required before interruption

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
- If Stage 7.5 is claimed complete, verify `stage-07b/novelty_gap_report.json` exists and `novelty_gate_pass == true`
- If Stage 9.5 is claimed complete, verify `stage-09b/reproducibility_design_report.json` exists and `reproducibility_design_pass == true`
- If Stage 15.5 is claimed complete, verify `stage-15b/claim_scope_report.json` exists and `claim_scope_pass == true`
- If Stage 17.5 is claimed complete, verify `stage-17b/writing_compliance_report.json` exists and `writing_compliance_pass == true`
- If Stage 18.5 is claimed complete, verify `stage-18b/bibliography_quality_report.json` exists and `bibliography_quality_pass == true`
- If Stage 21.5 is claimed complete, verify `stage-21b/reproducibility_bundle_report.json` exists and `reproducibility_bundle_pass == true`
- If Stage 23 is claimed complete, verify citation verification artifacts still exist, were not invalidated, and do not report `hallucinated > 0` or `unverifiable > 0`
- If Stage 24.5 is claimed complete, verify `stage-24b/academic_integrity_report.json` exists and `academic_integrity_pass == true`
- If Stage 25 is claimed complete, verify the polished package still includes reproducibility metadata and the expected bibliography/package artifacts
- If Stage 24 is claimed complete, verify `stage-24/review_state.json` and `stage-24/AUTO_REVIEW.md` still exist
- If Stage 26 progress is claimed, verify `stage-26-pre/numeric_truth_report.json` exists; if Stage 26 is claimed complete, verify that report still reflects `numeric_truth_gate_pass == true`
- If Stage 27 is claimed complete, verify figure-quality artifacts still exist and still correspond to the stage-25 polished package
- If Stage 28 is claimed complete, verify `stage-28/submission_format_report.json` still exists and still reflects a passing final gate
- If Stage 28.5 is claimed complete, verify `stage-28b/final_acceptance_report.json` exists and `final_acceptance_pass == true`
- If any required late-stage artifact is missing or stale → fail with `E_RESUME_03`

### Step 5 — Determine Resume Point
Let `resume_stage = checkpoint.last_completed_stage + 1`.
Do NOT skip to a later stage — resume from exactly after the last successful checkpoint.

### Step 6 — Prepare Resume State
- Do NOT overwrite existing `checkpoint.json`
- Append to `stage_history.jsonl` only (never truncate)
- Resume run does NOT count as a new pivot attempt
- Rollback/routing rules are identical to fresh start
- Preserve inherited venue/template configuration and all late-stage authenticity/citation/figure/reproducibility invariants

### Step 7 — Execute Resume
Transition `pipeline_state.status` to `running` and hand off to the resumed stage skill.
