---
name: arc-01-02-problem-decompose
description: Stage 2 — Decompose the research goal into prioritized, testable sub-questions.
metadata:
  category: pipeline-stage
  trigger-keywords: "problem decompose,sub-questions,stage 2"
  applicable-stages: "2"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Break the SMART goal into ≥3 prioritized sub-questions, each with a clear testability clause. The decomposition determines what the literature search covers and what the experiments must validate.

---

## Quality Contract

`problem_tree.md` MUST satisfy all of the following:
- Contains ≥3 numbered sub-questions (SQ1, SQ2, SQ3)
- Each sub-question has a `Question:` field with a specific, answerable claim
- Each sub-question has a `Testability:` clause describing how it can be answered empirically
- Sub-questions collectively cover the full scope of `goal.md`

---

## Inputs
`artifacts/<run_id>/stage-01/goal.md`

---

## Outputs

### `problem_tree.md`
```
# Problem Decomposition

## Research Question
[One sentence from goal.md]

## Sub-Questions

### SQ1: [Priority 1 — highest impact on core claim]
- Question: [specific question]
- Testability: [empirical test description]
- Dependency: [which other SQ must complete first, or "none"]

### SQ2: ...
### SQ3: ...
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| ≥ 3 sub-questions | File contains SQ1, SQ2, SQ3 sections |
| Each has `Question:` | Non-empty question field |
| Each has `Testability:` | Non-empty testability clause |
| Collective scope coverage | All major parts of goal.md addressed |

**Failure**: If any check fails → `E02`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E02` | Fewer than 3 sub-questions, or missing question/testability fields |

---

## Retry Policy
Max retries: **1** — retry with more directive prompting if < 3 sub-questions produced.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Prior Artifact
Read `artifacts/<run_id>/stage-01/goal.md`. Extract: the topic sentence, scope inclusions/exclusions, domain, and falsifiable claim.

### Step 2 — Identify Sub-Questions
Generate ≥3 sub-questions (SQ1, SQ2, SQ3, ...) from the goal. Each must:
- Be answerable through empirical evaluation (not purely theoretical)
- Be specific enough that a concrete experiment could be designed for it
- Not overlap significantly with other SQs (distinct sub-claims)

### Step 3 — Prioritize and Order
Assign priority to each SQ:
- **P0**: Core claim directly tests the falsifiable hypothesis
- **P1**: Necessary preconditions for P0 (e.g., baseline implementations)
- **P2**: Ablations and robustness checks

Specify dependencies: which SQ must be answered before others can begin.

### Step 4 — Write `problem_tree.md`
Write the file following the Outputs template exactly. Each SQ must have `Question:` and `Testability:` fields filled.

### Step 5 — Validate
Run all checks from the Validation table. If < 3 SQs or missing fields → `E02`.
