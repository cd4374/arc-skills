# arc-skills v4.1

Pure-skills research pipeline for AutoResearchClaw. **Executable as a standalone skill chain by any AI agent** — the workflow is agent-portable, while successful execution still depends on real local capabilities such as Python execution, network access, and local LaTeX.

## Design Philosophy

**Skill = Task Definition + Output Contract + Acceptance Rules.**

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
| `arc-00-01-research-pipeline` | Full chain orchestrator for Stage 0 + stages 1–28 (Phase 0-J) | N/A (protocol def) |
| `arc-00-02-status` | Read-only state inspector | Yes — 7 steps |
| `arc-00-03-resume` | Resume from checkpoint | Yes — 7 steps |
| `arc-00-04-environment-probe` | Environment detection (execution + local LaTeX + network) — MUST run first, BLOCKING on missing capabilities | Yes — 11 steps |
| `arc-00-05-project-recheck` | Unified project diagnostic + iterative improvement. Analyzes existing projects (ARC or external), executes refactoring for external projects, delegates all fixes to main pipeline skills, iterates until quality target or max iterations, then passes all trust gates. CORE INVARIANT: always produces high-quality verifiable reproducible paper with LaTeX + PDF. | Yes — 14 steps |
| `arc-09-01-paper-review-loop` | External adversarial review (provider-neutral, multi-round) | Yes — 9 steps |
| `arc-09-02-paper-polish` | De-AI Polish + Reverse Outline + final checks | Yes — 7 steps |

**All 28 stage skills** now have complete sections: Purpose · Quality Contract · Inputs · Outputs · **Procedure** · Validation · Error Codes · Retry Policy · State Transition. Gate/decision stages additionally have Gate Behavior and/or Rollback Contract.

---

## Stage Map

| Phase | Skills | Description |
|-------|--------|-------------|
| **0: Environment** | arc-00-04 | Execution + LaTeX + Network detection (BLOCKING on missing capabilities) |
| A: Scoping | arc-01-01 → arc-01-02 | Topic → Problem decomposition |
| B: Literature | arc-02-01 → arc-02-04 | Search → Collect (BLOCKING on unverifiable DOI/arXiv) → Screen → Extract |
| C: Synthesis | arc-03-01 → arc-03-02 | Synthesis → Hypothesis generation |
| D: Experiment Design | arc-04-01 → arc-04-03 | Design (GATE) → Code gen → Resource plan |
| E: Execution | arc-05-01 → arc-05-02 | Run experiments → Iterative refine (with auto-repair) |
| F: Analysis | arc-06-01 → arc-06-02 | Result analysis (with data validation) → Research decision (pivot/refine/proceed) |
| G: Writing | arc-07-01 → arc-07-04 | Outline → Draft (10-step with De-AI + Reverse Outline) → Review → Revision |
| H: Finalization | arc-08-01 → arc-08-04 | Quality (GATE) → Archive → Export (local LaTeX + reproducibility bundle) → Verify (multi-layer) |
| I: Review & Polish | arc-09-01 → arc-09-02 | External Review Loop → De-AI Polish (citation-stable, verified bib preserved) |
| J: Trust Gates | arc-10-01 → arc-10-03 | Claim-Evidence Trace (GATE, requires arc-10-04 numeric-truth pre-gate with output `stage-26-pre/numeric_truth_report.json`) → Figure Quality (GATE) → Submission Format (GATE, checks explicit reproducibility artifact) |

---

## Gate Stages (6 top-level + 1 pre-gate)

