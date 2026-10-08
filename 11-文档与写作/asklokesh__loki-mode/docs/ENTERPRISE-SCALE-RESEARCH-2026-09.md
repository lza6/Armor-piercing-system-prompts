# Why people-services penetrated and agent products did not, and what Loki must ship

Research doc. Backlog-led. Every claim carries a file:line or a URL.
Status: complete. Six research desks reported (delivery model, competitive
frontier, large-org rollout, enterprise readiness, adoption instrumentation,
Blitzy). Sections 1 to 3 are external evidence, section 4 is measured in this
repo, section 5 is the backlog that falls out of both.

Two standards hold throughout. A desk's "could not find" is an **absent
measurement, not a finding**. A vendor's number about its own product is a
**claim, not a result**, and is labelled as such every time.

Date: 2026-09-15. Repo state: v9.50.4, commit 07a9ef18.

## The question, as asked

Founder, verbatim:

> why cognition devin, factory.ai and 8090.ai didn't penetrate as accenture,
> infosys or whatever did with people services... i want to make our product
> that level and scalable

Target buyer, verbatim:

> anyone from legacy to cutting edge, why not? for example, I work for Disney
> and we have hundreds of development teams working on old apps and new apps
> both .. we are not regulated industry, like this there are thousands

Two things follow, and they redirect everything below.

**It is not a compliance story.** The buyer says outright they are not a
regulated industry. This repo already ships opt-in token auth, OIDC bearer
validation, a four-role scope model, tenant data isolation, webhook HMAC, and
honest docs for authentication (385 lines), authorization (461), audit logging
(609) and air-gapped operation (91). `docs/ENTERPRISE-IDENTITY-ROADMAP.md`
separates what ships from what does not, and section 4 there refuses to claim
browser SSO, SAML, SCIM, per-tenant RBAC or SOC2. That boundary is honest and
it is not what blocks a Disney-shaped account.

**It is a spread problem.** Hundreds of heterogeneous teams, legacy and
greenfield side by side, no central mandate. The question is what happens on
team number 200's repo, not what happens in procurement.

## 1. What the services firms actually sell

### 1.1 The premise is partly wrong, and the correction is the answer

Devin and Factory **do** have large enterprise deployments. The strongest
evidence is customer-side rather than vendor-side: Nubank's own engineering
blog describes migrating an ETL system *"with over 6 million lines of code"*,
internally estimated at *"over 18 months with a thousand engineers"*, completed
*"in just weeks"*.

So the question is not why they failed to penetrate. It is **why penetration
requires a services layer**. All three route through bodies:

- Cognition and Infosys: "hybrid delivery pods"
- 8090 and EY: "tens of thousands of consultants"
- Anthropic and Accenture: roughly 30,000 trained, plus forward-deployed
  engineers

**The model vendors are buying the services layer rather than selling seats
into it.** That inverts the founder's question. The thing to copy from
Accenture is not headcount; it is that somebody absorbs ambiguity and carries
accountability, and every agent vendor at enterprise scale has concluded they
must rent that function rather than replace it.

**And they copied the motion, which makes the gap sharper, not softer.**
Cognition already runs a forward-deployed delivery org: 24 Customer Engineering
roles out of 65 open, Deployed Engineers in 12+ metros plus dedicated Partner
Deployed Engineers (PRIMARY, cognition.com/careers). It already ships an
outcome guarantee: *"If Devin delivers less engineering value than you're
paying for, Cognition will fund your usage up to $10M until it does"*
(PRIMARY, cognition.com/blog/ai-guarantee). And Cognizant is already deploying
Devin across its own engineering org and its client base, with Cognition
embedding forward-deployed engineers.

So the framing is not "agents sell seats, services sell people." The
penetration gap survives the copying, and that is the finding.

### 1.2 Why it survives: the guarantee and the contract disclaim different things

Read Cognition's guarantee and its enterprise MSA together (both PRIMARY). The
MSA carries **no SLAs**, caps liability at 12 months of fees, and puts output
evaluation on the customer: *"It is Customer's responsibility to evaluate
whether Output is appropriate for its use case ... and Licensor disclaims all
liability for any consequences arising from Customer's acceptance of or failure
to review Output."* It disclaims that the service will *"achieve any intended
result."*

So the vendor is accountable for **burning your budget inefficiently** and not
accountable for **shipping a bad change**. An SOW is the reverse: accountable
for the deliverable. That inversion is the structural cap on account depth, and
it is visible in the vendor's own legal text rather than inferred.

Note also that the remedy is credits against future usage. A buyer whose
program failed does not want more credits for the thing that failed. A services
firm's remedy is that the vendor fixes it at its own cost.

### 1.3 The expansion mechanic, which is the actual answer to the question

Infosys client bands, FY26 Q1 (PRIMARY, SEC exhibit): 1,861 active clients, of
which **$1M+: 1,011, $10M+: 317, $50M+: 85, $100M+: 41**. Top 25 accounts are
35.2% of a $19.3B company. Nobody signs a $100M relationship on day one; the
bands are audited evidence that the climb *is* the business model.

The mechanic behind the climb:

- The **MSA** is signed once at enterprise level: liability, indemnities,
  security, data handling, IP assignment, audit rights, and a **rate card**.
  Months of legal, procurement and security effort.
- Every later piece of work is an **SOW under that MSA**: scope, duration, team
  shape, price from the rate card.
- The **rate card is the scaling primitive**. Once it exists, a new team is
  arithmetic, not negotiation.

Engagement #1 pays the entire fixed cost. Engagement #2 pays almost none of it
and can be approved by a line-of-business budget holder without re-running
legal or security. That is why "195 of our top 200 clients for 10 or more
years" (PRIMARY, Accenture) and 41 clients past $100M exist.

