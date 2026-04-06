---
name: arc-00-05-project-recheck
description: Unified project diagnostic + iterative improvement tool. Analyzes existing projects (ARC-structured or external), generates diagnostic reports, executes prioritized fixes, validates improvements, and loops until quality gates pass or max iterations reached. For external projects, it refactors structure into ARC-compatible artifacts while preserving blocking checks instead of fabricating stage completion. Ensures final output includes compilable LaTeX and PDF for a high-quality, verifiable, reproducible research paper.
metadata:
  category: orchestrator
  trigger-keywords: "recheck,diagnose,improve,iterate,project review,refactor,assess,fix,repair,enhance"
  applicable-stages: "0-28 (standalone tool)"
  priority: "1"
  version: "4.1"
  author: researchclaw
---

## Purpose

Unified diagnostic + iterative improvement tool that:

1. **Detects project type**: Determines if project follows ARC structure or is an external project
2. **Analyzes current state**: Diagnoses what's done, incomplete, or problematic
3. **Generates refactoring plan**: For non-ARC projects, maps external structure to ARC standard structure without claiming missing evidence is already complete
4. **Executes improvements**: Applies prioritized fixes with validation
5. **Iterates until quality gates pass**: Loops through diagnose → fix → validate until target reached or max iterations exhausted
6. **Ensures final output**: Produces compilable LaTeX + PDF + verified citations + reproducibility bundle only after all blocking checks really pass

**CORE INVARIANT — This skill MUST satisfy:**
> When execution completes in an agent CLI (Claude Code, Codex, etc.), the output MUST be a **high-quality, verifiable, reproducible, authentic academic paper** with at minimum `main.tex` and compiled `main.pdf`.

This invariant is **non-negotiable** and overrides all other considerations. If at any point the invariant cannot be satisfied, the skill MUST fail with a clear error rather than produce a degraded or fake output.

**This skill does NOT modify core skills chain logic.** It is a standalone diagnostic + improvement tool that can run independently or alongside the main pipeline.

---

## Quality Contract

### CORE INVARIANT (Non-Negotiable)

**FINAL OUTPUT MUST BE:** A high-quality, verifiable, reproducible, authentic academic paper with:

#### Hard Requirements (Must Satisfy All):
- ✅ `main.tex` — LaTeX source file
- ✅ `main.pdf` — Compiled PDF file (compile error-free)
- ✅ `references.bib` — Citations with verifiable DOIs/arXiv IDs
- ✅ `experiment_summary.json` — Real metrics from actual execution (no fabrication)
- ✅ `reproducibility_report.json` — Complete reproducibility documentation
- ✅ **≥30 citations** in references.bib unless topic constraints are explicitly justified
- ✅ **≥20% of citations from recent 5 years** (published 2021-2026) unless topic constraints are explicitly justified
- ✅ Figures satisfy venue-aware count, style, file-quality, traceability, and authenticity requirements

#### Citation Quality Standards:
- Each citation MUST have a verifiable DOI or arXiv ID
- Self-citations limited to ≤20% of total references
- At least 3 different citation sources (journals, conferences, arXiv, books)

#### Figure Authenticity Standards:
- Each figure MUST be evidence-bearing, not decorative filler
- Each figure MUST be traceable to experiment artifacts or verified source material
- Figure style/quality MUST satisfy the resolved template rules

**ANTI-HALLUCINATION:** The paper must contain ZERO:
- ❌ Fabricated experiment data or metrics
- ❌ Unverifiable citations (DOI must resolve)
- ❌ Decorative or fabricated figures presented as evidence
- ❌ Uncompilable LaTeX
- ❌ Unsubstantiated claims without evidence

**If at ANY point this invariant cannot be satisfied, the skill MUST fail with `E_RECHECK_INVARIANT_BROKEN` rather than produce degraded output.**

---

`project_diagnostic_report.json` MUST contain:
- `project_type`: "arc_structured" | "external" | "empty"
- `completeness_score`: 0.0–1.0 based on stage coverage
- `critical_gaps`: array of blocking issues
- `improvement_plan`: prioritized array of actionable items
- `refactoring_plan`: (for external projects) structured migration path to ARC structure

`iteration_state.json` MUST maintain:
- `iteration_count`: current iteration number
- `changes_made`: array of all modifications with validation results
- `gates_passed`: array of required late-stage stages/gates cleared
- `remaining_issues`: array of unresolved problems
- `quality_delta`: improvement in quality score per iteration

**Final output MUST satisfy** (when mode is "full"):
- `main.tex` compiles to `main.pdf` without errors
- `references.bib` contains only verifiable DOI/arXiv entries and satisfies citation-count/recency requirements unless justified
- `experiment_summary.json` contains only real execution metrics (no fabricated data)
- All claims traceable to evidence in artifacts/
- All figures are evidence-bearing and traceable to real experiment artifacts

---

## Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `project_path` | Path to project directory | Yes |
| `run_id` | Existing run ID (for ARC projects) | No |
| `max_iterations` | Maximum improvement iterations (default: 40) | No |
| `target_quality` | Minimum quality threshold (default: 0.80) | No |
| `auto_approve` | Automatically approve passing gates | No (default: false) |

---

## Outputs

