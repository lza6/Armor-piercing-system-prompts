# Tools Index

Tools provide specific capabilities that agents can use to accomplish tasks.

## Tool Categories

### Evaluation Tools
**Path**: `tools/evaluation/`

Tools for assessing LLM output quality.

| Tool | Purpose | Approval |
|------|---------|----------|
| `directScore` | Score response against criteria | No |
| `pairwiseCompare` | Compare two responses | No |
| `generateRubric` | Generate scoring rubric | No |
| `extractCriteria` | Extract criteria from task | No |

---

### Research Tools
**Path**: `tools/research/`

Tools for gathering and processing information.

| Tool | Purpose | Approval |
|------|---------|----------|
| `webSearch` | Search the web | No |
| `readUrl` | Extract content from URL | No |
| `extractClaims` | Identify claims in text | No |
| `verifyClaim` | Cross-reference a claim | No |
| `synthesize` | Combine findings | No |

---

### Orchestration Tools
**Path**: `tools/orchestration/`

Tools for managing multi-agent workflows.

| Tool | Purpose | Approval |
|------|---------|----------|
| `delegateToAgent` | Route task to agent | No |
| `parallelExecution` | Run tasks concurrently | No |
| `waitForCompletion` | Wait for async tasks | No |
| `synthesizeResults` | Combine agent outputs | No |
| `handleError` | Manage failures | No |

## Tool Design Patterns

### Standard Tool Structure

```typescript
export const toolName = tool({
  description: "Clear description of what tool does",
  
  parameters: z.object({
    // Required parameters first
    requiredParam: z.string().describe("What this parameter is for"),
    
    // Optional parameters with defaults
    optionalParam: z.number().default(10)
      .describe("What this parameter controls")
  }),
  
  // Approval for dangerous operations
  needsApproval: false, // or true, or function
  
  // Strict mode for guaranteed schema compliance
  strict: true,
  
  execute: async (input) => {
    try {
      const result = await performOperation(input);
      return { success: true, data: result };
    } catch (error) {
      return {
        success: false,
        error: {
          code: error.code ?? "UNKNOWN",
          message: error.message,
          retryable: isRetryable(error)
        }
      };
    }
  },
  
  // Optional: control what model sees
  toModelOutput: (result) => ({
    summary: result.data.summary,
    truncated: result.data.full.length > 5000
  })
});
```

### Error Response Pattern

```typescript
interface ToolError {
  code: string;       // Machine-readable error code
  message: string;    // Human-readable message
  retryable: boolean; // Whether retry might help
  details?: object;   // Additional context
}

interface ToolResult<T> {
  success: boolean;
  data?: T;
  error?: ToolError;
  metadata: {
    executionTimeMs: number;
    [key: string]: any;
  };
}
```

## Adding New Tools

1. Determine category or create new: `tools/<category>/`
2. Create tool file: `tools/<category>/<tool-name>.md`
3. Define:
   - Purpose and description
   - Input parameters with Zod schema
   - Output schema
   - Error codes
   - Usage examples
4. Update this index
5. Assign to relevant agents

## Tool Selection Guidelines

### When Agent Needs To...

| Action | Tool Category | Suggested Tools |
|--------|---------------|-----------------|
| Assess quality | Evaluation | directScore, pairwiseCompare |
| Find information | Research | webSearch, readUrl |
| Verify facts | Research | verifyClaim, extractClaims |
| Coordinate work | Orchestration | delegateToAgent |
| Wait for results | Orchestration | waitForCompletion |

