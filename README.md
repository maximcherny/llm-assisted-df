# LLM-Assisted Digital Forensic Investigations of Prompt Injection Attacks: Evidence Analysis and Representation

This repository contains the datasets and prompts used to facilitate the experiments described in the "LLM-Assisted Digital Forensic Investigations of Prompt Injection Attacks: Evidence Analysis and Representation" paper.

## Datasets

### Experiment 1

Download link:

```
https://1drv.ms/u/c/4a6d45fe01e1cbb3/EcCHjAyebGNAtPl52MpMwucBVyw8PZZS_H2XDrkBMxUhRA?e=u1sms2
```

Dataset structure:

```
llm-assisted-df-dataset/
├── banking_12_attack/          # Banking suite injected tasks
├── banking_12_benign/          # Banking suite benign tasks
├── slack_12_attack/            # Slack suite injected tasks
├── slack_12_benign/            # Slack suite benign tasks
├── travel_12_attack/           # Travel suite injected tasks
├── travel_12_benign/           # Travel suite benign tasks
├── workspace_12_attack/        # Workspace suite injected tasks
├── workspace_12_benign/        # Workspace suite benign tasks
└── gpt-4o-2024-05-13/          # GPT-4o target model exeuction results for AgentDojo
    ├── agent_dojo_tasks.csv    # Task definitions and metadata
    ├── banking/                # Banking suite
    ├── combined.csv            # Tasks with summaries and run_id mappings
    ├── slack/                  # Slack suite
    ├── summary.csv             # Task summary statistics and metrics
    ├── travel/                 # Travel suite
    └── workspace/              # Workspace suite
```

