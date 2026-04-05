---
name: arc-07-03-peer-review
description: Stage 18 — Generate ≥2 distinct simulated peer reviews with evidence-based critiques.
metadata:
  category: pipeline-stage
  trigger-keywords: "peer review,review draft,stage 18"
  applicable-stages: "18"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Stress-test the paper draft by generating reviews from distinct reviewer personas. Reviews must be evidence-based and identify specific weaknesses.

---

## Quality Contract

`reviews.md` MUST satisfy:
- ≥2 distinct reviewer personas (different expertise/perspectives)
- Each review has ≥1 major concern with a specific reference to a section or claim in `paper_draft.md`
- Each review gives a clear Accept / Borderline / Reject recommendation
- Reviews do not duplicate each other's concerns (distinct critique angles)

---

## Inputs
`artifacts/<run_id>/stage-17/paper_draft.md`

---

## Outputs

### `reviews.md`
```
# Simulated Peer Reviews

## Reviewer 1: [Profile — e.g., Theory-focused ML researcher]
### Summary
[Paragraph summary of the paper]
### Major Concerns
- [Concern with specific reference: "Section 3.2 claims...but..."]
### Minor Concerns
- [Concern]
### Questions for Authors
- [Specific question]
### Recommendation: [Accept / Borderline / Reject]

---

## Reviewer 2: [Profile — e.g., Applied researcher focused on reproducibility]
...
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| ≥2 reviews | File has ≥2 reviewer sections |
| Distinct personas | Reviewer profiles differ in expertise/focus |
| ≥1 major concern per review | Each review has ≥1 item under Major Concerns |
| Specific references | Concerns reference actual paper content (not generic) |
| Recommendation present | Each review has a clear recommendation |

**Failure**: If < 2 distinct reviews or any review lacks a major concern → `E18`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E18` | < 2 reviews, or reviews lack specific major concerns |

---

## Retry Policy
Max retries: **1** — retry if < 2 distinct reviews could be generated.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read the Paper Draft
Read `paper_draft.md` in full. Note: the contribution claim, methodology, experimental results, limitations, and writing quality.

### Step 2 — Design Reviewer Personas
Design ≥2 distinct reviewer profiles:
- **Persona 1** (e.g., Theory-focused ML researcher): focuses on methodological soundness, novelty, theoretical guarantees
- **Persona 2** (e.g., Applied researcher / reproducibility advocate): focuses on experimental setup, baselines, statistical significance
- Additional personas could focus on: writing clarity, ablation completeness, related work coverage

Each persona should have a distinct expertise that leads to different critique angles.

### Step 3 — Write Reviews
For each reviewer persona:
1. Read the paper from that reviewer's perspective
2. Identify 1–2 MAJOR concerns: specific claims in the paper that are weak, unsubstantiated, or methodologically flawed. Reference exact sections or claims.
3. Identify 1–2 MINOR concerns: writing clarity issues, missing details, figure/table descriptions
4. Write 1–2 questions the reviewer would ask the authors
5. Assign a recommendation: Accept / Borderline / Reject

**Major concern template**: "Section X claims [specific claim], but [what is wrong or missing]. The paper should [specific fix]."

### Step 4 — Verify Non-Duplication
Check that the major concerns from different reviewers are NOT duplicates of each other. Each reviewer should have at least one distinct critique angle.

### Step 5 — Write reviews.md
Follow the Outputs template exactly. Preserve the persona profiles clearly at the top of each review section.

### Step 6 — Validate
Check: ≥2 reviewers, distinct personas, ≥1 major concern per review, specific section references, clear recommendation. Fail → `E18`.
