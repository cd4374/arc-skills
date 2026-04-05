---
name: arc-08-01-quality-gate
description: Stage 20 (GATE, NONCRITICAL) — Final quality gate assessment. Pipeline blocks here until approved. NONCRITICAL: failure does not block deliverables.
metadata:
  category: pipeline-stage
  trigger-keywords: "quality gate,approval,stage 20"
  applicable-stages: "20"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Assess whether the revised paper meets a minimum quality threshold for submission readiness. This is a GATE stage but is NONCRITICAL — rejection rolls back to stage 16, not abort.

---

## Quality Contract

`quality_report.json` MUST set:
- `quality_score` ∈ [0.0, 1.0]
- `meets_threshold: true` only if all critical issues are resolved:
  - Abstract within 180–220 words
  - ≥2 research gaps identified in Related Work
  - Limitations section explicitly acknowledges ≥1 specific limitation
  - All citation references are verifiable

---

## Inputs
`artifacts/<run_id>/stage-19/paper_revised.md`
`artifacts/<run_id>/stage-14/experiment_summary.json`

---

## Outputs

### `quality_report.json`
```json
{
  "quality_score": 0.78,
  "meets_threshold": true,
  "section_scores": {
    "abstract": 0.85,
    "introduction": 0.72,
    "method": 0.80,
    "experiments": 0.75,
    "results": 0.78,
    "discussion": 0.70,
    "limitations": 0.65
  },
  "critical_issues": [],
  "warnings": []
}
```

### `quality_checklist.md`
```
# Quality Checklist

- [x] Abstract 180-220 words
- [x] Title ≤14 words
- [x] All figure/table refs valid
- [x] Limitations section present
- [x] All citations verifiable
```

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` | `done`, advance to stage 21 |
| `approve` | `done`, advance to stage 21 |
| `reject` | `rejected`, pipeline rolls back to stage 16 |

---

## Rollback Contract
On `reject`: rollback target is **stage 16** (`arc-07-01-paper-outline`). Prior stages 17–19 are versioned before re-execution.

---

## Procedure

### Step 1 — Read Paper and Experiment Summary
Read `paper_revised.md` and `experiment_summary.json`.

### Step 2 — Score Each Section
Assign a score (0.0–1.0) to each section:
- Abstract: word count 180–220, PMR+ structure, no undefined acronyms
- Introduction: contribution clear, ≥3 paragraphs, gap stated
- Related Work: ≥1 full page, synthesizes not just lists
- Method: notation defined, assumptions stated
- Experiments: all hypotheses have supporting evidence
- Results: all claims grounded in experiment_summary metrics
- Discussion: limitations acknowledged
- Limitations: ≥1 specific limitation named

### Step 3 — Check Critical Issues
Verify:
- Abstract 180–220 words
- Related Work identifies ≥2 research gaps
- Limitations subsection acknowledges ≥1 specific limitation
- All citations reference verifiable entries

### Step 4 — Write quality_report.json
Compute `quality_score` as the average of section scores. Set `meets_threshold: true` only if all critical issues are resolved.

### Step 5 — Write quality_checklist.md
Mark each checklist item as complete or incomplete.

### Step 6 — Block at Gate
Set status to `blocked_approval`. Wait for approve/reject. On `approve`: advance to stage 21. On `reject`: initiate rollback to stage 16.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| `quality_report.json` valid | JSON with `quality_score` and `meets_threshold` |
| `quality_checklist.md` exists | File present |
| Abstract within range | 180–220 words |
| Limitations present | ≥1 specific limitation named |

**Failure**: Missing quality report or critical structural issues → `E20`

---

## Noncritical Note
This stage is **NONCRITICAL**. If `skip_noncritical=true` and this stage fails (not rejected), the pipeline continues without blocking deliverables.

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E20` | Quality report missing required fields |

---

## Retry Policy
Max retries: **0** — gate rejection uses rollback path.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`
