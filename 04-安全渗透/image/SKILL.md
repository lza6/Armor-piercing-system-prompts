---
name: image
description: >
  Use this skill when creating, generating, or optimizing marketing images. Trigger phrases include: "generate an image for this post," "create a hero image," "write a prompt for Midjourney," "make an OG image," "create product mockups," "design a social media graphic," "optimize our images for the web," "create a profile banner," "write a Flux prompt," "ensure brand consistency in visuals."
---

# Marketing Image Creation and Optimization Skill

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

## What This Skill Does

This skill covers marketing image creation from end to end. It addresses AI image generation tools and prompt writing, product mockups, hero images, social media graphics, OG images, profile banners, image optimization for the web, and brand consistency across all visual output.

Marketing images are often underinvested compared to copy. But a mismatched or generic image in a LinkedIn post can cut engagement by 40%. A slow-loading image on a landing page increases bounce rate. A blurry hero image signals amateurism before a visitor reads a single word.

The question that separates adequate from excellent visual marketing: does every image look like it belongs to the same product, or does it look like it was found in different places by different people?

---

## Part 1: AI Image Generation Tools and When to Use Each

The AI image generation landscape has matured. Different tools suit different use cases. Using the right tool for the right job saves significant time and produces better results.

### Flux (Black Forest Labs)

Flux produces the most photorealistic AI images currently available for consumer use. Its strengths are realistic human faces, product photography-style images, and scenes with accurate lighting.

Use Flux when you need:
- Realistic people in real environments (office settings, team photos, user scenarios).
- Product-adjacent imagery that looks like photography.
- High-detail scenes where photorealism matters.

Flux has two widely used versions: Flux Pro (slower, highest quality) and Flux Dev (faster, slightly lower quality). Both are accessible through Replicate, fal.ai, or direct API.

### Midjourney

Midjourney produces images with a distinctive artistic quality. It excels at stylized illustration, abstract concepts, and compositions with strong visual impact. It is less strong at photorealism and accurate text rendering.

Use Midjourney when you need:
- Hero images with an artistic or concept-driven feel.
- Illustrations for blog content.
- Abstract visualizations of ideas.
- Social images where visual distinctiveness matters more than realism.

Midjourney runs inside Discord, which is a friction point. The latest versions (V6 and beyond) have improved photorealism significantly but still trail Flux for human portraits.

### DALL-E (OpenAI, including GPT Image)

DALL-E is integrated into ChatGPT and the OpenAI API. It is the most accessible AI image tool for users already in the OpenAI ecosystem.

Use DALL-E / GPT Image when:
- You are working inside ChatGPT and want speed over quality.
- You need quick concept images for internal presentations.
- You need to iterate rapidly on ideas without switching tools.
- You want to combine text analysis with image generation in one workflow.

GPT Image (the latest iteration) has improved significantly in prompt adherence and handles text within images better than most other tools. Generating social graphics with text overlaid is now a reasonable use case.

### Ideogram

Ideogram is specifically strong at one thing: accurate text rendering within images. This makes it the best tool for:
- Social graphics that include readable text.
- Quote cards.
- Promotional images with a headline.
- Any image where the text must be legible and correctly spelled.

Most other AI image generators struggle with text. Ideogram treats text as a first-class element. If your image includes words, use Ideogram first.

### Choosing Between Tools

A practical decision tree:

- Image includes important text (a headline, a stat, a quote): Use Ideogram.
- Image needs to look photographic with real-seeming people: Use Flux Pro.
- Image is illustrative, artistic, or conceptual: Use Midjourney.
- You need speed and are already in ChatGPT: Use DALL-E / GPT Image.

---

## Part 2: Prompt Writing for Marketing Images

Prompt writing is a craft. A vague prompt produces a generic image. A specific prompt produces an image that looks like it was art-directed.

### Prompt Structure for AI Images

A well-structured image prompt has five components:

1. Subject: What is in the image?
2. Style or treatment: What visual style should it have?
3. Lighting: How is the scene lit?
4. Composition: Where is the subject positioned? What is in the background?
5. Technical specs: Aspect ratio, quality, resolution notes.

Weak prompt: "A woman at a laptop working."

