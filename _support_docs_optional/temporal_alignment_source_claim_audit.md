# Source-to-Claim Audit Table

Local source folder reviewed: [ideas-rp](/d:/ai-safety-research/1/ideas-rp)

| Claim or manuscript use | Local source | Support type | Confidence | Keep / remove / verify | Notes |
| --- | --- | --- | --- | --- | --- |
| Multi-turn conversational structure can materially affect jailbreak success | `2601.02670v1.pdf` | direct | high | Keep | Supports multi-turn trajectory sensitivity and query-efficient multi-turn attack structure. |
| Context-congruent framing can relax safety constraints | `2601.20903v1.pdf` | direct | high | Keep | Useful for motivating context-sensitive trajectory effects. |
| Structured semantic cloaking can delay or weaken safety triggering | `2603.16192v1.pdf` | direct | high | Keep | This is the strongest local source for the chosen stress condition. |
| Optimization-based jailbreak methods exist in the broader attack landscape | `2603.03081v1.pdf` | indirect | medium | Keep only in background | Do not use this to support the temporal-monitoring contribution. |
| Decomposition-based stealth attacks exist as an appendix robustness family | `6261_Stealthy_Jailbreaking_Att.pdf` | indirect | medium | Keep only as optional appendix context | Not needed for the core story. |
| Alignment hysteresis is already established in prior work | none in local folder | none | low | Remove or verify | The paper should frame this as the project's tested hypothesis, not settled prior art. |
| Monitoring or introspection interventions already reduce hidden safety failures in this setting | none in local folder | none | low | Remove or verify | The local source set does not justify this statement. |
| The current notebook run empirically confirms the core claim | current run artifacts only | none for confirmation | low | Remove | Latest run is mock-mode with demo prompts and is explicitly not claim-ready. |
| The revised notebook supports full-pipeline execution, audit export, and source traceability | `temporal_alignment_hysteresis_experiment.ipynb`, latest run artifacts | direct artifact evidence | high | Keep | Method claim only, not an empirical outcome claim. |

## Safe Introduction Claims From The Current Source Set
- Multi-turn trajectory structure matters for LLM safety evaluation.
- Context construction and semantic obfuscation can change safety behavior.
- Structured semantic cloaking is an appropriate stress condition for a temporal safety study.

## Claims That Need Verification Before Submission
- prior monitoring success in the exact evaluation setting
- introspection-based mitigation effectiveness
- any statement implying that alignment hysteresis is already established in the literature
- any statement implying that the current workspace already contains confirmatory empirical results