### `project_diagnostic_report.json`
```json
{
  "project_type": "arc_structured",
  "project_path": "/path/to/project",
  "run_id": "rc-20260405-143022",
  "assessed_at": "2026-04-06T00:30:00Z",
  "completeness_score": 0.72,
  "structure_score": 0.95,
  "quality_indicators": {
    "has_latex": true,
    "has_pdf": true,
    "has_reproducibility_bundle": true,
    "has_verified_citations": true,
    "has_experiment_code": true,
    "has_experiment_results": true
  },
  "stages_status": {
    "completed": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19],
    "incomplete": [20, 21],
    "missing": [22, 23, 24, 25, 26, 27, 28]
  },
  "critical_gaps": [
    {
      "gap_id": "GAP-001",
      "severity": "blocking",
      "description": "Stage 20 (quality gate) not passed",
      "affected_stages": [20, 22, 23, 24, 25, 26, 27, 28],
      "recommendation": "Run arc-08-01-quality-gate to assess paper quality"
    }
  ],
  "warnings": [
    {
      "warning_id": "WARN-001",
      "severity": "warning",
      "description": "Limited literature coverage (only 23 papers vs. target 50)",
      "recommendation": "Expand literature search in Stage 4"
    }
  ],
  "improvement_plan": [
    {
      "priority": 1,
      "action": "Pass quality gate at Stage 20",
      "estimated_impact": 0.15,
      "steps": ["Review quality_report.md", "Address critical issues", "Re-run quality gate"]
    },
    {
      "priority": 2,
      "action": "Complete remaining stages 22-28",
      "estimated_impact": 0.25,
      "steps": ["Run arc-08-03-export-publish", "Verify citations", "Run review loop", "Polish paper", "Pass required late-stage gates"]
    }
  ],
  "refactoring_plan": null
}
```

### `external_project_diagnostic.json` (for external projects only)
```json
{
  "project_type": "external",
  "original_structure": {
    "has_readme": true,
    "has_code": true,
    "has_paper_draft": false,
    "has_latex": false,
    "has_experiments": true,
    "has_data": false,
    "has_requirements": true,
    "build_system": "setup.py"
  },
  "arc_mapping": {
    "possible_topics": ["Auto-detected from code/comments"],
    "suggested_run_id": "rc-YYYYMMDD-HHMMSS",
    "stage_alignment": [
      {"arc_stage": 1, "external_equivalent": "README.md title", "confidence": 0.8},
      {"arc_stage": 4, "external_equivalent": "requirements.txt", "confidence": 0.6},
      {"arc_stage": 10, "external_equivalent": "src/", "confidence": 0.9}
    ]
  },
  "refactoring_steps": [
    {
      "phase": 1,
      "description": "Initialize ARC structure",
      "actions": [
        "Create artifacts/<run_id>/ directory",
        "Create config.arc.yaml",
        "Map existing code to experiment design"
      ],
      "estimated_complexity": "low"
    },
    {
      "phase": 2,
      "description": "Migrate literature and knowledge",
      "actions": [
        "Extract topic from README/abstracts",
        "Identify related work from comments/docs",
        "Create knowledge synthesis documents"
      ],
      "estimated_complexity": "medium"
    },
    {
      "phase": 3,
      "description": "Integrate experiment code",
      "actions": [
        "Map existing experiments to Stage 10-13",
        "Verify code reproducibility",
        "Create execution environment"
      ],
      "estimated_complexity": "high"
    }
  ],
  "recommended_entry_point": "arc-01-01-topic-init"
}
```

### `iteration_state.json`
```json
{
  "iteration_count": 3,
  "started_at": "2026-04-05T10:00:00Z",
  "last_updated": "2026-04-06T00:30:00Z",
  "target_quality": 0.80,
  "max_iterations": 5,
  "target_reached": false,
  "changes_made": [
    {
      "iteration": 1,
      "timestamp": "2026-04-05T10:15:00Z",
      "action": "Fixed abstract word count (was 250, now 205)",
      "result": "abstract_score: 0.65 -> 0.85",
      "validation_passed": true
    },
    {
      "iteration": 2,
      "timestamp": "2026-04-05T14:20:00Z",
      "action": "Added missing limitations section",
      "result": "limitations_score: 0.0 -> 0.75",
      "validation_passed": true
    }
  ],
  "gates_passed": [],
  "remaining_issues": [
    {
      "issue_id": "ISSUE-001",
      "description": "Related work does not identify 2+ research gaps",
      "severity": "blocking",
      "last_attempt": "iteration 2",
      "suggested_fix": "Add explicit gap analysis subsection"
    }
  ],
  "quality_history": [
    {"iteration": 0, "score": 0.55},
    {"iteration": 1, "score": 0.68},
    {"iteration": 2, "score": 0.72}
  ]
}
```

### `improvement_final_report.json`
```json
{
  "run_id": "rc-20260405-143022",
  "total_iterations": 4,
  "final_quality_score": 0.82,
  "target_reached": true,
  "invariant_satisfied": true,
  "gates_passed": ["citation_verify", "paper_review_loop", "paper_polish", "numeric_truth", "claim_evidence_trace", "figure_quality", "submission_format"],
  "final_outputs": {
    "paper_path": "artifacts/<run_id>/stage-25/main.pdf",
    "latex_path": "artifacts/<run_id>/stage-25/main.tex",
    "citation_report_path": "artifacts/<run_id>/stage-23/verification_report.json",
    "submission_report_path": "artifacts/<run_id>/stage-28/submission_format_report.json",
    "latex_compiles": true,
    "citations_verified": true,
    "reproducibility_bundle": true
  },
  "remaining_issues": []
}
```

### `diagnostic_summary.md`
```markdown
# Project Diagnostic Summary

## Project: /path/to/project
## Run ID: rc-20260405-143022
## Assessed: 2026-04-06T00:30:00Z

### Overall Health: 72%

| Category | Score | Status |
|----------|-------|--------|
| Structure | 95% | ✅ Good |
| Completeness | 65% | ⚠️ Incomplete |
| Quality | 55% | ❌ Needs Work |
| Reproducibility | 80% | ✅ Acceptable |

### Critical Gaps (Blocking)

1. **[GAP-001]** Stage 20 quality gate not passed
   - Impact: Cannot proceed to export stages
   - Fix: Review and address quality issues

2. **[GAP-002]** No verified PDF output
   - Impact: Cannot submit paper
   - Fix: Complete stages 22-28

### Improvement Progress

- Iteration 1: Quality 0.55 → 0.68 (+0.13) ✅
- Iteration 2: Quality 0.68 → 0.72 (+0.04) ✅
- Iteration 3: Quality 0.72 → 0.75 (+0.03) ✅
- Iteration 4: Quality 0.75 → 0.82 (+0.07) ✅ TARGET REACHED

### Recommended Actions

1. **HIGH PRIORITY**: Pass quality gate (Stage 20)
2. **HIGH PRIORITY**: Export paper with LaTeX (Stage 22)
3. **MEDIUM PRIORITY**: Expand literature coverage

### Final Outputs

- Paper: `artifacts/<run_id>/stage-25/main.pdf`
- LaTeX: `artifacts/<run_id>/stage-25/main.tex`
- Citations: `artifacts/<run_id>/stage-25/references.bib` (verified)
- Reproducibility: `artifacts/<run_id>/stage-25/reproducibility_report.json`
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| `project_diagnostic_report.json` valid | Valid JSON with required fields |
| `project_type` correctly identified | One of: "arc_structured", "external", "empty" |
| `completeness_score` in range | 0.0–1.0 |
| All stages accounted | No stage number appears in both completed and missing |
| External projects have refactoring_plan | Non-null with phase descriptions |
| `iteration_state.json` incremented | iteration_count increases on each iteration |
| Hallucination check passed | No fabricated data, no fake citations |
| Stage-23 verification present | `artifacts/<run_id>/stage-23/verification_report.json` exists |
| Stage-25 polished package present | `artifacts/<run_id>/stage-25/main.tex`, `main.pdf`, and `references.bib` exist |
| Stage-28 submission report present | `artifacts/<run_id>/stage-28/submission_format_report.json` exists |
| Late-stage gates recorded | `improvement_final_report.json.gates_passed` includes citation verify, numeric truth, claim trace, figure quality, and submission format passes |
| Invariant explicitly satisfied | `improvement_final_report.json.invariant_satisfied == true` |
| LaTeX compiles cleanly | pdflatex produces PDF without errors |

**Failure**: Missing required output files → `E_RECHECK_01`

---

## Error Codes

| Code | Trigger | Retryable |
|------|---------|-----------|
| `E_RECHECK_INVARIANT_BROKEN` | **CORE INVARIANT VIOLATED** — Cannot produce a verifiable, reproducible, authentic academic paper with LaTeX + PDF output. This is a fatal error that cannot be retried. | **No (abort)** |
| `E_RECHECK_CITATION_COUNT` | Citation count < 30 in references.bib | No (must add more citations) |
| `E_RECHECK_CITATION_RECENCY` | < 20% of citations from recent 5 years | No (must update/add recent citations) |
| `E_RECHECK_01` | Required output files missing | No |
| `E_RECHECK_02` | Project path does not exist | No |
| `E_RECHECK_03` | Cannot read project files | Yes (1 retry) |
| `E_RECHECK_04` | Project type ambiguous | No (requires human clarification) |
| `E_RECHECK_05` | Hallucination detected (fabricated data/citations) | No (abort) |
| `E_RECHECK_06` | LaTeX compilation failure | Yes (fix and retry iteration) |
| `E_RECHECK_07` | Max iterations reached before both target quality and required late-stage gates were achieved | No (report final state) |
| `E_RECHECK_08` | Citation unverifiable | Yes (fix citation or remove claim) |

---

## Retry Policy
Max retries: **1** — retry only for transient file read issues or LaTeX compilation failures.

---

## State Transition

```
idle → diagnosing → improving → validating → done | failed
                    ↓
            iteration_complete
                    ↓
            (loop if more iterations and target not reached)
