# Design Reference

## Tech Stack Conflicts

These combinations produce silent failures or incoherent output. Never combine them:

| Never combine | Why |
|---|---|
| Tailwind + CSS Modules on the same element | Specificity conflicts, unpredictable cascade |
| Framer Motion + CSS transitions on the same element | Double-animating the same property causes jank |
| styled-components or emotion + Tailwind | Two competing class systems fighting for the same DOM node |
| Heroicons + Lucide + Font Awesome in one project | Visual inconsistency, size mismatches, bundle bloat |
| Multiple Google Font families as display fonts | Competing personalities cancel each other out |
| Glassmorphism backdrop-filter + solid `border: 1px solid` | Solid borders shatter the layered depth illusion |
| Dark background + `#ffffff` text at full opacity | Too harsh; use `rgba(255,255,255,0.85)` or `#f0f0f0` |
| Tailwind v4 `@theme` + dynamically constructed class names | `@theme` tokens generate utility classes JIT; if class names are built from variables or not present in scanned source, the class is purged and styles silently disappear. Fix: use static class names in source files, add to `safelist`, or define custom colors in `:root` + `extend.colors` in `tailwind.config.js` instead of `@theme` |

Before writing the first component, name the single CSS strategy for the project: Tailwind only, CSS Modules only, or CSS-in-JS only. Do not drift from it.

## Common Traps

Before submitting, check whether any of the following slipped in without intention:

- A purple or blue gradient over white as the hero background
- A three-part hero: large headline, one-line subtext, two CTA buttons side by side
- A grid of cards with identical rounded corners, identical drop shadows, identical padding
- A top navigation bar with logo left, links center, primary action far right
- Sections that alternate between white and `#f9f9f9`
- A centered icon or illustration sitting above a heading above a paragraph
- A four-column footer with equal-weight columns

Any of these can appear if they serve the design intentionally. They cannot appear by default.

Final test: if you swapped in completely different content and the layout still made sense without changes, you built a template, not a design. Redo it.

## Production Quality Baseline

Check before handoff. These are not aesthetic choices, they are non-negotiable.

> Treat the sections below as craft details, not defaults. Only apply them when they serve the locked visual direction. If removing a detail changes nothing about how the interface feels, leave it out.

### Accessibility
- Icon-only buttons need `aria-label`
- Actions use `<button>`, navigation uses `<a>` (not `<div onClick>`)
- Images need `alt` (or `alt=""` if decorative)
- Visible focus states: `focus-visible:ring-*` or equivalent; never `outline: none` without replacement

### Animation
- Honor `prefers-reduced-motion`: disable or reduce animations when set
- Animate `transform`/`opacity` only (compositor-friendly, no layout thrash)
- Never `transition: all`; list properties explicitly
- Interruptible animations: prefer CSS transitions for interactive state changes (hover, toggle, open/close) because they retarget mid-animation; reserve keyframe animations for staged sequences that run once (e.g., staggered page enters)
- Staggered enter: split content into semantic chunks with ~100ms delay; titles into words at ~80ms; typical enter uses `opacity: 0 → 1`, `translateY(12px) → 0`, and `blur(4px) → 0`
- Subtle exit: use a small fixed `translateY(-12px)` instead of full height; keep duration ~150ms `ease-in`, shorter and softer than enter
- Contextual icon swaps: animate with `scale: 0.25 → 1`, `opacity: 0 → 1`, and `blur: 4px → 0px`. With a spring library: `{ type: "spring", duration: 0.3, bounce: 0 }`. Without: keep both icons in DOM (one absolute) and cross-fade with CSS using `cubic-bezier(0.2, 0, 0, 1)`
- Scale on press: buttons use `scale(0.96)` on active/press via CSS transitions so the press can be interrupted; add a `static` prop to disable when motion would be distracting
- Page-load guard: use `initial={false}` on animated presence wrappers for toggles, tabs, and icon swaps to prevent enter animations on first render; do not use it for intentional page-load entrance sequences

