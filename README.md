# PROPTER — PRompt OPTimizER

A generic, reusable framework for **iterative system prompt optimization** of any AI agent, powered by Claude Code and MCP tools.

No scripts. No custom code. Just Claude Code reading structured docs, calling your agent via MCP, scoring responses with G-Eval, and improving the prompt — in a loop.

## Who is this for?

- Developers building AI agents (chatbots, assistants, tool-using agents)
- Teams who want systematic, measurable prompt improvement
- Anyone tired of manual prompt tweaking with no tracking

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and configured
- MCP tools connected to your agent (e.g., n8n workflows, HTTP endpoints, Supabase)
- An AI agent with a system prompt you want to optimize

## Quick Start

1. **Copy** the `prompt-optimizer/` folder into your project
2. **Edit `integrations.md`** — tell the framework how to call your agent and where prompts are stored
3. **Edit `test_book/functional_tests.json`** — replace `{{PLACEHOLDER}}` values with your agent's specifics (see the `_guide` object in the file)
4. **Edit `test_book/nonfunctional_tests.json`** — add security/adversarial tests (defaults work for most agents)
5. **Run**: open Claude Code in this folder and say _"Read instructions.md and start optimizing"_

Claude Code will first **validate your integration** (Step 0), then loop autonomously: test → evaluate → improve → repeat — until the stop target is reached. Every prompt version is saved to `prompts/` for full traceability.

## File Structure

```
propter/
├── CLAUDE.md                        # Entry point for Claude Code
├── instructions.md                  # THE LOOP — the core optimization cycle
├── integrations.md                  # How Claude Code connects to your agent
├── evaluation_task.md               # G-Eval rubrics and per-test scoring
├── evaluation_run.md                # Weights, composite score, stop target
├── improvements.md                  # Prompt improvement techniques
├── pruning.md                       # Backtracking and pruning rules
├── test_book/
│   ├── functional_tests.json        # Does the agent do what it should?
│   └── nonfunctional_tests.json     # Security, adversarial, edge cases
├── test_runs/
│   └── run_template.json            # Template for recording test results
├── prompts/
│   └── v0.md                        # Each prompt version saved as a file
├── version_history.md               # Version tracker: prompt + scores + status
└── README.md                        # This file
```

### Key files explained

| File | Purpose |
|------|---------|
| `instructions.md` | The brain. Defines the full optimize loop Claude Code follows. |
| `integrations.md` | The bridge. Tells Claude Code how to reach your agent and update prompts. |
| `evaluation_task.md` | The judge. G-Eval rubrics for scoring each individual test response. |
| `evaluation_run.md` | The scoreboard. Aggregates scores, defines the stop target. |
| `improvements.md` | The playbook. Techniques for generating better prompt variants. |
| `pruning.md` | The safety net. Reverts when things go wrong. |
| `prompts/` | The archive. Full text of every prompt version (v0.md, v1.md, ...). |
| `version_history.md` | The log. Full history of every prompt version and its scores. |

## How to Customize

### For your agent
1. Update `integrations.md` with your agent's endpoint, body format, and prompt storage location
2. Write tests in `test_book/` that match your agent's real-world use cases
3. Adjust dimension weights in `evaluation_run.md` (e.g., increase `safety` weight for medical agents)
4. Set your stop target in `evaluation_run.md` (default: 0.95)

### For your evaluation
- Add or remove scoring dimensions in `evaluation_task.md`
- Change test/eval split ratio in `evaluation_run.md`
- Customize improvement strategies in `improvements.md`

## Limitations & Adapting

| Limitation | Workaround |
|------------|------------|
| **Single-turn only** | For multi-turn agents, test individual turns or create multi-message test sequences |
| **No tool inspection** | If your agent uses tools internally and you can't inspect them, remove `tool_usage` from scoring dimensions |
| **Self-eval bias** | The same LLM family judges the output — consider external validation for critical systems |
| **Streaming/async agents** | Ensure your integration returns the full response, not a stream — use webhook callbacks if needed |
| **Prompt-only optimization** | This framework optimizes the system prompt, not tools, RAG, or model parameters |

## Credits

- **G-Eval** — Liu et al., 2023. LLM-based evaluation with chain-of-thought scoring.
- **OWASP LLM Top 10** — Security test patterns for LLM applications.
- **CO-STAR, RISEN, TIDD-EC, RODES** — Prompt engineering frameworks referenced in `improvements.md`.
- Presented at **aiTinkerers** meetup, 2026.

## License

MIT