Strong prompt: "A woman in her early 30s sitting at a minimal white desk, looking at a laptop screen with a focused expression. Natural light from a large window to her left. Slight depth of field. Professional but relaxed home office setting. Neutral tones, slight warm color grade. 16:9 aspect ratio. Photorealistic."

The strong prompt directs the model on every variable that matters. You are the art director.

### Prompts for Specific Marketing Use Cases

**Hero image (SaaS product landing page):**
"Abstract visualization of data flowing between connected nodes, clean and modern, deep blue background with bright teal and white accent lines, no text, suitable for a hero section, wide horizontal composition, 1440x900 pixels."

**LinkedIn post image (people-focused):**
"A diverse team of three professionals reviewing content on a shared laptop screen in a bright modern office. Warm natural light. Candid, collaborative feel. Not posed. Photorealistic. 1:1 square crop."

**Blog post header (concept illustration):**
"Minimalist flat design illustration of a person presenting charts to a small group. Soft pastel color palette. Isometric style. No text. Clean white background. 16:9 aspect ratio."

**Social proof quote card (Ideogram):**
"A clean typographic quote card. Text: '[Customer quote here]'. White background. Brand-specific font style: sans-serif, modern. Company name below quote in smaller text. Minimal decorative border in [brand color]. 1080x1080 pixels."

### Iterating on Prompts

Your first prompt rarely produces the final image. Expect two to five iterations.

After each generation, identify one or two specific things to change. Do not rewrite the entire prompt unless the result is completely off. Isolate the variable: "Make the background more minimal" or "Change the lighting to feel more overcast and moody" is a targeted refinement.

Save prompts that produce good results. Build a prompt library organized by image type. You will reuse and modify these more than you expect.

---

## Part 3: Product Mockup Creation

Product mockups place your software interface (or physical product) into a realistic context. They make abstract software tangible and are essential for landing pages, app stores, and social media.

### Types of Product Mockups

**Device mockups:** Your app or dashboard shown inside a laptop, phone, or tablet frame. These are the most common SaaS mockup type.

**Browser mockups:** Your web app shown inside a realistic browser window. Cleaner than device mockups for web-first products.

**Environment mockups:** Your product shown in use in a real setting. A dashboard visible on a monitor in a real office photo. More expensive to produce but very high impact.

**Comparison mockups:** Before/after or side-by-side views showing your product alongside a manual process or competitor output.

### Tools for Creating Mockups

**Mockup generators (fastest):** Smartmockups, Shots.so, Screely, Mockuper. Upload your screenshot and get a device frame around it in seconds. Good for landing pages and blog posts.

**Figma mockup templates:** More customizable than mockup generators. You control the frame, shadow, background, and angle. Search for device frame components in the Figma community.

**AI-assisted mockups:** Tools like Flair.ai specialize in placing product images into AI-generated environments. For physical products, this is faster than a professional photo shoot.

### Tips for High-Quality Product Mockups

Use a retina-resolution screenshot of your product. Low-resolution screenshots look blurry when zoomed into a device frame.

Choose a background that matches your brand palette. A product screenshot in a device frame floating against a matching brand color looks cohesive. A screenshot floating on a random gradient looks like a stock template.

Add a subtle shadow to the device or browser frame. This adds depth and separates the mockup from the background.

Show real data in the screenshot. Placeholder text (lorem ipsum) or obviously fake data ("Company ABC," "$1,234,567") breaks believability. Use realistic-looking but anonymized data.

---

## Part 4: Hero Image Strategy

Your hero image is the first visual impression on your homepage or landing page. It communicates your brand's aesthetic before any copy is read.

### Hero Image Approaches

**Abstract or conceptual:** An illustration or rendered graphic that represents your product's purpose metaphorically. This works well for products with complex or abstract value propositions (data security, AI analytics, workflow automation).

**Product screenshot:** A direct view of your interface. This works best for products with visually appealing dashboards or clearly understandable UIs. Put the most compelling screen front and center.

**People and context:** Someone in the act of using your product or experiencing the outcome it delivers. This works well for productivity, collaboration, and consumer-facing tools.

**Split or composite:** Combines product screenshot with contextual imagery. A realistic office environment on one side, product UI on the other. Requires more design effort but can communicate both context and product simultaneously.

