# Version History

Each prompt version is saved as a full file in `prompts/vX.md`. The table below tracks scores, status, and a brief summary.

| Version | Iteration | Timestamp | Prompt | Train Composite | Eval Composite | Status | Parent | Changes | Notes |
|---------|-----------|-----------|--------|-----------------|----------------|--------|--------|---------|-------|
| v0 | 0 | — | [v0.md](prompts/v0.md) — initial prompt from agent | — | — | baseline | — | — | Starting point before optimization |
| v1 | 1 | 2024-01-15 10:30 | [v1.md](prompts/v1.md) — added CO-STAR structure | 0.68 | — | improving | v0 | Applied CO-STAR framework, added role definition and constraints | First structured version |
| v2 | 2 | 2024-01-15 11:15 | [v2.md](prompts/v2.md) — strengthened safety rules | 0.74 | — | improving | v1 | Added explicit safety instructions and refusal patterns | Safety +0.15, task_completion stable |
| v3 | 3 | 2024-01-15 12:00 | [v3.md](prompts/v3.md) — added output format examples | 0.71 | — | regressing | v2 | Added few-shot examples for output formatting | format +0.10, but task_completion -0.08. Regression #1 |

_The rows above are examples. Delete them and start from v0 when using this framework._
