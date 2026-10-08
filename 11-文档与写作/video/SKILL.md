---
name: video
description: >
  Use this skill for creating, generating, and producing video content for marketing using AI tools and production workflows. Triggers on: AI video generation, Runway, Kling, Pika, HeyGen, Synthesia, Veo, Remotion, explainer video, product demo video, talking head video, video script, video SEO, video distribution, video content strategy, and any request to plan or produce marketing video without a full production team.
---

# Video Content for Marketing

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

## Video in the Modern Marketing Stack

Video is the format that most marketing teams under-produce and most buyers prefer. Buyers in both B2B and B2C consistently report preferring video over text when learning about a product, evaluating options, and making decisions. Yet most companies produce video irregularly, treat it as expensive, and rely on a creative agency once or twice per year.

The shift that changed this is AI. A set of tools that emerged between 2022 and 2025 reduced the cost and time required to produce professional-quality video to a fraction of what it previously required. You no longer need a film crew, a studio, or a motion graphics team to produce a compelling product video. You need the right tools, a good script, and a production workflow.

The question worth sitting with before building a video program is this: what does your buyer need to see and hear in order to trust your product enough to take the next step, and can a video deliver that more effectively than any other format?

For most companies, the answer is yes for at least three or four content categories. Those are the categories to start with.

---

## AI Video Generation Tools

The AI video generation landscape is active and evolving. The tools break into two categories: those that generate video from text or image inputs, and those that produce avatar-based or talking-head video from a script.

**Runway (runwayml.com)**

Runway is one of the most capable general-purpose AI video generation tools. It generates video from text prompts, from image prompts, and from reference video footage that it can extend or modify. Its Gen-3 Alpha model produces cinematic, realistic output that works well for stylized brand footage, abstract product visualizations, and promotional content that does not require photorealistic accuracy.

Runway is best for: generating atmospheric or mood-setting footage, extending existing video clips, and creating visuals for ads or social content where a realistic human presence is not required. It is not the best tool for generating a person speaking naturally on camera.

Typical generation time is 20 to 90 seconds of output per generation. You iterate through multiple generations to find the output that matches your creative direction.

**Kling (klingai.com)**

Kling, developed by Kuaishou, generates video with particularly strong physics and motion coherence. It handles camera movement, subject motion, and environmental detail well. Its text-to-video output often looks more grounded and realistic than Runway's more stylized aesthetic.

Kling is best for: product visualizations, nature and environmental footage, and scenes where realistic motion and physical behavior are important. A coffee product shot where steam rises naturally from the cup, or a running shoe in motion, benefits from Kling's physics fidelity.

**Pika (pika.art)**