```

---

## Procedure

### Step 1 — Validate Project Path

```bash
ls -la {project_path}
```

**If path does not exist:**
- Fail with `E_RECHECK_02`
- Report: "Project path does not exist: {project_path}"

**If path exists but is empty:**
- Set `project_type: "empty"`
- Generate minimal refactoring_plan pointing to `arc-01-01-topic-init`
- Proceed to Step 9

### Step 2 — Detect Project Type

Check for ARC structure indicators:

```bash
# ARC project indicators
ls {project_path}/artifacts/ 2>/dev/null
ls {project_path}/artifacts/*/pipeline_state.json 2>/dev/null
ls {project_path}/config.arc.yaml 2>/dev/null

# External project indicators
ls {project_path}/README.md 2>/dev/null
ls {project_path}/src/ 2>/dev/null
ls {project_path}/main.py 2>/dev/null
ls {project_path}/setup.py 2>/dev/null
ls {project_path}/pyproject.toml 2>/dev/null
```

**Decision tree:**
```
├─ artifacts/<run_id>/pipeline_state.json exists? → ARC project
├─ config.arc.yaml exists? → ARC project
├─ src/ or main.py or README.md exists? → External project
└─ None of above → Empty project
```

### Step 3 — Analyze Project Structure

**For ARC-structured projects:**
For each stage directory in `artifacts/<run_id>/`:
1. Check if directory exists and is non-empty
2. Verify required output files are present
3. Check for `quality_report.json` or other quality indicators
4. Note any missing or incomplete stages

**Calculate completeness_score:**
```
completeness_score = (completed_stages + 0.5 * incomplete_stages) / 28
```

**For external projects:**
Detect existing structure:

| Indicator | What It Suggests |
|-----------|------------------|
| `README.md` | Topic/title extraction possible |
| `requirements.txt` | Dependencies can be catalogued |
| `src/` or `lib/` | Code exists for experiment |
| `*.tex` or `*.md` (paper-like) | Existing draft for paper stages |
| `data/` or `dataset/` | Data for experiments |
| `.git/` | Version control present |
| `paper.pdf` or `draft.pdf` | Existing paper output |
| `experiments/` | Already has experiment structure |

**Map to ARC stages:**
- README title → Stage 1 (topic seed only)
- requirements.txt + src/ → evidence for code/dependency inventory, not verified literature completion
- src/ + data/ → candidate inputs for Stages 10-12 after real execution validation
- *.tex draft → candidate inputs for Stages 16-17 after structure/content validation

### Step 3.5 — Execute External Project Refactoring (External projects in "full" mode only)

**If project is external AND mode is "full":**
Execute the refactoring plan to convert project into an ARC-compatible artifact layout without asserting that blocking checks have already passed.

#### Step 3.5.1 — Initialize ARC-Compatible Structure
```bash
# Create run_id if not provided
run_id = "{project_path.basename}-$(date +%Y%m%d-%H%M%S)"

# Create only the ARC directories needed to preserve migrated evidence and future pipeline outputs
mkdir -p artifacts/{run_id}/stage-00
mkdir -p artifacts/{run_id}/stage-01
mkdir -p artifacts/{run_id}/stage-04
mkdir -p artifacts/{run_id}/stage-10/experiment
mkdir -p artifacts/{run_id}/stage-16
mkdir -p artifacts/{run_id}/stage-17
mkdir -p artifacts/{run_id}/migration

# Create config.arc.yaml
cat > config.arc.yaml << 'EOF'
execution:
  backend: "unknown_until_probe"
  conda:
    auto_create: false
    python_version: "3.10"

paper:
  venue: "auto_resolve_after_analysis"
  language: "en"

run_id: "{run_id}"
original_project_path: "{project_path}"
refactored_from: "external"
EOF
```

#### Step 3.5.2 — Extract and Migrate Topic (Stage 01)
```bash
# Extract title from README.md or detect from code comments
if [ -f "{project_path}/README.md" ]; then
    title=$(head -5 {project_path}/README.md | grep -E "^#|Title:" | head -1)
fi

# Extract abstract if present
abstract=$(grep -A 10 "abstract" {project_path}/README.md 2>/dev/null | head -20 || echo "")

# Write to stage-01
cat > artifacts/{run_id}/stage-01/topic.json << EOF
{
  "run_id": "{run_id}",
  "topic": "{title}",
  "abstract": "{abstract}",
  "refactored_from": "external",
  "original_files_used": ["{project_path}/README.md"]
}
EOF
```

#### Step 3.5.3 — Migrate Literature (Stage 02-04)
```bash
# Extract dependencies to infer related work
if [ -f "{project_path}/requirements.txt" ]; then
    # Extract package names as potential related work indicators
    packages=$(cat {project_path}/requirements.txt | grep -v "^#" | cut -d'=' -d'<' -d'>' -d'!' -d'[' | head -20)
fi

# Check for existing citations/references
if [ -f "{project_path}/references.bib" ]; then
    cp {project_path}/references.bib artifacts/{run_id}/stage-04/references.bib
elif [ -f "{project_path}/paper.tex" ]; then
    # Extract bib entries from existing tex
    grep -E "@article|@inproceedings|@book" {project_path}/paper.tex > artifacts/{run_id}/stage-04/references.bib 2>/dev/null || true
fi

# Create literature summary
cat > artifacts/{run_id}/stage-04/literature_summary.json << EOF
{
  "run_id": "{run_id}",
  "papers_collected": $(wc -l < artifacts/{run_id}/stage-04/references.bib 2>/dev/null || echo 0),
  "source": "requirements.txt + code imports",
  "packages_identified": ["$(echo $packages | tr '\n' ',' | sed 's/"/\\"/g')"]
}
EOF
```

#### Step 3.5.4 — Migrate Code and Experiments (Stage 10-13)
```bash
# Copy experiment code
if [ -d "{project_path}/src" ]; then
    cp -r {project_path}/src/* artifacts/{run_id}/stage-10/experiment/ 2>/dev/null || true
elif [ -d "{project_path}/experiments" ]; then
    cp -r {project_path}/experiments/* artifacts/{run_id}/stage-10/experiment/ 2>/dev/null || true
elif [ -f "{project_path}/main.py" ]; then
    mkdir -p artifacts/{run_id}/stage-10/experiment
    cp {project_path}/main.py artifacts/{run_id}/stage-10/experiment/
fi

# Copy data if exists
if [ -d "{project_path}/data" ]; then
    cp -r {project_path}/data artifacts/{run_id}/stage-10/experiment/data 2>/dev/null || true
elif [ -d "{project_path}/dataset" ]; then
    cp -r {project_path}/dataset artifacts/{run_id}/stage-10/experiment/data 2>/dev/null || true
fi

# Copy requirements
if [ -f "{project_path}/requirements.txt" ]; then
    cp {project_path}/requirements.txt artifacts/{run_id}/stage-10/experiment/requirements.txt
fi

# Create migration metadata only; experiment design still requires the normal pipeline stages
cat > artifacts/{run_id}/stage-10/experiment/migration_metadata.json << EOF
{
  "run_id": "{run_id}",
  "source": "refactored_from_external",
  "original_code_path": "{project_path}",
  "status": "migrated_unvalidated_inputs",
  "note": "Real experiment design must still be produced by the normal pipeline stages"
}
EOF
```

#### Step 3.5.5 — Migrate Paper Draft (Stage 16-17)
```bash
# Find and copy existing paper
if [ -f "{project_path}/paper.tex" ]; then
    cp {project_path}/paper.tex artifacts/{run_id}/stage-17/main.tex
elif [ -f "{project_path}/paper.md" ]; then
    # Preserve the markdown draft as migrated evidence; do not fabricate a fake TeX paper
    cp {project_path}/paper.md artifacts/{run_id}/stage-17/paper_migrated.md
fi

if [ -f "{project_path}/references.bib" ]; then
    cp {project_path}/references.bib artifacts/{run_id}/stage-17/references.bib
fi

# Create outline from existing structure
cat > artifacts/{run_id}/stage-16/outline.json << EOF
{
  "run_id": "{run_id}",
  "outline_source": "migrated_from_external",
  "sections_detected": ["introduction", "method", "experiments", "results", "conclusion"]
}
EOF
```

#### Step 3.5.6 — Require Real Environment Probe (Stage 00)
```bash
# External-project refactoring MUST NOT fabricate environment capability.
# Record that probe execution is still required.
cat > artifacts/{run_id}/stage-00/environment_recheck_required.json << EOF
{
  "run_id": "{run_id}",
  "probe_required": true,
  "reason": "external_project_refactor",
  "blocking_skills": ["arc-00-04-environment-probe"],
  "status": "pending_real_probe",
  "recorded_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF
```

#### Step 3.5.7 — Record Refactoring Complete
```bash
# Update run_id in references for later stages
echo "Refactoring complete. Created run_id: {run_id}"

# Write refactoring summary
cat > artifacts/{run_id}/refactoring_summary.json << EOF
{
  "run_id": "{run_id}",
  "original_path": "{project_path}",
  "refactored_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "migrated_items": {
    "topic": "{project_path}/README.md",
    "literature_inventory": "requirements.txt and existing bibliography copied as unverified inputs",
    "code": "{project_path}/src or {project_path}/main.py",
    "paper": "{project_path}/paper.tex or paper.md"
  },
  "created_structure": "artifacts/{run_id}/stage-{00,01,04,10,16,17}/",
  "ready_for_iteration": true,
  "ready_for_iteration_note": "ready to continue diagnosis and delegated fixes, not ready to claim completed validation",
  "blocking_followups": [
    "arc-00-04-environment-probe",
    "arc-02-02-literature-collect or arc-08-04-citation-verify",
    "arc-08-03-export-publish",
    "arc-10-01-claim-evidence-trace-gate",
    "arc-10-02-figure-quality-gate",
    "arc-10-03-submission-format-gate"
  ]
}
EOF
```

**After refactoring, the external project has an ARC-compatible artifact skeleton with run_id. It is NOT considered stage-complete: environment probing, literature verification, execution validation, export, and the required late-stage gates must still run for real before the project can be treated as ARC-complete. Continue to Step 4 with the migrated artifacts/ structure.**

### Step 4 — Check Quality Indicators

Verify existence of key outputs:

| Indicator | Location | Significance |
|-----------|----------|-------------|
| `has_latex` | `*.tex` files exist | Paper can be compiled |
| `has_pdf` | `*.pdf` files exist | Paper output available |
| `has_reproducibility_bundle` | `reproducibility_report.json` exists | Research is reproducible |
| `has_verified_citations` | `references.bib` plus stage verification artifacts | Citations are authentic only after real verification |
| `has_experiment_code` | `experiment/` or `src/` | Code for experiments exists |
| `has_experiment_results` | `experiment_summary.json` or `runs/` | Real results exist |

### Step 5 — Generate Diagnostic Report

Before finalizing the diagnostic report, optionally consult the portable shared experience store at `meta_store_path` if it exists. Reuse only validated lessons such as recurring reviewer pain points, repeated failure patterns, or remediations that already worked in prior runs/projects; never treat meta guidance as evidence that a current blocking check has already passed.

**Identify critical_gaps (blocking issues):**
- Quality gate not passed
- Missing or unverified environment probe for runnable/exportable pipeline
- Missing LaTeX/PDF output
- Unverified or insufficient citations
- Missing required late-stage gates
- Figures lack provenance, authenticity, or required quality metadata

**Generate improvement_plan sorted by priority:**
1. Priority 1: Fix blocking issues
2. Priority 2: Improve quality scores
3. Priority 3: Polish and finalize

### Step 6 — Initialize Iteration State

Create or read existing `iteration_state.json`:
- Set `target_quality`, `max_iterations`
- Initialize `quality_history` with current score
- Set `iteration_count: 0`

### Step 7 — Execute Fixes via Main Pipeline Skills

**All fixes MUST be delegated to the appropriate main pipeline skills.** This ensures the improvement loop uses the exact same logic as the main pipeline.

**Hard prerequisite before any export or late-stage delegation:** `arc-00-04-environment-probe` must have produced a real blocking/pass result for the active run. A migrated placeholder such as `stage-00/environment_recheck_required.json` is only evidence that probing is still required; it is never sufficient to invoke stage 22 or any of stages 23–28.

#### For Critical Gaps (Priority 1):

**GAP-001: Quality gate not passed**
- Delegate to: `/invoke arc-07-04-paper-revision` (if review exists) or `/invoke arc-08-01-quality-gate`
- If `arc-08-01` rejects: Read `quality_report.json` → delegate specific fixes to appropriate skills
- Loop: `arc-08-01-quality-gate` → reject → fix → `arc-08-01-quality-gate` until approved

**GAP-002: Missing PDF/LaTeX**
- Prerequisite: a real Stage-0 environment probe result confirming execution/LaTeX/network capability for this run
- Delegate to: `/invoke arc-08-03-export-publish`
- On compile failure: `/invoke arc-09-02-paper-polish` to fix LaTeX issues

**GAP-003: Unverified or insufficient citations**
- Delegate to: `/invoke arc-08-04-citation-verify`
- On unverifiable DOI: Fix or remove citation → re-verify
- On citation-count or recency failure: expand verified bibliography upstream, then re-verify

#### For Quality Improvements (Priority 2):

All Priority 2 improvements that modify paper content MUST be delegated to:
- `/invoke arc-07-04-paper-revision` — for revision-based fixes

All Priority 2 improvements that require running experiments MUST be delegated to:
- `/invoke arc-05-02-iterative-refine` — for experiment-based improvements

#### Fix Execution Protocol:

1. **Read diagnostic report** for specific issues
2. **Delegate to appropriate skill** (see table below)
3. **Await skill completion**
4. **Record the change** in `iteration_state.json`
5. **Validate output** exists and is non-empty

| Issue Type | Delegate To | Loop Until |
|------------|-------------|------------|
| Quality gate rejection | `arc-08-01-quality-gate` | Approved |
| Paper content fix | `arc-07-04-paper-revision` | Done |
| LaTeX/PDF export | `arc-08-03-export-publish` | PDF exists |
| Citation verification | `arc-08-04-citation-verify` | All verified and citation thresholds pass |
| Experiment improvement | `arc-05-02-iterative-refine` | Metrics improved |
| Paper polish | `arc-09-02-paper-polish` | Done |
| Review loop | `arc-09-01-paper-review-loop` | Done |
| Claim evidence gate | `arc-10-01-claim-evidence-trace-gate` | Passed |
| Figure quality gate | `arc-10-02-figure-quality-gate` | Passed |
| Submission format gate | `arc-10-03-submission-format-gate` | Passed |

**Key principle: arc-00-05 NEVER modifies paper content directly. All fixes go through the main pipeline skills.**

### Step 8 — Record Fix Results

After each delegated skill completes:
1. Verify the skill produced its expected output
2. Record in `iteration_state.json`:
   ```json
   {
     "iteration": N,
     "delegated_to": "arc-08-01-quality-gate",
     "result": "approved|rejected",
     "issues_fixed": ["abstract word count", "missing limitations"],
     "remaining_issues": []
   }
   ```
3. If skill rejected/failed, continue to next fix (do not block)

### Step 9 — Anti-Hallucination Verification

After each change that affects data/citations/claims:

```bash
# Verify no fabricated metrics
python3 -c "
import json
with open('experiment_summary.json') as f:
    data = json.load(f)
    # Check metrics are realistic
    for k, v in data.get('all_metrics', {}).items():
        assert v['mean'] >= 0 and v['mean'] <= 1
        assert v['std'] >= 0
        assert v['n_seeds'] >= 3
"

# Verify citations resolve
for doi in $(grep -o '10\.[0-9]*/[0-9]*' references.bib | sort -u); do
    status=$(curl -s -o /dev/null -w "%{http_code}" "https://doi.org/$doi")
    assert [ "$status" = "200" ] || [ "$status" = "301" ]
