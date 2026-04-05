---
name: arc-02-02-literature-collect
description: Stage 4 — Retrieve candidate papers from real databases with mandatory DOI/arXiv verification. BLOCKING on unverifiable references to prevent hallucinated literature.
metadata:
  category: pipeline-stage
  trigger-keywords: "literature collect,candidates,stage 4"
  applicable-stages: "4"
  priority: "1"
  version: "4.0"
  author: researchclaw
---

## Purpose
Execute the search plan to retrieve candidate papers from **real, verifiable databases only**. Every paper MUST have a resolvable DOI or arXiv ID — no paper is accepted without proof of existence. This stage **BLOCKS** on unverifiable references to prevent hallucinated literature from entering the pipeline.

**CRITICAL**: This stage prevents the most common form of academic hallucination — fabricated citations. Papers without verified identifiers are rejected outright.

---

## Quality Contract

`candidates.jsonl` MUST satisfy:
- Contains ≥50 candidate papers total
- Each line is valid JSON with `title` and a **verified** `doi` OR `arxiv_id`
- **Every DOI must resolve via https://doi.org/{doi} (HTTP 200/303)**
- **Every arXiv ID must exist via https://arxiv.org/abs/{id} (HTTP 200)**
- Papers are deduplicated by `doi` first, then `arxiv_id`
- ≥2 data sources are represented in the results
- `verification_report.json` must show `verified_count == total_count`

---

## Inputs
`artifacts/<run_id>/stage-03/search_plan.yaml`
`artifacts/<run_id>/stage-03/sources.json`
`artifacts/<run_id>/stage-03/queries.json`

---

## Outputs

### `candidates.jsonl`
One JSON object per line, **all verified**:
```json
{
  "id": "openalex:W123456789",
  "title": "...",
  "abstract": "...",
  "year": 2023,
  "authors": "...",
  "venue": "...",
  "doi": "10.xxxx/xxxxx",
  "doi_verified": true,
  "arxiv_id": null,
  "arxiv_verified": false,
  "verification_timestamp": "2026-04-05T12:00:00Z"
}
```

Every entry MUST have either:
- `doi_verified: true` (DOI resolves successfully)
- `arxiv_verified: true` (arXiv page exists)

### `verification_report.json`
```json
{
  "total_candidates": 52,
  "doi_verified": 48,
  "arxiv_verified": 4,
  "verification_failed": 0,
  "rejected_hallucinations": 0,
  "verification_pass": true,
  "sources_used": ["openalex", "arxiv", "semantic_scholar"],
  "verification_timestamp": "2026-04-05T12:00:00Z"
}
```