**A second, independent mechanism, and it may be decisive.** SOW work is paid
from an already-approved **labor and contractor** budget. A seat licence hits
the **software** budget: different approvers, different annual cycle, and many
enterprises structurally resist moving money between the two. Engagement #2
draws on a pool already sized for it; a seat expansion competes for a
separately governed pool. (Industry-standard practice, reasoning rather than a
quoted contract; corroborated by the disclosed contract-type mix.)

### 1.4 What buyers are actually paying for

Average quality from these firms is mediocre and everyone knows it. Four
properties outweigh that:

- **Accountability transfer.** A VP who staffs 40 people from Infosys and fails
  has a vendor to point at. A VP who deploys an autonomous agent and fails has
  only themselves.
- **Single throat to choke.** One MSA, one escalation path, dozens of teams.
- **Capacity elasticity.** Add 30 people next quarter, remove them after, no
  hiring or firing. Infosys reports 82.7% utilization, so it deliberately
  carries roughly 15% slack. **That slack is the product.**
- **Ambiguity absorption**, the underrated one. When a spec is incomplete, a
  staffed engagement has humans who go and find out: attend the meeting, chase
  the product owner, ask whoever wrote the old code. **The buyer never has to
  write a complete spec.** That is much of what is being purchased.

Underneath all of it is **labor cost arbitrage**: 323,788 Infosys people,
largely in India, billed at US and European prices. The pyramid, bench and
utilization target are instruments for harvesting that spread.

### 1.5 The strategic opening

Every advantage above is financed by a wage spread that narrows over time. An
agent vendor does not participate in it; its unit cost is compute, which falls.
That is the strongest competitive claim available, and an incumbent cannot
answer it by reorganizing, because **their engine is their headcount**.

Accenture is already converting: gen-AI bookings of **$5.9B in FY2025**
(PRIMARY), booked inside MSAs it already had, at accounts it already had, on a
rate card it already had. That is the number that should worry every agent
vendor. Reports of 11,000+ exits alongside ~70,000 being reskilled (SECONDARY)
describe a firm keeping the accountability layer and shrinking the labor layer.
**The window is the period before that conversion completes.**

## 2. What the agent vendors sell, and the ceiling it creates

### 2.0 The three pricing models, from primary pages

**Factory** (PRIMARY, factory.ai/pricing and /enterprise): Pro $20/mo, Plus
$100/mo, Max $200/mo; Teams $60/mo per team plus $40/mo per seat up to 10
seats; Business custom **up to 150 seats**; Enterprise custom with on-prem and
customer-managed keys. The enterprise page is software only: SSO/SAML, SOC 2,
GDPR, ISO 42001, and the sole service language is *"a dedicated account manager
and 24/7 assistance."* No implementation services, no forward-deployed
engineers, no migration delivery, no outcome guarantee. A named tier topping
out at 150 seats is not shaped for a 5,000-engineer rollout.

**8090** (PRIMARY, 8090.ai/pricing and /custom-delivery): self-serve
*"$200 /user/month"* with *"Tokens Billed Separately"*, and **8090 Enterprise
"Starting at $1M/yr"**. The enterprise motion is a managed service outright:
*"We work with your team to define requirements, workflows, and constraints"*
and *"We host, secure, and maintain the application in production."* The $1M
floor is an SOW minimum in all but name. 8090 correctly identified that the
enterprise buyer wants a deliverable and an owner, not a tool.

**Cognition**: pricing is an **ABSENCE**. cognition.com/pricing returned 404
and devin.ai/pricing returned 429. Every circulating ACU figure is secondary
and is not quoted here. Confirmed from primary sources only that the unit is
consumption.

### 2.1 The ceiling, in one line

A seat or consumption licence expands by seat count, so every expansion is a
software purchase needing new budget, procurement and possibly security review,
**and the customer must supply the two things the services firm supplied**:
someone to absorb ambiguity and someone accountable when a change breaks
production. Expansion is gated on the customer's willingness to take on risk
and work, which is exactly backwards from what they wanted to buy.

A capacity or outcome contract expands by SOW under an existing MSA: no new
procurement, and no additional buyer risk, because the vendor is contractually
accountable.

### 2.2 The unoccupied position, and it is this account

8090 aims at regulated industries (healthcare, insurance, aerospace, energy,
financial services, US government) at a $1M/yr floor. Factory sells software
with no delivery accountability. Cognition sells consumption with an
hours-based guarantee sitting on a legal layer that disclaims fitness for
purpose. The incumbents have converted their tooling but, on unverified
reports, not yet their pricing model.

**Disney is explicitly not regulated, which prices 8090 out of it.** The
position to aim at is verified delivery, sold as capacity with an
acceptance-based guarantee, to large non-regulated enterprises with many teams
and a mix of legacy and new applications, landed through one enterprise
agreement and expanded by generated work orders.

The founder is describing his own account.

**Corrected after the Blitzy desk reported (2.5): that position is NOT
unoccupied.** Blitzy sells per delivered line with unlimited seats, builds
delivery in-house rather than renting an SI, and prices brownfield ingestion as
a standalone product. They hold most of this ground already. What remains
genuinely unoccupied is narrower and is what the rest of this document is
about: a **portable machine-checkable receipt a buyer can re-verify
independently**, a **verdict that refuses to go green when verification did not
happen**, a **harness the customer controls** rather than a hosted proprietary
one, and a **free self-serve path to production** beneath their $50K floor.

### 2.3 Where pilots die

Gartner (attributable to the newsroom URL; body returned 403 and was not read):
*"Over 40% of agentic AI projects will be canceled by the end of 2027, due to
escalating costs, unclear business value or inadequate risk controls."*

