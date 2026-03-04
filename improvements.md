# Prompt Improvement Techniques

This file provides frameworks and strategies for generating better prompt variants.

---

## Prompt Engineering Frameworks

### Comparison Matrix

| Framework | Best for | Structure | Strengths |
|-----------|----------|-----------|-----------|
| **CO-STAR** | General-purpose agents | Context, Objective, Style, Tone, Audience, Response | Well-rounded, easy to apply |
| **RISEN** | Role-heavy agents | Role, Instructions, Steps, End goal, Narrowing | Strong role definition |
| **TIDD-EC** | Complex multi-step tasks | Task, Instructions, Do, Don't, Examples, Constraints | Great for guardrails |
| **RODES** | Output-focused agents | Role, Objective, Details, Examples, Sense-check | Strong output specification |

### CO-STAR

Use when the agent needs a well-rounded prompt covering context, style, and audience.

| Element | Description |
|---------|-------------|
| **C**ontext | Background information the agent needs |
| **O**bjective | What the agent should accomplish |
| **S**tyle | Writing style (formal, casual, technical) |
| **T**one | Emotional tone (friendly, professional, empathetic) |
| **A**udience | Who the agent is talking to |
| **R**esponse | Expected response format |

### RISEN

Use when role definition is critical (e.g., customer support, medical assistant).

| Element | Description |
|---------|-------------|
| **R**ole | Who the agent is |
| **I**nstructions | What to do |
| **S**teps | How to do it (ordered steps) |
| **E**nd goal | What success looks like |
| **N**arrowing | Constraints and boundaries |

### TIDD-EC

Use when you need strong guardrails — lots of "do this, don't do that."

| Element | Description |
|---------|-------------|
| **T**ask | The main task |
| **I**nstructions | Detailed instructions |
| **D**o | Things the agent should do |
| **D**on't | Things the agent must avoid |
| **E**xamples | Few-shot examples |
| **C**onstraints | Hard limits and boundaries |

### RODES

Use when the output format and quality are the primary concern.

| Element | Description |
|---------|-------------|
| **R**ole | Agent's role |
| **O**bjective | What to achieve |
| **D**etails | Specific requirements |
| **E**xamples | Example outputs |
| **S**ense-check | Self-verification instructions |

---

## Improvement Strategies

### 1. Worst-N (default)

Target the N lowest-scoring dimensions.

**Process:**
1. Sort dimensions by average score (ascending)
2. Take the bottom N (default: 2)
3. For each, analyze the test responses that scored lowest
4. Identify what the prompt is missing or saying wrong for that dimension
5. Rewrite the relevant prompt section

### 2. Highest-Weight

Target the lowest-scoring dimension among those with the highest weight.

**Process:**
1. Filter dimensions with weight ≥ 20%
2. Among those, find the one with the lowest score
3. Focus improvement entirely on that dimension
4. Use more aggressive rewriting (larger changes)

### 3. Biggest-Drop

Target dimensions that regressed the most compared to the previous run.

**Process:**
1. Compute delta for each dimension vs previous run
2. Sort by delta (ascending — biggest drops first)
3. Focus on recovering the dropped dimensions
4. Often means reverting specific changes while keeping improvements

---

## How to Generate a Variant

Follow this process when creating a new prompt version:

### 1. Analyze Errors
- Look at the lowest-scoring test responses
- Identify common failure patterns (missing info, wrong tone, safety gaps, format issues)
- Group failures by root cause

### 2. Identify Weak Section
- Map failures to specific sections of the current prompt
- Is the issue in the role definition? The instructions? The constraints? The examples?
- If the prompt lacks a section entirely, that's your target

### 3. Choose a Framework
- If the current prompt has no clear structure → apply CO-STAR or RISEN from scratch
- If the prompt is structured but weak in guardrails → add TIDD-EC elements
- If the output format is the issue → apply RODES elements
- If the prompt is already well-structured → make targeted edits, don't restructure

### 4. Rewrite
- Make the minimum change needed to address the identified weakness
- Preserve what's working well (don't rewrite sections that score high)
- Be specific: vague instructions → concrete instructions with examples
- Add constraints for safety issues
- Add examples for format/quality issues

### 5. Deploy
- Update the prompt via `integrations.md`
- The loop continues from Step 3 in `instructions.md`

---

## Tips

- **Small changes first.** A single targeted edit often outperforms a full rewrite.
- **Don't fix what's not broken.** If safety scores are 1.0, don't touch safety instructions.
- **Add examples when stuck.** Few-shot examples in the prompt are powerful for format and quality.
- **Be specific about what NOT to do** when safety scores are low.
- **If multiple frameworks fail**, consider that the issue may be in the test expectations, not the prompt.
