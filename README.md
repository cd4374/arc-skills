# arc-skills

Pure-skills research pipeline for AutoResearchClaw, using stage-addressable skill names:

- Format: `arc-<major>-<minor>-<name>`
- `major`: phase index (`00` orchestrator/control, `01`..`08` pipeline phases)
- `minor`: sub-stage index within the phase

## Entry points

- `/arc-00-01-research-pipeline` — run full pipeline chain
- `/arc-00-02-status` — show run status from state files
- `/arc-00-03-resume` — resume from checkpoint

## Stage structure

- `arc-01-*` Phase A: Scoping
- `arc-02-*` Phase B: Literature
- `arc-03-*` Phase C: Synthesis
- `arc-04-*` Phase D: Experiment Design
- `arc-05-*` Phase E: Execution
- `arc-06-*` Phase F: Analysis/Decision
- `arc-07-*` Phase G: Writing
- `arc-08-*` Phase H: Finalization

## State files

Runs persist state under run root:

- `artifacts/<run_id>/pipeline_state.json`
- `artifacts/<run_id>/checkpoint.json`
- `artifacts/<run_id>/stage_history.jsonl`
- `artifacts/<run_id>/decision_history.json`
- `artifacts/<run_id>/heartbeat.json`
- `artifacts/<run_id>/pipeline_summary.json`

## Notes

Gate-equivalent stages:
- `arc-02-03-literature-screen`
- `arc-04-01-experiment-design`
- `arc-08-01-quality-gate`

Decision loop guard:
- `arc-06-02-research-decision` supports `proceed/refine/pivot`
- max pivot/refine loops = 2, then forced proceed with warning
