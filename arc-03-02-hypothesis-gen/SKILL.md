---
name: arc-03-02-hypothesis-gen
description: Stage 8 — Generate ≥2 falsifiable hypotheses grounded in identified research gaps.
metadata:
  category: pipeline-stage
  trigger-keywords: "hypothesis gen,hypotheses,stage 8"
  applicable-stages: "8"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Translate research gaps into concrete, falsifiable hypotheses. Each hypothesis must be specific enough that an experiment can definitively support or refute it.

---

## Quality Contract

`hypotheses.md` MUST satisfy:
- ≥2 hypotheses, each labeled H1, H2, ...
- Each hypothesis has: (a) a grounded gap citation, (b) a quantitative prediction, (c) a falsification condition (what would prove the hypothesis wrong)
- Each hypothesis has a named primary metric
- Hypotheses are testable within the resource budget from `hardware_profile.json`

---

## Inputs
`artifacts/<run_id>/stage-07/synthesis.md`
`artifacts/<run_id>/stage-07/gap_analysis.json`
`artifacts/<run_id>/stage-01/hardware_profile.json`

---

## Outputs

### `hypotheses.md`
```
# Research Hypotheses

## H1: [One-sentence falsifiable claim]
- **Grounded in**: Gap #1 from synthesis
- **Prediction**: [If we do X, we will observe Y — with quantitative terms]
- **Falsification condition**: [We will conclude H1 is false if we observe Z]
- **Primary metric**: [metric name, e.g., accuracy]
- **Success threshold**: [numeric value]

## H2: ...
```

### `hypothesis_index.json`
```json
[
  {"id": "H1", "claim": "...", "primary_metric": "accuracy", "success_threshold": 0.85},
  {"id": "H2", "claim": "...", "primary_metric": "perplexity", "success_threshold": 18.0}
]
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| ≥2 hypotheses | File has H1 and H2 sections |
| Each has `**Prediction**:` | Non-empty quantitative prediction |
| Each has `**Falsification condition**:` | Non-empty falsification clause |
| Each has `**Primary metric**:` | Named metric with numeric threshold |
| Each grounded in a gap | Gap ID cited in each hypothesis |

**Failure**: If any check fails → `E08`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E08` | < 2 hypotheses, or any hypothesis missing prediction/falsification/metric |

---

## Retry Policy
Max retries: **1** — retry with multi-perspective prompting if < 2 falsifiable hypotheses produced.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Synthesis and Hardware Profile
Read `synthesis.md`, `gap_analysis.json`, and `hardware_profile.json`.

### Step 2 — Identify Hypothesis Opportunities
For each gap in `gap_analysis.json`, determine:
- Is there a specific intervention or technique that could address this gap?
- What would a quantitative prediction look like? (e.g., "Method X improves over Y by 10% on metric Z")
- What result would definitively falsify the hypothesis?

### Step 3 — Formulate Hypotheses
Write ≥2 hypotheses (H1, H2, ...). Each must be:
- **Specific**: Names a concrete method, dataset, and metric
- **Quantitative**: Includes a numeric prediction or threshold
- **Falsifiable**: Clearly states what evidence would refute it
- **Grounded**: Explicitly cites the gap ID it addresses
- **Feasible**: Testable within the compute budget in `hardware_profile.json`

Good hypothesis template:
> If we apply [METHOD] to [DATASET], then [METRIC] will improve by [AMOUNT] compared to [BASELINE], because [MECHANISM].

### Step 4 — Assign Metrics and Thresholds
For each hypothesis, assign:
- Primary metric: what to measure (e.g., accuracy, F1, perplexity)
- Success threshold: numeric value that indicates success
- Falsification condition: what result would mean the hypothesis is false

### Step 5 — Write hypotheses.md and hypothesis_index.json
Follow the Outputs templates.

### Step 6 — Validate
Check: ≥2 hypotheses, each with prediction, falsification condition, primary metric, and gap citation. Fail → `E08`.