done
```

**If hallucination detected:**
- Fail immediately with `E_RECHECK_05`
- Write `HALLUCINATION_REPORT.md` with evidence
- Do NOT proceed with iteration

### Step 10 — Compile and Validate LaTeX

```bash
cd {project_path}/artifacts/<run_id>/stage-25/
pdflatex -interaction=nonstopmode main.tex > compile.log 2>&1
bibtex main > bibtex.log 2>&1
pdflatex -interaction=nonstopmode main.tex > compile.log 2>&1
pdflatex -interaction=nonstopmode main.tex > compile.log 2>&1

# Check for errors
if grep -q "Error" compile.log; then
    fail with `E_RECHECK_06`
fi

# Check PDF exists and is non-empty
if [ ! -s main.pdf ]; then
    fail with `E_RECHECK_06`
fi
```

### Step 11 — Reassess Quality and Check Exit Conditions

After all fixes applied:
1. Run relevant quality checks
2. Calculate new quality score
3. Compare to previous score
4. Record `quality_delta` in iteration log
5. Update `quality_history`
6. Increment `iteration_count`

A Stage-20 quality score alone is not enough to unlock export or claim completion. Before proceeding beyond this step, confirm that any required Stage-0 probe has run for real and that Stage 20 is approved rather than merely tolerated via noncritical iteration behavior.

**Exit conditions:**
- If `quality_score >= target_quality`: **Proceed to Step 12** (Pass Remaining Gates). This does NOT yet mean success.
- If `iteration_count >= max_iterations` AND (`quality_score < target_quality` OR required late-stage gates are still missing):
  - Write `improvement_final_report.json` with `target_reached: false` and `invariant_satisfied: false`
  - Fail with `E_RECHECK_07` — max iterations reached before final acceptance conditions were met
- Otherwise: **Loop to Step 7** (continue iterating)

### Step 12 — Pass Remaining Gates

If quality target reached, complete remaining stages in canonical order:

```bash
# A real environment probe is mandatory before export or late-stage execution.
[ -f artifacts/<run_id>/stage-00/environment.json ] || /invoke arc-00-04-environment-probe

