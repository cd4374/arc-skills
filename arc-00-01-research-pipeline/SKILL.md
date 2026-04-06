---
name: arc-00-01-research-pipeline
description: Pure skills orchestrator for the ARC pipeline with 28 numbered stages (1-28) plus Stage 0 environment probe. This is the sole user-facing entrypoint and supports exactly two modes: idea_start and existing_project_improvement.
metadata:
  category: orchestrator
  trigger-keywords: "arc,pipeline,skills-only,research workflow,run research"
  applicable-stages: "1-28"
  priority: "1"
  version: "4.2"
  author: researchclaw
---

## Purpose
Chain all numbered stages (1–28) plus Stage 0 in canonical order, enforce the state machine, handle gates, rollback, retry, and the decision loop. This is the **sole orchestration authority** — no other skill writes state files.

## Global Output Invariants

A run is only successful if the final output is a **high-quality, verifiable, reproducible, authentic academic paper**. The orchestrator MUST preserve these non-negotiable invariants across every stage, retry loop, rollback, and late-stage regeneration:

- Real execution evidence only; no fabricated experiments, metrics, or claims
- Real local LaTeX compilation only; no fake PDF or placeholder export path
- Verified citations only; hallucinated/unverifiable references are blocking
- Final bibliography should satisfy at least 30 verified references and at least 20% references from the last 5 years, unless explicit topic constraints are documented
- Figures must satisfy venue-aware count, quality, style, traceability, and authenticity requirements
- Reproducibility metadata must remain present and semantically intact
- Missing critical execution / LaTeX / network capability is always blocking

The orchestrator MUST fail, rollback, or block when these invariants are violated; it must never silently degrade them to keep the chain moving.

---

## Inputs
- `mode` (optional, default `idea_start`): `idea_start` or `existing_project_improvement`
- `topic` (required when `mode=idea_start`): single-sentence research idea
- `project_path` (required when `mode=existing_project_improvement`): existing paper/code project to diagnose and improve
- `run_id` (optional): default `rc-YYYYMMDD-HHMMSS`
- `from` (optional): skill ID to resume from (e.g., `arc-04-02-code-generation`)
- `target_venue` (optional, default `neurips`): target submission venue or journal alias
- `template_version` (optional): explicit template version such as `neurips_2025`, `iclr_2026`, `ieeetran_journal`, or `pattern_recognition`
- `publication_type` (optional): `conference` or `journal`
- `submission_mode` (optional): `anonymous`, `camera_ready`, or `preprint`
- `auto_approve_gates` (optional, default false)
- `skip_noncritical` (optional, default true)

## Mode Dispatch

This skill is the **only user-facing pipeline entrypoint**.

| `mode` | Required primary input | Behavior |
|---|---|---|
| `idea_start` | `topic` | Default path. Run Stage 0, then execute the full pipeline from scoping through late-stage gates. |
| `existing_project_improvement` | `project_path` | Existing paper/code improvement path. Run Stage 0, then delegate paper/code re-check and iterative improvement to `arc-00-05-project-recheck`. |

Paper/code re-check is not a third mode and is not a separate user-facing entry path. It belongs entirely under `existing_project_improvement`.

**Mode validation rules:**
- Reject any mode other than `idea_start` or `existing_project_improvement`
- Reject `idea_start` if `topic` is missing
- Reject `existing_project_improvement` if `project_path` is missing
- Do not treat `project_path` as a substitute for `topic` in `idea_start`

---

## Stage 0: Environment Probe (BLOCKING)

Stage 0 (`arc-00-04-environment-probe`) runs BEFORE Stage 1 to detect ALL critical capabilities:

1. **Execution Backend** — Required for Stages 10-13
2. **LaTeX Compilation** — Required for Stages 22, 28
3. **Network Connectivity** — Required for Stages 3-4 (DOI/arXiv verification)

**Blocking Logic:**

| Condition | Action |
|-----------|--------|
| `pipeline_capable: true` | Proceed to Stage 1 |
| `pipeline_capable: false` | **ABORT immediately** with `E00_CRITICAL_MISSING` |
| `execution_capable: false` | Block — cannot run real experiments |
| `latex_capable: false` | Block — cannot generate PDF without local LaTeX |
| `network_capable: false` | Block — cannot verify literature authenticity |

**Anti-Hallucination Guarantee**: The pipeline will NEVER proceed if it cannot verify the authenticity of:
- Experiment data (execution backend required)
- Paper PDF (LaTeX required)
- Literature citations (network + DOI/arXiv verification required)

---

## State Files (write authority: orchestrator only)

### `pipeline_state.json`
```json
{
  "run_id": "rc-20260405-143022",
  "current_stage": 5,
  "current_skill": "arc-02-03-literature-screen",
  "status": "running",
  "pivot_count": 0,
  "started_at": "2026-04-05T14:30:22Z",
  "updated_at": "2026-04-05T14:35:01Z"
}
```

### `checkpoint.json` (written on stage done)
```json
{"last_completed_stage": 4, "last_completed_name": "LITERATURE_COLLECT", "run_id": "...", "timestamp": "..."}
```

### `stage_history.jsonl` (append-only)
```jsonl
{"timestamp":"...","event_type":"stage_start","stage_number":1,"stage_skill":"arc-01-01-topic-init","status":"running"}
{"timestamp":"...","event_type":"stage_end","stage_number":1,"stage_skill":"arc-01-01-topic-init","status":"done","decision":"proceed","elapsed_sec":105}
```

### `decision_history.json` (append on pivot/refine)
```json
[{"attempt":1,"decision":"refine","rollback_target":"ITERATIVE_REFINE","rollback_stage_num":13,"timestamp":"..."}]
```

### `pipeline_summary.json` (written on pipeline end)
```json
{"run_id":"...","stages_executed":28,"stages_done":28,"stages_failed":0,"final_status":"done","generated":"..."}
```

---

## Stage Chain (28 stages + Stage 0)

| # | Skill ID | Stage Name | Gate? | Noncritical? | Blocking? | Requirements |
|---|----------|------------|-------|--------------|-----------|--------------|
| 0 | arc-00-04-environment-probe | ENVIRONMENT_PROBE | **BLOCKING** | — | **YES** | Execution + LaTeX + Network all required |
| 1 | arc-01-01-topic-init | TOPIC_INIT | — | — | — | — |
| 2 | arc-01-02-problem-decompose | PROBLEM_DECOMPOSE | — | — | — | — |
| 3 | arc-02-01-search-strategy | SEARCH_STRATEGY | — | — | — | Network required |
| 4 | arc-02-02-literature-collect | LITERATURE_COLLECT | **BLOCKING** | — | **YES (on hallucination)** | DOI/arXiv verification mandatory |
| 5 | arc-02-03-literature-screen | LITERATURE_SCREEN | **GATE** | — | — | — |
| 6 | arc-02-04-knowledge-extract | KNOWLEDGE_EXTRACT | — | — | — | — |
| 7 | arc-03-01-synthesis | SYNTHESIS | — | — | — | — |
| 8 | arc-03-02-hypothesis-gen | HYPOTHESIS_GEN | — | — | — | — |
| 9 | arc-04-01-experiment-design | EXPERIMENT_DESIGN | **GATE** | — | — | — |
| 10 | arc-04-02-code-generation | CODE_GENERATION | — | — | — | Requires Stage 0 execution pass |
| 11 | arc-04-03-resource-planning | RESOURCE_PLANNING | — | — | — | Requires Stage 0 execution pass |
| 12 | arc-05-01-experiment-run | EXPERIMENT_RUN | — | — | — | **REQUIRES execution backend** |
| 13 | arc-05-02-iterative-refine | ITERATIVE_REFINE | — | — | — | **REQUIRES execution backend** |
| 14 | arc-06-01-result-analysis | RESULT_ANALYSIS | — | — | — | — |
| 15 | arc-06-02-research-decision | RESEARCH_DECISION | — | — | — | — |
| T | arc-07-00-template-resolve | TEMPLATE_RESOLVE | — | — | — | Resolves venue/journal template contract before writing/export |
| 16 | arc-07-01-paper-outline | PAPER_OUTLINE | — | — | — | Requires resolved template manifest |
| 17 | arc-07-02-paper-draft | PAPER_DRAFT | — | — | — | Requires resolved template manifest |
| 18 | arc-07-03-peer-review | PEER_REVIEW | — | — | — | — |
| 19 | arc-07-04-paper-revision | PAPER_REVISION | — | — | — | — |
| 20 | arc-08-01-quality-gate | QUALITY_GATE | **GATE** | yes | **YES (before stage 22+)** | Must pass before export / citation verify / polish / late-stage gates |
| 21 | arc-08-02-knowledge-archive | KNOWLEDGE_ARCHIVE | — | yes | — | Per-run archive only |
| M | arc-00-06-meta-optimizer | META_OPTIMIZER | — | yes | — | Distills reusable guidance from the current run's real artifacts |
| 22 | arc-08-03-export-publish | EXPORT_PUBLISH | — | — | — | **REQUIRES LaTeX** and an approved Stage-20 quality gate |
| 23 | arc-08-04-citation-verify | CITATION_VERIFY | — | — | — | — |
| 24 | arc-09-01-paper-review-loop | PAPER_REVIEW_LOOP | — | yes | — | — |
| 25 | arc-09-02-paper-polish | PAPER_POLISH | — | — | — | — |
| 26 | arc-10-01-claim-evidence-trace-gate | CLAIM_EVIDENCE_TRACE_GATE | **GATE** | — | — | Requires pre-gate `arc-10-04` and evidence-direction consistency |
| 27 | arc-10-02-figure-quality-gate | FIGURE_QUALITY_GATE | **GATE** | — | — | Validates `stage-25/deliverables/charts/` against the polished package |
| 28 | arc-10-03-submission-format-gate | SUBMISSION_FORMAT_GATE | **GATE** | — | — | **REQUIRES LaTeX compile pass** and intact numeric pre-gate evidence |

