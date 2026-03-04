Read `instructions.md` to understand the full optimization loop and begin working.

## Permissions

- **Prompt updates** (via `integrations.md`): autonomous — update the system prompt without asking.
- **Test execution**: autonomous — call the agent and record results.
- **Test results & version history**: autonomous — write to `test_runs/` and `version_history.md`.
- **Workflow / infrastructure changes**: NEVER touch without explicit user approval.
- **Deploy to production**: ALWAYS ask the user before deploying a final prompt.

## Core Rules

- **No scripts.** Execute everything using MCP tools connected to Claude Code.
- **Always use `test_runs/run_template.json`** as the template when saving test run results.
- **Track every version** in `version_history.md` — no prompt change goes unrecorded.
- **Stop when the stop target is reached** (defined in `evaluation_run.md`), then propose deploy to the user.
