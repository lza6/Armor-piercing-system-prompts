# Generate Rubric Tool

## Purpose

Automatically generate a scoring rubric for a given evaluation criterion. Creates detailed descriptions for each score level to ensure consistent evaluation.

## Tool Definition

```typescript
import { tool } from "ai";
import { z } from "zod";

export const generateRubric = tool({
  description: `Generate a scoring rubric for an evaluation criterion.
Creates detailed descriptions for each score level.
Use when you need to establish consistent evaluation standards.`,

  parameters: z.object({
    criterionName: z.string()
      .describe("Name of the criterion (e.g., 'Factual Accuracy')"),
    
    criterionDescription: z.string()
      .describe("What this criterion measures"),
    
    scale: z.enum(["1-3", "1-5", "1-10"]).default("1-5")
      .describe("Scoring scale to use"),
    
    domain: z.string().optional()
      .describe("Domain context (e.g., 'medical writing', 'code review')"),
    
    includeExamples: z.boolean().default(true)
      .describe("Include example text for each score level"),
    
    strictness: z.enum(["lenient", "balanced", "strict"]).default("balanced")
      .describe("How strictly to define score boundaries")
  }),

  execute: async (input) => {
    return generateRubricWithLLM(input);
  }
});
```

## Input Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| criterionName | string | Yes | Name of criterion |
| criterionDescription | string | Yes | What criterion measures |
| scale | enum | No | Scoring scale (default: 1-5) |
| domain | string | No | Domain for context |
| includeExamples | boolean | No | Include examples (default: true) |
| strictness | enum | No | Score boundary strictness |

## Output Schema

```typescript
interface GeneratedRubric {
  success: boolean;
  
  criterion: {
    name: string;
    description: string;
  };
  
  scale: {
    min: number;
    max: number;
    type: string;
  };
  
  levels: {
    score: number;
    label: string;        // e.g., "Excellent", "Poor"
    description: string;  // Detailed description
    characteristics: string[];  // Key characteristics
    example?: string;     // Example text at this level
  }[];
  
  scoringGuidelines: string[];
  
  edgeCases: {
    situation: string;
    guidance: string;
  }[];
  
  metadata: {
    domain: string | null;
    strictness: string;
    generationTimeMs: number;
  };
}
```

## Usage Example

```typescript
const rubric = await generateRubric.execute({
  criterionName: "Code Readability",
  criterionDescription: "How easy the code is to read and understand",
  scale: "1-5",
  domain: "code review",
  includeExamples: true,
  strictness: "balanced"
});

// Result:
// {
//   criterion: {
//     name: "Code Readability",
//     description: "How easy the code is to read and understand"
//   },
//   scale: { min: 1, max: 5, type: "1-5" },
//   levels: [
//     {
//       score: 1,
//       label: "Poor",
//       description: "Code is extremely difficult to understand...",
//       characteristics: [
//         "No meaningful variable names",
//         "Deeply nested logic without explanation",
//         "No comments on complex sections"
//       ],
//       example: "function x(a,b,c){return a?b+c:c-b;}"
//     },
//     {
//       score: 5,
//       label: "Excellent",
//       description: "Code is immediately understandable...",
//       characteristics: [
//         "Self-documenting variable and function names",
//         "Appropriate comments explaining 'why'",
//         "Clear logical structure"
//       ],
//       example: "function calculateShippingCost(weight, distance, expedited) {\n  // Base rate plus per-mile charge\n  const baseCost = weight * BASE_RATE_PER_KG;\n  ..."
//     },
//     ...
//   ],
//   scoringGuidelines: [
//     "Focus on clarity for someone unfamiliar with the codebase",
//     "Consider both naming and structure",
//     "Comments should explain 'why', not 'what'"
//   ],
//   edgeCases: [
//     {
//       situation: "Code uses domain-specific abbreviations",
//       guidance: "Accept if abbreviations are standard in the domain"
//     }
//   ]
// }
```

## Rubric Templates

### Factual Accuracy (1-5)
```
5: All claims factually correct, properly sourced
4: Minor factual issues, non-critical
3: Some factual errors, main points correct
2: Multiple factual errors affecting reliability
1: Fundamentally incorrect or misleading
```

### Clarity (1-5)
```
5: Immediately understandable, well-structured
4: Clear with minor ambiguities
3: Generally clear, some confusing sections
2: Difficult to follow, unclear organization
1: Incomprehensible or incoherent
```

### Completeness (1-5)
```
5: Addresses all aspects comprehensively
4: Covers main points, minor gaps
3: Addresses core requirements, notable gaps
2: Missing significant required elements
1: Fails to address the question
```

## Implementation Notes

1. **Domain Adaptation**: Rubrics should reflect domain-specific expectations
2. **Boundary Clarity**: Clear distinctions between adjacent scores
3. **Example Quality**: Examples should be realistic, not strawmen
4. **Edge Case Coverage**: Anticipate common ambiguous situations
5. **Calibration**: Test rubric against known samples before use

