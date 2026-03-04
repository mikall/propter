# The Optimization Loop

This is the core cycle. Follow each step in order. Do not skip steps.

---

## Step 0 — Validate Integration (run once)

Before starting the loop, verify that the integration works end-to-end.

1. **Read `integrations.md`** — understand endpoint, body format, response parsing, prompt storage
2. **Test the agent call**: send a simple message (e.g., "Hello") to the agent endpoint
   - If the call fails → stop. Report the error to the user. Do not proceed.
   - If the call succeeds → verify you can parse the response field correctly
3. **Test tool inspection** (if `Available: yes` in integrations.md):
   - Verify you can retrieve `tools_used` and `latency_s` from the response or execution logs
   - If unavailable despite being marked `yes` → set to `no` and note that `tool_usage` dimension will be skipped
4. **Test prompt storage**:
   - Read the current prompt from storage → confirm it returns a non-empty string
   - Write it back unchanged → confirm the write succeeds
   - If read or write fails → stop. Report the error to the user.
5. **Save the initial prompt**: copy the current prompt to `prompts/v0.md`

If all checks pass, proceed to Step 1. This step runs only once per optimization session.

---

## Step 1 — Understand the Agent

Read `integrations.md` to learn:
- How to call the agent (endpoint, body format, headers)
- How to read the agent's response
- How to inspect tool usage and latency (if available)
- Where the system prompt is stored and how to update it

## Step 2 — Load Tests

Read both files in `test_book/`:
- `functional_tests.json` — positive tests: does the agent do what it should?
- `nonfunctional_tests.json` — security, adversarial, edge-case tests

Split tests into **train** and **eval** sets (default: 70% train / 30% eval, randomized). Use train for optimization, eval for final validation only.

## Step 3 — Execute Tests

For each test in the current split:
1. Send the test `message` to the agent (via `integrations.md`)
2. Collect: `response`, `latency_s`, and `tools_used` (if tool inspection is available — see Step 0)
3. If tool inspection is unavailable, set `tools_used` to `null` in results
4. Record raw results

## Step 4 — Score Each Response

Read `evaluation_task.md` for the G-Eval rubrics.

For each test response, score across all active dimensions (default 5, or 4 if tool inspection is unavailable):
- `task_completion` (0.0–1.0)
- `tool_usage` (0.0–1.0) — **skip if tool inspection unavailable**, redistribute weight per `evaluation_run.md`
- `response_quality` (0.0–1.0)
- `safety` (0.0–1.0)
- `format_compliance` (0.0–1.0)

Use chain-of-thought reasoning before assigning each score.

## Step 5 — Compute Run Score

Read `evaluation_run.md` for weights, composite formula, and the **stop target**.

1. Compute weighted composite score for each test
2. Compute aggregate scores for the run (averages across all tests)
3. Compare with previous run (if any): compute delta %

## Step 6 — Save Results

Save the full run to `test_runs/` using the template from `test_runs/run_template.json`.

File naming: `run_vX_iY.json` where X = prompt version, Y = iteration number.

## Step 7 — Update Version History

1. **Save the full prompt** to `prompts/vX.md` (where X = version number)
2. **Add a new row** to `version_history.md` with:
   - Version number, iteration, timestamp
   - Prompt summary (1-2 sentences) + link to `prompts/vX.md`
   - Train composite score, eval composite score (if evaluated)
   - Status (improving / regressing / pruned / deployed)
   - Parent version, changes made, notes

## Step 8 — Stop Check

Read the **stop target** from `evaluation_run.md`.

- If composite score on **train** ≥ stop target:
  1. Run the **eval** split to validate
  2. If eval composite ≥ stop target → **STOP**. Propose deploy to user.
  3. If eval composite < stop target → note overfitting, continue loop
- If not reached → continue to Step 9

## Step 9 — Decision

Compare current run with previous run:

### If improved (or first run):
1. Read `improvements.md`
2. Analyze which dimensions scored lowest
3. Select an improvement strategy (worst-N, highest-weight, biggest-drop)
4. Generate a new prompt variant targeting weak areas
5. Go to Step 10

### If worsened:
1. Read `pruning.md`
2. Check if this is the Nth consecutive regression (default threshold: 3)
3. If threshold reached → prune: revert to last improving version, change approach
4. If not yet → try a smaller, more targeted improvement
5. Go to Step 10

## Step 10 — Update Prompt

1. Save the new prompt to `prompts/vX.md` (where X = new version number)
2. Write the new prompt to the storage location defined in `integrations.md`
3. Verify the update was successful (read back if possible)

## Step 11 — Repeat

Go back to **Step 3** with the new prompt.

---

## Important Rules

- **Never skip the eval split** when the stop target is reached on train.
- **Always save results** before making changes — no data loss.
- **Track everything** in `version_history.md`.
- **Ask the user** before deploying to production.
- If an integration call fails, retry once. If it fails again, stop and report the error.
