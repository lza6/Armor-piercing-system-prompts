---
name: revops
description: >
  Use this skill for revenue operations questions, lead lifecycle management, and marketing-to-sales handoff. Triggers on: MQL definition, SQL definition, lead scoring, lead routing, pipeline stage design, CRM automation, data hygiene, deal desk, marketing attribution, funnel analysis, why leads are not converting, why deals are stalling, and any question about aligning marketing and sales around shared data.
---

# Revenue Operations

## Mandatory Content Standards

- Match the output length to the task. For full audits, strategies, plans, or multi-part deliverables, write 1,500 to 10,000 words. For quick tasks, single assets, snippets, or narrow revisions, keep the output concise and provide only the useful variations, rationale, and next steps.
- Write in a way that sounds like a knowledgeable human wrote it. No robotic or templated phrasing.
- Use short sentences. One idea per sentence. One focus per paragraph.
- Use active voice. Never passive constructions.
- Address the reader directly using "you" and "your."
- Use bullet points only when they genuinely improve readability.
- No em dashes. Replace with commas, parentheses, semicolons, or new sentences.
- End every sentence with a period.
- No hashtags, emojis, or asterisks.
- No filler phrases like "in conclusion" or "in summary."
- No warnings or disclaimers.
- No AI cliches: no "game-changer," "unlock," "leverage," "dive into," "cutting-edge," "transformative."
- Use specific examples, data points, and scenarios.
- Pose at least one thought-provoking question.
- Keep paragraphs short. Headers scannable. Structure mobile-friendly.
- Every section connects to a practical next step.

---

## What Revenue Operations Actually Does

Revenue operations (RevOps) is the function that connects marketing, sales, and customer success around shared data, shared processes, and shared accountability. It is not a glorified CRM admin role. It is not a reporting function. RevOps owns the systems, definitions, and handoff rules that determine whether a lead becomes revenue.

Most companies have a version of this problem: marketing generates leads, sales ignores most of them, and everyone argues about pipeline numbers in the quarterly review. RevOps exists to eliminate that argument by making the process clear, measurable, and consistently enforced.

The question worth sitting with is this: if your VP of Marketing and VP of Sales looked at the same CRM report today, would they agree on what it means?

If the answer is no, you have a RevOps problem, not a marketing problem or a sales problem.

---

## Lead Lifecycle Design

The lead lifecycle is the formal path a person travels from first touch to closed revenue. Most teams sketch this on a whiteboard once and never revisit it. That is a mistake. The lifecycle definition is the single document that governs everything downstream, including what gets routed, what gets scored, what triggers automation, and what sales is held accountable for working.

A solid lifecycle has five components:

**Stages with clear definitions.** Every stage must have an entry criterion and an exit criterion. "MQL" is not a definition. "Contact has reached a score of 75 or higher based on demographic fit and behavioral engagement, and has not previously been disqualified" is a definition.

**Stage owners.** Marketing owns Subscriber through MQL. Sales owns SQL through Closed. Customer success owns Onboarding through Renewal. Nobody owns the transition moments, which is exactly where leads die. Fix that.

**Stage timestamps.** Every time a record moves from one stage to the next, log the date. Without timestamps you cannot calculate velocity, which means you cannot diagnose where the funnel is breaking.

**Rejection paths.** When a sales rep decides a lead is not worth working, what happens? If the answer is "it sits there," you have a black hole problem. Define what happens to rejected MQLs, including who reviews the rejection reason and whether those leads re-enter a nurture sequence.

**Re-entry rules.** A prospect who downloaded a whitepaper six months ago and just requested a demo is not a new lead. Your lifecycle needs logic for re-engaging records that already exist in the CRM.

---

## MQL and SQL Definitions

Arguing over MQL and SQL definitions is one of the most expensive sports in B2B marketing. Every week spent in that argument is a week the handoff process stays broken.

Here is a framework that resolves the argument quickly.

