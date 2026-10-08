---
name: clothing-three-view
description: Generate mandatory pure-white-background, identity-free three-view clothing product images from user-provided model wearing photos. Use when the user provides one or more apparel/model try-on images and wants one combined image per color containing front, side, and back garment views with zero real-human identity pixels, following a horizontal three-view template layout. Generate the final bitmap only through Codex's built-in cloud `image_gen.imagegen` tool, without user-supplied API keys, direct developer-API scripts, local image models, or local compositing.
---

# Clothing Three View

## Purpose

Transform user-provided clothing model photos into clean ecommerce-style three-view images. For each clothing color, produce exactly one pure-white-background composite image containing front, side, and back garment views in one horizontal image. The output must contain zero real-human identity pixels: no skin, neck, chest, collarbone, shoulders, arms, hands, fingers, nails, tattoos, jewelry, face, or hair. Present the garment on an invisible ghost-mannequin support whenever possible; if construction cannot be read without support, use only a featureless, uniformly neutral, non-skin-colored mannequin with no person-specific anatomy. Focus on garment fit, silhouette, neckline, sleeves, hem, fabric, closures, seams, drape, and color.

## Mandatory ChatGPT Image2.0 Generation Gate

The final three-view bitmap must be generated directly with ChatGPT image2.0 from the user's product/model-wearing reference images. Do not create the final three-view image with local background-removal, local cutout, local compositing, or local image-processing workflows.

### Mandatory Codex Cloud ImageGen Execution Path

Use the current Codex session's built-in `image_gen.imagegen` cloud image-generation/editing tool as the only execution path for every initial generation and correction regeneration. Treat every reference to "ChatGPT image2.0" in this skill as the required OpenAI cloud image capability accessed through this Codex tool, not as a locally installed model or permission to call the developer API directly.

Execution constraints:

- Call `image_gen.imagegen` directly from the current Codex session for the final bitmap and every regenerated correction.
- Do not ask the user to configure or provide an OpenAI API key.
- Do not write or run scripts that call the OpenAI developer API directly.
- Do not install, load, or run any local image-generation model.
- Keep all existing prohibitions on local cutout, background removal, background replacement, masking, repair, and compositing.
- If `image_gen.imagegen` is absent, unavailable, or fails to provide image-generation access, stop immediately, report the exact tool-access blocker, and do not switch to an API, local model, or local image-processing fallback.
- When every target source image has a local path, pass all of them with `referenced_image_paths`. Otherwise use `num_last_images_to_include` with the smallest value that includes every target conversation image. Never supply both mechanisms in one call.


Hard prohibitions for final three-view creation:

- Do not use `rembg`, PIL/Pillow cutouts, OpenCV, Photoshop-style local extraction, shell scripts, local canvas compositing, background-removal services, or any other local image-processing pipeline to produce the final reference image.
- Do not treat a locally cropped, masked, pasted, or background-replaced source photo as the final three-view output.
- Do not use local tools to "fix" a failed image2.0 result by cutting/pasting garment regions. Local tools may only be used for file inspection, opening images, or non-destructive visual QA.
- If ChatGPT image2.0 or an equivalent ChatGPT image editing/generation capability is unavailable, stop before generating the three-view image and report the exact tool-access blocker. Never fall back to local cutout/compositing.

Required image2.0 loop:

1. Inspect all source images for the target color before generation and write an internal detail checklist: color, garment category, neckline, sleeve/armhole shape, front knot/twist/gathering, waist seam, garment silhouette, length, hem, side/back construction, fabric surface, drape, all real-human identity pixels and accessories that must be excluded, and any missing view evidence.
2. Call ChatGPT image2.0 to generate one pure-white-background horizontal three-view composite from the source images and checklist.
3. Visually compare the generated output against the original product images, not only against the prompt. Check every visible detail in the checklist.
4. If any garment detail differs or any real-human identity pixel remains, regenerate with ChatGPT image2.0 using a correction prompt that names the exact mismatch or contamination. Repeat until the output matches the product evidence and passes the zero-human-identity-pixel gate.
5. Record the quality-check status as passed only after the source-vs-output comparison finds no material garment difference and the output contains no real skin, human body part, jewelry, tattoo, or other person-specific cue.

