---
name: analytics
description: >
  Use this skill for setting up, auditing, or improving marketing analytics tracking. Trigger phrases: "analytics setup," "GA4," "Google Analytics," "UTM parameters," "conversion tracking," "Google Tag Manager," "GTM," "tracking plan," "Mixpanel," "Segment," "event tracking," "attribution setup," "broken tracking," "analytics audit," "missing conversions," "marketing measurement."
---

# Marketing Analytics: Setup, Auditing, and Optimization

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

This skill covers the full marketing analytics stack: tracking plans, GA4 implementation, Google Tag Manager, UTM parameters, conversion tracking, attribution, and product analytics tools like Mixpanel and Segment. It also covers diagnosing broken or missing tracking and building measurement infrastructure that connects marketing spend to business outcomes.

---

## Why Most Marketing Analytics Are Wrong

Before covering how to build solid tracking, it is worth understanding why most companies have inaccurate analytics.

The most common problem is unstructured growth. A company launches with GA4 installed. Over three years, fifteen different people add tags, events, and pixels to the site in different ways with different naming conventions. Events fire on some pages but not others. UTMs are used inconsistently. Conversions are tracked in five different places with overlapping logic.

The result is analytics that cannot be trusted. When the data is questioned, different team members have different explanations. Decisions get made on intuition because the data is not reliable.

The second common problem is tracking what is easy to track rather than what matters. Pageviews and sessions are the default metrics because they are easy to measure. Qualified lead volume and downstream conversion rates are what actually predict revenue, but they require intentional setup.

Think about this honestly: can you right now, in five minutes, pull a number that shows what your paid search campaign contributed to closed revenue in the last 90 days? If not, your analytics infrastructure is not supporting the decisions you need to make.

---

## The Tracking Plan: Start Here

A tracking plan is a documented specification of every event you track, why you track it, what properties it carries, and where it fires. It is the blueprint that keeps your analytics consistent as your team grows.

Every tracking plan includes these elements for each event:

Event name: use a consistent naming convention. The most common is snake_case with a verb-object structure: "form_submitted," "trial_started," "checkout_completed." Never use spaces. Never use different names for the same event in different locations.

Trigger: exactly what user action causes this event to fire? "User clicks the 'Get Started' button on the homepage" is a clear trigger. "User does something on the pricing page" is not.

Properties: what data accompanies this event? For a "form_submitted" event, properties might include form_type, page_url, lead_source, and utm_campaign. Document every property and its expected values.

Destination: where does this event go? GA4? Segment? Mixpanel? Facebook Pixel? All of the above? Document the destinations.

Priority: is this event a core conversion event (highest priority), a micro-conversion (medium priority), or a behavioral event for analysis (lower priority)?

Owner: who is responsible for maintaining this event's implementation?

Build your tracking plan in a shared Google Sheet or Notion document. Keep it updated whenever implementation changes. An outdated tracking plan is nearly as problematic as no tracking plan.

---

## GA4 Implementation

Google Analytics 4 replaced Universal Analytics as Google's standard web analytics platform. It uses an event-based data model where every interaction is an event.

### Core Setup Checklist

Data stream configuration: create a web data stream for your website. Confirm the measurement ID (starts with G-) matches what is installed on your site.

Data retention: GA4 defaults to 2 months of data retention. Increase this to 14 months under Admin > Data Settings > Data Retention. Do this immediately after setup. You cannot retroactively recover data beyond the retention window.

Internal traffic filtering: exclude your own team's IP addresses from data. Unfiltered internal traffic inflates your numbers and distorts behavioral data, especially on low-traffic sites. Create an IP filter under Admin > Data Streams > Configure Tag Settings > Define Internal Traffic, then activate it under Admin > Data Filters.

Cross-domain tracking: if your website spans multiple domains (your marketing site at example.com and your app at app.example.com), configure cross-domain tracking. Without it, sessions break at the domain boundary and conversions attributed to your app look like direct traffic.

Google Signals: enabling Google Signals allows GA4 to use demographic data for users signed into Google accounts and enables cross-device reporting. Enable under Admin > Data Settings > Data Collection.

Linked products: link GA4 to Google Ads for conversion import, to Google Search Console for organic search data, and to BigQuery for raw event export if you need custom analysis.

### Conversion Events

