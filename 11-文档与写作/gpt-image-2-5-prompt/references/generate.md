# Generate Images

Use this reference for text-to-image requests. Translate the user's desired result into visible, inspectable instructions rather than abstract mood words alone.

## General generation pattern

Include only applicable fields:

```text
Deliverable and use:
Subject and action:
Composition and placement:
Style, medium, and materials:
Lighting, palette, and atmosphere:
Required text:
Constraints and exclusions:
```

Short requests do not need every label. Keep simple prompts simple.

## Photographs and cinematic scenes

- Describe the subject, framing, light, texture, and intended degree of realism.
- Say `photorealistic` or `real photograph` when that distinction matters.
- For people, state the body framing, gaze, pose, relative scale, and physical interaction with objects.
- For wide, rainy, low-light, neon, or cinematic scenes, describe scale, atmosphere, and color instead of relying on a single mood adjective.
- State unwanted retouching, glamorization, or artificial polish only when it conflicts with the intended result.

## Exact text and branded layouts

- Put required copy in quotes and preserve it verbatim.
- State where it appears, how many times it appears, and the desired typographic treatment.
- For unusual names, spell the word letter by letter when needed.
- Request no additional text, unrelated logos, or watermarks when those would be errors.
- Treat spelling and legibility as post-generation checks, not guaranteed outcomes.

## Information graphics, educational visuals, and charts

- Name the audience, learning or communication goal, and exact artifact type.
- Supply the real labels, numbers, relationships, axes, legends, and citations that must appear. Do not ask the image model to invent factual data.
- Favor readable hierarchy, consistent iconography, clear arrows, sufficient whitespace, and restrained decoration.
- For slides, describe the canvas, sections, hierarchy, and visual language as an artifact specification rather than an illustration concept.
- Verify both the visual result and its factual relationships.

## Interfaces

- Describe the product as an existing, usable interface.
- Specify real interface elements, layout, hierarchy, spacing, typography, and device framing.
- Avoid concept-art language when the result should resemble a shipped product.
- Treat the result as a visual preview, not functional UI code.

## Comics and sequential scenes

- Define one concrete visual beat per panel.
- Number the panels and describe actions, staging, and continuity explicitly.
- Keep the sequence readable and avoid packing multiple story events into one panel.

## Logos and transparent assets

- Describe the brand, defining shapes, silhouette, negative space, padding, and small-size legibility.
- Ask for original, non-infringing work when creating a new mark.
- For transparent output, request an isolated centered asset in the prompt and use the API settings in [api-parameters.md](api-parameters.md).
- Do not confuse a painted checkerboard or white background with alpha transparency.

## Historical or real-world scenes

- Name the place and date when they define the context.
- State the period details that matter, but do not assume inferred clothing, staging, signage, or surroundings are historically accurate.
- Inspect or independently verify historically important details before use.
