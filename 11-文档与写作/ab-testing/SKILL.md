---
name: ab-testing
description: >
  Use this skill when planning, designing, or analyzing A/B tests and growth experiments. Trigger phrases: "run an A/B test," "design an experiment," "write a hypothesis," "calculate sample size," "build an experimentation backlog," "ICE scoring," "statistical significance," "multivariate test," "growth experimentation," "test velocity."
---

# A/B Testing and Growth Experimentation

## Mandatory Content Standards

- Match the output length to the task. For full audits, strategies, plans, or multi-part deliverables, write 1,500 to 10,000 words. For quick tasks, single assets, snippets, or narrow revisions, keep the output concise and provide only the useful variations, rationale, and next steps.
- Write in a way that sounds like a knowledgeable human wrote it. No robotic or templated phrasing.
- Use short sentences. One idea per sentence. One focus per paragraph.
- Use active voice. Never passive constructions.
- Address the reader directly using "you" and "your."
- Use bullet points only when they genuinely improve readability.
- Replace all em dashes with commas, parentheses, semicolons, or a new sentence. No hidden Unicode characters.
- End every sentence with a period.
- No hashtags, emojis, or asterisks.
- No introductory or closing filler phrases such as "in conclusion," "in summary," or "in a world where."
- No warnings, notes, or disclaimers. Stick to requested output.
- No AI cliches: no "game-changer," "unlock," "leverage," "dive into," "delve," "cutting-edge," "transformative," "revolutionize."
- No excessive adjectives or adverbs. Let specifics do the work.
- No broad generalizations. Every claim tied to specific context.
- Use specific examples, data, and scenarios.
- Pose at least one thought-provoking question per skill.
- Mobile-friendly: short paragraphs, clear headers, scannable.
- Practical and actionable. Every section connects to a next step.

---

## What This Skill Covers

This skill walks you through the complete experimentation lifecycle: identifying what to test, writing hypotheses with teeth, sizing tests correctly, running them without bias, reading results accurately, and building a program that compounds results over time.

A/B testing is not just a tactic. It is a decision-making discipline. Teams that treat it as a one-off tactic get occasional wins. Teams that treat it as a system get compounding ones.

---

## Why Most A/B Tests Fail

Before covering how to run tests well, it is worth being honest about why most tests produce nothing useful.

The most common failure is running underpowered tests. A company gets excited about a headline change, runs it for four days, sees a 12% lift, and calls it a win. Then they ship the winner and see no meaningful change in revenue. What happened? The test never reached statistical significance. The 12% lift was noise.

The second most common failure is testing the wrong things. Teams often test button colors and font sizes because those are easy to implement. But a button color test on a checkout page is unlikely to move revenue by more than a fraction of a percent even if it wins. Testing the value proposition on the same page could move it by 20%.

The third failure is running tests one at a time. If you run one test per month, you get 12 learning cycles per year. Teams that run 50 tests per year compound their learning at a rate that makes the monthly team irrelevant within two years.

The fourth failure is not having a clear owner. When multiple people can stop a test, change its parameters, or interpret results differently, tests become political rather than scientific.

Ask yourself: how many tests did your team ship in the last 90 days? If the answer is fewer than five, velocity is the first problem to solve.

---

## The Hypothesis Framework

Every test needs a hypothesis before it starts. Not a guess. A structured prediction with a mechanism.

A weak hypothesis sounds like: "We think changing the CTA button to green will increase conversions."

A strong hypothesis sounds like: "We believe that changing the CTA from 'Get Started' to 'Start Your Free Trial' will increase signups on the pricing page by at least 10%, because the current CTA is ambiguous about whether the product costs money, and users who are unsure of pricing are less likely to click."

The structure is: "We believe [change] will cause [specific metric] to [increase or decrease] by [minimum meaningful amount] because [mechanism]."

The mechanism is the most important part. It forces you to articulate why you think the change will work. If you cannot articulate the mechanism, you do not understand the problem well enough to design a useful test.

Write the mechanism from the user's perspective. "Users who see X are more likely to do Y because they feel Z." Ground it in behavior, not assumptions about what looks better.

Every hypothesis should also specify a minimum detectable effect. If a 2% lift in conversion rate would be meaningful, design the test to detect a 2% lift. If only a 15% lift would justify the engineering effort, size the test accordingly.