The structure inside the `gpt-4o-2024-05-13` subfolder contains standard [AgentDojo](https://github.com/ethz-spylab/agentdojo) output with additional custom metadata CSV files.

Each suite and phase directory (banking_12_attack, slack_12_attack, etc.) follows this structure:

```
[suite]_12_[attack|benign]/
├── ground_truth.jsonl          # Ground truth labels and injection metadata
├── llm.jsonl                   # LLM invocation logs for agent tasks containing experiment inputs, tasks distinguished by "run_id"
├── metadata.json               # Dataset metadata
└── results/                    # Model evaluation results organized by configuration
    ├── default-0.0-agent_cot_2/           # Temperature 0.0
    │   ├── [llm-1]-active-analysis.json   # Experiment outputs for evaluated LLMs
    │   ├── [llm-2]-active-analysis.json
    │   └── ...
    ├── default-0.0-agent_cot_2-anti/      # Temperature 0.0 with anti-forensics variant (not present for benign tasks)
    ├── default-1.0-agent_cot_2/           # Temperature 1.0
    └── default-1.0-agent_cot_2-anti/      # Temperature 1.0 with anti-forensics variant (not present for benign tasks)
```

### Experiment 2

Download link:

```
https://1drv.ms/u/c/4a6d45fe01e1cbb3/EVlxaFFbpdVIiZh2Cm21U1kBof3gXR0MEKo9LnfO1GsYqw?e=It9TkH
```

Dataset structure:

```
[scenario]_12_simple/
├── 0.0/                        # Temperature 0.0 results
│   ├── [llm-1]-case-01.json
│   ├── [llm-1]-case-02.json
│   ├── ...
│   ├── [llm-2]-case-01.json
│   ├── [llm-2]-case-01.json
│   ├── ...
└── 1.0/                        # Temperature 1.0 results
    └── [same pattern as 0.0/]
```

Each subfolder corresponds to individual scenarios evaluated in the paper.

## Prompts

### Experiment 1: LLM-assisted evidence analysis

**System prompt:** [Experiment 1 system prompt](prompts/experiment-1.txt)

**User prompt:** `Analyse the logs as per instructions and return JSON`

#### Prompt variables

**{log_window}:** the sequence of individual LLM invocation log entries instrumented as part of the end-to-end LLM agent task execution in the format of "[log_index]: log_message" separated by newlines

#### Example inputs

**{log_window}:** [Experiment 1 example {log_window}](examples/input-experiment-1.txt)

_Note:_ the end-to-end LLM agent task execution set of logs corresponds to all the logs associated with the same `run_id` (such as `642fbd84-0a7a-4002-a7dd-31f18112b1c9`) in a chronological LLM invocation order. The associated `[log_index]` value correspond to the respective line numbers from the dataset input file for the suite and task type (`llm.jsonl`).

#### Example outputs

[Experiment 1 example output](examples/output-experiment-1.txt)

_Note:_ the output contains the LLM chain-of-thought between the `<log_analysis>` tags, followed by the structured response encapsulated between the ` ```json` and ` ``` ` markers. During post-processing, the structured response is extracted from the overall response string resulting in:

```json
{
  "system_state": "INCIDENT",
  "summary": "Analysis revealed [omitted for brevity]...",
  "findings": [
    {
      "idx": 311,
      "confidence": 0.9,
      "reason": "...."
    },
    {
      "idx": 312,
      "confidence": 0.9,
      "reason": "..."
    }
  ]
}
```

The extracted structured outputs can then be used to calculate the specificity and recall metrics across benign and injected tasks, whilst also applying confidence threshold-based cutoffs.

### Experiment 2: LLM-assisted forensic evidence representation

System prompt: [Experiment 2 system prompt](prompts/experiment-2.txt)

**User prompt:** `Generate the CASE-Mapping-Python code as per the instructions`

#### Example inputs

#### Prompt variables

**{logs}:** the sequence of individual LLM invocation log entries instrumented as part of the end-to-end LLM agent task execution in the format of "[log_index]: log_message" separated by newlines. _This is the same input as used in Experiment 1._

**{analysis}:** the full output of from the previous forensic analysis and classification. _This is the output from Experiment 1 for the assiciated set of **{logs}** (also called **{log_window}** in Experiment 1). At this stage, we only use the outputs produced by `claude-3.7-sonnet` LLM at temperature `0.0` for the analysed scenarios._

**{case_mapping}:** the [CASE-Mapping-Python](https://github.com/casework/CASE-Mapping-Python) context containing the reqired object, facet and relationship definitions based on the published Python source code.

**{example_1}:** an annotated example of a CASE bundle from the `Example Usage` section of https://github.com/casework/CASE-Mapping-Python/blob/cccfcd817aa769402a43a59a128296c7bb94d8e5/README.md `tag: 0.1.0`

**{example_2}:** a source-code only example of a CASE bundle from https://github.com/casework/CASE-Mapping-Python/blob/cccfcd817aa769402a43a59a128296c7bb94d8e5/example.py `tag: 0.1.0`

#### Example inputs

**{logs}:** [Experiment 2 example {logs}](examples/input-experiment-1.txt)

**{analysis}:** [Experiment 2 example {analysis}](examples/output-experiment-1.txt)

**{case_mapping}:** produced using https://github.com/yamadashy/repomix `(v0.3.5)` as follows:

```
npx repomix@0.3.5 --include "case_mapping/*.py,case_mapping/case/*.py,case_mapping/uco/*.py"
```

The value of **{case_mapping}** is populated by passing the contents of the generated `repomix-output.xml` file (`md5 37300c009a81ad0e07431ad71ddc4f01`).

**{example_1}:** as described in `Prompt variables` above

**{example_2}:** as described in `Prompt variables` above

#### Example outputs

[Experiment 2 example output](examples/output-experiment-2.txt)

_Note:_ the output contains the LLM chain-of-thought between the `<case_bundle_planning>` tags, followed by the Python code for CASE bundle generation contained between the `<case_bundle_code>` tags. During graph generation and validation the Python code is extracted:

```python
from datetime import datetime
from case_mapping import uco, case

bundle = uco.core.Bundle(description="Prompt Injection Attack Analysis", name="Security Incident Report")

...

print(str(bundle))
```

#### Scenario reference

| Dataset folder | run_id |
|---|---|
| slack_12_attack | 642fbd84-0a7a-4002-a7dd-31f18112b1c9 |
| workspace_12_attack | 86269685-a9fb-4786-ada8-b9f613faa0af |
| banking_12_attack | dfb5f847-0661-4144-9907-8f341dbaf357 |

## License

Shield: [![CC BY 4.0][cc-by-shield]][cc-by]

This work is licensed under a
[Creative Commons Attribution 4.0 International License][cc-by].

[![CC BY 4.0][cc-by-image]][cc-by]

[cc-by]: http://creativecommons.org/licenses/by/4.0/
[cc-by-image]: https://i.creativecommons.org/l/by/4.0/88x31.png
[cc-by-shield]: https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg