---
name: schema
description: >
  Use this skill for structured data and schema markup questions. Triggers on: JSON-LD, schema.org, structured data, rich snippets, rich results, FAQ schema, Product schema, Article schema, Organization schema, Review schema, LocalBusiness schema, HowTo schema, BreadcrumbList schema, Event schema, schema validation, Google Rich Results Test, and any question about making content eligible for enhanced search features or improving visibility in AI-powered search.
---

# Schema Markup and Structured Data

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

## What Schema Markup Does

Schema markup is structured data that you add to a webpage to help search engines understand exactly what the page contains. Without schema, Google infers meaning from your HTML, text, and links. With schema, you tell Google directly: this page is a product listing, this is a review, this is a recipe, this is a business address.

That distinction matters because it makes your content eligible for rich results, the enhanced SERP features that show star ratings, prices, FAQs, event dates, and other information directly in search results before a user clicks. Rich results get more clicks than plain blue links. The average click-through rate for rich results is roughly 20 to 30 percent higher than for standard organic listings, depending on the SERP feature and query type.

Schema also matters for AI search. When AI assistants like Google's AI Overviews, Perplexity, and ChatGPT with web access pull structured information about your business or products, they rely on machine-readable data. Clear schema makes your content easier to parse correctly, which means it is more likely to be cited accurately.

The question worth considering is this: if your product page has no structured data and a competitor's product page has complete Product schema with reviews and pricing, which page gives the search engine more reason to feature it prominently?

---

## JSON-LD vs. Microdata vs. RDFa

You have three ways to add structured data to a webpage. JSON-LD is the format you should use.

**JSON-LD** (JavaScript Object Notation for Linked Data) places the structured data in a script tag in the page's head or body. It does not require you to modify your HTML structure at all. You can add it, update it, and remove it without touching the visible content on the page. Google explicitly recommends JSON-LD.

**Microdata** embeds structured data attributes directly into your HTML elements. It requires restructuring your HTML. Changing the schema means changing the HTML. It is harder to audit and harder to maintain.

**RDFa** works similarly to microdata. It was popular before JSON-LD became the standard. There is rarely a reason to use it for new implementations.

Use JSON-LD. Place the script tag in the head element when possible. Add it to the body before the closing tag when CMS limitations require it. Google crawls and processes both locations.

---

## The Schema Types That Matter Most

Schema.org lists hundreds of types. Most of them are irrelevant for marketing purposes. Focus on the types that Google actively uses to generate rich results.

**Organization**

Add Organization schema to your homepage and, optionally, your About page. This schema tells Google your company name, logo, URL, social profiles, and contact information. It strengthens the Knowledge Panel that appears when users search for your brand directly.

A basic Organization markup:

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Acme Software",
  "url": "https://www.acmesoftware.com",
  "logo": "https://www.acmesoftware.com/images/logo.png",
  "sameAs": [
    "https://twitter.com/acmesoftware",
    "https://linkedin.com/company/acmesoftware"
  ],
  "contactPoint": {
    "@type": "ContactPoint",
    "telephone": "+1-800-555-0100",
    "contactType": "customer support"
  }
}
```

**Product**

Add Product schema to every product or service page where pricing and availability are relevant. This schema enables price, availability, and review data to appear directly in search results. For e-commerce sites, complete Product schema is not optional. It is a baseline requirement for competing in product search.

Required properties for Product schema: name, image, description. Recommended properties that unlock rich results: offers (with price, currency, and availability), aggregateRating, and brand.

A common mistake is adding Product schema without an Offers property. Without Offers, you get no price display in search results, which eliminates the primary benefit of Product schema.

**Article**

Add Article schema (or its subtypes BlogPosting and NewsArticle) to every blog post and editorial piece. This helps Google understand publication dates, authorship, and content type, which are relevant signals for freshness-sensitive queries.

Key properties: headline, image, datePublished, dateModified, author, and publisher. The dateModified property is particularly important. If you update a post, update this field. Google uses it to decide whether to re-crawl and re-index.

**FAQ**

Add FAQ schema to any page that contains a question-and-answer format. This schema was historically used to generate expandable FAQ features directly in Google's search results. Google has reduced the frequency of FAQ rich results in standard searches but still uses the markup to better understand page content, and some categories of queries still surface FAQ features.

Structure each question-answer pair correctly:

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "How long does onboarding take?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Most customers complete onboarding within 5 business days. The timeline depends on the complexity of your data migration and the number of users."
      }
    }
  ]
}
```

