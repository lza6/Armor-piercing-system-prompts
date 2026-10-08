# Competitive intel, 30 days to 2026-09-14

Window: **2026-08-15 to 2026-09-14**. Every claim below carries a URL or a
file:line. Anything without one is in the UNVERIFIED section at the bottom and
is deliberately excluded from the ranked list.

**Prior document.** `docs/COMPETITIVE-NEXT-10.md` was written 2026-09-09 and is
the same shape of deliverable (ranked build list vs factory.ai and 8090, with
already-exists verdicts). This document does NOT supersede it. It covers what is
new since it, and every ranked item below states whether it is a NEXT-10 item or
not. Re-proposing a NEXT-10 open item as a fresh discovery would be this repo's
signature failure committed at the document level rather than the item level.

**Evidence labelling used throughout:**

- `[measured]` - a command run against this repo on 2026-09-14, output quoted.
- `[sourced: URL]` - a page actually fetched on 2026-09-14.
- `vendor-claimed` - the vendor's own page or blog. Evidence that a CLAIM exists,
  not that the capability works.
- `UNVERIFIED` - could not be cited. Never ranked.

---

## 1. What genuinely shipped in the window (2026-08-15 to 2026-09-14)

### Factory.ai

**Aug 27, 2026 - "What it Takes for Coding Agents to Complete Large Software
Tasks"** [sourced: https://factory.ai/news/what-it-takes-for-coding-agents-to-complete-large-software-tasks]

The most strategically relevant thing any competitor published in this window.
Factory argues a single agent validating its own work stops early with "much of
the outcome absent", and reports that splitting the run into orchestrator /
implementer / validator separated by an information barrier they call **"the
wall"** produces large gains. The validator authors an independent measurement
instrument BEFORE implementation begins, so the implementer cannot tailor work to
visible test cases.

Numbers, all **vendor-claimed**, on ProgramBench (partial-credit behavioral
parity, not pass/fail):

| Subject | Single agent | With the wall |
|---|---|---|
| GDAL | 36% | 90% |
| 7-Zip | 54% | 95% |
| DuckDB | 34% | 80% |
| Fable 5 median, 24 tasks | 56.7% | 89.3% |
| Kimi K3 median | 45.1% | 75.4% |
| GPT-5.6-Sol median | 48.6% | 66.2% |

Note on reading these: ProgramBench's own paper reports that "none fully resolve
any task" [sourced: https://arxiv.org/pdf/2605.03546]. That is NOT a
contradiction of Factory's figures - ProgramBench scores partial behavioral
parity, so a high median partial score and zero fully-resolved tasks are
compatible. Treat the table as vendor-claimed on a partial-credit metric.