| Skill | Rollback on Reject |
|-------|-------------------|
| arc-00-04-environment-probe | **N/A (blocking)** — pipeline cannot start without execution/LaTeX/network |
| arc-02-02-literature-collect | **N/A (blocking on hallucination)** — unverifiable citations rejected per-entry |
| arc-02-03-literature-screen | arc-02-02-literature-collect |
| arc-04-01-experiment-design | arc-03-02-hypothesis-gen |
| arc-08-01-quality-gate | arc-07-01-paper-outline |
| arc-10-01-claim-evidence-trace-gate | arc-07-04-paper-revision |
| arc-10-04-numeric-truth-gate (pre-gate for stage 26) | arc-07-04-paper-revision |
| arc-10-02-figure-quality-gate | arc-08-03-export-publish (then re-run stages 23–25) |
| arc-10-03-submission-format-gate | arc-08-03-export-publish (then re-run stages 23–25) |

---

## Noncritical Stages

**Stage 0** (arc-00-04-environment-probe) is **CRITICAL and BLOCKING** — failure here blocks the entire pipeline. Checks:
- Execution backend (GPU/CPU/SSH) — required for experiments
- LaTeX compilation (pdflatex + bibtex) — required for paper export
- Network connectivity (doi.org + arxiv.org) — required for literature verification

**Stage 4** (arc-02-02-literature-collect) is **CRITICAL and BLOCKING on hallucinations** — any unverifiable DOI/arXiv ID triggers `E04_HALLUCINATION_DETECTED`.

**Stage 20** (arc-08-01), **Stage 21** (arc-08-02), and **Stage 24** (arc-09-01) are noncritical — failures or degraded-local review outcomes are logged but do not abort deliverables.

**Stage 25** (arc-09-02-paper-polish) is **always critical** — De-AI patterns and structural failures MUST block export.

**Stage 23** (arc-08-04) blocks on hallucinated citations (E23) — this check runs before Trust Gates to catch fake references early.

**Stages 26–28** are **always critical** trust gates:
- Stage 26 (arc-10-01): block on unsupported or untraceable claims (E26), and on numeric pre-gate failures (`E26_NUMERIC_*`) from `arc-10-04`
- Stage 27 (arc-10-02): block on figure quality failures (E27)
- Stage 28 (arc-10-03): block on submission format noncompliance (E28)

---

## Decision Loop

- `proceed` → next stage
- `refine` → arc-05-02 (counts against cap)
- `pivot` → arc-03-02 (counts against cap)
- **Cap: 2 total pivot/refine cycles**, then forced proceed with `quality_warning.txt`

---

## Phase I: Review & Polish (Stages 24–25)

### arc-09-01-paper-review-loop
External adversarial multi-round review. Breaks the self-review trap. It reviews the stable stage-22 package, records remediation handoff items for downstream execution, and does not mutate the canonical package in place. If no independent reviewer is available, the stage downgrades to `review_mode: degraded_local` and records that downgrade explicitly instead of silently pretending independence.

| Constant | Value |
|----------|-------|
| `MAX_ROUNDS` | 4 |
| `POSITIVE_THRESHOLD` | score ≥ 6/10 AND verdict "ready" or "almost" |
| `REVIEWER_ROLE` | `external_adversarial_reviewer` |
| `REVIEWER_MODEL` | `executor-selected` |
| `REVIEWER_PROVIDER` | `executor-selected` |
| `HUMAN_CHECKPOINT` | `false` |
| `REVIEWER_DIFFICULTY` | `medium` \| `hard` \| `nightmare` |

**Difficulty modes:**
- `medium`: executor supplies curated context to an independent reviewer if available
- `hard`: + Reviewer Memory + Debate Protocol
- `nightmare`: reviewer verifies paper/code/results directly when the executing agent can support it

### arc-09-02-paper-polish
Dedicated De-AI Polish Pass + Reverse Outline Test + final structural verification.

Key checks:
- No forbidden AI words (delve, pivotal, landscape, etc.)
- Reverse outline forms coherent narrative
- Section word counts within target ranges
- All claims from experiment_summary covered
- No stale section files
- Regenerates the final polished package used by stages 26–28

## Phase J: Trust Gates (Stages 26–28)

### arc-10-01-claim-evidence-trace-gate
Builds a machine-checkable claim-evidence matrix and blocks unsupported/untraceable claims. Stage 26 execution requires pre-gate `arc-10-04`, which writes `stage-26-pre/numeric_truth_report.json`.

### arc-10-04-numeric-truth-gate (pre-gate for Stage 26)
Validates quantitative claim consistency against experiment artifacts before claim-evidence gating.

### arc-10-02-figure-quality-gate
Validates figure readability, statistical consistency, and data traceability.

### arc-10-03-submission-format-gate
Final submission gate: enforces compile cleanliness, venue format, and complete reproducibility-ready bundle.

### Late-stage canonical handoff
- Stage 22 produces the canonical review/export bundle at `artifacts/<run_id>/stage-22/` (`paper_final.md`, `main.tex`, `main.pdf`, `references.bib`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, `deliverables/`)
- Stage 23 verifies citations against `artifacts/<run_id>/stage-22/`, emits `artifacts/<run_id>/stage-23/references_verified.bib`, and does not mutate the stage-22 bundle in place
- Stage 24 performs adversarial review against the same stage-22 bundle and records review artifacts under `artifacts/<run_id>/stage-24/`
- Stage 25 regenerates the canonical polished bundle at `artifacts/<run_id>/stage-25/` (`paper_polished.md`, `main.tex`, `main.pdf`, `references.bib`, `compile_report.json`, `manifest.json`, `reproducibility_report.json`, `deliverables/`) from stage-22 content plus the stage-23 verified bibliography
- Stages 26–28 validate only the stage-25 polished bundle, with Stage 28 writing gate artifacts under `artifacts/<run_id>/stage-28/`
- If stage 27 or 28 rejects, rollback returns to stage 22 so stages 23–25 can be re-run against a refreshed export before trust gates resume

### Why this arrangement
- Review and citation verification happen against a stable exported package
- Polish creates a new canonical package instead of mutating the review package in place
- Trust gates validate the final polished artifacts, not an older draft/export snapshot
- Late rollback preserves the verification chain by forcing stages 23–25 to run again after stage-22 regeneration

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
| E08 | HYPOTHESIS_GEN | 1 | — |
| E09 | EXPERIMENT_DESIGN | 0 (→rollback) | — |
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
| E16 | PAPER_OUTLINE | 1 | — |
| E17 | PAPER_DRAFT | 1 | — |
| E18 | PEER_REVIEW | 1 | — |
| E19 | PAPER_REVISION | 1 | — |
| E20 | QUALITY_GATE | 0 (→rollback) | — |
| E21 | KNOWLEDGE_ARCHIVE | 1 | — |
| E22 | EXPORT_PUBLISH | 2 | Stage-22 export package failed local LaTeX compilation, reporting, or packaging |
| E23 | CITATION_VERIFY | 1 (blocks export) | Stage-23 citation verification found hallucinated or unverifiable citations |
| E24 | PAPER_REVIEW_LOOP | 1 | Stage-24 review loop state failure, or reviewer downgrade/failure explicitly logged in review artifacts |
| E25 | PAPER_POLISH | 1 (blocks export) | Stage-25 polish, citation-stability, reproducibility, or package regeneration failure |
| E26 | CLAIM_EVIDENCE_TRACE_GATE | 0 (→rollback) | Unsupported/untraceable claims |
| E26_NUMERIC_MISMATCH | NUMERIC_TRUTH_GATE (pre-gate) | 0 | Quantitative claim mismatches experiment data |
| E26_NUMERIC_UNVERIFIABLE | NUMERIC_TRUTH_GATE (pre-gate) | 0 | Quantitative claim not traceable |
| E27 | FIGURE_QUALITY_GATE | 1 (→rollback) | Figure quality or traceability failed |
| E28 | SUBMISSION_FORMAT_GATE | 2 (→rollback) | Compile, format, package, or reproducibility noncompliance |
