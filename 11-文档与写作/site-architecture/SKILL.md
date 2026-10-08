---
name: site-architecture
description: >
  Use this skill for planning, mapping, and structuring website page hierarchy, navigation, and internal linking. Triggers on: sitemap planning, page hierarchy, navigation structure, primary navigation, secondary navigation, footer navigation, URL structure, breadcrumbs, internal linking strategy, pillar pages, cluster pages, topic clusters, content architecture, and any question about how to organize a website so it works well for both users and search engines.
---

# Site Architecture

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

## Why Site Architecture Matters

Site architecture is the organizational structure of your website. It determines how pages relate to each other, how users move between them, and how search engines understand the scope and hierarchy of your content.

Most companies build their site architecture organically. A developer adds pages when someone requests them. Marketing creates a blog section when content starts. A new product launches with a new section. Over time, the site grows into something that reflects the history of decisions made by different teams, rather than a coherent structure designed for users and search engines.

The cost of poor architecture is real. Users who cannot find what they need leave. Google crawlers that encounter confusing link structures assign lower authority to important pages. Pages buried four or five clicks from the homepage get crawled less often and rank less well than pages at shallower depths.

The question to sit with before redesigning any site architecture is this: if a visitor landed on your homepage with no prior knowledge of your company, could they reach the most important page on your site in two clicks?

If the answer is no, your architecture has a depth problem.

---

## Sitemap Planning

A sitemap is the master list of pages on your site and the hierarchical relationships between them. Not the XML sitemap submitted to Google (that is a different artifact). This is a structural document that defines what pages exist, how they are organized, and how they connect.

Start every site architecture project by building the sitemap before designing any page or writing any content.

The sitemap planning process has four steps:

**Step 1: Inventory what exists.** Crawl the current site with Screaming Frog or a similar tool. Export every URL with its current status code, title, and inbound internal link count. This gives you the complete picture of what you are working with.

**Step 2: Map the user journeys.** Identify the three to five most important paths a user should take through your site. For a SaaS product targeting operations managers, those paths might be: Homepage to Product to Pricing to Free Trial, or Homepage to Blog to Case Study to Demo Request. Each path is a series of pages, and each page needs to exist and to link to the next step naturally.

**Step 3: Map the keyword landscape.** For each major topic you want to rank for, identify the page that should own that keyword. Every topic needs exactly one primary page. If two pages target the same keyword, you need to consolidate or differentiate.

**Step 4: Define the hierarchy.** Organize pages into a tier structure. Tier one is the homepage. Tier two is the main category or section pages (Products, Solutions, Resources). Tier three is the individual pages within those sections (individual product pages, solution landing pages, blog posts). Tier four and below are rarely appropriate for pages you want to rank.

Produce the sitemap as a visual diagram. A spreadsheet works for auditing, but a diagram shows the relationships between pages in a way that is easier to reason about and communicate.

---

## Page Hierarchy Design

Page hierarchy determines the relationship between parent pages and child pages. A clear hierarchy helps users understand where they are in the site and where they can go. It helps search engines understand which pages are more important based on their position in the structure.

The homepage sits at the top. Every other page exists at some distance from the homepage, measured by the number of clicks required to reach it.

**Tier one (homepage).** One page. It links to every top-level section of the site. Every page on the site should be able to trace a link path back to the homepage.

**Tier two (section pages).** These are the main category pages. For a software company, typical tier two pages include Products, Solutions (by industry or use case), Pricing, Blog or Resources, Company, and About. The number of tier two pages should not exceed eight to ten. More than that indicates the top-level organization needs to be reconsidered.

**Tier three (subcategory or pillar pages).** These are the primary content or product pages within each section. Individual product pages, solution pages for each industry, feature pages, and resource category hubs all live here. Pages at this level are linked from their parent tier two page and from the main navigation.

**Tier four (leaf pages).** These are individual posts, articles, guides, or detail pages within a category. They should be linked from their parent tier three page and from related tier four pages through internal links.

Pages deeper than four levels (requiring five or more clicks from the homepage) should be rare. If you have important content buried that deep, move it up or create a shortcut path through internal linking.

---

## Navigation Structure

Navigation is the user-facing expression of your site hierarchy. The structure you design in the sitemap becomes the links users see and click. Navigation design has three components: primary navigation, secondary navigation, and footer navigation.

**Primary Navigation**

The primary navigation appears in the site header and is visible on every page. It is the most prominent navigation element and should reflect your most important user paths.

Include only pages that serve the majority of your visitors. Typical primary navigation for a B2B software company includes:

- Product (or Products, if you have multiple).
- Solutions (organized by industry or use case, accessible via a dropdown).
- Pricing.
- Resources (or Blog, depending on how content-heavy the site is).
- Company or About.
- A prominent call to action (Get a Demo, Start Free Trial, Sign Up).

Keep primary navigation items to five or six. More than six items creates visual noise and decision fatigue. If you have more than six genuine priorities, you need to consolidate, not expand the navigation.

