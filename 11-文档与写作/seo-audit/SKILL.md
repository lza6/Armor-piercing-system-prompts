---
name: seo-audit
description: >
  Use this skill for full SEO audits and SEO analysis tasks. Triggers on: SEO audit, technical SEO, crawlability issues, indexability, Core Web Vitals, mobile SEO, HTTPS, title tag optimization, meta description, heading structure, URL structure, internal linking analysis, image alt text, keyword targeting, content gaps, E-E-A-T, backlink profile, competitor gap analysis, and any request to diagnose why a site is not ranking or to produce a prioritized SEO action plan.
---

# SEO Audit

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

## Why SEO Audits Matter

An SEO audit is a systematic review of everything that affects how well a site performs in organic search. Most sites have fixable problems that suppress rankings. A structured audit finds those problems in a priority order that reflects actual impact.

The critical question to ask before starting any audit is this: is the problem that the site cannot be found at all (a technical problem), or that it is found but not chosen (a content and relevance problem)? The answer shapes where you spend your time.

A full SEO audit covers four categories: technical, on-page, content, and off-page. You cannot ignore one category and expect the others to compensate. A site with excellent content but serious crawl issues will underperform. A site that is technically perfect but has no backlinks and thin content will also underperform.

This skill produces a prioritized action list at the end of every audit, organized by impact and effort.

---

## Technical SEO Audit

Technical SEO covers everything that affects how search engines crawl, render, and index the site. If a page cannot be crawled, it will not be indexed. If it cannot be indexed, it will not rank. Technical issues are always the first thing to check.

**Crawlability**

Start by checking the robots.txt file. Navigate to yourdomain.com/robots.txt. Verify that the file is not accidentally blocking crawlers from important sections of the site. A single misconfigured line like "Disallow: /" blocks everything. This happens more often than it should, often from a developer who set it during staging and forgot to revert before launch.

Review your XML sitemap. Navigate to yourdomain.com/sitemap.xml. Verify that:
- The sitemap exists and is accessible.
- It includes all important pages.
- It does not include pages that are noindexed, redirected, or returning 4xx errors.
- Google Search Console shows the sitemap submitted and the number of pages indexed versus submitted. Large gaps between submitted and indexed pages indicate a problem.

Use Screaming Frog, Sitebulb, or a similar crawler to map the site. Look for crawl traps (infinitely paginating pages, session IDs in URLs, filter combinations that generate thousands of duplicate URLs), orphaned pages with no internal links, and excessive crawl depth (pages that require more than three clicks from the homepage to reach).

**Indexability**

In Google Search Console, navigate to the Index Coverage report. Review the breakdown of indexed pages, excluded pages, and error pages. Pages excluded by noindex, canonical tags, or crawl errors appear here.

For each excluded page, determine whether the exclusion is intentional. A staging version of a page excluded by noindex is correct. A key product page excluded because someone added a noindex tag during testing and never removed it is a serious problem.

Check canonical tags across the site. Every page with a canonical should point to the correct preferred version of that URL. Canonicals that point to themselves (self-canonicals) are fine. Canonicals that point to a redirected URL, a 404 page, or the wrong page create indexing confusion.

**Core Web Vitals**

Core Web Vitals are three performance metrics that Google uses as ranking signals for pages competing in search results. In practice, their direct ranking impact is modest for most sites, but poor Core Web Vitals correlate strongly with poor user experience, which hurts conversion rates regardless of ranking.

The three metrics are:

Largest Contentful Paint (LCP): How long it takes for the largest visible element to load. Target is under 2.5 seconds. Common problems include unoptimized hero images, render-blocking scripts, and slow server response time.

Interaction to Next Paint (INP): How quickly the page responds to user input. Target is under 200 milliseconds. Common problems include heavy JavaScript execution that blocks the main thread.

Cumulative Layout Shift (CLS): How much the page layout shifts during loading. Target is under 0.1. Common causes include images without defined dimensions, ads that load after content, and web fonts that swap late.

Find your Core Web Vitals data in Search Console under the Core Web Vitals report, or use PageSpeed Insights at pagespeed.web.dev for individual URL analysis. Google's data reflects real-user measurements from Chrome users, which is more reliable than lab measurements alone.

**Mobile Friendliness**

Google indexes from a mobile-first perspective. It crawls and evaluates your site as a mobile user. If your site looks fine on desktop but has usability issues on mobile (text too small to read, elements too close together to tap, content wider than the screen), those issues hurt your rankings.

Test mobile usability in Search Console under the Mobile Usability report. Run individual pages through the Mobile-Friendly Test at search.google.com/test/mobile-friendly.

**HTTPS**

Every page on your site should serve over HTTPS. If your site still has pages serving over HTTP, fix that immediately. Mixed content (HTTPS pages that load HTTP resources like images or scripts) is also a problem. Check for mixed content issues using the browser console on key pages.

Verify that your HTTPS redirect chain is clean. HTTP should redirect to HTTPS in one step. A redirect chain that goes HTTP to HTTPS to www.HTTPS to HTTPS is slower than necessary and adds unnecessary complexity.

