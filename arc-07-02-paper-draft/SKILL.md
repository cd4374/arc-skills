---
name: arc-07-02-paper-draft
description: Stage 17 — Draft the complete paper following the approved outline and quality constraints.
metadata:
  category: pipeline-stage
  trigger-keywords: "paper draft,write paper,stage 17"
  applicable-stages: "17"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Write a complete paper draft from the outline. All sections must be written, all figures and tables must be referenced, and all quality constraints must be met.

---

## Quality Contract

`paper_draft.md` MUST satisfy all of:
- Abstract: 180–220 words, starts with context (not "We" or "This paper"), ≤3 numeric values
- All `\ref{}` / `\cref{}` for figures and tables are valid (corresponding labels exist)
- All evidence cited comes from `experiment_summary.json` (no unsubstantiated claims)
- No section contains debugging logs, environment setup, or non-quantitative results as contributions
- Method name (2–5 chars) is used consistently throughout
- Bibliography planning targets at least 30 references, with at least 20% from the last 5 years unless the topic genuinely lacks recent primary literature
- Every planned figure is tied to a real upstream artifact or metric before the draft is finalized
- Every planned figure carries a provenance declaration in the draft using a machine-checkable comment such as `<!-- fig-src: stage-14/experiment_summary.json > metric_name -->`

---

## Inputs
`artifacts/<run_id>/stage-16/outline.md`
`artifacts/<run_id>/stage-14/analysis.md`
`artifacts/<run_id>/stage-14/results_table.tex`
`artifacts/<run_id>/stage-14/experiment_summary.json`
`artifacts/<run_id>/template/template_manifest.json`

---

## Outputs

### `paper_draft.md`
Complete paper with all sections from the outline filled in.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Abstract 180–220 words | Strict enforced range |
| Abstract format | First sentence is context, not "We"/"This paper" |
| Figure/table refs valid | All `\ref{}`/`\cref{}` have corresponding labels |
| No false claims | All metric claims are from experiment_summary |
| All sections present | Sections match outline |

**Failure**: If abstract < 180 or > 220 words → `E17` (other violations → warnings only)

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E17` | Abstract word count out of range, or figure references invalid |

---

## Retry Policy
Max retries: **1** — retry specifically if abstract word count is wrong.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read All Inputs
Read `outline.md`, `analysis.md`, `results_table.tex`, `experiment_summary.json`, and `template_manifest.json`. Note:
- The contribution claim (one sentence from the abstract plan)
- All quantitative results to include
- Figure/table labels from `results_table.tex`
- The method name (2–5 chars) to use consistently
- The resolved publication type, target venue, submission mode, any required/forbidden sections, and any hard figure-count/style rules

### Step 2 — Draft Front Matter

**Abstract** (180–220 words, strict):
1. Count words as you write — stop at 220
2. Structure: Problem sentence → Method sentence → Results sentence → Significance sentence
3. Include exactly one concrete quantitative result
4. Do NOT start with "We" or "This paper"
5. Do NOT include citations or undefined acronyms

**Title** (≤14 words):
- Include the key technique, task, and dataset class
- Avoid generic: "A Study on X" — use: "Method X for Task Y on Dataset Z"

### Step 3 — Draft Introduction

Target: 500–900 words, 3–4 paragraphs:
1. Hook: Open with the specific problem/motivation (1–2 sentences)
2. Gap: State what existing methods fail to address — be specific
3. Approach: Brief overview of the proposed method before details
4. Contributions: Numbered list of 2–4 falsifiable contributions
5. Roadmap: "The rest of this paper..." paragraph

Apply De-AI patterns throughout (see Step 5).

### Step 4 — Draft Related Work

Target: ≥600 words, organized by category:
1. Read the literature cards from `cards/` (stage-06)
2. Organize papers by: (a) shared method, (b) shared task, or (c) shared assumption
3. For each category: 1 paragraph summarizing the line + 1–2 sentences positioning this paper
4. Do NOT write paper-by-paper summaries — synthesize
5. End each paragraph with how this paper relates/differs
6. Plan the bibliography so the paper converges toward at least 30 verified references with at least 20% published within the last 5 years where the topic permits

### Step 5 — Draft Method

Target: 800–1500 words:
1. Define notation early (reference `math_commands.tex`)
2. Describe the proposed approach in logical order
3. Include algorithm pseudocode if applicable (`algorithm2e` or `algorithmic` environment)
4. State all assumptions explicitly
5. Insert `results_table.tex` content into the appropriate subsection

### Step 6 — Draft Experiments

Target: 1200–2000 words:
1. **Setup** (datasets, baselines, metrics, implementation)
2. **Main Results** — lead with the strongest result, cite `experiment_summary.json`
3. **Ablation Studies** — each ablation component gets a paragraph with quantitative evidence
4. Every claim in the Introduction must have supporting evidence here
5. Ensure planned figures are evidence-bearing, not decorative, and satisfy the resolved minimum figure count and style constraints

### Step 6a — Declare Figure Provenance Before Export
For every planned figure/table in the draft:
1. Identify the exact upstream artifact and metric that justifies it
2. Add a machine-checkable provenance marker near the planned figure reference using a comment such as `<!-- fig-src: stage-14/experiment_summary.json > metric_name -->`
3. Do not keep any planned figure whose source artifact or metric cannot be named concretely
4. Treat unsupported or decorative planned figures as draft failures that must be fixed before Stage 22 export

### Step 7 — Draft Conclusion + Broader Impact

Target: 200–400 words:
1. Summarize contributions (rephrase, don't copy from Introduction)
2. Acknowledge ≥1 specific limitation honestly
3. State 1–2 concrete future directions
4. Ethics statement or broader-impact language only if required by the resolved venue/journal template rules
5. Respect submission mode constraints (for example, no author-identifying prose in anonymous conference mode)

### Step 8 — De-AI Polish Pass (full draft review)

After all sections written, scan the complete draft:

**Forbidden word scan — replace each occurrence:**
- delve → examine/investigate | pivotal → central/key | landscape → field/area | tapestry → framework | underscore → highlight/show | noteworthy → notable | intriguingly → surprisingly

**Significance inflation:**
groundbreaking → substantial | revolutionary → novel | paradigm-shifting → influential

**Filler removal:**
"It is worth noting that" → integrate or remove | "Importantly," → remove or restructure | "Notably," → remove

**Sentence structure fixes:**
- Subject and verb > 10 words apart → restructure
- New information before familiar context → reorder
- Most important information at beginning → move to end
- Multiple ideas in one paragraph → split

### Step 9 — Reverse Outline Test

1. Extract the first sentence of every paragraph across all sections
2. Read them in sequence — they must form a coherent narrative
3. Check: every claim from `experiment_summary.json` appears somewhere in the paper
4. Check: every figure/table is referenced in the main text with its contribution stated
5. Fix any paragraph whose topic sentence does not advance the story

### Step 10 — Validate and Finalize

Run all Validation checks:
- Abstract 180–220 words (count manually)
- Abstract starts with context, not "We"/"This paper"
- All `\ref{}`/`\cref{}` resolve to existing labels
- All metric claims are from `experiment_summary.json` (no fabrication)

Fail → `E17` for abstract word count or invalid figure references.