One qualitative failure mode is worth more than the circulating ratios: an
early Devin evaluation reported failures as *unpredictable*, with autonomy
itself the liability, pursuing impossible solutions for days rather than
recognizing a blocker. The specific ratio is unverified and not cited. The
mechanism is structural rather than capability-bound: **an agent that cannot
recognize when it is blocked cannot be trusted with unsupervised capacity**,
which is precisely what a staffed engagement sells.

### 2.4 The benchmark layer the market prices agents on

From the Competitive desk, all primary:

- **The industry's evidence layer visibly broke this window.** Measured via
  `gh api`: SWE-bench Verified took 14 submissions in all of 2026, every one
  from a single harness (`mini-swe-agent`), **zero from frontier labs**, and
  nothing at all between 26 Feb and 1 Sep. Every circulating 2026 figure for
  Opus 5, Fable 5.1 or GPT-5.6 corresponds to no submission at all.
- **Its named successor is compromised.** SWE-bench Pro was independently
  audited by Datacurve (26 May) at 8.5% false positives and 24% false
  negatives, with Claude Opus 4.6/4.7 marked CHEATED on 12 to 25% of passing
  trials for running `git log --all` to read the gold patch out of `.git`
  history shipped inside the task containers.
- **Factory's Q3 went into governance, not capability.** It signed the
  open-weights letter and added a local and air-gapped path via NVIDIA, which
  contests ground we would otherwise call ours.
- Factory's routing claim, primary and self-run: routed runs reach 99% of the
  frontier model's pass rate on Terminal-Bench 2 and 96% on Legacy-Bench, at
  about 20% lower cost per successful run (24 Aug 2026). Vendor self-run, so a
  claim, never a result.
- Do not quote: the circulating DeepSeek V4 "80.6% SWE-bench Verified" has no
  primary source; DeepSeek's own changelog does not report SWE-bench at all.
  Meta Muse should be dropped from the competitive set (the 8 Sep launch is a
  consumer agent with no coding capability).

This is the strongest external support the verifiable-receipt thesis has ever
had. The shared industry yardstick is unattended and its successor is measurably
gameable, which is precisely the condition under which a per-run, machine
checkable receipt beats a leaderboard number.

### 2.5 Blitzy: the nearest competitor, and it contradicts section 1

Added at the founder's direction. "Near competitor" is **confirmed**: same
thesis as ours (spec in, multi-day autonomous build, PR out, brownfield
capable). Their own post names the neighbours: *"Factory and Devin are the
closest examples."*

Access note: `blitzy.com` and `docs.blitzy.com` return HTTP 403 to direct
fetches (Cloudflare). All primary text below came through a text proxy, with
raw bytes pulled for the six pages the findings turn on.

**Mission, verbatim:** *"Autonomous software development at enterprise scale ...
Blitzy reverse engineers 100M+ lines of existing code, constructs a deep
architectural understanding, then autonomously builds, refactors, and
modernizes. The result is 80% + of entire projects delivered with end-to-end
tested code. Built for the enterprise codebases foundation models have never
seen."*

**They are a counter-example to section 1, and it is the most important thing
here.** Every other vendor rents the services layer (Cognition through
Cognizant, 8090 through EY, Anthropic through Accenture). Blitzy **builds
delivery in-house and bundles it into the price**: Forward Deployed Engineers,
Forward Deployed Designers, a Field CTO, AI Solutions Consultants. Same insight
that autonomous coding at enterprise scale needs humans and change management,
but the margin and the account control stay inside Blitzy.

**The unit of sale is lines of code, with unlimited seats** (PRIMARY, pricing
page): *"Pricing follows a transparent two-rate model: $0.10 per line onboarded
for reverse engineering and $0.20 per line generated for forward engineering."*

| Tier | Price | Scope |
|---|---|---|
| Sandbox / Reverse Engineer | $0 | self-serve, no card |
| Concept Validation | $50K | 2 months, guided |
| Structured Pilot | $250K | 6 months, 5M lines RE / 1.25M generated |
| Commercial | ~$500K/yr | first 20M lines, Blitzy Cloud VPC |
| Enterprise | ~$5M/yr | ~50M lines, 2 FDEs |
| Transformation | ~$50M/yr | ~500M lines, Field CTO + 6 FDEs + 2 designers |

That is the furthest point from seat-based SaaS in this entire study, and it is
the model section 1 argues we need. Someone is already executing it.

**They sell our wedge as a priced product.** Reverse engineering at $0.10/line
is a standalone deliverable: an enterprise can buy ingestion alone and receive
codebase documentation. "Legacy is unclaimed" takes further damage here, beyond
what section 3.1 records.

**Funding, and a correction to the founder's framing.** $200M at a $1.4B
valuation, announced 2026-05-05, led by Northzone with PSG, Battery, Jump,
NFX and others; roughly $204M raised total, so this round is nearly all of it.
Founded 2023. **"Fully investor led" is confirmed as to backing and refuted as
to revenue**: they sell $50K to $50M contracts, disclose *"dozens of Global
2000"* customers, and publish *"$2.91 ARR generated per dollar burned since
January 2025"*. Unaudited and vendor-stated, but it is a claim to having
revenue, which is the opposite of investor-funded-only. No ARR or customer
count is disclosed anywhere primary.

**Their benchmark evidence is better than ours.** SWE-Bench Pro Public 66.5%
(486/731) in March, 84.95% in June, both **commissioned third-party audits** by
Quesma, who state plainly: *"We were contracted by Blitzy to audit and verify
their results ... We audited their execution environment and required them to
apply our recommendations before validating the run."* Verdict: *"We found
nothing that would affect the score."*

