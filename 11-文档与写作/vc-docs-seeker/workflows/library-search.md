# General Library Documentation Search

**Use when:** User asks about entire library/framework

**Speed:** ⚡⚡ Moderate (30-60s)
**Token usage:** 🟡 Medium
**Accuracy:** 📚 Comprehensive

## Trigger Patterns

- "Documentation for [LIBRARY]"
- "[LIBRARY] getting started"
- "How to use [LIBRARY]"
- "[LIBRARY] API reference"

## Workflow

Use Context7 MCP first for general library documentation. Use the scripts below only as fallback when Context7 coverage is missing or when the user explicitly wants llms.txt/repository-style discovery.

```bash
# FALLBACK STEP 1: Execute detect-topic.js script
node scripts/detect-topic.js "<user query>"
# Returns: {"isTopicSpecific": false} for general queries

# FALLBACK STEP 2: Execute fetch-docs.js script (handles URL construction)
node scripts/fetch-docs.js "<user query>"
# Script constructs context7.com URL automatically
# Script handles GitHub/website URL patterns
# Returns: llms.txt content with 5-20+ URLs

# FALLBACK STEP 3: Execute analyze-llms-txt.js script
cat llms.txt | node scripts/analyze-llms-txt.js -
# Groups URLs: critical, important, supplementary
# Recommends: agent distribution strategy
# Returns: {totalUrls, grouped, distribution}

# FALLBACK STEP 4: Deploy agents based on script recommendation
# - 1-3 URLs: Single agent or direct WebFetch
# - 4-10 URLs: Split across 3-5 parallel workers
# - 11+ URLs: Deploy 7 agents or phased approach

# FALLBACK STEP 5: Aggregate and present
# Synthesize findings: installation, concepts, API, examples
```

## Examples

**Astro framework:**
```bash
# Execute scripts (no manual URL construction)
node scripts/detect-topic.js "Documentation for Astro"
# {"isTopicSpecific": false}

node scripts/fetch-docs.js "Documentation for Astro"
# Script fetches: context7.com/withastro/astro/llms.txt
# Returns: llms.txt with 8 URLs

node scripts/analyze-llms-txt.js < llms.txt
# {totalUrls: 8, distribution: "3-agents", grouped: {...}}

# Split across 3 parallel workers as recommended:
# Worker 1: Getting started, installation, setup
# Worker 2: Core concepts, components, layouts
# Worker 3: Configuration, API reference

# Aggregate and present comprehensive report
```

## Agent Distribution

**1-3 URLs:** Single agent
**4-10 URLs:** 3-5 agents (2-3 URLs each)
**11-20 URLs:** 7 agents (balanced)
**21+ URLs:** Two-phase (critical first, then important)

## Known Libraries

- Next.js: `vercel/next.js`
- Astro: `withastro/astro`
- Remix: `remix-run/remix`
- shadcn/ui: `shadcn-ui/ui`
- Better Auth: `better-auth/better-auth`

## Fallback

Scripts handle fallback automatically:
1. `fetch-docs.js` tries context7.com
2. If 404, script suggests WebSearch for llms.txt
3. If still unavailable: [Repository Analysis](./repo-analysis.md)