# Stage 7.5: Novelty gap gate (must pass before hypothesis generation)
if [ ! -f artifacts/<run_id>/stage-07b/novelty_gap_report.json ]; then
    /invoke arc-03-03-novelty-gap-gate
fi

# Stage 9.5: Reproducibility design gate (must pass before coding)
if [ ! -f artifacts/<run_id>/stage-09b/reproducibility_design_report.json ]; then
    /invoke arc-04-04-reproducibility-design-gate
fi

# Stage 15.5: Result claim gate (must pass before writing)
if [ ! -f artifacts/<run_id>/stage-15b/claim_scope_report.json ]; then
    /invoke arc-06-03-result-claim-gate
fi

# Stage 17.5: Writing compliance gate (must pass before peer review)
if [ ! -f artifacts/<run_id>/stage-17b/writing_compliance_report.json ]; then
    /invoke arc-07-05-writing-compliance-gate
fi

# Stage 18.5: Bibliography quality gate (must pass before review findings propagate)
if [ ! -f artifacts/<run_id>/stage-18b/bibliography_quality_report.json ]; then
    /invoke arc-08-05-bibliography-quality-gate
fi

# Stage 20 must be approved before any stage-22+ handoff can be treated as valid.
/invoke arc-08-01-quality-gate

# Stage 21.5: Reproducibility bundle gate (must pass before export)
if [ ! -f artifacts/<run_id>/stage-21b/reproducibility_bundle_report.json ]; then
    /invoke arc-08-06-reproducibility-bundle-gate
