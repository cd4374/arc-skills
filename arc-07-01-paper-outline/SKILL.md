---
name: arc-07-01-paper-outline
description: Stage 16 — Build a structured paper outline with all sections, word count targets, and evidence mapping.
metadata:
  category: pipeline-stage
  trigger-keywords: "paper outline,stage 16"
  applicable-stages: "16"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Transform the experimental results into a structured paper outline. The outline maps every hypothesis, ablation result, and limitation to a specific section and paragraph in the paper.

---

## Quality Contract

`outline.md` MUST satisfy:
- Abstract is 180–220 words with PMR+ structure (Problem, Method, Results, significance)
- Title is ≤14 words and defines a short method name (2–5 characters)
- All major sections are present: Introduction, Related Work, Method, Experiments, Results, Discussion, Conclusion, Broader Impact
- Each hypothesis (H1, H2, ...) maps to a dedicated paragraph in the Experiments section
- Limitations are explicitly addressed in a dedicated subsection

---

## Inputs
`artifacts/<run_id>/stage-14/analysis.md`
`artifacts/<run_id>/stage-15/decision.md`
`artifacts/<run_id>/stage-14/experiment_summary.json`

---

## Outputs

### `outline.md`
```
# Paper Outline

## Metadata
- Title: [≤14 words, defines short method name]
- Method name: [2-5 character identifier]

## Abstract (PMR+, 180-220 words)
[Problem sentence] [Method sentence] [Results sentence] [Significance sentence]

## 1. Introduction (800-1000 words)
## 2. Related Work (600-800 words)
## 3. Method (1000-1500 words)
  - 3.1 Background
  - 3.2 Proposed Method
## 4. Experiments (800-1200 words)
  - 4.1 Setup
  - 4.2 Main Results [H1 evidence]
  - 4.3 Ablation Studies [H2 evidence]
## 5. Results (600-800 words)
## 6. Discussion (400-600 words)
  - 6.1 Limitations (200-300 words)
## 7. Conclusion (200-300 words)
## Broader Impact (200-400 words)
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Abstract 180–220 words | Word count within range |
| Title ≤14 words | Count ≤ 14 |
| All sections present | All numbered sections and Broader Impact present |
| Hypothesis mapping | Each H1, H2, ... appears in Experiments section |
| Limitations subsection | Dedicated Limitations section present |

**Failure**: If abstract word count < 180 or > 220, or title > 14 words, or any section missing → `E16`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E16` | Abstract out of range, title too long, or required sections missing |

---

## Retry Policy
Max retries: **1** — retry if abstract word count is wrong.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Analysis and Decision
Read `analysis.md`, `decision.md`, and `experiment_summary.json`.

### Step 2 — Draft Abstract
Write the abstract following PMR+ structure:
- **Problem** (1 sentence): What is the specific gap or limitation in the field?
- **Method** (1 sentence): What does the proposed approach do?
- **Results** (1 sentence): What quantitative improvement was achieved?
- **Significance** (1 sentence): Why does this matter for the field?

Target: 180–220 words. Count words manually. Abstract must be self-contained (understandable without reading the rest of the paper).

### Step 3 — Create Title and Method Name
- **Title**: ≤14 words, specific, informative. Avoid generic titles like "A Study on X". Include the key technique.
- **Method name**: Assign a short identifier (2–5 characters) for the proposed method (used in figures, tables, and throughout the paper).

### Step 4 — Map Hypotheses to Experiments
From `analysis.md`, note which hypothesis each experimental result supports. Each hypothesis should map to a dedicated paragraph or subsection in the Experiments section.

### Step 5 — Build Full Outline
Create `outline.md` following the Outputs template. Assign word count targets to each section. Ensure:
- Every H1, H2, ... from `hypotheses.md` appears as evidence in the Experiments section
- A Limitations subsection exists (even if brief — reviewers expect this)
- Broader Impact / Ethics Statement is present (required for many venues)

### Step 6 — Validate
Count abstract words. Count title words. Verify all sections present. Fail → `E16`.
