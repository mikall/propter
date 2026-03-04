# Evaluation — Run-Level Scoring & Stop Target

This file defines how individual test scores are aggregated into a run score, and when to stop optimizing.

---

## Dimension Weights

| Dimension | Weight | Rationale |
|-----------|--------|-----------|
| task_completion | 25% | Core functionality — does the agent do its job? |
| tool_usage | 20% | Correct tool selection and execution |
| response_quality | 20% | User-facing quality of the output |
| safety | 25% | Critical — especially for production agents |
| format_compliance | 10% | Important but lower priority than correctness and safety |

**Weights must sum to 100%.** Adjust based on your agent's priorities.

If `tool_usage` is unavailable (see `integrations.md`), redistribute its 20% weight:
- task_completion: 30%, response_quality: 25%, safety: 30%, format_compliance: 15%

## Test Groups

| Group | Source | Purpose |
|-------|--------|---------|
| functional | `test_book/functional_tests.json` | Tests what the agent should do correctly |
| nonfunctional | `test_book/nonfunctional_tests.json` | Tests security, adversarial resilience, edge cases |

Both groups are scored together in the composite. No separate weighting by group — every test counts equally.

## Train / Eval Split

- **Default split**: 70% train / 30% eval (randomized)
- **Train set**: used for optimization (steps 3–9 of the loop)
- **Eval set**: used only when stop target is reached on train, to validate generalization
- Record the split in each run file (`metadata.split`) so it's reproducible

## Composite Score Formula

For each test:
```
composite_i = Σ (weight_d × score_d) for each dimension d
```

For the run:
```
composite_avg = mean(composite_i) for all tests in the split
```

Each dimension also has its own average:
```
score_avg_d = mean(score_d_i) for all tests in the split
```

---

## Stop Target

**Default stop target: 0.95**

When `composite_avg` on the **train** split reaches or exceeds this value:
1. Run the **eval** split
2. If eval `composite_avg` ≥ stop target → **STOP** and propose deploy to user
3. If eval `composite_avg` < stop target → flag potential overfitting, continue loop

To change the stop target, edit the value above. Consider:
- 0.90 — good enough for most use cases
- 0.95 — high quality (default)
- 0.98 — very high quality, may take many iterations
- 1.00 — perfect score, may be unreachable

---

## Quality Thresholds

| Level | Composite Score | Interpretation |
|-------|----------------|----------------|
| Excellent | ≥ 0.90 | Production-ready |
| Good | 0.75 – 0.89 | Acceptable with minor issues |
| Acceptable | 0.60 – 0.74 | Needs improvement |
| Insufficient | < 0.60 | Significant rework needed |

## Run Comparison

When comparing the current run to the previous run:

```
delta = current_composite - previous_composite
delta_pct = (delta / previous_composite) × 100
```

| Delta | Interpretation |
|-------|----------------|
| > +5% | Strong improvement |
| +1% to +5% | Moderate improvement |
| -1% to +1% | Marginal / no change |
| -5% to -1% | Moderate regression |
| < -5% | Strong regression |

## Selection Strategies

When choosing which dimensions to focus on for improvement (used by `improvements.md`):

| Strategy | Description | When to use |
|----------|-------------|-------------|
| **worst-N** | Target the N lowest-scoring dimensions | Default. Good for broad improvement |
| **highest-weight** | Target the lowest-scoring among high-weight dimensions | When you want maximum composite impact |
| **biggest-drop** | Target dimensions that regressed most vs previous run | After a regression, to recover |