Dropdown menus are appropriate when a section has three or more subsections that users might want to navigate to directly. Use mega menus (larger dropdowns with multiple columns) for sites with many products or solutions. Keep dropdowns shallow. A dropdown within a dropdown (a third level of navigation) is almost always a sign that the information architecture needs to be rethought.

**Secondary Navigation**

Secondary navigation appears within sections of the site. On a product page, secondary navigation might include tabs for Features, Integrations, and Pricing. On a resource hub, it might include filtering by content type. Secondary navigation helps users orient themselves within a section and move between related pages without returning to the global navigation.

Design secondary navigation based on user behavior data. Check your analytics to see which pages users visit in sequence within each section. Those paths should be supported by secondary navigation links.

**Footer Navigation**

Footer navigation serves two audiences: users who scrolled past the main content without finding what they needed, and crawlers that use footer links to discover additional pages.

Organize footer navigation into columns by section. Include links to all major sections of the site plus the pages that do not fit naturally in the primary navigation (terms of service, privacy policy, contact, sitemap, careers). Do not duplicate the primary navigation exactly. Use the footer to surface secondary pages that deserve visibility but do not belong in the main header.

---

## URL Structure Decisions

URL structure is a decision made once and very difficult to change later, because URL changes require redirects that add complexity and can disrupt rankings during transition. Make URL decisions deliberately before launching a site or major section.

Principles for URL structure:

**Short and descriptive.** A URL should communicate the page topic clearly. /blog/email-marketing-guide is better than /blog/post?id=4821. The former is human-readable. The latter is not.

**Hyphens, not underscores.** Google treats hyphens as word separators. Underscores are treated as connecting characters, which means email_marketing is read as one word, not two. Use hyphens.

**Lowercase only.** Capital letters in URLs create duplicate content risks. /Blog and /blog are technically different URLs that could both be served, creating a duplicate content issue. Use lowercase consistently.

**Remove unnecessary dates from evergreen content.** A URL like /blog/2019/02/email-marketing-guide will look stale by the time you update the guide in 2025. /blog/email-marketing-guide is cleaner and does not signal staleness. Reserve dates in URLs for genuinely date-specific content like news or press releases.

**Mirror the hierarchy.** URL structure should reflect your site hierarchy. /solutions/manufacturing/ makes clear that manufacturing is a subsection of solutions. /manufacturing-solutions/ loses that hierarchical context.

**Avoid stop words.** Articles (a, an, the) and prepositions (for, in, of) in URLs add length without adding meaning. /guide-to-email-marketing can become /email-marketing-guide without any loss of clarity.

**Subdirectory vs. subdomain for major content sections.** Blog content hosted at domain.com/blog passes link equity to the root domain and benefits from the domain's overall authority. Blog content hosted at blog.domain.com is treated as a separate site by Google, which means it builds authority independently. For most companies, keeping content in subdirectories is better for SEO.

---

## Breadcrumb Strategy

Breadcrumbs are the navigational trail that shows a user where they are in the site hierarchy. They appear near the top of the page, typically below the primary navigation, and look like: Home > Blog > Email Marketing > This Post Title.

Breadcrumbs serve three purposes. They help users orient themselves on a page they may have arrived at directly from search (with no context about the site structure). They provide one-click navigation back to parent pages. They give search engines a clear hierarchical signal about the page's position in the site.

Implement breadcrumbs on:
- All blog posts and articles.
- All product or feature pages.
- All location pages.
- Any page more than two levels deep from the homepage.

Do not implement breadcrumbs on the homepage or on flat section landing pages (like your main Pricing page) where there is no meaningful hierarchy to display.

Pair breadcrumb implementation with BreadcrumbList schema markup. This makes the breadcrumb path eligible to appear directly in search results, showing the page's position in the site hierarchy as part of the search result. This improves click-through rate by giving users more context before they click.

Ensure that each breadcrumb link points to a real, accessible page. A breadcrumb that links to a 404 or a redirect is a usability failure.

---

## Internal Linking Architecture

Internal linking is the practice of linking from one page on your site to another. It is one of the highest-impact SEO activities you can do, and one of the most consistently neglected.

Internal links do two things. First, they transfer authority from linked pages to destination pages. A page with many authoritative external backlinks passes some of that authority to the pages it links to internally. Pages with more internal links pointing to them rank better than pages with fewer. Second, internal links guide users and crawlers to related content, reducing bounce rates and improving crawl coverage.

**Hub and spoke internal linking.** On a pillar page (a comprehensive guide on a broad topic), link out to all related cluster pages (detailed posts on subtopics). On each cluster page, link back to the pillar page. This structure signals to Google that the pillar page is the authoritative hub for that topic, and that the cluster pages provide depth on specific subtopics.