---

## Prioritization: ICE Scoring

When you have more test ideas than time to run them, you need a prioritization framework. ICE scoring is the most widely used and practical one.

ICE stands for Impact, Confidence, and Ease.

Score each dimension on a 1 to 10 scale:

Impact: How much will this test move the needle if it wins? A test on your highest-traffic page with the weakest current conversion rate should score higher than a test on a low-traffic confirmation page.

Confidence: How confident are you that this test will produce a meaningful result? Confidence is driven by qualitative evidence (user interviews, session recordings, heatmaps), quantitative evidence (funnel data, exit rates), and precedent from similar tests in your industry.

Ease: How fast can you ship this test? A headline change that takes two hours to implement scores higher than a full page redesign that takes six weeks.

Multiply the three scores together. Rank your backlog by ICE score. Run the top-ranked tests first.

ICE is not perfect. A test with a high ease score but low impact might inflate your velocity without producing useful learnings. Some teams weigh impact more heavily or add a fourth dimension like "Learning Value" to capture tests that might not move the primary metric but will teach you something important about user behavior.

The goal of ICE scoring is to make prioritization fast and explicit. It replaces debates about whose idea to test first with a shared, transparent ranking system.

---

## Sample Size Calculation

Running a test before you have enough data is like stopping a clinical trial after 20 patients. The results are meaningless.

Sample size depends on four inputs:

1. Your baseline conversion rate. What percentage of users currently convert?
2. Your minimum detectable effect (MDE). What is the smallest lift that would be meaningful?
3. Your desired statistical significance level. Most teams use 95% (p = 0.05).
4. Your desired statistical power. Most teams use 80%, meaning the test has an 80% chance of detecting a real effect if one exists.

Here is a concrete example. Suppose your current landing page converts at 3%. You want to detect a 10% relative lift, which means you are looking for a 0.3 percentage point increase to 3.3%. With a 95% significance threshold and 80% power, you need approximately 30,000 users per variant. That means 60,000 total users across both variants.

If your page gets 1,000 visitors per week, that test will take 60 weeks to complete. That is not a feasible test at that MDE. You have two options: raise your MDE (accept that you will only detect larger effects) or find a higher-traffic page to test on.

Use a sample size calculator for every test before you start. Evan Miller's calculator at evanmiller.org is reliable and free. Type in your baseline rate, your MDE, and your significance and power levels. The output tells you how many users per variant you need.

If your sample size requirement exceeds your traffic, do not run the test. Move to a higher-traffic surface or accept a larger MDE.

---

## Test Design Fundamentals

A well-designed test isolates a single variable. If you change the headline, the subheadline, and the CTA in the same test, you will not know which change drove the result.

There are three valid reasons to test multiple changes at once:

1. You are running a multivariate test (MVT) with enough traffic to support it.
2. You are testing a completely new page design where you want to learn whether the new direction works before optimizing individual elements.
3. The changes are inseparable, meaning one change does not make sense without the other.

For everything else, isolate the variable.

Your control (the A) should always be your current live version. Never test two new versions against each other without also including the original as a control. Otherwise, you risk declaring a winner that is actually just less bad than the other variant, when neither is better than the original.

Randomize assignment at the user level, not the session level. If a user sees the control version on Monday and the variant on Tuesday, they corrupt your data. Use a consistent hashing method tied to a user identifier (logged-in user ID or a stable cookie).

Avoid launching tests right before major events. If you start a homepage test on November 25th, your Cyber Week traffic will likely behave very differently from your baseline. Wait until traffic patterns normalize before drawing conclusions.

Run tests for a minimum of one full business cycle. For B2B companies, that usually means two to four weeks minimum, since purchasing behavior on Mondays differs from Fridays. For e-commerce, run through at least one full week to capture weekend behavior.

---

## Statistical Significance vs. Practical Significance

These two concepts are different, and confusing them is expensive.

Statistical significance tells you the probability that your result is not due to random chance. A 95% confidence level means there is only a 5% chance the observed difference between control and variant occurred randomly.

Practical significance asks: is the effect large enough to matter?

A test can be statistically significant but practically meaningless. If you run a test on a high-traffic page and detect a 0.02% lift in conversion rate at 99% confidence, the result is statistically rock-solid but commercially irrelevant. Shipping that winner will not change your business.

