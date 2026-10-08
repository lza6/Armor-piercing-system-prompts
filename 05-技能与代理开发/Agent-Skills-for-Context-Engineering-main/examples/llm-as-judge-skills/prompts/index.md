# Prompts Index

Prompts are reusable templates that define how agents and tools interact with LLMs.

## Prompt Categories

### Evaluation Prompts
**Path**: `prompts/evaluation/`

Templates for quality assessment tasks.

| Prompt | Purpose | Used By |
|--------|---------|---------|
| `direct-scoring-prompt` | Evaluate single response | Evaluator Agent, directScore tool |
| `pairwise-comparison-prompt` | Compare two responses | Evaluator Agent, pairwiseCompare tool |

---

### Research Prompts
**Path**: `prompts/research/`

Templates for information gathering and synthesis.

| Prompt | Purpose | Used By |
|--------|---------|---------|
| `research-synthesis-prompt` | Synthesize findings | Research Agent |

---

### Agent System Prompts
**Path**: `prompts/agent-system/`

System prompts for agent definitions.

| Prompt | Purpose | Used By |
|--------|---------|---------|
| `orchestrator-prompt` | Multi-agent coordination | Orchestrator Agent |

## Prompt Template Format

### Standard Structure

```markdown
# Prompt Name

## Purpose
Brief description of what this prompt accomplishes.

## Prompt Template
```markdown
[The actual prompt with {{variables}}]
```

## Variables
| Variable | Description | Required |
|----------|-------------|----------|
| var_name | What it contains | Yes/No |

## Example Usage
Concrete example showing inputs and expected outputs.

## Best Practices
Guidelines for using this prompt effectively.
```

### Variable Syntax

Use Handlebars-style templating:

```markdown
{{variable}}                 # Simple substitution
{{#if condition}}...{{/if}} # Conditional section
{{#each array}}...{{/each}} # Iteration
```

## Prompt Design Principles

### 1. Clear Role Definition
Tell the model exactly what it is and what it's doing.

```markdown
You are an expert evaluator assessing the quality of AI-generated responses.
```

### 2. Explicit Instructions
Don't assume the model will infer requirements.

```markdown
For each criterion:
1. First, identify specific evidence from the response
2. Then, determine the appropriate score based on the rubric
3. Finally, provide actionable feedback
```

### 3. Structured Output
Specify the exact format you need.

```markdown
Format your response as structured JSON:
```json
{
  "scores": [...],
  "summary": {...}
}
```
```

### 4. Guard Rails
Include constraints and warnings.

```markdown
Important Guidelines:
- Do NOT prefer responses simply because they are longer
- Do NOT prefer responses based on their position (A vs B)
- Focus on the specified criteria
```

## Adding New Prompts

1. Determine category or create new: `prompts/<category>/`
2. Create prompt file: `prompts/<category>/<prompt-name>.md`
3. Include:
   - Purpose
   - Template with variables
   - Variable documentation
   - Example usage
   - Best practices
4. Update this index

## Prompt Testing Checklist

- [ ] Variables render correctly
- [ ] Output format is parseable
- [ ] Edge cases are handled
- [ ] Instructions are unambiguous
- [ ] Examples match expected output
- [ ] Constraints are clear