An MQL (Marketing Qualified Lead) is a contact that marketing believes meets the minimum criteria for a sales conversation. That definition requires two things: a fit threshold and an intent threshold. Fit means the person works at a company that matches your ICP. Intent means they have taken actions that suggest they are exploring your category, not just consuming content.

An SQL (Sales Qualified Lead) is an MQL that a sales rep has reviewed and accepted as worth actively working. The rep verifies that the contact has confirmed authority, the company has a real problem you solve, there is a timeline, and there is budget access or at least a path to it.

The handoff moment between MQL and SQL is where most B2B funnels bleed. Marketing passes leads that do not meet the real-world criteria sales would apply. Sales rejects leads without documenting why. Neither team learns anything.

Fix this with a Service Level Agreement. The SLA defines:

- How quickly sales must review an MQL after it arrives (common benchmarks are 24 to 48 business hours for inbound, 72 hours for outbound-sourced MQLs).
- What constitutes an accepted lead versus a rejected lead.
- What rejection reason codes exist and what each one means.
- What happens to a rejected lead and who owns the re-engagement decision.

Document this in one page. Put it in the CRM as a reference. Review adherence to it monthly, not quarterly.

---

## Lead Scoring Models

A lead score is a number that summarizes how likely a contact is to be worth a sales conversation. Most lead scoring models are built wrong. They reward behavior without accounting for fit, or they weight actions that correlate with curiosity rather than purchase intent.

Build your model in two layers.

The first layer is demographic fit score. This covers firmographic and persona-level signals. Company size, industry, geography, revenue, job title, and seniority all factor here. A VP of Engineering at a 500-person SaaS company scores differently than an intern at the same company. Assign positive points for signals that match your ICP and negative points for signals that disqualify (for example, a student email domain or a one-person company when your minimum contract size requires 50 seats).

The second layer is behavioral engagement score. This covers what the person has done. Assign points based on the relative purchase intent each action suggests. A pricing page visit signals more intent than a blog read. A demo request signals more intent than a webinar registration. A free trial activation signals more than either.

A working starting point for behavioral scores looks like this:

- Pricing page view: 20 points.
- Product page view: 10 points.
- Demo request form submission: 50 points.
- Free trial activation: 40 points.
- Webinar registration: 5 points.
- Webinar attendance: 10 points.
- Case study download: 8 points.
- Blog read: 2 points.
- Email click: 3 points.
- Repeat website visit within 7 days: 5 points.

Score decay matters. A contact who scored 80 points from activity six months ago and has not visited the site since is not the same as a contact who scored 80 points last week. Build score decay into your model. Most MAP and CRM platforms support time-based decay rules.

Calibrate your MQL threshold by looking backward. Pull your last 50 closed-won deals. Calculate what their lead score was at the time of first sales contact. That distribution gives you a data-driven baseline for where to set the threshold, rather than guessing.

---

## Lead Routing Rules

Lead routing is the process of getting the right lead to the right sales rep as fast as possible. Bad routing wastes leads. It sends an enterprise account to an SMB rep, routes a West Coast prospect to an East Coast rep who calls at 9 AM their time, or drops leads into a round-robin pool where nobody owns the follow-up.

Build routing logic that accounts for:

**Territory.** Define territory by geography, industry vertical, company size, or some combination. Make sure every possible incoming lead maps to exactly one territory. Gaps in territory mapping create leads that fall into no one's queue.

**Account ownership.** If a contact works at a company that already exists in your CRM with an assigned account owner, route the contact to that owner automatically. This prevents reps from unknowingly cold-calling into active accounts or competitors unknowingly working the same opportunity.

**Round-robin within territory.** For territories with multiple reps, define the round-robin logic. Weight the distribution if reps have different capacity targets. Track assignment history so the system does not skip reps.

**Overflow rules.** What happens when a rep is on vacation, at capacity, or has left the company? Define escalation paths. An unworked lead that sits for 48 hours because a rep is out of office is a lead you likely lost.

**Speed-to-lead.** Research consistently shows that contacting a lead within five minutes of form submission dramatically increases conversion rates compared to waiting even an hour. Build your routing to trigger immediate notification to the rep, not just a daily digest email.

Test your routing quarterly. Submit test leads and verify they land where they should.

---

## Pipeline Stage Design

A pipeline stage represents a specific moment in the sales process, not a general feeling about where a deal is. Stages like "In Progress" or "Active" tell you nothing. Stages tied to buyer actions and commitments tell you everything.

A well-designed pipeline has six to eight stages. Fewer and you lose visibility. More and reps stop updating them.

Here is a stage framework for a mid-market SaaS product:

1. MQL Accepted: Sales rep has reviewed the lead and agreed to work it.
2. Discovery Scheduled: A discovery call is booked with a qualified contact.
3. Discovery Complete: The rep has documented BANT or MEDDIC qualification data in the CRM.
4. Solution Presented: A demo or tailored presentation has been delivered.
5. Proposal Sent: A formal proposal or quote has been sent and acknowledged.
6. Negotiation: Commercial terms are actively being discussed.
7. Closed Won: Contract signed or payment received.
8. Closed Lost: Deal ended without a purchase.

Each stage needs an exit criterion. "Discovery Complete" means the rep has logged answers to five specific discovery fields in the CRM, not that the call happened. This distinction matters because it forces data quality and makes your pipeline predictable.

Attach a probability percentage to each stage based on your actual historical win rates, not best guesses. If 30 percent of deals that reach Proposal Sent close, your stage probability is 30 percent, not 50 percent because it "feels close."

---

## CRM Automation

CRM automation handles the repetitive handoff and notification tasks that humans forget or deprioritize. The goal is not to replace human judgment but to ensure that process steps happen consistently and that nothing falls through.

The most valuable automations to build first:

**MQL notification.** When a record's lead score crosses your MQL threshold, immediately notify the assigned sales rep via email and Slack. Include the key signals that drove the score, the company name, the contact's role, and a link to the CRM record.

**SLA tracking.** Start a timer when an MQL is created. If the rep has not logged an activity or moved the stage within your SLA window, trigger an escalation alert to the rep's manager.

**Stage progression reminders.** If a deal has been in the same stage for longer than your average stage duration, alert the rep. A deal sitting in "Proposal Sent" for 30 days when your average is 10 days is a deal in trouble.

**Data enrichment on create.** When a new contact or company is created, trigger enrichment from a tool like Clearbit, ZoomInfo, or Apollo to populate missing firmographic fields. This prevents reps from spending time on manual research.

**Closed lost triggers.** When a deal is marked Closed Lost, automatically enroll the contact in a long-term nurture sequence, tag the account for future re-engagement review, and send a loss survey if appropriate.

**Deal desk triggers.** When a deal reaches a certain dollar threshold or includes non-standard terms, automatically notify the deal desk or CFO for approval routing.

Document every automation in a master list. Include what triggers it, what it does, who it notifies, and when it was last reviewed. Automations that nobody documents are automations that nobody maintains.

---

## Data Hygiene

Bad data is the most reliable way to make RevOps fail. Routing rules break when required fields are blank. Lead scores are wrong when email domains are misspelled. Attribution reports lie when UTM parameters are inconsistent.

Data hygiene is not a one-time cleanup project. It is an ongoing operational practice.

Build these processes into your regular RevOps rhythm:

**Deduplication.** Run a deduplication check monthly. Most CRMs have native dedup tools or integrate with tools like LeanData or Dedupely. Duplicate records inflate your MQL counts, cause leads to be routed to multiple reps, and corrupt funnel reporting.

**Required field enforcement.** Define which fields must be populated before a record can advance through each pipeline stage. Enforce this at the CRM level, not just in policy. If a rep cannot move a deal to "Discovery Complete" without filling in the company size and use case fields, the data stays clean.

