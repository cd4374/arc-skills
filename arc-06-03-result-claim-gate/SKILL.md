---
name: arc-06-03-result-claim-gate
description: Stage 15.5 (GATE) — Freeze the claim-scope contract before template resolution and paper outlining. Converts experiment evidence into an explicit allowed-claims contract so the paper cannot outrun the results.
metadata:
  category: pipeline-stage
  trigger-keywords: "result claim gate,claim scope,allowed claims,stage 15.5"
  applicable-stages: "15.5"
  priority: "1"
  version: "1.1"
  author: researchclaw
---

## Purpose
Derive an explicit claim-scope contract from the completed experiments before paper writing starts. This stage prevents the writing flow from turning weak, partial, or ambiguous results into broad paper claims.

---

## Quality Contract

`claim_scope_report.json` MUST:
- Enumerate every intended top-level paper claim with a stable `claim_id`
- Classify each claim as `allowed`, `allowed_with_caveat`, or `disallowed`
- Record at least one evidence pointer into `experiment_summary.json` or `analysis.md`
- Record mandatory wording constraints for every `allowed_with_caveat` claim
- Set `claim_scope_pass: true` only if every core contribution claim is either `allowed` or `allowed_with_caveat`

Hard constraints:
- `disallowed` claims MUST NOT appear in stage 16, 17, 19, 22, or 25 outputs
- `allowed_with_caveat` claims MUST carry their caveat wording forward into writing
- If stage 15 was a forced proceed, `forced_scope_narrowing: true` MUST be recorded
- The gate MUST block (`claim_scope_pass: false`) if any core claim is unsupported

---

## Inputs
`artifacts/<run_id>/stage-14/analysis.md`
`artifacts/<run_id>/stage-14/experiment_summary.json`
`artifacts/<run_id>/stage-15/decision.md`
`artifacts/<run_id>/stage-15/decision_routing.json`

---

## Outputs

### `artifacts/<run_id>/stage-15b/claim_scope_report.json`
```json
{
  "claim_scope_pass": true,
  "forced_scope_narrowing": false,
  "claims": [
    {
      "claim_id": "CS01",
      "claim_text": "Method X improves macro-F1 over the baseline on dataset D.",
      "status": "allowed",
      "evidence": [
        {
          "artifact": "stage-14/experiment_summary.json",
          "pointer": "primary_metrics.h1_main"
        }
      ],
      "required_caveat": null
    },
    {
      "claim_id": "CS02",
      "claim_text": "Method X generalizes across datasets.",
      "status": "allowed_with_caveat",
      "evidence": [
        {
          "artifact": "stage-14/analysis.md",
          "pointer": "supported_on_single_dataset_only"
        }
      ],
      "required_caveat": "Limit the scope to the evaluated datasets and avoid universal generalization language."
    }
  ],
  "disallowed_claim_ids": [],
  "warnings": []
}
```

### `artifacts/<run_id>/stage-15b/allowed_claims.md`
Human-readable summary of:
- allowed claims
- allowed claims that require explicit caveats
- disallowed claims that the paper must avoid

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `claim_scope_pass=true` | `done`, advance to stage 15.7 (template-resolve) |
| `approve` and `claim_scope_pass=true` | `done`, advance to stage 15.7 |
| `reject` or `claim_scope_pass=false` | `rejected`, rollback to stage 15 |

---

## Rollback Contract
On `reject` or `claim_scope_pass=false`: rollback target is **stage 15** (`arc-06-02-research-decision`).

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`

---

## Procedure

### Step 1 — Read Result Artifacts and Decision Context
Read `analysis.md`, `experiment_summary.json`, `decision.md`, and `decision_routing.json`.

### Step 2 — Enumerate Intended Paper Claims
Extract the claims that stage 16 is likely to elevate into the title, abstract, introduction, and contributions list. Include:
- main performance claims
- robustness or generalization claims
- novelty or superiority claims tied to empirical evidence

### Step 3 — Compare Claims Against Evidence
For each claim:
- mark `allowed` if the evidence is strong and direct
- mark `allowed_with_caveat` if the evidence is partial, narrow, or conditional
- mark `disallowed` if the evidence is missing, contradicted, or too weak for paper-level framing

### Step 4 — Write Wording Constraints
For each `allowed_with_caveat` claim, write the exact scope restriction needed downstream. Examples:
- evaluated datasets only
- pilot evidence only
- no causal language
- no broad generalization language

### Step 5 — Handle Forced Proceed Safely
If `decision_routing.json` shows `forced_proceed: true`, do NOT treat that as permission to keep unsupported claims. Instead:
- set `forced_scope_narrowing: true`
- narrow the allowed claim set aggressively
- record any blocked claim in `disallowed_claim_ids`

### Step 6 — Write Outputs
Write `claim_scope_report.json` and `allowed_claims.md` under `artifacts/<run_id>/stage-15b/`.

### Step 7 — Gate Decision
If any core claim is unsupported (not `allowed` or `allowed_with_caveat`), set `claim_scope_pass=false` and fail with `E15B_UNSUPPORTED_CORE_CLAIM`. Otherwise block for approval and then advance to stage 15.7.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Report exists | `claim_scope_report.json` exists and valid JSON |
| Claim coverage | Every intended top-level claim has a `claim_id` and a status |
| Evidence pointers present | Every claim has at least one evidence pointer |
| Caveats explicit | Every `allowed_with_caveat` claim has non-empty `required_caveat` |
| Gate pass flag | `claim_scope_pass == true` |

**Failure**: Missing claim-scope contract, evidence linkage, or unsupported core claim → `E15B`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E15B` | Claim-scope report missing, malformed, or not fully classified |
| `E15B_UNSUPPORTED_CORE_CLAIM` | A core contribution claim is unsupported but not marked `disallowed` |
| `E15B_MISSING_CAVEAT` | An `allowed_with_caveat` claim lacks explicit wording constraints |

---

## Retry Policy
Max retries: **0** — gate rejection uses rollback path.

---

## State Transition
`pending` → `running` → `done` | `failed`