fi

# Export must exist before citation verify or later gates can run.
[ -f artifacts/<run_id>/stage-22/main.pdf ] || /invoke arc-08-03-export-publish

# Stage 23 must exist before late-stage completion can be claimed.
if [ ! -f artifacts/<run_id>/stage-23/verification_report.json ]; then
    /invoke arc-08-04-citation-verify
fi

# Stage 24: Paper review loop (if not already run)
if [ ! -f artifacts/<run_id>/stage-24/AUTO_REVIEW.md ]; then
    /invoke arc-09-01-paper-review-loop
fi

# Stage 24.5: Academic integrity gate (must pass before camera-ready)
if [ ! -f artifacts/<run_id>/stage-24b/academic_integrity_report.json ]; then
    /invoke arc-09-03-academic-integrity-gate
fi

# Stage 25: Paper polish
if [ ! -f artifacts/<run_id>/stage-25/paper_polished.md ]; then
    /invoke arc-09-02-paper-polish
fi

# Stage 26: Numeric truth gate then claim-evidence gate
/invoke arc-10-04-numeric-truth-gate
/invoke arc-10-01-claim-evidence-trace-gate

# Stage 27: Figure quality gate
/invoke arc-10-02-figure-quality-gate

# Stage 28: Submission format gate
/invoke arc-10-03-submission-format-gate