Do not add FAQ schema to a page where the questions are not visibly present in the page content. Google explicitly prohibits schema that does not match visible content.

**HowTo**

Add HowTo schema to pages that walk through a process step by step. This schema makes the steps eligible to display directly in search results, with numbered steps and optional images. It works well for tutorial content and instructional guides.

Each step needs a name and text property at minimum. Adding an image to each step improves the chances of the rich result appearing.

**Review and AggregateRating**

Reviews are among the most powerful signals in search results. A star rating displayed directly on a search result increases click-through rate significantly. Add AggregateRating to Product, LocalBusiness, Software, and other appropriate entity types.

Do not add review schema to reviews you wrote yourself about your own products. Google's guidelines prohibit first-party reviews presented as independent assessments. The aggregate rating must reflect genuine customer reviews from a legitimate review system.

**LocalBusiness**

Add LocalBusiness schema (or a more specific subtype like Restaurant, Hotel, or MedicalBusiness) to the homepage and location pages of any business with a physical location. This schema feeds directly into Google Maps and local search results.

Required properties: name, address (using PostalAddress), telephone, and url. Add openingHoursSpecification if your business has regular hours. Add geo coordinates if you want to strengthen map placement.

**BreadcrumbList**

Add BreadcrumbList schema to every page that has a breadcrumb navigation trail. This enables the breadcrumb path to display in search results instead of the raw URL. Breadcrumbs in search results improve clarity for the user and reduce wasted clicks from misleading URLs.

Ensure the schema breadcrumb path exactly matches the visible breadcrumb on the page. Mismatches are a validation error.

**Event**

Add Event schema to any page promoting a specific event with a date, location, and registration option. This schema makes events eligible to appear in Google's event discovery features and in search results for event-related queries.

Required properties: name, startDate, and location. Add endDate, description, organizer, offers (for ticket pricing), and eventAttendanceMode (to indicate whether the event is in-person, online, or hybrid).

---

## Implementing JSON-LD Correctly

The most common implementation approach for sites without a CMS that handles schema automatically is to add a script tag in the page's head section.

```html
<head>
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Product",
    "name": "Project Management Software",
    ...
  }
  </script>
</head>
```

For CMS-managed sites, the approach varies:

**WordPress:** Plugins like Yoast SEO, Rank Math, and Schema Pro handle most common schema types automatically. They generate schema from existing post metadata. Verify what they produce using the Rich Results Test.

**Shopify:** Shopify's default theme generates basic Product schema. Many stores need a custom app or manual theme edits to add complete schema with AggregateRating.

**Webflow:** Webflow does not generate schema natively. You add it through custom code blocks in the page's head section or through a custom attribute approach using Webflow's CMS.

**Next.js and React:** Use the next-seo package or a similar library that lets you define JSON-LD as a component with dynamic data from your content API.

For pages where the schema data should be dynamic (product prices that change, events with live availability, articles with updated dates), pull the data from your CMS or database into the schema at render time. Static schema that does not reflect current page content is a validation error and a user experience problem.

---

## Common Schema Mistakes

These errors appear repeatedly in schema audits.

**Marking up content that is not visible on the page.** If a property value is not visible somewhere on the page, do not include it in schema. Google requires that structured data reflect actual page content.

**Missing required properties.** Each schema type has required properties for rich result eligibility. Product schema without Offers cannot display pricing. Review schema without a numeric ratingValue cannot display stars. Check the Google Search Central documentation for required versus recommended properties for each type.

**Using the wrong type for the content.** A blog post marked up as an Article is fine. A blog post marked up as a Product is not. An organization's contact page marked up as a LocalBusiness when the company has no physical storefront will likely fail validation.

**Duplicate schema on the same page.** Some CMS plugins and themes generate schema automatically. If you also add schema manually, you may end up with two conflicting blocks for the same type. The Rich Results Test will flag this. Audit your pages for duplicate schema.

