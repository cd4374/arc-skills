---
name: arc-08-04-citation-verify
description: Stage 23 (CRITICAL) — Verify citations in final paper. Reuses Stage 4 verification for known references, and verifies NEW citations added during writing. Hallucinated citations MUST block export.
metadata:
  category: pipeline-stage
  trigger-keywords: "citation verify,references,bib,stage 23"
  applicable-stages: "23"
  priority: "1"
  version: "5.0"
  author: researchclaw
---

## Purpose

Verify all citations in the final paper against Stage 4's verified literature corpus. **NEW citations** (added during paper writing) require fresh verification. Hallucinated citations MUST block the pipeline.

---

## Two-Tier Verification Strategy

| Citation Source | Verification Method | Cost |
|-----------------|---------------------|------|
| **Tier 1: From Stage 4** | Reuse `candidates.jsonl` verification | Zero (cached) |
| **Tier 2: NEW citations** | Multi-layer API verification | Network calls |

**Key Insight**: Stage 4 already verified ~50+ papers with DOI/arXiv checks. Stage 23 should NOT re-verify these — only verify NEW citations introduced during writing.

---

## Procedure

### Step 1 — Load Stage 4 Verified Corpus

Read `artifacts/<run_id>/stage-04/candidates.jsonl`:
- Build a lookup: `{doi: verified_entry}` and `{arxiv_id: verified_entry}`
- These are **pre-verified** — no need to re-check

### Step 2 — Extract Paper Citations

Extract all citation keys from `paper_final.md`:
- ML conferences: `\citep{...}`, `\citet{...}`
- IEEE venues: `\cite{...}`
- Match against `references.bib` to get DOI/arXiv/title for each

### Step 3 — Classify Citations (NEW vs KNOWN)

```python
for citation in paper_citations:
    if citation.doi in stage4_verified_dois:
        mark_as("KNOWN_VERIFIED", source="stage-04")
    elif citation.arxiv_id in stage4_verified_arxiv:
        mark_as("KNOWN_VERIFIED", source="stage-04")
    else:
        mark_as("NEW_CITATION", requires_verification=True)
```

Record counts:
- `known_verified`: from Stage 4
- `new_citations`: require fresh verification

### Step 4 — Verify NEW Citations Only

For each NEW citation, attempt verification in priority order:

**A. DOI → CrossRef**
```bash
curl -sLH "Accept: application/x-bibtex" "https://doi.org/{doi}"
```

**B. arXiv ID**
```bash
curl -s "https://arxiv.org/abs/{arxiv_id}"
```

**C. OpenAlex Title Search**
```bash
curl -s "https://api.openalex.org/works?filter=title.search:{TITLE}&per_page=5"
```

### Step 5 — Mark Results

| Status | Condition |
|--------|-----------|
| `KNOWN_VERIFIED` | Found in Stage 4 corpus (no re-verification needed) |
| `NEW_VERIFIED` | NEW citation verified via API |
| `UNVERIFIABLE` | NEW citation, no API could confirm |
| `HALLUCINATED` | NEW citation, appears fabricated |

### Step 6 — Generate Report

Write `verification_report.json`:
```json
{
  "total_citations": 47,
  "known_verified": 35,
  "new_verified": 8,
  "unverifiable": 2,
  "hallucinated": 2,
  "stage_blocked": true,
  "breakdown": {
    "from_stage4": 35,
    "new_citations": 12,
    "new_verified": 8,
    "new_unverifiable": 2,
    "new_hallucinated": 2
  },
  "hallucinated_citations": [
    {"citation_key": "fake2023method", "reason": "DOI 10.xxx/yyy does not resolve"}
  ]
}
```

### Step 7 — Blocking Decision

If `hallucinated > 0`:
1. Set `stage_blocked: true`
2. Fail with `E23` (hard block)

---

## Inputs
`artifacts/<run_id>/stage-04/candidates.jsonl` (Stage 4 verified corpus)
`artifacts/<run_id>/stage-22/paper_final.md`
`artifacts/<run_id>/stage-22/references.bib`

---

## Outputs

### `artifacts/<run_id>/stage-23/verification_report.json`
Full verification status with Stage 4 reuse breakdown.

### `artifacts/<run_id>/stage-23/references_verified.bib`
BibTeX with verification tags.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| All citations processed | Report covers every `\cite{}` in paper |
| Stage 4 corpus loaded | `known_verified` count matches lookup |
| NEW citations verified | Each NEW has verification attempt logged |
| No hallucinations | `hallucinated == 0` |

**Failure**: `hallucinated > 0` → `E23`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E23` | Stage-23 citation verification found hallucinated or unverifiable citations in the stage-22 package |
| `E23_STAGE4_MISSING` | Stage-4 verified corpus (`candidates.jsonl`) is missing |

---

## Retry Policy
Max retries: **1** — only on network failures for NEW citations.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Key Rules

1. **Reuse Stage 4**: Known citations skip re-verification
2. **Verify NEW only**: Citations added during writing need fresh verification
3. **Block on hallucination**: Fake citations must be removed before Stage 25
4. **Audit trail**: Report shows Stage 4 vs NEW breakdown