# Stage 28.5: Final acceptance gate
/invoke arc-10-05-final-acceptance-gate
```

Record each completed late-stage gate in `gates_passed` array. Do not mark final acceptance until a real Stage-0 probe result exists, Stage 20 is approved, and all critical gates (novelty, reproducibility design, result claim, writing compliance, bibliography quality, reproducibility bundle, academic integrity, numeric truth, claim-evidence trace, figure quality, submission format, final acceptance) have passed.

### Step 13 — Generate Final Reports

**Write `improvement_final_report.json`:**
- Total iterations
- Final quality score
- Whether target was reached
- Whether the core invariant was satisfied
- Which late-stage gates passed
- Final output locations
- Any remaining issues

**Update `diagnostic_summary.md`:**
- Add improvement progress section
- Show quality_history trend
- List final outputs

### Step 14 — Final Validation

**MUST verify CORE INVARIANT is satisfied:**

1. **LaTeX + PDF exist and are valid:**
   ```bash
   ls -la artifacts/<run_id>/stage-25/main.tex
   ls -la artifacts/<run_id>/stage-25/main.pdf
   file artifacts/<run_id>/stage-25/main.pdf  # verify it's a real PDF
   ```

2. **Environment and citations are verifiable:**
   ```bash
   # A real probe result must exist; placeholder migration markers are not enough.
   [ -f artifacts/<run_id>/stage-00/environment.json ] || fail with `E_RECHECK_INVARIANT_BROKEN`

   # Each DOI in the polished bibliography must resolve, and stage-23 verification must exist.
   [ -f artifacts/<run_id>/stage-23/verification_report.json ] || fail with `E_RECHECK_INVARIANT_BROKEN`
   for doi in $(grep -o '10\.[0-9]*/[0-9]*' artifacts/<run_id>/stage-25/references.bib | sort -u); do
     status=$(curl -s -o /dev/null -w "%{http_code}" "https://doi.org/$doi")
     [ "$status" = "200" ] || [ "$status" = "301" ] || fail with `E_RECHECK_INVARIANT_BROKEN`
   done
   ```

3. **Experiment data is real (not fabricated):**
   ```bash
   # Verify experiment_summary.json records traceable metrics with non-negative dispersion.
   # Do not assume every legitimate metric is normalized to [0,1].
   python3 -c "
   import json
   with open('artifacts/<run_id>/stage-14/experiment_summary.json') as f:
       data = json.load(f)
       for k, v in data.get('all_metrics', {}).items():
           assert 'mean' in v, 'Metric missing mean'
           assert v['std'] >= 0, 'Std must be non-negative'
           assert v['n_seeds'] >= 1, 'Need at least one recorded seed/run'
   "
   ```

4. **Paper compiles cleanly:**
   ```bash
   cd artifacts/<run_id>/stage-25/
   pdflatex -interaction=nonstopmode main.tex | grep -q "Error" && fail
   [ -s main.pdf ] || fail
   ```

5. **Citation hard requirements:**
   ```bash
   # Count total citations
   total=$(grep -c "^@" artifacts/<run_id>/stage-25/references.bib 2>/dev/null || echo 0)
   [ "$total" -ge 30 ] || fail with `E_RECHECK_CITATION_COUNT`

   # Check recent citations (2021-2026)
   recent=$(grep -E "year\s*=\s*\{(202[1-6])" artifacts/<run_id>/stage-25/references.bib 2>/dev/null | wc -l)
   recent_pct=$((recent * 100 / total))
   [ "$recent_pct" -ge 20 ] || fail with `E_RECHECK_CITATION_RECENCY`

   # Require late-stage verification artifact rather than trusting migrated bibliography alone
   [ -f artifacts/<run_id>/stage-23/verification_report.json ] || fail with `E_RECHECK_INVARIANT_BROKEN`
   ```

**If ANY check fails:**
- Fail immediately with `E_RECHECK_INVARIANT_BROKEN`
- Write `INVARIANT_BREAK_REPORT.md` explaining which requirement was violated
- Do NOT report success if the paper cannot be compiled or verified

**If ALL checks pass:**
- Report `invariant_satisfied: true` in final report
- Output the final paper location

---

Verify all required output files exist and are valid:
- `project_diagnostic_report.json` exists and is valid
- `iteration_state.json` exists and is valid
- `diagnostic_summary.md` exists
- For external projects: `external_project_diagnostic.json` also exists
- If mode is "full": `improvement_final_report.json` exists AND `invariant_satisfied: true`

---

## Integration with Main Pipeline

This skill can run:
1. **Standalone**: On any project directory without pipeline context
2. **Pre-pipeline**: Before running `arc-00-01-research-pipeline` to assess a new project
3. **Post-pipeline**: After pipeline completion to identify improvement opportunities
4. **Between stages**: To diagnose issues and propose fixes

**This skill is always in "full" mode** — it diagnoses, iterates improvements, and continues until quality target is reached and all required late-stage gates pass, or until max iterations is exhausted.

**Does NOT modify pipeline state** — it generates reports, migrated artifacts, and delegated-fix outputs, but it does not mark core orchestrator state files as completed on behalf of skipped checks.

---

## Anti-Hallucination Guarantees

**These guarantees exist to enforce the CORE INVARIANT.**

This skill verifies:
- **Authenticity**: File existence and content are real, not fabricated
- **Citation Verifiability**: Every DOI resolves via doi.org (no fake references)
- **Citation Sufficiency**: Reference count and recent-reference ratio satisfy the hard paper standards unless explicitly justified
- **Experiment Reality**: All experiment results come from actual code execution, not simulation
- **Figure Authenticity**: Figures are evidence-bearing and traceable to real experiment outputs
- **Compilation**: LaTeX produces a real PDF without errors
- **Claim Traceability**: Every claim in the paper traces to evidence in artifacts/

**On ANY violation:**
1. Immediately abort with `E_RECHECK_INVARIANT_BROKEN` (if core invariant broken) or `E_RECHECK_05` (if hallucination detected)
2. Write detailed `HALLUCINATION_REPORT.md` or `INVARIANT_BREAK_REPORT.md`
3. Pipeline CANNOT proceed with fake, unverifiable, or dishonestly logged outputs

**The CORE INVARIANT is absolute:** If the project cannot produce an authentic, verifiable, reproducible academic paper with LaTeX + PDF, the skill MUST fail rather than produce degraded output.

---

## Example Usage

```bash
# Diagnose and iterate until quality gates pass
/invoke arc-00-05-project-recheck
{
  "project_path": "/path/to/project",
  "run_id": "rc-20260405-143022",
  "max_iterations": 5,
  "target_quality": 0.80,
  "auto_approve": true
}
```

---

## Key Design Principles

1. **Delegation not implementation**: arc-00-05 diagnoses and decides WHICH skill to call, but never implements fixes directly. All improvements go through main pipeline skills.
2. **Consistency with main pipeline**: The improvement loop uses the EXACT same skills as the main pipeline (arc-07-04, arc-08-01, arc-08-03, etc.)
3. **CORE INVARIANT is absolute**: If the project cannot produce an authentic paper with LaTeX + PDF, fail rather than produce degraded output
4. **Transparent logging**: Every delegated skill call recorded with before/after evidence
5. **Fail fast on violation**: Hallucination or invariant break = immediate abort
6. **Guaranteed output**: If target reached, final output is compilable LaTeX + PDF