---

## Late-Stage Canonical Handoff

The authoritative paper/package changes exactly twice in the back half of the pipeline:
0. Template semantics are resolved once into `artifacts/<run_id>/template/template_manifest.json` by `arc-07-00-template-resolve`, and all later writing/export/gate stages must reuse that manifest rather than guessing venue rules
1. Stage 22 exports the canonical review package at `stage-22/` (`paper_final.md`, `main.tex`, `main.pdf`, `references.bib`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, `deliverables/`, `template_manifest.json`)
2. Stage 23 verifies citations against that stage-22 package and produces `references_verified.bib` without mutating stage-22 artifacts in place
3. Stage 24 reviews that same stage-22 package without mutating it in place
4. Stage 25 regenerates the canonical polished package at `stage-25/` (`paper_polished.md`, `main.tex`, `main.pdf`, `references.bib`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, `deliverables/`, `template_manifest.json`) using the stage-22 content plus the stage-23 verified bibliography
5. Stages 26–28 validate the stage-25 package only

Late-stage acceptance is cumulative, not substitutive: later gates cannot "make up for" missing authenticity, citation sufficiency, figure authenticity, or reproducibility semantics that should already exist in the exported package.

## Stage 26 Pre-Gate Execution Order

For Stage 26 validation, execution order is strict:
1. Run `arc-10-04-numeric-truth-gate`
2. Require output at `artifacts/<run_id>/stage-26-pre/numeric_truth_report.json`
3. Record Stage 26 as incomplete until both the pre-gate and the main gate succeed
4. If `numeric_truth_gate_pass=true`, continue immediately to `arc-10-01-claim-evidence-trace-gate`
5. If pre-gate fails, treat as a Stage 26 gate failure, do not checkpoint Stage 26 as complete, and rollback to stage 19
6. Resume/status logic must treat `stage-26-pre/numeric_truth_report.json` as required integrity evidence whenever Stage 26 progress is claimed

---

## State Machine

```
pending → running → done | blocked_approval | failed
blocked_approval → approved | rejected | paused
approved → done
rejected → pending (rollback)
failed → retrying | paused
retrying → running
paused → running
done → (terminal)
```

---

## Blocking Stages (No Rollback)

These stages BLOCK the pipeline and cannot rollback — they require user intervention:

| Stage | Blocking Condition | Required Fix |
|-------|-------------------|--------------|
| Stage 0 | `pipeline_capable: false` | Install missing execution/LaTeX/network |
| Stage 4 | `E04_HALLUCINATION_DETECTED` | Remove unverifiable literature |
| Stage 20 | Stage-20 quality gate not approved | Fix paper quality issues before any stage-22+ handoff |

## Gate Rollback

| Gate (rejected) | Rollback Target |
|-----------------|----------------|
| Stage 0 arc-00-04 | **N/A (BLOCKING)** — fix missing capabilities, then re-run pipeline |
| Stage 4 arc-02-02 (hallucination) | **N/A (BLOCKING)** — unverifiable entries rejected, fetch more candidates |
| Stage 5 arc-02-03 | Stage 4 arc-02-02 |
| Stage 9 arc-04-01 | Stage 8 arc-03-02 |
| Stage 20 arc-08-01 | Stage 16 arc-07-01 |
| Stage 26 arc-10-01 | Stage 19 arc-07-04 |
| Stage 26 pre-gate arc-10-04 | Stage 19 arc-07-04 |
| Stage 27 arc-10-02 | Stage 22 arc-08-03 (invalidate stages 23–25 and re-run them before returning to the remaining late-stage gates) |
| Stage 28 arc-10-03 | Stage 22 arc-08-03 (invalidate stages 23–25 and re-run them before returning to the remaining late-stage gates) |

On rollback: version existing directories (`stage-04/` → `stage-04_v1/`), do not overwrite checkpoint. If rollback returns to stage 22 from stages 27 or 28, version and invalidate `stage-23/`, `stage-24/`, and `stage-25/` so they are re-executed against the refreshed export. The regenerated stage-22 package MUST differ by repaired inputs, corrected content, or refreshed package artifacts before stages 23–28 can be treated as meaningfully re-run; a no-op late-stage loop is not an acceptable resolution.

---

## Decision Loop (Stage 15)

| Decision | Next Stage | Counts Against Cap |
|----------|-----------|-------------------|
| `proceed` | arc-07-01 | no |
| `refine` | arc-05-02 | yes |
| `pivot` | arc-03-02 | yes |

**MAX_DECISION_PIVOTS = 2**. On cap reached: force `proceed`, write `quality_warning.txt` to run root.

---

## Noncritical Stages

- **Stage 20** (arc-08-01): may be treated as noncritical for logging/iteration purposes, but it MUST be approved before Stage 22 export or any later stage can claim final acceptance; `skip_noncritical=true` does not waive this prerequisite
- **Stage 21** (arc-08-02): fail → continue with `skip_noncritical=true`
- **Stage 24** (arc-09-01): fail → continue (noncritical — downgrade or failure must be logged honestly in review artifacts, and any discovered downstream blocking concerns must remain visible rather than implied to be waived)

## Always Blocking Stages

These stages ALWAYS block the pipeline (no skip, no continue):

- **Stage 0** (arc-00-04): fail → **ABORT** (E00_CRITICAL_MISSING — missing execution/LaTeX/network)
- **Stage 4** (arc-02-02): hallucination → **BLOCK** (E04_HALLUCINATION_DETECTED — unverifiable literature)
- **Stage 23** (arc-08-04): fail → **blocks** (E23 — citation verification found hallucinated or unverifiable citations; `E23_CITATION_THRESHOLD` blocks on unjustifiably insufficient citations)
- **Stage 25** (arc-09-02): fail → **blocks** (E25 — polish, citation thresholds/stability, reproducibility, or package regeneration failure)
- **Stage 26** (arc-10-01): fail → **blocks** (E26 — unsupported, untraceable, or direction-conflicted claims)
- **Stage 27** (arc-10-02): fail → **blocks** (E27 — figure quality, traceability, or authenticity failure against the polished package)
- **Stage 28** (arc-10-03): fail → **blocks** (E28 — numeric-truth, compile, format, package, citation, figure-authenticity, or reproducibility noncompliance)

---

## Per-Stage Retry Caps

| Stage | Max Retries |
|-------|------------|
| 0, 4 (hallucination), 5, 9, 20, 26 | 0 |
| 4 (network failure), 10, 12, 13, 28 | 2 |
| 24, 25, 27 | 1 |
| All others | 1 |

---

## Error Code Registry (E00–E28)

