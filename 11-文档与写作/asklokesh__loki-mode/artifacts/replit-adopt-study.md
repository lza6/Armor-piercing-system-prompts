Right, return as text. Here is the report.

---

# What Replit Does Well, and What Autonomi Should Adopt

*A founder briefing. Every claim traces to a scout finding or a verified file in `/Users/lokesh/Downloads/adopt`. Where scouts disagreed I used the number I could verify by hand (route files: I counted 11 in `artifacts/api-server/src/routes/`, not the 21 one scout claimed; I use 11 below).*

The one-sentence version: **Replit ships a real, wired, pretty full-stack app because a contract file exists as a build artifact and everything else is mechanically generated from it. Autonomi ships static shells because it hand-writes disconnected files with a 160:1 process-vs-product prompt and no design or completeness direction.** The fix is structural, not a better prompt.

---

## 1. Top 5 Strategic Moves (ranked by impact on build quality)

### 1. Contract-first, then generate code FROM the contract
**What it is:** One hand-authored `openapi.yaml` (989 lines, 24 operations, verified) is the *only* API contract. Orval reads it once and emits BOTH the frontend React-Query hooks AND the backend Zod validators. The frontend never hand-writes a fetch call; it imports `useGetDashboardSummary()`.
**Why it produces better apps:** A page physically cannot render without a typed hook that maps to a real endpoint. A static mock is impossible to pass off as real because the types won't compile. This is the single load-bearing mechanism behind "wired, not static."
**Autonomi gap:** Autonomi hand-writes frontend and backend as disconnected files that drift. Nothing forces the client and server to agree on a contract. This is the root cause of the static-shell output, not a symptom.

### 2. A real backend tier is the DEFAULT, not an add-on
**What it is:** The template *starts* as a monorepo containing an Express 5 + Drizzle + Postgres `api-server` with real SQL aggregation (DAU/WAU/MAU, `SUM`/`COUNT DISTINCT`), 12 tables, a sync scheduler, and a 308-line seed script (verified).
**Why:** "Waitlist" cannot produce a page-with-no-server when the substrate already has a persistence tier. The structure prevents the shortcut.
**Autonomi gap:** Autonomi's "waitlist" produces a page and no server. It treats a backend as something to add; Replit treats it as the floor.

### 3. Expert-mode stack pinning (fewer degrees of freedom = fewer failure modes)
**What it is:** `.replit [agent]` pins `stack=PNPM_WORKSPACE`, `expertMode=true`. The agent starts inside a known-good strict-TS monorepo instead of choosing a stack per build.
**Why:** A narrow, well-understood substrate is a massive quality lever. Every novel stack choice is a novel failure mode. Replit removed that entire class.
**Autonomi gap:** Autonomi is provider-agnostic and effectively re-derives structure per build. It has no pinned "this is the shape every full-stack app takes" substrate, so each build reinvents the skeleton and can skip the hard parts.

### 4. Brand-extracted design tokens, not default-shadcn
**What it is:** `index.css` derives its palette from real brand screenshots. The comment literally reads *"Colors pulled directly from autonomi.dev screenshots"* with primary indigo `#5B35E8 = hsl(253 80% 57%)` (verified). Every component references role tokens (`bg-card`, `text-primary`), never raw hex. A full parallel `.dark {}` block ships in the same pass.
**Why:** This is the single biggest reason it looks *designed*. Generated apps look default because they ship neutral slate + hardcoded colors. Token-first + brand-derived is the fix, and it re-themes the whole app from one variable block.
**Autonomi gap:** No brand-extraction step and no design direction in the build prompt at all. Output defaults to generic shadcn.

### 5. Three real UI states per data surface (skeleton / empty / first-run sample)
**What it is:** Every data surface ships a shimmer skeleton, a dashed-border empty state with icon + CTA, and a first-run "you're viewing sample data" banner, backed by seed data so the first screen is never blank.
**Why:** This is the highest-leverage "looks finished" signal and it maps directly to the founder's time-to-first-wow mandate. It is what separates "designed" from "AI-default."
**Autonomi gap:** No completeness direction means the happy path is all that gets built. Loading/empty/first-run states are exactly what a shell skips.