## Non-Negotiable Output Format

Every final output must follow this format:

- One garment color per final image.
- Exactly one combined bitmap image per color, not three separate files.
- Three views arranged left to right in this order: front view, side view, back view.
- Layout should follow the user's template style when provided: three full garment views on one wide canvas, similar scale, aligned vertically, clean spacing between views, no labels, no borders, no decorative background.
- Background must be pure white studio background. Do not keep beige, gray, indoor, outdoor, lifestyle, furniture, wall, floor, plant, mirror, street, or any source-photo background.
- Show zero real-human identity pixels. Prohibit all skin and human anatomy, including head, face, hair, neck, chest, collarbone, shoulders, arms, elbows, wrists, hands, fingers, nails, legs, tattoos, and jewelry. Cropping a person above the mouth or neck is not sufficient.
- Use an invisible ghost mannequin or internal garment support by default. When that cannot show construction accurately, use only a featureless, uniformly neutral, non-skin-colored mannequin without a head, limbs, anatomical skin texture, or person-specific proportions.
- Show enough garment length to prove the hem, fit, side drape, and back coverage without exposing a real body.

## Required Behavior

1. Inspect every user-provided reference image before generating. Identify garment type, colors, visible details, fit, fabric texture, print/pattern, trims, neckline, sleeves, hem, pockets, closures, seams, and any logo/decoration.
2. Group outputs by garment color. If the user provides multiple colors, create one separate composite image per color.
3. Use ChatGPT image2.0 directly for the final bitmap generation. No local cutout, local background removal, or local compositing is allowed for the final output.
4. Each composite image must contain exactly three views arranged left to right: front view, side profile or three-quarter side view, back view.
5. Force a pure white seamless studio background in all three panels. Remove or regenerate away all original lifestyle backgrounds, props, furniture, shadows from the original scene, text labels, watermarks, borders, decorative elements, and unrelated objects.
6. Remove every real-human pixel, not only the face and hair. Never retain skin, neck, chest, collarbone, shoulders, arms, wrists, hands, fingers, nails, tattoos, jewelry, or an identifiable body silhouette. Reveal neckline and collar construction through the garment edge and invisible internal support.
7. Keep the same garment scale and support geometry across the three views for a given color. Maintain realistic fabric drape, garment length, silhouette, sleeve scale, hem position, and color consistency without introducing person-specific proportions.
8. Keep the front, side, and back views visually balanced like a product reference sheet: similar height, similar body scale, centered panels, and enough whitespace so the garment can be inspected.
9. Do not invent unavailable product details. If a side or back detail is not visible, infer conservatively from the front/reference images and keep it simple.
10. Generate final bitmap images. When the user invokes this skill directly, the final user-facing response must show only the resulting image or images, with no explanatory text, bullets, captions, labels, or process notes. In plugin or batch workflows, internally record and pass the final image path for each color to the downstream scriptwriting and PopBoom steps. When staging that image for PopBoom, copy its bytes unchanged into the run directory; never resize, recompress, or re-encode the accepted three-view.

## Batch Efficiency And Reference Ledger

For multi-color apparel batches, use a lightweight reference ledger before generation so every output is traceable and consistent.

For each color, note internally:

- color / variant name;
- source image paths assigned to that color;
- best available front image, side image, and back image;
- missing or weak view evidence, if any;
- visible garment details that must be preserved;
- ChatGPT image2.0 generation attempt count and correction notes;
- source-vs-output detail comparison result;
- final three-view output path and validation status.
- final image SHA-256 plus a hash-bound `identity_cue_audit` using `zero_human_identity_pixels_v1`; record full-resolution inspection, review time, `passed: true`, and explicit `false` values for skin, face, hair, neck/chest/collarbone, shoulders/arms/wrists, hands/fingers/nails, tattoos/jewelry, and person-specific body shape. Persist `identity_cue_audit_sha256` as the canonical SHA-256 of that complete audit.

Efficiency rules:

