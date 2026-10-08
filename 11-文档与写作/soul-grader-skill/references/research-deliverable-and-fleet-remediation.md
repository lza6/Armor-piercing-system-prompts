# SOUL.md research deliverables and fleet remediation notes

Use this reference when a SOUL.md task is bigger than a one-file grade: field-guide research, fleet-wide SOUL audits, live remediation, or user-facing review artifacts.

## Research artifact pattern

For broad SOUL.md research requests, produce three layers rather than only a chat answer:

1. **Canonical standard** — the durable rubric/field-guide reference that future graders can reuse.
2. **Evidence ledger** — examples, counterexamples, loaded docs, and fleet observations that justify the standard.
3. **Review surface** — a polished human-readable artifact, usually static HTML, that the user can skim visually over Tailscale/local web access.

The HTML artifact should be self-contained when practical: embedded CSS, readable typography, anchor navigation, verdict cards, score tables, before/after examples, and an explicit source-basis section. Do not require a build pipeline for a research report unless the user asked for an app.

## Sub-agent swarm pattern

When the user asks to “go deep” or “launch a swarm,” split work into independent research lanes, then consolidate into the class-level standard:

- **Hermes semantics lane** — what Hermes actually loads/injects, and what belongs in SOUL vs AGENTS/CLAUDE/skills/memory/manifests.
- **Quality/rubric lane** — mission, identity, positive role boundaries, judgment, relationship posture, success orientation, placement, and system-level blocker rules.
- **Fleet/examples lane** — review existing agent SOUL files for patterns, contradictions, and high-leverage remediation.
- **Deliverable lane** — turn the findings into a polished static artifact with navigation and clear decisions.

Have each lane return evidence and actionable rules, not free-floating opinions. Consolidate conflicts explicitly before writing the canonical reference.

## Fleet remediation pattern

If research turns into live SOUL edits:

1. Verify roster/manifest status and the actual profile/workspace identity paths.
2. Back up every target file before edits.
3. Preserve SOUL as the stable identity and judgment layer; move approval matrices, escalation procedures, verification checklists, startup, runbooks, volatile state, and long workflows into `AGENTS.md` / `CLAUDE.md`, skills, manifests, or system controls.
4. For unrostered profile stubs, archive before removal when any state/history may matter; verify no gateway or service depends on the profile.
5. Use real profile-load or chat smokes where credentials permit. If a provider token is expired, report that as an auth blocker rather than treating the SOUL edit as unverified.
6. Record durable outcomes in the research note and affected manifests, and scan edited identity docs for accidental secret shapes.

## Tailscale/local review surface

When the user asks for a file they can view over Tailscale:

- Prefer a static HTML report for rich research output.
- Put durable research copies under the corpus, typically `docs/research/`, and downloadable/viewable convenience copies under the active profile cache when needed.
- Include the exact path and, when served, the URL in the final response.
- If using Tailscale Serve or a local server, verify the server is actually reachable before claiming it is viewable.

## Anti-patterns observed

- Long SOUL files packed with startup rituals, exact service state, provider commands, or current workflow notes.
- Generic values lists without positive ownership boundaries or a decision lens.
- Treating “friendly/proactive/helpful” as sufficient voice guidance.
- Reporting a remediation as complete without distinguishing identity-file success from unrelated provider-auth failures.
