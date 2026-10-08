# Read URL Tool

## Purpose

Extract and parse content from a given URL. Returns structured text content with metadata about the source.

## Tool Definition

```typescript
import { tool } from "ai";
import { z } from "zod";

export const readUrl = tool({
  description: `Read and extract content from a URL.
Returns the main text content, stripped of navigation and ads.
Use after webSearch to get full content from relevant results.`,

  parameters: z.object({
    url: z.string().url()
      .describe("The URL to read"),
    
    contentType: z.enum(["auto", "article", "documentation", "paper", "code"]).default("auto")
      .describe("Hint for content type to optimize extraction"),
    
    maxLength: z.number().min(1000).max(50000).default(10000)
      .describe("Maximum characters to return"),
    
    extractSections: z.boolean().default(true)
      .describe("Whether to identify and label sections"),
    
    includeMetadata: z.boolean().default(true)
      .describe("Include author, date, and other metadata")
  }),

  execute: async (input) => {
    return extractUrlContent(input);
  }
});
```

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| url | string | Yes | URL to read |
| contentType | enum | No | Content type hint |
| maxLength | number | No | Max chars (default: 10000) |
| extractSections | boolean | No | Label sections |
| includeMetadata | boolean | No | Include metadata |

## Output Schema

```typescript
interface ReadUrlResult {
  success: boolean;
  
  url: string;
  title: string;
  
  content: {
    full: string;
    sections?: {
      heading: string;
      level: number;  // h1=1, h2=2, etc.
      content: string;
    }[];
  };
  
  metadata?: {
    author?: string;
    publishedDate?: string;
    lastModified?: string;
    description?: string;
    keywords?: string[];
    source: string;
  };
  
  stats: {
    totalCharacters: number;
    truncated: boolean;
    sectionsFound: number;
  };
  
  error?: {
    code: string;
    message: string;
  };
}
```

## Usage Example

```typescript
const content = await readUrl.execute({
  url: "https://eugeneyan.com/writing/llm-evaluators/",
  contentType: "article",
  maxLength: 15000,
  extractSections: true,
  includeMetadata: true
});

// Result:
// {
//   success: true,
//   url: "https://eugeneyan.com/writing/llm-evaluators/",
//   title: "Evaluating the Effectiveness of LLM-Evaluators",
//   content: {
//     full: "LLM-evaluators, also known as LLM-as-a-Judge...",
//     sections: [
//       {
//         heading: "Key considerations before adopting an LLM-evaluator",
//         level: 2,
//         content: "Before reviewing the literature..."
//       },
//       ...
//     ]
//   },
//   metadata: {
//     author: "Eugene Yan",
//     publishedDate: "2024-06-15",
//     source: "eugeneyan.com"
//   },
//   stats: {
//     totalCharacters: 15000,
//     truncated: true,
//     sectionsFound: 8
//   }
// }
```

## Content Type Handling

| Type | Optimization |
|------|-------------|
| article | Prioritize main content, skip sidebars |
| documentation | Preserve code blocks, keep structure |
| paper | Extract abstract, sections, references |
| code | Preserve formatting, syntax highlighting |
| auto | Detect type from content |

## Error Handling

```typescript
const errorCodes = {
  "URL_NOT_FOUND": "Page does not exist (404)",
  "ACCESS_DENIED": "Page requires authentication (401/403)",
  "TIMEOUT": "Request timed out",
  "BLOCKED": "Access blocked by robots.txt or rate limit",
  "INVALID_CONTENT": "Content could not be parsed",
  "UNSUPPORTED_TYPE": "Content type not supported (e.g., binary)"
};
```

## Implementation Notes

1. **Respect robots.txt**: Check and honor robots.txt directives
2. **Rate Limiting**: Don't hammer the same domain
3. **User Agent**: Use appropriate user agent string
4. **Timeouts**: Set reasonable timeouts (10-30s)
5. **JavaScript Rendering**: Consider headless browser for JS-heavy sites
6. **Caching**: Cache content for repeated reads

