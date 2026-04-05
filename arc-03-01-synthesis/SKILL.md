---
name: arc-03-01-synthesis
description: Stage 7 — Cluster knowledge cards into topic themes and identify ≥2 concrete research gaps.
metadata:
  category: pipeline-stage
  trigger-keywords: "synthesis,gaps,stage 7"
  applicable-stages: "7"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Organize the literature into clusters and surface the gaps between existing work and the research goal. Gaps become the foundation for hypotheses.

---

## Quality Contract

`synthesis.md` MUST satisfy:
- Papers are grouped into ≥2 thematic clusters, each with a named theme and shared-method or shared-task rationale
- ≥2 research gaps are identified, each with: (a) what is missing, (b) why it matters, (c) evidence from cards supporting the gap claim
- Gap claims are grounded in specific card evidence (citation to card paper_id)

---

## Inputs
`artifacts/<run_id>/stage-06/cards/`
`artifacts/<run_id>/stage-06/cards_index.json`

---

## Outputs

### `synthesis.md`
```
# Knowledge Synthesis

## Topic Clusters

### Cluster 1: [Theme Name]
- Papers: [card IDs]
- Shared method/theme: [what unites these papers]
- Consistent findings: [what results appear across multiple papers]

### Cluster 2: ...

## Research Gaps

### Gap 1: [Specific gap]
- What is missing: [precise description]
- Why it matters: [impact on the field]
- Evidence: [card IDs that reveal this gap]

### Gap 2: ...
```

### `gap_analysis.json`
```json
[
  {"gap_id": 1, "title": "...", "evidence_papers": ["W123", "W456"]}
]
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| ≥2 clusters | `synthesis.md` has ≥2 named `### Cluster` sections |
| ≥2 gaps | `synthesis.md` has ≥2 named `### Gap` sections |
| Each gap has evidence | Each gap lists ≥1 supporting `paper_id` |
| `gap_analysis.json` exists | Valid JSON array matching gap sections |

**Failure**: If any check fails → `E07`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E07` | < 2 clusters, or < 2 gaps with evidence |

---

## Retry Policy
Max retries: **1** — retry with more structured synthesis prompting if clustering produces no useful organization.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read All Knowledge Cards
Read all files in `cards/`. Build a mental map of: core contributions, methods used, datasets evaluated, and reported limitations.

### Step 2 — Cluster Papers by Theme
Group papers that share: a similar method architecture, a similar task/dataset, or a similar theoretical framing. Each cluster should have ≥2 papers. Name each cluster with a descriptive theme (e.g., "Transformer-based Methods", "Graph Neural Networks for Recommendation").

For each cluster, note:
- Which papers belong and why they cluster together
- Consistent findings (results that appear across multiple papers in the cluster)

### Step 3 — Identify Research Gaps
Compare each cluster against the sub-questions from stage-02 and against the falsifiable claim from stage-01. Look for:
- What existing methods fail to address
- What datasets are missing from evaluations
- What theoretical guarantees are not proven
- What hyperparameters have not been ablated

Identify ≥2 gaps. Each gap needs:
- **What is missing**: precise description
- **Why it matters**: impact on field or on the current research goal
- **Evidence**: specific card IDs that support this gap claim

### Step 4 — Write synthesis.md
Follow the Outputs template exactly.

### Step 5 — Write gap_analysis.json
JSON array matching the gap sections in synthesis.md.

### Step 6 — Validate
Check ≥2 clusters, ≥2 gaps, each gap has evidence. Fail → `E07`.