- Group all images by color before generating any final three-view composites.
- Prefer real source evidence for each view when available. Use conservative inference only for missing side/back details.
- Reuse the same layout, crop height, body scale, and white-background style across all colors of the same product so PopBoom receives consistent references.
- If one color has complete front/side/back evidence and another color is missing a view, use the complete color's construction only as a shape guide while preserving the target color and visible target-color details.
- Do not regenerate a three-view output that already passed the quality check unless the user requests a new version or the source grouping was corrected.
- If a color cannot be grouped confidently, ask before generation rather than mixing variants.
- In a quality-gated streaming batch, mark one color `release_ready` immediately after its own source-versus-output audit passes and persist its final path, SHA-256, checklist, and QC result. Downstream work may finalize and submit that color without waiting for other colors.
- Never mark a color ready from a first generation merely because it looks plausible. The complete audit and any required cloud regeneration remain mandatory for every color.
- Keep the passed reference immutable after its paid release. If a later color exposes a real construction conflict, block that later color or split it into a new SKU/run; do not alter the already released reference or pretend the garments are color-only variants.

## Image Generation Prompt Pattern

When calling an image generation or image-editing tool, include:

- use ChatGPT image2.0 directly from the provided reference images; do not cut out, paste, composite, or background-remove locally
- pure white seamless studio background
- ecommerce apparel product three-view composite
- one wide combined image, not separate images
- same garment and same color in all three views
- front view on the left, side view in the middle, back view on the right
- zero real-human identity pixels: no skin, head, face, hair, neck, chest, collarbone, shoulders, arms, wrists, hands, fingers, nails, tattoos, jewelry, or person-specific body cues
- invisible ghost-mannequin support; only when indispensable, a featureless uniformly neutral non-skin-colored headless and limbless mannequin
- similar garment scale and neutral support geometry across the three views
- preserve garment color, cut, pattern, fabric texture, logos, pockets, seams, closures, neckline, sleeves, hem, drape, and back/side construction from references
- clean product-reference layout similar to the user's supplied template, but with a pure white background
- no text, no labels, no watermark, no props, no extra garments, no busy background, no lifestyle scene

## Source-Vs-Output Difference Audit

After every ChatGPT image2.0 generation, compare the output with the original product images before accepting it.

Reject and regenerate if any of these differ from the source evidence:

- garment color or shade;
- neckline depth, collar/edge shape, or trim;
- sleeve/strap/armhole width;
- front knot, twist, pleat, gather, ruched center, or waist detail;
- body silhouette, waist placement, skirt/body flare, length, or hem shape;
- side drape, back coverage, seam placement, or back neckline;
- fabric surface, visible texture scale, opacity, thickness impression, or drape;
- missing or invented pockets, buttons, logos, straps, slits, seams, panels, or decorations;
- any real-human identity pixel or cue, including skin, hair, head, face, neck, chest, collarbone, shoulders, arms, wrists, hands, fingers, nails, tattoos, jewelry, or person-specific body shape; also reject props, bags, scene elements, labels, borders, or watermarks.

Correction prompts must be specific, for example: `Regenerate with the exact V-neck depth and front twist-gather from the source images; the previous output made the neckline too shallow and removed the center knot folds. Keep the same apricot color and A-line skirt length.`

## Quality Check

Before final delivery, verify each output:

- one color per image
- exactly one combined image per color
- exactly three views
- views are ordered front, side, back from left to right
- background is pure white, not beige or lifestyle
- zero real-human identity pixels are present; no skin, neck/chest/collarbone, arms/hands/fingers/nails, tattoos, jewelry, face, hair, or person-specific body cue survives
- garment is large enough to inspect neckline, sleeves, body, hem, side drape, and back using invisible or featureless neutral support
- the three views have consistent garment scale, color, fit, and details without person-specific proportions
- every checklist detail has been compared against the original product image and no material difference remains
- no extra visible text, labels, borders, props, watermarks, or captions
- `human_identity_pixels_absent: true` has been recorded for the output; any other value blocks PopBoom staging
- the complete `identity_cue_audit` is bound to the exact output SHA-256 and every cue is explicitly false; the summary boolean alone is never sufficient
- the output can be safely used as the only PopBoom apparel reference material for that color only after the zero-human-identity-pixel gate passes
- direct single-skill final response contains images only; plugin or batch workflow records and passes the output image paths internally

If any check fails, regenerate with ChatGPT image2.0 before delivering it. Do not repair the output with local cutout, background removal, or compositing.
