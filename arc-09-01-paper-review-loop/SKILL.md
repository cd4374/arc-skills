---
name: arc-09-01-paper-review-loop
description: Stage 24 — External adversarial multi-round paper review loop. Uses an independent expert reviewer to score the stage-22 package and produce remediation handoff guidance until threshold or max rounds.
metadata:
  category: pipeline-stage
  trigger-keywords: "paper review,external review,adversarial review,stage 24"
  applicable-stages: "24"
  priority: "1"
  version: "4.2"
  author: researchclaw
---

## Purpose

Run an external adversarial reviewer against the canonical stage-22 review package. The reviewer scores 1–10, lists weaknesses, and the loop records prioritized remediation guidance for the downstream polish/regeneration path. This breaks the self-review trap — the agent that wrote the paper is NOT the one judging it.

**This is a noncritical stage** — failures are logged honestly but do not abort the pipeline.

---

## Quality Contract

`review_state.json` MUST contain the final outcome:
- `status: "completed"` when `score >= 6` AND verdict contains "ready" or "almost"
- `status: "completed"` when `round >= MAX_ROUNDS` (exhausted iterations)
- `verdict: "ready"` | `"almost"` | `"not ready"`
- `score: N/10`
- `review_mode: "independent" | "degraded_local"`
- `warning_flags` records downgrade conditions such as reviewer non-independence
- `downstream_blockers` records issues that later stages MUST still respect even though Stage 24 itself is noncritical
- Review notes must explicitly flag citation-floor, recent-reference-ratio, figure-authenticity, and claim-traceability weaknesses when present

`AUTO_REVIEW.md` MUST contain all round entries with raw reviewer responses preserved verbatim.

---

## Inputs
`artifacts/<run_id>/stage-22/paper_final.md` (canonical stage-22 review package paper)
`artifacts/<run_id>/stage-23/verification_report.json` (citation verification)
`artifacts/<run_id>/stage-14/experiment_summary.json` (for result claims)
`artifacts/<run_id>/stage-14/analysis.md`
`artifacts/<run_id>/stage-24/review_state.json` (if resuming)
`artifacts/<run_id>/stage-24/REVIEWER_MEMORY.md` (nightmare mode)

---

## Outputs

### `review_state.json`
```json
{
  "run_id": "rc-YYYYMMDD-HHMMSS",
  "round": 3,
  "status": "in_progress",
  "difficulty": "nightmare",
  "reviewer_identity": "external_adversarial_reviewer",
  "reviewer_model": "gpt-5.4",
  "reviewer_provider": "codex_mcp",
  "review_mode": "independent",
  "warning_flags": [],
  "downstream_blockers": [],
  "last_score": 5.5,
  "figure_authenticity_score": 6.0,
  "last_verdict": "not ready",
  "threshold_met": false,
  "max_rounds_reached": false,
  "pending_experiments": [],
  "timestamp": "2026-04-05T15:00:00Z"
}
```

### `AUTO_REVIEW.md`
Cumulative review log — append one entry per round.

### `REVIEWER_MEMORY.md` (nightmare mode)
Persistent reviewer state across rounds.

Path convention for stage 24 artifacts:
- `artifacts/<run_id>/stage-24/review_state.json`
- `artifacts/<run_id>/stage-24/AUTO_REVIEW.md`
- `artifacts/<run_id>/stage-24/REVIEWER_MEMORY.md`

---

## Constants

| Constant | Value | Notes |
|----------|-------|-------|
| `MAX_ROUNDS` | 40 | Hard cap — loop stops here even without positive assessment |
| `POSITIVE_THRESHOLD` | score >= 6/10 AND verdict contains "ready" or "almost" | Stop condition |
| `REVIEWER_ROLE` | `external_adversarial_reviewer` | Reviewer should be executor-independent if available |
| `REVIEWER_MODEL` | `gpt-5.4` | Intended reviewer model via Codex MCP |
| `REVIEWER_PROVIDER` | `codex_mcp` | Intended independent reviewer path |
| `HUMAN_CHECKPOINT` | `false` | Set `true` to pause between rounds |
| `REVIEWER_DIFFICULTY` | `nightmare` | Fixed — only nightmare mode is allowed |

