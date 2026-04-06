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
- Use the resolved template contract from `artifacts/<run_id>/template/template_manifest.json`
- Support both conference and journal publication targets from the supported venue families
- Include all `\ref{}`/`\cref{}` that resolve to existing labels

`main.pdf` MUST:
- Be produced from `main.tex`
- Contain no undefined references/citations in the final build log
- Be backed by a bibliography and figure set that already aim to satisfy downstream citation-floor and figure-authenticity gates

`compile_report.json` MUST:
- Record compile command sequence, final exit code, and warning summary
- Include `undefined_references` and `undefined_citations` counts (both must be 0)

`manifest.json` MUST list all artifacts with accurate file paths and checksum fields, including integrity coverage for `reproducibility_report.json`, `code/`, and `charts/`.

Each generated figure in `charts/` MUST have a provenance sidecar file (`<figure>.provenance.json`) that records the source artifact, metric key(s), generation command or method, and an explicit `evidence_bearing: true` / `decorative: false` authenticity declaration.

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
`artifacts/<run_id>/template/template_manifest.json`

---

## Outputs

### `artifacts/<run_id>/stage-22/main.tex`
Compiled LaTeX source targeting the resolved conference or journal template.

### `artifacts/<run_id>/stage-22/paper_final.md`
Final source copy.

### `artifacts/<run_id>/stage-22/references.bib`
Complete BibTeX bibliography.

### `artifacts/<run_id>/stage-22/code/`
Bundled final experiment code snapshot with README.

### `artifacts/<run_id>/stage-22/charts/`
Generated figures with error bars plus one provenance sidecar per figure (`<figure>.provenance.json`).

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
  "template_manifest": "template_manifest.json",
  "template_asset_source": "execution_workspace",
  "experiment_code": "code/",
  "figures": [
    {"file": "charts/fig1.pdf", "provenance": "charts/fig1.provenance.json"},
    {"file": "charts/fig2.pdf", "provenance": "charts/fig2.provenance.json"}
  ],
  "compile_report": "compile_report.json",
  "reproducibility_report": "reproducibility_report.json",
  "checksums": {
    "main.tex": "sha256:...",
    "main.pdf": "sha256:...",
    "references.bib": "sha256:...",
    "template_manifest.json": "sha256:...",
    "reproducibility_report.json": "sha256:...",
    "code/": "sha256:...",
    "charts/": "sha256:..."
  },
  "total_artifacts": 9
}
```

### `artifacts/<run_id>/stage-22/deliverables/`
Complete copy of all output artifacts in a single directory. This is the canonical package consumed by stages 23–25 before stage 25 regenerates the final polished package, and it MUST preserve `reproducibility_report.json` for the remaining late-stage gates.

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
| `manifest.json` valid | `artifacts/<run_id>/stage-22/manifest.json` has artifact paths + checksums + `template_manifest.json` + `total_artifacts >= 9` |
| `deliverables/` complete | `artifacts/<run_id>/stage-22/deliverables/` contains all listed files including `template_manifest.json` |

**Failure**: If environment precheck, compile, report generation, or packaging fails after 2 fix attempts → `E22`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E22` | Stage-22 export package failed local LaTeX compilation, reporting, or packaging |
| `E22_NO_LATEX_ENV` | `environment.json` does not confirm `pipeline_capable: true` and `latex_capable: true` |
| `E22_REPRODUCIBILITY_MISSING` | `reproducibility_report.json` is missing or semantically incomplete |
| `E22_FIGURE_PROVENANCE_MISSING` | A generated figure is missing a valid provenance sidecar or authenticity declaration |

---

## Retry Policy
Max retries: **2** — each retry fixes specific compilation errors.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Environment, Template Contract, and Input Artifacts
Read `artifacts/<run_id>/stage-00/environment.json`, `artifacts/<run_id>/template/template_manifest.json`, `artifacts/<run_id>/stage-19/paper_revised.md`, `artifacts/<run_id>/stage-13/experiment_final/` (final code bundle), `artifacts/<run_id>/stage-14/results_table.tex`, and `artifacts/<run_id>/stage-14/experiment_summary.json`.

Fail fast if `artifacts/<run_id>/stage-00/environment.json` does not show `pipeline_capable: true` and `latex_capable: true`. This stage uses only the local LaTeX toolchain declared by Stage 0.

### Step 2 — Convert to LaTeX
Convert `artifacts/<run_id>/stage-19/paper_revised.md` to `artifacts/<run_id>/stage-22/main.tex`:
- Apply the resolved template from `template_manifest.json` rather than guessing the venue
- Confirm the manifest's `template_id` belongs to the supported template families declared by the skills chain
- Require the manifest to point to real template assets already present in the execution workspace, and preserve that contract in the export package
- Support conference and journal front matter, author-handling, citation commands, and section requirements from the manifest
- Insert `results_table.tex` into the appropriate section
- Convert all markdown formatting to LaTeX equivalents
- Add `\label{}` to all figures, tables, and equations for `\ref{}` / `\cref{}`

### Step 3 — Generate Figures
From `artifacts/<run_id>/stage-14/experiment_summary.json`, generate charts into `artifacts/<run_id>/stage-22/charts/`:
- Main results bar chart (all conditions × primary metric)
- Ablation effects line/bar chart
- Error bars from `metric_std`
- Satisfy any hard minimum figure count and figure-style requirements declared in `template_manifest.json`
- Prefer vector outputs; if raster is required, keep export quality high enough for downstream DPI checks
- Save as PDF/PNG in `charts/`
- For each generated figure, also write `<figure>.provenance.json` with:
  - `source_artifact`
  - `metric_keys`
  - `generation_method` or command summary
  - `evidence_bearing: true`
  - `decorative: false`
  - timestamp / provenance metadata sufficient for later authenticity checks
- Treat a figure without a valid provenance sidecar as an export failure rather than allowing late-stage gates to guess provenance

### Step 4 — Build References Bibliography
Write `artifacts/<run_id>/stage-22/references.bib` from stage-14 or prior verified inputs. Ensure every `\cite{}` / `\citep{}` / `\citet{}` in `artifacts/<run_id>/stage-22/main.tex` has a corresponding BibTeX entry. Prefer a bibliography that already satisfies downstream targets of at least 30 verified references and at least 20% references from the last 5 years unless topic constraints are explicit.

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
- `template_manifest.json`
- `code/` (final experiment code snapshot from `experiment_final/`)
- `charts/` (all figure files)
- `compile_report.json`
- `manifest.json` (list of all artifacts + checksums)
- `reproducibility_report.json`

Treat this package as the canonical stage-22 export that downstream stages review and polish rather than rewriting stage-22 artifacts in place. The package should already expose enough bibliography coverage and figure provenance for later citation-floor and figure-authenticity gates to pass without inventing new evidence late in the pipeline.

### Step 9 — Validate
Check `artifacts/<run_id>/stage-00/environment.json`, compile exit code, no undefined refs/cites, valid `artifacts/<run_id>/stage-22/compile_report.json`, valid `artifacts/<run_id>/stage-22/reproducibility_report.json`, valid `artifacts/<run_id>/stage-22/manifest.json`, and complete `artifacts/<run_id>/stage-22/deliverables/`. Fail → `E22`.
