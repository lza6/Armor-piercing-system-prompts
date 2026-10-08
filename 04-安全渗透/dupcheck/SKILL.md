---
name: dupcheck
description: "Check if a vulnerability has already been reported. Searches platform hacktivity + local findings. Usage: /dupcheck <vuln_type> e.g. /dupcheck XSS in search endpoint"
disable-model-invocation: false
---
Check for duplicate reports: $ARGUMENTS

1. Determine the platform and program from `scope.yaml` in the current directory.
2. Use `bounty-platforms` MCP tool `search_hacktivity` with platform, program, and "$ARGUMENTS" as the query.
3. Also search local findings: `uv run python3 $CLAUDE_PROJECT_DIR/tools/dedup_findings.py --stats --db findings.json`
4. Read `hacktivity.md` if it exists and grep for related terms.
5. Report:
   - Exact or near matches from hacktivity (potential duplicates)
   - Related reports that might overlap
   - If the area appears heavily reported (high duplicate risk)
   - Verdict: likely unique, possible duplicate, or high duplicate risk

## Writeup Cross-Reference (if writeup-search MCP is available)

After checking local findings, also search the writeup database:
- Use `search_writeups` MCP tool with "<finding description> <target>"
- If similar writeups exist, assess whether your finding is novel or a known pattern
- Mention relevant prior art in the "Known Techniques" section of the report

## Top-Tier Duplicate Analysis

Duplicate risk is about overlap of exploit primitive and affected asset, not keyword similarity.

Report four verdict fields:
- `same_asset_same_primitive`: likely duplicate unless your impact is strictly stronger
- `same_primitive_different_asset`: possible duplicate; explain scope difference and novelty
- `same_asset_different_primitive`: usually unique; prove a different root cause
- `known_class_new_chain`: often worth reporting if the chain reaches a new impact tier

Check disclosed writeups for patch language and response tone. If triagers historically close this class as N/A, require chain proof before submission. If public reports stop at a weaker impact, frame your report around the new capability, not the shared first step.
