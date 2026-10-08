# Competitive Analysis: Loki vs ralph-tui + zeroshot (2026-07-08, honest)

Source: fanned-out research agent, verified against competitor repos/docs + the
Loki repo. Lead with the uncomfortable truths.

## The uncomfortable truth
zeroshot (github.com/the-open-engine/zeroshot) has the SAME trust thesis as Loki
-- "the verifier stays independent from the model that wrote the code": blind
validators, fail-closed gates, actionable rejections, complexity tiers, SQLite
resume -- implemented MORE LEGIBLY and more focused. Loki is NOT uniquely "the
verification tool." Claiming blind-review / independent-verifier as a Loki-only
differentiator would be fabrication (the founder's #1 forbidden outcome).

ralph-tui (github.com/subsy/ralph-tui) is a different animal: a terminal-UI
orchestrator for Geoff Huntley's "Ralph" loop (`while :; do cat PROMPT.md |
claude-code; done`). It beats Loki on ERGONOMICS + SIMPLICITY + community
momentum, NOT verification (Ralph pushes verification onto the operator).

## What they do BETTER than Loki (honest)
- zeroshot: trust story is TIGHTER + more legible ("blind validation + fail-closed
  + actionable rejections" = one sentence a skeptic trusts). Loki's 8+3+1 gates +
  council + RARV-C is more thorough but far harder to explain/audit.
- zeroshot: EVIDENCE STALENESS -- fails closed when evidence is stale / older than
  IMPLEMENTATION_READY / lacks usable evidence, with published status/timestamp/
  command/exitCode/output. Better guarantee than Loki's "gates passed."
- zeroshot: COMPLEXITY-PROPORTIONAL validators (TRIVIAL=0, SIMPLE=1, STANDARD=2,
  CRITICAL=5) -- only pays for verification the task warrants. Cost/speed win.
- zeroshot: per-run ISOLATION dial (none/worktree/--docker); native GitLab/Jira/
  Azure backends; honest scope discipline ("not for 'make it faster'").
- ralph-tui: the TUI cockpit, multi-machine WebSocket fleet control, radical
  legibility, 7 agent CLIs, viral community (awesome-ralph, ~2.4k stars).

## What Loki genuinely has that they LACK (verified, not overclaimed)
- Spec-to-DEPLOYED-PRODUCT scope (both competitors stop at task/issue level).
- Persistent cross-project MEMORY (episodic/semantic/procedural + RAG). Neither
  has a learning layer (zeroshot's SQLite is run-state, not learned knowledge).
- Anti-sycophancy Devil's-Advocate re-review triggered by unanimity.
- Semantic anti-fake-green DETECTORS (mock-integrity, test-mutation) -- dedicated
  deterministic catchers for tautological/fitted tests.
- Legacy-healing mode (brownfield archaeology/stabilize/modernize).
NOTE: breadth is also a LIABILITY -- spec-to-product implies Loki handles
everything, a bigger promise; an over-confident run that fabricates progress on an
ill-defined task is the WORST outcome for a trust brand.

## Honest gaps (where a user would prefer a competitor)
1. Legibility: a skeptic can trust zeroshot/Ralph in minutes; Loki's surface is
   hard to audit. A trust product that is itself hard to trust-verify is a real
   problem.
2. Cost/speed proportionality: zeroshot scales 0-5 validators; Loki runs a lot
   for every change.
3. Terminal ergonomics: ralph-tui's cockpit.
4. Isolation ergonomics: zeroshot's clean --docker dial.
5. Community/momentum: Loki is comparatively invisible in the public conversation.

## Prioritized recommendations (impact-ranked; moat-safe)
1. [HIGHEST, S-M, zero moat cost] Make the trust layer LEGIBLE + PROVABLE in one
   screen: a one-sentence "why you can trust this" + `loki verify --explain` that
   prints exactly which gates ran, their evidence, freshness, pass/fail. Loki has
   MORE verification than zeroshot but COMMUNICATES/PROVES it worse. This
   STRENGTHENS the moat by making it demonstrable.
2. [M, moat REINFORCEMENT] Evidence STALENESS + tool-neutral handoff contract:
   timestamp gate evidence, invalidate on state change, fail closed when stale /
   older than the ready-marker / lacks usable evidence.
3. [M, careful] Complexity-proportional verification: wire detect_complexity() to
   scale the gate/council battery like zeroshot's 0/1/2/5 -- but ONLY ADD
   validators for hard tasks, NEVER drop below a hard floor for anything
   shippable. Cost/speed win with the floor as the guard.
4. [M, no moat cost] Per-run isolation dial incl. Docker (match none/worktree/docker).
5. [S, moat reinforcement] Scope honesty: when "done" is undescribable, say so +
   degrade to inconclusive (never fabricate). Make it first-class + visible.
6. [L, nice-to-have] A real terminal cockpit / strong TUI status view.
7. [M] Broaden issue backends (GitLab/Jira/Azure) for team parity.

## Single most important move + the one thing NOT to copy
- DO: make the trust layer provable to a skeptic in 60 seconds (recs 1+2). Loki
  likely has more verification than zeroshot but demonstrates it worse; in a
  market where buyers are skeptical, the tool that PROVES trust fastest wins.
- DO NOT COPY: Ralph's "operator supplies verification / senior babysits / ~90%
  then human takes over." It is simpler + viral but the OPPOSITE of Loki's thesis.
  Pushing verification back onto the user deletes the moat to chase a crowd that
  was never Loki's.