For example, a pillar page on "Email Marketing" would link to cluster pages on "Email Deliverability," "Subject Line Best Practices," "Email Segmentation," and "Email Automation." Each cluster page links back to the pillar page and cross-links to other relevant cluster pages.

**Link from high-authority pages.** Identify your pages with the most external backlinks (your homepage, your most-cited blog posts, your most-shared resources). These are your highest-authority pages. Link from them to the pages you most want to rank. The link transfer effect is real and measurable.

**Use descriptive anchor text.** Anchor text is the clickable text of a link. "Click here" tells Google nothing about the destination page. "Email marketing guide" tells Google exactly what the destination page covers. Use keyword-rich, descriptive anchor text for internal links.

**Link to pages users would naturally want next.** The best internal links are ones that genuinely serve the reader's next question. A blog post about email deliverability naturally links to a post about email authentication setup. A feature page about automation naturally links to a case study about a customer who used automation to save time. Forced or unrelated internal links provide less value than natural ones.

**Audit for orphaned pages.** An orphaned page has no internal links pointing to it. Google can only discover it from the XML sitemap, which means it may be crawled infrequently and will accumulate no link equity. Run a crawl and identify pages with zero inbound internal links. Either link to them from relevant pages or decide whether they should exist.

---

## Pillar and Cluster Page Models

The pillar and cluster model is a content architecture approach that groups related content around a central hub. It produces two benefits: it gives users a clear path through a complex topic, and it signals to search engines that your site has deep, organized expertise on that topic.

**The pillar page** is a comprehensive page covering a broad topic at a high level. It defines the topic, covers all major subtopics briefly, and links to deeper resources on each. A pillar page on "CRM Software" might be 3,000 to 5,000 words, covering what a CRM is, how to choose one, the main features to look for, and how to implement one. It links to detailed cluster pages on each of those subtopics.

The pillar page targets a high-volume, broad keyword. It is not meant to answer every possible question in full depth. It is meant to be the authoritative starting point and the hub that organizes the rest of the content cluster.

**Cluster pages** are the detailed posts within a topic cluster. Each cluster page covers one specific subtopic in depth. A cluster page on "How to Choose a CRM" covers that specific question in 1,500 to 3,000 words, with practical guidance, comparison frameworks, and examples. It links back to the CRM pillar page and to related cluster pages.

Cluster pages target longer, more specific keywords (often called long-tail keywords). They individually generate modest traffic but collectively generate significant volume. More importantly, the interlinking structure distributes authority across the cluster and reinforces the pillar page's position for the broad keyword.

Build topic clusters intentionally. For each broad topic you want to own in search, create the pillar page first, then identify the 10 to 20 subtopics that belong in the cluster, and create those cluster pages systematically. Measure the cluster's performance as a unit, not just individual page rankings.

---

## Site Architecture and User Experience

Site architecture is often discussed purely as an SEO topic, but it affects user experience just as directly. Users who cannot find what they are looking for leave. Users who feel lost within a site lose confidence in the product or company.

Apply these user experience principles to architecture decisions:

**Match navigation labels to user vocabulary.** Users look for the language they already use, not your internal terminology. If your customers call it "analytics" and your navigation says "Insights Hub," they will look for analytics and not find it. User research, customer support logs, and site search data reveal how your users describe what they are looking for.

**Prioritize the primary conversion paths.** Every page should have a clear path to the next step that matters for your business. A blog post should link to a relevant case study or product page. A product page should link to pricing and to a trial or demo. An About page should link to Contact or Careers. The paths do not need to be aggressive. They need to be clear.

**Design for scanners.** Most users scan before they read. Navigation labels, section headers, and descriptive URLs all support scanning behavior. A site where you cannot tell what a page is about from its title and URL is a site designed for the company's internal logic, not for the user.

**Test with real users.** Tree testing is a research method where users are given a list of tasks and asked to navigate a text-only version of the site hierarchy. It reveals whether users can predict where to find information based on your navigation structure. Run tree tests before launching a new architecture, not after.

---

## When to Rebuild vs. When to Reorganize

Not every site with architecture problems needs a full rebuild. Full rebuilds are expensive, time-consuming, and introduce significant SEO risk if not managed carefully. Often, targeted reorganization and improvement produces better results with less disruption.

Rebuild the architecture when:

- The site hierarchy is more than four levels deep for important content.
- The URL structure is so inconsistent or poorly designed that fixing it requires site-wide redirects anyway.
- The navigation has grown to include more than 10 top-level items with no logical grouping.
- Analytics show that users consistently cannot find their way to the most important pages.

Reorganize without rebuilding when:

- A specific section is poorly structured but the rest of the site works.
- Navigation labels are confusing but the underlying structure is sound.
- Internal linking is sparse but existing pages are well organized.
- A new content cluster needs to be built out within an otherwise functional site.

Either way, plan the work before executing it. Architecture changes made without a plan create new inconsistencies to fix later. Document the intended structure, the URL mapping, the redirect plan, and the internal link updates before a single page moves.
