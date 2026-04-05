---
name: arc-08-03-export-publish
description: Stage 22 — Export the paper as a complete, submission-ready package with LaTeX, figures, and code bundle.
metadata:
  category: pipeline-stage
  trigger-keywords: "export publish,final package,stage 22"
  applicable-stages: "22"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Package the revised paper into the canonical stage-22 submission bundle: compilable LaTeX, bibliography, auto-generated figures, experiment code, and a complete deliverables directory that later stages can review, polish, and revalidate.

---

## Quality Contract

`main.tex` MUST:
- Compile successfully with LaTeX + bibliography passes (exit code 0)
- Use a recognized conference template (NeurIPS / ICML / ICLR)
- Include all `\ref{}`/`\cref{}` that resolve to existing labels

`main.pdf` MUST:
- Be produced from `main.tex`
- Contain no undefined references/citations in the final build log

`compile_report.json` MUST:
- Record compile command sequence, final exit code, and warning summary
- Include `undefined_references` and `undefined_citations` counts (both must be 0)

`manifest.json` MUST list all artifacts with accurate file paths and checksum fields, including integrity coverage for `reproducibility_report.json`, `code/`, and `charts/`.

`reproducibility_report.json` MUST:
- Record execution commands, dependency/environment summary, and seed summary
- Preserve provenance needed for downstream reproducibility validation
- Match the stage-22 package semantics carried into stage 25

`deliverables/` directory MUST contain:
- `main.tex`
- `main.pdf`
- `references.bib`
- `paper_final.md` (source markdown)
- `code/` (experiment code bundle)
- `charts/` (figure files)
- `compile_report.json`
- `manifest.json`
- `reproducibility_report.json`

This bundle is the canonical stage-22 handoff. Stage 23 verifies citations against it, stage 24 reviews it, and stage 25 regenerates an updated polished package for trust-gate validation.

---

## Inputs
`artifacts/<run_id>/stage-00/environment.json`
`artifacts/<run_id>/stage-19/paper_revised.md`
`artifacts/<run_id>/stage-13/experiment_final/` (final experiment code snapshot)
`artifacts/<run_id>/stage-14/results_table.tex`
`artifacts/<run_id>/stage-14/experiment_summary.json`

---

## Outputs

### `artifacts/<run_id>/stage-22/main.tex`
Compiled LaTeX source targeting the appropriate conference template.

### `artifacts/<run_id>/stage-22/paper_final.md`
Final source copy.

### `artifacts/<run_id>/stage-22/references.bib`
Complete BibTeX bibliography.

### `artifacts/<run_id>/stage-22/code/`
Bundled final experiment code snapshot with README.

### `artifacts/<run_id>/stage-22/charts/`
Generated figures with error bars.

### `artifacts/<run_id>/stage-22/main.pdf`
Compiled PDF ready for packaging and downstream gate checks.

### `artifacts/<run_id>/stage-22/compile_report.json`
```json
{
  "commands": [
    "pdflatex main.tex",
    "bibtex main",
    "pdflatex main.tex",
    "pdflatex main.tex"
  ],
  "final_exit_code": 0,
  "undefined_references": 0,
  "undefined_citations": 0,
  "warnings": []
}
```

### `artifacts/<run_id>/stage-22/reproducibility_report.json`
```json
{
  "execution_commands": [
    "python -m experiment.run --config configs/final.yaml"
  ],
  "seed_summary": {
    "global_seed": 42,
    "per_condition": {
      "baseline": [42, 43, 44],
      "method": [45, 46, 47]
    }
  },
  "environment_summary": {
    "python": "3.10.12",
    "frameworks": {
      "torch": "2.1.0"
    },
    "device": "cuda:0"
  },
  "provenance": {
    "experiment_source": "stage-13/experiment_final",
    "results_source": "stage-14/experiment_summary.json"
  }
}
```

### `artifacts/<run_id>/stage-22/manifest.json`
```json
{
  "paper": "main.tex",
  "pdf": "main.pdf",
  "bibliography": "references.bib",
  "experiment_code": "code/",
  "figures": ["charts/fig1.pdf", "charts/fig2.pdf"],
  "compile_report": "compile_report.json",
  "reproducibility_report": "reproducibility_report.json",
  "checksums": {
    "main.tex": "sha256:...",
    "main.pdf": "sha256:...",
    "references.bib": "sha256:...",
    "reproducibility_report.json": "sha256:...",
    "code/": "sha256:...",
    "charts/": "sha256:..."
  },
  "total_artifacts": 8
}
```

