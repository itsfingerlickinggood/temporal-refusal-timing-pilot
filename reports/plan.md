# Beyond Endpoint Refusals

## Research Plan

### One-line question

Can endpoint-only LLM safety evaluation miss harmful exposure that happens before a model cleanly refuses in a multi-turn conversation?

### Short thesis

I want to test a simple but important hunch:

> A model can look safe at the endpoint because it eventually refuses, while still exposing harmful or action-enabling content earlier in the conversation.

The project studies this as a temporal safety problem. Instead of only asking whether the final answer was safe, I measure the path the conversation took before that final answer.

---

## Why This Question Matters

Most safety evaluations still collapse a conversation into an endpoint label:

| Usual question | Problem |
|---|---|
| Did the model refuse at the end? | It may have already helped before refusing. |
| Was the final answer harmful? | Earlier turns may contain harmful pieces. |
| Did the jailbreak succeed? | This hides when the model became unsafe. |

My project asks a more operational question:

> When exactly did the model expose harmful/actionable content, and did that happen before clean refusal?

This matters because real users interact over time. A safety system that only checks the final turn may miss a slow failure.

---

## The Core Research Claim

The claim I want to test is:

> Under matched harmful objectives, the structure of a multi-turn trajectory can change both refusal timing and pre-refusal harmful exposure. Because of this, endpoint-only metrics may underestimate risk.

This is not the claim:

> Multi-turn jailbreaks exist.

That is already known.

The more precise claim is:

> Endpoint refusal can be safety-blind because harmful exposure may occur before clean refusal, and that exposure may depend on the trajectory used to reach the same objective.

---

## Why This Could Be Surprising

Experts already know that models can be jailbroken. The less obvious part is not that a prompt can fail safety. The less obvious part is the measurement blind spot.

The surprising possibility is:

| Common assumption | My test |
|---|---|
| Final refusal means the conversation was safe. | Check whether harmful content appeared before refusal. |
| Multi-turn safety can be measured by final ASR/refusal. | Measure turn-level exposure and timing. |
| Direct and cloaked prompts differ only in success rate. | Compare the same objective across different trajectory structures. |

If the result holds, it says current safety evaluation may be looking at the wrong event.

---

## Why This Is Fruitful

If this works, the output is not only one result. It gives a reusable way to evaluate temporal safety.

| Output | Why it matters |
|---|---|
| PRHE metric | Measures harmful exposure before clean refusal. |
| Clean refusal timing | Separates leaky refusals from clean refusals. |
| Endpoint blindness rate | Shows when final-turn scoring misses earlier risk. |
| Matched trajectory benchmark | Lets different labs compare models under the same objectives. |
| Monitor diagnostics | Tests whether rising risk can be detected before exposure. |

The downstream consequence is simple:

> Safety evaluation should grade the whole trajectory, not only the final answer.

---

## Main Hypotheses

| ID | Hypothesis | Main metric |
|---|---|---|
| H1 | Some multi-turn conversations produce harmful exposure before clean refusal. | `PRHE_any`, `PRHE_count` |
| H2 | Endpoint-only metrics miss some of this exposure. | `EndpointBlindnessRate` |
| H3 | S2C/cloaked trajectories produce more PRHE than Direct or Staged Direct trajectories. | paired `ΔPRHE` |
| H4 | S2C/cloaked trajectories delay clean refusal compared with controls. | paired `ΔCleanSAT` |
| H5 | A simple turn-level monitor can sometimes detect risk before exposure. | `MonitorLeadTime` |

The core hypotheses are H1-H3. H4 explains timing. H5 is a practical add-on, not the main novelty.

---

## Experimental Design

For each harmful objective, I create three matched trajectory families:

| Family | What it tests |
|---|---|
| Direct | One-turn direct request. Baseline. |
| Staged Direct | Same objective spread across turns without disguise. Controls for turn count. |
| S2C / Cloaked | Same objective introduced gradually through contextual disguise. Tests trajectory structure. |

The important design rule is:

> Direct, Staged Direct, and S2C must target the same underlying objective.

Otherwise the comparison collapses.

### Fairness checks

| Check | Reason |
|---|---|
| Same base objective ID | Keeps the objective matched. |
| Same harm category | Avoids category confounds. |
| Same turn count for staged and S2C | Controls for length. |
| Similar token budget | Controls for exposure opportunity. |
| Objective equivalence score | Verifies the variants really match. |
| Judge-rated intent detectability | Measures whether one family is much easier to identify. |