---

## Reviewer Identity Contract

The intended reviewer path is **Codex/GPT-5.4 via MCP**.

1. The preferred independent reviewer is a Codex-served GPT-5.4 reviewer reachable through MCP
2. If that path is unavailable, set `review_mode: degraded_local`, add `warning_flags: ["codex_mcp_unavailable"]`, and preserve all discovered blocking concerns in `downstream_blockers`
3. Degraded-local review may inform downstream fixes, but it must never be misrepresented as independent review
4. Stage 24 remains noncritical, but later critical gates must still respect every blocking concern recorded here

## Difficulty Mode

### Nightmare (only mode)
Reviewer independently verifies claims against paper, code, citations, figures, and experiment artifacts rather than accepting a curated summary at face value.

**Reviewer Memory Structure:**

After each round, update `artifacts/<run_id>/stage-24/REVIEWER_MEMORY.md`:

```markdown
# Reviewer Memory

## Round N — Score: X/10

### Suspicion Log
- **[timestamp]** Claim "X improves Y by Z%" — need to verify against experiment data
- **[timestamp]** Method section mentions "novel attention" — compare to cited papers
- **[timestamp]** Figure 3 shows error bars — check if they match reported std

### Unresolved Concerns
- [ ] Verify: Main claim about accuracy improvement
- [ ] Verify: Novelty claim in Introduction
- [ ] Check: Statistical significance of ablation results

### Patterns Detected
- Recurring: Vague claims without specific numbers
- Recurring: Method description doesn't match cited code
- Recurring: Figures referenced but not explained

### Executor Rebuttals
- R1: Executor argued X is novel — PARTIALLY SUSTAINED (needs more evidence)
- R2: Executor fixed figure 3 — SUSTAINED (resolved)

### Trust Score Evolution
- Round 1: 0.3 (many suspicious claims)
- Round 2: 0.5 (some fixes, but new concerns)
- Round 3: 0.6 (improving, but verification needed)
```

**Trust Score System:**
- 0.0-0.3: Low trust — heavy skepticism, demand more evidence
- 0.3-0.6: Moderate trust — some concerns remain
- 0.6-0.8: Good trust — minor issues only
- 0.8-1.0: High trust — paper is solid

### Nightmare (+ Evidence Verification)
Reviewer independently verifies claims against:
- `experiment_summary.json` (numeric claims)
- `references_verified.bib` and `verification_report.json` (citation accuracy)
- Source code in `experiment_final/` (method claims)
- `charts/` plus figure provenance artifacts (figure authenticity)

---

## Procedure

### Step 1 — Initialize / Resume

Check for existing `artifacts/<run_id>/stage-24/review_state.json`:

| State | Action |
|-------|--------|
| File absent | Fresh start — initialize round = 1 |
| `status == "completed"` | Fresh start (previous loop finished) |
| `status == "in_progress"` AND timestamp > 24h old | Fresh start (stale — abandoned) |
| `status == "in_progress"` AND timestamp <= 24h | Resume from saved round + 1 |

On fresh start: read all inputs, create `artifacts/<run_id>/stage-24/AUTO_REVIEW.md`, and write initial `artifacts/<run_id>/stage-24/review_state.json`.

### Step 2 — Build Context Package

Build the nightmare review package with:
- Current exported paper text
- Experiment results summary
- Prior round notes (if round > 1)
- `artifacts/<run_id>/stage-24/REVIEWER_MEMORY.md` from prior rounds
- Trust score history
- Direct access to:
  - `experiment_summary.json` (verify all numeric claims)
  - `artifacts/<run_id>/stage-23/references_verified.bib` and `verification_report.json` (verify citations)
  - Source code in `experiment_final/` (verify method implementations)
  - figure files and provenance records in the canonical package (verify authenticity)

### Step 3 — Select Reviewer and Review