**UTM parameter governance.** Create a UTM naming convention and document it. Enforce it across every marketing channel. Inconsistent UTMs (utm_source=LinkedIn versus utm_source=linkedin versus utm_source=LI) create attribution gaps that make channel performance reporting unreliable.

**Email validation.** Validate email addresses at the point of form submission. Remove hard bounces from your database monthly. A contact database full of bad emails depresses deliverability scores, which affects everyone's inbox placement.

**List hygiene for segments.** Review your active CRM segments and audience lists quarterly. Segments built on criteria that no longer reflect your ICP will generate MQLs that sales rejects at high rates.

The ROI of data hygiene is not abstract. Calculate your average MQL value and multiply by the number of MQLs your team generates per month. Then estimate what percentage of those are built on bad data (low acceptance rates, high bounce rates, missing firmographic fields). That number is the cost of letting hygiene slip.

---

## Deal Desk Processes

The deal desk manages complex, non-standard deals that require pricing exceptions, custom contract terms, legal review, or executive approval. Without a defined deal desk process, these deals either stall while reps wait for approvals, or reps make commitments they are not authorized to make.

A functional deal desk process includes:

**Submission criteria.** Define exactly which deal characteristics require deal desk involvement. Common triggers include deals above a certain ACV, multi-year contracts, custom payment terms, volume discounts beyond standard tiers, or any contract language changes.

**Submission format.** Reps should submit a deal desk request using a standardized form that captures deal size, customer name, the specific exception being requested, the business justification, and the customer's decision timeline.

**Turnaround SLA.** The deal desk commits to a response time based on deal complexity. Simple pricing approvals might have a 4-hour SLA. Contracts requiring legal review might have 48 hours. Publish these SLAs so reps know what to tell customers.

**Approval routing.** Map which approvals require which stakeholders. A 10 percent discount might need only a sales manager. A 30 percent discount might need the VP of Sales and CFO. A custom SLA guarantee might need the head of Customer Success. Document this routing matrix.

**Tracking.** Every deal desk request should be logged in the CRM or a connected tool. Track volume by rep, deal size, exception type, and approval status. If one rep generates 40 percent of all deal desk requests, that is a coaching conversation. If discount requests are clustering in a specific product tier, that is a pricing conversation.

---

## Marketing Attribution in the CRM

Attribution tells you which marketing activities contributed to closed revenue. Every company has a version of this problem: the CMO asks what marketing generated last quarter, and the answer is a guess wrapped in a spreadsheet.

Get attribution right by solving it at the CRM level, not the spreadsheet level.

**First touch attribution** credits the first marketing interaction a contact had before becoming a customer. This model shows you which channels create awareness and bring new prospects into your funnel.

**Last touch attribution** credits the final marketing interaction before an MQL or opportunity was created. This model shows you which activities drive conversion moments.

**Multi-touch attribution** distributes credit across all marketing touchpoints in the buyer's journey. This is the most accurate model but also the most complex to implement. Tools like Bizible (now Marketo Measure), HubSpot Attribution, or Salesforce's native campaign influence features support this.

**Linear attribution** gives equal credit to every touchpoint. It is a reasonable middle ground when you lack the data to support true influence weighting.

Whichever model you choose, implement it consistently. The model matters less than the consistency. Changing attribution models mid-year makes year-over-year comparison impossible.

Practical implementation steps:

1. Ensure every CRM lead has a populated Lead Source field that maps to a defined channel taxonomy.
2. Connect your marketing automation platform to your CRM so campaign interactions are logged as activities on contact records.
3. Create a campaign influence report that shows pipeline and closed-won revenue associated with each campaign, not just MQL volume.
4. Report pipeline influence monthly alongside MQL volume so marketing is accountable to revenue, not just lead generation metrics.

---

## Diagnosing Funnel Breakdowns

When a pipeline is not producing revenue, the problem is almost always visible in the funnel data. You just need to know where to look.

Work through this diagnostic sequence:

**Is lead volume the problem?** Compare MQL volume this period versus the same period last year. If MQLs are down significantly, the problem is in marketing's ability to generate demand. Look at traffic, conversion rates by channel, and form submission rates.

**Is lead quality the problem?** Look at the MQL-to-SQL acceptance rate. If marketing is generating MQL volume but sales is rejecting more than 30 to 40 percent of them, the scoring model or the MQL definition is too loose. Pull the rejection reason codes and find the patterns.

**Is follow-up speed the problem?** Pull average time-to-first-contact for MQLs by rep and by territory. If your average is over 48 hours, you are losing leads to faster competitors. Check SLA adherence by rep.

**Is discovery the problem?** Look at the Discovery Scheduled to Discovery Complete conversion rate. If reps are booking discovery calls but not logging outcomes, you have a coaching and process adoption problem, not a lead quality problem.

**Is late-stage conversion the problem?** Look at your Proposal Sent to Closed Won rate. If it is unusually low, examine deal size (are reps sending proposals to companies that cannot afford the product?), proposal quality (is your proposal template competitive?), and competitive displacement (are specific competitors showing up in late-stage losses?).

**Is the right segment the problem?** Break your funnel conversion rates by company size, industry, lead source, and geographic territory. A company-wide average that looks acceptable often hides a segment that is performing terribly and dragging the business down.

Each diagnostic step produces a specific action. If follow-up speed is the problem, the action is enforcing the SLA and adding rep notifications. If lead quality is the problem, the action is tightening the scoring model or the MQL definition. Never let a diagnostic conversation end without assigning an owner and a deadline.

---

## RevOps Reporting Cadence

RevOps produces three types of reports on three different cadences.

**Weekly operational reports** cover the metrics that require immediate action. These include MQL volume, MQL acceptance rate, SLA compliance rate, and pipeline created this week. Share these in the Monday team meeting. The goal is to catch breakdowns before they compound.

**Monthly funnel reports** cover stage-by-stage conversion rates, average deal velocity, marketing attribution by channel, and rep-level pipeline metrics. These reports support coaching decisions and resource allocation decisions.

**Quarterly strategic reviews** compare actual pipeline and revenue versus plan, analyze trends in win/loss patterns, assess ICP accuracy based on actual closed-won data, and recommend structural changes to scoring models, territory design, or stage definitions.

The mistake most teams make is running only monthly or quarterly reporting. By the time you see a problem in a monthly report, you have already lost four weeks of leads to it.

---

## RevOps Tech Stack Basics

You do not need an expensive tool stack to run effective RevOps. You need the right tools connected correctly.

The core stack for a team of 10 to 50 sales reps typically includes:

- A CRM (Salesforce, HubSpot, or Pipedrive depending on complexity and budget).
- A marketing automation platform (HubSpot, Marketo, or Pardot) connected bidirectionally to the CRM.
- A lead enrichment tool (Clearbit, ZoomInfo, Apollo, or Clay) that auto-populates missing fields.
- A sales engagement tool (Outreach, Salesloft, or Apollo) for sequence automation and activity tracking.
- A BI or reporting layer (Salesforce reports, HubSpot reporting, Tableau, or Looker) for funnel and attribution analysis.

The connections between these tools matter more than which tools you choose. A Salesforce instance that does not sync with your marketing automation platform creates two sources of truth, which means you have no source of truth.

Map your data flows before adding new tools. Every new tool is a new integration point that can break.

---

## Where to Start

If RevOps is new for your team or your current setup is broken, start with the three things that produce the fastest impact:

1. Agree on MQL and SQL definitions in writing and get both the VP of Marketing and VP of Sales to sign off.
2. Build lead score decay so old, cold scores do not clog your MQL queue.
3. Create a weekly SLA compliance report and share it with sales leadership.

These three actions address the three most common RevOps failures: definitional disagreement, stale pipeline data, and unaccountable follow-up processes. Every other improvement builds on these foundations.