Pika is designed for shorter, faster video generation with a strong focus on usability. It allows you to generate short clips from images or text, modify existing video (changing a character's outfit, transforming the setting), and produce motion graphics-style content.

Pika is best for: quick social video iterations, image-to-video conversion (animating a product photo), and repurposing existing creative assets into motion.

**HeyGen (heygen.com)**

HeyGen generates avatar-based video where a digital human delivers a script. You choose an avatar (from HeyGen's library or create one from your own likeness), type the script, and HeyGen produces a video of the avatar speaking naturally with synchronized lip movements.

HeyGen is best for: product explainers, internal training content, personalized sales videos, and educational content where a human presenter improves clarity but producing a live-action video would require scheduling, a set, and a camera operator. Many companies use HeyGen to produce localized video in multiple languages without hiring separate presenters for each language.

The quality of HeyGen's lip sync and natural speech rendering has improved significantly and is now production-quality for most business contexts. The main limitation is the avatar's inability to convey strong emotional nuance. Use it for clear, informational content rather than emotionally driven brand storytelling.

**Synthesia (synthesia.io)**

Synthesia occupies a similar category to HeyGen. It produces presenter-based video from a script and avatar. Synthesia has a stronger enterprise focus, with more robust access controls, team collaboration features, and a larger library of multilingual presenters.

Synthesia is best for: companies producing high-volume presenter-based video at scale, particularly for HR, training, and enablement content that needs to be produced across multiple languages and updated regularly.

**Veo (Google DeepMind)**

Google's Veo model, available through Vertex AI and integrated into YouTube's production tools, is one of the most capable text-to-video generators for high-fidelity, cinematic output. It handles complex prompts, long-form sequences, and maintains visual consistency across shots more reliably than many alternatives.

As of mid-2025, Veo is most accessible through Google's enterprise tools and YouTube's Shorts creation features. If you produce content for YouTube or work within Google's ecosystem, Veo is worth integrating into your workflow.

---

## Programmatic Video with Remotion

Remotion (remotion.dev) is a framework for building videos programmatically using React and JavaScript. Instead of editing video in a timeline tool, you write code that defines every element of the video: which text appears when, which graphics move where, which data values populate which fields.

Remotion is best for: data-driven video at scale. If you need to produce 500 personalized product videos where each video shows a specific customer's company name, their usage data, or their custom configuration, Remotion is the right tool. It produces videos from a template and a data source.

Use cases where Remotion produces significant leverage:

**Personalized sales outreach videos.** Input a list of prospects with their company name, industry, and a specific pain point. Generate 200 unique 30-second videos where each video opens with the prospect's company name, references their industry, and walks through a relevant product use case.

**Dynamic report videos.** A monthly business review video that pulls in actual metrics from your analytics database, renders them as animated charts, and produces a polished video report without any manual editing.

**Social content templates.** A repeating content format (a weekly stat, a quote, or a product feature highlight) built as a Remotion template. Each week, update the data and export a new video in minutes.

Remotion requires React development skills. If you have a developer on staff or a developer who works with your team, the setup investment (typically 2 to 4 days for a first template) pays back quickly for high-volume use cases. A team producing 20 personalized videos per week manually versus generating them from a Remotion template will recoup that setup time within weeks.

---

## Explainer Video Scripts

An explainer video answers the question: what does this product do and why should I care? It is typically 60 to 120 seconds long and targets a viewer who has some awareness of your category but has not yet committed to understanding your specific approach.

An explainer video script follows a tight structure:

**Opening (0 to 10 seconds): The problem in the viewer's world.** Name the specific frustration, inefficiency, or situation that your product addresses. Do not open with your company name or logo. The viewer does not care yet. They care about whether the problem you name matches theirs.

Example: "Your finance team is still closing the books manually. You're pulling reports from three different systems and reconciling them in a spreadsheet that breaks every quarter-end."

**Middle (10 to 50 seconds): How your product addresses the problem.** Describe your solution in terms of what it does for the user, not how it is built. Show the key step or workflow that eliminates the problem you named. Keep the vocabulary simple. Avoid jargon. Use motion graphics or screen recording to show the product doing the relevant thing, rather than talking about it in the abstract.

**Close (50 to 90 seconds): What happens next and why now.** Describe what life looks like after the problem is solved. End with a clear call to action: start a free trial, watch a full demo, or talk to the team. One action. Not three options.

Write explainer scripts at a reading pace of 130 to 150 words per minute. A 90-second script should be approximately 200 to 225 words. Read your script aloud before finalizing it. Scripts that look natural on paper often sound stilted when spoken.

---

## Product Demo Video Structure

A product demo video is a walkthrough of the product that shows a specific use case from start to finish. Unlike an explainer video (which describes what the product does conceptually), a demo video shows the product working in real conditions.

Product demo videos should be structured around a use case, not a feature list. A feature tour is not a demo. A demo shows a specific user achieving a specific outcome using your product.

Structure a demo video this way:

**State the scenario (30 seconds).** Name the user, their role, and the task they are trying to complete. "This is Sarah. She's a finance manager at a 200-person company. Every month, she needs to close the books, reconcile three bank accounts, and generate a board-ready report. Historically this has taken her team 3 days."

**Show the starting state (30 seconds).** Open the product dashboard. Show where Sarah begins. This grounds the viewer in a realistic starting point and gives the product context.

**Walk through the core workflow (2 to 4 minutes).** Show Sarah completing the task using your product. Narrate each step by connecting the action to the outcome, not just describing what is happening on screen. "Sarah imports the bank data with one click. The system matches transactions automatically, which is what used to take her team an afternoon."

**Show the result (30 to 60 seconds).** What did Sarah produce? Show the output. Quantify the time or effort saved. "The report that used to take 3 days is done in 45 minutes."

**Call to action (15 seconds).** One step the viewer can take to try this themselves.

Screen recording tools like Loom, ScreenStudio, or CleanMyMac's recording feature produce clean screen recordings that work as a demo base. Record at 1080p minimum. Zoom in on key interactions so small UI elements are legible even when the video is watched on a phone.

---

## Talking Head Video Setup

Talking head video is the format where a person speaks directly to camera. It is the most personal video format and, when done well, it builds trust faster than any other format. The challenge is that most people produce talking head video with poor lighting, poor audio, or a distracting background, which makes the format feel unprofessional.

The four variables that determine talking head video quality in order of impact:

**Audio quality.** Bad audio destroys a video. Viewers will watch a slightly blurry video, but they will close a video with bad audio within 30 seconds. A USB condenser microphone (the Blue Yeti or Rode NT-USB are reliable choices) positioned 8 to 12 inches from the speaker's face makes an enormous difference over laptop built-in microphones. An affordable lavalier microphone clipped to the collar also works well for desk setups.

**Lighting.** One well-placed key light makes a talking head video look professional. Place a ring light or a large softbox directly in front of the speaker and slightly above eye level. Avoid windows behind the speaker (creates silhouette). Avoid overhead lighting only (creates unflattering shadows). A simple desk ring light costs $40 to $80 and transforms the image quality.

**Background.** A clean, uncluttered background keeps the viewer focused on the speaker. A bookshelf with intentionally placed items works well. A plain wall works. A busy, chaotic background draws attention away from the speaker. If filming in a less-than-ideal environment, consider a virtual background or a portable backdrop.

**Camera position and angle.** The camera should be at or slightly above eye level. A camera significantly below eye level creates an unflattering angle. A camera significantly above eye level makes the speaker look small. Position the camera so the speaker's eyes are roughly in the upper third of the frame.

For teams that produce talking head video regularly, a simple, permanent studio setup (dedicated desk or corner with fixed lighting, microphone, and backdrop) eliminates the setup time barrier that causes video to be deprioritized.

---

## Video SEO

Video content can rank in Google search results, appear in YouTube search, and surface in Google's AI Overviews. Getting that visibility requires treating video with the same SEO attention you apply to written content.

**YouTube SEO:**

YouTube is the second largest search engine. Videos that rank in YouTube search can generate significant, compounding organic traffic. The factors that matter:

- Title: Include the primary keyword naturally near the beginning. "How to Set Up Email Automation in HubSpot (Step by Step)" targets the keyword while being descriptive.
- Description: Write 200 to 300 words in the description. Include the primary keyword, related terms, and a clear summary of the video content. YouTube uses the description to understand the video's topic.
- Tags: Less important than they used to be, but still worth adding 5 to 10 relevant tags.
- Transcript/Captions: Upload a transcript or use YouTube's auto-caption feature and correct errors. YouTube uses caption text to understand content, which improves ranking for long-tail queries.
- Chapters: Add timestamp chapters in the description. YouTube displays these in search results and allows viewers to jump to relevant sections, which improves engagement.
- Thumbnail: The thumbnail is the most important click-through rate factor. Use a custom thumbnail with a clear, legible title and a human face (faces increase click-through rate). Bright, high-contrast images outperform dark or subtle ones.

**Embedding video for Google search:**

When you embed a video on a page of your website, add VideoObject schema markup to that page. This makes the video eligible to appear as a rich result in Google search with a thumbnail and duration. Add Clip markup to mark timestamps that correspond to specific sections, which allows Google to surface specific chapters of the video for relevant queries.

---

## Video Distribution Strategy

Creating video without a distribution plan wastes the production investment. Video distribution is the plan for getting each video in front of the right audience through the right channels.

A video distribution plan covers three tiers:

**Owned channels.** Your website, your email list, your social media accounts, and your YouTube channel. These are the channels you control directly. Publish every video to owned channels first. Embed product and explainer videos on relevant website pages. Feature recent videos in email newsletters. Post videos to your LinkedIn, Instagram, and TikTok accounts adapted to each platform's format requirements.

**Paid channels.** Video ads on LinkedIn, Meta, and YouTube allow you to reach audiences beyond your existing followers. Video performs well as paid ad creative because the autoplay behavior in feeds captures attention. Repurpose your best-performing organic videos as paid ad creative rather than producing separate ad-only content.

**Earned channels.** Coverage in industry publications, shares by influencers or partners, and embedding by other websites. You cannot control earned distribution, but you can encourage it by creating genuinely useful or shareable video content and reaching out to relevant publications and podcasts that might embed or reference your videos.

Different video types work better on different channels:

- Short-form educational content (60 to 90 seconds): TikTok, Instagram Reels, LinkedIn, YouTube Shorts.
- Product demos and explainers (2 to 5 minutes): YouTube, website product pages, email nurture sequences.
- Customer testimonials and case study videos: Website, LinkedIn, sales outreach.
- Long-form tutorials and deep dives (10 to 30 minutes): YouTube, course platforms, community channels.

---

## Building a Video Production Pipeline

A sustainable video production pipeline produces video consistently without requiring every video to be a major project. The key is systematizing the repeatable work so that creative effort goes into content and strategy, not logistics.

A minimum viable video production pipeline for a small marketing team:

**Content planning (monthly):** Decide which video topics to cover in the month based on your content calendar. Prioritize topics that serve specific pipeline stages or address the most common buyer questions.

**Scripting (per video):** Use a consistent scripting template. For short-form social video, a one-page script. For explainer videos, a three-section script (problem, solution, CTA). For demo videos, a scenario-based outline with narration notes. Scripting should take 30 to 60 minutes per video when you use templates.

**Production (per video):** Decide which format and tool matches the content. Talking head for credibility and personality content. Screen recording for tutorials and demos. AI-generated visuals for abstract or atmospheric content. Avatar video for scalable multilingual content. The right tool varies by use case. Do not default to one tool for all video types.

**Editing (per video):** Use a tool matched to the complexity of the edit. CapCut handles basic cuts, captions, and music for short-form social video efficiently. DaVinci Resolve handles more complex editing. Descript edits video by editing a transcript, which is faster for talking head content with heavy narration. For AI-generated content assembled from multiple clips, Runway or Adobe Premiere handle the assembly.

**Captions and localization:** Add captions to every video. Captions are required for accessibility and significantly increase completion rates on social platforms where videos autoplay silently. If you produce content for multiple languages, tools like HeyGen and Synthesia handle the localization in the initial production step.

**Publishing and tracking:** Publish with consistent metadata (title, description, tags, thumbnail) using a checklist. Track performance weekly: views, watch time, click-through rate, and any conversion events attributed to the video (sign-ups, demo requests, or purchases from the landing page where the video is embedded).

The output of this pipeline, run consistently, is two to four videos per week from a team with no dedicated videographer. That volume is enough to build a YouTube channel, maintain a social presence, and keep the product pages current without treating video as a special project.