### Performance
- Transition specificity: never `transition: all`; list exact properties (e.g., `transition-property: scale, opacity`). Tailwind's `transition-transform` covers `transform, translate, scale, rotate`; use `transition-[scale,opacity,filter]` for mixed properties
- GPU compositing: only use `will-change` for `transform`, `opacity`, or `filter`. Never `will-change: all`. Add only when you notice first-frame stutter; do not apply preemptively to every element
- Images: explicit `width` and `height` (prevents layout shift)
- Below-fold images: `loading="lazy"`
- Critical fonts: `font-display: swap`

### Touch and Mobile
- `touch-action: manipulation` (prevents double-tap zoom delay)
- Full-bleed layouts: `env(safe-area-inset-*)` for notch devices
- Modals and drawers: `overscroll-behavior: contain`
- Hover guard: wrap interactive hover states with `@media(hover:hover)` so they only apply on pointer devices, not touch screens. Tailwind: `[@media(hover:hover)]:hover:bg-...`. Without this, a tapped element on mobile gets a permanent hover state until the next tap elsewhere.

### Typography Details
- Text wrapping: `text-wrap: balance` on headings and short text blocks (≤6 lines in Chromium, ≤10 in Firefox); `text-wrap: pretty` on body paragraphs and longer text; leave default on code blocks and pre-formatted text
- Font smoothing: apply `-webkit-font-smoothing: antialiased; -moz-osx-font-smoothing: grayscale` once on the root layout (macOS only)
- Tabular numbers: use `font-variant-numeric: tabular-nums` for counters, timers, prices, number columns, or any dynamically updating numbers
- Letter-spacing scales with font size: display type needs negative tracking to look engineered rather than stretched. Two tiers: roughly -0.022em for display sizes (32px and above), -0.012em for mid-range (20–28px), normal at 16px and below. Apply to any display-weight typeface, not just geometric sans. Positive letter-spacing on large headlines is always wrong.

### Surfaces
- Concentric border radius: calculate `outerRadius = innerRadius + padding` so nested rounded corners feel intentional, not mechanical; if padding exceeds `24px`, treat layers as separate surfaces and choose each radius independently
- Optical alignment: nudge icons by eye, not just by math, so buttons feel centered; buttons with text and an icon use slightly less padding on the icon side (e.g., `pl-4 pr-3.5`); play triangles and asymmetric icons should shift `1px`-`2px` toward the heavier side, or fix the SVG directly
- Shadows over borders: use layered `box-shadow` for depth on cards, buttons, and elevated elements so the surface feels lifted, not fenced in; reserve actual `border` for dividers, table cells, and layout separation (applies primarily to light mode; on dark surfaces see the dark-mode surface hierarchy rule below)
- Image outlines: add a subtle inset outline so images hold their own depth without altering layout dimensions: `outline: 1px solid rgba(0,0,0,0.1); outline-offset: -1px` (light) or `outline: 1px solid rgba(255,255,255,0.1); outline-offset: -1px` (dark)
- Minimum hit area: keep every interactive target at least 40×40px so even small controls feel generous and precise; extend with a centered pseudo-element when the visible element is smaller, and never let hit areas of two interactive elements overlap
- Light-mode app surface hierarchy: adjacent nested surfaces must be visually distinguishable. Minimum: background-color step of at least 4% lightness between sidebar and main area, and between main area and cards; or a shadow of at least `0 1px 3px rgba(0,0,0,0.10)` on elevated cards. A white card on a near-white background with `box-shadow: 0 1px 2px rgba(0,0,0,0.05)` is invisible -- that is not depth, it is noise.
- Dark-mode surface hierarchy: the page canvas is a near-black solid (e.g. `#08090a`). Elevation is communicated by adding semi-transparent white overlays on top of that canvas: cards at `rgba(255,255,255,0.02)`, elevated surfaces at `0.04`, prominent panels at `0.05`. Borders follow the same logic: `rgba(255,255,255,0.05)` for subtle, `0.08` for standard. Traditional drop shadows (dark on dark) are nearly invisible; luminance stepping through background opacity is the primary depth cue on dark surfaces.
- Border radius system: define a named radius scale during direction lock instead of picking values ad-hoc. A minimal scale is 3–4 tiers (e.g. `{4px, 8px, 12px, pill}`); a richer system might run 6–8 tiers. The point is committing to a named set before the first component so that all surfaces speak the same spatial language — not covering every possible radius value.

## Reflex Fonts to Reject

LLMs default to these because they dominate training data. Using them signals "no decision was made." Pick from foundries with a clear voice instead. The ban is on reflex use as a display face; informed product-UI use (e.g. Inter for a dense data table) is allowed when justified.

Reject: Inter, DM Sans, DM Serif Display, DM Serif Text, Outfit, Plus Jakarta Sans, Instrument Sans, Instrument Serif, Space Grotesk, Space Mono, IBM Plex Sans, IBM Plex Serif, IBM Plex Mono, Syne, Fraunces, Newsreader, Lora, Crimson Pro, Crimson Text, Playfair Display, Cormorant, Cormorant Garamond.

## Font Selection Procedure

1. Write three words that describe the brand (e.g. "precise, minimal, fast").
2. Name the three fonts you would reach for reflexively.
3. Reject all three.
4. Pick a typeface from a named foundry (Klim, Commercial Type, Colophon, Grilli Type, OH no Type, Village, etc.) or an open-source option with a clear personality that matches the brand words. Be able to explain why that specific typeface in one sentence.

## Color System: OKLCH Rules

- Use OKLCH instead of HSL. OKLCH is perceptually uniform: equal numeric changes produce equal perceived changes across the spectrum.
- Reduce chroma as lightness approaches the extremes. At 85% lightness a chroma around 0.08 is enough; pushing to 0.15 looks garish. At 15% lightness, tighten chroma similarly.
- Tint neutrals toward the brand hue with a chroma of 0.005 to 0.01. Even this faint amount is perceptible and creates subconscious cohesion.
- 60-30-10 is about visual weight, not pixel count. 60% neutral/surface, 30% secondary text and borders, 10% accent.
- Never use gray text on a colored background. Use a shade of the background hue at reduced lightness instead.

## Theme Matrix

Choose light or dark deliberately based on audience and context. Neither is a default.

| Context | Direction | Reason |
|---|---|---|
| Trading or analytics dashboard, night-shift use | Dark | High data density; reduced glare during long sessions |
| Children's reading or learning app | Light | Welcoming, low fatigue for eyes still developing contrast sensitivity |
| Enterprise SRE or observability tool | Dark | Operator context; dark surfaces read at a glance in low-light NOC rooms |
| Weekend planning, recipes, journaling | Light | Ambient daytime use; light feels casual and approachable |
| Music player or media browser | Dark | Content-forward; dark surfaces recede and let media pop |
| Hospital or clinical patient portal | Light | Trust and legibility are paramount; clinical associations favor light |
| Vintage or artisanal brand site | Cream/warm light | Dark would clash with the analog material references |

If the answer is not obvious from the context, default to light. If the user's context implies both modes, ship light first and layer dark-mode tokens on top.

## Absolute Bans (CSS-Pattern Level)

These patterns appear in the majority of AI-generated interfaces. Each one has a specific rewrite.

