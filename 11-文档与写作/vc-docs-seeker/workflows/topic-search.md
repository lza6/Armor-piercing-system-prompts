# Topic-Specific Documentation Search

**Use when:** User asks about specific feature/component/concept

**Speed:** ⚡ Fastest (10-15s)
**Token usage:** 🟢 Minimal
**Accuracy:** 🎯 Highly targeted

## Trigger Patterns

- "How do I use [FEATURE] in [LIBRARY]?"
- "[LIBRARY] [COMPONENT] documentation"
- "Implement [FEATURE] with [LIBRARY]"
- "[LIBRARY] [CONCEPT] guide"

## Workflow

Use Context7 MCP first for topic-specific documentation. Use the scripts below only as fallback when Context7 coverage is missing or insufficient.

```bash
# FALLBACK STEP 1: Execute detect-topic.js script
node scripts/detect-topic.js "<user query>"
# Returns: {"topic": "X", "library": "Y", "isTopicSpecific": true}

# FALLBACK STEP 2: Execute fetch-docs.js script (handles URL construction automatically)
node scripts/fetch-docs.js "<user query>"
# Script constructs: context7.com/{library}/llms.txt?topic={topic}
# Script handles fallback if topic URL fails
# Returns: llms.txt content with 1-5 URLs

# FALLBACK STEP 3: Process results based on URL count
# - 1-3 URLs: Read directly with WebFetch tool
# - 4-5 URLs: Split across 2-3 parallel workers when available

# FALLBACK STEP 4: Present findings
# Focus on specific feature: installation, usage, examples
```

## Examples

**shadcn date picker:**
```bash
# Execute script (automatic URL construction)
node scripts/detect-topic.js "How do I use date picker in shadcn?"
# {"topic": "date", "library": "shadcn/ui", "isTopicSpecific": true}

node scripts/fetch-docs.js "How do I use date picker in shadcn?"
# Script fetches: context7.com/shadcn-ui/ui/llms.txt?topic=date
# Returns: 2-3 date-specific URLs

# Read URLs directly with WebFetch
# Present date picker documentation
```

**Next.js caching:**
```bash
# Execute scripts (no manual URL needed)
node scripts/detect-topic.js "Next.js caching strategies"
# {"topic": "cache", "library": "next.js", "isTopicSpecific": true}

node scripts/fetch-docs.js "Next.js caching strategies"
# Script fetches: context7.com/vercel/next.js/llms.txt?topic=cache
# Returns: 3-4 URLs

# Process URLs via 2 parallel workers
# Present caching strategies
```

## Benefits

✅ 10x faster than full docs
✅ No filtering needed
✅ Minimal context load
✅ Best user experience

## Fallback

If topic URL returns 404:
→ Fallback to [General Library Search](./library-search.md)