**Sep 1, 2026 - CLI v0.209.0** [sourced: https://docs.factory.ai/changelog/release-notes]
Default model for new sessions became GPT-5.6 Sol. Session archiving.

**Aug 17 - Sep 1, 2026 - changelog band** [sourced: same]
Roughly twenty releases. Read in bulk, it is overwhelmingly **UI and session
polish**, not capability: diff line wrapping, sidebar width persistence, session
search speed, model-list search box, voice dictation accuracy, markdown tables,
mermaid rendering. Three items have substance: `droid doctor` for connectivity
diagnostics (Aug 25), the explorer droid gaining every read-only tool (Aug 18),
and org admins being able to disable image generation (Aug 21).

**Sep 9, 2026 - Factory listed on Claude Marketplace**
[sourced: https://factory.ai/news/claude-marketplace]
Enterprises can spend existing Anthropic commitments on Factory, removing a
separate procurement cycle. This is a **distribution and procurement** move, not
an engineering one. It is ranked separately below for that reason.

### Frontier models

**Sep 3, 2026 - GPT-6 Astra** [sourced: https://openai.com/index/gpt-6-astra/,
https://simonwillison.net/2026/Sep/3/gpt6-astra/]. Closed multimodal reasoning
model; headline upgrade is native computer use.

**Claude Fable 5.1** - released in the window per
[sourced: https://patmcguinness.substack.com/p/claude-fable-51-gpt-6-astra-and-the].
Exact date not established from a primary Anthropic source; see UNVERIFIED.

**Meta Muse Spark 1.3** - named in the same roundup as one of five models pushing
efficiency at lower cost. Secondary source only; see UNVERIFIED.

### Independent / community signal

**Verification is now the named bottleneck.** Multiple September HN digests
converge on the same sentence: the constraint is no longer generation speed but
verification capacity
[sourced: https://www.developersdigest.tech/blog/what-hacker-news-gets-right-about-ai-coding-agents-2026].
This is the market moving toward this repo's stated wedge, not away from it.

**Docket** (github.com/yielab/docket) [sourced: https://github.com/yielab/docket]
is the closest direct competitor to the receipt. It gives every AI-written commit
a hash-chained JSONL audit log with a `docket audit verify` command. It is
self-labelled `v0.2.0-beta.2` and warns "Expect breaking changes between beta
releases"; no public star count was visible on the page I fetched, and the fetch
could not establish repo age or last-commit date, so **treat its adoption as
unmeasured rather than zero**. The durable point is architectural, and does not
depend on adoption: verification is hash-chain only, with **no cryptographic
signatures and no offline third-party verification**. Its own docs concede "an
operator able to delete all docket state can erase both the log and its backup."

---

## 2. Context: important, but OUTSIDE the 30-day window

Kept separate so nothing here is mistaken for a recent development.

- **Apr 1, 2026 - Legacy-Bench** [sourced: https://factory.ai/news/legacy-bench,
  https://github.com/factory-ai/legacy-bench]. Apache-2.0, 19 stars. COBOL 46%,
  Java 7 32%, BASIC/C89/Fortran/Assembly 5-6% each. Pass rates **16.9% to 42.5%**
  across 12 model-agent combos, against >70% for the same models on
  Terminal-Bench 2 and SWE-bench Verified. Droid + GPT-5.3-Codex tops it at
  42.5%. Ten sample tasks are public; the full set requires contacting Factory.
  Factory states "We welcome evaluation submissions."
- **Jun 30, 2026 - "Building to the Test: Coding Agents Deliver What You Check,
  Not What You Requested"** [sourced: https://arxiv.org/pdf/2606.28430]. Ma,
  Kereopa-Yorke, Schultz. Independent, non-vendor evidence for exactly the
  failure mode Factory's "wall" post claims to fix.
- **Jun 1, 2026 - Factory Router** [sourced: https://factory.ai/news/factory-router].
  "cuts token spend by 20-25%", private research preview, measured on
  Terminal-Bench 2 and Legacy-Bench against an Opus 4.7 baseline.
- **Aug 13, 2026 - Agent Effectiveness** [sourced: https://factory.ai/news/agent-effectiveness].
  Thirty-two days out, so just outside. **"Currently in Private Preview"** - a
  claim of a product, not a shipped one. Measures cycle time, work intent, and
  attribution of sessions to shipped artifacts.
- **Jan 23, 2026 - Signals** [sourced: https://factory.ai/news/factory-signals].
  Shipped and running daily; friction detection that auto-files Linear tickets
  and assigns them to Droid.
- **Sep 25, 2025 - Terminal-Bench 58.75%** [sourced: https://factory.ai/news/terminal-bench].
  Self-reported by Factory, run on their own Ubuntu machines.
- **Aug 13, 2026 - DeepSeek V4-Pro GA and `deepseek-harness`** (MIT, everything a
  plugin) [sourced: https://www.digitalapplied.com/blog/deepseek-harness-open-source-agent-framework-2026].
  Secondary source; the post itself notes the package is `v0.1.0-rc.5` with no
  GitHub release behind it.

### 8090.ai

**Nothing shipped in the window that I could cite.** The site sells
Software Factory (an "AI-native SDLC control plane") and 8090 Enterprise
[sourced: https://www.8090.ai/]. Most recent datable events are the EY.ai PDLC
launch (Mar 2026) and a $135M Series A led by Salesforce Ventures (Jun 2026)
[sourced: https://siliconangle.com/2026/06/29/ai-software-development-startup-8090-nabs-135m-funding-round/].
Absence of evidence reported as a finding, not filled with a guess.

**CORRECTION to a prior repo audit, and it goes the inconvenient way.** This
repo previously established that 8090's public site yields no verifiable product
evidence, and I was asked to confirm that in one fetch and move on. It does not
confirm [sourced: https://www.8090.ai/software-factory]. Present today: a free
trial at `factory.8090.ai` ("Get Started for Free"), public documentation at
`docs.8090.ai`, a demo video playlist, a pricing page, and a published CMS
claims-modernization case study. Still absent: benchmark numbers, a changelog,
and any version history.

So the accurate 2026-09-14 statement is **not** "no public product surface". It
is: 8090 has a self-serve on-ramp and docs, but publishes **no versioned release
history and no benchmarks**, which is what actually blocks capability comparison.
Recording this as a correction rather than a confirmation because the prior
finding's premise has changed, and re-asserting a stale finding as verified is
the same failure shape as an uncited claim.

### Correction to a prior repo finding

The repo previously established that factory.ai, 8090 and Devin have **no**
G2/Capterra/ProductHunt listings. **That is now outdated for factory.ai.** A G2
product page exists at https://www.g2.com/products/factory-ai/reviews. I could
not fetch it (403), and the secondary report of "1 G2 review, 3 Product Hunt
reviews, Capterra 4.2"
[sourced: https://theaiagentindex.com/agents/factory-ai] is second-hand, so treat
the counts as unconfirmed. The correct statement is: **a listing now exists, with
too few reviews for aggregate sentiment comparison.** For 8090 I found listings
only on SourceForge/Slashdot aggregators, no G2/Capterra/ProductHunt.

---

## 3. Ranked build list

Ranked by (user value delivered) / (effort). Each item states whether it closes a
gap or extends a lead, and its NEXT-10 status.

### 1. Put an information barrier between checklist authoring and implementation

**Closes a gap. Extends an existing lead once closed. Not in NEXT-10.**

**ALREADY EXISTS: PARTIALLY - and the existing half is the valuable half.**
I nearly filed this as "build a validator role", which would have repeated the
8-of-11 mistake. The check refuted it:

- The checklist is derived from the PRD **before** implementation
  `[measured]` `autonomy/run.sh:20483` instructs the agent to create
  `.loki/checklist/checklist.json` from the PRD on the first iteration, with
  per-item verification checks (`file_exists`, `file_contains`, `tests_pass`,
  `grep_codebase`, `command`). `checklist_init` is called at
  `autonomy/run.sh:22349`; the module is `autonomy/prd-checklist.sh`.
- It is **independently re-verified after** a completion claim `[measured]`
  `checklist_verify` at `autonomy/run.sh:23973`, and
  `council_reverify_checklist` at `:24858` and `:25604`, executed by the
  deterministic `autonomy/checklist-verify.py`.

So Loki already has a pre-authored, deterministically-rechecked standard. What it
does **not** have is Factory's wall: the **same agent** writes the checklist and
then implements against it. That is precisely the failure mode
`arXiv:2606.28430` names, and it is independent of Factory's marketing.

**Do:** have the checklist authored by a dispatch that does not see, and is not
continued by, the implementing context. The verification machinery is already
built, so this is a dispatch-boundary change, not a new subsystem. Highest
value-over-effort item on this list because the expensive half already exists.

### 2. Tamper-evidence: nothing to build, and a stale internal note to retract

**Extends the lead. Not in NEXT-10. NO BUILD REQUIRED.**

**ALREADY EXISTS: YES, FULLY.** I nearly ranked "wire `writeWitness`" as the
single highest-value item on this list, on the strength of an internal memory
(`project-audit-chain-not-tamper-proof`) stating it existed with **zero
production callers**. Checking the source refuted that memory outright. It is
outdated and should be retracted:

- `writeWitness` is defined at `src/audit/crosslink.js:234` and **has production
  callers** `[measured]`: `src/audit/subscriber.js:142` (periodic) and `:155`
  (session end), both via `writeWitnessSafely`, plus the wrapper export at
  `src/audit/index.js:226,278`.
- The reconciliation half that the memory said was missing now exists:
  `reconcileWitnessedPrefix` at `src/audit/crosslink.js:560` `[measured]`.
- Most importantly it is **wired into the verdict**, not merely present:
  `crosslink.js:392` calls it and `:398-399` ANDs `witnessedPrefix.valid` into
  the overall `valid`. The comparison is prefix-based, not tip-equality, so
  legitimate chain growth does not false-positive, and the three states
  (`checked` / `no_records` / `unreadable`) are explicitly never collapsed, with
  `no_records` documented as "NOT a pass."

So the honest competitive position is stronger than the memory implied, and it
needs no work. Against Docket: `autonomy/receipt_jwt.py` exists `[measured]` and
`loki proof verify --jwks <url|file>` checks an Ed25519 attestation against a
published key set including **from a local `jwks.json` with no network**
`[measured: autonomy/loki:900-910]`, so a third party can check WHO produced a
receipt with no API token and no key import. Docket has neither signatures nor
offline verification, and concedes an operator can erase its log and backup.

**Do: nothing in code.** Update the memory file so a future session does not
re-propose this. The lesson generalises and is why this section survives at rank
2 despite requiring no build: a memory is evidence of what was true when written,
never of what is true now.

### 3. Persist a per-run human-intervention counter

**Closes a gap against Agent Effectiveness. Cheap. Not in NEXT-10.**

**REFUTED 2026-09-14 by the integrator. ALREADY EXISTS: FULLY.** This item was
filed on the belief that the reader exists and the writer does not. Both halves
of the chain ship:

- **Writer:** `handle_pause` in `autonomy/run.sh` increments
  `.loki/state/interventions.json` (the counter block at `autonomy/run.sh:25712`).
  Counted there deliberately: every pause path funnels through `handle_pause`,
  and the `_PAUSE_IN_PROGRESS` guard makes one blocking pause count once.
- **Proof:** `autonomy/lib/proof-generator.py:1147-1151` reads that file into the
  journey; `:1544-1545` mirrors it to top-level `proof["interventions"]`.
- **Reader:** `_interventions_value` in `autonomy/lib/trust_trajectory.py`.

Verified empirically: replaying the writer's logic yields count 1 then 2 across
two pauses. **No build required.**

The trap that made it look missing: `_AXIS_HIGHER_IS_BETTER = {"interventions":
False, ...}` reads like a disable flag. It is POLARITY (lower-is-better), and
`iterations` is `False` for the same reason. There are only three `_AXIS_*`
dicts and none gates availability. The axis honestly reports `available: false`
until a proof carries the count, which is by design, not a gap.

The stale docstring that seeded this belief was corrected in `a9a02681`.

Original (incorrect) reasoning retained below for the record:

**~~ALREADY EXISTS: THE READER DOES, THE WRITER DOES NOT.~~** This is the
reader/writer key contract trap this repo has hit before.
`autonomy/lib/trust_trajectory.py` already declares `interventions` as a tracked
metric with a lower-is-better direction `[measured: lines 13, 46, 54, 61]`, and
`_interventions_value` reads it at line 145. But its own docstring says
`[measured]`: *"There is no per-run intervention counter persisted today."* It is
gated off (`"interventions": False` at line 46).

Loki already computes things Factory's Private-Preview product does not, notably
**cost-per-VERIFIED-task** `[measured: autonomy/lib/trust_metrics.py:397]`, whose
denominator is a verified outcome rather than a shipped artifact. Factory's
headline metric is "how much independent work Droids do between human
interactions" - which is exactly the field Loki declares and never fills.

**Do:** persist the counter at the existing `check_human_intervention` site and
flip the flag. Small diff, and it completes a metric surface that is otherwise
stronger than the competitor's.

### 4. Run healing mode against Legacy-Bench

**Extends the lead, on the one axis with the most headroom. Not in NEXT-10.**

**ALREADY EXISTS: NO.** `[measured]` `benchmarks/` contains SWE-bench,
swebench-pro-pilot, HumanEval and internal A/B harnesses. Zero Legacy-Bench
coverage; the only `legacy|COBOL` hits are unrelated (`speed-benchmark.sh`,
equivalence reports, SWE-bench patch files).

This is the best-matched external scoreboard in existence for this repo's
brownfield wedge (`project-brownfield-wedge-evidence`: proof-of-function on
legacy migration is unclaimed by every vendor). It is Apache-2.0, runnable via
the Harbor harness, and Factory explicitly welcomes submissions. Frontier agents
score 16.9-42.5% where they score >70% elsewhere, so the headroom is real and the
ceiling is not yet claimed by anyone credible.

**Caveat that must not be skipped:** only 10 of the tasks are public and the full
set requires contacting Factory, who also own the benchmark and currently top it.
A self-run number on 10 public tasks is a pilot, not a leaderboard claim, and must
be published as such.

### 4b. Surface the receipt at the end of the run (added after founder steer)

**Closes a delivery gap, not a capability gap. Very cheap. Not in NEXT-10.**

The founder's framing reorders this list: rank by whether an item reduces human
round-trips or proves completion, not by feature parity. Under that lens the
sharpest question is not "do we have proof" but "does the user GET it at the end
of one run, without asking".

**ALREADY EXISTS: GENERATION YES, SURFACING BARELY.** `[measured]` The receipt is
produced automatically and is opt-OUT, not opt-in: `LOKI_PROOF` defaults to `1`
at `autonomy/run.sh:26810`, `:26894` and `:26979` (all `${LOKI_PROOF:-1}`), and
`generate_proof_of_run` is called on the terminal path "fire-and-forget on both
success and failure runs" (`:26893`). It is regenerated idempotently after
HANDOFF.md and commit writers finish so `proof.tree_sha256` describes the exact
returned tree (`:26975-26981`).

So we already beat the bar the steer set. What is thin is the last inch: the
only place the artifact's location reaches the user is a single echo,
`autonomy/run.sh:8115` - `(or open $proofs_dir/$latest/index.html)`. A user who
does not already know `loki proof` exists can finish a successful run without
ever learning a checkable receipt was written for them.

**Do:** print the receipt path plus the one-line verdict and the
`loki proof verify <id>` command in the end-of-run summary. This is the
highest value-per-line-of-code item on the list: the expensive machinery is
built, defaulted on, and currently under-announced.

### 5. Publish the verification-bottleneck position while the market names it

**Extends the lead. Zero engineering effort. Not in NEXT-10.**

**ALREADY EXISTS: THE CAPABILITY YES, THE POSITIONING ARTIFACT NO.**
`loki verify --fast` runs exogenous checks only, no model call and no network, so
every verdict is reproducible by anyone at the same commit
`[measured: autonomy/loki:18131-18145]`, at a documented 19ms diff-scoped against
an 11,040ms baseline. `loki proof verify` re-checks a receipt for tamper and
drift with `--human` prose output `[measured: autonomy/loki:36195-36210]`.

Independent HN consensus has now converged on "verification is the bottleneck",
and the nearest competing artifact (Docket) is a 0-star beta with no signatures.
This is the cheapest item here: the product is built, the market just started
using our vocabulary.

### Models are not products: the cost-per-task question

The founder's comparison set mixes two categories and they must not be blurred.
**factory.ai and 8090 are PRODUCTS** and are ranked above. **DeepSeek, Muse,
Claude Fable and GPT-6 Astra are MODELS.** For a provider-agnostic harness the
useful question about a model is not feature parity but: can we route to it, and
at what cost per completed task.

**ALREADY EXISTS: TIERED ROUTING YES, CHEAP-FRONTIER ROUTE NO.** `[measured]`
`providers/models.sh:19-24` exposes generic `small|medium|high` tiers and
explicitly forbids callers naming a vendor model, with `medium` the default.
That is the right abstraction and it is already built. But the only MiniMax
reference in the entire repo is a **comment**, not a route:
`providers/opencode.sh:25` records "MiniMax M2.5 resolved 75.8 at $36.64 total
vs Claude Opus 4.6 at 75.6". That is a measured near-parity result at roughly
7.5x lower cost, sitting in a comment.

Two competitive signals make this timely. Factory **removed MiniMax M2.5 from
model selection** on Aug 11 [sourced: https://docs.factory.ai/changelog/release-notes],
and DeepSeek V4 ships open weights under MIT (Aug 13 V4-Pro GA, secondary
source). A competitor retiring the cheap near-parity model is the moment a
provider-agnostic harness can differentiate on cost per completed task.

**Do:** promote the opencode cheap-frontier result from a comment to a selectable
tier with a published cost-per-completed-task number. Low effort, directly serves
the "2-3x value" ask, and it is the one item here where being provider-agnostic
is a structural advantage rather than a parity feature.

**Honest limit:** the $36.64 figure is a single internal benchmark run recorded
in a comment. It must be re-measured before it is published anywhere external.

### Ranked separately: apply to the Claude Marketplace partner waitlist

**Business decision, not an engineering item** - listed apart so it does not
distort the value-over-effort ordering above.

Factory listed on Sep 9 [sourced: https://factory.ai/news/claude-marketplace] and
the marketplace lets enterprises spend existing Anthropic commitments, which
removes a procurement cycle. Loki already ships the required artifacts
`[measured]`: `.claude-plugin/marketplace.json` and
`plugins/loki-mode/.claude-plugin/plugin.json` at 9.50.1. Entry is via a partner
waitlist [sourced: https://claude.com/platform/marketplace], so this is a founder
call, not a build.

### Deliberately NOT ranked (open NEXT-10 items, status unchanged)

Re-listed only so this document cannot be read as re-proposing them. See
`docs/COMPETITIVE-NEXT-10.md` for the evidence:

- **NEXT-10 item 5** - `loki init` writes dead config keys. Open.
- **NEXT-10 item 6** - `llm_review` verdict influence deferred. Open by design.
- **NEXT-10 item 8** - flaky `test-review-assurance-tail.sh`. Open; deliberately
  not patched.
- **NEXT-10 item 10** - hard command blocklist. Designed, not built.

---

## 4. UNVERIFIED

Excluded from the ranked list. Listed so the absence is a finding.

- **Factory, "Why model routing must be in the harness" (24 Aug 2026).** The news
  index lists this title and date, but three URL attempts 404'd and two searches
  failed to surface the post. Worse, three **conflicting** cost figures circulate
  for Factory routing: "20-25%" (Factory Router, Jun 1, primary source), "58%",
  and "cost per successful run is 80.5% of Opus on Terminal-Bench 2 and 78.0% on
  Legacy-Bench" (both secondary). I did not pick one. The post may exist; I could
  not read it, and no routing figure other than the Jun 1 "20-25%" should be
  quoted.
- **Claude Fable 5.1 exact release date.** Named in a secondary roundup only; no
  primary Anthropic announcement fetched.
- **Meta Muse Spark 1.3.** Secondary roundup only. The founder named "Muse"; I
  could not confirm whether this is the same product.
- **DeepSeek V4 SWE-bench figures.** Several secondary blogs give numbers. No
  primary DeepSeek source fetched, so none are reproduced here.
- **G2/Capterra/ProductHunt review counts for factory.ai.** g2.com returned 403.
  The listing exists; the counts are second-hand.
- **Factory's ProgramBench table.** Vendor-claimed, self-run. ProgramBench is an
  academic benchmark (arXiv:2605.03546) but I found no independent leaderboard
  entry confirming Factory's figures (benchmarklist.com returned 403).
- **"Claude Mythos Preview tops SWE-bench Verified at 93.9%"** surfaced in
  search. Not fetched from a primary source, outside the window, and this repo
  has been burned by exactly this class of number
  (`project-swebench-9967-was-string-counting`). Recorded, not used.
