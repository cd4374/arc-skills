---
name: arc-01-02-problem-decompose
description: Stage 2 (PROBLEM_DECOMPOSE). Break the goal into prioritized sub-questions.
metadata:
  category: pipeline-stage
  trigger-keywords: "problem decompose,sub-questions,stage 2"
  applicable-stages: "2"
  priority: "1"
  version: "1.0"
  author: researchclaw
---

## Stage
- Canonical stage: 2 `PROBLEM_DECOMPOSE`

## Inputs
From `artifacts/<run_id>/stage-01/`:
- `goal.md`

## Outputs
Write to `artifacts/<run_id>/stage-02/`:
- `problem_tree.md`

## Definition of done
At least 3 prioritized, testable sub-questions are documented.
