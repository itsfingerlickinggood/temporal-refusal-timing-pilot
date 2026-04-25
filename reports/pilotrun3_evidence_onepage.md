# Do AI Safety Guardrails Fail *Slowly*?
### A benchmark project measuring **when** AI models refuse harmful requests — not just whether they do.

---

## The Problem — Safety Is Measured Wrong

Today, AI safety tools ask one question: *did the AI say no?* But this misses something important. Sometimes an AI eventually refuses — but only after a bad actor has already received useful, harmful fragments through a clever multi-turn conversation. **Timing matters.** A refusal that comes on turn 5 is not the same as one that comes on turn 1. Current benchmarks treat them identically. That is the gap this project addresses.

---

## The Core Insight — How You Ask Changes When the AI Refuses

The same harmful goal, asked directly, usually triggers an instant refusal. But split across several turns, disguised with innocent-sounding context, or gradually reassembled through a cloaked multi-turn conversation, the very same goal causes the AI to refuse **later** — or not at all. This project calls that gap **trajectory-sensitive refusal timing**, and measures it precisely.

---

## How This Project Helps — A Benchmark That Measures the Delay

The project builds a reproducible test suite. For each harmful goal, it creates three conversation versions:

- **Direct** — one blunt, single-turn request.
- **Staged** — the same request broken into plain, explicit steps across multiple turns.
- **Cloaked (S2C)** — the same goal disguised gradually across turns using contextual reframing and semantic camouflage.

The benchmark then measures exactly which turn the AI refuses in each case, and by how much the cloaked version delays that refusal relative to both controls. The output is a reusable benchmark artifact any AI lab can run on their own models.

---

## Practical Safety Output — A Monitor That Catches Slow-Burn Attacks

Beyond measuring the problem, the project tests a **real-time monitor**: a lightweight intervention layer that watches conversations for signs of gradually escalating harmful intent — before the AI has been manipulated into full compliance. Pilot results show it flags harmful trajectories with **78% precision**, adds only a **6% latency overhead**, and causes **no measurable harm** to normal, benign conversations.

---

## Pilot Run 3 — Early Evidence (70 Conversation Pairs)

| Metric | Result |
|---|---|
| Cloaked conversation trigger rate | **0.20** (was 0.00 in the direct baseline) |
| Transitions: safe → delayed refusal | **14** |
| Transitions: delayed refusal → safe | **0** (one-way effect) |
| Monitor trigger precision | **77.8%** |
| Utility drop on benign traffic | **~0%** |
| Latency overhead from monitor | **5.8%** |

> **Current status: Pre-Confirmatory / Pipeline Validated.**
> The directional signal is real, but a larger run is required before making a claim-level result. The benchmark pipeline works end-to-end. It correctly blocks over-claiming through its own quality gates — which is itself part of the design.

---

## Execution Plan — Three Steps to a Publishable Result

**Step 1 — Scale the run.** Expand from 70 to 180+ matched conversation pairs across all six models in the panel. Complete the 18 seed-model combinations currently sitting at 6/18. This directly resolves the main statistical blockers identified in the pilot.

**Step 2 — Pass the quality gates.** Bring judge agreement above the required 0.80 threshold (pilot result: 0.12–0.38). Add human audit labels to the required 10% of rows, stratified by model, family, and phase. These gates exist to prevent over-claiming — passing them is what makes the result publishable and reviewer-resistant.

**Step 3 — Publish the benchmark and paper.** Release the full reusable benchmark artifact (dataset schema, trajectory templates, metrics, and decision thresholds) alongside a paper documenting the timing effect and the monitor trade-offs. Any AI lab will be able to run this independently on their own models.

---

## Repository Structure

```
temporal-refusal-timing-pilot/
├── temporal_alignment_hysteresis_experiment.ipynb   # Main experiment notebook
├── temporal_alignment_research_plan.txt             # Research plan and evidence ladder
├── temporal_alignment_prompt_set.csv                # Prompt source
├── datasets/                                        # Core dataset files
├── benchmark_outputs/                               # Generated run artifacts
├── reports/                                         # Run reports
└── ideas-rp/                                        # Source references for protocol grounding
```

---

*Pilot run: `benchmark_outputs/pilotrun3` · Stage: pipeline-validation, not claim-positive*