Label it precisely: **commissioned audit**, materially stronger than the vendor
self-run numbers this repo has been burned by, and **not disinterested**, since
Blitzy chose and paid the auditor. Quesma's own chart caption carries the
caveat Blitzy's marketing omits: *"All SWE-Bench Pro Public scores are
self-reported."* No Quesma artifact naming 84.95% was located, and no
denominator was disclosed for it, so comparability with 486/731 is unconfirmed.

### 2.6 Where Blitzy is vulnerable, and why the receipt survives

The differentiator holds, but narrowly, and for a reason documented in their
own docs rather than asserted by us.

Blitzy **does** run dynamic validation: test suites, performance benchmarks,
build and compile checks, lint, static analysis, integration and E2E, CI/CD
outputs, fed back into generation. They have a Rules engine with a
`require-test-coverage` rule whose text targets the same anti-pattern our
mock-integrity and test-mutation detectors do: *"Tests must invoke real
production functions, not reimplement business logic."*

But:

- **The documented enforcement of that rule is a human instruction**, not an
  automated verdict: *"Flag any test file where business logic appears outside
  of imports, setup, invocation, or assertions."*
- **Their primary control point is prose.** The Agent Action Plan is
  human-reviewed (*"your primary control point"*), and the Project Guide
  summarizing what was built and *"what work remains"* is human-readable text.
- **The burden of recording what went unverified is placed on the human.** The
  reviewer is *"fully accountable"* and must *"document review coverage,
  explicitly state what was reviewed versus what was not."* That is the tell:
  in our system the gaps ledger is emitted by the machine and gates the
  headline; in theirs a person is asked to write down what they did not check.
- **No documented fail-closed verdict.** Nothing found describes the system
  refusing to call work done when it cannot verify. Their framing is the
  inverse: 80% done, here is the remaining 20%, human adjudicates.
- **The harness is proprietary and hosted.** They fuse models; we run on Claude
  Code, Codex, Cline and Aider, so the customer keeps the harness and the exit.
- **No self-serve path to production.** A $50K floor cedes the bottom of the
  market, and their own free-tier limit contradicts itself by 10x across their
  pages (1M lines on the homepage, 100K in the pricing FAQ).
- **Jobs are not cancelable** and consume quota, under per-line billing.

**Stated plainly, because it is the credible version:** for a single artifact
Blitzy publishes more verification evidence than we do (their curl C-to-Rust
demo cites 215,153 lines, 7,312 tests passing, Miri and ASan clean, 80.05%
coverage; vendor self-run and not upstream-accepted). What they do not have is
a portable, machine-checkable receipt a buyer can re-verify independently, or a
verdict that refuses to go green when verification did not happen.

**Where they beat us outright:** black-box and air-gapped deployment, SOC 2
Type II and ISO 27001, 100M+ line ingestion as a priced deliverable, in-house
delivery, commissioned audits, security reachability triage reconciled against
Snyk, Mend, Sonatype and GitHub Advanced Security, and $200M.

## 3. What actually stops the spread past team N

Landed from the Large-Org Rollout desk.

A standard applies to this whole section, the same one applied to every repo
measurement above: **a desk's "could not find" is an absent measurement, not a
finding.** An earlier draft of these headings said REFUTED. That promoted an
absence past the bar this document holds everywhere else, and it is corrected
below.

### 3.1 The mainframe-exit market is crowded and discredited; this buyer's legacy is JVM and .NET

Gartner, June 2026 (SECONDARY reporting of direct quotes; the Gartner release
itself returned 403): *"More than 70 percent of mainframe exit projects
initiated in 2026 will fail to produce the intended benefits due to an
overestimation of generative AI tooling capabilities"*, and *"by 2030, 75
percent of vendors operating in the 'mainframe exit' market will either pivot
their business models or cease to exist."* A market with predicted 75% vendor
mortality is crowded and discredited, not empty.

A framing correction that matters more: COBOL and mainframe modernization
concentrates in banking, insurance and government, exactly the sectors this
buyer excluded. A media, retail, travel or logistics enterprise's "decades-old
apps" are Java 6/8, .NET Framework, Struts, Perl, PHP monoliths and on-prem
Oracle. **Building a legacy strategy from COBOL vendor evidence answers a
question the founder did not ask.** This sharpens section 4.1 rather than
weakening it: the JVM and .NET gap IS the legacy gap for this buyer.

Scope the Gartner numbers honestly: they cover the **mainframe-exit market**,
and the desk itself says mainframe is the wrong legacy here. So they refute a
claim this repo never made. What they do establish is that "AI legacy
modernization" as a *category* is overclaimed, so it is a poor banner to march
under even though the underlying work is real.

The genuine opening left is narrower and unbranded: Gartner's own
recommendation of modernization *in place*, plus framework and dependency
upgrades, dead-code archaeology, ownership reconstruction, and characterization
tests for untested code. That is adjacent to what healing mode already targets.

**Narrowed again by the Blitzy desk (2.5).** Blitzy prices exactly this work as
a product: $0.10 per line reverse-engineered, sold standalone, explicitly aimed
at *"the enterprise codebases foundation models have never seen"*, with $200M
behind it. Non-mainframe brownfield is therefore not unbranded either. Treat
the remaining opening as the verification and control properties listed at the
end of 2.2, applied to this work, rather than the work itself.

### 3.2 No demand found for attestation as a deliverable; the evidenced pain is review capacity, and the market's answer is triage

No practitioner or engineering leader asking for a verification artifact could
be found. Recorded as an absence, not filled with a weaker source, and **not
promoted to a refutation**. Two reasons the inference is weak, both visible in
the desk's own evidence:

- It searched for practitioners *asking for* a receipt. Nobody asked for a
  risk-based approval bot either; Zalando built one from pain. "No one
  requested mechanism X" is weak evidence about X when X is a mechanism.
- The Roblox quote names *"reviewing, validating, and **trusting** it"*. That
  is the trust family, not a move away from it.

