---
name: arc-09-02-paper-polish
description: Stage 25 — Dedicated paper polish stage. Applies De-AI Polish Pass, Reverse Outline Test, and final quality checks to regenerate the canonical polished package for trust-gate validation.
metadata:
  category: pipeline-stage
  trigger-keywords: "paper polish,de-ai,reverse outline,stage 25"
  applicable-stages: "25"
  priority: "1"
  version: "4.1"
  author: researchclaw
---

## Purpose

Polish the stage-22 export package to remove AI writing patterns, fix logical narrative gaps, enforce section word counts, and regenerate the final canonical package used by trust gates. This is a **critical stage** — output MUST pass all quality checks or the pipeline is blocked.

---

## Quality Contract

`paper_polished.md` MUST satisfy ALL of:

### De-AI Polish Pass
- No forbidden words: delve, pivotal, landscape, tapestry, underscore, noteworthy, intriguingly
- No significance inflation: groundbreaking, revolutionary, paradigm-shifting (use measured language)
- No formulaic transitions: "In this section, we..."
- No filler phrases: "It is worth noting that", "Importantly,", "Notably,"
- Subject and verb close together; familiar context before new information
- Most important information near end of sentence
- Each paragraph does exactly one job
- Verbs for actions, not nominalized nouns

### Reverse Outline Test
- Topic sentences (first sentence of every paragraph) form a coherent narrative in sequence
- Every claim from the Claims-Evidence Matrix appears in the paper
- Every experiment/figure supports a stated claim
- No paragraph whose topic sentence does not advance the story

### Structural Requirements
- Abstract: 180–220 words (stage 17 contract still enforced)
- Introduction: 1–1.5 pages, ends with roadmap
- Related Work: ≥1 full page (3–4 substantive paragraphs)
- Method: 1.5–2 pages
- Experiments: 2.5–3 pages
- Conclusion: ~0.5 pages
- All `\ref{}`/`\cref{}` have corresponding labels
- All citation commands have BibTeX entries
- No TODO/FIXME/XXX markers
- No `[VERIFY]` markers remaining unchecked
- No stale section files (every .tex in `sections/` is `\input`ed)

---

## Inputs
`artifacts/<run_id>/stage-22/paper_final.md`
`artifacts/<run_id>/stage-16/outline.md`
`artifacts/<run_id>/stage-14/experiment_summary.json`
`artifacts/<run_id>/stage-23/verification_report.json`
`artifacts/<run_id>/stage-23/references_verified.bib`
`artifacts/<run_id>/stage-22/reproducibility_report.json`
`artifacts/<run_id>/stage-24/AUTO_REVIEW.md` (reviewer feedback if available)

---

## Outputs

### `artifacts/<run_id>/stage-25/paper_polished.md`
Final polished paper ready for trust-gate validation.

### `artifacts/<run_id>/stage-25/main.tex`
Polished LaTeX source regenerated from `paper_polished.md` using the same package schema as stage 22.

### `artifacts/<run_id>/stage-25/references.bib`
Final bibliography derived from `stage-23/references_verified.bib`, pruned to the citations still used by the polished paper. No new citations may be introduced at this stage without re-verification.

### `artifacts/<run_id>/stage-25/main.pdf`
Compiled PDF produced from the polished LaTeX package.

### `artifacts/<run_id>/stage-25/compile_report.json`
Compile summary for the polished package.

### `artifacts/<run_id>/stage-25/manifest.json`
Artifact manifest for the polished package with paths and checksums, including `reproducibility_report.json`, `code/`, and `charts/` integrity coverage.

### `artifacts/<run_id>/stage-25/deliverables/`
Final polished submission bundle for downstream trust gates. This directory mirrors the stage-22 package structure, but with `paper_polished.md` as the canonical paper source, `references_verified.bib` propagated into the final `references.bib`, and `reproducibility_report.json` preserved.

