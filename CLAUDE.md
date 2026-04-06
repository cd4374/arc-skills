# ARC-Skills: Project Alignment Document

## Project Objective (Core Alignment Point)

When the full chain succeeds, it MUST yield a **high-quality, verifiable, reproducible, authentic academic paper** with at minimum:

1. **Real LaTeX sources** — locally compiled, not generated/placeholder
2. **Locally compiled PDF** — via `pdflatex` + `bibtex`, exit code 0
3. **Verified references** — every citation has resolvable DOI or arXiv ID
4. **Real experiment-backed claims** — all quantitative claims traceable to actual execution
5. **Submission-format compliance** — meets target venue specifications

**This is the non-negotiable target. All skills, gates, and workflows must align to this.**

---

## Hard Paper Standards (from paper_standards.md)

### Must Satisfy (Blocking if Failed)

| Standard | Minimum | Gate/Stage |
|----------|---------|------------|
| **LaTeX Compilation** | Successful `pdflatex` + `bibtex`, exit code 0 | Stage 22 |
| **PDF Generation** | Non-empty PDF file exists | Stage 22 |
| **Citation Verification** | All citations resolvable (DOI/arXiv) | Stage 4, 23 |
| **Reference Count** | ≥30 verified references | Stage 23 |
| **Reference Recency** | ≥20% from last 5 years | Stage 23 |
| **Figure Count** | ≥3-5 figures (venue-dependent) | Stage 27 |
| **Figure Authenticity** | Traceable to experiment outputs | Stage 27 |
| **Numeric Truth** | Claims match experiment data | Stage 26-pre |
| **Reproducibility Bundle** | Code + environment + seeds documented | Stage 21.5 |

### Quality Targets (Non-blocking but Logged)

- **Bibliography style**: Consistent (IEEEtran / NeurIPS / ACL / etc.)
- **Figure quality**: Vector preferred, ≥300 DPI raster, colorblind-friendly
- **Statistical rigor**: Error bars, significance tests, multiple seeds
- **Ablation studies**: Required for AI/ML methods
- **Limitations section**: Required, honest

---

## Critical Blocking Points

### Stage 0: Environment Probe (E00_*)

**BLOCKS ENTIRE PIPELINE** if any critical capability missing:

1. **Execution Backend** (`execution_capable: true`)
   - Required: Stages 10-13 (experiment design, code gen, resource planning, execution)
   - Options: Local GPU, Local CPU, SSH Remote

2. **Local LaTeX** (`latex_capable: true`)
   - Required: Stages 22, 28 (PDF compilation)
   - Must: `pdflatex` and `bibtex` installed locally
   - **NO FALLBACK**: No Overleaf, no cloud compilation

3. **Network Connectivity** (`network_capable: true`)
   - Required: Stages 3-4 (literature search, DOI verification)
   - Must: Reachable `doi.org` and `arxiv.org`

**Action on Failure**: Abort immediately, write `environment_error.md` with fix instructions.

### Stage 4: Literature Collect (E04_HALLUCINATION_DETECTED)

**BLOCKS** if any candidate lacks verified DOI or arXiv ID.

- Every paper MUST resolve via `curl https://doi.org/{doi}`
- Every arXiv ID MUST exist at `https://arxiv.org/abs/{id}`
- **NO EXCEPTIONS**: Unverifiable citations are rejected, not warned

### Late-Stage Gates (Stages 26-28.5)

All **BLOCKING** — no bypass:

- **Stage 26-pre** (arc-10-04): Numeric truth validation
- **Stage 26** (arc-10-01): Claim-evidence traceability
- **Stage 27** (arc-10-02): Figure quality and authenticity
- **Stage 28** (arc-10-03): Submission format compliance
- **Stage 28.5** (arc-10-05): Final acceptance aggregate

---

## Anti-Hallucination Guarantees

### 1. Citation Authenticity

- Stage 4: Fetch only from real APIs (OpenAlex, arXiv, Semantic Scholar)
- Stage 4: Verify every DOI/arXiv before accepting
- Stage 23: Re-verify all citations before final bibliography
- **Violation**: `E04_HALLUCINATION_DETECTED` or `E23` → BLOCK

### 2. Experiment Reality

- Stage 12: Execute real code, no simulation
- Stage 12: Anti-fabrication checks (timestamps, metric variation, logs)
- Stage 12: `execution_verified: true` required
- **Violation**: `E12_FABRICATION` → BLOCK, non-retryable

### 3. PDF Authenticity

- Stage 22: Local compilation only
- Stage 22: `compile_report.json` with exit code 0
- Stage 22: No undefined references/citations
- **Violation**: `E22` → retry 2x, then BLOCK

### 4. Claim Verifiability