Conversely, a test can fail to reach statistical significance but still teach you something important. If your variant performed worse across every segment you looked at, you have learned that the direction was wrong, even if the p-value did not clear 0.05.

Evaluate results on both dimensions. Always ask: if this result is real, does it change anything?

---

## Building an Experimentation Backlog

A strong experimentation backlog looks like a product roadmap, not a wish list.

Structure each item in your backlog with:

- A clear hypothesis (with mechanism)
- The page or surface being tested
- The primary metric
- The secondary metrics you will monitor
- The estimated sample size needed
- The ICE score
- The owner (the person responsible for running and shipping the test)
- The current status (ideas, in design, running, concluded, shipped)

Review and replenish the backlog weekly. Aim to always have at least six to eight fully written hypotheses ready to ship at any moment. This prevents velocity from stalling because a test concluded and no one had a next idea ready.

Sources for new test ideas:

Customer interviews surface objections and confusion that you did not know existed. If five separate customers say they did not understand what your product did when they first landed on your site, that is a signal to test your above-the-fold messaging.

Session recordings (Hotjar, FullStory, Microsoft Clarity) show where users hesitate, rage click, or abandon. These behaviors often point to friction that quantitative data alone cannot diagnose.

Funnel analysis shows where users drop off. If 60% of users who reach your pricing page leave without clicking anything, that page is worth a prioritized series of tests.

Competitor research shows what alternatives are testing. Tools like SimilarWeb, Semrush, and direct observation of competitor pages over time show directional changes in their messaging and design. You cannot know whether their changes are winning, but you can generate hypotheses based on what they are trying.

Customer support tickets contain the exact language customers use when they are frustrated or confused. Mine support tickets for recurring objections. Then test copy that directly addresses those objections.

---

## Experiment Velocity

The teams that generate the most learning over time are not the teams with the best individual experiments. They are the teams that run the most experiments with acceptable rigor.

Velocity requires three organizational inputs:

First, a streamlined shipping process. If every test requires a design review, an engineering sprint, and a QA cycle, you will never run more than a few tests per month. Invest in tooling that lets marketers and growth team members launch tests without engineering involvement. Tools like Optimizely, VWO, Convert, and LaunchDarkly reduce the cycle time between idea and live test.

Second, a culture of low attachment to outcomes. Teams that become emotionally invested in winning experiments lose objectivity. If a team member's career success depends on their experiments winning, they will find ways to rationalize stopping tests early or interpreting results favorably. Build a culture where a well-designed experiment that produces a clear null result is considered a success.

Third, documentation of learnings. Every concluded test should produce a short write-up: the hypothesis, the result, and the insight. These write-ups compound. In 18 months, they become a knowledge base that prevents repeating failed approaches and accelerates hypothesis generation.

A team running 50 experiments per year with a 30% win rate learns more in one year than a team running 12 experiments per year with a 50% win rate. Volume matters.

---

## Multivariate Testing

Multivariate testing (MVT) lets you test multiple variables simultaneously and analyze interactions between them.

For example, you might want to test two headline variants, three image variants, and two CTA variants on a landing page simultaneously. A full factorial MVT would test all 12 combinations (2 x 3 x 2).

The benefit is efficiency. Instead of running three separate sequential tests, you get data on all combinations at once. You also see whether combinations produce effects that neither element would produce alone. A certain headline might perform average on its own, but paired with a specific image, it might significantly outperform everything else.

The downside is traffic. MVTs require substantially more traffic than A/B tests. A 12-combination test needs roughly 12 times the traffic of a simple two-variant A/B test to reach the same confidence level per combination. Most companies outside of the largest e-commerce and SaaS platforms do not have the traffic for true MVTs.

A practical alternative is a sequential testing approach. Run the most impactful variable first, lock in the winner, then test the next most impactful variable against the winner. This is slower than a full MVT, but it works at lower traffic volumes and still isolates learnings clearly.

---

## Common Testing Mistakes

Running tests too short. The most common and most damaging mistake. Always calculate sample size before you start. Never stop early because the results look good.

Peeking at results. Checking your results every day and stopping when you see statistical significance is a form of p-hacking. It inflates your false positive rate significantly. Set a stopping rule before you start: either a sample size threshold or a minimum runtime, and stick to it.

