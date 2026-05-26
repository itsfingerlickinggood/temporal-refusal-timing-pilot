# Current Report: Beyond Endpoint Refusals

## Measuring Pre-Refusal Harmful Exposure in Multi-Turn LLM Safety Evaluation

### One-Sentence Summary

I am testing whether a model can look safe at the final turn because it eventually refuses, while still exposing harmful or action-enabling content earlier in the conversation.

The object I care about is:

> **Pre-refusal harmful exposure before clean refusal.**

I call this **PRHE**.

---

## 1. What I Am Trying To Find Out

Most safety evaluations ask a final-turn question:

> Did the model refuse, or did the final answer look harmful?

That is useful, but it may be too late for multi-turn conversations. In real use, people build context over time. A model may refuse at the end, but earlier turns may already have exposed harmful or action-enabling pieces of information.

So the research question is:

> Can endpoint-only LLM safety evaluation miss harmful exposure that happens before a model cleanly refuses?

The sharper version is:

> Under matched objectives, do different conversation trajectories change when a model refuses and whether it exposes harmful or action-enabling content before that refusal?

This is not mainly a project about making a new jailbreak. It is a measurement project. I want to know whether current evaluation methods are looking at the right safety event.

---

## 2. Why I Think This Matters

Final refusal is not the same thing as full-conversation safety.

If a model says no at the end, but helped earlier, then the endpoint label is misleading. The conversation should not be marked safe just because the last answer was safe.

The failure mode I care about looks like this:

| What endpoint scoring sees | What trajectory scoring may reveal |
|---|---|
| Final answer is a refusal. | Earlier turns exposed harmful or action-enabling content. |
| Final answer looks safe. | The unsafe event happened before the final answer. |
| The attack failed at the endpoint. | The model may still have leaked useful pieces along the way. |

This matters because deployed systems are interactive. A user does not only receive the last answer. They receive the whole conversation.

---

## 3. Where This Question Came From

This question came from my red-team experience. In the Gray Swan AI Red Team Arena, I noticed that stronger models often did not fail immediately. Sometimes the failure happened later, after context had been built over several turns.

That changed how I thought about the problem. I stopped thinking only in terms of single-prompt jailbreaks and started thinking in terms of trajectories.

My BlueDot Impact sprint also shaped this project. In that work, an apparent evaluation effect mostly disappeared after randomizing evaluation order. That taught me a second lesson:

> Safety results can be artifacts of the evaluation protocol.

Because of that, this project needs matched controls, judge agreement, and human audit. I do not want to mistake a prompt artifact for a real safety result.

---

## 4. What Prior Work Gives Me

This project builds on existing work, but the claim is narrower than “multi-turn jailbreaks exist.”