Each stage emits its code on failure. Orchestrator maps: retryable → retry; non-retryable → fail or rollback.

**Stage 24 (Paper Review Loop, noncritical):**
- `E24`: Stage-24 review loop state could not be written or resumed cleanly
- `E24_REVIEWER_UNAVAILABLE`: No independent reviewer is available; continue only with explicit, honest `review_mode: degraded_local` logging, with downstream blockers preserved for later critical gates
- `E24_CODEX_MCP_UNAVAILABLE`: Intended Codex/GPT-5.4 MCP reviewer path was unavailable; continue only with explicit degraded-local logging

**Entry-mode validation:**
- `E00_INVALID_MODE`: `mode` is neither `idea_start` nor `existing_project_improvement`
- `E00_MISSING_TOPIC`: `mode=idea_start` but `topic` was not provided
- `E00_MISSING_PROJECT_PATH`: `mode=existing_project_improvement` but `project_path` was not provided

**Stage 0 (Environment Probe):**
- `E00_CRITICAL_MISSING`: Critical capability missing (execution/LaTeX/network) — BLOCKING, no retry
- `E00_NO_EXECUTION_ENV`: No execution backend — BLOCKING
- `E00_NO_LATEX`: LaTeX compilation missing — BLOCKING
- `E00_NETWORK_UNREACHABLE`: doi.org/arxiv.org unreachable — retry once, then BLOCK

**Stage 4 (Literature Collect):**
- `E04_HALLUCINATION_DETECTED`: Unverifiable DOI/arXiv detected — BLOCKING, no retry
- `E04_INSUFFICIENT_VERIFIED`: < 50 verified candidates — retry allowed
- `E04_NETWORK_FAILURE`: Cannot reach DOI/arXiv verification servers — retry allowed

**Trust Gates (Stages 26-28):**
- `E26`: CLAIM_EVIDENCE_TRACE_GATE failed (unsupported, untraceable, or direction-conflicted claims)
- `E26_DIRECTION_CONFLICT`: CLAIM_EVIDENCE_TRACE_GATE failed because evidence contradicts the paper's stated direction, ranking, or hypothesis-support claim
- `E26_NUMERIC_MISMATCH`: NUMERIC_TRUTH pre-gate failed (quantitative claim mismatches experiment outputs)
- `E26_NUMERIC_UNVERIFIABLE`: NUMERIC_TRUTH pre-gate failed (quantitative claim not traceable to experiment artifacts)
- `E26_NUMERIC_ROUNDING_POLICY_VIOLATION`: NUMERIC_TRUTH pre-gate failed (rounding/formatting changed claim meaning)
- `E26_NUMERIC_PRECHECK_FAILED`: CLAIM_EVIDENCE_TRACE_GATE could not start because the numeric pre-gate report was missing or failed
- `E27`: FIGURE_QUALITY_GATE failed (figure quality, traceability, or authenticity against the polished package)
- `E28`: SUBMISSION_FORMAT_GATE failed (numeric-truth, compile, format, package, citation, figure-authenticity, or reproducibility noncompliance)
- `E28_NUMERIC_PRECHECK_FAIL`: SUBMISSION_FORMAT_GATE failed because `stage-26-pre/numeric_truth_report.json` was missing or no longer passing

---

## Meta-Experience Integration

The orchestrator may consult `arc-00-06-meta-optimizer` without making it mandatory for correctness:
- **Read use:** before Stage 1 in `idea_start` mode, or before delegating to `arc-00-05-project-recheck` in `existing_project_improvement` mode, it may ask `arc-00-06-meta-optimizer` to summarize reusable guidance from current real run artifacts or from explicitly provided prior artifacts
- **Write use:** after Stage 21 knowledge archive and before Stage 22 export handoff, it may call `arc-00-06-meta-optimizer` in write mode to distill validated directions, recurring failure patterns, reviewer pain points, and effective remediation patterns from the current run
- **Purity rule:** no built-in repository memory store is assumed; any reuse must come from explicit real artifacts rather than a shipped non-skill directory

## Related commands
- `/arc-00-02-status` — inspect `pipeline_state.json`, `stage_history.jsonl`
- `/arc-00-03-resume` — continue from `checkpoint.json`
