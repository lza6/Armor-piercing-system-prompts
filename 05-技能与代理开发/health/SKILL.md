---
name: health
description: Invoke when Claude ignores instructions, behaves inconsistently, hooks malfunction, or MCP servers need auditing. Audits the full six-layer config stack and flags issues by severity. Not for debugging code or reviewing PRs.
metadata:
  version: "3.8.0"
---

# Claude Code Configuration Health Audit

Prefix your first line with 🥷 inline, not as its own paragraph.


Audit the current project's Claude Code setup with the six-layer framework:
`CLAUDE.md → rules → skills → hooks → subagents → verifiers`

The goal is to find violations and identify the misaligned layer, calibrated to project complexity.

**Output language:** Check in order: (1) CLAUDE.md `## Communication` rule (global takes precedence over local); (2) language of the user's recent conversation messages; (3) default English. Apply the detected language to all output.

Keep the user informed of progress through the three steps: data collection, analysis, and synthesis.

## Step 0: Assess project tier

Pick tier:

| Tier | Signal | What's expected |
|------|--------|-----------------|
| **Simple** | <500 project files, 1 contributor, no CI | CLAUDE.md only; 0–1 skills; no rules/; hooks optional |
| **Standard** | 500–5K project files, small team or CI present | CLAUDE.md + 1–2 rules files; 2–4 skills; basic hooks |
| **Complex** | >5K project files, multi-contributor, multi-language, active CI | Full six-layer setup required |

**Apply only the detected tier's requirements.**


## Step 1: Collect all data

Run `bash "${CLAUDE_SKILL_DIR:-$HOME/.agents/skills/health}/scripts/collect-data.sh"` to collect all configuration data. The script outputs labeled sections covering: tier metrics, CLAUDE.md (global + local), settings/hooks/MCP, rules, skill inventory, context budget, conversation history, conversation signals, and skill security content.

## Step 1b: MCP Live Check

After the bash block completes, for each MCP server listed in the settings, attempt to call it and verify it actually responds. Do this before launching analysis agents.

For each server name found in Step 1:
1. Call any known tool from that server with minimal input (e.g., a search tool with a trivial query, or a list/read tool on a known path). If no tool name is known, attempt the server's first listed tool with safe default arguments.
2. If the call succeeds: mark `live=yes`.
3. If it fails or times out: mark `live=no`, note the error.

Record the result as a table:

```
MCP Live Status:
  server_name    live=yes  (N tools available)
  other_server   live=no   error: connection refused / tool not found / API key invalid
```

Pass this table to Agent 1 for inclusion in the MCP findings section.

**If API keys are required**: look for relevant env var names in the server config (e.g., `XCRAWL_API_KEY`, `OPENAI_API_KEY`). Do not attempt to validate the key value itself -- just note whether the env var is set: `echo $VAR_NAME | head -c 5` (5 chars only, do not print the full key).

## Gotchas

Before interpreting Step 1 output, check these known failure modes.

**Data collection silent failures**
- `jq` not installed: conversation extract and conversation signals print `(unavailable: jq not installed or parse error)`. BEHAVIOR and conversation-based context checks will be empty -- treat as [INSUFFICIENT DATA], not a finding.
- `python3` not on PATH: all MCP/hooks/allowedTools sections print `(unavailable)`. Do not flag those areas when the data source itself failed.
- `settings.local.json` absent: hooks, MCP, and allowedTools all show `(unavailable)`. Normal for projects using global settings only -- not a misconfiguration.

**MEMORY.md path construction**
- Path built with `sed 's|[/_]|-|g'` on `pwd`. Unusual characters produce the wrong project key. If MEMORY.md shows `(none)` but the user mentions prior sessions, verify the path manually before flagging as [!].

**Conversation extract scope**
- Only the 2 most recent `.jsonl` files are sampled, skipping the active session. Findings from fewer than 2 files carry low signal, always tag [LOW CONFIDENCE].

**MCP token estimate**
- Assumes ~25 tools/server and ~200 tokens/tool. Servers with many or few tools cause large over/under-estimates. Treat as directional, not precise.

**Tier misclassification edge cases**
- The bash block excludes `node_modules/`, `dist/`, and `build/`, but not all generators. Monorepos with `.next/`, `__pycache__/`, or `.turbo/` output can inflate the file count and trigger COMPLEX tier falsely. Recheck manually if the tier feels wrong.

## Step 2: Analyze with tier-adjusted depth

Summarize what was collected (word counts, skills found, conversation files sampled), confirm the tier, then proceed:

- **SIMPLE:** Analyze locally from Step 1 data. Do not launch subagents. Prioritize core config checks; skip conversation cross-validation unless evidence is obvious.
- **STANDARD/COMPLEX:** Launch two subagents in parallel with the collected data pasted inline. Do not pass file paths. Before pasting, replace any credential values (API keys, tokens, passwords) with `[REDACTED]`.

**Fallback:** If either subagent fails (API error, timeout, or empty result), do not abort. Analyze that layer locally from Step 1 data instead and note "(analyzed locally -- subagent unavailable)" in the affected section of the report.

### Agent 1 -- Context + Security Audit (uses conversation signals only)

Read `agents/inspector-context.md` from this skill's directory. It specifies which Step 1 sections to paste and the full audit checklist. Paste `CONVERSATION SIGNALS`, not the full `CONVERSATION EXTRACT`, so Agent 1 can inspect enforcement gaps and context pressure without duplicating the heaviest evidence block.

### Agent 2 -- Control + Behavior Audit (uses conversation evidence)

Read `agents/inspector-control.md` from this skill's directory. It specifies which Step 1 sections to paste and the full audit checklist.

## Step 3: Synthesize and present

Aggregate the local analysis and any agent outputs into one report:

---

**Health Report: {project} ({tier} tier, {file_count} files)**

### [PASS] Passing

Render a compact table of checks that passed. Include only checks relevant to the detected tier. Limit to 5 rows. Omit rows for checks that have findings.

| Check | Detail |
|-------|--------|
| settings.local.json gitignored | ok |
| No nested CLAUDE.md | ok |
| Skill security scan | no flags |

### [!] Critical -- fix now

Rules violated, missing verification definitions, dangerous allowedTools, MCP overhead >12.5%, required-path `Access denied`, active cache-breakers, and security findings.

### [~] Structural -- fix soon

CLAUDE.md content that belongs elsewhere, missing hooks, oversized skill descriptions, single-layer critical rules, model switching, verifier gaps, subagent permission gaps, and skill structural issues.

### [-] Incremental -- nice to have

New patterns to add, outdated items to remove, global vs local placement, context hygiene, HANDOFF.md adoption, skill invoke tuning, and provenance issues.

---

If all three issue sections are empty, output one short line in the output language like: `All relevant checks passed. Nothing to fix.`

## Non-goals
- Never auto-apply fixes without confirmation.
- Never apply complex-tier checks to simple projects.
- Flag issues, do not replace architectural judgment.


**Stop condition:** After the report, ask in the output language:
> "Should I draft the changes? I can handle each layer separately: global CLAUDE.md / local CLAUDE.md / rules / hooks / skills / MCP."

Do not make any edits without explicit confirmation.
