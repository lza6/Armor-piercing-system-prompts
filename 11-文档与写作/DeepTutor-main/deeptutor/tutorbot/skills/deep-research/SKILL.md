---
name: deep-research
description: "Multi-agent research reports on any topic."
metadata: {"nanobot":{"emoji":"🔬","requires":{"bins":["deeptutor"]}}}
always: false
---

# Deep Research

Use the `exec` tool to invoke DeepTutor's multi-agent research pipeline for comprehensive topic analysis.

## When to Use

- User needs a **research report** or **literature review**
- User wants a **comparison** of approaches, frameworks, or technologies
- User asks for a **learning path** or **structured overview** of a field
- The topic requires aggregating information from multiple sources

## Command

```bash
deeptutor run deep_research "<topic>" --format json -l <lang> --config-json '<json>'
```

### Config JSON Fields

| Field | Values | Description |
|-------|--------|-------------|
| `mode` | `notes`, `report`, `comparison`, `learning_path` | Output format |
| `depth` | `quick`, `standard`, `deep` | Research thoroughness |
| `sources` | `["kb", "web", "papers"]` | Information sources to use |

## Examples

Standard report from web + papers:
```bash
deeptutor run deep_research "Recent advances in attention mechanisms" --format json -l en --config-json '{"mode":"report","depth":"deep","sources":["papers","web"]}'
```

Quick comparison:
```bash
deeptutor run deep_research "Rust vs Go for backend services" --format json -l en --config-json '{"mode":"comparison","depth":"quick","sources":["web"]}'
```

Learning path from knowledge base:
```bash
deeptutor run deep_research "Machine learning fundamentals" --format json -l zh --config-json '{"mode":"learning_path","depth":"standard","sources":["kb"]}' --kb ml-textbook
```

KB + web hybrid:
```bash
deeptutor run deep_research "Agentic RAG vs traditional RAG" --format json -l zh --config-json '{"mode":"comparison","depth":"deep","sources":["kb","web"]}' --kb my-papers
```

## Important

- **All three config fields (`mode`, `depth`, `sources`) are required.** Always use `--config-json` to pass them together.
- `depth=deep` can take **several minutes** — use `timeout=300` or higher with the `exec` tool.
- Always pass `--format json` to get parseable NDJSON output.
- Parse lines with `"type": "content"` and concatenate for the full report.
