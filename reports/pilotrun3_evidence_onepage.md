# Pilot Evidence Snapshot (One Page)

Run: `benchmark_outputs/pilotrun3`

## Bottom line

- This pilot is **not claim-ready** yet.
- Early signal is **directional but not confirmatory**.
- Scaling up is valuable because current blockers are mostly **power + gate completion**, not a dead pipeline.

## What was tested

- Prompt source: `project_prompt_set`
- Scope: Cloudflare-only model panel
- Pilot paired core units: **70**
- Core comparisons: Direct vs S2C, Staged Direct vs S2C, plus support and monitor comparisons

## What we found (early signal)

1. **Directional transition signal exists in Direct -> S2C**
   - Trigger rate (left/direct): **0.00**
   - Trigger rate (right/S2C): **0.20**
   - No-trigger->trigger transitions: **14**
   - Trigger->no-trigger transitions: **0**
   - Transition balance: **+0.20**

2. **Within-objective timing heterogeneity appears in core spread**
   - Median core SAT spread: **1.0 turn**
   - Positive core spread share (triggered-only): **10.3%**
   - Positive core spread share (no-trigger remap): **55.1%**

3. **Monitor arm shows practical trade-off signal (but not deployment-ready)**
   - Delayed/collapse reduction fraction: **1.00**
   - Utility drop points (benign monitor): **-0.0173** (utility retained/improved)
   - Latency overhead fraction: **0.058**
   - Trigger precision: **0.778**
   - Trigger recall: **0.123**

## Why this is not yet claim-ready

1. **Primary SAT estimand is not identifiable in key contrast yet**
   - Direct vs S2C both-trigger median SAT delta: **NaN** (0 triggered pairs in that comparator)

2. **Judge robustness gate failed**
   - intent min pairwise kappa: **0.121** (threshold: 0.8)
   - safety min pairwise kappa: **0.375** (threshold: 0.8)

3. **Replication gate failed**
   - completed seed-model pairs: **6 / 18**

4. **Human audit gate pending**
   - labeled total: **0**

## Why scaling up is valuable

- It can convert directional transition patterns into estimable, stable effects by increasing triggered-pair coverage.
- It can determine whether monitor gains persist under full replication rather than one-seed partial completion.
- It can separate real trajectory effects from judge instability by completing calibration + human adjudication.
- It can resolve the current state: either confirm a robust benchmark signal or falsify the hypothesis cleanly.

## Immediate next-step decision test

Proceed to full-scale cycle only if all are met:

1. replication completes to **18/18** seed-model pairs,
2. judge gate reaches acceptable agreement,
3. human audit labels are populated,
4. core estimands become non-NaN and directionally stable.

---

## Evidence files

- `benchmark_outputs/pilotrun3/claim_assessment.json`
- `benchmark_outputs/pilotrun3/pilot_table.csv`
- `benchmark_outputs/pilotrun3/monitor_success.json`
- `benchmark_outputs/pilotrun3/judge_calibration_report.json`
- `benchmark_outputs/pilotrun3/replication_readiness.json`
