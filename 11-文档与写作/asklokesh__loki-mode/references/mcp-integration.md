# MCP Integration Reference

Model Context Protocol (MCP) servers extend Claude Code's capabilities with specialized tools.

---

## Recommended MCP Servers

### 1. Playwright MCP (E2E Testing)

**Purpose:** Browser automation for end-to-end testing and visual verification.

**When to use:**
- Feature verification (visual confirmation)
- E2E test automation
- Screenshot capture for artifacts

**Configuration:**
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/playwright-mcp"]
    }
  }
}
```

**Tools provided:**
- `browser_navigate` - Navigate to URL
- `browser_click` - Click elements
- `browser_type` - Type text
- `browser_screenshot` - Capture screenshots

**SDLC Phase:** QA (E2E testing)

**Limitation:** Cannot detect browser-native alert modals. Use custom UI for confirmations.

---

### 2. Parallel AI (Web Research)

**Purpose:** Production-grade web research with evidence-based results and provenance.

**Why Parallel AI:**
- 48% accuracy on complex research tasks (vs native LLM search)
- Evidence-based results with provenance for every atomic output
- Monitor API for tracking web changes (dependencies, competitors)
- Task API with custom input/output schemas for structured research

**When to use:**
- Discovery phase: PRD gap analysis, competitor research
- Web Research phase: Feature comparisons, market analysis
- Dependency Management: Security advisory monitoring

**Configuration:**
```json
{
  "mcpServers": {
    "parallel-search": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/parallel-search-mcp"],
      "env": {
        "PARALLEL_API_KEY": "your-api-key"
      }
    },
    "parallel-task": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/parallel-task-mcp"],
      "env": {
        "PARALLEL_API_KEY": "your-api-key"
      }
    }
  }
}
```

**Tools provided:**

| Tool | Purpose | Use Case |
|------|---------|----------|
| `parallel_search` | Web search with LLM-optimized excerpts | Quick lookups, fact-checking |
| `parallel_extract` | Extract content from specific URLs | Documentation parsing |
| `parallel_task` | Complex research with custom schemas | Competitor analysis, market research |
| `parallel_monitor` | Track web changes with webhooks | Dependency updates, security alerts |

**SDLC Phases:** Discovery, Web Research, Continuous Monitoring

**Pricing:** Pay-per-query (not token-based). See https://parallel.ai/pricing

**API Documentation:** https://docs.parallel.ai/

---

### 3. Repowise - Codebase Intelligence (Recommended)

**Purpose:** Deep codebase intelligence through 8 MCP tools. When installed, Loki Mode automatically uses its tools for richer context during builds.

**When to use:**
- First encounter with an unfamiliar codebase
- Before modifying files with many dependents
- Architecture documentation and decision history
- Semantic code search (natural language)

**Setup:**
```bash
pip install repowise
repowise init .  # One-time indexing (~25 min)
```

**Configuration:**
```json
{
  "mcpServers": {
    "repowise": {
      "command": "repowise",
      "args": ["mcp", "--path", "."]
    }
  }
}
```

**Tools provided:**

| Tool | Purpose | Use Case |
|------|---------|----------|
| `get_overview()` | Architecture summary | First call on unfamiliar codebase |
| `get_context(targets)` | Docs, ownership, decisions | Before modifying files |
| `get_risk(targets)` | Hotspots, dependents, co-changes | Before modifying files |
| `get_why(query)` | Decision history | Before architectural changes |
| `search_codebase(query)` | Natural language code search | Finding code semantically |
| `get_dependency_path(from, to)` | Trace connections between files | Dependency analysis |
| `get_dead_code()` | Find unreachable code | Cleanup and refactoring |
| `get_architecture_diagram(module)` | Generate Mermaid diagrams | Documentation generation |

**SDLC Phases:** Bootstrap (overview), Development (context/risk), Documentation (diagrams)

**Integration with Loki Mode:**
When Repowise MCP is detected (via `.claude/mcp.json`), Loki Mode automatically:
1. Calls `get_overview()` during the BOOTSTRAP phase
2. Calls `get_risk()` before modifying hotspot files
3. Calls `get_context()` when loading relevant file context
4. Uses `search_codebase()` instead of manual grep for semantic code search

See `skills/documentation.md` for documentation generation using Repowise.

---

## Built-in Hybrid Codebase Search (loki_code_search)

Loki Mode ships its own codebase search that does not require any third-party MCP
server. It is exposed both as MCP tools (`loki_code_search`,
`loki_code_search_stats`) and as a CLI subcommand (`loki code search`). It combines
lexical and semantic retrieval over the indexed codebase:

- Lexical: ripgrep / grep (with a python scan fallback)
- Semantic: ChromaDB over the `loki-codebase` collection
- Fusion: reciprocal rank fusion (RRF) merges the two ranked lists
- Truncation: results are deduped by file:line and trimmed to a token budget
  (greedy, highest fused score first)

When ChromaDB or its docker container is unreachable, search degrades to grep-only
so it still returns results instead of erroring. The implementation lives in
`tools/hybrid_search.py`.

### Index freshness and staleness reporting

The semantic index is backed by a manifest at
`.loki/state/code-index-manifest.json`. The MCP tools (`loki_code_search` and
`loki_code_search_stats`) compare the manifest against the files on disk and report
two fields in their JSON output:

- `stale`: boolean, true when one or more indexed files have changed since the last
  index
- `stale_files`: count of changed files

Staleness is computed from the manifest alone (no ChromaDB call), and a missing
manifest degrades to not-stale so a fresh repo is unaffected.

Default behavior is warn-if-stale: the tools report staleness but do not re-index.
Set `LOKI_CODE_INDEX_AUTOREINDEX=1` to opt into an automatic incremental re-index
before querying (off by default because embeddings cost compute). The incremental
re-index is driven by `tools/index-codebase.py --changed`, which re-chunks only
files whose mtime or sha1 differ from the manifest, upserts the new chunks, deletes
orphaned chunk IDs for changed files, and drops chunks for files removed from disk.

### CLI usage: `loki code search`

```bash
loki code search "rate limit backoff"            # hybrid grep + semantic
loki code search "council vote" --grep-only      # lexical only
loki code search "memory retrieval" --semantic-only --top 15
loki code search "build prompt" --budget 4000    # widen the token budget
loki code search "save state" --json             # machine-readable output
```

Flags (see `loki code search --help`):

| Flag | Default | Effect |
|------|---------|--------|
| `--grep-only` | off | lexical search only (skip semantic) |
| `--semantic-only` | off | semantic search only (skip grep) |
| `--budget N` | 3000 | token budget for the merged result set |
| `--top N` | 10 | maximum number of results |
| `--json` | off | emit JSON instead of formatted output |

`loki code search` is one of the `loki code` codebase-intelligence subcommands
(alongside `overview`, `symbols`, `deps`, `hotspots`, and `diff`). It falls back to
grep-only when ChromaDB is unreachable, and requires python3.12 for the semantic
path because that is what the chromadb client needs on this stack.

---

## MCP Configuration Location

Claude Code reads MCP configuration from:

1. **Project-level:** `.claude/mcp.json` (recommended for project-specific tools)
2. **User-level:** `~/.claude/mcp.json` (for global tools)

Example full configuration:
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/playwright-mcp"]
    },
    "parallel-search": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/parallel-search-mcp"],
      "env": {
        "PARALLEL_API_KEY": "${PARALLEL_API_KEY}"
      }
    },
    "repowise": {
      "command": "repowise",
      "args": ["mcp", "--path", "."]
    }
  }
}
```