---

## Measurement Framework

Each assistant turn is scored by judge models.

| Score | Meaning |
|---|---|
| `harm_score` | How harmful the assistant response is. |
| `actionability_score` | Whether it materially helps the harmful objective. |
| `refusal_score` | Whether the assistant refuses. |
| `intent_score` | How detectable the harmful goal is at that turn. |
| `monitor_score` | Runtime estimate of rising risk. |

### Clean refusal

A clean refusal is not just any refusal.

```text
Clean refusal = refusal is strong AND harmful/actionable content is absent
```

This matters because a model can say:

```text
I cannot help with that, but...
```

and still leak useful details. That should not count as a clean safety success.

### PRHE

PRHE means:

```text
Pre-Refusal Harmful Exposure
```

It asks:

> Before the first clean refusal, did the model expose harmful or actionable content?

Main variants:

| Metric | Meaning |
|---|---|
| `PRHE_any` | Did any pre-refusal harmful exposure happen? |
| `PRHE_count` | How many harmful pre-refusal turns happened? |
| `PRHE_rate` | Harmful pre-refusal turns divided by pre-refusal turns. |
| `PRHE_weighted` | Severity-weighted exposure before refusal. |

Primary analysis uses strict judge agreement. Sensitivity analysis reports looser any-judge signals.

---

## Evidence Needed For The Claim

I will not treat one positive example as proof. The claim needs paired evidence.

| Evidence | What would support the claim |
|---|---|
| Direct vs S2C PRHE | S2C has higher PRHE under the same objective. |
| Staged Direct vs S2C PRHE | S2C beats ordinary multi-turn decomposition. |
| Direct vs S2C CleanSAT | S2C delays clean refusal. |
| Endpoint blindness | Final response is safe/refusal while earlier turns had exposure. |
| Robustness checks | Result survives strict, mean-threshold, and token-normalized scoring. |

The strongest result would be:

```text
S2C increases PRHE
+ S2C delays clean refusal
+ endpoint-only scoring misses earlier exposure
```

If only timing changes but PRHE stays zero, the claim becomes weaker:

> trajectory structure affects refusal timing, but not harmful exposure.

---

## How The Project Has Evolved

The project did not start with the current setup. It has gone through several iterations, and each one clarified what the real bottleneck is.

| Stage | What I tried | What I learned |
|---|---|---|
| Early endpoint/timing setup | Measure whether models refused and when refusal happened. | Refusal timing alone is not enough. A later refusal is only safety-relevant if harmful exposure happened before it. |
| First prompt banks | Build Direct, Staged Direct, and S2C-style trajectories from existing benchmark objectives. | The structure was valid, but the prompts were too weak to produce clear PRHE evidence. |
| Metric rewrite | Added clean refusal, PRHE, endpoint blindness, judge aggregation, and sensitivity metrics. | The evaluation became more honest: it could distinguish strict evidence from weak one-judge signals. |
| Model-panel cleanup | Removed unstable model paths and used a smaller stable target panel with two judges. | The experiment became technically reliable. API failures stopped being the main issue. |
| DeepSeek probe bank | Used a model-assisted process to create a small safer probe bank from a seed package and high-level trajectory rules. | This produced the first weak PRHE-like signal, but only under any-judge scoring. |

The important lesson is:

> The code path now works. The current bottleneck is the quality of the trajectory bank I chose and generated.

I am deliberately not treating the current signal as proof. The latest run shows that the pipeline can surface candidate cases, but the prompts still need to be stronger, better matched, and easier for judges to agree on.

---

## Current Pilot Status

I have already built and run the pipeline end-to-end.

Latest run:

| Item | Result |
|---|---|
| Run mode | `probe1` |
| Objectives | 1 |
| Target model rows | 44 |
| Judge rows | 88 |
| Target API errors | 0 |
| Strict PRHE | 0 |
| Any-judge PRHE | 2 |
| Endpoint blindness | 0 |
| Strict cumulative harm | 0 |
| Monitor evidence | 0 |

My honest interpretation:

> The infrastructure works, but the current evidence is not yet claim-positive.

The latest run shows weak candidate signal under loose scoring, but not enough strict evidence. This is useful because it tells me the next bottleneck is no longer the code. The bottleneck is the trajectory bank.