Three things are being conflated, and only two are evidenced:

1. **Verification is costly.** CONFIRMED. DORA 2025: time saved in creation is
   re-allocated to auditing. Stack Overflow 2025: 45.2% say debugging AI code
   takes longer.
2. **Practitioners want LESS TO READ.** CONFIRMED, verbatim. ITK maintainers:
   *"The current stream of AI generated pull requests is a bit overwhelming to
   me. It is hard for me to review them carefully"*; *"AI-generated code
   requires more careful review than human-written code. Every line is
   suspect."* None of them asked for proof a change had been verified.
   **A receipt is one more artifact to read, not one fewer.**
3. **Orgs want a machine-checkable artifact.** UNEVIDENCED.

What the two best-matched large orgs actually built when they hit this exact
wall is the strongest available signal, and it is not a receipt:

- **Zalando** (250+ teams, retail, non-regulated) built a risk-based PR approval
  bot with rules derived from production incident analysis, auto-approving the
  33% of PRs classified low risk, cutting lead time 20 to 40%. That routes
  scarce human attention; it does not prove correctness.
- **Roblox**, having named trust as the bottleneck explicitly, built agentic
  guardrails, real-time API access policy, and Exemplar Alignment. Gating and
  alignment to precedent, not attestation.

The market's answer to review pain is **triage and gating**, not attestation.
A receipt converts to value only if it lets a human **skip reading something**,
that is, only if it drives a routing decision. Zalando's bot is that product
shape. A signed manifest is not, unless it feeds one.

**This does not demote the receipt; it names its destination.** To route any PR
away from a human you need a signal that says "we checked this and it passed"
and is distinguishable from "we could not check". That is precisely what the
receipt is. It also explains why section 4.1 is the top of the backlog rather
than a competing priority: on a Maven or .NET repo the build fact is `not_run`
on every run, and `weak` (`proof-generator.py:1563`) collapses `not_run`,
`inconclusive` and `skipped` into one bucket, so a routing rule reading that
signal sends **everything** to a human on exactly the repos that are the bulk
of the estate. Stack coverage is the prerequisite for the routing play, not the
alternative to it.

`project-competitive-anchor-verifiable-receipt` and
`project-adoption-strategy-v8` are not refuted by a could-not-find. They need a
qualifier: the receipt earns its keep by driving a routing decision, not by
existing. That amendment is deliberately deferred until the Delivery Model desk
lands, since it bears on unit of sale.

### 3.3 The binding constraint is review capacity

- **Roblox**, QCon AI Boston 2026: *"AI tools are generating more code than
  ever, yet the time it takes to ship that code safely to production remains
  stagnant. The bottleneck has shifted from writing code to reviewing,
  validating, and trusting it."*
- **Oversight becomes nominal without anyone deciding to abandon it.**
  arXiv 2605.02273 (EASE 2026): most AI-generated PRs receive no review at all;
  when reviewed, agents rather than humans dominate. Review metrics stop
  reflecting human oversight. (Abstract-level only; body extraction failed
  twice, recorded as a gap.)
- **Agent output inflates PR size, feeding back into the same constraint.**
  Zalando measured the shift into the 500-1k and 1k-2k line buckets and states
  large PRs *"discourage reviewers and slow down delivery."*
- Trust has not improved with capability: DORA ~30% little or no trust; Stack
  Overflow 3% highly trust, 46% distrust, 66% hit "almost right, but not quite".

### 3.4 Per-team harness cost does not amortize the obvious way

- arXiv 2602.11988 (ETH Zurich / LogicStar), 138 real tasks, 4 frontier models:
  *"providing context files does not generally improve task success rates,
  while increasing inference cost by over 20% on average."* LLM-generated
  context files *reduced* resolution rates. *"Repository overviews, although
  popular and recommended by model providers, are not helpful."*
  **So the standard scaling answer, a generated CLAUDE.md or AGENTS.md per
  repo, has been tested and does not pay for itself.**
- Anthropic's own large-codebase guidance (FLAGGED, vendor about own product)
  requires a DRI, dedicated pre-rollout infrastructure work, and a
  configuration review every three to six months; without central governance
  *"knowledge will stay tribal and adoption will plateau."* It concedes the
  hierarchical approach breaks down on some legacy estates and defers guidance.

### 3.5 Adoption is social, so mandates do not scale it