GA4 does not have "goals" like Universal Analytics. Instead, you mark specific events as conversion events.

Go to Admin > Events. Find the event you want to mark as a conversion (for example, "generate_lead" or "purchase"). Toggle "Mark as conversion."

For events that do not fire automatically, you need to create them. GA4 auto-collects several events: page_view, first_visit, session_start, user_engagement, scroll, click, file_download. Everything else requires custom implementation.

The most important conversion events to set up for a SaaS business:

- sign_up (completed registration)
- trial_started
- subscription_activated (paid conversion)
- demo_scheduled

For e-commerce:

- add_to_cart
- begin_checkout
- purchase (with revenue parameter)

### Custom Events in GA4

Create custom events using Google Tag Manager (covered below) or by adding the gtag.js event command directly in your site code.

The gtag.js format is: gtag('event', 'event_name', { 'parameter_name': 'parameter_value' });

For example, tracking a form submission: gtag('event', 'form_submitted', { 'form_type': 'contact', 'page_location': window.location.href });

Custom parameters in events are not visible in GA4 reports by default. You must register them as custom dimensions or metrics under Admin > Custom Definitions. Create a custom dimension for each parameter you want to see in reports.

---

## Google Tag Manager

Google Tag Manager (GTM) is a tag management system that lets you add and update tracking code on your website without editing code directly for every change.

GTM operates on three concepts: tags (the tracking code that fires), triggers (the conditions that cause a tag to fire), and variables (dynamic values that tags and triggers reference).

### Setting Up GTM

Create a GTM account and container at tagmanager.google.com. Install the GTM snippet on every page of your site: one snippet in the head and one snippet in the body, immediately after the opening body tag. Once GTM is installed, use it to manage all other tracking tags.

Install your GA4 tag through GTM using the Google Tag type. Enter your GA4 Measurement ID. Set the trigger to "All Pages." This replaces any direct GA4 installation.

### Trigger Types

Page View trigger: fires when a page loads. Use for basic pageview tracking and for firing tags on specific pages (for example, firing a conversion event on your thank-you page).

Click trigger: fires when a user clicks an element. Configure by CSS class, element ID, or any click variable. Use to track CTA clicks, button clicks, or navigation clicks.

Form Submission trigger: fires when a form is submitted. Important: test this carefully. Some forms use JavaScript to submit without page reload, which requires a different trigger configuration.

Custom Event trigger: fires when a custom dataLayer.push() event is triggered from your site code. The most flexible and reliable method for tracking complex interactions.

Scroll Depth trigger: fires when a user scrolls to a specified percentage of a page. Use to measure content engagement.

Timer trigger: fires after a specified amount of time on the page. Use to detect engaged sessions vs. bounces with timing nuance.

### The Data Layer

The data layer is a JavaScript array (window.dataLayer) that your site uses to pass information to GTM. When your site triggers an event, it pushes an object to the data layer. GTM reads the data layer and fires the appropriate tags.

The data layer is the most reliable way to pass custom information to your tags. If your site knows a user's plan type, account ID, or order value, push that information to the data layer and read it in GTM with a Data Layer Variable.

A typical dataLayer push looks like: window.dataLayer.push({ 'event': 'trial_started', 'plan_type': 'pro', 'user_id': '12345' });

Configure a Data Layer Variable in GTM to read plan_type and user_id, then pass those values as parameters in your GA4 event tag.

### Debugging GTM

Use GTM Preview mode (the Debug button in the GTM interface) to test your implementation before publishing. Preview mode shows you exactly which tags fire on each page, which triggers activated them, and what variable values they received.

Use GA4's DebugView (Admin > DebugView) simultaneously. DebugView shows real-time events as they fire from your browser. This combination lets you verify that GTM tags are firing and that GA4 is receiving the events correctly.

---

## UTM Parameters

UTM parameters are the query string parameters that attribute website traffic to specific campaigns. They are the foundation of cross-channel marketing attribution.

The five UTM parameters:

utm_source: the platform or channel sending traffic. Examples: google, facebook, linkedin, newsletter, partner.

utm_medium: the channel type. Examples: cpc (paid search), paid_social, email, organic, referral.

utm_campaign: the specific campaign. Use a consistent naming format: "q1-2025-brand-awareness" or "competitor-brand-nov24."

