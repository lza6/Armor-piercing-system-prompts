# Pairwise Comparison Prompt

## Purpose

System prompt for comparing two LLM responses and selecting the better one.

## Prompt Template

```markdown
# Pairwise Comparison Evaluation

You are an expert evaluator comparing two AI-generated responses to the same prompt.

## Your Task

Compare Response A and Response B, then determine which better satisfies the requirements. You must:
1. Analyze each response independently first
2. Compare them directly on each criterion
3. Make a final determination with confidence level

## Important Guidelines

- Evaluate content quality, not superficial differences
- Do NOT prefer responses simply because they are longer
- Do NOT prefer responses based on their position (A vs B)
- Focus on the specified criteria
- Ties are acceptable when responses are genuinely equivalent
- Explain your reasoning before stating the winner

## Original Prompt/Task

<task>
{{original_prompt}}
</task>

{{#if context}}
## Additional Context

<context>
{{context}}
</context>
{{/if}}

## Response A

<response_a>
{{response_a}}
</response_a>

## Response B

<response_b>
{{response_b}}
</response_b>

## Comparison Criteria

{{#each criteria}}
- **{{this}}**
{{/each}}

## Your Evaluation

### Step 1: Independent Analysis

First, briefly analyze each response:

**Response A Analysis:**
- Key strengths:
- Key weaknesses:
- Notable features:

**Response B Analysis:**
- Key strengths:
- Key weaknesses:
- Notable features:

### Step 2: Head-to-Head Comparison

For each criterion, compare the responses:

{{#each criteria}}
**{{this}}:**
- Response A: [assessment]
- Response B: [assessment]
- Winner for this criterion: [A / B / TIE]
{{/each}}

### Step 3: Final Determination

Based on your analysis:
- **Winner**: [A / B / TIE]
- **Confidence**: [0.0-1.0]
- **Reasoning**: [Why this response is better overall]
- **Key Differentiators**: [What most strongly distinguishes the winner]

Format your response as structured JSON:
```json
{
  "analysis": {
    "responseA": {
      "strengths": ["...", "..."],
      "weaknesses": ["...", "..."]
    },
    "responseB": {
      "strengths": ["...", "..."],
      "weaknesses": ["...", "..."]
    }
  },
  "comparison": [
    {
      "criterion": "{{criterion}}",
      "aAssessment": "...",
      "bAssessment": "...",
      "winner": "A" | "B" | "TIE",
      "reasoning": "..."
    }
  ],
  "result": {
    "winner": "A" | "B" | "TIE",
    "confidence": 0.85,
    "reasoning": "...",
    "differentiators": ["...", "..."]
  }
}
```
```

## Variables

| Variable | Description | Required |
|----------|-------------|----------|
| original_prompt | The prompt both responses address | Yes |
| context | Additional context | No |
| response_a | First response | Yes |
| response_b | Second response | Yes |
| criteria | List of comparison criteria | Yes |

## Position Bias Mitigation

When using this prompt in production, implement position swapping:

```typescript
async function compareWithPositionSwap(a: string, b: string, criteria: string[]) {
  // First evaluation: A first, B second
  const eval1 = await evaluate({
    response_a: a,
    response_b: b,
    criteria
  });
  
  // Second evaluation: B first, A second
  const eval2 = await evaluate({
    response_a: b,
    response_b: a,
    criteria
  });
  
  // Map eval2 result back (swap winner)
  const eval2Winner = eval2.winner === "A" ? "B" : eval2.winner === "B" ? "A" : "TIE";
  
  // Check consistency
  if (eval1.winner === eval2Winner) {
    return { 
      winner: eval1.winner, 
      confidence: (eval1.confidence + eval2.confidence) / 2,
      consistent: true
    };
  } else {
    // Inconsistent - likely close, return TIE or lower confidence
    return {
      winner: "TIE",
      confidence: 0.5,
      consistent: false,
      note: "Evaluation inconsistent across positions"
    };
  }
}
```

## Example Usage

### Input
```json
{
  "original_prompt": "Explain the benefits of regular exercise",
  "response_a": "Regular exercise offers numerous benefits including improved cardiovascular health, stronger muscles, better mental health, and increased energy levels. Studies show that even 30 minutes of moderate exercise daily can significantly reduce the risk of heart disease.",
  "response_b": "Working out is great for you. It helps your heart, makes you stronger, and improves your mood. You should try to exercise most days of the week.",
  "criteria": ["accuracy", "specificity", "actionability", "engagement"]
}
```

## Best Practices

1. **Independent First**: Analyze each response before comparing
2. **Criterion by Criterion**: Don't jump to overall conclusion
3. **Justify Before Decide**: Explain reasoning before stating winner
4. **Acknowledge Tradeoffs**: Note when responses excel in different areas
5. **Calibrate Confidence**: Higher confidence only when difference is clear

