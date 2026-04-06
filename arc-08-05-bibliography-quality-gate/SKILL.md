---
name: arc-08-05-bibliography-quality-gate
description: Stage 18.5 (GATE) — Check bibliography completeness and basic quality before peer-review findings propagate downstream.
metadata:
  category: pipeline-stage
  trigger-keywords: "bibliography quality gate,bib quality,reference completeness,stage 18.5"
  applicable-stages: "18.5"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Purpose
Catch bibliography defects before they survive into quality gate, export, and citation verification. This stage focuses on entry completeness, unresolved verification markers, visible citation drift, and stage-appropriate quantity/recency expectations.

---

## Quality Contract

`bibliography_quality_report.json` MUST:
- Confirm every citation key used in the draft has a bibliography entry
- Confirm no bibliography entry remains tagged as unresolved (`[VERIFY]` or similar)
- Record interim bibliography count and recent-reference ratio
- Record duplicate-key and missing-field issues
- Set `bibliography_quality_pass: true` only if all blocking checks pass

Hard constraints:
- Every in-text citation MUST resolve to an entry
- No unresolved verification markers may remain in the bibliography
- No duplicate citation keys with inconsistent metadata may remain
- Core bibliography metadata fields (author, title, year) must exist for every cited entry

---

## Inputs
`artifacts/<run_id>/stage-17/paper_draft.md`
`artifacts/<run_id>/stage-04/candidates.jsonl`
`artifacts/<run_id>/stage-17/references.bib`

---

## Outputs

### `artifacts/<run_id>/stage-18b/bibliography_quality_report.json`
```json
{
  "bibliography_quality_pass": true,
  "citation_keys_used": 28,
  "entries_present": 28,
  "recent_reference_ratio": 0.25,
  "duplicate_keys": [],
  "missing_fields": [],
  "unresolved_entries": [],
  "warnings": [
    "Reference count is below the final target of 30; continue adding verified literature upstream."
  ],
  "blocking_issues": []
}
```

### `artifacts/<run_id>/stage-18b/bibliography_quality_checklist.md`
A short checklist for citation-key coverage, unresolved markers, required fields, and interim recency/count status.

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| All used keys resolve | Every in-text citation key has a bibliography entry |
| No unresolved entries | No entry remains marked `[VERIFY]` or equivalent |
| Required fields present | Author, title, and year exist for every cited entry |
| No duplicate key conflict | No duplicate citation keys with inconsistent metadata |
| Gate pass flag | `bibliography_quality_pass == true` |

**Failure**: Missing entries, unresolved markers, or duplicate conflicts → `E18B`

---

## Gate Behavior

| Action | Outcome |
|--------|---------|
| `auto_approve_gates=true` and `bibliography_quality_pass=true` | `done`, advance to stage 19 |
| `approve` and `bibliography_quality_pass=true` | `done`, advance to stage 19 |
| `reject` or `bibliography_quality_pass=false` | `rejected`, rollback to stage 17 |

---

## Rollback Contract
On `reject` or `bibliography_quality_pass=false`: rollback target is **stage 17** (`arc-07-02-paper-draft`).

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E18B` | Bibliography quality report missing or blocking checks failed |
| `E18B_MISSING_ENTRIES` | One or more in-text citation keys have no bibliography entry |
| `E18B_UNRESOLVED_ENTRY` | An unresolved verification marker remains in the bibliography |
| `E18B_DUPLICATE_KEY_CONFLICT` | Duplicate citation key conflict detected |
| `E18B_MISSING_FIELDS` | Required metadata fields missing from cited entries |

---

## Retry Policy
Max retries: **0** — gate rejection uses rollback path.

---

## State Transition
`pending` → `running` → `blocked_approval` | `failed`

---

## Procedure

### Step 1 — Read Draft and Bibliography
Read `paper_draft.md`, `references.bib`, and the verified literature corpus index.

### Step 2 — Extract Citation Keys
Extract all citation keys used in the draft and compare them with the bibliography entries.

### Step 3 — Check Entry Completeness
For each cited entry, verify required metadata fields exist and scan for unresolved verification markers.

### Step 4 — Check Interim Quality
Compute bibliography count, recent-reference ratio, duplicate-key conflicts, and other non-blocking quality warnings.

### Step 5 — Write Outputs
Write `bibliography_quality_report.json` and `bibliography_quality_checklist.md` under `stage-18b/`.

### Step 6 — Gate Decision
If any cited key is missing, unresolved markers remain, or duplicate-key conflicts exist, fail with `E18B`. Otherwise block for approval and then advance to stage 19.