- Stage 14: All claims linked to `experiment_summary.json` metrics
- Stage 26-pre: Numeric claims validated against raw data
- Stage 26: Claim-evidence matrix built
- **Violation**: `E26_NUMERIC_MISMATCH` or `E26` → BLOCK

---

## Workflow Alignment

### Canonical Entrypoints

**Primary**: `/arc-00-01-research-pipeline`
- Mode `idea_start`: Research from scratch
- Mode `existing_project_improvement`: Iterate on existing work

**Recovery**: `/arc-00-03-resume`
- Resume from checkpoint
- Preserves all gate decisions

**Status**: `/arc-00-02-status`
- Read-only inspection
- No state changes

### Decision Loop Cap

- **Max 2 pivot/refine cycles**
- Then: `proceed` with `quality_warning.txt`
- Prevents infinite iteration

### Gate Behavior

| Gate | On Reject | Critical? |
|------|-----------|-----------|
| Stage 5 (Literature Screen) | Rollback to Stage 4 | No |
| Stage 7.5 (Novelty Gap) | Rollback to Stage 7 | **Yes** |
| Stage 9 (Experiment Design) | Rollback to Stage 8 | No |
| Stage 9.5 (Reproducibility) | Rollback to Stage 9 | **Yes** |
| Stage 15.5 (Result Claims) | Rollback to Stage 15 | **Yes** |
| Stage 17.5 (Writing Compliance) | Rollback to Stage 17 | **Yes** |
| Stage 18.5 (Bibliography Quality) | Rollback to Stage 17 | **Yes** |
| Stage 20 (Quality Gate) | Rollback to Stage 16 | No |
| Stage 21.5 (Reproducibility Bundle) | Rollback to Stage 20 | **Yes** |
| Stage 24.5 (Academic Integrity) | Rollback to Stage 24 | **Yes** |
| Stage 26 (Claim-Evidence) | Rollback to Stage 19 | **Yes** |
| Stage 27 (Figure Quality) | Rollback to Stage 22 | **Yes** |
| Stage 28 (Submission Format) | Rollback to Stage 22 | **Yes** |
| Stage 28.5 (Final Acceptance) | Rollback to Stage 28 | **Yes** |

---

## Template Alignment

### Supported Venues (Stage 15.7)

**Conferences** (8-9 pages main + references):
- `neurips_2026`, `neurips_2025`
- `icml_2026`, `icml_2025`
- `iclr_2026`, `iclr_2025`
- `aaai_2026`
- `acl_2025`, `emnlp_2025`, `naacl_2025`
- `cvpr_2026`, `iccv_2025`, `eccv_2026`
- `kdd_2025`

**Journals** (no hard page limit):
- `ieeetran_journal`
- `elsevier_cas_sc`, `pattern_recognition`
- `springer_lncs_journal_style`

### Template Asset Status

| Venue | Status | Source |
|-------|--------|--------|
| NeurIPS | Ready | Included `.sty` |
| ICML | Ready | Included `.sty` |
| ICLR | Ready | Self-contained |
| AAAI | Ready | Self-contained |
| IEEE | Ready | Standard class |
| Elsevier | Ready | Standard class |
| Springer | Ready | Standard class |
| ACL | Partial | Requires `acl.sty` download |
| CVPR | Partial | Requires `cvpr.sty` download |

**Stage 22 Behavior**:
- If `asset_status == "ready"`: Compile normally
- If `asset_status == "contract_only"` AND files missing: `E_TEMPLATE_ASSET_MISSING`

---

## File Structure Alignment

### Required Outputs (Stage 28)

```
artifacts/<run_id>/
├── stage-28/
│   ├── paper_final.pdf              # Compiled PDF (MUST exist)
│   ├── main.tex                     # LaTeX source
│   ├── references.bib               # Verified bibliography
│   ├── compile_report.json          # Exit code 0, no undefined refs
│   ├── manifest.json                # Artifact checksums
│   ├── reproducibility_report.json  # Seeds, env, commands
│   └── deliverables/                # Complete submission package
│       ├── main.tex
│       ├── main.pdf
│       ├── references.bib
│       ├── code/                    # Experiment code
│       ├── charts/                  # Figures with provenance
│       └── ...
├── template/
│   └── template_manifest.json       # Resolved venue rules
└── pipeline_summary.json            # Final status
```

### State Files (Orchestrator Only)

- `pipeline_state.json`: Current stage
- `checkpoint.json`: Last completed stage
- `stage_history.jsonl`: Execution log
- `decision_history.json`: Pivot/refine record

---

## Error Code Alignment

### Blocking Errors (Non-retryable)

