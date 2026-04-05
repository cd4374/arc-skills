---
name: arc-00-01-research-pipeline
description: Pure skills orchestrator for the ARC pipeline with 28 numbered stages (1-28) plus Stage 0 environment probe. Run this to execute the full chain from topic to polished paper.
metadata:
  category: orchestrator
  trigger-keywords: "arc,pipeline,skills-only,research workflow,run research"
  applicable-stages: "1-28"
  priority: "1"
  version: "4.1"
  author: researchclaw
---

## Purpose
Chain all numbered stages (1–28) plus Stage 0 in canonical order, enforce the state machine, handle gates, rollback, retry, and the decision loop. This is the **sole orchestration authority** — no other skill writes state files.

---

## Inputs
- `topic` (required): single-sentence research idea
- `run_id` (optional): default `rc-YYYYMMDD-HHMMSS`
- `from` (optional): skill ID to resume from (e.g., `arc-04-02-code-generation`)
- `auto_approve_gates` (optional, default false)
- `skip_noncritical` (optional, default true)

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
| 16 | arc-07-01-paper-outline | PAPER_OUTLINE | — | — | — | — |
| 17 | arc-07-02-paper-draft | PAPER_DRAFT | — | — | — | — |
| 18 | arc-07-03-peer-review | PEER_REVIEW | — | — | — | — |
| 19 | arc-07-04-paper-revision | PAPER_REVISION | — | — | — | — |
| 20 | arc-08-01-quality-gate | QUALITY_GATE | **GATE** | yes | — | — |
| 21 | arc-08-02-knowledge-archive | KNOWLEDGE_ARCHIVE | — | yes | — | — |
| 22 | arc-08-03-export-publish | EXPORT_PUBLISH | — | — | — | **REQUIRES LaTeX** |
| 23 | arc-08-04-citation-verify | CITATION_VERIFY | — | — | — | — |
| 24 | arc-09-01-paper-review-loop | PAPER_REVIEW_LOOP | — | yes | — | — |
| 25 | arc-09-02-paper-polish | PAPER_POLISH | — | — | — | — |
| 26 | arc-10-01-claim-evidence-trace-gate | CLAIM_EVIDENCE_TRACE_GATE | **GATE** | — | — | Requires pre-gate `arc-10-04` |
| 27 | arc-10-02-figure-quality-gate | FIGURE_QUALITY_GATE | **GATE** | — | — | — |
| 28 | arc-10-03-submission-format-gate | SUBMISSION_FORMAT_GATE | **GATE** | — | — | **REQUIRES LaTeX compile pass** |

---

## Late-Stage Canonical Handoff

The authoritative paper/package changes exactly twice in the back half of the pipeline:
1. Stage 22 exports the canonical review package at `stage-22/` (`paper_final.md`, `main.tex`, `main.pdf`, `references.bib`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, `deliverables/`)
2. Stage 23 verifies citations against that stage-22 package and produces `references_verified.bib` without mutating stage-22 artifacts in place
3. Stage 24 reviews that same stage-22 package without mutating it in place
4. Stage 25 regenerates the canonical polished package at `stage-25/` (`paper_polished.md`, `main.tex`, `main.pdf`, `references.bib`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, `deliverables/`) using the stage-22 content plus the stage-23 verified bibliography
5. Stages 26–28 validate the stage-25 package only

## Stage 26 Pre-Gate Execution Order

For Stage 26 trust validation, execution order is strict:
1. Run `arc-10-04-numeric-truth-gate`
2. Require output at `artifacts/<run_id>/stage-26-pre/numeric_truth_report.json`
3. If `numeric_truth_gate_pass=true`, continue to `arc-10-01-claim-evidence-trace-gate`
4. If pre-gate fails, treat as Stage 26 gate failure and rollback to stage 19

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
| Stage 27 arc-10-02 | Stage 22 arc-08-03 (invalidate stages 23–25 and re-run them before returning to trust gates) |
| Stage 28 arc-10-03 | Stage 22 arc-08-03 (invalidate stages 23–25 and re-run them before returning to trust gates) |

On rollback: version existing directories (`stage-04/` → `stage-04_v1/`), do not overwrite checkpoint. If rollback returns to stage 22 from stages 27 or 28, version and invalidate `stage-23/`, `stage-24/`, and `stage-25/` so they are re-executed against the refreshed export.

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

- **Stage 20** (arc-08-01): fail → continue with `skip_noncritical=true`
- **Stage 21** (arc-08-02): fail → continue with `skip_noncritical=true`
- **Stage 24** (arc-09-01): fail → continue (noncritical — downgrade or failure must be logged in review artifacts)

## Always Blocking Stages

These stages ALWAYS block the pipeline (no skip, no continue):

- **Stage 0** (arc-00-04): fail → **ABORT** (E00_CRITICAL_MISSING — missing execution/LaTeX/network)
- **Stage 4** (arc-02-02): hallucination → **BLOCK** (E04_HALLUCINATION_DETECTED — unverifiable literature)
- **Stage 23** (arc-08-04): fail → **blocks** (E23 — citation verification found hallucinated or unverifiable citations)
- **Stage 25** (arc-09-02): fail → **blocks** (E25 — polish, citation-stability, reproducibility, or package regeneration failure)
- **Stage 26** (arc-10-01): fail → **blocks** (E26 — unsupported/untraceable claims)
- **Stage 27** (arc-10-02): fail → **blocks** (E27 — figure quality failure)
- **Stage 28** (arc-10-03): fail → **blocks** (E28 — compile, format, package, or reproducibility noncompliance)

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
- `E24_REVIEWER_UNAVAILABLE`: No independent reviewer is available; continue only with explicit `review_mode: degraded_local` logging

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
- `E26`: CLAIM_EVIDENCE_TRACE_GATE failed (unsupported/untraceable claims)
- `E26_NUMERIC_MISMATCH`: NUMERIC_TRUTH pre-gate failed (quantitative claim mismatches experiment outputs)
- `E26_NUMERIC_UNVERIFIABLE`: NUMERIC_TRUTH pre-gate failed (quantitative claim not traceable to experiment artifacts)
- `E26_NUMERIC_ROUNDING_POLICY_VIOLATION`: NUMERIC_TRUTH pre-gate failed (rounding/formatting changed claim meaning)
- `E26_NUMERIC_PRECHECK_FAILED`: CLAIM_EVIDENCE_TRACE_GATE could not start because the numeric pre-gate report was missing or failed
- `E27`: FIGURE_QUALITY_GATE failed (figure quality or traceability)
- `E28`: SUBMISSION_FORMAT_GATE failed (compile, format, package, or reproducibility noncompliance)

---

## Related commands
- `/arc-00-02-status` — inspect `pipeline_state.json`, `stage_history.jsonl`
- `/arc-00-03-resume` — continue from `checkpoint.json`