**Redirect Health**

Map your redirect structure. Chains longer than two steps slow down crawling and dilute link equity. Loops break crawlers entirely. 302 (temporary) redirects where 301 (permanent) redirects should be used can suppress the ranking authority transfer.

---

## On-Page SEO Audit

On-page SEO covers the elements on each individual page that communicate relevance to search engines and users.

**Title Tags**

The title tag is the primary signal for what a page is about. It appears as the clickable headline in search results. Audit your title tags for:

- Length. Google displays approximately 50 to 60 characters. Titles longer than this get truncated. Run your titles through a SERP preview tool to see how they appear.
- Uniqueness. Every page should have a unique title tag. Duplicate titles create confusion about which page targets which keyword.
- Keyword placement. The primary keyword should appear early in the title, ideally in the first two to three words.
- Descriptiveness. Does the title accurately describe the page? Titles that attract clicks but disappoint users hurt your click-through rate signals.

Avoid template titles that are identical across every page except for a variable like the category name. "Buy Shoes Online - YourBrand" works once. "Buy Running Shoes Online - YourBrand," "Buy Hiking Boots Online - YourBrand" is better.

**Meta Descriptions**

Meta descriptions do not directly affect rankings, but they significantly affect click-through rate. A well-written meta description that matches search intent and includes a call to action will generate more clicks than a generic or auto-generated one.

Aim for 150 to 160 characters. Include the primary keyword naturally. Write for the searcher, not the crawler. Every page with strategic value should have a custom meta description.

**Heading Hierarchy**

Each page should have exactly one H1 tag that clearly states the topic of the page. The H1 does not need to match the title tag exactly, but it should be closely related. Subheadings (H2, H3) should create a logical outline of the page content.

Common heading problems to audit:

- Multiple H1 tags on one page.
- H1 tags that are too generic (like "Home" on the homepage) or too long.
- Heading tags used for styling purposes rather than structural organization.
- No headings at all on content-heavy pages.

**URL Structure**

Clean, descriptive URLs are better than long, parameter-filled URLs. Compare /blog/how-to-reduce-customer-churn to /blog?p=4821&cat=12. The first one communicates the topic clearly. The second communicates nothing.

Audit your URL structure for:

- Dynamic parameters that could be replaced with clean static URLs.
- Excessively long URLs (more than 60 to 70 characters).
- URLs that contain dates when the content is not time-sensitive (a guide on email marketing does not need /2019/ in the URL).
- Inconsistent capitalization or use of underscores instead of hyphens (hyphens are preferred for word separation).

**Image Alt Text**

Every meaningful image should have descriptive alt text. Alt text serves two purposes: it helps search engines understand the image content, and it provides accessibility for users who rely on screen readers.

Audit for:

- Images with no alt attribute.
- Images with generic alt text like "image1.jpg" or "photo."
- Images where the alt text is keyword-stuffed rather than descriptive.
- Decorative images that should have empty alt text (alt="") rather than descriptive text.

---

## Content SEO Audit

Content audits examine the depth, relevance, and uniqueness of the pages on the site. Technical health without strong content does not produce rankings.

**Keyword Targeting**

Review whether each important page targets a specific, researched keyword or keyword cluster. Pages that target no clear keyword or that target a keyword no one searches for will not generate organic traffic regardless of their technical quality.

For each key page, identify the primary keyword, the estimated monthly search volume, and the current ranking position. Use Google Search Console's Performance report to see which queries are generating impressions and clicks for each URL.

Map your keyword targeting across the site to check for cannibalization. Two pages targeting the same keyword will compete against each other. Identify competing pages and decide whether to consolidate them, differentiate them with distinct angles, or redirect one to the other.

**Content Depth**

Google's algorithms consistently reward content that comprehensively covers a topic. "Comprehensive" does not mean long for the sake of length. It means that a reader who finishes the page has a complete answer to the question they came with.

Audit your key pages against the competing pages that rank in positions one through five for the target keyword. What do those pages cover that yours does not? What subtopics, questions, or data points appear consistently across top-ranking pages?

Use tools like Semrush's SEO Content Template or Clearscope to identify topically related terms that appear in high-ranking competitor content. Content that covers topically related terms tends to rank better than content that repeats the primary keyword more often.

**Thin Content**

Pages with fewer than 300 words and no clear purpose suppress the site's overall quality signal. Audit for thin pages and decide whether to:

- Expand the content to a useful depth.
- Consolidate the page with a related, more complete page.
- Noindex the page if it serves a functional purpose but should not appear in search results.

Product pages with nothing but a product name, one sentence, and a price are thin. So are location pages that have the same content template repeated across 50 locations with only the city name swapped.

**E-E-A-T Signals**

E-E-A-T stands for Experience, Expertise, Authoritativeness, and Trustworthiness. Google uses these concepts to evaluate the quality of content, particularly for queries related to health, finance, legal advice, and significant purchasing decisions (the "Your Money or Your Life" categories).

Audit your content for E-E-A-T signals:

- Author bylines on editorial content. Who wrote this? Do they have credentials in the topic area?
- Author pages with biography information.
- Company information: About page, team page, physical address, contact information.
- Citations and external sources linked from the content.
- Social proof: awards, press mentions, certifications.
- Accurate and up-to-date information. Outdated statistics or stale guidance undermine trust.

**Duplicate and Near-Duplicate Content**

Duplicate content across multiple URLs dilutes ranking signals and confuses Google about which URL to rank. Common causes:

- Print-friendly versions of pages accessible at separate URLs.
- www and non-www versions both accessible without a canonical or redirect.
- HTTP and HTTPS versions both accessible.
- Trailing slash and non-trailing slash versions both accessible (/page/ and /page both resolve).
- URL parameters that generate multiple versions of the same content (/products?color=red and /products?color=blue both showing the same product with slightly different content).

Crawl the site and identify pages with a high similarity ratio. Use canonical tags or 301 redirects to consolidate duplicate and near-duplicate pages.

---

## Off-Page SEO Audit

Off-page SEO covers signals that come from outside your site, primarily backlinks and brand mentions.

**Backlink Profile Overview**

Pull your backlink profile using Ahrefs, Semrush, or Moz. Review:

- Total referring domains. One site linking to you 100 times counts as one referring domain, not 100 backlinks. Referring domains is the more meaningful metric.
- Domain Rating or Domain Authority of your top referring domains. Links from authoritative sites pass more equity than links from low-authority sites.
- Anchor text distribution. A natural profile has mostly branded anchors (your company name), URL anchors, and generic terms, with some keyword-rich anchors. If your profile is dominated by exact-match keyword anchors, it looks manipulated.
- Link velocity trends. Sudden spikes in link acquisition that are followed by a drop-off look unnatural.

**Toxic Link Review**

Not all backlinks are beneficial. Spammy links from link farms, comment spam, or purchased link schemes can hurt your site. Use Ahrefs or Semrush's toxicity filters to identify links with patterns of low quality.

For genuinely toxic links, use Google's Disavow Tool to tell Google to ignore them. The disavow file is a measure of last resort. Most naturally acquired links, even from low-quality sites, are simply ignored by Google rather than actively hurting your rankings.

**Competitor Gap Analysis**

Identify three to five competitors that rank for your target keywords. Run a competitor link intersection analysis to find domains that link to multiple competitors but not to you. These are your best link acquisition opportunities because:

- The linking site is clearly willing to link to sites in your category.
- They already know the topic area.
- You have a clear pitch: they link to your competitors, and you offer something worth linking to.

Also audit competitors' top-performing content. Which pages have they built that attract links naturally? Understanding what attracts links in your category tells you what to build.

---

## Prioritized Action List

Every audit ends with a prioritized action list. Prioritize by the combination of SEO impact and implementation effort.

**Priority 1 (Fix immediately):**
- Robots.txt accidentally blocking crawlers from key pages.
- Important pages noindexed unintentionally.
- Site serving over HTTP without redirect to HTTPS.
- Critical pages returning 404 or 500 errors.
- Redirect loops or chains longer than two steps.

**Priority 2 (Fix in the next sprint):**
- Missing or duplicate title tags on key pages.
- No XML sitemap or sitemap not submitted to Search Console.
- Core Web Vitals failing on the most valuable landing pages.
- Keyword cannibalization on high-value target keywords.
- Missing canonical tags on paginated or parameterized content.

**Priority 3 (Fix in the next 30 to 60 days):**
- Meta descriptions missing on important pages.
- Thin content pages that could be expanded or consolidated.
- Images without alt text across high-traffic pages.
- Internal linking gaps between related content.
- E-E-A-T signals missing from key editorial pages.

**Priority 4 (Ongoing):**
- Content expansion for pages ranking in positions 4 through 15 for target keywords.
- Backlink acquisition targeting competitor link gaps.
- Schema markup implementation by page type.
- Page speed improvements beyond the most critical Core Web Vitals.

The order matters. A site that fixes title tags but still has crawl issues has not made meaningful progress. Technical foundation first, then on-page optimization, then content depth, then off-page authority building.

---

## Tools for Running a Full Audit

A complete audit uses at least three or four tools, because no single tool covers every dimension.

**Google Search Console:** Free. Required for indexation data, Core Web Vitals field data, search performance data by page and query, mobile usability issues, and schema enhancement reports. If you are not connected to Search Console, connect it before anything else.

**Google PageSpeed Insights:** Free. Analyzes Core Web Vitals for individual URLs using both lab and field data. Provides specific recommendations for each metric.

**Screaming Frog SEO Spider:** Free up to 500 URLs, paid for larger sites. Crawls the site and surfaces technical issues including broken links, redirect chains, duplicate content, missing title tags, and missing alt text.

**Ahrefs or Semrush:** Paid, and worth the investment for serious SEO work. Provides backlink data, keyword ranking data, competitor analysis, and site audit functionality that goes beyond what Screaming Frog covers.

**Google Analytics:** Free. Provides behavioral data by page (bounce rate, session duration, goal conversions) that helps prioritize which pages are most valuable to optimize.

An audit without data is an opinion. Run the tools. Export the data. Work from evidence.
