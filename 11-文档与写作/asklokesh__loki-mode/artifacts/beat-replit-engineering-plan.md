# Beat Replit: Structural Generation-Quality Program

Owner: Autonomi. Status: build-ready plan (not yet implemented). Date: 2026-07-04.
Evidence base: `artifacts/replit-adopt-study.md` (hand-verified study of Replit's
generated "Adopt" app) + BQ-1 diagnosis (task #11) + FV-1 harness (task #8).

## The honest thesis

Replit ships a real, wired, designed full-stack app because **a contract file
exists as a build artifact and everything else is mechanically generated from
it.** Autonomi ships static shells because it hand-writes disconnected files
with no contract, no real-backend floor, and no design/completeness direction.

**The fix is STRUCTURAL, not a better prompt.** You cannot prompt your way to
contract-first codegen, a real DB tier, or a design system. This is a multi-week
program of engine scaffolding, sequenced below.

Measured baseline (honest "before", 2026-07-04):
- `loki plan` on the AI Adoption Portal PRD: **2.4 hr (1.4-3.4 hr), 41 iterations**.
- Replit's Adopt (the target): **8 pages, 11 API routes, 12 DB tables**, real
  Express 5 + Postgres/Drizzle + OpenAPI->Orval codegen, brand-derived design
  tokens, skeleton/empty/first-run states.
- Founder 10x targets: accuracy 70%->90%+, cost ~$10->~$1, speed ~10min->2-3min.

## The two hard lines (re-drawn, do not cross)

**Gap A (IN SCOPE) = the generated app's OWN quality.** Contract-first codegen,
a real Express+Drizzle+Postgres backend generated *into the app*, a design
system, UI states. This is what we build.

**Gap B (OUT OF SCOPE) = Replit's managed CLOUD platform.** Hosted/managed
Postgres provisioning, managed Auth, Secrets vault, autoscale deploy infra,
supply-chain quarantine-as-a-service. This is a multi-quarter PaaS program.
A generated-into-the-app Drizzle/Postgres data layer is Gap A; *provisioning and
operating* that database as a platform is Gap B. Draw the line here every time.

**Anti-teaching-to-the-test test (founder rule: no one-off wins, no fabrication).**
Every change must pass: *"Would this help an ARBITRARY full-stack PRD, or only
the Adoption Portal?"* A reusable Express+Drizzle+OpenAPI scaffold PASSES (general
capability). Anything referencing adoption/leaderboard/connectors, or validated
by re-running THIS PRD, FAILS. Verify each slice on a THROWAWAY generic spec
(e.g. a todo API, a bookmarks app) first, never on the Adoption PRD.

## The program (dependency-ordered milestones)

### M1 -- Contract-first codegen spine  [highest leverage; the missing spine]
The single load-bearing mechanism. Make the engine emit an OpenAPI spec (or
equivalent contract) as a FIRST-CLASS build artifact, then run codegen to derive
typed client hooks + validators FROM it -- so the frontend physically cannot
render without a typed hook mapping to a real endpoint. A static mock stops being
possible because types won't compile.
- Adopt: Orval (OpenAPI -> react-query hooks + zod validators), `clean:true`,
  generated code stamped "do not edit". Human surface shrinks to spec + schema +
  one fetch mutator + business logic.
- Autonomi work: a build-lane step that (a) has the model author `openapi.yaml`,
  (b) runs Orval codegen, (c) fails the build if a page calls a hook with no
  backing endpoint. Wire into run.sh build phase, not a prompt string.
- Size: L. Trust-critical (generation path) -> single-threaded + council.
- Verify: on a throwaway "bookmarks API" spec, confirm generated hooks compile
  and a page cannot fetch a non-existent endpoint.

### M2 -- Real-backend-as-floor scaffold  [makes "static shell" structurally impossible]
Every full-stack build STARTS as a monorepo containing an Express 5 + Drizzle +
Postgres api-server with a real (SQLite-or-Postgres) persistence tier + seed
script -- the FLOOR, not an add-on. "Waitlist" cannot degrade to a static page
when the substrate already has a data tier.
- Adopt: pnpm workspace; `lib/db` (Drizzle schema + `drizzle-zod` for derived
  insert/select types) is the 2nd source of truth; `artifacts/api-server` does
  real SQL aggregation; a 308-line seed script so screen 1 is never blank.
- Autonomi work: a pinned "full-stack app skeleton" template the engine
  scaffolds when the spec implies a backend (the FV signal already detects this),
  vs a static-only template when it genuinely doesn't. Expert-mode stack pinning:
  stop re-deriving structure per build.
- Size: L. Verify: throwaway "task tracker" spec -> real POST persists to DB,
  FV-1 harness reports functional_status=verified (not shell).
- ARCHITECTURE DECISION (learned 2026-07-04, a first M2 attempt): generate the
  server from TEMPLATE FILES + placeholder substitution, NOT bash nested heredocs.
  A heredoc emitting JS hits a collision -- bash expands the JS `${...}` template
  literals and `` `backticks` `` -- which is fragile and error-prone (bit both the
  M1 yaml heredoc and the M2 JS heredoc). Ship a `templates/` dir of real files
  with `__RESOURCE__`/`__COLL__`/`__FIELDS__` tokens and a small substitutor
  (Node or sed), so the generated code is real code reviewed as code, not escaped
  strings. This is cleaner + more modular and avoids the whole escaping class.

### M3 -- Design system pass  [why it looks designed, not default]
A tokens-first design pass wired into the build: role tokens (`bg-card`,
`text-primary`), a real palette (brand-extracted when a brand exists, a strong
modern default otherwise), a parallel `.dark{}` block, and three UI states per
data surface (skeleton / empty / first-run-with-sample-data).
- Adopt: `index.css` derives palette from brand screenshots; every component uses
  role tokens never raw hex; shimmer/empty/first-run states backed by seed data.
- Autonomi work: a design-direction module in the build lane (the "ultramodern,
  award-winning UI" the founder wants for greenfield) + a completeness checklist
  (loading/empty/error/first-run) the FV/quality gate enforces. This is the ONE
  place a prompt-layer helps -- but only atop M1/M2 structure.
- Size: M. Verify: throwaway spec -> generated app has all three states + tokens,
  no raw hex, dark mode present.

### M4 -- Wire FV into the verdict (FV-2, founder-gated)
Once M1-M3 make real apps the norm, wire the FV-1 functional signal into the
completion verdict so "Verified" REQUIRES the app to actually work. This is our
differentiator over Replit (they ship pretty apps unproven; we prove behavior).
Council + founder reviews the reclassification before it locks (task #9).

## How Autonomi WINS (not a Replit copy)

Replit builds pretty apps but ships NO proof they work. Autonomi's FV harness
verifies behavior. The winning combination: **generate Replit-caliber apps
(M1-M3) AND prove they function (M4)** = polish + honesty. Plus the cost/speed
levers (convergence work already shipped this session: fewer wasted iterations;
Haiku-default for cheap builds) drive toward $1 / 2-3min once the structure stops
requiring 41 iterations to converge.

## What is NOT claimed

- No "10x" or "beat Replit" claim is made until a real build on a stronger engine
  is measured head-to-head (accuracy/cost/speed/quality) and the numbers support
  it. The current honest baseline (2.4hr) shows we are FAR from the target today.
- This plan is not implemented. It is the executable program derived from the
  study. A fresh session executes M1 first, single-threaded, council-verified,
  proven on a throwaway spec, never teaching-to-the-test.
