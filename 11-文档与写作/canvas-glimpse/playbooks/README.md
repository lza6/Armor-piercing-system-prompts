# Canvas authoring playbooks

You're about to publish something to the Glimpse canvas. **Pick the playbook that
matches what you're showing, read it, then author from it.** Each playbook is
short on purpose: a crisp *use when*, a *recipe*, and a *minimal snippet* you can
copy and adapt. One artifact often combines several (a plan that embeds a diagram
and a comparison) — open every playbook that applies before you write HTML.

These are authoring guidance, not runtime. Nothing here changes how `glimpse`
behaves — see the [canvas SKILL](../SKILL.md) for the actual CLI (`open`,
`publish`, `ask`, `reply`, `bridge`, …).

## The set

| Playbook | Use when |
|---|---|
| [diagram](diagram.md) | Show a flow, architecture, state machine, or relationship — anything better drawn than described. |
| [table](table.md) | Turn dense, same-shaped records into a scan-friendly surface. |
| [plan](plan.md) | Explain a technical or product plan **before** implementing it. |
| [code](code.md) | Render source, a diff, or before/after cleanly. |
| [input](input.md) | Collect a **structured decision** from the human (ties to `glimpse ask`). |
| [comparison](comparison.md) | Weigh options / tradeoffs, or show current vs. target. |
| [slides](slides.md) | Build a paced, one-idea-per-screen deck (only when asked for a deck). |

Not sure? Default to **diagram** for structure, **table** for records,
**plan** for "what I'm about to do", **comparison** for "which one", and
**input** the moment you need the human to *choose* rather than *read*.

## Design direction (the default look)

Match the existing `examples/` house style so the canvas feels like one product.
The starter [`base.html`](base.html) encodes all of it — **copy it and replace the
`<main>`.** The direction, in one breath:

- **Editorial, developer-tool calm.** A soft off-white (dark: near-black slate)
  page, white/elevated cards with a hairline border and a whisper of shadow, one
  **indigo** accent (`#4c5fd5` light / `#93a4ff` dark) used sparingly.
- **Signature:** a **monospace eyebrow** label above each section
  (`PLAYBOOK · DIAGRAM`) and thin accent rules — a nod to Glimpse's terminal
  origin. It's the one flourish; keep everything else quiet.
- **Reading measure.** Prose sits at ~680px; don't run text the full width.
  Wide blocks (tables, diagrams) may bleed to ~820px+.
- **Type:** system sans for body, system mono (`ui-monospace, "SF Mono", …`) for
  eyebrows/labels/code. No web-font dependency, so it renders instantly and
  identically offline.

If the artifact is *about a specific product's UI*, prefer that product's own
design system instead, so the artifact faithfully shows the real thing.

## First-class light **and** dark mode (required)

Every artifact must be fully legible and look intentional in **both** modes.
`base.html` does this correctly — reuse its mechanism:

- Define tokens on `:root` (light), override under
  `@media (prefers-color-scheme: dark)` **and** under `:root[data-theme="dark"]`
  so an explicit toggle beats the OS setting.
- Ship the small toggle button + script from `base.html` (persists via
  `localStorage`; works when the file is opened directly).
- Check **contrast, borders, code blocks, and any Mermaid/CDN-rendered content in
  both modes.** Mermaid bakes its theme at init — re-initialize and re-render on
  theme change (see [diagram.md](diagram.md); the example wires this up).
- `color-scheme: light dark` is set so native controls (scrollbars, form widgets)
  follow the theme too.

## Portability & overflow (non-negotiable)

Glimpse renders **plain HTML in a real Chrome**, and artifacts should open the
same way directly from disk. So:

- **Self-contained.** Inline your CSS/JS. CDN scripts (e.g. Mermaid) are fine —
  the canvas Chrome has internet — but nothing should require a build step or a
  sibling file.
- **No horizontal overflow at any nesting level.** This is the #1 way artifacts
  break. Defenses (all in `base.html`):
  - Flex/grid **children** need `min-width:0`; grid tracks use
    `minmax(min(240px,100%),1fr)`, never a fixed `minmax(240px,1fr)`.
  - Long unbreakable text — URLs, file paths, monospace IDs, badges — must
    `overflow-wrap:anywhere` (inline) or live in a scroll/`text-overflow:ellipsis`
    container (chips, cells).
  - Code blocks: `pre{overflow:auto;max-width:100%}`.
  - Tables: wrap in `.table-wrap{overflow-x:auto}` so a wide table scrolls itself
    instead of pushing the page.
- **Verify before you claim done.** `glimpse shot /tmp/out.png` then read the PNG,
  or open the file in Chrome. Check both themes and a narrow width.

## Recipe, every time

1. Pick the playbook(s). Read them.
2. `cp skills/canvas/playbooks/base.html /tmp/<slug>.html`; replace `<main>`.
3. Lead with the answer / the question the artifact resolves — not the mechanics.
4. Publish: `glimpse publish <slug> "Human Title" /tmp/<slug>.html`
   (or `glimpse ask …` for a decision — see [input.md](input.md)).
5. Screenshot both themes, confirm no overflow, then point the user at it in chat.

## Worked examples

Every playbook has a matching artifact under
[`examples/playbooks/`](../../../examples/playbooks/) you can open and crib from;
light + dark screenshots live in `examples/screenshots/`.