---

## 2. Top 5 Technical Moves

### 1. The Orval codegen chain (the missing spine)
`orval.config.ts` produces two outputs from one `openapi.yaml`: a react-query client (`api-client-react`, split mode, custom `customFetch` mutator, `baseUrl:/api`) and Zod validators (`api-zod`, with coerce + useDates/useBigInt). `clean:true` + "do not edit" stamps make generated code disposable. The human surface shrinks to: the spec, the Drizzle schema, one `custom-fetch.ts`, and business logic. **This is the concrete mechanism that turns a spec into hooks the frontend calls and validators the backend enforces.**

### 2. Two sources of truth that reinforce each other
API shape from OpenAPI (Orval); DB shape from Drizzle via `drizzle-zod` (`createInsertSchema` + `$inferSelect`). Backend routes get DB-derived insert validators and select types for free. Real persistence, real derived types, no mocked data.

### 3. End-to-end typecheck as a build gate
Root `build = pnpm run typecheck && pnpm -r build`. `typecheck` runs `tsc --build` across project-referenced libs then per-artifact `--noEmit`, on ONE strict shared `tsconfig.base` (`strictNullChecks`, `noImplicitAny`, `noEmitOnError`, etc.) that every package extends. **A contract change that breaks any consumer fails the build, not runtime.** This automatically catches "frontend calls an endpoint the backend doesn't implement."

### 4. Supply-chain quarantine baked into the workspace
`minimumReleaseAge: 1440` (verified) = no npm version installs until public for 1 day (the window most malicious releases are caught), with a narrow `minimumReleaseAgeExclude` allowlist and a DO-NOT-DISABLE comment. `onlyBuiltDependencies` blocks postinstall scripts for all but 4 vetted deps (kills the most common npm compromise vector). `overrides` force-pin a vulnerable transitive esbuild and prune ~120 non-linux-x64 native binaries. pnpm is *enforced* via a preinstall user-agent guard. These are one-line defaults that make generated apps safer than most hand-written ones.

### 5. The shared typed transport layer + app-layer security
`custom-fetch.ts` (~400 lines, human-owned) is wired into every generated hook: typed `ApiError`/`ResponseParseError`, RFC7807 problem+json parsing, base-URL + bearer-token injection (works from web cookies and Expo). Plus real security hygiene: pino redacts auth/cookie headers; the OTLP ingest route strips prompt/content/source_code attributes at the trust boundary; 500s return generic bodies (no stack-trace leak).

---

## 3. The SINGLE Highest-Leverage Thing to Adopt FIRST

**Emit a contract file as a first-class build step, then codegen the client + validators from it, BEFORE writing any page or route.**

This directly closes the static-shell gap. The earlier diagnosis was that Autonomi's build prompt is 160:1 process-vs-product with no design or completeness direction. Contract-first codegen is the structural fix that no amount of prompt-tuning delivers, because it makes the shell *impossible to build*:

For "landing page with a waitlist," Autonomi should:
1. Emit a tiny spec: `POST /waitlist {email} -> 201`, `GET /waitlist/count -> {count}`.
2. Codegen the client hook + Zod validator from it.
3. Generate the form to call the hook, and the route to satisfy it.
4. Scaffold `waitlist_signups` table + seed a few rows.

The form is then *born wired*. A page cannot render without a typed hook, a hook cannot exist without an endpoint, an endpoint cannot compile without a table. The 160:1 process prompt stops mattering because completeness is now enforced by the type system, not by instructions the model may ignore.

Everything else in this report (design tokens, three UI states, supply-chain gates) is high-value but *additive*. This one is *load-bearing*: it is the difference between a real app and a shell. Ship it first.

---

## 4. What NOT to Copy (scope OUT)

**Replit's managed cloud platform.** A large share of what makes Replit's *platform* work is infrastructure Autonomi should not rebuild:

- **The platform-injected UI layer.** `button.tsx` references `hover-elevate`/`active-elevate2` and a `--button-outline` var that grep confirms exist nowhere in source: Replit injects them via a global runtime stylesheet. Don't chase these; they aren't in the exportable artifact.
- **Managed Auth / DB / Storage / Secrets blueprints.** Replit's `integrations` skill hands the agent OAuth and secret-store wiring because Replit *operates* the auth and secret infrastructure. Replicating that is a multi-quarter infra program, not a build-engine feature.
- **The full two-tier skill library at moat scale** (~50 build-infra + ~44 vertical app-generator skills). That is a distribution/moat play with its own roadmap; it is not what closes the static-shell gap.
- **Autoscale deploy target + `postMerge`/`postBuild` platform hooks** as literally wired. The *pattern* (reconcile deps + DB schema on merge) is worth adopting; the specific `.replit`/autoscale plumbing is Replit-platform-coupled.

Adopt the *build-time* mechanisms (codegen, tokens, typecheck gate, supply-chain defaults). Leave the *managed-cloud* mechanisms (auth/db/storage/secrets/hosting) out of scope.

---

## 5. How Autonomi WINS: polish + honesty beats polish alone

Here is the sharp, honest framing, and it is Autonomi's actual moat.

**Replit builds pretty apps but never proves they work.** The scouts verified this directly:
- **Zero automated tests.** No `*.test.ts`, no vitest/jest config anywhere (verified: `find` returns nothing). `msw` is installed but has no test suite using it. "Production-grade" here is typecheck + codegen + supply-chain, with *no behavioral verification*.
- **Validation only on the way out, not in.** Only `health.ts` imports the generated Zod schema, and only to validate the *response* (verified: it's the sole route importing `api-zod`). Request boundaries are unguarded: `ingest.ts` hand-parses OTLP, `people.ts` uses raw `parseInt` on query params. The contract exists but the backend barely enforces it.
- **No drift gate.** Codegen freshness is convention-only; `post-merge.sh` never runs `orval --check`. Generated code can silently go stale.

So Replit's guarantee is "it compiles and it's pretty." It is *not* "it works."

**Autonomi's guarantee is exactly the gap Replit left open.** The trust layer (RARV-C, 8 gates, completion council, verified-completion evidence gate) proves *behavior*, not just build-pass. The winning combination:

1. **Adopt Replit's completeness-forcing structure** (contract-first codegen + real backend + seed + design tokens + three states) so the output is a real, pretty, wired app.
2. **Layer Autonomi's FV harness on top** to prove the round-trip: submit the waitlist form -> assert a row lands in the DB -> assert `GET` returns count+1. Replit's typed-hook chain makes this the *natural* state to verify; Autonomi makes "a real POST persists and a real GET reflects it" an explicit gate (maps directly to the open FV-1/FV-2 tasks).
3. **Go one better than Replit on the gaps it left:**
   - Generate Zod validation middleware on params/query/body for *every* route (not just `health.ts`, not just responses).
   - Add a codegen drift gate in CI (`orval` then fail if generated files changed).
   - Generate at least a smoke/route test suite by default. Replit ships zero; Autonomi's verified-completion gate is the genuine differentiator, so *use* it.

**The positioning:** Replit gives you a beautiful app you have to hope works. Autonomi gives you a beautiful app it *proved* works. Polish is table stakes once you adopt the codegen spine and the design system. Polish + verified honesty is the thing no competitor is selling, and it is precisely where the trust layer already lives. Adopt the structure that makes apps real and pretty; keep the trust layer that makes them provably correct. That combination is the win.

---

*Caveats for accuracy: route-file count is 11 (hand-verified), not the 21/30-endpoint figure one scout reported. The `hover-elevate`/`--button-outline` UI classes are Replit-runtime-injected and not in the artifact. All other cited numbers (989-line spec, 24 operations, 308-line seed, `minimumReleaseAge:1440`, `#5B35E8`, zero test files, health.ts-only Zod) are verified against the files.*