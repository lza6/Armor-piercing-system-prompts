# Phase-coupled SOUL refresh

Use this reference when a deployed agent's `SOUL.md` was written for an earlier phase and now fights the current product/workflow state.

## Trigger signals

- The user says the SOUL is "too tied to Phase 0" or another old phase.
- A grade is acceptable numerically but the identity is stale for current work.
- Current ops docs/build state have advanced, but SOUL/CLAUDE/AGENTS/manifest still describe a bootstrap/intake role.
- The agent's product mechanism is failing and the identity should steer toward quality gates/taste loops rather than checklist collection.

## Procedure

1. Load the normal `soul-grader` sources and grade the live SOUL first.
2. Read the agent design doc plus the current build/phase docs, not just the original intake doc.
3. Identify what should be durable identity versus volatile phase/runbook state.
4. Rewrite SOUL around the durable role and current decision lens:
   - name the real product loop;
   - add positive role ownership and operator/peer boundaries;
   - add a domain-specific core thesis;
   - add ranked optimization priorities;
   - define identity-level truthfulness and success orientation;
   - keep distinctive voice only where safe.
5. Patch companion `CLAUDE.md` / `AGENTS.md` if they contradict the new SOUL or need the detailed approval, evidence, verification, escalation, or completion policy removed from SOUL. Do not grade those companion files as part of the SOUL score.
6. Record durable evidence:
   - timestamped backup paths;
   - target files changed;
   - secret scan result;
   - fresh soul-grader score;
   - companion-doc drift status;
   - whether a live gateway/session restart was intentionally not performed.
7. Update the fleet roster/manifest when the status or phase changed. Do not leave roster/manifest claiming the old phase after refreshing identity.

## Pitfalls

- Do not turn SOUL into the phase plan. Point to `docs/ops/*`, manifests, or memory/session docs for current operational state.
- Do not invent access, publishing authority, credentials, health, or connector success to make the new phase sound more mature.
- Do not restart live client/business gateways just to reload identity unless the user/operator approved that operational action.
- Do not stop after a good SOUL score if adjacent files still contradict it; either patch the adjacent docs or report the drift as unresolved.

## Example evidence shape

```md
- Rewritten SOUL: `/path/to/SOUL.md`
- Companion docs patched: `/path/to/CLAUDE.md` (`AGENTS.md` symlink)
- Backup: `/path/to/SOUL.md.bak-YYYYMMDDTHHMMSSZ`
- Grade: `98/100`, verdict `Excellent`, deployability `Approved`
- Secret scan: `PASS`
- Roster/manifest: updated from old phase status to current phase status
- Runtime note: gateway active; no restart performed; fresh session/restart may be needed to reload identity
```