Select a reviewer using this priority:
1. Codex/GPT-5.4 reviewer via MCP (`review_mode: independent`)
2. If the intended Codex MCP path is unavailable, set `review_mode: degraded_local`, add `warning_flags: ["codex_mcp_unavailable"]`, and continue with explicit warning

A degraded-local review can inform downstream polish, but it does not erase any blocking issue discovered in citation verification, numeric truth, claim traceability, figure authenticity, or submission-format checks. Any such concern should also be recorded in `downstream_blockers` so later stages must respect it.

Record in `artifacts/<run_id>/stage-24/review_state.json`:
- `reviewer_identity`
- `reviewer_model`
- `reviewer_provider`
- `review_mode`: `independent` | `degraded_local`
- `warning_flags`: empty when independent review is used; otherwise include the downgrade reason

**Prompt to reviewer:**

```
You are an external adversarial reviewer operating through Codex MCP with GPT-5.4 in nightmare mode.

Round {N}/{MAX_ROUNDS} of adversarial review.

**Your Task:**
1. Score this paper 1–10 for its target venue
2. List critical weaknesses ranked by severity
3. For each weakness: specify the MINIMUM fix
4. Decide READY / ALMOST / NOT READY
5. Score figure authenticity separately from 0–10
6. Explicitly assess whether the paper is on track for at least 30 verified references, at least 20% recent references, and evidence-bearing non-decorative figures
7. Treat any mismatch between paper claims, verified citations, figure provenance, or source code as a blocking concern

**Be brutally honest.** Do not reward polish if the evidence chain is weak.

**Nightmare Evidence Checklist:**
- [ ] Every numeric claim matches `experiment_summary.json`
- [ ] Every citation used in the paper is present in `references_verified.bib` and consistent with `verification_report.json`
- [ ] Method description matches actual code in `experiment_final/`
- [ ] Every major figure is evidence-bearing, not decorative, and consistent with its provenance record
- [ ] Claim direction/significance language matches the evidence rather than overstating it

[Paper content]
```

### Step 4 — Parse Assessment

Extract:
- **Score** (numeric 1–10)
- **Verdict** ("ready" / "almost" / "not ready")
- **Action items** (ranked list of fixes)

**STOP CONDITION**: If `score >= 6` AND verdict contains "ready" or "almost" → loop terminates.

### Step 5 — Debate Protocol (nightmare mode)

Executor may rebut up to 3 weaknesses per round.

**Claude's Rebuttal:**
```markdown
### Rebuttal to Weakness #1: [title]
- **Accept / Partially Accept / Reject**
- **Argument**: [why criticism is invalid or already addressed]
- **Evidence**: [specific code, results, or prior fixes]
```

Send to reviewer for ruling:
- **SUSTAINED**: weakness withdrawn from action items
- **OVERRULED**: keep as-is
- **PARTIALLY SUSTAINED**: revise scope

Update `artifacts/<run_id>/stage-24/REVIEWER_MEMORY.md` with debate outcome.

### Step 6 — Prepare Remediation Handoff

For each action item, highest priority first:
- Translate the reviewer criticism into a concrete remediation item for downstream execution
- Classify the likely owner: analysis update, paper rewrite, experiment rerun, citation cleanup, or packaging fix
- Record whether the issue appears addressable within Stage 25 or would require rollback to an earlier stage
- If an issue would still be blocking for Stages 23 or 26–28 (for example fake/unverifiable citations, unsupported claim direction, decorative figures, or reproducibility drift), also record it in `downstream_blockers`
- Preserve the canonical `artifacts/<run_id>/stage-22/` package unchanged during this review stage
- Do not regenerate paper text, bibliography, PDF, or deliverables in Stage 24; canonical regeneration remains owned by Stage 25

### Step 7 — Update Reviewer Memory

**Nightmare mode:**

Update `artifacts/<run_id>/stage-24/REVIEWER_MEMORY.md`:
- Add new suspicions from this round
- Mark resolved concerns
- Update trust score based on fixes
- Log rebuttal outcomes