**Outdated schema that no longer matches the page.** A product page that changed its price but still has the old price in schema. An event page for an event that has passed. An article with a dateModified from two years ago despite being updated last month. Schema maintenance is part of the ongoing job.

**Self-referencing reviews.** Adding review content that was written by your own team or that you fabricated. Google can detect patterns that suggest first-party review manipulation and will either ignore the schema or penalize the site.

**Broken JSON syntax.** A missing comma, an unclosed bracket, or an extra quotation mark will cause the entire schema block to fail. Always validate after implementation.

---

## Validating Schema

Use Google's Rich Results Test as your primary validation tool. You can access it at search.google.com/test/rich-results. Enter a URL or paste the raw HTML and the tool will show you:

- Which rich result types the page is eligible for.
- Which properties are detected.
- Which required properties are missing.
- Errors and warnings.

A warning does not prevent rich results. An error typically does.

Use the Schema Markup Validator at validator.schema.org for general schema validation that is not specific to Google's rich results eligibility. This catches JSON syntax errors and invalid property values.

In Google Search Console, navigate to the Enhancements section. Each schema type you have implemented will appear as a separate report if Google has detected it across multiple pages. These reports show errors and warnings at scale, which is more efficient than testing pages individually.

After fixing a schema error, use the URL Inspection tool in Search Console to request re-indexing. Rich result eligibility may not appear in search results immediately after fixing errors. Allow 48 to 72 hours.

---

## Schema and AI Search Visibility

AI-powered search features including Google's AI Overviews, Perplexity, and ChatGPT's web browsing capability all rely on their ability to understand and parse your content accurately.

Clear, complete schema helps in two specific ways.

First, it reduces ambiguity. When a search engine or AI model is deciding whether your business is a SaaS product, a consulting firm, or a physical services business, your Organization or LocalBusiness schema gives it a direct answer. Ambiguous content gets interpreted incorrectly more often.

Second, it signals structured information that can be cited with confidence. A Product schema with accurate pricing, availability, and reviews is easier for an AI system to cite than a page where pricing is buried in a comparison table and reviews are scattered across customer testimonials written in different formats.

Implementing schema is not a guarantee of AI citation or rich result appearance. It is a prerequisite. Content that lacks structured data competes at a disadvantage against content that provides it.

---

## Schema for Multi-Location and E-Commerce Sites

Multi-location businesses and e-commerce stores face additional complexity.

**Multi-location:** Add a separate LocalBusiness schema for each location. Each location page should have a unique schema block with that location's specific address, phone number, hours, and coordinates. Do not use the same schema for multiple locations. Shared schema produces incorrect data in Maps and local search.

If your site has a corporate parent and multiple subsidiary locations, consider using the Organization type at the corporate level and LocalBusiness at each location level, with the parentOrganization property linking them.

**E-commerce product pages:** Add Product schema to every product page. Scale this by templating the schema in your product page template and pulling data dynamically from your product database. Manually adding schema to thousands of product pages is not practical.

For sites with product variants (different colors, sizes, or configurations at different prices), apply the schema to the parent product and use the hasVariant property to reference variants, or apply separate schema to each variant if each has its own URL.

**Category pages:** Do not add Product schema to category or collection pages. These pages list multiple products and are not themselves products. You can add BreadcrumbList schema and potentially a custom ItemList schema, but Product schema belongs only on the individual product page.

---

## Prioritizing Schema by Page Type

Start with the schema types that produce the highest impact for your site type:

For a SaaS company:
- Organization on the homepage.
- SoftwareApplication on product pages.
- FAQPage on pricing and support pages.
- Article or BlogPosting on every blog post.

For an e-commerce store:
- Organization on the homepage.
- Product with Offers and AggregateRating on every product page.
- BreadcrumbList on category and product pages.

For a local services business:
- LocalBusiness on the homepage and location pages.
- FAQPage on service pages.
- Review schema where reviews are published.

For a content publisher or blog:
- Organization on the homepage.
- Article or BlogPosting on every article.
- FAQPage where applicable.
- BreadcrumbList across the site.

Implement the highest-priority types first. Validate them. Monitor Search Console for eligibility and errors. Then add secondary types in subsequent work cycles.

Schema is an ongoing practice, not a one-time project. Every new page type requires a schema decision. Every content update may require a schema update. Build that into your content production workflow.