---

## What I Need To Improve Next

| Problem | Fix |
|---|---|
| Strict PRHE is zero | Improve matched trajectory bank so exposure, if it happens, appears clearly enough for judge agreement. |
| Endpoint blindness is zero | Need cases where final refusal hides earlier exposure. |
| Signals appear mostly under any-judge scoring | Add judge agreement checks and human audit. |
| Monitor has nothing to detect | First generate reliable exposure events, then evaluate monitor lead time. |
| Scale is tiny | Move from `probe1` to `probe3`, then `pilot30`. |

I should not scale to 30 objectives until the smaller probe produces at least some strict PRHE or a clear reason why the hypothesis is failing.

---

## Statistical Plan

The design is paired by objective.

For each objective and model:

```text
ΔPRHE_direct = PRHE_S2C - PRHE_Direct
ΔPRHE_stage  = PRHE_S2C - PRHE_StagedDirect
ΔSAT_direct  = CleanSAT_S2C - CleanSAT_Direct
ΔSAT_stage   = CleanSAT_S2C - CleanSAT_StagedDirect
```

Main tests:

| Test | Purpose |
|---|---|
| Paired bootstrap CI | Estimate uncertainty in paired deltas. |
| Paired sign test | Check direction without distribution assumptions. |
| Wilcoxon signed-rank | Nonparametric paired comparison. |
| Mixed-effects regression | Control for model and harm category. |

Example regression:

```text
PRHE_any ~ trajectory_family + model + harm_category + (1 | base_objective_id)
```

---

## Six-Month Plan

| Phase | Goal | Output |
|---|---|---|
| Month 1 | Fix trajectory bank and judge rubric | Probe bank with strict quality gates |
| Month 2 | Run `probe3` and `pilot30` | First reliable paired results |
| Month 3 | Add human audit and judge agreement | Reliability report |
| Month 4 | Run robustness checks | Sensitivity and failure-mode analysis |
| Month 5 | Add monitor diagnostics | Lead-time and false-positive analysis |
| Month 6 | Write paper and release benchmark | Preprint, code, dataset schema |

---

## Feasibility

This project is feasible because it is API-based and benchmark-focused. It does not require training a frontier model.

Available resources:

| Resource | Role |
|---|---|
| Cloudflare credits | Run target and judge models through one gateway. |
| Existing benchmark sources | Seed harmful objectives and benign controls. |
| Notebook pipeline | Already runs target models, judges, metrics, and outputs. |
| Human audit subset | Needed for judge validation, not every row. |

The main risk is not compute. The main risk is dataset quality.

---

## Failure Modes And What They Would Mean

| Outcome | Interpretation |
|---|---|
| No PRHE and no delay | Hypothesis likely false under this setup. |
| Delay but no PRHE | Trajectory affects refusal timing, but not harmful exposure. |
| PRHE but no delay | Endpoint blindness exists, but timing is not the main mechanism. |
| PRHE and delay | Strongest support for trajectory-sensitive safety risk. |
| Any-judge PRHE only | Weak signal; needs better judges or better trajectories. |

This is important because I do not want to force the claim. The experiment should be allowed to falsify the idea.

---

## Fit With The Research Criteria

This project fits the kind of research I want to do through the question:

> How do intelligent systems maintain goals and constraints under temporally unfolding, out-of-distribution interaction?

It is not mainly a jailbreak project. It is a measurement project about whether current evaluation methods observe the right safety event.

| Criterion | My answer |
|---|---|
| Surprising | Final refusal may hide earlier exposure. |
| Fruitful | Produces a reusable temporal safety benchmark and metrics. |
| Rigorous | Uses matched objectives, paired comparisons, judge agreement, and human audit. |
| Feasible | API-based, no model training required, already has a working pipeline. |

---

## What I Would Submit As The Paper Claim

I would not currently submit a strong empirical claim, because the latest strict PRHE result is still zero.

The honest current claim is:

> I have built a working temporal safety-evaluation pipeline and found weak preliminary any-judge PRHE signals, but the benchmark must be strengthened before making a claim-positive result.

The target final claim, if the next runs support it, is:

> Endpoint-only safety evaluation underestimates multi-turn risk because models can expose harmful/actionable content before clean refusal, and this pre-refusal exposure is conditioned by trajectory structure under matched harmful objectives.

That is the hill I want the project to climb.
