# Audit Context Reference

This audit verifies the `process/context/` durable knowledge layer.

## Audit Scope

Check:

- Root router: `process/context/all-context.md`
- Group entrypoints: `process/context/*/all-*.md`
- Current context docs: `process/context/**/*.md`
- Claude agents: `.claude/agents/*.md`
- Codex agents: `.codex/agents/*.toml`
- Shared skills: `.claude/skills/*/SKILL.md`
- Codex discovery path: `.agents/skills`
- Context validator scripts under `.claude/skills/vc-audit-context/scripts/`

## Classification

Classify findings as:

- **Broken reference**: Concrete `process/context/...` path does not exist.
- **Unindexed context**: Context file exists but is not listed by the root router or its owning `all-*.md` entrypoint.
- **Missing group entrypoint**: Directory under `process/context/` lacks `all-{group}.md`.
- **Legacy entrypoint drift**: old root or group README-style context entrypoints still exist.
- **Agent routing gap**: Critical agent still bypasses `process/context/all-context.md`.
- **Skill discovery gap**: Skill exists in `.claude/skills` but is not reachable through `.agents/skills`.
- **Skill routing gap**: Skill exists but is neither routed from canonical surfaces nor explicitly allowlisted.
- **Skill schema warning**: Skill frontmatter contains non-Codex system keys or discovery metadata drift.
- **Skill cross-ref failure**: Skill references a missing local `references/`, `scripts/`, `templates/`, or `assets/` path.
- **Skill dependency warning**: Skill references another skill in a way that creates avoidable cycles or confusing dependency chains.
- **Confusable skill warning**: Two skill names are close enough to confuse routing or discovery.
- **Agent parity gap**: Claude and Codex agent definitions do not have matching names.
- **Agent content drift**: Claude and Codex agent definitions exist but their normalized instructions differ.
- **Stale workflow alias**: Old documentation-manager, plan-manager, root-docs, or legacy docs-validation routes remain in active agent/skill instructions.
- **Migration risk**: File is large or heavily referenced and should use wrapper-based migration.

## Required Checks

1. Confirm `.agents/skills` resolves to `.claude/skills`.
2. Confirm `audit-context` exists at both:
   - `.claude/skills/vc-audit-context/SKILL.md`
   - `.agents/skills/vc-audit-context/SKILL.md`
3. Confirm `process/context/all-context.md` indexes every current context `.md` file.
4. Confirm each context group directory has a canonical `all-{group}.md` entrypoint.
5. Confirm critical Claude/Codex surfaces mention `process/context/all-context.md`:
   - `AGENTS.md`
   - `CLAUDE.md`
   - `.claude/agents/vc-update-process-agent.md`
   - `.codex/agents/update-process-agent.toml`
   - `.claude/agents/vc-research-agent.md`
   - `.codex/agents/research-agent.toml`
6. Confirm concrete backticked `process/context/...` references exist.
7. Confirm every skill folder is reachable through `.agents/skills`.
8. Confirm Claude and Codex agent names are mirrored.
9. Confirm active workflow instructions do not route to stale documentation-manager, plan-manager, root-docs, or legacy docs-validation flows.
10. Confirm every kept shared skill is either routed from the canonical surfaces in `skill-routing-policy.json` or explicitly allowlisted there with a reason.

## Validators

Run the full validator suite during a context/skill-quality audit:

```bash
node .claude/skills/vc-audit-context/scripts/validate-context-discovery.mjs
node .claude/skills/vc-audit-context/scripts/validate-skill-routing.mjs
node .claude/skills/vc-audit-context/scripts/validate-skill-cross-refs.mjs
node .claude/skills/vc-audit-context/scripts/validate-skill-dependencies.mjs
node .claude/skills/vc-audit-context/scripts/validate-confusable-skills.mjs
node .claude/skills/vc-audit-context/scripts/generate-skills-catalog.mjs --check
```

For agent/skill harness validators (agent parity, skill frontmatter, README.md sync, protocol wiring), see the `audit-vc` skill:

```bash
node .claude/skills/vc-audit-vc/scripts/validate-agent-parity.mjs
node .claude/skills/vc-audit-vc/scripts/validate-skills.mjs
node .claude/skills/vc-audit-vc/scripts/validate-guide-sync.mjs
node .claude/skills/vc-audit-vc/scripts/validate-protocol-wiring.mjs
```

Interpret `failures` as required fixes. Interpret `warnings` as audit findings that may be
acceptable for legacy or upstream-style skills unless the user asks for strict cleanup.

The generated catalog lives at:

- process/context/generated-skills-catalog.json

It is deterministic and should be regenerated whenever shared skill inventory, routing, or allowlist policy changes.

## Output Format

Return:

```markdown
## Context Audit Summary

| Check | Status | Notes |
|---|---|---|
| Router exists | PASS/FAIL | ... |
| Context files indexed | PASS/FAIL | ... |
| Group entrypoints | PASS/FAIL | ... |
| Claude/Codex wiring | PASS/FAIL | ... |
| Skill discovery | PASS/FAIL | ... |
| Agent parity | PASS/FAIL | ... |
| Broken refs | PASS/FAIL | ... |

### Actions Taken
- ...

### Decisions Needed
- ...
```

## Migration Guidance

When moving docs:

1. Move one group at a time.
2. Leave wrappers for root files with many references.
3. Update router and group `all-*.md` entrypoint in the same patch.
4. Update direct references in agents, skills, plans, and context docs.
5. Run validator after every move.