### Hero Image Checklist

- Does the image communicate what the product is or does, at a glance?
- Does it match the tone (professional, playful, technical, warm) of the brand?
- Does it feel like it belongs with the surrounding copy, or does it feel generic?
- Is it optimized for mobile viewport (does it still look good at 375px wide)?
- Is the file size under 200KB in WebP format?
- Does it have an alt text description for accessibility and SEO?

---

## Part 5: Social Media Graphics

Social media graphics must work without audio, often without being enlarged, and sometimes without the surrounding caption. They need to communicate a message in the first 1.5 seconds.

### Size Reference by Platform

- LinkedIn feed post: 1200x627 pixels (1.91:1) or 1080x1080 (1:1 square).
- Instagram feed: 1080x1080 (square) or 1080x1350 (4:5 portrait).
- Twitter / X: 1200x675 pixels (16:9).
- Facebook feed: 1200x630 pixels.
- LinkedIn event banner: 1776x444 pixels.
- LinkedIn company page cover: 1128x191 pixels.

Use the correct size for each platform. An image designed for Twitter displayed on LinkedIn will be cropped incorrectly.

### Design Principles for Social Images

**One message per image.** If you have three things to say, make three images.

**High contrast.** Text must be readable on both light and dark screens, in feed environments with competing images around it.

**Brand consistency.** The image should be recognizable as yours within one second. Consistent font, consistent color palette, consistent visual style.

**Text legibility on mobile.** Most social images are viewed on phones. Test readability at thumbnail size before publishing.

**Leave safe zones.** Many platforms overlay your account name, follower count, or engagement metrics over parts of the image. Design the important content away from edges and corners.

### Types of High-Performing Social Images

**Stat cards:** "[Specific statistic] about [topic your audience cares about]." Simple background, bold number, source credited. Shareable because the stat itself is the value.

**Quote cards:** A short, sharp quote from a customer, a piece of research, or your own expertise. Works best when the quote is specific and surprising.

**Before/after comparisons:** Two-panel images showing a before state (the problem) and an after state (the result). Effective for productivity, design, and data products.

**Process breakdowns:** A numbered list as a visual. "4 steps to [outcome]" presented as a designed graphic. These get saved and reshared regularly.

---

## Part 6: OG Images

OG images (Open Graph images) appear when someone shares a URL on social media, Slack, or messaging apps. They are automatically pulled from the meta tags in your page HTML.

A well-designed OG image dramatically increases click-through from shared links. A missing or generic OG image makes your link look like an afterthought.

### OG Image Specifications

- Recommended size: 1200x630 pixels.
- File format: JPG or PNG (keep under 300KB).
- Content safe zone: Keep important elements within 1080x550 pixels centered, to account for cropping across platforms.

### What to Include in an OG Image

- Your product or company logo.
- The page title or a short compelling headline.
- A visual element (illustration, screenshot, or abstract background) that matches your brand.
- If the page is a blog post: the post title and optionally the author name.

### Generating OG Images at Scale

For products with many pages (blog, documentation, feature pages), generating custom OG images manually is impractical. Automate with:

- Vercel OG (Next.js): Generate OG images dynamically using HTML and CSS templates.
- Satori: An open-source library that converts HTML to SVG for OG image generation.
- Cloudinary dynamic transformations: Generate text-overlaid images from templates via URL parameters.
- Bannerbear or Placid: Paid tools that generate images from templates based on API inputs.

Automating OG images requires one well-designed template. The template defines the brand elements. The automation populates the page-specific content (title, date, category).

---

## Part 7: Profile Banners

Profile banners appear on LinkedIn company pages, Twitter/X profiles, YouTube channels, and GitHub organization pages. They are often ignored and often generic.

Your banner is free advertising space that sits above every impression your profile makes. Use it intentionally.

### What a Strong Profile Banner Contains

- Your brand's visual identity (colors, logo, font style).
- A specific message: your tagline, your product's core benefit, or a current campaign.
- Optionally: a call to action or a URL (works well on LinkedIn where visitors look for more information).

### Banner Size Reference