- `E00_CRITICAL_MISSING`: Environment insufficient
- `E00_NO_EXECUTION_ENV`: No backend
- `E00_NO_LATEX`: LaTeX not installed
- `E04_HALLUCINATION_DETECTED`: Fake citations
- `E07B_CODEX_MCP_REQUIRED`: Codex MCP unavailable (Stage 7.5)
- `E12_FABRICATION`: Simulated experiments
- `E12_NO_EXECUTION_ENV`: Backend lost mid-pipeline
- `E24_CODEX_MCP_REQUIRED`: Codex MCP unavailable (Stage 24)
- All `*B` gate rejections

### Retryable Errors

- Network failures (E00_NETWORK_UNREACHABLE, E04_NETWORK_FAILURE)
- Transient compile errors (E22)
- Timeout (E12_TIMEOUT)

### Rollback Triggers

- Gate rejections (E05, E07B, E09, E09B, E15B, E17B, E18B, E21B, E24B, E26, E27, E28, E28B)
- Decision loop exhaustion

---

## Experience Accumulation (Stage 20.5)

### Per-Run Learning

- Extract reusable patterns from successful runs
- Store in `meta_guidance.json`
- Do NOT rely on external databases

### Cross-Run Knowledge

- Manual aggregation of `meta_guidance.json` files
- Optional: Centralized registry (out of scope for pure-skills)

---

## External Integration

### Codex MCP (Stage 7.5 & Stage 24) — **MANDATORY**

**Required for**:
- **Stage 7.5** (arc-03-03): Novelty gap adversarial validation
- **Stage 24** (arc-09-01): External paper review

**Configuration**:
- Must have Codex CLI installed with MCP enabled
- Must have GPT-5.4 model access
- Reviewer must run in independent context (separate from executor)

**Behavior on Unavailable**:
- **Stage 7.5**: Fail with `E07B_CODEX_MCP_REQUIRED` — pipeline BLOCKED
- **Stage 24**: Fail with `E24_CODEX_MCP_REQUIRED` — pipeline BLOCKED
- **NO FALLBACK**: Degraded local review is **NOT PERMITTED**

**Why Mandatory**:
- Independence: Executor cannot judge its own work
- Adversarial guarantee: Genuine external scrutiny
- Bias prevention: Separate context prevents contamination
- Quality assurance: Self-review is insufficient for academic rigor

### Required APIs

- **OpenAlex**: Literature search (optional but recommended)
- **Semantic Scholar**: Citations (optional)
- **arXiv**: Preprint access (required for verification)
- **doi.org**: Citation verification (required)

---

## Success Criteria

A run is **SUCCESSFUL** if and only if:

1. ✅ Stage 0 passes (environment capable)
2. ✅ Stage 4 passes (≥50 verified candidates, no hallucinations)
3. ✅ Stage 7.5 passes (Codex MCP confirms novelty gaps)
4. ✅ Stage 12 completes (real experiments executed)
5. ✅ Stage 22 produces valid PDF (exit code 0)
6. ✅ Stage 23 verifies ≥30 citations
7. ✅ Stage 24 completes (Codex MCP external review)
8. ✅ Stage 26-pre validates numeric claims
9. ✅ Stage 26 approves claim-evidence traceability
10. ✅ Stage 27 approves figure quality
11. ✅ Stage 28 approves submission format
12. ✅ Stage 28.5 grants final acceptance

**Missing any = FAILURE**, regardless of partial outputs.

**Note**: Stages 7.5 and 24 require **Codex MCP**. If MCP is unavailable, these stages block with `E07B_CODEX_MCP_REQUIRED` or `E24_CODEX_MCP_REQUIRED`. No bypass available.

---

## Developer Notes

### Adding New Skills

1. Follow 11-section structure: Purpose, Quality Contract, Inputs, Outputs, Procedure, Validation, Error Codes, Retry Policy, Gate Behavior (if gate), Rollback Contract (if gate/decision), State Transition
2. Update `arc-00-01-research-pipeline` stage map
3. Update this CLAUDE.md if adding critical gates
4. Test against full pipeline

### Modifying Existing Skills

1. Maintain backward compatibility for state files
2. Update error codes if changing failure modes
3. Update README.md
4. Verify gate dependencies

### Testing

```bash
# Full pipeline test
/arc-00-01-research-pipeline
{
  "topic": "Test: linear regression on toy data",
  "target_venue": "neurips",
  "auto_approve_gates": true
}

# Expected: Completes to Stage 28 with valid PDF
```

---

## Version History

- v4.2: Current — Added late-stage gates (26-28.5), Codex MCP integration, template resolution
- v4.1: Added environment probe (Stage 0), blocking execution/LaTeX/network checks
- v4.0: Initial pure-skills architecture

---

**Remember**: The goal is not "a paper-shaped artifact." The goal is a **verifiable, reproducible, authentic academic paper**. Every decision should ask: "Does this serve the hard standards?"
