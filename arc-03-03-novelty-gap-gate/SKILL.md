---
name: arc-03-03-novelty-gap-gate
description: Stage 7.5 (GATE — BLOCKING) — Verify that the proposed research gaps remain genuinely open via mandatory Codex MCP adversarial review before hypothesis generation proceeds. BLOCKS if Codex MCP unavailable.
metadata:
  category: pipeline-stage
  trigger-keywords: "novelty gap gate,novelty check,research gap,stage 7.5,codex mcp"
  applicable-stages: "7.5"
  priority: "1"
  version: "1.1"
  author: researchclaw
  blocking: true
  requires_codex_mcp: true
---

## Purpose
Block weak or already-closed research directions before they become hypotheses and experiments. This stage turns the synthesized gap analysis into an explicit novelty decision with evidence pointers and a **mandatory** adversarial check via Codex MCP.

**CRITICAL**: Codex MCP external review is **REQUIRED**. Pipeline **BLOCKS** if unavailable.

---

## Quality Contract

`novelty_gap_report.json` MUST:
- Enumerate each candidate gap with a stable `gap_id`
- Record the closest prior work from the verified literature corpus
- Classify each gap as `novel`, `partially_novel`, or `not_novel`
- Record a novelty score from 1–10
- Record `external_review.backend: "codex_mcp"` with `available: true`
- Set `novelty_gate_pass: true` only if every core gap is `novel` or `partially_novel`

Hard constraints:
- `not_novel` core gaps MUST block progression to hypothesis generation
- Any `partially_novel` gap MUST carry a scope-narrowing note into the next stage
- **Codex MCP adversarial review is REQUIRED** — if unavailable, stage **FAILS** with `E07B_CODEX_MCP_REQUIRED`

---

## Inputs
`artifacts/<run_id>/stage-07/synthesis.md`
`artifacts/<run_id>/stage-07/gap_analysis.json`
`artifacts/<run_id>/stage-04/candidates.jsonl`

---

## Outputs

### `artifacts/<run_id>/stage-07b/novelty_gap_report.json`
```json
{
  "novelty_gate_pass": true,
  "external_review": {
    "backend": "codex_mcp",
    "available": true,
    "reviewer_model": "gpt-5.4"
  },
  "gaps": [
    {
      "gap_id": "G1",
      "gap_text": "Current methods do not evaluate robustness under distribution shift X.",
      "closest_prior_work": [
        {
          "title": "Author et al. 2024",
          "reason": "addresses adjacent setting but not shift X"
        }
      ],
      "novelty_score": 7,
      "status": "novel",
      "scope_note": null
    }
  ],
  "blocking_issues": []
}
```

**Note**: `external_review.available` must be `true`. If Codex MCP is unavailable, the stage fails before writing this file.

### `artifacts/<run_id>/stage-07b/novelty_gap_summary.md`
Human-readable summary of which gaps remain open, which are only partially open, and which are blocked.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| All gaps classified | Every candidate gap has a `gap_id`, score, and status |
| Prior work linked | Every gap cites at least one closest prior work entry |
| **Codex MCP review completed** | `external_review.backend == "codex_mcp"` AND `available == true` |
| No blocked core gaps | No core gap is marked `not_novel` |
| Gate pass flag | `novelty_gate_pass == true` |

**Failure**: 
- If any core gap is `not_novel` → `E07B_GAP_NOT_NOVEL`
- If Codex MCP unavailable → `E07B_CODEX_MCP_REQUIRED`
- If report incomplete → `E07B`

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `novelty_gate_pass=true` | `done`, advance to stage 8 |
| `approve` and `novelty_gate_pass=true` | `done`, advance to stage 8 |
| `reject` or `novelty_gate_pass=false` | `rejected`, rollback to stage 7 |

---

## Rollback Contract
On `reject` or `novelty_gate_pass=false`: rollback target is **stage 7** (`arc-03-01-synthesis`). Revise the identified gaps before generating hypotheses.

---

## Error Codes

| Code | Trigger | Retryable |
|------|---------|-----------|
| `E07B` | Novelty gap report missing, malformed, or indicates a blocked core gap | No |
| `E07B_GAP_NOT_NOVEL` | At least one core gap is already addressed by prior work | No |
| `E07B_CODEX_MCP_REQUIRED` | **Codex MCP adversarial review is required but unavailable** — pipeline BLOCKED | **No** |
| `E07B_INDEPENDENCE_VIOLATION` | Reviewer detected as non-independent | **No** |

---

## Retry Policy
Max retries: **0** — gate rejection uses rollback path.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`

---

## Procedure

### Step 1 — Read Synthesis and Gap Analysis
Read `synthesis.md`, `gap_analysis.json`, and the verified literature corpus from `candidates.jsonl`.

### Step 2 — Enumerate Candidate Gaps
Extract each proposed gap that is intended to support later hypotheses. Distinguish core gaps from optional extensions.

### Step 3 — Link Closest Prior Work
For each gap, identify the closest prior verified papers in the stage-04 corpus and record why they do or do not close the gap.

### Step 4 — Run Adversarial Novelty Review (REQUIRED)

**Codex MCP is MANDATORY** for adversarial novelty review.

1. **Attempt to initialize Codex MCP**:
   - Verify MCP server is running
   - Confirm GPT-5.4 model access
   - Ensure independence from executor context

2. **If Codex MCP is available**:
   - Run adversarial review of each candidate gap
   - Record `external_review.backend: "codex_mcp"`, `available: true`
   - Incorporate reviewer assessment into novelty scores

3. **If Codex MCP is UNAVAILABLE**:
   - **FAIL immediately** with `E07B_CODEX_MCP_REQUIRED`
   - Write `CODEX_MCP_ERROR.md` with setup instructions
   - **BLOCK pipeline** — do not proceed to hypothesis generation without external validation
   - **NO DEGRADED MODE** — local novelty assessment is insufficient

The adversarial reviewer must explicitly confirm that gaps remain genuinely open given the verified literature corpus.

### Step 5 — Score and Classify
For each gap:
- `novel` — prior work does not close the gap
- `partially_novel` — adjacent prior work exists, but scope remains open
- `not_novel` — prior work already closes the gap sufficiently

Assign `novelty_score` 1–10 and a `scope_note` for any `partially_novel` gap.

### Step 6 — Write Outputs
Write `novelty_gap_report.json` and `novelty_gap_summary.md` under `stage-07b/`.

### Step 7 — Gate Decision
If any core gap is `not_novel`, set `novelty_gate_pass=false` and fail with `E07B_GAP_NOT_NOVEL`. Otherwise block for approval, then advance to stage 8.
