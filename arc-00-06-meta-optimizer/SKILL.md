---
name: arc-00-06-meta-optimizer
description: Support orchestrator for experience accumulation and reuse. Reads validated run artifacts, distills reusable guidance into explicit run-owned outputs, and can summarize prior real artifacts when they are explicitly provided.
metadata:
  category: orchestrator
  trigger-keywords: "meta optimize,experience accumulation,cross-run memory,portable experience,reuse"
  applicable-stages: "pre-run, post-stage-21"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Read completed-run artifacts and distill reusable experience into explicit outputs owned by real runs. This skill may also summarize prior real artifacts when they are explicitly provided, so the pipeline can reuse validated directions, common failure patterns, and reviewer pain points without relying on a built-in non-skill store.

This skill is **supporting, not authoritative**: it never waives blocking gates, never fabricates evidence, and never substitutes for per-run validation.

---

## Quality Contract

The experience outputs MUST remain portable and reusable:
- `experience_log.jsonl` MUST be append-only when an existing log is supplied
- `best_patterns.json` MUST summarize validated successful patterns rather than raw speculation
- `failure_patterns.json` MUST summarize recurring failure modes and effective remediations
- `reviewer_pain_points_digest.md` MUST contain concise digests of recurring reviewer criticisms grounded in real completed runs
- Read mode MUST return reusable guidance without mutating per-run artifacts
- Write mode MUST only distill from real run artifacts; fabricated lessons are forbidden

This skill is noncritical. If prior artifacts are missing or outputs cannot be written, it must fail honestly without blocking the core pipeline.

---

## Inputs
| Input | Description | Required |
|-------|-------------|----------|
| `mode` | `read` or `write` | Yes |
| `run_id` | Active or completed run ID | Required in write mode |
| `artifacts_path` | Root artifacts path for the run | Required in write mode |
| `prior_artifacts_paths` | Optional list of prior real artifact roots to summarize in read mode | No |
| `output_dir` | Output directory for distilled experience files; default `artifacts/<run_id>/meta/` in write mode and `artifacts/meta-read/` in read mode | No |
| `target_venue` | Venue/topic hint for read-mode filtering | No |
| `topic_or_project_class` | Topic/project-class hint for read-mode filtering | No |

---

## Outputs

### `meta/experience_log.jsonl`
Append-only run-level experience entries.

### `meta/best_patterns.json`
```json
{
  "patterns": [
    {
      "label": "review-driven figure provenance fixes",
      "evidence_count": 3,
      "applies_to": ["conference", "ml"],
      "notes": ["Stage 27 failures dropped after explicit provenance sidecars"]
    }
  ]
}
```

### `meta/failure_patterns.json`
```json
{
  "patterns": [
    {
      "failure": "figure authenticity drift",
      "seen_count": 2,
      "typical_stage": "arc-10-02-figure-quality-gate",
      "effective_remediation": "regenerate charts with provenance sidecars and re-run stages 23-25"
    }
  ]
}
```

### `meta/reviewer_pain_points/<run_id>_digest.md`
Concise digest of recurring reviewer complaints from real stage-24 review artifacts.

### `meta/meta_guidance.json` (read mode)
```json
{
  "relevant_patterns": [],
  "common_failures": [],
  "reviewer_pain_points": [],
  "notes": []
}
```

---

## Procedure

### Step 1 — Resolve Mode and Store Path
Resolve `mode` and the target `meta_store_path` (default `meta/`). Reject any mode other than `read` or `write`.

### Step 2 — Read Mode
If `mode=read`:
- Read `experience_log.jsonl`, `best_patterns.json`, and `failure_patterns.json` if present
- Filter entries by `target_venue` and/or `topic_or_project_class` when provided
- Summarize only reusable, validated guidance
- Write `meta_guidance.json`
- Do not modify any per-run artifacts

### Step 3 — Write Mode
If `mode=write`:
- Read completed run artifacts such as `stage_history.jsonl`, `decision_history.json`, stage-24 review outputs, stage-25 polished package summaries, and late-stage gate reports
- Distill: what worked, what failed repeatedly, which reviewer concerns recurred, and which remediations were validated
- Append a new record to `experience_log.jsonl`

### Step 4 — Update Best Patterns
Merge validated successful patterns into `best_patterns.json` using counts rather than replacing prior data.

### Step 5 — Update Failure Patterns
Merge recurring failure modes and validated remediations into `failure_patterns.json` using counts rather than replacing prior data.

### Step 6 — Write Reviewer Pain-Point Digest
If stage-24 review artifacts exist, summarize recurring pain points into `meta/reviewer_pain_points/<run_id>_digest.md`.

### Step 7 — Validate
Check that required meta outputs exist for the chosen mode and that append-only invariants are respected.

### Step 8 — Return Noncritical Status
If the store is missing, empty, or unwritable, fail honestly with a noncritical error and do not claim any reusable guidance was produced.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Valid mode | `mode` is `read` or `write` |
| Append-only log respected | Existing `experience_log.jsonl` entries are preserved |
| Read output valid | `meta_guidance.json` exists in read mode |
| Write outputs valid | Required store files exist or are updated in write mode |
| Portable path | Store resides under the declared `meta_store_path` |
| No fabricated guidance | Guidance is grounded in real run artifacts |

**Failure**: Missing required mode inputs or corrupt store handling that cannot be bypassed honestly.

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E_META_INVALID_MODE` | `mode` is not `read` or `write` |
| `E_META_MISSING_RUN` | Write mode missing `run_id` or `artifacts_path` |
| `E_META_STORE_UNWRITABLE` | Meta store path could not be created or updated |
| `E_META_STORE_CORRUPT` | Existing meta store files are corrupt or unreadable |
| `E_META_READ_NO_DATA` | Read mode found no relevant prior experience |

---

## Retry Policy
Max retries: **1** — retry only on transient file-write problems.

---

## State Transition
`pending` → `running` → `done` | `failed`