Microsoft, arXiv 2607.01418, tens of thousands of engineers (FLAGGED:
Microsoft measuring a Microsoft-owned product, though it also covers a
competitor's tool and is candid about limits): skip-level peer usage above 25%
produced **+216% odds of trying**; manager influence was strong on trial
(+82%) but weak on retention (+22%). Counterintuitively, 60+ days of prior IDE
Copilot use predicted **12 to 15% LOWER retention**, while active shippers
(2+ PRs/week) retained at +31%. Pilot enthusiasm is a poor predictor of
sustained use.

### 3.6 The measured-outcome literature, ranked by trustworthiness

| Study | Design | Finding | Independence |
|---|---|---|---|
| METR, arXiv 2507.09089 | RCT, 16 devs, 246 tasks | **19% SLOWER**, while devs believed they were 20% faster | PRIMARY, non-vendor. Best design, smallest n |
| DORA 2025 | Large survey | 90% use; ~30% little/no trust; raises throughput AND instability | PRIMARY |
| Stack Overflow 2025 | Large survey | 84% use; 3% highly trust | PRIMARY |
| Google, arXiv 2410.12944 | RCT, 96 engineers | ~21% faster, *"confidence interval is large"* | FLAGGED: Google measuring Google tooling |
| Microsoft, arXiv 2607.01418 | Quasi-experimental | +24.0% PRs/engineer/day | FLAGGED: Microsoft measuring its own product |
| Faros AI 2026 | Observational telemetry | +441% time in review, 31% merged unreviewed | **VENDOR CLAIM**, sells the measurement platform |

The ordering is the finding: the cleanest design found a slowdown, and the two
large speedups both come from vendors measuring their own products. The widely
quoted "+441% review time" and "31% merged with zero review" are Faros numbers
from an observational before/after within customer orgs; the same report carries
a 15,324% ROI figure, which is the credibility tell for the set.

**DORA's amplification finding is the honest answer to "what breaks in
legacy":** AI amplifies existing organizational capability rather than
substituting for it. In a hundred-team estate with uneven test maturity that
predicts divergent outcomes: strong-test teams accelerate, weak-test legacy
teams get measurably worse. No controlled study isolating agent performance on
large old codebases with poor tests was found. That is a gap in the literature,
not something to reason into existence.

## 4. Where Loki itself breaks under that load

All measured this session, first hand. Every zero carries a positive control
taken in the same command.

### 4.1 The receipt cannot go fully green on a JVM or .NET repo

This is the central finding.

The main run path classifies a target's build three ways in
`enforce_build_check` (`autonomy/run.sh:10250`). Only three stacks are
**runnable**: an npm `build` script, `go build`, and `cargo build`
(`run.sh:10286-10296`). Maven, Gradle, .NET, sbt, Bazel, Make, CMake, meson,
autotools, setup.py and pyproject all classify as `present`, which records
`not_run`.

That is deliberate and honest. Four council rounds drove the code to a
list-free design where an unrecognized build is an honest gap and never a fake
N/A; the comment at `run.sh:10320` explains that ignorance must not read as
VERIFIED. **The defect is coverage, not integrity.**

The consequence is mechanical:

```
build_class=present -> status not_run
  -> weak = ("not_run","inconclusive","skipped","failed")   proof-generator.py:1563
  -> _compute_degraded appends the build item                            :1592-1597
  -> _compute_degraded feeds _compute_headline                           :1450-1451
  -> `if tests_verified and not degraded and diff_nonempty: return "VERIFIED"`  :1722-1723
```

So a non-empty degraded ledger **caps the run at "VERIFIED WITH GAPS"**. To be
precise: a `not_run` build does not force NOT VERIFIED (only a `failed` build
does, through `any_failed`). It caps. A Maven, Gradle, .NET, sbt, Bazel, Make or
CMake repo can therefore never reach full VERIFIED, on any run, however good the
work was.

Do not conflate two ledgers: disabled-gate entries are appended *after* the
headline is computed, deliberately (`:1457-1462`), and cannot move the verdict.
Only the facts-derived `degraded[]` gates it.

A second surface is stricter still. `own-render.py:_is_ready`
(`autonomy/lib/own-render.py:138-141`) requires `facts.build.ran` true and
status not `not_run` before a run may render green at all. That is one surface
with a single call site (`run.sh:27129`), written for a non-technical owner, so
it should be cited as that surface rather than as the universal path.

**The tests half fails the same way, and it is worse.** `_is_ready` also
requires `facts.tests.status` in (passed, verified). `enforce_test_coverage`
(`run.sh:11844`) has a rich JS/TS branch (declared `scripts.test`, then vitest,
jest, mocha) plus python, go and cargo. Whole-tree runner coverage across
`autonomy/`, `loki-ts/src/`, `dashboard/`, `providers/` and `mcp/`:

| runner | hits | |
|---|---|---|
| pytest | 72 | control |
| go test | 28 | control |
| npm test | 27 | control |
| cargo test | 12 | control |
| gradle | 9 | every `run.sh` hit is the build-manifest detection list at `:10303-10304`, not a runner |
| mvn | 1 | `autonomy/hooks/migration-hooks.sh:101`, healing path only |
| dotnet | 1 | `run.sh:10998`, a comment explicitly deferring C# |

A Java-Maven or .NET team therefore fails **both** halves of the green gate: no
runnable build and no recognized test runner. In a Disney-shaped estate the JVM
and .NET repos are the bulk of the legacy surface, which is exactly the
brownfield wedge the product claims. The differentiator is structurally
unreachable on the repos that matter most.

Note the two stack surfaces are separate and must not be conflated.
`detect_test_command` (`autonomy/hooks/migration-hooks.sh:89`) is a seven-branch
manifest-at-root chain belonging to the healing and migration path; it has zero
references in `autonomy/run.sh` (control: `save_state` 28).

### 4.2 The escape hatch does not reach the repos that need it

An earlier draft of this section called `LOKI_TEST_COMMAND` "undiscoverable".
That was too generous. Verified: it is **unreachable** on a non-JS repo.

Inside `enforce_test_coverage`, the branch that reads it opens with
`if [ -f "${TARGET_DIR:-.}/package.json" ]` (`run.sh:11855`) and the variable
is consumed 41 lines later at `run.sh:11896`, still inside that branch. A
Maven or .NET team cannot use it to point Loki at `mvn verify` or
`dotnet test`, because control never enters the branch that reads it.

It is also absent from **0** user-facing files across `docs/`, `README.md`,
`SKILL.md` and `wiki/` (controls: `LOKI_PROVIDER` 17 files,
`LOKI_MAX_ITERATIONS` 17), surfaced only inside a runtime error string on the
healing path, and absent from the project config map, so it cannot be committed
per-project. There are 1,116 distinct `LOKI_*` variables in source.

### 4.2b The agent is teachable per-repo; the verifier is not

This is the sharpest framing of the whole gap. A team CAN teach the agent its
conventions: `AGENTS.md` is read on the main path (3 references in
`autonomy/run.sh`). The build gate never consults it: **0** references to
`AGENTS.md` inside `enforce_build_check` (`run.sh:10250-10410`), against a
control of 14 `package.json` references in the same range.

So team 2 can make the agent behave correctly on their Java service and still
cannot make the receipt say verified. Onboarding effort buys agent quality and
buys nothing at all in proof.

### 4.2c Route asymmetry: the Bun route turns the same condition into a PASS

The bash route records an honest `not_run`. The Bun route does not.
`loki-ts/src/runner/quality_gates.ts:464-467`:

```ts
if (!existsSync(join(cwd, "package.json"))) {
  return { passed: true, detail: "test_coverage: no test-results.json and no package.json -- skipping" };
}
```

A repo with no `package.json`, which is every Java, .NET, Go-without-npm and
Python repo, receives `passed: true` for a suite that never ran. That is a
skip-as-PASS: precisely the fake-green the bash route was driven over four
council rounds to eliminate. It is a route asymmetry and it flatters exactly
the estate this document is about.

`loki doctor` does not help. `cmd_doctor` (`autonomy/loki:13186`) is 866 lines
and every build or test mention concerns provider login or our own detector
scripts. Nothing in it inspects the user's repo for a build system or a test
runner. So the fix is to surface the condition where a new team already looks,
not merely to document a variable.

### 4.3 First-run failure data is structurally unobtainable in an enterprise

A prior claim of mine, that telemetry says nothing about where a first run
dies, is **refuted**: `first_run_blocked` exists (`autonomy/telemetry.sh:344`)
with a blocker enum (`:195-208`) and four real call sites.

The real situation is worse than the claim was. That event is gated off twice
for exactly the target cohort. `_loki_analytics_enabled` (`:271-296`) requires
telemetry enabled **and** a separate explicit analytics opt-in that defaults
off, while `_loki_telemetry_auto_off` (`:23-56`) auto-disables on
`LOKI_ENTERPRISE=true`, `CI`, `GITHUB_ACTIONS`, `JENKINS_URL` and
non-interactive shells. In a hundred-team enterprise the base gate is off, so
the analytics gate can never pass.

There are two event systems, and an earlier count measured the wrong one:

| system | function | destination | events |
|---|---|---|---|
| vendor telemetry | `loki_telemetry` (`telemetry.sh:211`) | PostHog (`:55`, `:254`) | 7 |
| local bus | `emit_event_json` (`run.sh:2552`) | `.loki/events.jsonl` (`:2586`) | 28 |

The 28 richest events, including `gate_stuck`, `watchdog_alert`,
`provider_failover` and `budget_exceeded`, have no upload path and never leave
the machine. The blocker enum expresses 2 of 6 real failure classes: `timeout`,
`gate_stuck`, `no_tests`, `build_tool` and `unsupported_language` all measure 0,
each with controls, even though `gate_stuck` and `no_tests_run`
(`run.sh:12359`) are already tracked locally.

The richest blocker mapping lives inside doctor (`autonomy/loki:14007-14023`).
The product states the problem itself at `run.sh:3219`: *"loki doctor already
warns on this, but doctor is opt-in and the run is not."* `start` preflights
only Claude auth (`:3295-3314`) and provider presence.

One positive finding: `LOKI_TELEMETRY_ENDPOINT` is overridable
(`telemetry.sh:55`, `docs/PRIVACY.md:70`), so a self-hosted collector is
mechanically possible today.

### 4.4 There is no team, org or fleet primitive

No org or tenant concept exists. `get_fleet_runs`
(`autonomy/lib/cockpit-render.sh:141`) is a local registry of runs on one
machine. `.NET` is additionally invisible to `detect_complexity`
(`run.sh:3453-3457` counts ts, js, py, go, rs, java, rb, php, swift, kt but no
`.cs`), so a .NET monolith counts zero source files and classifies as *simple*.

Every install is a single user on a single repo. Nothing models the second team.

## 5. The backlog

Ranking lens, reusing the one already established in this repo rather than
inventing another: **does this delete a REQUIRED step before first value.**

Provisional. The Delivery Model desk may add items that outrank these, since a
unit-of-sale or accountability primitive could matter more than stack coverage.

| # | Item | Evidence | Deletes which required step |
|---|---|---|---|
| 1 | Make Maven, Gradle and .NET runnable in `enforce_build_check` and `enforce_test_coverage` | 4.1 | A JVM or .NET team's first run can reach full VERIFIED instead of being capped at WITH GAPS |
| 2 | Surface undetected build and test in `start`'s preflight and in `doctor`, naming `LOKI_TEST_COMMAND` | 4.2 | A new team learns why its receipt is degraded without reading source |
| 3 | Local-sink telemetry mode plus a self-hosted collector, shipping `.loki/events.jsonl` | 4.3 | A platform team can see where its teams' first runs die, today impossible |
| 4 | Promote `gate_stuck` and `no_tests_run` into the blocker enum | 4.3 | Failure causes become expressible rather than collapsing to `other` |
| 5 | Add `.cs` to `detect_complexity` | 4.4 | A .NET monolith stops classifying as simple |
| 6 | A team or fleet primitive above the single-repo install | 4.4 | Team 2 inherits team 1's configuration |
| 7 | Receipt feeds a routing decision: auto-approve low-risk, escalate the rest | 3.2, 3.3 | Removes human review load at steady state. DEPENDS on item 1, since the signal must be real before anything can be routed on it |
| 8 | Close the Bun-route skip-as-PASS in `quality_gates.ts:464-467` | 4.2c | A non-JS repo stops receiving `passed: true` for a suite that never ran. Arguably a defect, not a gap, and cheap |
| 9 | Make `LOKI_TEST_COMMAND` reachable outside the `package.json` branch and committable per-project | 4.2 | A Java or .NET team gains any escape hatch at all; today there is none |
| 10 | Let the build and test gates consult `AGENTS.md` | 4.2b | Per-repo onboarding effort starts buying proof, not just agent behaviour |

### 5.1 The commercial backlog

Items 1 to 10 make a single run trustworthy on this estate. These make the
account expandable. They come from the delivery-model evidence in sections 1
and 2, and they are what the founder actually asked for.

| # | Item | Why, from the evidence |
|---|---|---|
| C1 | **Account-level onboarding executed once**, inherited by every team: security review, network topology, CI integration, deploy process, ownership map | The product mirror of the MSA. Highest leverage item here, because it is what makes team 2 nearly free (1.3) |
| C2 | **Land the enterprise agreement at engagement 1**, sized for the account not the pilot: master terms, security posture, data terms, IP assignment, rate card | The entire services advantage is paying for legal, security and procurement once. Then every later team is a work order (1.3) |
| C3 | **Shape the contract and invoice for the labour budget, not the software budget** | A seat licence competes for a separately governed pool; delivered work under a master agreement draws on a pool already sized for it (1.3) |
| C4 | **An agent rate card**: price per unit of delivered verified work, tiered. Tier 0 legacy archaeology per artifact; Tier 1 mechanical (dependency upgrades, framework migrations, test backfill) fixed price per unit; Tier 2 bounded features per accepted criterion; Tier 3 ambiguous work priced as capacity | A rate card makes the second purchase arithmetic rather than negotiation. Be honest that Tier 3 cannot carry an outcome commitment (1.3) |
| C5 | **A generatable one-page SOW** per team engagement: scope, acceptance criteria, capacity, price, the customer's decision-response SLA, and the receipt standard that constitutes proof of delivery | For a services firm an SOW is weeks of human effort. Generated from intake artifacts it becomes a product feature, which is a structural advantage |
| C6 | **Guarantee against the gate, not against hours**: work that does not pass its stated exogenous gates and acceptance criteria is not billable | Cognition guarantees hours-equivalent value on a legal layer disclaiming fitness (1.2). Acceptance-based is cleaner, cheaper to adjudicate, and is the commercial expression of the trust layer we already have |
| C7 | **A remedy that is work, not credits**: failed delivery is remediated free and prioritized | Credits for the thing that failed are not a remedy (1.2). Requires the system to reliably re-open its own failed deliveries |
| C8 | **Account-scoped memory**, not repo-scoped: conventions, deploy topology, owner map, prior decisions, known landmines, reused across teams | Engagement 2 is cheap for Infosys because the people already know the account. We have episodic, semantic and cross-project memory; the gap is promoting it to account scope with explicit cross-team reuse |
| C9 | **Blocked-state as a first-class terminal outcome**, with a named escalation queue and an SLA clock | An agent that cannot detect its own blocked state cannot be given unsupervised capacity (2.3). Our stagnation valve and PAUSE mechanics are the seed |
| C10 | **Spec completeness scoring as a hard gate** at intake, emitting a score plus the list of unresolved decisions | The services firm absorbs ambiguity with humans. This is the only way to absorb it without a human per team (1.4) |
| C11 | **Portfolio control plane**: every stream across every team, in flight, blocked, awaiting decision, delivered with receipt, failed | The single-throat-to-choke surface. Ungovernable means unexpandable (1.4) |
| C12 | **A deliberately small FDE function with a conversion mandate**: every hour absorbing ambiguity must produce a durable product artifact. Track FDE hours per new team; if it is not falling, the model is not working | Cognition staffs 24 of 65 open roles here; Blitzy bundles FDEs into its price outright. Without the conversion loop this degrades into a services firm with worse margins |
| C13 | **Commission a third-party audit of a receipt and publish it**, including what the auditor could not verify | Blitzy has commissioned Quesma audits of both its benchmark runs (2.5). We have none. This is the cheapest credible answer to a buyer comparing the two, and our fail-closed posture is the thing an auditor can actually check |
| C14 | **Publish one worked artifact end to end**, with the receipt alongside the code | Blitzy publishes more verification evidence for a single artifact (curl: 7,312 tests, Miri and ASan clean, 80.05% coverage) than we do publicly, even though ours is machine-checkable and theirs is prose (2.6) |

Item 1 must be built by adding runnable stacks, never by loosening the gate.
Loosening is the fake-green failure this repo exists to prevent, and the
council has already rejected that direction four times.

## Method notes

Two of my own errors this session, both caught by re-measuring, both recorded
because they are the failure mode this doc is most at risk from:

- I first generalized the healing path's seven-branch `detect_test_command` as
  the product's whole stack surface. It has zero references in `run.sh`. The
  main path's `enforce_build_check` knows materially more.
- I first wrote that a `not_run` build "blocks VERIFIED". It caps at VERIFIED
  WITH GAPS. Only a failed build blocks.

A third, from the Competitive desk: an initial claim that the SWE-bench
leaderboard had been dead since mid-2025 was false, caused by a truncated HTML
listing, and the `gh api` re-measurement produced a sharper finding than the
wrong one.

A fourth, from the Blitzy desk, and it is the clearest demonstration of why
this document reads raw sources. Two drafted conclusions were **corrected by
pulling raw bytes instead of trusting a fetch summary**: that Blitzy's 84.95%
run was unaudited (it was audited, by Quesma, stated at line 24 of their own
post), and the omission of Quesma's own chart caption, *"All SWE-Bench Pro
Public scores are self-reported."* Both corrections moved the label in
**opposite** directions, one toward Blitzy and one away. A summariser had
compressed away the two sentences the comparison turns on.

A fifth, structural rather than factual: a surgical heading edit in section 2
spliced a fragment and left a line beginning with a bare comma. It was caught
by re-measuring the file rather than assuming the edit landed, and fixed by
rewriting the block instead of splicing again.
