# Delegate to Agent Tool

## Purpose

Route a task to a specialized agent for execution. Handles context passing, result collection, and error management.

## Tool Definition

```typescript
import { tool } from "ai";
import { z } from "zod";

export const delegateToAgent = tool({
  description: `Delegate a task to a specialized agent.
Use when a subtask requires specific capabilities.
Pass complete context needed for the agent to succeed.`,

  parameters: z.object({
    agentName: z.enum(["evaluator", "researcher", "writer", "analyst"])
      .describe("Name of the agent to delegate to"),
    
    task: z.string()
      .describe("Clear description of what the agent should do"),
    
    context: z.object({
      previousOutputs: z.array(z.string()).optional()
        .describe("Outputs from prior steps this agent needs"),
      
      documents: z.array(z.string()).optional()
        .describe("Relevant documents or data"),
      
      constraints: z.array(z.string()).optional()
        .describe("Requirements or limitations to observe")
    }).optional(),
    
    expectedOutput: z.object({
      format: z.enum(["text", "json", "markdown", "structured"])
        .describe("Expected output format"),
      
      schema: z.string().optional()
        .describe("JSON schema if format is structured"),
      
      maxLength: z.number().optional()
        .describe("Maximum length constraint")
    }).optional(),
    
    timeout: z.number().default(60000)
      .describe("Timeout in milliseconds")
  }),

  execute: async (input) => {
    return executeAgentDelegation(input);
  }
});
```

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| agentName | enum | Yes | Target agent |
| task | string | Yes | Task description |
| context | object | No | Context and dependencies |
| expectedOutput | object | No | Output requirements |
| timeout | number | No | Timeout ms (default: 60000) |

## Output Schema

```typescript
interface DelegationResult {
  success: boolean;
  
  agentName: string;
  task: string;
  
  output: {
    content: string | object;
    format: string;
  };
  
  execution: {
    startTime: string;
    endTime: string;
    durationMs: number;
    tokenUsage: {
      prompt: number;
      completion: number;
    };
  };
  
  error?: {
    code: string;
    message: string;
    retryable: boolean;
  };
}
```

## Available Agents

### Evaluator Agent
```typescript
await delegateToAgent.execute({
  agentName: "evaluator",
  task: "Evaluate the quality of this response against accuracy and clarity criteria",
  context: {
    documents: [responseToEvaluate],
    constraints: ["Use 1-5 scale", "Include justification"]
  },
  expectedOutput: { format: "structured" }
});
```

### Researcher Agent
```typescript
await delegateToAgent.execute({
  agentName: "researcher",
  task: "Research current best practices for LLM evaluation",
  context: {
    constraints: ["Focus on 2024 publications", "Include citations"]
  },
  expectedOutput: { format: "markdown" }
});
```

### Writer Agent
```typescript
await delegateToAgent.execute({
  agentName: "writer",
  task: "Write an executive summary of these research findings",
  context: {
    previousOutputs: [researchFindings],
    constraints: ["Maximum 500 words", "Non-technical audience"]
  },
  expectedOutput: { format: "text", maxLength: 2500 }
});
```

### Analyst Agent
```typescript
await delegateToAgent.execute({
  agentName: "analyst",
  task: "Analyze the trade-offs between direct scoring and pairwise comparison",
  context: {
    documents: [evaluationData],
    constraints: ["Quantitative where possible"]
  },
  expectedOutput: { format: "structured" }
});
```

## Error Handling

```typescript
const errorCodes = {
  "AGENT_NOT_FOUND": "Specified agent does not exist",
  "TIMEOUT": "Agent did not complete within timeout",
  "CONTEXT_TOO_LARGE": "Context exceeds agent's capacity",
  "INVALID_OUTPUT": "Agent output did not match expected format",
  "AGENT_ERROR": "Agent encountered an error during execution"
};
```

## Implementation Notes

1. **Context Optimization**: Compress context if needed before passing
2. **Timeout Handling**: Set realistic timeouts per agent type
3. **Retry Logic**: Implement retries for transient failures
4. **Audit Trail**: Log all delegations for traceability
5. **Resource Management**: Track token usage across delegations

