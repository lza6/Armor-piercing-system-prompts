---
name: content-humanizer
description: >
  Use this skill when the user wants to make AI-generated or stiff content sound
  more human, natural, specific, clear, and brand-aligned. Trigger phrases include
  "humanize this," "make this sound less AI," "remove AI tone," "make this more
  natural," "rewrite for human readers," "AI-pattern audit," "polish this draft,"
  and "make this sound like me."
---

# Content Humanizer

## Mandatory Content Standards

- Write in active voice.
- Use short sentences and plain language.
- Address the user as "you" and "your."
- Use `.marketing-os/product-context.md` when it exists. If it is blank or missing, recommend running `product-marketing` first.
- Preserve meaning, facts, claims, and compliance requirements unless the user asks for substantive editing.
- Do not invent stories, data, quotes, credentials, or outcomes.
- Explain major edits when they affect strategy, claims, or tone.
- Do not use em dashes, hashtags, emojis, or filler closings.
- End every full deliverable with one specific next step.

## Objective

Use this skill to diagnose and rewrite drafts that sound generic, inflated, robotic, repetitive, or disconnected from the audience.

This skill complements `copy-editing`. Use `copy-editing` for broad editorial improvements. Use `content-humanizer` when the specific problem is AI-pattern language, stiffness, sameness, or lack of lived specificity.

Next step: identify the target voice before rewriting.

## AI-Pattern Audit

| Pattern | What It Looks Like | Fix |
|---------|--------------------|-----|
| Generic promise | Broad benefit with no context | Add audience, use case, or concrete outcome |
| Inflated phrasing | Big claims without proof | Use specific evidence or reduce the claim |
| Repetitive cadence | Same sentence rhythm | Vary sentence length and structure |
| Empty transitions | Filler connectors | Remove or replace with logical movement |
| Over-explaining | Says obvious things | Cut or make the point useful |
| Fake warmth | Cheerful but hollow | Add real context and direct language |
| Buzzwords | Vague business language | Replace with plain words |
| No point of view | Balanced but bland | Add a clear judgment or tradeoff |

Next step: mark the three patterns that hurt trust most.

## Rewrite Modes

| Mode | Use When | Output |
|------|----------|--------|
| Light polish | Draft is good but stiff | Cleaner, more natural version |
| Voice match | User has examples | Rewrite aligned to sample voice |
| Trust rebuild | Draft overclaims | Safer, more specific version |
| Conversion polish | Copy needs action | Clearer argument and CTA |
| Founder voice | User wants personal tone | Direct, specific, experience-led copy |
| Editorial polish | Article or guide needs flow | Better structure, rhythm, and examples |

Next step: choose one rewrite mode before editing.

## Humanization Rules

- Replace vague claims with concrete details.
- Cut sentences that do not move the reader forward.
- Use contractions when the brand voice permits them.
- Add real constraints, tradeoffs, or examples.
- Keep paragraphs short.
- Use verbs that show action.
- Remove performative excitement.
- Replace abstract nouns with specific subjects.
- Keep CTA language direct.

Do not hide weak strategy with prettier language.

Next step: fix the argument before polishing style.

## Output Format

| Section | Original Issue | Rewrite Decision |
|---------|----------------|------------------|

Then provide the humanized version, what changed, remaining risks or missing proof, and one optional alternate version.

For line edits, use:

| Original | Revised | Why |
|----------|---------|-----|

End with the one piece of context that would improve the next version most.

