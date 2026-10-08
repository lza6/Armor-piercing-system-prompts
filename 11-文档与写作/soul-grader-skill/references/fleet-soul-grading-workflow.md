# Fleet SOUL grading workflow

Use this reference when the user asks to grade multiple SOUL.md files across a Hermes fleet, including knwldg's own SOUL.

## Trigger

- “grade all the souls in the fleet”
- “audit fleet SOULs”
- “review deployed agent identities”
- “compare SOUL quality across agents”

## Workflow

1. Load `references/soul-md-grading-standard.md` before grading. It remains the normative source.
2. Read the roster and active manifests to determine active rostered Hermes agents, retired agents, and non-Hermes services.
3. Collect only active rostered Hermes `SOUL.md` files for the active-fleet headline.
4. Include the current agent’s own SOUL when the user asks for the whole fleet.
5. Treat retired/archive entries separately. Do not open archived material if manifests warn it may contain secrets; exclude retired agents from the active-fleet average unless the user explicitly asks for historical archive grading.
6. Classify non-Hermes services such as FastAPI/MCP control-plane services as `N/A / no SOUL` unless they have an LLM-facing identity artifact.
7. Scan collected SOULs for common secret/token/private-key shapes before quoting or reporting.
8. Grade each SOUL using all 11 rubric rows before classifying scope. Award placement credit when operating detail is correctly left to companion docs; never award duplication credit.
9. Optionally collect companion `AGENTS.md` / `CLAUDE.md` files for contradiction, placement, and system-blocker evidence only. Do not grade those files as part of the SOUL score. If unavailable, mark system blockers not assessed.
10. Flag unrostered live profiles as deployability anomalies. A generic/default live profile with unknown scope and no manifest/roster entry is not deployable as a fleet identity.
11. Write a durable report under the corpus, preferably `docs/research/YYYY-MM-DD-fleet-soul-grades.md`, and validate it with a secret-shape scan before finalizing.

## Reporting shape

Include:

- scope and method
- normative source path
- collection artifact path, if any
- secret-hygiene statement
- active-fleet headline and average
- score table
- category breakdown table
- companion-doc placement/contradiction check, or `not assessed`
- per-agent findings
- non-scored/excluded entries
- fleet-level priorities
- suggested replacement snippets for repeated issues

## Pitfalls

- Do not average retired agents into the active fleet.
- Do not print archived secret-bearing files.
- Do not treat `knwldg-mcp`-style services as SOUL-bearing Hermes profiles unless an identity artifact exists.
- Do not let subagent failures stop the task; if delegated graders fail, continue manually from the loaded rubric and collected SOUL files.
- Do not stop after a chat summary. Produce a durable report and validate it.
