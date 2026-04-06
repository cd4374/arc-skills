# arc-skills v4.2

Pure-skills research pipeline for AutoResearchClaw. **Executable as a standalone skill chain by any AI agent** — the workflow is agent-portable, while successful execution still depends on real local capabilities such as Python execution, network access, and local LaTeX.

Core non-negotiable objective: when the full chain succeeds, it should yield a **high-quality, verifiable, reproducible, authentic academic paper** with at minimum real LaTeX sources, a locally compiled PDF, verified references, real experiment-backed claims, and submission-format compliance.

## Quick Start

### Prerequisites Checklist

Before running the pipeline, verify you have:

| Requirement | Check Command | Required For |
|-------------|---------------|--------------|
| Python 3.9+ | `python --version` | Stages 10-13 (experiments) |
| PyTorch | `python -c "import torch; print(torch.__version__)"` | Neural network experiments |
| GPU/CUDA | `nvidia-smi` or `torch.cuda.is_available()` | GPU experiments |
| Local LaTeX | `pdflatex --version` | Stages 22, 28 (PDF compilation) |
| BibTeX | `bibtex --version` | Bibliography compilation |
| Network | `curl -I https://doi.org/10.1000/182` | Stages 3-4 (citation verification) |

### Install

```bash
# Clone or copy the skills
cp -R arc-skills/* ~/.claude/skills/

# Verify installation
ls ~/.claude/skills/arc-00-01-research-pipeline/
```

Canonical entrypoint — start from an idea:
```text
/arc-00-01-research-pipeline
{
  "topic": "your research idea",
  "target_venue": "neurips"
}
```

Canonical entrypoint — improve an existing paper/code project:
```text
/arc-00-01-research-pipeline
{
  "mode": "existing_project_improvement",
  "project_path": "/path/to/project"
}
```

Recovery helper:
```text
/arc-00-03-resume
```

`arc-00-01-research-pipeline` is the only user-facing pipeline entry. It supports exactly two modes: default `idea_start` and `existing_project_improvement`.

## Current Limitations

### Template Assets

This repository provides **skill definitions only**. Some conference templates require official style files from organizers:

| Template | Status | Additional Files Required |
|----------|--------|---------------------------|
| NeurIPS | Ready | Uses included `neurips_2026.sty` |
| ICML | Ready | Uses included `icml2026.sty` |
| ICLR | Ready | Self-contained |
| AAAI | Ready | Self-contained |
| ACL/EMNLP/NAACL | Partial | Requires `acl.sty` from [official release](https://github.com/acl-org/acl-style-files) |
| CVPR/ICCV/ECCV | Partial | Requires `cvpr.sty` from [official release](https://github.com/cvpr-org/author-kit) |
| IEEE | Ready | Uses standard `IEEEtran` class |
| Elsevier | Ready | Uses standard `elsarticle` class |
| Springer | Ready | Uses standard `svjour3` class |

**If the pipeline resolves to a template that requires external style files not present in your workspace, Stage 22 (export) will block with `E_TEMPLATE_ASSET_MISSING`.**

### Execution Environment

The pipeline **BLOCKS** (does not degrade) if any of these are missing:

- **No execution backend** → Cannot run experiments (Stages 10-13)
- **No local LaTeX** → Cannot compile PDF (Stages 22, 28)
- **No network** → Cannot verify citations (Stages 3-4)

This is intentional — the pipeline refuses to produce paper-shaped artifacts without real evidence backing.

### Known Issues

1. **Template resolution (Stage 15.7)** runs after hypothesis generation. Ideally, venue-aware decisions (e.g., conference vs journal citation preferences) should influence earlier stages.

2. **External review (Stage 24)** defaults to degraded local review if Codex MCP is unavailable. For full adversarial guarantees, configure a dedicated MCP reviewer.

3. **Experience accumulation (Stage 20.5)** stores per-run guidance only. Cross-run knowledge sharing requires manual aggregation.

## Quick Start with Templates

### Option 1: Use Built-in Templates (Recommended for Testing)

For venues with self-contained templates (NeurIPS, ICML, ICLR, AAAI, IEEE, Elsevier, Springer):

```bash
# Templates are auto-copied to workspace by Stage 15.7
# Just specify target venue:
/arc-00-01-research-pipeline
{
  "topic": "your research idea",
  "target_venue": "neurips"
}
```

### Option 2: Use Official Conference Templates

For venues requiring official style files (ACL, CVPR):

```bash
# 1. Download official template
mkdir -p /path/to/workspace/templates/acl
cd /path/to/workspace/templates/acl
curl -L -o acl.sty https://raw.githubusercontent.com/acl-org/acl-style-files/master/acl.sty

# 2. Run pipeline
/arc-00-01-research-pipeline
{
  "topic": "your research idea",
  "target_venue": "acl",
  "template_path": "/path/to/workspace/templates"
}
```

### Option 3: Custom Template

To use your own LaTeX template:

```bash
# Place your template files in workspace
/arc-00-01-research-pipeline
{
  "topic": "your research idea",
  "target_venue": "custom",
  "template_manifest": {
    "template_id": "custom",
    "asset_status": "user_supplied",
    "asset_dir": "/path/to/your/template"
  }
}
```

## Template Quick Reference

```
Conference Targets (8 pages main + references):
  neurips, icml, iclr, aaai, acl, cvpr, kdd

Journal Targets (no hard page limit):
  ieee_journal, elsevier_journal, springer_journal

Anonymous submission (default for conferences):
  All conference templates default to anonymous mode

Camera-ready:
  Pass "submission_mode": "camera_ready" for final version
```

## Design Philosophy

**Skill = Task Definition + Output Contract + Acceptance Rules.**

## Hard Paper Standards

The chain target is not merely “a paper-shaped artifact”. Final acceptance should satisfy hard academic-output standards:

- Real local LaTeX compilation is mandatory; no fake or placeholder PDF path
- Verified literature only; fabricated/unverifiable references are blocking failures
- Quantitative claims must remain traceable to real experiment artifacts
- The paper should normally carry **at least 30 references**
- The paper should normally maintain **at least 20% of references from the last 5 years**
- Figures must satisfy venue-aware hard standards for count, readability, style, and file quality
- Figures must be traceable to real experiment outputs; fabricated or decorative figures are unacceptable
- Missing critical execution / LaTeX / network capability is blocking, not degradable

These standards should shape every iteration loop, review loop, and re-check flow.

Each SKILL.md describes:
- **What** must be produced (output schema)
- **What quality** the output must meet (quality contract)
- **What happens** on failure (error codes, retry policy)

Each SKILL.md does **NOT** describe:
- How to generate the output (execution is the Agent's responsibility)
- What tools or models to use (Agent decides based on its own capabilities)

---

## Skill Structure

All 28 stage skills follow this mandatory structure:

| Section | Meaning | Required |
|---------|---------|---------|
| `## Purpose` | What this stage accomplishes | Yes |
| `## Quality Contract` | Hard constraints on the output — failure conditions | Yes |
| `## Inputs` | Required input files and their schema | Yes |
| `## Outputs` | Required output files and their schema | Yes |
| `## Procedure` | Step-by-step execution protocol (numbered) | **Yes (all 28 stage skills)** |
| `## Validation` | Machine-checkable acceptance criteria | Yes |
| `## Error Codes` | Failure types this stage can emit | Yes |
| `## Retry Policy` | Max retries for each error code | Yes |
| `## Gate Behavior` | (gate stages only) approve/reject routing | Gate only |
| `## Rollback Contract` | (gate/decision stages) rollback targets | Gate + decision |
| `## State Transition` | How this stage affects pipeline status | Yes |

**Orchestrator skills** (`arc-00-*`) use a superset structure: Stage Map + State Machine + Error Registry + Gate/Rollback/Decision rules — they are the protocol definition, not the execution.

---

## Orchestrators

| Skill | Role | Has Procedure |
|-------|------|--------------|
| `arc-00-01-research-pipeline` | Canonical user-facing entrypoint. Runs the pipeline in exactly two modes: default `idea_start`, or `existing_project_improvement` for paper/code re-check and iterative improvement. | N/A (protocol def) |
| `arc-00-02-status` | Read-only state inspector | Yes — 7 steps |
| `arc-00-03-resume` | Resume from checkpoint | Yes — 7 steps |
| `arc-00-04-environment-probe` | Internal/support orchestrator for environment detection (execution + local LaTeX + network). Automatically required before pipeline progress. | Yes — 11 steps |
| `arc-00-05-project-recheck` | Internal/support orchestrator used by `existing_project_improvement` mode. Diagnoses existing projects, restructures external ones into ARC-compatible artifacts, delegates fixes to main pipeline skills, and iterates until quality target and required late-stage gates pass or max iterations is exhausted. | Yes — 14 steps |
| `arc-00-06-meta-optimizer` | Internal/support orchestrator for experience accumulation and reuse. Distills reusable guidance from real artifacts into run-owned outputs without relying on bundled non-skill storage. | Yes — 8 steps |
| `arc-09-01-paper-review-loop` | External adversarial review via Codex/GPT-5.4 MCP in nightmare mode, with degraded-local review logged honestly only as fallback. | Yes — 9 steps |
| `arc-09-02-paper-polish` | De-AI Polish + Reverse Outline + provenance-preserving final checks | Yes — 7 steps |

**All 28 stage skills** now have complete sections: Purpose · Quality Contract · Inputs · Outputs · **Procedure** · Validation · Error Codes · Retry Policy · State Transition. Gate/decision stages additionally have Gate Behavior and/or Rollback Contract.

---

## Stage Map

| Phase | Skills | Description |
|-------|--------|-------------|
| **0: Environment** | arc-00-04 | Execution + LaTeX + Network detection (BLOCKING on missing capabilities) |
| A: Scoping | arc-01-01 → arc-01-02 | Topic → Problem decomposition |
| B: Literature | arc-02-01 → arc-02-04 | Search → Collect (BLOCKING on unverifiable DOI/arXiv) → Screen → Extract |
| C: Synthesis | arc-03-01 → 7.5 → arc-03-02 | Synthesis → Novelty Gap Gate (BLOCKING) → Hypothesis generation |
| D: Experiment Design | arc-04-01 → 9.5 → arc-04-03 | Design (GATE) → Reproducibility Design Gate (BLOCKING) → Code gen → Resource plan |
| E: Execution | arc-05-01 → arc-05-02 | Run experiments → Iterative refine (with auto-repair) |
| F: Analysis | arc-06-01 → 15 → 15.5 → 15.7 | Result analysis → Research decision → Result Claim Gate (BLOCKING) → Template Resolve |
| G: Writing | arc-07-01 → 17 → 17.5 → 18 → 18.5 | Outline → Draft → Writing Compliance Gate (BLOCKING) → Review → Bibliography Quality Gate (BLOCKING) → Revision |
| H: Finalization | arc-08-01 → 20.5 → 21 → 21.5 → arc-08-03 → arc-08-04 | Quality (GATE) → Meta Optimizer → Archive → Reproducibility Bundle Gate (BLOCKING) → Export → Verify |
| I: Review & Polish | arc-09-01 → 24.5 → arc-09-02 | External nightmare review → Academic Integrity Gate (BLOCKING) → De-AI Polish |
| J: Late-Stage Gates | arc-10-01 → 27 → 27.5 → 28 → 28.5 | Claim-Evidence Trace (GATE, requires arc-10-04 numeric-truth pre-gate) → Numeric Truth (pre-GATE) → Figure Quality (GATE) → Submission Format (GATE) → Final Acceptance Gate (BLOCKING) |

---

## Supported Venue / Template Families

The pipeline is now venue-aware. Template semantics are resolved before writing/export and preserved through submission gates.

Template/template-family rules are defined directly by the skills chain as text contracts rather than bundled JSON registries or asset folders.

The chain remains venue-aware, but because this repo is pure-skills-only:
- no built-in registry file is assumed
- no bundled schema file is assumed
- no bundled template asset directory is assumed
- if the chosen venue requires official LaTeX assets that are not already present in the execution workspace, stage 22 must block honestly rather than degrade

| Family | Example template IDs | Publication type |
|-------|-----------------------|------------------|
| `neurips` | `neurips_2025`, `neurips_2024` | conference |
| `iclr` | `iclr_2026`, `iclr_2025` | conference |
| `icml` | `icml_2026`, `icml_2025` | conference |
| `aaai` | `aaai_2026` | conference |
| `acl_family` | `acl_2025`, `emnlp_2025`, `naacl_2025` | conference |
| `cv_family` | `cvpr_2026`, `iccv_2025`, `eccv_2026` | conference |
| `kdd` | `kdd_2025` | conference |
| `ieee_conf` | `ieeetran_conference` | conference |
| `ieee_journal` | `ieeetran_journal` | journal |
| `elsevier_journal` | `elsevier_cas_sc`, `pattern_recognition` | journal |
| `springer_journal` | `springer_lncs_journal_style` | journal |

Conference targets should carry explicit page-limit rules and usually default to `submission_mode: anonymous`. Journal targets may use `page_limit_policy` instead of a hard main-body cap and usually default to `submission_mode: preprint` unless camera-ready delivery is requested.

Template-contract validation rules also carry hard figure standards, including minimum figure count, recommended figure-count range, and style/file-quality requirements (`caption_required`, `legend_required_for_multiseries`, `units_required_when_numeric_axes`, `color_consistent_across_figures`, `font_readable_min_pt`, `vector_preferred`, `raster_dpi_min`, `subfigure_label_style`).

---

## Gate Stages (17 gates + 1 pre-gate)

| Stage | Skill | Rollback on Reject |
|-------|-------|-------------------|
| 0 | arc-00-04-environment-probe | **N/A (blocking)** — pipeline cannot start without execution/LaTeX/network |
| 4 | arc-02-02-literature-collect | **N/A (blocking on hallucination)** — unverifiable citations rejected per-entry |
| 5 | arc-02-03-literature-screen | arc-02-02-literature-collect |
| 7.5 | arc-03-03-novelty-gap-gate | arc-03-01-synthesis |
| 9 | arc-04-01-experiment-design | arc-03-02-hypothesis-gen |
| 9.5 | arc-04-04-reproducibility-design-gate | arc-04-01-experiment-design |
| 15.5 | arc-06-03-result-claim-gate | arc-06-02-research-decision |
| 17.5 | arc-07-05-writing-compliance-gate | arc-07-02-paper-draft |
| 18.5 | arc-08-05-bibliography-quality-gate | arc-07-02-paper-draft |
| 20 | arc-08-01-quality-gate | arc-07-01-paper-outline |
| 21.5 | arc-08-06-reproducibility-bundle-gate | arc-08-01-quality-gate |
| 24.5 | arc-09-03-academic-integrity-gate | arc-09-01-paper-review-loop |
| 26 | arc-10-01-claim-evidence-trace-gate | arc-07-04-paper-revision |
| 27 | arc-10-04-numeric-truth-gate (pre-gate for stage 26) | arc-07-04-paper-revision |
| 27.5 | arc-10-02-figure-quality-gate | arc-08-03-export-publish (then re-run stages 23–25) |
| 28 | arc-10-03-submission-format-gate | arc-08-03-export-publish (then re-run stages 23–25) |
| 28.5 | arc-10-05-final-acceptance-gate | arc-10-03-submission-format-gate |

---

## Noncritical Stages

**Stage 0** (arc-00-04-environment-probe) is **CRITICAL and BLOCKING** — failure here blocks the entire pipeline. Checks:
- Execution backend (GPU/CPU/SSH) — required for experiments
- LaTeX compilation (pdflatex + bibtex) — required for paper export
- Network connectivity (doi.org + arxiv.org) — required for literature verification

**Stage 4** (arc-02-02-literature-collect) is **CRITICAL and BLOCKING on hallucinations** — any unverifiable DOI/arXiv ID triggers `E04_HALLUCINATION_DETECTED`.

The intended paper-quality floor is also explicit: references should normally reach at least 30 verified items, with at least 20% from the last 5 years, unless the topic genuinely lacks recent literature. This is a quality target for writing/review loops, not a license to fabricate citations.

**Stage 20** (arc-08-01), **Stage 20.5** (arc-00-06), **Stage 21** (arc-08-02), and **Stage 24** (arc-09-01) are noncritical for iteration bookkeeping, but only Stage 20.5, 21 and 24 may truly be skipped. Stage 20 must still be approved before Stage 22 export or any later stage can claim valid late-stage handoff or final acceptance; failures or degraded-local review outcomes must be logged honestly.

**Stage 25** (arc-09-02-paper-polish) is **always critical** — De-AI patterns and structural failures MUST block export.

**Stage 23** (arc-08-04) blocks on hallucinated citations, unverifiable citations, or unjustified citation-threshold failures (`E23`, `E23_CITATION_THRESHOLD`) — this check runs before late-stage gates to catch fake or insufficient references early.

**New Critical Gates (Stages 7.5, 9.5, 15.5, 17.5, 18.5, 21.5, 24.5, 28.5):**
- Stage 7.5 (arc-03-03): Blocks non-novel research gaps from entering hypothesis generation (E07B)
- Stage 9.5 (arc-04-04): Blocks coding until reproducibility contract is explicit (E09B)
- Stage 15.5 (arc-06-03): Blocks writing until claim-scope is frozen and evidence-linked (E15B)
- Stage 17.5 (arc-07-05): Blocks peer review until structure/marker/anti-slop compliance (E17B)
- Stage 18.5 (arc-08-05): Blocks downstream propagation of bibliography defects (E18B)
- Stage 21.5 (arc-08-06): Blocks export if reproducibility bundle is incomplete (E21B)
- Stage 24.5 (arc-09-03): Blocks camera-ready if integrity/disclosure issues exist (E24B)
- Stage 28.5 (arc-10-05): Final aggregate gate — blocks "success" if any critical upstream gate failed (E28B)

**Stages 26–28** are **always critical** late-stage gates:
- Stage 26 (arc-10-01): block on unsupported, untraceable, or direction-conflicted claims (E26, `E26_DIRECTION_CONFLICT`), and on numeric pre-gate failures (`E26_NUMERIC_*`) from `arc-10-04`
- Stage 27 (arc-10-02): block on figure quality or authenticity failures against the polished package (E27)
- Stage 28 (arc-10-03): block on numeric-truth, submission format, citation-threshold, figure-authenticity, or reproducibility noncompliance (E28, `E28_NUMERIC_PRECHECK_FAIL`)

---

## Decision Loop

- `proceed` → next stage
- `refine` → arc-05-02 (counts against cap)
- `pivot` → arc-03-02 (counts against cap)
- **Cap: 2 total pivot/refine cycles**, then forced proceed with `quality_warning.txt`

---

## Phase I: Review & Polish (Stages 24–25)

### arc-09-01-paper-review-loop
External adversarial multi-round review. Breaks the self-review trap. It reviews the stable stage-22 package, records remediation handoff items for downstream execution, and does not mutate the canonical package in place. If no independent reviewer is available, the stage downgrades to `review_mode: degraded_local` and records that downgrade explicitly instead of silently pretending independence. Degraded-local review may inform downstream fixes, but it cannot waive blocking concerns for citation authenticity, numeric truth, claim traceability, figure authenticity, or submission-format compliance.

| Constant | Value |
|----------|-------|
| `MAX_ROUNDS` | 40 |
| `POSITIVE_THRESHOLD` | score ≥ 6/10 AND verdict "ready" or "almost" |
| `REVIEWER_ROLE` | `external_adversarial_reviewer` |
| `REVIEWER_MODEL` | `gpt-5.4` |
| `REVIEWER_PROVIDER` | `codex_mcp` |
| `HUMAN_CHECKPOINT` | `false` |
| `REVIEWER_DIFFICULTY` | `nightmare` |

**Reviewer mode:**
- `nightmare`: Codex/GPT-5.4 MCP independently verifies paper, code, citations, numeric claims, and figure provenance; if MCP is unavailable, the stage downgrades honestly to `degraded_local` and logs that downgrade explicitly

### arc-09-02-paper-polish
Dedicated De-AI Polish Pass + Reverse Outline Test + final structural verification.

Key checks:
- No forbidden AI words (delve, pivotal, landscape, etc.)
- Reverse outline forms coherent narrative
- Section word counts within target ranges
- All claims from experiment_summary covered
- No stale section files
- Regenerates the final polished package used by stages 26–28

## Phase J: Late-Stage Gates (Stages 26–28)

### arc-10-01-claim-evidence-trace-gate
Builds a machine-checkable claim-evidence matrix and blocks unsupported/untraceable claims. Stage 26 execution requires pre-gate `arc-10-04`, which writes `stage-26-pre/numeric_truth_report.json`.

### arc-10-04-numeric-truth-gate (pre-gate for Stage 26)
Validates quantitative claim consistency against experiment artifacts before claim-evidence gating.

### arc-10-02-figure-quality-gate
Validates venue-aware hard figure-count thresholds, readability, style consistency, file quality, statistical consistency, data traceability, and figure authenticity against the canonical polished package (`stage-25/deliverables/charts/` plus `stage-25/main.tex`).

### arc-10-03-submission-format-gate
Final submission gate: enforces numeric-truth pre-gate integrity, compile cleanliness, schema-valid template/report contracts, page/anonymity/front-matter checks, figure-gate consistency including authenticity, citation-threshold checks, and complete reproducibility-ready bundle.

### Late-stage canonical handoff
- Template semantics are resolved once at `artifacts/<run_id>/template/template_manifest.json` and reused by stages 16–28
- Stage 22 produces the canonical review/export bundle at `artifacts/<run_id>/stage-22/` (`paper_final.md`, `main.tex`, `main.pdf`, `references.bib`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, `template_manifest.json`, `deliverables/`)
- Stage 23 verifies citations against `artifacts/<run_id>/stage-22/`, emits `artifacts/<run_id>/stage-23/references_verified.bib`, and does not mutate the stage-22 bundle in place
- Stage 24 performs adversarial review against the same stage-22 bundle and records `review_state.json` plus `AUTO_REVIEW.md` under `artifacts/<run_id>/stage-24/`
- Stage 25 regenerates the canonical polished bundle at `artifacts/<run_id>/stage-25/` (`paper_polished.md`, `main.tex`, `main.pdf`, `references.bib`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, `template_manifest.json`, `deliverables/`) from stage-22 content plus the stage-23 verified bibliography
- Stage 27 writes `artifacts/<run_id>/stage-27/figure_quality_report.json`, which must match the polished package and resolved venue-specific figure rules
- Stages 26–28 validate only the stage-25 polished bundle; Stage 26/28 also require intact `stage-26-pre/numeric_truth_report.json`, and Stage 28 writes gate artifacts under `artifacts/<run_id>/stage-28/`
- If stage 27 or 28 rejects, rollback returns to stage 22 so stages 23–25 can be re-run against a refreshed export before the remaining late-stage gates resume; the refreshed export must include a real repaired change, not a no-op loop through the same artifacts

### Why this arrangement
- Review and citation verification happen against a stable exported package
- Polish creates a new canonical package instead of mutating the review package in place
- Trust gates validate the final polished artifacts, not an older draft/export snapshot
- Late rollback preserves the verification chain by forcing stages 23–25 to run again after stage-22 regeneration

## Late-stage acceptance matrix

| Acceptance dimension | Primary stage | Must prove | Blocking rule |
|---------------------|---------------|------------|---------------|
| Environment reality | Stage 0 (`arc-00-04`) | Real execution, local LaTeX, and network are available | Missing critical capability blocks pipeline start |
| Citation authenticity | Stage 23 (`arc-08-04`) | Every citation is verified or reused from verified corpus | Any hallucinated/unverifiable citation blocks |
| Citation sufficiency | Stage 23 / 25 / 28 | Bibliography reaches ≥30 verified refs and ≥20% from last 5 years, or has explicit topic justification | Unjustified failure blocks late-stage handoff |
| Review observability | Stage 24 (`arc-09-01`) | Review artifacts are recorded honestly, including degraded-local fallback when needed, and any discovered blocking concerns remain visible downstream | Missing/false review logging is not acceptable, though stage 24 itself remains noncritical |
| Claim traceability | Stage 26 (`arc-10-01`) | Every important claim maps to evidence, and directional/hypothesis-support wording matches that evidence | Unsupported or direction-conflicted claims block |
| Numeric truth | Stage 26 pre-gate (`arc-10-04`) | Quantitative statements match experiment artifacts, and the pre-gate report remains present whenever Stage 26 progress is claimed | Numeric mismatch or missing pre-gate evidence blocks |
| Figure quality | Stage 27 (`arc-10-02`) | Figures satisfy count, readability, style, and file-quality rules | Critical figure failure blocks |
| Figure authenticity | Stage 27 / 28 | Figures are evidence-bearing and traceable to real experiment outputs | Decorative/fabricated figure blocks |
| Reproducibility semantics | Stage 22 / 25 / 28 | Commands, seeds, environment, and provenance are preserved | Missing or semantically broken reproducibility artifact blocks |
| Submission-format compliance | Stage 28 (`arc-10-03`) | Compile clean, schema-valid, venue-valid final package | Format/package noncompliance blocks |

This matrix is the final acceptance contract for the whole chain: a run is not "successful" unless all blocking dimensions are satisfied.

---

## Canonical Error Codes (E00–E28)

| Code | Stage | Retry Cap | Description |
|------|-------|-----------|-------------|
| E00_CRITICAL_MISSING | ENVIRONMENT_PROBE | 0 | Critical capability missing (execution/LaTeX/network) — BLOCKING |
| E00_NO_EXECUTION_ENV | ENVIRONMENT_PROBE | 0 | No execution backend |
| E00_NO_LATEX | ENVIRONMENT_PROBE | 0 | LaTeX compilation environment missing |
| E00_NETWORK_UNREACHABLE | ENVIRONMENT_PROBE | 1 | doi.org or arxiv.org unreachable |
| E00_PROBE_FAILED | ENVIRONMENT_PROBE | 1 | Probe failed |
| E00_CONDA_NOT_FOUND | ENVIRONMENT_PROBE | 0 | Conda not installed |
| E00_CONDA_ENV_FAILED | ENVIRONMENT_PROBE | 1 | Conda environment creation failed |
| E00_SSH_UNREACHABLE | ENVIRONMENT_PROBE | 0 | SSH remote configured but unreachable |
| E01 | TOPIC_INIT | 0 | — |
| E02 | PROBLEM_DECOMPOSE | 1 | — |
| E03 | SEARCH_STRATEGY | 1 | — |
| E04_INSUFFICIENT_VERIFIED | LITERATURE_COLLECT | 2 | < 50 verified candidates |
| E04_HALLUCINATION_DETECTED | LITERATURE_COLLECT | 0 | Unverifiable DOI/arXiv detected — BLOCKING |
| E04_NETWORK_FAILURE | LITERATURE_COLLECT | 2 | Cannot reach doi.org/arxiv.org for verification |
| E04_NO_DOI_OR_ARXIV | LITERATURE_COLLECT | 0 | Candidate lacks both DOI and arXiv ID |
| E05 | LITERATURE_SCREEN | 0 (→rollback) | — |
| E06 | KNOWLEDGE_EXTRACT | 1 | — |
| E07 | SYNTHESIS | 1 | — |
| E07B | NOVELTY_GAP_GATE | 0 (→rollback) | Novelty gap report missing or blocked core gap |
| E07B_GAP_NOT_NOVEL | NOVELTY_GAP_GATE | 0 | Core gap is already addressed by prior work |
| E07B_EXTERNAL_REVIEW_MISSING | NOVELTY_GAP_GATE | 0 | External adversarial review path unavailable |
| E08 | HYPOTHESIS_GEN | 1 | — |
| E09 | EXPERIMENT_DESIGN | 0 (→rollback) | — |
| E09B | REPRODUCIBILITY_DESIGN_GATE | 0 (→rollback) | Reproducibility design report missing required fields |
| E09B_SEED_POLICY_MISSING | REPRODUCIBILITY_DESIGN_GATE | 0 | Seed policy missing or `num_seeds < 3` |
| E09B_ENV_LOCK_MISSING | REPRODUCIBILITY_DESIGN_GATE | 0 | No dependency/environment lock artifact declared |
| E09B_STATS_METHOD_MISSING | REPRODUCIBILITY_DESIGN_GATE | 0 | Statistical comparison method missing |
| E10 | CODE_GENERATION | 2 | — |
| E11 | RESOURCE_PLANNING | 1 | — |
| E12 | EXPERIMENT_RUN | 2 | >50% runs failed |
| E12_NO_EXECUTION_ENV | EXPERIMENT_RUN | 0 | No execution backend |
| E12_FABRICATION | EXPERIMENT_RUN | 0 | Fabricated data |
| E13 | ITERATIVE_REFINE | 2 | — |
| E13_LOW_COMPLETION | ITERATIVE_REFINE | 1 | Low completion rate |
| E13_REPAIR_EXHAUSTED | ITERATIVE_REFINE | 0 | Max repairs reached |
| E14 | RESULT_ANALYSIS | 1 | — |
| E14_DATA_INVALID | RESULT_ANALYSIS | 0 | Data validation failed |
| E15 | RESEARCH_DECISION | 1 | — |
| E15B | RESULT_CLAIM_GATE | 0 (→rollback) | Claim-scope report missing or not fully classified |
| E15B_UNSUPPORTED_CORE_CLAIM | RESULT_CLAIM_GATE | 0 | Core contribution claim is unsupported |
| E15B_MISSING_CAVEAT | RESULT_CLAIM_GATE | 0 | `allowed_with_caveat` lacks explicit wording |
| E16 | PAPER_OUTLINE | 1 | — |
| E17 | PAPER_DRAFT | 1 | — |
| E17B | WRITING_COMPLIANCE_GATE | 0 (→rollback) | Writing compliance report failed |
| E17B_MISSING_SECTION | WRITING_COMPLIANCE_GATE | 0 | Required sections missing |
| E17B_LIMITATIONS_MISSING | WRITING_COMPLIANCE_GATE | 0 | Limitations section missing or vague |
| E17B_MARKERS_REMAIN | WRITING_COMPLIANCE_GATE | 0 | TODO/FIXME/VERIFY markers remain |
| E17B_AI_PATTERNS | WRITING_COMPLIANCE_GATE | 0 | Forbidden writing patterns remain |
| E18 | PEER_REVIEW | 1 | — |
| E18B | BIBLIOGRAPHY_QUALITY_GATE | 0 (→rollback) | Bibliography quality report failed |
| E18B_MISSING_ENTRIES | BIBLIOGRAPHY_QUALITY_GATE | 0 | In-text citation keys have no entry |
| E18B_UNRESOLVED_ENTRY | BIBLIOGRAPHY_QUALITY_GATE | 0 | Unresolved verification marker remains |
| E18B_DUPLICATE_KEY_CONFLICT | BIBLIOGRAPHY_QUALITY_GATE | 0 | Duplicate citation key conflict |
| E18B_MISSING_FIELDS | BIBLIOGRAPHY_QUALITY_GATE | 0 | Required metadata fields missing |
| E19 | PAPER_REVISION | 1 | — |
| E20 | QUALITY_GATE | 0 (→rollback) | — |
| E21 | KNOWLEDGE_ARCHIVE | 1 | — |
| E21B | REPRODUCIBILITY_BUNDLE_GATE | 0 (→rollback) | Reproducibility bundle report failed |
| E21B_ENTRYPOINT_MISSING | REPRODUCIBILITY_BUNDLE_GATE | 0 | No runnable reproduction entrypoint |
| E21B_ENV_LOCK_MISSING | REPRODUCIBILITY_BUNDLE_GATE | 0 | No dependency/environment lock artifact |
| E21B_RUNTIME_RECORD_MISSING | REPRODUCIBILITY_BUNDLE_GATE | 0 | Runtime environment recording missing |
| E21B_CLAIM_LINK_MISSING | REPRODUCIBILITY_BUNDLE_GATE | 0 | Core claims cannot be traced to bundled artifacts |
| E22 | EXPORT_PUBLISH | 2 | Stage-22 export package failed local LaTeX compilation, reporting, or packaging |
| E23 | CITATION_VERIFY | 1 (blocks late-stage handoff) | Stage-23 citation verification found hallucinated, unverifiable, or unjustifiably insufficient citations |
| E24 | PAPER_REVIEW_LOOP | 1 | Stage-24 review loop state failure, or reviewer downgrade/failure explicitly and honestly logged in review artifacts |
| E24B | ACADEMIC_INTEGRITY_GATE | 0 (→rollback) | Academic integrity report failed |
| E24B_ANONYMITY_LEAK | ACADEMIC_INTEGRITY_GATE | 0 | Anonymous submission leaks identity |
| E24B_DISCLOSURE_MISSING | ACADEMIC_INTEGRITY_GATE | 0 | Required disclosure missing |
| E24B_SELF_CITATION_POLICY | ACADEMIC_INTEGRITY_GATE | 0 | Self-citation policy violation |
| E24B_MARKERS_REMAIN | ACADEMIC_INTEGRITY_GATE | 0 | Integrity/disclosure markers remain |
| E25 | PAPER_POLISH | 1 (blocks late-stage handoff) | Stage-25 polish, citation-stability/thresholds, reproducibility, or package regeneration failure |
| E26 | CLAIM_EVIDENCE_TRACE_GATE | 0 (→rollback) | Unsupported/untraceable claims |
| E26_NUMERIC_MISMATCH | NUMERIC_TRUTH_GATE (pre-gate) | 0 | Quantitative claim mismatches experiment data |
| E26_NUMERIC_UNVERIFIABLE | NUMERIC_TRUTH_GATE (pre-gate) | 0 | Quantitative claim not traceable |
| E27 | FIGURE_QUALITY_GATE | 1 (→rollback) | Figure quality, traceability, or authenticity failed |
| E28 | SUBMISSION_FORMAT_GATE | 2 (→rollback) | Compile, format, package, citation-threshold, figure-authenticity, or reproducibility noncompliance |
| E28B | FINAL_ACCEPTANCE_GATE | 0 (→rollback) | Final acceptance report failed |
| E28B_PACKAGE_INCOMPLETE | FINAL_ACCEPTANCE_GATE | 0 | Final paper package incomplete |
| E28B_CRITICAL_GATE_FAILED | FINAL_ACCEPTANCE_GATE | 0 | One or more critical upstream gates failed |
| E28B_CITATION_VERIFY_MISSING | FINAL_ACCEPTANCE_GATE | 0 | Citation verification missing or failed |
| E28B_INTEGRITY_MISSING | FINAL_ACCEPTANCE_GATE | 0 | Academic integrity output missing or failed |

---

## Prerequisites (Detailed Setup)

### 1. Execution Backend

For GPU experiments:

```bash
# Check CUDA availability
python -c "import torch; print(torch.cuda.is_available())"

# Expected output: True
```

For CPU experiments (slower):

```bash
# Verify Python environment
python --version  # >= 3.9
python -c "import torch; print(torch.__version__)"
```

### 2. Local LaTeX Installation

**macOS:**
```bash
brew install --cask mactex-no-gui
# Or minimal install:
brew install --cask basictex
sudo tlmgr update --self
sudo tlmgr install latexmk
```

**Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install texlive-full texlive-science texlive-publishers
# Or minimal install:
# sudo apt-get install texlive-base texlive-latex-extra texlive-science texlive-publishers
```

**Verify:**
```bash
pdflatex --version  # Should show version
bibtex --version     # Should show version
```

### 3. Network Connectivity

```bash
# Required for citation verification
curl -s -o /dev/null -w "%{http_code}" https://doi.org/10.1000/182
# Expected: 200, 301, or 303

curl -s -o /dev/null -w "%{http_code}" https://arxiv.org/abs/2301.00001
# Expected: 200
```

### 4. Python Environment

```bash
# Create conda environment (recommended)
conda create -n arc-research python=3.10
conda activate arc-research

# Install PyTorch (adjust for your system)
conda install pytorch torchvision torchaudio -c pytorch

# Additional packages
pip install numpy scipy matplotlib pandas
```

---

## Troubleshooting

### E00_CRITICAL_MISSING (Stage 0 blocked)

**Symptom:** Pipeline stops immediately with "Critical capabilities missing"

**Fix:**
```bash
# Check what's missing
cat environment_error.md

# Install missing components
# - For LaTeX: see Prerequisites section above
# - For execution: set up conda environment
# - For network: check internet connection
```

### E04_HALLUCINATION_DETECTED (Stage 4 blocked)

**Symptom:** "Unverifiable DOI/arXiv detected"

**Cause:** Some papers in search results lack valid identifiers

**Fix:** Pipeline automatically filters these. If all candidates rejected:
- Check network connectivity
- Try broader search terms
- Verify `doi.org` and `arxiv.org` are reachable

### E22 (Stage 22 LaTeX compile failed)

**Symptom:** "Stage-22 export package failed local LaTeX compilation"

**Common causes:**

1. **Missing template style file:**
   ```bash
   # For ACL/CVPR: download official style files
   # See "Quick Start with Templates" section above
   ```

2. **LaTeX package not installed:**
   ```bash
   # Install missing packages (TeX Live)
   sudo tlmgr install <package-name>
   
   # Or (MiKTeX)
   mpm --install <package-name>
   ```

3. **Compilation errors:**
   ```bash
   # Check compile log
cat artifacts/<run_id>/stage-22/compile_report.json
   ```

### E26_NUMERIC_MISMATCH (Pre-gate for Stage 26)

**Symptom:** "Quantitative claim mismatches experiment data"

**Cause:** Paper claims numbers not found in `experiment_summary.json`

**Fix:**
- Verify `experiment_summary.json` contains all reported metrics
- Check metric keys match exactly (case-sensitive)
- Re-run experiments if data is stale

### E24_CODEX_MCP_REQUIRED (Stage 24 BLOCKED)

**Symptom:** "Codex MCP external review is REQUIRED but unavailable"

**Cause:** Codex MCP not configured or unreachable

**Impact:** Pipeline **BLOCKS** — cannot proceed to late-stage gates without genuine external review

**Fix (REQUIRED):**
```bash
# 1. Ensure Codex is installed and configured
# See: https://github.com/codex/codex-cli

# 2. Verify MCP server is running
codex mcp status

# 3. Confirm GPT-5.4 model access
# Requires appropriate API key and permissions

# 4. Retry pipeline once MCP is available
/arc-00-03-resume
```

**Why this is mandatory:**
- Self-review violates scientific independence
- Local models share context with the executor, creating bias
- The adversarial guarantee requires a genuinely independent reviewer
- No degraded fallback is permitted — fix the MCP connection or abort

---

## FAQ

**Q: Can I run without GPU?**
A: Yes, set `execution.backend: local_cpu`. Experiments will be slower.

**Q: Can I use Overleaf instead of local LaTeX?**
A: No. The pipeline requires local compilation for Stage 22/28. Overleaf is not supported.

**Q: What if I only have some templates?**
A: The pipeline works with available templates. For missing official templates (ACL, CVPR), it will block at Stage 22 with clear instructions on what to download.

**Q: How long does a full run take?**
A: Depends on experiment complexity:
- Simple idea → paper: 2-6 hours
- With real experiments: 4-12 hours (mostly experiment time)
- With iteration loops: add 1-3 hours per pivot/refine cycle

**Q: Can I resume from a checkpoint?**
A: Yes, use `/arc-00-03-resume` or specify `from` parameter in entrypoint.

**Q: Where are outputs saved?**
A: All artifacts in `artifacts/<run_id>/stage-XX/` directories. Final package at `artifacts/<run_id>/stage-28/`.

---

## Development

### Adding New Templates

See `templates/README.md` for template contribution guidelines.

### Testing Skills

```bash
# Run environment probe
/arc-00-04-environment-probe

# Check specific skill
/arc-00-02-status
{
  "run_id": "your-run-id"
}
```

### Debugging

Set `auto_approve_gates: false` to manually review each gate decision.

---

## License

This project is part of the AutoResearchClaw ecosystem. See LICENSE file for details.

## Citation

If you use this pipeline in your research, please cite:

```bibtex
@software{arc_skills,
  title = {ARC-Skills: Automated Research Chain},
  year = {2026},
  url = {https://github.com/researchclaw/arc-skills}
}
```
