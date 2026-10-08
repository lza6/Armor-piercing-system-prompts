---
name: vc-scout
description: "Fast codebase scouting using shell search and optional parallel research agents. Use for file discovery, task context gathering, and quick scoped searches across directories."
argument-hint: "[search-target] [ext]"
trigger_keywords: find files, where is, search codebase
layer: helper
metadata:
  author: claudekit
  version: "1.0.0"
---

# Scout

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Fast, token-efficient codebase scouting using parallel agents to find files needed for tasks.

## Arguments
- Default: Scout using local shell search plus optional parallel `research-agent` delegation (`./references/internal-scouting.md`)
- `ext`: Scout using external Gemini/OpenCode CLI tools in parallel (`./references/external-scouting.md`)

## When to Use

- Beginning work on feature spanning multiple directories
- User mentions needing to "find", "locate", or "search for" files
- Starting debugging session requiring file relationships understanding
- User asks about project structure or where functionality lives
- Before changes that might affect multiple codebase parts

## Quick Start

1. Analyze user prompt to identify search targets
2. Use a wide range of Grep and Glob patterns to find relevant files and estimate scale of the codebase
3. Use local shell search first, then optionally spawn parallel `research-agent` workers with divided directories when the search space is large
4. Collect results into concise report

## Configuration

Read from `.claude/.vc.json` (falls back to legacy `.claude/.ck.json` if present):
- `gemini.model` - Gemini model (default: `gemini-3-flash-preview`)

## Workflow

### 1. Analyze Task
- Parse user prompt for search targets
- Identify key directories, patterns, file types, lines of code
- Determine optimal SCALE value of subagents to spawn

### 2. Divide and Conquer
- Split codebase into logical segments per agent
- Assign each agent specific directories or patterns
- Ensure no overlap, maximize coverage

### 3. Register Scout Tasks
- Parallel task registration and external orchestration patterns are optional, not required for normal scouting in this repo
- See `references/task-management-scouting.md` only when you are intentionally coordinating a larger parallel scout workflow

### 4. Spawn Parallel Agents
Load appropriate reference based on decision tree:
- **Internal (Default):** `references/internal-scouting.md` (shell search plus optional `research-agent` parallelism)
- **External:** `references/external-scouting.md` (Gemini/OpenCode)

**Notes:**
- `TaskUpdate` each task to `in_progress` before spawning its agent (skip if Task tools unavailable)
- Prompt detailed instructions for each subagent with exact directories or files it should read
- Remember that each subagent has less than 200K tokens of context window
- Amount of subagents to-be-spawned depends on the current system resources available and amount of files to be scanned
- Each subagent must return a detailed summary report to a main agent

### 5. Collect Results

- Timeout: 3 minutes per agent (skip non-responders)
- `TaskUpdate` completed tasks; log timed-out agents in report (skip if Task tools unavailable)
- Aggregate findings into single report
- List unresolved questions at end

## Report Format

```markdown
# Scout Report

## Relevant Files
- `path/to/file.ts` - Brief description
- ...

## Unresolved Questions
- Any gaps in findings
```

## References

- `references/internal-scouting.md` - Using shell search and optional parallel `research-agent` work
- `references/external-scouting.md` - Using Gemini/OpenCode CLI
- `references/task-management-scouting.md` - Optional task-registration patterns for larger scout coordination
