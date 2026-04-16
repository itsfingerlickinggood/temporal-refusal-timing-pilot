# Notebook-Plan Consistency Report

Notebook reviewed: [temporal_alignment_hysteresis_experiment.ipynb](/d:/ai-safety-research/1/temporal_alignment_hysteresis_experiment.ipynb)
Revised plan: [temporal_alignment_research_plan_revised.txt](/d:/ai-safety-research/1/temporal_alignment_research_plan_revised.txt)

## Summary
The notebook now matches the revised plan at the level of executable logic and artifact structure. It is not yet evidence-complete because the workspace still lacks a real project prompt set and audited non-mock runs.

## Mismatches Found In The Original Notebook
- It loaded tasks from an unrelated prompt-injection benchmark artifact.
- It treated `baseline_context` as a core analysis arm, which diluted the main claim.
- It ran only one target model while the plan described a fixed panel.
- It simulated mitigation by manually boosting safety scores after the run.
- It simulated replication by resampling and adding jitter.
- It did not include benign controls for utility retention.
- It created a vague source traceability map instead of a concrete support table.

## What Was Patched
- Core conditions were tightened to `baseline_direct`, `s2c_cloaked`, `s2c_monitor`, `benign_control`, and `benign_monitor`.
- The dataset loader now prefers a project-specific prompt file and labels demo fallbacks as non-evidentiary.
- The main loop now executes the full fixed model panel in the notebook logic.
- The monitor is now an intervention branch with escalation prompting, not a post hoc score edit.
- Replication is now represented as a pending fresh-run schedule instead of fake repeated results.
- Judge calibration now uses three judges and reports pairwise kappa.
- Human audit export is now stratified by model, condition, and phase.
- Source traceability now records concrete source rows and unsupported anchors.

## Remaining Gaps
- `temporal_alignment_prompt_set.csv` does not yet exist, so the current run is explicitly `claim_ready = false`.
- Seed replication is scaffolded but not complete; only the default run seed has been executed in the validation pass.
- The local PDF folder does not support stronger monitoring/introspection motivation claims.
- The notebook still validates the pipeline with demo placeholders rather than producing submission-grade results.

## Current Artifacts Produced By The Revised Notebook
Latest validation run: [benchmark_outputs/run_20260413_180604_0aa3c59a](/d:/ai-safety-research/1/benchmark_outputs/run_20260413_180604_0aa3c59a)

Key outputs:
- `turn_level_logs.csv`
- `trajectory_metrics.csv`
- `core_pairs.csv`
- `monitor_pairs.csv`
- `utility_pairs.csv`
- `judge_calibration_report.json`
- `replication_schedule.csv`
- `replication_readiness.json`
- `human_audit_sample.csv`
- `ideas_rp_source_traceability.csv`

## Readiness Verdict
- Method alignment: good
- Evidence alignment: incomplete
- Submission readiness from current artifacts: no
- Fastest unlock for submission-grade evidence: populate the real prompt set and rerun in non-mock mode
