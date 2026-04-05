---
name: arc-02-04-knowledge-extract
description: Stage 6 — Extract structured knowledge cards from the shortlisted papers.
metadata:
  category: pipeline-stage
  trigger-keywords: "knowledge extract,cards,stage 6"
  applicable-stages: "6"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Transform the shortlist into structured knowledge cards. Each card captures: what the paper does, how it works, what results it achieves, and how it relates to the current research goal. These cards are the source material for synthesis and hypothesis generation.

---

## Quality Contract

`cards/` directory MUST satisfy:
- One `.json` card file per shortlisted paper (card count = shortlist count)
- Each card has non-empty `core_contribution`, `method`, and `key_results` fields
- `cards_index.json` lists all cards with their paper IDs

---

## Inputs
`artifacts/<run_id>/stage-05/shortlist.jsonl`

---

## Outputs

### `cards/{paper_id}.json`
```json
{
  "paper_id": "openalex:W123456789",
  "title": "...",
  "authors": ["Author A", "Author B"],
  "year": 2023,
  "venue": "NeurIPS",
  "core_contribution": "[One-sentence summary of the main contribution]",
  "method": "[Algorithm, architecture, or technical approach — be specific]",
  "key_results": ["[Result 1 with metric]", "[Result 2 with metric]"],
  "limitations": ["[Limitation 1]", "[Limitation 2]"],
  "relation_to_topic": "[How this paper relates to the current research goal]",
  "experimental_setup_relevant": true,
  "code_available": false
}
```

### `cards_index.json`
```json
[{"paper_id":"W123456789","title":"...","card_file":"cards/W123456789.json"}]
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| Card per paper | `cards/` has ≥ 1 file per entry in `shortlist.jsonl` |
| `core_contribution` non-empty | All cards have a non-empty string |
| `method` non-empty | All cards have a non-empty string |
| `key_results` non-empty | All cards have at least 1 result entry |
| `cards_index.json` exists | File present and lists all cards |

**Failure**: If any check fails → `E06`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E06` | Missing card for any shortlisted paper, or card missing required fields |

---

## Retry Policy
Max retries: **1** — retry if API failures prevented fetching content for >50% of papers.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Read Shortlist
Read `shortlist.jsonl` from stage-05. Count the number of shortlisted papers.

### Step 2 — Fetch Full Content
For each paper in the shortlist, attempt to fetch the full paper or extended abstract:
- arXiv: `curl -s "https://export.arxiv.org/api/query?id_list={arxiv_id}"` → parse abstract, authors, published year
- OpenAlex: `curl -s "https://api.openalex.org/works/{openalex_id}?mailto=anonymous@example.com"` → parse abstract, authors, year, venue, citation_count

If paper content is inaccessible, create a card from the title, abstract, and metadata available.

### Step 3 — Create Knowledge Cards
For each paper, create `cards/{paper_id}.json` with:
- `paper_id`: the openalex_id or arxiv_id
- `title`, `authors`, `year`, `venue`: from fetched metadata
- `core_contribution`: One-sentence summary of the main contribution (infer from abstract)
- `method`: The algorithm, architecture, or technical approach — be specific (e.g., "Transformer encoder with 12 layers, 768-dim hidden, 12 attention heads")
- `key_results`: Array of specific quantitative results with metrics (e.g., "94.2% accuracy on ImageNet, 3.8x faster than ResNet-50")
- `limitations`: Specific limitations stated in the paper
- `relation_to_topic`: How this paper connects to the current research goal and sub-questions
- `experimental_setup_relevant`: Boolean — whether the paper's experiments use datasets/setup relevant to the current problem
- `code_available`: Boolean — whether official code is publicly available

### Step 4 — Write cards_index.json
List all card files with their paper IDs and titles.

### Step 5 — Validate
Check: card exists for every shortlisted paper, all required fields non-empty. Fail → `E06`.
