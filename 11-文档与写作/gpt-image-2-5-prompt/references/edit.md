# Edit Images

Use this reference whenever the request includes one or more input images. The prompt should make the intended change narrow and make preserved details explicit.

## Required inputs

- Confirm that every necessary reference image is actually available.
- Number references in input order and assign each one a role such as subject, identity, clothing, product, style, layout, or background.
- Explain how the references combine and where each referenced element belongs.
- Do not infer unseen content or silently swap one reference's role for another.

## Editing pattern

```text
Edit goal:
Change only:
Reference roles:
Preserve exactly:
Match or adapt:
Do not add or change:
```

Use natural language instead of labels when the edit is simple. The important distinction is between the requested change and the invariants.

## Local edits and object changes

- Name the object and the single operation: remove, replace, move, recolor, or add.
- Preserve camera angle, composition, pose, lighting, shadows, surrounding objects, and color treatment when those should remain stable.
- If moving or inserting an element, specify its destination, relative scale, perspective, contact, and how scene lighting should affect it.
- Use a mask when the available tool supports it and an exact editable region is needed.

## Identity, products, and clothing

- State the facial features, proportions, pose, body framing, product geometry, label, or other identity-defining details that must remain recognizable.
- For clothing changes, give the clothing references that role and allow only the outfit to change.
- For product placement, separate the product reference from the scene or style reference and preserve label legibility.
- Inspect the result; prompting cannot guarantee pixel-identical preservation.

## Style transfer

- Assign the style reference a limited role such as palette, texture, lighting, or medium.
- Describe the new subject independently.
- State which structural or identity details must not be inherited from the style image.

## Translation and layout preservation

- Provide the target language and, when possible, the exact translated copy.
- Ask to replace only the source text while preserving the layout, hierarchy, diagrams, colors, and other visual elements.
- Check for untranslated fragments, spelling errors, overflow, and unintended layout changes.

## Sketch to realistic image

- Preserve the drawing's layout, perspective, proportions, and intended objects.
- Add realism through plausible materials, lighting, texture, and environment.
- State that no new elements or text should be introduced when creative reinterpretation would be harmful.

## Transparent cutouts

- Ask to isolate the subject with a clean silhouette and preserved geometry and labels.
- Exclude halos, fringing, scenery, solid backdrops, checkerboards, and unwanted shadows.
- Also set transparent background and a compatible output format as described in [api-parameters.md](api-parameters.md).
- Inspect the decoded alpha channel around hair, glass, shadows, and object edges.

## Preservation limit

Repeated image-model edits can alter details that were supposed to stay fixed. Restate critical constraints and inspect every result. If a region must remain pixel-identical, create the changed region separately and composite it into the approved original with an image-editing workflow instead of relying on prompting alone.
