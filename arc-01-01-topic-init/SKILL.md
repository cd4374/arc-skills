---
name: arc-01-01-topic-init
description: Stage 1 — Define and formalize the research topic as a SMART goal.
metadata:
  category: pipeline-stage
  trigger-keywords: "topic init,goal,stage 1"
  applicable-stages: "1"
  priority: "1"
  version: "3.0"
  author: researchclaw
---

## Purpose
Formalize the user's research idea into a well-scoped SMART goal with hardware environment profile. This stage establishes the foundation for all downstream decisions.

---

## Quality Contract

`goal.md` MUST satisfy all of the following:
- Topic is a single sentence (< 100 words)
- Scope is defined (what is included and explicitly excluded)
- Domain is identified (e.g., ml, nlp, cv, rl, theory)
- A falsifiable claim is present (can be supported or refuted by experiments)

`hardware_profile.json` MUST be accurate for the current execution environment.

---

## Inputs
- `topic` (required): raw research idea from the user — any length, any format
- `constraints` (optional): target conference, time budget, domain hints

---

## Outputs

### `goal.md`
```
# Research Goal

## Topic
[1-sentence research topic]

## Scope
[Explicit inclusions and exclusions]

## Constraints
- Domain: [identified domain]
- Target: [conference if specified]
- Falsifiable claim: [the testable assertion]
```

### `hardware_profile.json`
```json
{
  "compute_backend": "cuda" | "mps" | "cpu",
  "gpu_model": "...",
  "gpu_memory_gb": 0,
  "python_version": "3.x",
  "torch_available": true | false
}
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| `goal.md` exists | File non-empty |
| Topic is single sentence | ≤ 100 words in Topic section |
| Scope is defined | Both inclusions and exclusions present |
| Falsifiable claim | Contains a testable assertion |
| `hardware_profile.json` valid | JSON with `compute_backend` field |

**Failure**: If any check fails → `E01`

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E01` | Topic is not falsifiable, empty, or unreadable |

---

## Retry Policy
Max retries: **0** — topic initialization failures cannot be retried automatically.

---

## State Transition
`pending` → `running` → `done` | `failed`

---

## Procedure

### Step 1 — Parse Topic
Read the raw `topic` input. If `constraints` is provided, note any conference targets, time budget, or domain hints.

### Step 2 — Write `goal.md`
Create `goal.md` with these four sections:
1. **Topic**: Condense the raw idea into exactly one sentence (< 100 words). Remove vagueness — a good topic sentence makes the falsifiable claim obvious.
2. **Scope**: List what the research INCLUDES and what it EXPLICITLY EXCLUDES. Be specific about dataset types, task types, and methodology scope.
3. **Domain**: Assign to one or more of `ml`, `nlp`, `cv`, `rl`, `theory`, `systems`, `security`.
4. **Falsifiable Claim**: Write one specific, testable assertion that experiments can either SUPPORT or REJECT. The claim must mention a metric and a direction (e.g., "Method X outperforms Method Y by ≥5% on metric Z").

### Step 3 — Detect Hardware
Detect the current execution environment and write `hardware_profile.json`:
- Linux/macOS: run `python3 -c "import torch; print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'cpu')"` to get GPU model
- Check GPU memory: `nvidia-smi --query-gpu=memory.total --format=csv,noheader` (Linux) or `system_profiler SPDisplaysDataType` (macOS)
- Fall back to `cpu` if no GPU

### Step 4 — Validate Outputs
Run all checks from the Validation table. If any fails, fail with `E01`.
