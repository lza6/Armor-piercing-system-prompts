# GPT Image 2.5 Models and Request Settings

Use this reference only when model choice, API settings, or actual execution is relevant. Keep these settings outside the visual prompt.

Source status: official OpenAI documentation verified on 2026-09-10. Recheck the [image prompting guide](https://developers.openai.com/api/docs/guides/image-prompting), [image generation guide](https://developers.openai.com/api/docs/guides/image-generation), and model pages before treating mutable availability, limits, or pricing as current.

## Choose a model

| Need | Start with |
| --- | --- |
| Fast, high-quality everyday generation | `gpt-image-2.5-flare` |
| Demanding quality or precise editing | `gpt-image-2.5-sunburst` |

If Sunburst passes the quality requirement, Flare can be tested with the same prompt, references, dimensions, format, and explicit quality setting. Switch only if its quality remains acceptable and latency improves. Do not assume the faster model is cheaper; confirm current pricing separately.

Available aliases and dated snapshots documented at verification time:

- `gpt-image-2.5-flare`
- `gpt-image-2.5-flare-2026-09-08`
- `gpt-image-2.5-sunburst`
- `gpt-image-2.5-sunburst-2026-09-08`

There is no documented API model ID named only `gpt-image-2.5`.

## API entry points

- Use the Image API for direct one-shot generation or editing.
- Use the Responses API image generation tool for conversational or multi-turn image workflows.
- In the Image API, select the GPT Image 2.5 model directly.
- In the Responses API, select a supported mainline model at the response level and set the GPT Image 2.5 model in the image-generation tool's `model` field.

Do not include endpoint names or model IDs inside the visual description.

## Supported prompt-guide settings

| Parameter | Values |
| --- | --- |
| `model` | `gpt-image-2.5-flare` or `gpt-image-2.5-sunburst` |
| `quality` | `auto`, `low`, `medium`, `high`, `xhigh`, or `max` |
| `size` | `auto` or a supported `WIDTHxHEIGHT` resolution |
| `background` | `auto`, `opaque`, or `transparent` |

Common documented sizes include:

- `1024x1024`
- `1536x1024`
- `1024x1536`
- `2048x2048`
- `2048x1152`
- `3840x2160`
- `2160x3840`

Custom dimensions must satisfy all of these constraints:

- Each edge is at most 3,840 pixels.
- Both edges are multiples of 16.
- The longer edge is no more than three times the shorter edge.
- Total pixels are between 655,360 and 8,294,400.
- Outputs above 3,686,400 total pixels are documented as experimental.

## Quality selection

- Keep the same explicit quality setting when first comparing Flare and Sunburst.
- If the output misses a requirement, test a higher setting before rewriting the entire prompt.
- After quality passes, test lower settings for acceptable quality and improved latency.
- Use `xhigh` or `max` only when they solve an unmet requirement within the latency budget. A higher label does not guarantee a better result for every request.

## Transparency and output format

- For real transparency, set `background="transparent"` and use PNG or WebP.
- Use `output_compression` only with JPEG or WebP, not PNG.
- Preserve the returned PNG or WebP alpha channel when saving or processing the image.
- Check the decoded alpha channel; a visually drawn checkerboard is not transparency.

## Documentation caveats

- The complete runnable example in the GPT Image 2.5 prompting guide remains pinned to `gpt-image-2`; use it only as an API workflow baseline and substitute a supported GPT Image 2.5 model deliberately.
- A note about omitting `input_fidelity` in one editing example explicitly names `gpt-image-2`. Do not turn that older-model note into a GPT Image 2.5 rule without current API documentation.
- Keep mutable prices and account-specific rate limits out of the skill. Link users to the current official documentation when they ask.
