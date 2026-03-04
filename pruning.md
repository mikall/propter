# Pruning Rules & Backtracking

When prompt changes make things worse, this file defines when and how to backtrack.

---

## What is Pruning?

Pruning means reverting to a previous prompt version and choosing a different improvement path. It prevents wasting iterations going deeper into a failing approach.

## When to Prune

**Trigger: N consecutive regressions** (default: 3)

A regression occurs when the current run's composite score is lower than the previous run's composite score.

Track consecutive regressions:
- Regression → counter +1
- Improvement or no change → counter resets to 0
- When counter reaches N → trigger pruning

## How to Prune

### Step 1 — Revert

Go back to the **last improving version** (the version before the regression streak began).

- Find it in `version_history.md` — look for the last row with status "improving"
- Read the full prompt from `prompts/vX.md` (where X = that version number)
- Update the prompt via `integrations.md` to this version

### Step 2 — Mark History

In `version_history.md`:
- Mark the reverted-to version as the new parent
- Mark the pruned versions with status "pruned"
- Add a note explaining why pruning was triggered

### Step 3 — Choose a New Branch

The whole point of pruning is to try something **different**. Don't repeat the same approach.

Choose a different direction:

| What to change | Example |
|----------------|---------|
| **Different framework** | If you used CO-STAR, try TIDD-EC |
| **Different strategy** | If you used worst-N, try highest-weight |
| **Different focus area** | If you focused on safety, focus on task_completion |
| **Different technique** | If you added constraints, try adding examples instead |
| **Structural change** | If you edited sections, try restructuring the whole prompt |

### Step 4 — Continue

Generate a new prompt variant using the new approach and continue the loop from Step 3 in `instructions.md`.

---

## Concrete Example

```
Version history:
  v1 (baseline)       → composite: 0.65
  v2 (CO-STAR applied) → composite: 0.72  ✓ improving
  v3 (added safety)    → composite: 0.70  ✗ regression #1
  v4 (more safety)     → composite: 0.68  ✗ regression #2
  v5 (safety examples) → composite: 0.66  ✗ regression #3 → PRUNE

Pruning action:
  1. Revert to v2 (last improving version)
  2. Mark v3, v4, v5 as "pruned"
  3. New branch: switch from CO-STAR to TIDD-EC, focus on task_completion instead of safety
  4. Generate v6 from v2 using the new approach
```

---

## Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `consecutive_regressions_threshold` | 3 | Number of consecutive regressions before pruning |

To change the threshold, edit the value above. Lower values (2) are more aggressive — prune faster. Higher values (4-5) give more room to experiment but risk wasting iterations.