### `artifacts/<run_id>/stage-25/polish_report.json`
```json
{
  "de_ai_patterns_fixed": 12,
  "reverse_outline_gaps_fixed": 3,
  "word_counts": {
    "abstract": 198,
    "introduction": 680,
    "related_work": 820,
    "method": 1100,
    "experiments": 1650,
    "conclusion": 280
  },
  "structural_issues": [],
  "citation_issues": [],
  "pass": true,
  "warnings": ["Minor: one figure caption could be more descriptive"]
}
```

---

## Procedure

### Step 1 — De-AI Polish Pass

Scan `artifacts/<run_id>/stage-22/paper_final.md` for the following patterns and fix each, then write the revised result to `artifacts/<run_id>/stage-25/paper_polished.md`:

**Forbidden word replacement:**
| Replace | With |
|---------|------|
| delve | examine, investigate, explore |
| pivotal | central, key, critical |
| landscape | field, area, setting |
| tapestry | framework, structure |
| underscore | highlight, show, demonstrate |
| noteworthy | notable, important |
| intriguingly | surprisingly, notably |

**Significance inflation removal:**
- groundbreaking → substantial / significant
- revolutionary → novel / new
- paradigm-shifting → influential / important
- seminal → well-known / established

**Filler phrase removal:**
- "It is worth noting that" → remove or integrate
- "Importantly," → remove or vary structure
- "Notably," → remove or replace with specific observation

**Sentence structure fixes:**
- Subject-verb distance > 10 words → restructure to bring closer
- New information before familiar context → reorder sentences
- Most important info at beginning → move to end
- Multiple ideas in one paragraph → split into separate paragraphs
- Nominalized nouns → convert to verb form

### Step 2 — Reverse Outline Test

**Step 2a — Extract topic sentences:**
Pull the first sentence of every paragraph across all sections.

**Step 2b — Read in sequence:**
The topic sentences should form a standalone coherent narrative. If reading them sequentially skips between ideas or repeats themes redundantly, flag each problematic paragraph.

**Step 2c — Claim coverage check:**
Against `artifacts/<run_id>/stage-14/experiment_summary.json` and `artifacts/<run_id>/stage-16/outline.md`, verify:
- Every major claim appears in the paper
- Every quantitative result is cited with its corresponding experiment

**Step 2d — Evidence mapping:**
For each figure/table/experiment result, confirm:
- It is explicitly referenced in the main text
- Its contribution to a specific claim is stated or clearly implied

**Step 2e — Fix gaps:**
For each identified gap, rewrite the paragraph to either:
- Add the missing claim reference
- Make the evidence mapping explicit
- Remove the paragraph if it neither advances the narrative nor supports a claim

**Citation stability rule:**
- Do NOT introduce any new citation keys during polish
- Citation keys in `paper_polished.md` and regenerated `main.tex` must be a subset of the Stage-22 citation set
- If a new citation is absolutely necessary, the stage MUST fail and require re-verification upstream rather than adding it silently

### Step 3 — Section Word Count Validation

Count words per section (excluding LaTeX commands, `\ref{}`, citations):

| Section | Target Range | Fail If |
|---------|-------------|---------|
| Abstract | 180–220 | < 150 or > 250 |
| Introduction | 500–900 | < 400 or > 1000 |
| Related Work | 600–1200 | < 500 |
| Method | 800–1500 | < 600 or > 2000 |
| Experiments | 1200–2000 | < 800 or > 2500 |
| Conclusion | 200–400 | < 150 or > 500 |

Flag out-of-range sections in `artifacts/<run_id>/stage-25/polish_report.json`. Fix if possible within stage, else fail.

### Step 4 — Final Structural Checks

