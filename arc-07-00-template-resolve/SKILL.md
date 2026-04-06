---
name: arc-07-00-template-resolve
description: Pre-export template resolver — choose and normalize the target submission template family, version, and package rules before final LaTeX export.
metadata:
  category: pipeline-support
  trigger-keywords: "template resolve,venue template,journal template,latex template"
  applicable-stages: "pre-16, pre-22, pre-25, pre-28"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Resolve the paper's target venue into a machine-checkable template contract before export. This supporting skill selects the venue family, template variant, external asset readiness, compile engine, page/structure rules, and submission mode so downstream writing, export, polish, and submission-gate stages use the same assumptions.

---

## Quality Contract

`template_manifest.json` MUST:
- Declare exactly one `publication_target`
- Normalize venue aliases to a stable `template_id`
- Record `publication_type` (`conference` or `journal`)
- Record `submission_mode` (`anonymous`, `camera_ready`, or `preprint`)
- Record whether the resolved template assets are `bundled`, `contract_only`, or `user_supplied`
- Record compile engine + bibliography engine compatible with the selected template
- Record page/structure rules and figure rules that downstream stages can validate deterministically

Hard constraints:
- `template_resolved: true`
- `template_id` is from the supported families below
- `template_assets.asset_status` is explicit
- `validation_rules.page_limit_main_body` is explicit for conference targets, or `null` with `page_limit_policy` for journals
- If present, figure rules (`required_figure_count_min`, `recommended_figure_count_range`, `figure_style_rules`) are carried forward unchanged into the manifest
- If `template_assets.asset_status != "ready"`, the stage MUST explicitly note that stage 22 export will block until the official template files are available in the execution workspace

Supported template families:
- `neurips` (`neurips_2026`, `neurips_2025`, `neurips_2024`)
- `iclr` (`iclr_2026`, `iclr_2025`)
- `icml` (`icml_2026`, `icml_2025`)
- `aaai` (`aaai_2026`)
- `acl_family` (`acl_2025`, `emnlp_2025`, `naacl_2025`)
- `cv_family` (`cvpr_2026`, `iccv_2025`, `eccv_2026`)
- `kdd` (`kdd_2025`)
- `ieee_conf` (`ieeetran_conference`)
- `ieee_journal` (`ieeetran_journal`)
- `elsevier_journal` (`elsevier_cas_sc`, `pattern_recognition`) 
- `springer_journal` (`springer_lncs_journal_style`)

---

## Inputs
`artifacts/<run_id>/stage-00/environment.json`
`artifacts/<run_id>/stage-15/decision.md` (optional but preferred)
Pipeline input config:
- `target_venue` (optional, default `neurips`)
- `template_version` (optional)
- `publication_type` (optional: `conference` | `journal`)
- `submission_mode` (optional, default `anonymous` for conferences, `preprint` for journals)

---

## Outputs

### `artifacts/<run_id>/template/template_manifest.json`
```json
{
  "template_resolved": true,
  "publication_target": {
    "publication_type": "conference",
    "venue": "neurips",
    "year": 2026,
    "track": "main",
    "template_family": "neurips",
    "template_id": "neurips_2026",
    "submission_mode": "anonymous"
  },
  "template_assets": {
    "asset_status": "ready",
    "asset_source": "execution_workspace",
    "documentclass": "article",
    "style_files": ["neurips_2026.sty"],
    "bibliography_style": "plainnat",
    "citation_commands": ["citep", "citet", "cite"],
    "compile_engine": "pdflatex",
    "bibliography_engine": "bibtex",
    "bibliography_style_file": null
  },
  "validation_rules": {
    "page_limit_main_body": 9,
    "page_limit_policy": "main_body_only",
    "columns": 2,
    "requires_anonymous_submission": true,
    "required_sections": [
      "abstract",
      "introduction",
      "related_work",
      "method",
      "experiments",
      "limitations",
      "conclusion"
    ],
    "forbidden_sections": ["author_bio"],
    "author_block_policy": "anonymous_until_camera_ready",
    "required_figure_count_min": 4,
    "recommended_figure_count_range": [4, 8],
    "figure_style_rules": {
      "caption_required": true,
      "legend_required_for_multiseries": true,
      "units_required_when_numeric_axes": true,
      "color_consistent_across_figures": true,
      "font_readable_min_pt": 8,
      "vector_preferred": true,
      "raster_dpi_min": 300,
      "subfigure_label_style": "uppercase_letter"
    }
  },
  "notes": []
}
```