### Step 8 — Document Round

Append to `artifacts/<run_id>/stage-24/AUTO_REVIEW.md`:
```markdown
## Round N (timestamp)

### Assessment
- Score: X/10
- Verdict: [ready/almost/not ready]
- Key criticisms: [bullet list]

### Reviewer Raw Response
[verbatim — complete, unedited]

### Debate Outcome (nightmare mode)
- [weakness]: [SUSTAINED/OVERRULED/PARTIALLY SUSTAINED]

### Remediation Handoff
- [what should be changed downstream]

### Status
- [continuing to round N+1 / stopping]
```

Update `artifacts/<run_id>/stage-24/review_state.json`: increment round, update score/verdict, refresh `warning_flags`, and persist any `downstream_blockers` identified in the round.

Increment round counter → back to Step 2.

### Step 9 — Termination

When loop ends:
1. Write `status: "completed"` to `artifacts/<run_id>/stage-24/review_state.json`
2. Preserve `review_mode` and any `warning_flags` in the final state file
3. Write final summary to `artifacts/<run_id>/stage-24/AUTO_REVIEW.md`
4. List remaining blockers if max rounds reached without threshold

---

## Reviewer Memory Schema

```markdown
# Reviewer Memory

## Round Summary
| Round | Score | Trust | Key Concern |
|-------|-------|-------|-------------|
| 1 | 4.5 | 0.2 | Method not novel |
| 2 | 5.5 | 0.4 | Experiments incomplete |
| 3 | 6.0 | 0.6 | Minor clarity issues |

## Suspicion Patterns
- Pattern A: Claims without numbers → resolved in R2
- Pattern B: Vague novelty claims → partially resolved
- Pattern C: Missing ablations → still open

## Verification Status
- [x] Main result verified against experiment data
- [x] Citations checked
- [ ] Ablations run and verified
- [ ] Statistical tests added

## Trust Score Calculation
Trust = 0.3 * (fix_quality) + 0.3 * (evidence_strength) + 0.2 * (clarity) + 0.2 * (novelty_support)

Current: 0.6 (moderate trust, minor issues remain)
```

---

## Validation

| Check | Pass Condition |
|-------|---------------|
| `AUTO_REVIEW.md` exists | `artifacts/<run_id>/stage-24/AUTO_REVIEW.md` contains one entry per completed round |
| `review_state.json` valid | `artifacts/<run_id>/stage-24/review_state.json` is valid JSON with required fields |
| Downgrade recorded correctly | If independent review is unavailable, `review_mode == degraded_local` and `warning_flags` is non-empty |
| Downstream blockers preserved | Blocking concerns discovered during review are recorded in `downstream_blockers` rather than implied to be waived |
| Raw reviewer response preserved | Each round entry contains verbatim response |
| Remediation handoff recorded | Each round records concrete downstream remediation items without mutating the canonical stage-22 package |
| Loop terminates | At `MAX_ROUNDS` or when threshold met |
| Reviewer memory maintained | `artifacts/<run_id>/stage-24/REVIEWER_MEMORY.md` is updated each round (nightmare mode) |

**Failure**: Stage fails only on unrecoverable state (file write failure, corrupted state).

---

## Error Codes

| Code | Trigger |
|------|---------|
| `E24` | Stage-24 review loop state is corrupted or could not be written |
| `E24_ROUND_FAIL` | A review round completed without any measurable improvement |
| `E24_REVIEWER_UNAVAILABLE` | No independent reviewer is available; record `review_mode: degraded_local` and log the downgrade honestly |
| `E24_CODEX_MCP_UNAVAILABLE` | The intended Codex/GPT-5.4 MCP reviewer path was unavailable; continue only with explicit degraded-local logging |

---

## Retry Policy
Max retries: **1** — retry only on state file write failures.

---

## State Transition
`pending` → `running` → `done` | `failed`

Note: Even on `failed`, the paper may continue to `arc-09-02-paper-polish` for a final critical cleanup pass.