| Pattern | Why | Rewrite |
|---|---|---|
| `border-left` or `border-right` wider than 1px as a section accent | The single most overused "design touch" in admin and dashboard UIs; it looks like a mistake at anything beyond a hairline divider | Change element structure: use a colored dot, a short horizontal rule, a background swatch, or a typographic weight shift instead |
| `background-clip: text` gradient text | Decorative rather than meaningful; one of the top AI design tells; illegible when printed or in high-contrast mode | Use a solid brand color, a tinted neutral, or typographic weight for emphasis |
| `backdrop-filter: blur` glassmorphism as the default card surface | Expensive on low-power devices; overused; the layered-depth illusion breaks with a solid border | Use elevated surfaces via background color steps and `box-shadow` instead |
| Purple-to-blue gradients or cyan-on-dark accent systems | The canonical "AI design" color palette; communicates nothing about the brand | Pick a palette from the brand words via the OKLCH rules above |
| Generic rounded-rect card with `box-shadow` as the default container | Template thinking; applies the same container to every content type regardless of hierarchy | Default to cardless sections; only add card treatment when the content type requires it |
| Modals as a lazy escape for overflow UI | Interrupts flow and breaks browser back navigation; used when an inline expansion, drawer, or separate page would be better | Inline expand, detail panel, or dedicated route; modals only when the action truly requires focus-lock |
| `transition: all` or animating width/height/padding/margin | Forces the browser into layout recalculation on every frame | List exact properties (`transition-property: transform, opacity`); use `grid-template-rows: 0fr to 1fr` for height reveals |

## Motion Specifics

Complements the motion timing in the main SKILL.md constraints.

- No bounce or elastic easing. Real objects decelerate smoothly. Use exponential ease-out (`ease-out-quart`, `ease-out-quint`, or `cubic-bezier(0.16,1,0.3,1)`) for natural, high-quality deceleration.
- Animate `transform` and `opacity` only. Every other property triggers layout or paint.
- For height reveals, use `grid-template-rows: 0fr` to `1fr` transitions instead of animating `height` directly. It avoids the `height: auto` animation trap.
- Icon swaps: use a 120ms cross-fade with `opacity` and a subtle `scale(0.9)` to `scale(1)`. No rotation unless rotation is semantically meaningful (e.g. a chevron indicating direction change).
- Do not use `transition: all` even as a quick prototype shortcut. It animates layout, color, and font-size simultaneously, causing visible jank.

## DESIGN.md Scaffold (Optional, Production UIs)

For a multi-page or production UI, emit a short `DESIGN.md`-style summary before writing the first component. This forces enumeration of decisions that would otherwise be left implicit and lets the user correct direction early. The nine sections:

1. **Visual Theme and Atmosphere** - mood, density, design philosophy in 2-3 sentences
2. **Color Palette and Roles** - semantic name + value + functional role for each color token
3. **Typography Rules** - font family, size scale, weight scale, line-height, letter-spacing; a table if more than 4 levels
4. **Component Stylings** - buttons (all states), cards (if used), inputs, navigation; describe each with states (default, hover, active, disabled)
5. **Layout Principles** - spacing scale, grid columns, whitespace philosophy
6. **Depth and Elevation** - shadow system or background-color-step system; describe each level
7. **Do's and Don'ts** - 5 to 10 guardrails specific to this project, not generic rules
8. **Responsive Behavior** - breakpoints, how navigation collapses, touch target minimums
9. **Agent Prompt Guide** - a quick color reference (name: value pairs) + 3 to 5 example component prompts ready to paste into a follow-up request. Prompts must be specific enough to execute without further lookup: every value, every radius, every letter-spacing, every weight inlined. Example standard (values are illustrative, use the project's own tokens): "Create a hero on `{bg-canvas}`, headline at 48px weight 600, line-height 1.00, letter-spacing -0.022em, color `{text-primary}`, CTA at `{accent}` with `{btn-radius}` radius" — that level of specificity, not "hero with primary color and CTA button"

For a single component or quick prototype, skip this. The three-line thesis in SKILL.md is sufficient.

## AI Slop Test

Would a stranger glancing at the first viewport say "an AI made this" immediately? If yes, the committed direction was not committed enough. The usual culprits: reflex font, default purple accent, centered hero with generic card grid beneath. Fix the typography, the color system, or the layout until the answer flips.

---

*Rules in Reflex Fonts, Font Selection, OKLCH, Theme Matrix, Absolute Bans, Motion Specifics, and AI Slop Test adapted from [pbakaus/impeccable](https://github.com/pbakaus/impeccable) (Apache 2.0). DESIGN.md Scaffold adapted from [getdesign.md](https://getdesign.md) (MIT); concept credited to Google Stitch.*