Run all checks from `arc-07-02-paper-draft` Validation section plus:
- [ ] No stale `.tex` section files
- [ ] `references.bib` contains only cited entries
- [ ] No `[VERIFY]` markers remaining
- [ ] All figure/table `\label`/`\ref` pairs match
- [ ] Related Work ≥ 1 full page
- [ ] Title is specific and informative
- [ ] Contribution list in intro matches paper content
- [ ] Prepare the paper package so downstream trust gates can verify claims, figures, and submission format without further structural rewrites

### Step 5 — Integrate Reviewer Feedback

If `artifacts/<run_id>/stage-24/AUTO_REVIEW.md` exists, read the final round's action items and verify each was addressed. Flag any CRITICAL/MAJOR issues that remain unresolved.

### Step 6 — Preserve Verified Citations and Reproducibility Metadata
- Start from `artifacts/<run_id>/stage-23/references_verified.bib`; do not regenerate bibliography from scratch
- Prune unused verified entries only
- Fail if `artifacts/<run_id>/stage-25/paper_polished.md` references a citation key absent from `artifacts/<run_id>/stage-23/references_verified.bib`
- Carry forward `artifacts/<run_id>/stage-22/reproducibility_report.json` unchanged unless packaging paths/checksums need refresh
- Do not alter execution commands, seed summaries, environment summary, or provenance fields during polish

### Step 7 — Regenerate Final Package
Produce a refreshed polished package that mirrors the stage-22 schema:
- Regenerate `artifacts/<run_id>/stage-25/main.tex` from `artifacts/<run_id>/stage-25/paper_polished.md`
- Derive final `artifacts/<run_id>/stage-25/references.bib` from `artifacts/<run_id>/stage-23/references_verified.bib`
- Compile `artifacts/<run_id>/stage-25/main.pdf`
- Write `artifacts/<run_id>/stage-25/compile_report.json`
- Write `artifacts/<run_id>/stage-25/manifest.json` with artifact paths and checksums, including `reproducibility_report.json`, `code/`, and `charts/`
- Assemble `artifacts/<run_id>/stage-25/deliverables/` with `main.tex`, `main.pdf`, `references.bib`, `paper_polished.md`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, `code/`, and `charts/` for downstream trust gates

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| No forbidden AI words | `polish_report.de_ai_patterns_fixed == 0` after fix pass |
| Reverse outline coherent | Topic sentences form a logical narrative |
| All claims covered | Every claim from summary appears in paper |
| Abstract 180–220 words | Strict range enforced |
| Section word counts | All within target ranges |
| Citation verification | No `[VERIFY]` markers remaining |
| Citation set stable | Polished citation keys are a subset of Stage-22 citation keys; no new keys introduced |
| Verified bibliography preserved | Final `references.bib` is derived from `references_verified.bib` only |
| Reproducibility preserved | `reproducibility_report.json` is present and semantic fields are unchanged |
| No stale files | All `.tex` sections are referenced |
| Polished package ready | `main.tex`, `main.pdf`, `references.bib`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, and `deliverables/` are produced in the same schema expected from the stage-22 export handoff |

**Failure**: Any failed check → `E25` → pipeline blocked from trust-gate handoff.

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E25` | Stage-25 polish failed De-AI cleanup, structural validation, or package regeneration |
| `E25_ABSTRACT` | Polished abstract remains outside the required word-count range |
| `E25_CLAIM_GAP` | A required claim from `experiment_summary.json` is still missing from the paper |
| `E25_REVIEWER_UNRESOLVED` | A CRITICAL or MAJOR reviewer issue from stage 24 remains unresolved |
| `E25_CITATION_DRIFT` | Polish introduced citation drift relative to the verified stage-23 bibliography |
| `E25_REPRODUCIBILITY_DRIFT` | Polish changed reproducibility metadata semantically instead of preserving it |
| `E25_PACKAGE_INCOMPLETE` | The polished package is missing required artifacts or failed compile validation |

---

## Retry Policy
Max retries: **1** — retry if De-AI patterns remain or abstract word count is wrong after first pass.

---

## State Transition
`pending` → `running` → `done` | `failed`