Testing on the wrong metric. If your primary conversion event is a purchase but you are measuring test performance on clicks to the next step, you might optimize for clicks while hurting purchases. Measure what actually matters.

Novelty effects. Users behave differently toward new things. If you change a page significantly, users who are familiar with the old version may interact with the new version out of curiosity rather than genuine preference. Give tests enough time to move past the novelty phase.

Survivorship bias in segment analysis. After a test concludes, segment your results by device, traffic source, user type, and acquisition channel. Sometimes a test loses overall but wins significantly in a specific segment. That segment win is a signal worth following up on. But be careful: if you test enough segments, you will find a winner by chance. Use segment analysis to generate hypotheses, not to declare post-hoc winners.

---

## Reading Test Results

When a test concludes, work through these questions in order.

Did the test reach your pre-set sample size or runtime? If not, the result is inconclusive regardless of what the data shows.

What is the p-value, and does it clear your significance threshold? At 95% significance, you want p < 0.05. At 99% significance, you want p < 0.01.

What is the confidence interval around the lift? A result of "14% lift, 95% CI [2%, 26%]" is a real but uncertain win. A result of "14% lift, 95% CI [13%, 15%]" is a precise win. Wide confidence intervals mean you should consider running a follow-up test or shipping cautiously.

What happened to secondary metrics? If your variant improved signups but increased support ticket volume by 40%, the variant may have created confusion that the primary metric did not capture.

Are the results consistent across key segments? If the variant wins strongly on mobile but loses on desktop, you might ship only to mobile users. That is a more nuanced outcome than a global winner or loser.

What does this result tell you about your user, your product, or your market? Document the insight, not just the outcome.

---

## When to Stop a Test Early

You should only stop a test early in two situations.

First, if the variant is causing meaningful harm. If your variant reduces a critical metric by 30% or more within the first 20% of the planned sample size, the probability that it will recover is low. Stopping early to protect revenue is defensible in this case.

Second, if an external event has corrupted the test. A PR crisis, a major competitor announcement, or a site outage during the test introduces confounding variables that make results uninterpretable. Stop, investigate, and restart.

In all other cases, run the test to completion. Discipline here separates teams that build reliable institutional knowledge from teams that accumulate a collection of ambiguous results.

---

## Reporting and Stakeholder Communication

Test results need to be communicated clearly to non-technical stakeholders. Build a standard template for experiment summaries that includes:

- The hypothesis in plain language
- What you changed (with screenshots of control and variant)
- The primary metric result with confidence interval
- The secondary metric results
- The decision: ship, do not ship, or follow up
- The insight: what does this result tell you?
- The next hypothesis this result generates

Share experiment summaries in a shared document or wiki that the whole company can access. Executive teams that see a consistent stream of experiment results develop trust in the program. Teams that only report results when they win create skepticism.

Quarterly, report on experiment velocity (tests shipped), win rate, estimated revenue impact of winning tests, and the top three insights generated. This framing positions experimentation as a strategic capability rather than a marketing tactic.

---

## Tools Reference

Testing platforms: Optimizely, VWO, Convert, AB Tasty, LaunchDarkly (for feature flags and server-side testing), GrowthBook (open source).

Sample size calculators: Evan Miller's calculator, AB Testguide, Optimizely's sample size tool.

Session recording and heatmaps: Hotjar, FullStory, Microsoft Clarity, LogRocket.

Analytics: Google Analytics 4, Amplitude, Mixpanel, Heap.

Most teams at the growth stage (under $10M ARR) can run a strong experimentation program with Convert or VWO for frontend testing, Amplitude or Mixpanel for behavioral analytics, and Hotjar for qualitative signal. Enterprise teams usually add LaunchDarkly for server-side and feature flag testing.

---

## Getting Started

If your team does not have an active experimentation program, start here.

Week one: audit your funnel. Find the three pages or steps with the highest drop-off rate. These are your highest-impact testing surfaces.

Week two: run five customer interviews. Ask customers what they were unsure about when they first evaluated your product. Their answers will generate your first six hypotheses.

Week three: write your first five hypotheses using the structure above. ICE score them. Pick the top one and design the test.

Week four: launch your first test. Calculate the sample size. Set a stopping rule. Tell your team the date you will call the result.

From that point, your goal is to maintain a minimum of one live test at all times and to launch a new test within 48 hours of calling a result on any running test. Build from there.
