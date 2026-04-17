# Pilotrun3 One-Page Evidence (Direct Answer)

Run analyzed: benchmark_outputs/pilotrun3

## Q1) What did you find?

We ran a full pilot pipeline using the project prompt set and the fixed Cloudflare model panel, with 70 paired core units.

Early directional findings:
- Direct -> S2C trigger rate increased from 0.00 to 0.20.
- No-trigger -> trigger transitions were 14, while trigger -> no-trigger transitions were 0.
- Core timing heterogeneity appeared (median core SAT spread = 1.0 turn).

Practical monitor findings:
- Trigger precision = 0.778.
- Utility drop on benign traffic = -0.0173 (no observed utility harm).
- Latency overhead = 0.058.

## Q2) What is the early signal?

The early signal is that trajectory form changes safety behavior in a directional way (transition asymmetry and positive spread signal), but the key confirmatory SAT shift is not yet estimable in the most important contrast.

Specifically:
- Direct vs S2C both-trigger SAT delta is NaN (0 triggered pairs in that comparator).
- Staged vs S2C both-trigger SAT delta is 0.0.

So this is a real pilot signal, but still pre-confirmatory.

## Q3) What is the significance?

Significance today is methodological, not claim-level:
- The benchmark pipeline works end-to-end.
- It produces consistent directional and operational outputs (timing + monitor trade-offs).
- It also correctly blocks over-claiming through gates.

Under the research plan evidence ladder, this run is still pipeline-validation stage, not a claim-positive stage.

## Q4) Why is scaling up valuable?

Scaling up is valuable because current blockers are exactly the kind that larger, fully gated runs can resolve:
- Replication is incomplete (6/18 seed-model pairs).
- Judge agreement gate failed (intent min kappa 0.121, safety min kappa 0.375; gate = 0.8).
- Human audit labels are pending (0 labeled).

A full run can determine whether the directional pattern stabilizes into a robust effect or collapses under replication.

## Current status in one line

Promising early directional signal + functioning benchmark pipeline, but not claim-ready yet.

## Evidence sources

- benchmark_outputs/pilotrun3/claim_assessment.json
- benchmark_outputs/pilotrun3/pilot_table.csv
- benchmark_outputs/pilotrun3/monitor_success.json
- benchmark_outputs/pilotrun3/judge_calibration_report.json
- benchmark_outputs/pilotrun3/replication_readiness.json
- temporal_alignment_research_plan.txt (evidence ladder and downgrade rules)
- temporal_alignment_hysteresis_experiment.ipynb (gate-enforcement logic)
