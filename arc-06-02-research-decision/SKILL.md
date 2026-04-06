---
name: arc-06-02-research-decision
description: Stage 15 — Make a PROCEED / REFINE / PIVOT decision based on experimental evidence. Enforces a hard cap of 2 pivot/refine cycles.
metadata:
  category: pipeline-stage
  trigger-keywords: "research decision,proceed,refine,pivot,stage 15"
  applicable-stages: "15"
  priority: "1"
  version: "4.1"
  author: researchclaw
---

## Purpose
Decide whether to continue to paper writing (PROCEED), re-run experiments with changes (REFINE), or discard current hypotheses and regenerate them (PIVOT). The loop cap prevents infinite iteration.

---

## Quality Contract

`decision_routing.json` MUST contain one of:
- `{"decision": "proceed", ...}`
- `{"decision": "refine", ...}`
- `{"decision": "pivot", ...}`

`decision.md` MUST justify the decision with evidence from `analysis.md`.

When `pivot_count >= 2`: decision MUST be `"proceed"` (forced) and `forced_proceed: true` MUST be set.

---

## Inputs
`artifacts/<run_id>/stage-14/analysis.md`
`artifacts/<run_id>/stage-14/experiment_summary.json`
`artifacts/<run_id>/decision_history.json` (if exists)

---

## Outputs

### `decision.md`
```
# Research Decision — Stage 15

## Evidence Summary
[Key findings from analysis.md]

## Decision: PROCEED / REFINE / PIVOT

## Justification
[Evidence-based reasoning for the decision]

## If REFINE: What to Change
[Specific modifications to experiment configuration]

## If PIVOT: Why Hypotheses Are Inadequate
[Specific evidence that current hypotheses are falsified]
```

### `decision_routing.json`
```json
{
  "decision": "proceed" | "refine" | "pivot",
  "confidence": 0.82,
  "evidence_summary": "...",
  "rollback_target": null,
  "rollback_stage_num": null,
  "pivot_count_after": 0,
  "forced_proceed": false,
  "quality_warning": false
}
```

---

## Rollback Contract

This stage is the **decision point** — it does not gate approval but emits routing decisions.

| Decision | Next Stage | Counts Against Cap | Rollback Target |
|----------|-----------|-------------------|-----------------|
| `proceed` | `arc-06-03` (result claim gate) → `arc-07-00` (template) → `arc-07-01` | no | — |
| `refine` | `arc-05-02` | **yes** | stage-13 (`arc-05-02`) |
| `pivot` | `arc-03-02` | **yes** | stage-08 (`arc-03-02`) |

**MAX_DECISION_PIVOTS = 2**: If `decision_history.json` shows ≥2 prior `refine`/`pivot` decisions, this stage MUST:
1. Output `decision: "proceed"` with `forced_proceed: true`
2. Write `quality_warning.txt` to the run root
3. Pipeline continues to stage 15.5 (result-claim-gate) then 15.7 (template) then stage 16 (no actual rollback occurs)

On `refine`: orchestrator versions stage-13 through stage-15 (`stage-N/` → `stage-N_v{attempt}/`) then resumes from stage-13.
On `pivot`: orchestrator versions stage-08 through stage-15 then resumes from stage-08.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Valid decision value | `decision` is `proceed` \| `refine` \| `pivot` |
| Loop cap enforced | If `pivot_count >= 2` → `decision == "proceed"` and `forced_proceed == true` |
| `decision.md` exists | File with justification present |
| `quality_warning.txt` on forced | File written to run root when `forced_proceed == true` |

**Failure**: If decision cannot be determined or output is malformed → `E15`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E15` | Decision is not one of the three valid values, or loop cap violated |
| `E15_PIVOT_CAP` | Forced proceed triggered (informational — pipeline continues) |

---

## Retry Policy
Max retries: **1** — retry if LLM output is malformed (not a valid decision).

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Analysis and History
Read `analysis.md`, `experiment_summary.json`, and `decision_history.json` (if exists).

### Step 2 — Count Prior Pivot/Refine Cycles
If `decision_history.json` exists, count prior decisions that were `refine` or `pivot`. Let `pivot_count = that number`.

### Step 3 — Make Decision
Based on `analysis.md` hypothesis assessments:
- If ALL hypotheses are SUPPORTED with strong evidence → `proceed`
- If any hypothesis is NOT_SUPPORTED or INCONCLUSIVE but the result is salvageable (adjusting approach, adding data, fixing bugs) → `refine`
- If any hypothesis is NOT_SUPPORTED and the approach is fundamentally flawed (the core claim is wrong) → `pivot`

### Step 4 — Enforce Loop Cap
If `pivot_count >= 2`:
- Decision MUST be `proceed` (forced)
- Set `forced_proceed: true` in `decision_routing.json`
- Write `quality_warning.txt` to the run root with explanation

### Step 5 — Write decision_routing.json
```json
{
  "decision": "proceed" | "refine" | "pivot",
  "confidence": 0.0-1.0,
  "evidence_summary": "...",
  "rollback_target": null | "arc-03-02" | "arc-05-02",
  "rollback_stage_num": null | 8 | 13,
  "pivot_count_after": N,
  "forced_proceed": false | true,
  "quality_warning": false | true
}
```

### Step 6 — Write decision.md
Document the evidence-based justification for the decision, referencing specific numbers from `analysis.md`.

### Step 7 — Handle Routing
- `proceed` → advance to stage 15.5 (arc-06-03 result-claim-gate), then to 15.7 (template-resolve), then to stage 16 (arc-07-01)
- `refine` → initiate rollback to stage 13 (arc-05-02), increment pivot_count
- `pivot` → initiate rollback to stage 8 (arc-03-02), increment pivot_count
