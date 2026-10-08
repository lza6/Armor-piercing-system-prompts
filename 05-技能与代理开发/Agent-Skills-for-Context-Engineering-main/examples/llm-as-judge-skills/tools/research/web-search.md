# Web Search Tool

## Purpose

Search the web for relevant information on a given topic. Returns structured results with snippets, URLs, and metadata.

## Tool Definition

```typescript
import { tool } from "ai";
import { z } from "zod";

export const webSearch = tool({
  description: `Search the web for information on a topic.
Returns relevant results with snippets and URLs.
Use for gathering current information, verifying facts, or research.`,

  parameters: z.object({
    query: z.string()
      .describe("Search query - be specific for better results"),
    
    maxResults: z.number().min(1).max(20).default(10)
      .describe("Maximum number of results to return"),
    
    filters: z.object({
      dateRange: z.enum(["day", "week", "month", "year", "any"]).default("any")
        .describe("Limit results to a time period"),
      
      sourceType: z.enum(["all", "news", "academic", "documentation"]).default("all")
        .describe("Type of sources to prioritize"),
      
      excludeDomains: z.array(z.string()).optional()
        .describe("Domains to exclude from results")
    }).optional()
  }),

  execute: async (input) => {
    return performWebSearch(input);
  }
});
```

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| query | string | Yes | Search query |
| maxResults | number | No | Max results (default: 10) |
| filters.dateRange | enum | No | Time period filter |
| filters.sourceType | enum | No | Source type priority |
| filters.excludeDomains | string[] | No | Domains to exclude |

## Output Schema

```typescript
interface WebSearchResult {
  success: boolean;
  
  results: {
    title: string;
    url: string;
    snippet: string;
    source: string;      // Domain name
    publishedDate?: string;
    relevanceScore: number;
  }[];
  
  totalResults: number;
  
  metadata: {
    query: string;
    searchTimeMs: number;
    filtersApplied: string[];
  };
}
```

## Usage Example

```typescript
const results = await webSearch.execute({
  query: "LLM-as-a-Judge evaluation methods 2024",
  maxResults: 10,
  filters: {
    dateRange: "year",
    sourceType: "academic"
  }
});

// Result:
// {
//   success: true,
//   results: [
//     {
//       title: "Judging LLM-as-a-Judge with MT-Bench",
//       url: "https://arxiv.org/abs/...",
//       snippet: "We study the effectiveness of LLM-as-a-Judge...",
//       source: "arxiv.org",
//       publishedDate: "2024-01-15",
//       relevanceScore: 0.95
//     },
//     ...
//   ],
//   totalResults: 10,
//   metadata: {
//     query: "LLM-as-a-Judge evaluation methods 2024",
//     searchTimeMs: 342,
//     filtersApplied: ["dateRange:year", "sourceType:academic"]
//   }
// }
```

## Query Optimization Tips

1. **Specific Terms**: Use precise terminology
2. **Quotes**: Use quotes for exact phrases
3. **Operators**: Support site:, -term, OR
4. **Context**: Include relevant context terms
5. **Recency**: Add year for recent info

## Implementation Notes

1. **Rate Limiting**: Implement appropriate rate limits
2. **Caching**: Cache results for repeated queries
3. **Result Quality**: Filter out low-quality sources
4. **Error Handling**: Handle API failures gracefully
5. **Privacy**: Log queries appropriately

