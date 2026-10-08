# Skills Index

Skills are foundational knowledge modules that inform the design and implementation of agents, tools, and prompts.

## Available Skills

### LLM Evaluator
**Path**: `skills/llm-evaluator/llm-evaluator.md`

Covers LLM-as-a-Judge evaluation methodology including:
- Scoring approaches (direct, pairwise, reference-based)
- Evaluation metrics (classification, correlation)
- Known biases and mitigation strategies
- Implementation patterns

**Key Takeaways**:
- Use direct scoring for objective evaluations
- Use pairwise comparison for subjective preferences
- Always mitigate position bias
- Prefer classification metrics for interpretability

### Context Fundamentals
**Path**: `skills/context-fundamentals/context-fundamentals.md`

Covers context engineering principles including:
- Context window management
- Information hierarchy
- Context types (static, dynamic, ephemeral)
- Relevance filtering

**Key Takeaways**:
- Structure context by priority
- Be explicit over implicit
- Remove redundancy
- Signal freshness of information

### Tool Design
**Path**: `skills/tool-design/tool-design.md`

Covers agent tool design best practices including:
- Single responsibility principle
- Input/output schemas
- Error handling patterns
- AI SDK 6 features (approval, strict mode, examples)

**Key Takeaways**:
- Clear, validated schemas
- Predictable output structure
- Graceful error handling
- Consider approval for dangerous tools

## Skill Application Matrix

| Skill | Agents | Tools | Prompts |
|-------|--------|-------|---------|
| LLM Evaluator | Evaluator | directScore, pairwiseCompare | evaluation/* |
| Context Fundamentals | All | All (context params) | All (context handling) |
| Tool Design | All (tool selection) | All | orchestrator-prompt |

## Adding New Skills

1. Create skill directory: `skills/<skill-name>/`
2. Create main file: `skills/<skill-name>/<skill-name>.md`
3. Include:
   - Overview and purpose
   - Core principles
   - Practical patterns
   - Implementation examples
   - References
4. Update this index

## Skill Development Guidelines

- Focus on principles that transfer across implementations
- Include concrete examples and patterns
- Reference authoritative sources
- Keep content actionable, not just theoretical
- Update as understanding evolves