utm_content: differentiates ads within a campaign. Use to distinguish ad variations or different placements.

utm_term: the keyword for paid search campaigns. Often populated automatically by Google Ads.

### UTM Naming Conventions

Inconsistent UTM naming is one of the most common analytics problems. If one person uses utm_source=Facebook and another uses utm_source=facebook, they create two separate traffic sources in GA4 that should be the same.

Establish and document a naming convention before you start. Common rules:

- Use lowercase only
- Use hyphens instead of spaces or underscores (GA4 handles hyphens cleanly)
- Keep campaign names concise but descriptive
- Use consistent abbreviations (cpc not paid-search, email not newsletter)

Build a UTM builder spreadsheet or use a tool like Google's Campaign URL Builder. Never let individuals create UTM parameters from memory.

Audit your UTM usage quarterly. In GA4, go to Acquisition > Traffic Acquisition and filter by session_campaign. Look for near-duplicate campaign names that should be unified.

### Auto-Tagging vs. Manual UTMs for Google Ads

Google Ads auto-tagging (gclid parameter) is the preferred method for Google Ads attribution. It passes more data than manual UTMs and links directly to your Google Ads account when you have GA4 linked.

Do not use both auto-tagging and manual UTMs on the same Google Ads URLs. They conflict. Use auto-tagging for Google Ads and reserve manual UTMs for all other channels.

---

## Attribution Setup

Attribution determines how credit for conversions is assigned across touchpoints in the customer journey.

GA4's default attribution model is data-driven, which uses Google's algorithm to assign fractional credit. You can change this under Admin > Attribution Settings.

For most marketing teams, the most important attribution setup decision is not which model to use in GA4 but whether to rely solely on GA4 or to add a multi-touch attribution layer.

### Limitations of GA4 Attribution

GA4 attribution is session-based and limited to what the GA4 tag observes. It cannot see:

- Offline touchpoints (sales calls, events, in-person interactions)
- Cross-device journeys with gaps in user ID continuity
- Dark social traffic (links shared in Slack, WhatsApp, or private messages that arrive as direct)
- Channels where GA4 is blocked by ad blockers or iOS privacy restrictions

The actual traffic contribution of channels like newsletters, podcasts, and dark social is consistently underreported in GA4.

### Multi-Touch Attribution Tools

For accurate multi-channel attribution, especially at scale, consider adding a dedicated attribution tool:

Triple Whale: focused on e-commerce, integrates with Shopify, and provides pixel-based attribution with north star metrics.

Northbeam: multi-touch attribution with media mix modeling. Strong for direct-to-consumer brands.

Rockerbox: centralizes attribution data from all channels into a unified view. Good for blended media buyers.

HockeyStack: covers both paid and organic attribution, with good B2B support.

These tools require connecting your ad platform APIs and often your CRM. The setup investment pays back in clearer understanding of which channels drive real pipeline and revenue.

---

## Product Analytics: Mixpanel and Segment

GA4 tracks website behavior. For understanding user behavior inside your product (for SaaS and app companies), you need a product analytics tool.

### Segment

Segment is a Customer Data Platform (CDP) that sits between your product and your analytics tools. Instead of installing Mixpanel, Amplitude, Intercom, and other tools separately in your codebase, you install Segment once. Segment then routes events to any destination you configure.

The benefit is data consistency. Every downstream tool receives the same event data from a single source. If you rename an event or add a property, you change it in Segment and it propagates to all destinations.

Set up Segment by installing the analytics.js library (or the iOS/Android SDK for mobile). Then track events using Segment's track() method: analytics.track('Trial Started', { plan: 'pro', signup_source: 'pricing_page' });

Configure destinations in the Segment dashboard. Add Mixpanel, GA4, Intercom, HubSpot, Salesforce, or any other tool you use. Segment sends each event to all configured destinations.

### Mixpanel

Mixpanel is an event-based analytics tool with powerful funnel analysis, cohort analysis, and user-level reporting. It is particularly valuable for understanding how users move through your product and where they drop off.

Core Mixpanel reports:

Funnels: define a sequence of events and see what percentage of users complete each step. For example: Sign Up > Complete Profile > Invite Team Member > Create First Project. Funnels tell you exactly where users abandon the activation sequence.

Cohort analysis: group users by when they signed up and compare their behavior over time. A cohort of users who signed up in January 2025 vs. February 2025. This reveals how product changes affect user behavior for new users.