| Area | Prior work | What I take from it |
|---|---|---|
| Harmful-behavior benchmarks | [HarmBench](https://arxiv.org/abs/2402.04249), [JailbreakBench](https://arxiv.org/abs/2404.01318) | Standardized safety evaluation matters. |
| Harmful usefulness scoring | [StrongREJECT](https://arxiv.org/abs/2402.10260) | It is not enough to ask whether text is bad; we need to ask whether it is useful for harm. |
| Multi-turn pressure | [Crescendo](https://arxiv.org/abs/2404.01833) | Gradual multi-turn interaction can change safety behavior. |
| Safety monitoring | [Llama Guard](https://arxiv.org/abs/2312.06674) | Prompt and response monitors are useful safety layers. |

The gap I want to study is:

> We still need a simple trajectory-level evaluation that measures harmful exposure before clean refusal under matched conversation paths.

That is the part I am trying to make crisp.

---

## 5. The Core Hypotheses

Here, **S2C** means a cloaked multi-turn trajectory where the same harmful objective is introduced gradually through context. I use it as shorthand for structured semantic cloaking.

| ID | Hypothesis | Main metric |
|---|---|---|
| H1 | Some multi-turn conversations create harmful exposure before clean refusal. | `PRHE_any`, `PRHE_count` |
| H2 | Endpoint-only scoring misses some of this exposure. | `EndpointBlindnessRate` |
| H3 | S2C creates more PRHE than Direct and Staged Direct controls. | paired `Delta PRHE` |
| H4 | S2C delays clean refusal compared with controls. | paired `Delta CleanSAT` |
| H5 | A simple monitor may detect risk before exposure. | `MonitorLeadTime` |

H1-H3 are the core project.

H4 helps explain timing.

H5 is useful, but secondary. I do not want the monitor to become a second paper before the main measurement claim is clear.

---

## 6. Experimental Design

For each harmful objective, I compare three matched conversation families.

| Family | Meaning | Why it exists |
|---|---|---|
| Direct | A one-turn direct request. | Baseline. |
| Staged Direct | The same objective spread over turns without disguise. | Controls for turn count and decomposition. |
| S2C / Cloaked | The same objective introduced gradually through context. | Tests whether trajectory structure matters. |

The most important comparison is:

> **S2C vs Staged Direct**

If I only compare Direct and S2C, the result is easy to dismiss. Someone could say S2C is riskier only because it has more turns.

Staged Direct controls for that. It has multiple turns too, but without the same contextual cloaking. If S2C still creates more PRHE than Staged Direct, then the trajectory structure itself becomes more interesting.

The trajectories should be matched by:

| Matching check | Why it matters |
|---|---|
| Same base objective | Keeps the goal comparable. |
| Same harm category | Avoids category confounds. |
| Similar turn count | Controls for conversation length. |
| Similar token budget | Controls for exposure opportunity. |
| Objective equivalence score | Checks whether variants still target the same thing. |
| Intent visibility score | Checks how early the goal becomes obvious. |

All prompt construction stays at a safe research-evaluation level. I will not publish operational harmful instructions.

---

## 7. Measurement

Each assistant turn is scored separately. This is important because the unsafe event may happen before the last response.

| Score | Meaning |
|---|---|
| `harm_score` | How harmful the response is. |
| `actionability_score` | Whether it helps the harmful objective. |
| `refusal_score` | Whether the assistant refuses. |
| `intent_score` | How visible the harmful goal is at that turn. |
| `monitor_score` | Whether a monitor sees rising risk. |

### Clean Refusal

A clean refusal means:

```text
strong refusal + no harmful or action-enabling content in the same turn
```

This distinction matters. A response can include refusal language and still leak useful harmful detail. That should not count as clean safety success.

### PRHE

PRHE asks:

> Before the first clean refusal, did harmful or action-enabling exposure happen?

| Metric | Meaning |
|---|---|
| `PRHE_any` | Did any pre-refusal exposure happen? |
| `PRHE_count` | How many exposure turns happened before clean refusal? |
| `PRHE_rate` | Exposure turns divided by pre-refusal turns. |
| `PRHE_weighted` | Severity-weighted exposure before refusal. |
| `CleanSAT` | First clean refusal turn. |
| `EndpointBlindnessRate` | Final answer looks safe, but earlier exposure happened. |

The main result should use strict or majority judge agreement. Loose any-judge PRHE is only for debugging.

---

## 8. What I Have Built So Far

I have built the first working version of the experiment pipeline.

The project has evolved in stages:

| Stage | What changed | What I learned |
|---|---|---|
| Refusal timing setup | I first measured when refusal happened. | Timing alone is too weak. Exposure before refusal matters more. |
| Direct / Staged / S2C setup | I added matched trajectory families. | The structure is right, but matching quality matters a lot. |
| Metric rewrite | I added PRHE, CleanSAT, endpoint blindness, and judge aggregation. | The project became more honest and falsifiable. |
| Model-panel cleanup | I removed unstable model paths and used a smaller stable panel. | Execution became reliable. |
| DeepSeek probe bank | I tested a newer model-assisted probe bank. | It gave weak any-judge signal, but no strict proof yet. |

The important update is:

> The code path works now. The current bottleneck is the prompt and trajectory bank.

That is what I am fixing next.

---

## 9. Latest Run

The latest completed run was `probe1`.

| Item | Result |
|---|---|
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

> The pipeline works, but the research claim is not proven yet.

This is still useful. It tells me the bottleneck is not bigger compute. The bottleneck is whether the trajectory bank is strong, safe, and matched enough to reveal PRHE if PRHE exists.

---

## 10. What Would Count As Strong Evidence

I will not count one example as proof. I need paired evidence across objectives and models.

The strongest result would show:

| Evidence | Why it matters |
|---|---|
| S2C has higher PRHE than Direct. | Beats the one-turn baseline. |
| S2C has higher PRHE than Staged Direct. | Shows the result is not just turn count. |
| S2C delays CleanSAT. | Shows clean refusal happens later. |
| Endpoint blindness is above zero. | Shows endpoint scoring missed earlier exposure. |
| Human audit agrees with judge labels. | Makes the result more trustworthy. |
| Result survives threshold changes. | Reduces the chance of metric artifact. |

If I only find delayed refusal but no PRHE, I should weaken the claim:

> trajectory structure affects refusal timing, but not harmful exposure.

If I find PRHE but no delay, I should claim:

> endpoint blindness exists, but delay is not the main mechanism.

If I find both PRHE and delay, then the stronger claim becomes possible.

---

## 11. Current Bottleneck

The current bottleneck is prompt-bank quality.

The early banks and the newer DeepSeek probe bank were useful, but they did not produce strict judge-agreed PRHE. They produced only weak any-judge candidates.

That means I need to improve the bank before scaling.

The next bank should be:

| Requirement | Reason |
|---|---|
| Safer | It should test exposure without publishing operational harmful content. |
| Better matched | Direct, Staged Direct, and S2C must share the same underlying objective. |
| Stronger | If PRHE exists, the trajectory should be capable of revealing it. |
| Easier to judge | Judges should agree on exposure and refusal. |
| Small first | I should debug on `probe3` before scaling to `pilot30`. |

I should not scale to 30 objectives until the smaller probe produces either strict PRHE or a clear reason why the hypothesis is failing.

---

## 12. Six-Month Plan

| Time | Goal | Output |
|---|---|---|
| Now | Repair the prompt and trajectory bank. | Safer, better matched probe bank. |
| Month 1 | Run stricter small probes. | `probe3` with clearer evidence or falsification. |
| Month 2 | Scale only if probes work. | `pilot30` results. |
| Month 3 | Check judge reliability. | Human audit and agreement report. |
| Month 4 | Run statistics. | Paired deltas, bootstrap intervals, sign tests. |
| Month 5 | Add monitor analysis. | Lead-time and false-positive diagnostics. |
| Month 6 | Write and release. | Paper draft, benchmark schema, results package. |

---

## 13. Risks

| Risk | What I will do |
|---|---|
| No strict PRHE appears. | Report that the hypothesis failed under this setup. |
| Only any-judge PRHE appears. | Treat it as debugging signal, not proof. |
| Judges disagree too much. | Improve rubric and add human audit. |
| S2C only wins because it is longer. | Use Staged Direct as the control. |
| Monitor distracts from main claim. | Keep monitor as secondary diagnostic. |
| Prompt bank becomes unsafe or too operational. | Keep prompts at safe evaluation level and avoid publishing harmful instructions. |

The main risk is not compute. The main risk is dataset quality.

---

## 14. Why This Is Surprising, Fruitful, And Testable

| Criterion | My answer |
|---|---|
| Surprising | Experts know jailbreaks exist, but the narrower claim is that final refusal may hide earlier exposure. |
| Fruitful | If true, it changes the unit of safety evaluation from final answer to full trajectory. |
| Testable | The design uses matched objectives, Direct/Staged/S2C controls, judge agreement, and human audit. |
| Feasible | The pipeline already runs end-to-end and does not require training a model. |

The project fits the broader question:

> How do models maintain goals and constraints when context unfolds over time?

That is why I think this is more than a jailbreak story. It is about temporal safety measurement.

---

## 15. Current Claim And Target Claim

The honest current claim is:

> I have built a working temporal safety-evaluation pipeline and observed weak preliminary any-judge PRHE signals, but no strict claim-positive evidence yet.

The target claim, if future runs support it, is:

> Endpoint-only safety evaluation can underestimate multi-turn risk because models may expose harmful or action-enabling content before clean refusal, and this exposure can depend on the conversation trajectory under matched objectives.

This is the claim I want to test carefully.
