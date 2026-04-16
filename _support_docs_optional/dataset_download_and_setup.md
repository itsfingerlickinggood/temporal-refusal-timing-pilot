# Dataset Download and Setup (No Content Inspection)

This project can run claim-ready only when harmful prompts are sourced from:
- advbench
- harmbench
- jbb-behaviors

This setup uses official online files and does not print dataset rows.

## 1) Run the preparation script

From workspace root:

```powershell
D:/ai-safety-research/1/.venv/Scripts/python.exe scripts/prepare_claim_ready_datasets.py
```

## 2) Files created

The script writes these files:

- datasets/raw/advbench_harmful_behaviors_raw.csv
- datasets/raw/harmbench_behaviors_text_test_raw.csv
- datasets/raw/jbb_harmful_behaviors_raw.csv
- datasets/raw/jbb_benign_behaviors_raw.csv
- datasets/advbench_harmful_behaviors.csv
- datasets/harmbench_standard_malicious.csv
- datasets/jbb_behaviors.csv
- datasets/benign_controls.csv
- temporal_alignment_prompt_set.csv
- datasets/provenance_manifest.json

## 3) Official download sources used

- https://raw.githubusercontent.com/llm-attacks/llm-attacks/main/data/advbench/harmful_behaviors.csv
- https://raw.githubusercontent.com/llm-attacks/llm-attacks/main/data/advbench/harmful_strings.csv
- https://raw.githubusercontent.com/centerforaisafety/HarmBench/main/data/behavior_datasets/harmbench_behaviors_text_all.csv
- https://raw.githubusercontent.com/centerforaisafety/HarmBench/main/data/behavior_datasets/harmbench_behaviors_text_test.csv
- https://huggingface.co/datasets/JailbreakBench/JBB-Behaviors/resolve/main/data/harmful-behaviors.csv
- https://huggingface.co/datasets/JailbreakBench/JBB-Behaviors/resolve/main/data/benign-behaviors.csv

Note:
- The script currently uses HarmBench text_test for the normalized HarmBench file.
- The strings/all links above are included for reproducibility and optional extensions.

## 4) Use in the notebook

Open temporal_alignment_hysteresis_experiment.ipynb and run from Cell 1.

Cell 4 will automatically pick temporal_alignment_prompt_set.csv first and enforce source-policy checks.

## 5) Quick policy check

The run is claim-ready only if:
- OFFLINE_MOCK_MODE is false
- prompt source resolves to project_prompt_set
- harmful rows are tagged only with advbench, harmbench, jbb-behaviors
- harmful row count is at least the configured minimum