- LinkedIn company page: 1128x191 pixels.
- LinkedIn personal profile: 1584x396 pixels.
- Twitter / X: 1500x500 pixels.
- YouTube channel art: 2560x1440 pixels (safe zone: 1546x423 centered).
- GitHub: 1280x640 pixels.

### Updating Banners Strategically

Treat your banner like a billboard. Update it when you:
- Launch a new product or feature.
- Run a time-limited promotion.
- Announce an upcoming event.
- Have a major piece of content (a report, a guide) to promote.

A stale banner from two years ago signals that no one is actively managing the brand presence.

---

## Part 8: Image Optimization

Fast, well-formatted images improve both SEO rankings and user experience. A 1.2MB PNG hero image is a common and easily fixed problem.

### Image Format Guide

**WebP:** The best format for most marketing images. Significantly smaller file size than JPG or PNG at similar quality. All modern browsers support it. Use WebP as your default.

**AVIF:** Newer than WebP, even smaller file sizes, excellent quality. Browser support is slightly lower but growing. Use AVIF for high-traffic pages if your stack supports it.

**SVG:** Use for logos, icons, and simple vector illustrations. SVG files are infinitely scalable and extremely small. Never rasterize a logo for use on the web.

**JPG:** Use for photographs when WebP is not available. Do not use for images with text or sharp edges.

**PNG:** Use only when you need transparency (WebP also supports transparency and is smaller). Avoid large PNGs on landing pages.

### Compression Targets

- Hero images: Under 150KB in WebP.
- Blog post images: Under 100KB each.
- Social sharing images (OG): Under 300KB.
- Product screenshots in listings: Under 200KB.

Use Squoosh (browser-based), ImageOptim (Mac app), or your CDN's automatic compression (Cloudflare, Imgix, Cloudinary) to compress images.

### Lazy Loading

Add the loading="lazy" attribute to image tags that appear below the fold. This tells browsers to defer loading images until the user scrolls toward them. It reduces initial page load time without any visual degradation.

Above-the-fold images (hero, logo) should never be lazy loaded. They should be preloaded or included with the highest priority.

### Alt Text

Every marketing image should have a descriptive alt text. Write alt text that describes what is in the image, not what you want the image to convey.

Good: "A dashboard showing monthly email campaign performance metrics."
Bad: "Marketing analytics software."

Alt text serves two purposes: accessibility (screen readers use it) and SEO (search engines index it).

---

## Part 9: Brand Consistency in Visuals

Brand consistency does not mean identical images. It means a recognizable visual language that connects your images across channels.

### Defining Your Visual Language

Document these elements and apply them consistently:

**Primary color palette:** Two to three main colors. Every image uses at least one of these. Do not introduce new brand colors in marketing images without design review.

**Secondary texture or background treatment:** A gradient style, a pattern, a specific background color. Gives images a family resemblance without looking identical.

**Typography in images:** One headline font. One body font. Sizes and weights that match your website. Never use fonts in marketing images that do not appear on your website or brand guide.

**Photography style:** If you use photos of people, define the style (candid vs. posed, warm vs. cool tones, diverse vs. specific representation). Consistency in photography style is harder to achieve than consistency in illustration style.

**Illustration style:** If you use custom illustrations, document the style (flat, isometric, line art, painterly). Mixing multiple illustration styles in one brand makes the visual identity feel unintentional.

### Brand Consistency Audit

Quarterly, pull 20 randomly selected images from your website, blog, and social channels. View them together in a folder or presentation. Ask:

- Could you identify all 20 as belonging to the same brand without seeing the logo?
- Are the colors consistent?
- Is the typography consistent?
- Does the overall feeling (professional, playful, technical, warm) stay constant?

Inconsistency in your visual library usually develops gradually and is rarely noticed from inside. The audit makes it visible.

### Canva and Figma Templates for Team Consistency

If multiple people create marketing images on your team, templates are the most effective consistency tool.

Build templates in Canva or Figma for:
- LinkedIn post graphics.
- Twitter / X graphics.
- Blog post featured images.
- Quote cards.
- Event announcements.
- Promotional banners.

Lock brand elements (logo, colors, fonts) in the template. Leave only content areas editable. When anyone on the team creates an image from the template, the output is on-brand by default.