### `artifacts/<run_id>/stage-22/deliverables/`
Complete copy of all output artifacts in a single directory. This is the canonical package consumed by stages 23–25 before stage 25 regenerates the final polished package, and it MUST preserve `reproducibility_report.json` for downstream trust gates.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Environment precheck passed | `artifacts/<run_id>/stage-00/environment.json` has `pipeline_capable: true` and `latex_capable: true` |
| `main.tex` compiles | `artifacts/<run_id>/stage-22/main.tex` compiles with final exit code 0 |
| `main.pdf` exists | `artifacts/<run_id>/stage-22/main.pdf` exists and is non-empty |
| No undefined refs/cites | `artifacts/<run_id>/stage-22/compile_report.json` has both counts = 0 |
| `compile_report.json` valid | `artifacts/<run_id>/stage-22/compile_report.json` is valid JSON with command list, exit code, and warning summary |
| `reproducibility_report.json` valid | `artifacts/<run_id>/stage-22/reproducibility_report.json` includes execution commands, environment summary, and seed summary |
| `manifest.json` valid | `artifacts/<run_id>/stage-22/manifest.json` has artifact paths + checksums + `total_artifacts >= 8` |
| `deliverables/` complete | `artifacts/<run_id>/stage-22/deliverables/` contains all listed files |

**Failure**: If environment precheck, compile, report generation, or packaging fails after 2 fix attempts → `E22`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E22` | Stage-22 export package failed local LaTeX compilation, reporting, or packaging |
| `E22_NO_LATEX_ENV` | `environment.json` does not confirm `pipeline_capable: true` and `latex_capable: true` |
| `E22_REPRODUCIBILITY_MISSING` | `reproducibility_report.json` is missing or semantically incomplete |

---

## Retry Policy
Max retries: **2** — each retry fixes specific compilation errors.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Environment and Input Artifacts
Read `artifacts/<run_id>/stage-00/environment.json`, `artifacts/<run_id>/stage-19/paper_revised.md`, `artifacts/<run_id>/stage-13/experiment_final/` (final code bundle), `artifacts/<run_id>/stage-14/results_table.tex`, and `artifacts/<run_id>/stage-14/experiment_summary.json`.

Fail fast if `artifacts/<run_id>/stage-00/environment.json` does not show `pipeline_capable: true` and `latex_capable: true`. This stage uses only the local LaTeX toolchain declared by Stage 0.

### Step 2 — Convert to LaTeX
Convert `artifacts/<run_id>/stage-19/paper_revised.md` to `artifacts/<run_id>/stage-22/main.tex`:
- Apply the appropriate conference template (NeurIPS / ICML / ICLR)
- Insert `results_table.tex` into the appropriate section
- Convert all markdown formatting to LaTeX equivalents
- Add `\label{}` to all figures, tables, and equations for `\ref{}` / `\cref{}`

### Step 3 — Generate Figures
From `artifacts/<run_id>/stage-14/experiment_summary.json`, generate charts into `artifacts/<run_id>/stage-22/charts/`:
- Main results bar chart (all conditions × primary metric)
- Ablation effects line/bar chart
- Error bars from `metric_std`
- Save as PDF/PNG in `charts/`

### Step 4 — Build References Bibliography
Write `artifacts/<run_id>/stage-22/references.bib` from stage-14 or prior verified inputs. Ensure every `\cite{}` / `\citep{}` / `\citet{}` in `artifacts/<run_id>/stage-22/main.tex` has a corresponding BibTeX entry.

### Step 5 — Compile LaTeX and Produce PDF
Run bibliography-aware compile passes for `artifacts/<run_id>/stage-22/main.tex`:
```bash
pdflatex main.tex
bibtex main
pdflatex main.tex
pdflatex main.tex
```
Ensure `artifacts/<run_id>/stage-22/main.pdf` is produced.

### Step 6 — Write `artifacts/<run_id>/stage-22/compile_report.json`
Summarize compile commands, final exit code, undefined reference/citation counts, and warning summary.

### Step 7 — Write `artifacts/<run_id>/stage-22/reproducibility_report.json`
Write the reproducibility artifact owned by stage 22:
- execution command summary
- dependency/environment summary
- seed summary
- provenance linking experiment code and result artifacts

### Step 8 — Package Deliverables
Create `artifacts/<run_id>/stage-22/deliverables/` with:
- `main.tex` (final LaTeX source)
- `main.pdf` (compiled final PDF)
- `references.bib`
- `paper_final.md` (source markdown)
- `code/` (final experiment code snapshot from `experiment_final/`)
- `charts/` (all figure files)
- `compile_report.json`
- `manifest.json` (list of all artifacts + checksums)
- `reproducibility_report.json`

Treat this package as the canonical stage-22 export that downstream stages review and polish rather than rewriting stage-22 artifacts in place.

### Step 9 — Validate
Check `artifacts/<run_id>/stage-00/environment.json`, compile exit code, no undefined refs/cites, valid `artifacts/<run_id>/stage-22/compile_report.json`, valid `artifacts/<run_id>/stage-22/reproducibility_report.json`, valid `artifacts/<run_id>/stage-22/manifest.json`, and complete `artifacts/<run_id>/stage-22/deliverables/`. Fail → `E22`.
