---
name: arc-02-01-search-strategy
description: Stage 3 — Build a literature search plan with verified data sources and concrete queries.
metadata:
  category: pipeline-stage
  trigger-keywords: "search strategy,literature plan,stage 3"
  applicable-stages: "3"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Create a search plan that will retrieve the relevant literature corpus. The plan must specify data sources, query strategy, and coverage targets. Sources must be verified accessible before the plan is finalized.

---

## Quality Contract

`search_plan.yaml` + `sources.json` + `queries.json` together MUST satisfy:
- ≥2 data sources confirmed accessible (live API or reachable endpoint)
- ≥5 unique search queries, each ≤60 characters
- Queries collectively cover: the core method, comparison targets, and application domain
- At least one query targets each sub-question in `problem_tree.md`

---

## Inputs
`artifacts/<run_id>/stage-02/problem_tree.md`

---

## Outputs

### `search_plan.yaml`
```yaml
strategy:
  name: multi-source systematic
  description: [approach summary]
data_sources:
  - name: [e.g. OpenAlex]
    priority: 1
    verified: true
query_strategy:
  - type: [e.g. exact_topic]
    weight: high
```

### `sources.json`
```json
[{"name":"OpenAlex","verified":true},{"name":"arXiv","verified":true}]
```

### `queries.json`
```json
[
  {"query": "krylov subspace preconditioning", "type": "method", "covers": "SQ1"},
  {"query": "randomized numerical linear algebra", "type": "method", "covers": "SQ1"}
]
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| ≥2 verified sources | Each has `"verified": true` |
| ≥5 unique queries | Array length ≥ 5 |
| Query length | Each `query` ≤ 60 characters |
| Coverage | Each SQ has ≥1 covering query |

**Failure**: If any check fails → `E03`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E03` | < 2 verified sources, < 5 queries, or queries don't cover all SQs |

---

## Retry Policy
Max retries: **1** — retry if source verification fails (transient network issue).

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Problem Tree
Read `artifacts/<run_id>/stage-02/problem_tree.md`. Extract all sub-questions (SQ1, SQ2, ...). Note the domain and scope from `goal.md` (stage-01).

### Step 2 — Select Data Sources
Identify ≥2 suitable data sources from: OpenAlex, arXiv, Semantic Scholar, Google Scholar, DBLP. Verify each source is accessible:
- OpenAlex: `curl -s "https://api.openalex.org/sources?filter=host_venue.name:something" | head` → must return JSON
- arXiv: `curl -s "https://export.arxiv.org/api/query?search_query=all:test&start=0&max_results=1"` → must return XML
- Semantic Scholar: `curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=test&limit=1"` → must return JSON

Write `sources.json` with each source marked `"verified": true` only after confirmed accessible.

### Step 3 — Generate Queries
Generate ≥5 unique queries (≤60 chars each) covering:
- Core method of the proposed approach
- Comparison target baselines (e.g., "Transformer", "GNN", "fine-tuning")
- Application domain (from goal.md domain field)
- Each sub-question should have ≥1 covering query

### Step 4 — Write Outputs
Create `search_plan.yaml` (strategy description), `sources.json` (verified sources), and `queries.json` (queries with `covers` field linking to SQ).

### Step 5 — Validate
Run Validation checks. Fail → `E03`.