### `hallucination_rejects.jsonl` (only if any rejected)
```jsonl
{"title":"...","reason":"DOI 10.xxxx/xxxxx did not resolve (HTTP 404)","source":"semantic_scholar","rejected_at":"..."}
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| File exists and non-empty | `candidates.jsonl` present |
| ≥50 verified candidates | Line count ≥ 50, all verified |
| Each line valid JSON | `json.loads()` succeeds per line |
| **All DOIs resolve** | Every `doi` field has `doi_verified: true` OR entry uses arXiv |
| **All arXiv IDs exist** | Every `arxiv_id` field has `arxiv_verified: true` OR entry uses DOI |
| ≥2 sources | At least 2 distinct sources in `verification_report.json` |
| Verification report pass | `verification_pass: true` AND `verification_failed: 0` |

**BLOCKING Failure**: If any candidate lacks verified DOI/arXiv → `E04_HALLUCINATION_DETECTED`
**BLOCKING Failure**: If < 50 verified candidates → `E04_INSUFFICIENT_VERIFIED`

---

## Error Codes

| Code | Trigger | Retryable |
|------|---------|-----------|
| `E04_INSUFFICIENT_VERIFIED` | < 50 verified candidates after verification | Yes (2 retries) |
| `E04_HALLUCINATION_DETECTED` | ≥1 candidate has unverifiable DOI/arXiv | **No** (blocking) |
| `E04_NETWORK_FAILURE` | Cannot reach doi.org or arxiv.org for verification | Yes (2 retries) |
| `E04_NO_DOI_OR_ARXIV` | Candidate lacks both DOI and arXiv ID | **No** (rejected per-entry) |

---

## Retry Policy
Max retries: **2** — retry only on network failures (API/DOI/arXiv unreachable).
- **DO NOT retry** on hallucination detection (E04_HALLUCINATION_DETECTED) — this is a data quality violation
- On retry: re-fetch from sources, re-verify all candidates

---

## State Transition
`pending` → `running` → `verifying` → `done` | `blocked` | `failed`

---

## Procedure

### Step 1 — Read Search Plan
Read `search_plan.yaml`, `sources.json`, and `queries.json` from stage-03.

### Step 2 — Execute Searches
For each (source, query) pair in the plan:

**OpenAlex (preferred — includes DOI):**
```bash
curl -s "https://api.openalex.org/works?search=${encoded_query}&per-page=50&mailto=anonymous@example.com"
```
Parse results: `id`, `title`, `abstract`, `publication_year`, `authorships`, `doi` (extract from `ids.doi`).

**arXiv:**
```bash
curl -s "https://export.arxiv.org/api/query?search_query=all:${encoded_query}&start=0&max_results=50&sort=relevance"
```
Parse: `id` (arXiv ID like `2304.12345`), `title`, `summary`, `published`.

**Semantic Scholar:**
```bash
curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=${encoded_query}&limit=50&fields=title,abstract,year,authors,citationCount,externalIds"
```
Extract `DOI` from `externalIds` if present.

### Step 3 — Initial Deduplicate
Merge all results. Deduplicate by `doi` first, then `arxiv_id`.

### Step 4 — MANDATORY DOI Verification (BLOCKING)

For every candidate with a DOI:
```bash
# Verify DOI resolves
curl -s -o /dev/null -w "%{http_code}" -L "https://doi.org/${doi}"
```

| HTTP Code | Result |
|-----------|--------|
| 200, 303, 302 (redirects to real publisher) | `doi_verified: true` |
| 404, 410, 403 | `doi_verified: false` → REJECT as hallucination |
| Timeout/network error | Retry verification up to 2 times, then mark as network failure |

**Requirement**: Every DOI MUST resolve. No exceptions.

### Step 5 — MANDATORY arXiv Verification (BLOCKING)

For every candidate with an arXiv ID (no DOI):
```bash
# Verify arXiv page exists
curl -s -o /dev/null -w "%{http_code}" "https://arxiv.org/abs/${arxiv_id}"
```

| HTTP Code | Result |
|-----------|--------|
| 200 | `arxiv_verified: true` |
| 404, 403 | `arxiv_verified: false` → REJECT as hallucination |
| Timeout/network error | Retry verification up to 2 times |

**Requirement**: Every arXiv ID MUST have a real page. No exceptions.

### Step 6 — Reject Hallucinations

If any candidate fails verification:
1. Write to `hallucination_rejects.jsonl` with rejection reason
2. Remove from `candidates.jsonl`
3. Log the rejection in verification report

**If all candidates are rejected**: Fail with `E04_HALLUCINATION_DETECTED` (all sources produced unverifiable results — severe data quality issue)

### Step 7 — Write Verified candidates.jsonl

Write only verified candidates. Each entry must have:
- `doi_verified: true` OR `arxiv_verified: true`
- `verification_timestamp`

### Step 8 — Write verification_report.json

Record:
- `total_candidates`: count after verification
- `doi_verified`: count of DOI-verified papers
- `arxiv_verified`: count of arXiv-verified papers
- `verification_failed`: must be 0
- `rejected_hallucinations`: count of rejected unverifiable papers
- `verification_pass: true` only if `verification_failed == 0`

### Step 9 — Validate

Check:
- ≥50 verified candidates (not just raw fetched)
- `verification_failed == 0`
- All entries have verified DOI or verified arXiv
- ≥2 sources in verification report

**If verification_failed > 0**: Fail with `E04_HALLUCINATION_DETECTED` (blocking, no retry)
**If total_candidates < 50**: Fail with `E04_INSUFFICIENT_VERIFIED` (retry allowed)

---

## Anti-Hallucination Guarantee

This stage implements the following guarantees:

1. **No fabricated papers**: Every paper MUST come from a real API (OpenAlex, arXiv, Semantic Scholar)
2. **No fake DOIs**: Every DOI MUST resolve to a real publisher page
3. **No fake arXiv IDs**: Every arXiv ID MUST have a real abstract page
4. **Audit trail**: All rejections logged in `hallucination_rejects.jsonl`
5. **Blocking on unverifiable**: Pipeline stops if any hallucination is detected in the candidate set

This prevents the most common AI hallucination pattern in academic writing: citing papers that don't exist.