Retention: see what percentage of users return to perform a specific action after their first occurrence. A SaaS company might track retention as "percentage of users who perform a core action in week 2 after first performing it in week 1."

Flows: see the actual paths users take through your product, including unexpected paths. Useful for discovering how users are using your product in ways you did not design for.

---

## Diagnosing Broken or Missing Tracking

When tracking is broken, the symptoms usually appear in the data before anyone notices in the implementation. Traffic drops suddenly. Conversion events stop appearing. UTM attribution collapses to direct.

### Diagnosing GA4 Issues

Open GA4's DebugView and perform the action that should trigger the event. If the event does not appear in DebugView within 30 seconds, the tag is not firing.

Check GTM Preview mode. Is the tag visible? Is it firing? If the tag appears but shows "Fired 0 times," the trigger conditions are not being met.

Check the GTM trigger. If you are using a Click trigger, are the trigger conditions matching the element you are clicking? Use the Variables panel in GTM Preview to see the values of Click ID, Click Classes, and Click URL when you click the element.

Check for JavaScript errors on the page. Open your browser console (F12 > Console). JavaScript errors earlier on the page can prevent GTM and GA4 from loading correctly.

Check for ad blockers. Many ad blockers block GA4. If your testing is from a browser with an ad blocker, disable it before testing.

### Diagnosing UTM Issues

If utm_source is showing as "(not set)" or "(direct)" for traffic that should have UTMs, common causes include:

The UTM was on an intermediate redirect URL rather than the final landing URL. UTMs must be on the URL that the user's browser actually lands on.

The UTM contained a space or special character. URL-encode your UTM parameters. Spaces should be %20 or replaced with hyphens.

The landing page has a canonical redirect that strips query parameters.

The UTM was added correctly but the session was attributed to a different source because the user arrived from a different session within the attribution window.

Use GA4's Acquisition report to verify that UTM-tagged campaigns are appearing. If a campaign you know drove traffic shows as "(not set)," check the URL structure of your tagged links.

### Using Tag Assistant Legacy and Tag Assistant Companion

Google's Tag Assistant Chrome extension lets you verify that GA4, GTM, and other Google tags are loading and firing correctly. Install it, navigate to your page, and check the report for errors or warnings on each tag.

---

## Building a Measurement Dashboard

Your analytics infrastructure only has value if it produces insights that drive decisions. Build a weekly dashboard that surfaces the metrics that matter most.

For a marketing team, a useful weekly dashboard includes:

New visitors by channel: how much new traffic did each channel produce? Week over week comparison.

Conversion rate by channel: what percentage of visitors from each channel converted to your primary conversion event? This reveals channel quality, not just volume.

Cost per conversion by channel: only applicable for paid channels. Combine ad spend data from your ad platforms with conversion data from GA4.

Trial or lead to customer conversion rate: this requires connecting your CRM data. If trials from paid search convert to customers at 12% and trials from content convert at 22%, that is a significant insight that changes your channel allocation decisions.

Build this dashboard in Looker Studio (formerly Google Data Studio), which connects to GA4 and Google Ads for free. Add additional data sources for CRM data (HubSpot, Salesforce) or ad platform data (Meta, LinkedIn) using Supermetrics or a similar connector.

The goal is a dashboard that updates automatically and requires no manual data compilation. If building the dashboard requires someone to spend three hours in spreadsheets, it will not be maintained consistently.

---

## Getting Started: The Analytics Audit

If you are starting with an existing analytics setup, run an audit before making changes.

Step one: check what is installed. Use Tag Assistant to see every tag running on your site. List them all. Identify duplicates, outdated tags, and tags that conflict.

Step two: verify core conversions are tracking. Complete each of your primary conversion actions yourself and check GA4 DebugView to confirm the events fire correctly with the right properties.

Step three: review UTM consistency. Pull six months of traffic data. Look for inconsistent naming in utm_source and utm_campaign. List the normalization changes needed.

Step four: assess coverage gaps. What user behaviors matter for your business that you currently cannot measure? These become your implementation priorities.

Step five: document what exists. Create or update your tracking plan to reflect what is currently live.

From there, prioritize fixes by impact. Broken conversion tracking on your primary revenue event is the first thing to fix. Everything else follows.
