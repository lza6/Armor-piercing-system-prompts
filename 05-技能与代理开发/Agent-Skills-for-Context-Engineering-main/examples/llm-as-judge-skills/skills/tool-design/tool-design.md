# Agent Tool Design Skill

## Overview

Tools are the foundation of an agent's capabilities. An agent's ability to take meaningful actions depends entirely on how reliably it can generate valid tool inputs, how well those inputs align with user intent, and how effectively tool outputs inform next steps.

## Design Principles

### 1. Single Responsibility

Each tool should do one thing well. Complex operations should be composed from multiple tools.

```typescript
// Bad: Tool does too much
const analyzeAndSummarizeAndSend = { ... }

// Good: Separate concerns
const analyzeDocument = { ... }
const summarizeContent = { ... }
const sendEmail = { ... }
```

### 2. Clear Input Schemas

Use explicit, validated schemas with descriptive field names and constraints.

```typescript
const searchTool = tool({
  description: "Search for documents by semantic similarity",
  parameters: z.object({
    query: z.string().describe("Natural language search query"),
    limit: z.number().min(1).max(100).default(10)
      .describe("Maximum number of results to return"),
    filters: z.object({
      dateAfter: z.string().optional()
        .describe("ISO date string, only return docs after this date"),
      source: z.enum(["internal", "external", "all"]).default("all")
    }).optional()
  }),
  execute: async (input) => { ... }
});
```

### 3. Predictable Output Structure

Return consistent, typed output that the model can reliably parse.

```typescript
interface ToolResult<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    retryable: boolean;
  };
  metadata: {
    executionTimeMs: number;
    source?: string;
  };
}
```

### 4. Graceful Error Handling

Tools should never throw unhandled exceptions. Always return structured errors.

```typescript
execute: async (input) => {
  try {
    const result = await performAction(input);
    return { success: true, data: result };
  } catch (error) {
    return {
      success: false,
      error: {
        code: error.code ?? "UNKNOWN_ERROR",
        message: error.message,
        retryable: isRetryable(error)
      }
    };
  }
}
```

## Tool Categories

### Read-Only Tools
- Database queries
- API fetches
- File reads
- Search operations

Safe to execute without approval. Return data but don't mutate state.

### State-Modifying Tools
- Database writes
- File modifications
- API POST/PUT/DELETE
- System configuration changes

May require human approval. Consider `needsApproval` flag.

### Dangerous Tools
- File deletion
- Payment processing
- Production deployments
- Sending external communications

Should always require approval and audit logging.

## AI SDK 6 Tool Features

### Tool Execution Approval
```typescript
const deleteTool = tool({
  description: "Delete a file from the system",
  parameters: z.object({
    path: z.string()
  }),
  needsApproval: true, // Requires human approval
  execute: async ({ path }) => { ... }
});

// Dynamic approval based on input
const commandTool = tool({
  description: "Execute a shell command",
  parameters: z.object({
    command: z.string()
  }),
  needsApproval: ({ command }) => {
    return command.includes("rm") || command.includes("delete");
  },
  execute: async ({ command }) => { ... }
});
```

### Strict Mode
Enable native strict mode for guaranteed schema compliance:
```typescript
const strictTool = tool({
  description: "...",
  parameters: schema,
  strict: true, // Enable strict mode
  execute: async (input) => { ... }
});
```

### Input Examples
Help the model understand expected input format:
```typescript
const complexTool = tool({
  description: "Create a calendar event",
  parameters: eventSchema,
  inputExamples: [
    {
      title: "Team Standup",
      date: "2024-01-15",
      time: "09:00",
      duration: 30,
      attendees: ["alice@example.com", "bob@example.com"]
    }
  ],
  execute: async (input) => { ... }
});
```

### toModelOutput
Control what gets sent back to the model:
```typescript
const readFileTool = tool({
  description: "Read file contents",
  parameters: z.object({ path: z.string() }),
  execute: async ({ path }) => {
    const content = await fs.readFile(path, 'utf-8');
    return { path, content, size: content.length };
  },
  toModelOutput: (result) => {
    // Only send truncated content to model
    return {
      path: result.path,
      content: result.content.slice(0, 5000),
      truncated: result.content.length > 5000
    };
  }
});
```

## Best Practices

1. **Descriptive Names**: Tool names should clearly indicate their function
2. **Comprehensive Descriptions**: Include usage examples in tool descriptions
3. **Reasonable Defaults**: Provide sensible defaults for optional parameters
4. **Idempotency**: Design tools to be safely re-executable when possible
5. **Timeout Handling**: Implement timeouts for external operations
6. **Rate Limiting**: Protect against runaway tool execution
7. **Logging**: Log all tool invocations for debugging and audit

