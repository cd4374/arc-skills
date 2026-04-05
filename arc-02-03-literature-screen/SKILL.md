---
name: arc-02-03-literature-screen
description: Stage 5 (GATE) — Screen candidates for relevance and quality. Pipeline blocks here until approved.
metadata:
  category: pipeline-stage
  trigger-keywords: "literature screen,gate,stage 5"
  applicable-stages: "5"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Filter candidates to a high-quality shortlist. This is a GATE stage — the pipeline cannot advance past this point without explicit approval or `auto_approve_gates=true`.

---

## Quality Contract

`shortlist.jsonl` MUST satisfy:
- Contains **≥15 papers**
- Each paper has a `relevance_score` (0.0–1.0) and `screen_pass: true`
- Screening is documented in `screening_report.md` with pass/reject counts and methodology

---

## Inputs
`artifacts/<run_id>/stage-04/candidates.jsonl`

---

## Outputs

### `shortlist.jsonl`
One JSON object per line:
```json
{"id":"openalex:W123456789","title":"...","relevance_score":0.82,"quality_score":0.74,"screen_pass":true,"reason":"..."}
```

### `screening_report.md`
```
# Screening Report

## Methodology
[How relevance and quality were assessed]

## Results
- Total candidates: N
- Passed relevance: N
- Passed quality: N
- Final shortlist: N
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| ≥15 papers | `shortlist.jsonl` line count ≥ 15 |
| All have `screen_pass: true` | Every line has `screen_pass: true` |
| Has relevance scores | All lines have `relevance_score` field |
| `screening_report.md` exists | File present with methodology section |

**Failure**: If any check fails → `E05`

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` | `done`, advance to stage 6 |
| `approve` | `done`, advance to stage 6 |
| `reject` | `rejected`, pipeline rolls back to stage 4 |

---

## Rollback Contract
On `reject`: rollback target is **stage 4** (`arc-02-02-literature-collect`). Re-run from collection with the same or revised search plan.

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E05` | < 15 papers, or missing screening report, or `screen_pass` field missing |

---

## Retry Policy
Max retries: **0** — gate rejection is not a retry. Use the `reject` → `rollback` path instead.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`

---

## Procedure

### Step 1 — Read Candidates
Read `candidates.jsonl` from stage-04. This is the input corpus to screen.

### Step 2 — Score Each Candidate
For each candidate, assess:
- **Relevance score** (0.0–1.0): How directly does this paper address any sub-question (SQ1, SQ2, ...)? Score based on title + abstract match.
- **Quality score** (0.0–1.0): Is this from a credible venue (top-tier conference/journal), well-cited, or from authors with strong track record?

Papers pass screening if `relevance_score >= 0.5` AND `quality_score >= 0.4`.

### Step 3 — Select Shortlist
Select all passing papers (≥15 required). If fewer than 15 pass, lower the quality threshold slightly but document the reason. Do NOT force a shortlist to reach 15 — if genuinely < 15 relevant papers exist, note this limitation in the report.

### Step 4 — Write shortlist.jsonl
One JSON line per passing paper. Include `relevance_score`, `quality_score`, `screen_pass: true`, and a `reason` field (1–2 sentence justification for inclusion).

### Step 5 — Write screening_report.md
Document methodology, total candidates, pass/reject counts, and final shortlist size.

### Step 6 — Block at Gate
Set status to `blocked_approval`. Wait for approve/reject. On `approve`: advance to stage 6. On `reject`: initiate rollback to stage 4.