### `artifacts/<run_id>/template/template_notes.md`
Short human-readable explanation of the selected venue rules, asset readiness, assumptions, and any unresolved template limitations.

---

## Procedure

### Step 1 — Read Environment and Requested Target
Read `environment.json` and the pipeline input configuration. Fail if LaTeX is unavailable, because template resolution without a local compile path is not actionable for ARC.

### Step 2 — Normalize Venue Alias via Built-In Skill Contract
Resolve user-facing aliases using the supported families and template IDs declared in this skill. Preferred order:
1. exact `template_version`
2. exact family alias match for `target_venue`
3. publication-type default family and template

Examples:
- `neurips` → `neurips_2026`
- `iclr` → `iclr_2026`
- `icml` → `icml_2026`
- `ieee_journal` → `ieeetran_journal`
- `ieee_conf` → `ieeetran_conference`
- `pattern_recognition` or `pr` → `pattern_recognition`

If `publication_type=journal` and no family is given, prefer the default journal family declared by this skill contract.

### Step 3 — Load Template Record
Read the chosen template record from the registry and copy forward:
- `asset_dir`
- `asset_status`
- `documentclass`
- `style_files`
- `bibliography_style`
- `bibliography_style_file` (if present)
- `citation_commands`
- `compile_engine`
- `bibliography_engine`
- `validation_rules`

### Step 4 — Build Template Rules
Write deterministic rules for:
- page limit or journal page policy
- single/double-column expectation
- anonymous vs camera-ready metadata handling
- preferred citation commands
- compile engine and bibliography engine
- whether the asset directory is immediately compile-ready or still contract-only
- required/forbidden sections
- author-block policy
- minimum/recommended figure counts
- figure style and file-quality requirements

### Step 5 — Write Outputs
Write `template_manifest.json` and `template_notes.md` under `artifacts/<run_id>/template/`.
If the template is `contract_only`, the notes MUST say exactly which official files the user still needs to place into `asset_dir`.

### Step 6 — Validate
Ensure `template_resolved=true`, `registry_source` is recorded, a supported `template_id` exists, `asset_status` is explicit, and downstream stages can consume the manifest without guessing venue semantics.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Template manifest exists | `artifacts/<run_id>/template/template_manifest.json` exists and valid JSON |
| Template resolved | `template_resolved == true` |
| Supported template | `template_id` belongs to supported families |
| Compile path explicit | `compile_engine` and `bibliography_engine` are present |
| Asset path explicit | `asset_dir` is present in `template_assets` |
| Asset status explicit | `asset_status` is present in `template_assets` |
| Validation rules explicit | Page/structure/submission-mode/figure rules are present |

**Failure**: Missing or ambiguous template selection → `E_TEMPLATE_RESOLVE`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E_TEMPLATE_RESOLVE` | Could not resolve a unique supported venue/template target |
| `E_TEMPLATE_UNSUPPORTED` | Requested venue is outside supported families |
| `E_TEMPLATE_LATEX_UNAVAILABLE` | Local LaTeX unavailable, so template resolution cannot be honored |
| `E_TEMPLATE_ASSET_MISSING` | Resolved template is contract-only and required official files are not available in `asset_dir` |

---

## Retry Policy
Max retries: **1** — retry only after clarifying or correcting the requested venue/template target.

---

## State Transition
`pending` → `running` → `done` | `failed`