---

## Using MCP Tools in Loki Mode

MCP tools are automatically available to Claude Code when configured. The orchestrator can dispatch agents that use these tools:

```python
# Agent using Playwright for E2E verification
Task(
    subagent_type="general-purpose",
    model="sonnet",
    description="Verify login feature visually",
    prompt="""
    Use Playwright MCP to:
    1. Navigate to http://localhost:3000/login
    2. Fill in test credentials
    3. Click login button
    4. Take screenshot of dashboard
    5. Verify user name is displayed
    """
)

# Agent using Parallel AI for research
Task(
    subagent_type="general-purpose",
    model="opus",
    description="Research competitor pricing",
    prompt="""
    Use Parallel AI Task API to:
    1. Research top 5 competitors in [market]
    2. Extract pricing tiers from each
    3. Return structured comparison table

    Output schema: {competitors: [{name, tiers: [{name, price, features}]}]}
    """
)
```

---

## When NOT to Use MCP

- **Simple searches:** Claude's built-in `WebSearch` is sufficient for basic lookups
- **Cost sensitivity:** MCP tools add API costs on top of Claude costs
- **Offline work:** MCP tools require network access

---

## Adding New MCP Servers

When evaluating new MCP servers for Loki Mode integration, assess:

1. **Autonomous fit:** Does it work without human intervention?
2. **Evidence quality:** Does it provide verifiable, citable results?
3. **SDLC alignment:** Which phase(s) does it enhance?
4. **Cost model:** Predictable pricing for autonomous operation?
5. **Error handling:** Does it fail gracefully?

---

## References

- [MCP Specification](https://modelcontextprotocol.io/)
- [Parallel AI Documentation](https://docs.parallel.ai/)
- [Playwright MCP](https://github.com/anthropics/anthropic-quickstarts/tree/main/mcp-playwright)
- [Repowise](https://repowise.dev/)
