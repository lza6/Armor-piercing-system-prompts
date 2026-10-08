# Refine Images Across Turns

Use this reference when the user wants to modify an earlier output, develop a recurring character, or complete a sequence of related edits.

## Focus each turn

- Use the last approved image as the next input.
- Request one meaningful change per turn so its effect can be evaluated.
- Repeat critical preservation requirements even when the conversation already contains them.
- Compare the new output with the approved input before adding another instruction.
- If drift begins, return to the last accepted image rather than repeatedly editing a degraded result.

Example structure:

```text
Change only [single condition or element].
Keep [identity, composition, camera, layout, text, palette, and other invariants] unchanged.
Do not add [specific unwanted changes].
```

## Character consistency

Create a reusable character reference before producing many scenes. Record only visible defining traits:

- face shape, hair, skin, and distinguishing features;
- proportions and scale;
- signature clothing or accessories;
- rendering style and tonal character.

For each new scene, repeat the defining traits that must remain stable while changing the environment, pose, expression, or action. Inspect identity and proportions across the complete set, not only one image at a time.

## Text, products, and transparency

- Repeat exact copy and occurrence count whenever text must survive an edit.
- Repeat product geometry and label-preservation requirements in every product edit.
- Repeat the transparent-background requirement in later edits and preserve an alpha-capable output format.

## Stopping rule

Stop refining when the result satisfies the user's requirements. Do not accumulate decorative changes that were not requested. If prompting repeatedly fails a requirement that must be exact, report the limitation and recommend a deterministic compositing, typesetting, or image-editing step.
