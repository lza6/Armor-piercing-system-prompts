# CSS `grid-template-areas`

The **`grid-template-areas`** CSS property specifies named grid areas, establishing the cells in the grid and assigning them names. These names can then be referenced by grid items using the `grid-area` property.

## Syntax

```css
/* Keyword value */
grid-template-areas: none;

/* String values */
grid-template-areas: "header header header"
                     "sidebar content content"
                     "footer footer footer";

/* With empty cells */
grid-template-areas: "header header header"
                     "sidebar . content"
                     "footer footer footer";

/* Single row */
grid-template-areas: "logo nav nav actions";

/* Global values */
grid-template-areas: inherit;
grid-template-areas: initial;
grid-template-areas: revert;
grid-template-areas: revert-layer;
grid-template-areas: unset;
```

### Values

**`none`**

The grid container does not define any named grid areas. Grid items can still be placed by line number or span.

**`<string>+`**

A string is created for every row in the grid. Each row string lists the cells in that row, separated by whitespace. Each cell name defines a named grid area that spans the corresponding grid cell. Multiple adjacent cells with the same name form a single rectangular area.

A period (`.`) represents an unnamed (empty) cell. Multiple periods without whitespace between them are treated as a single empty cell token.

## Formal Definition

| Property | Value |
|---|---|
| Initial value | `none` |
| Applies to | Grid containers |
| Inherited | No |
| Computed value | As specified |
| Animation type | Discrete |

## Formal Syntax

```
grid-template-areas =
  none |
  <string>+
```

## Rules and Constraints

Grid template areas must follow these rules. Violating any of them makes the entire declaration invalid:

1. **Rectangular areas only.** Every named area must form a rectangle. L-shapes, T-shapes, and other non-rectangular shapes are invalid.

2. **Contiguous cells.** All cells belonging to a named area must be adjacent. Disconnected regions sharing the same name are invalid.

3. **Consistent row lengths.** Every row string must define the same number of cells. Rows with different cell counts are invalid.

```css
/* VALID — "sidebar" forms a rectangle */
grid-template-areas:
  "header  header  header"
  "sidebar content content"
  "sidebar footer  footer";

/* INVALID — "sidebar" forms an L-shape */
grid-template-areas:
  "header  sidebar sidebar"
  "content content sidebar"
  "footer  footer  footer";
```

## Basic Example

A classic "holy grail" layout with header, sidebar, main content, and footer:

```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  min-height: 100vh;
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }
```

```html
<div class="container">
  <header class="header">Site Header</header>
  <aside class="sidebar">Navigation</aside>
  <main class="main">Page Content</main>
  <footer class="footer">Site Footer</footer>
</div>
```

The `grid-area` property on each child references the named area defined in `grid-template-areas`. The browser places each element into the cells that match its area name.

## Responsive Layout with Media Queries

Named grid areas make responsive layouts straightforward. Redefine the area template at each breakpoint without changing the HTML:

```css
.dashboard {
  display: grid;
  gap: 16px;
  padding: 16px;
}

/* Mobile: single column stack */
@media (max-width: 639px) {
  .dashboard {
    grid-template-columns: 1fr;
    grid-template-areas:
      "stats"
      "chart"
      "table"
      "filters";
  }
}

/* Tablet: two columns */
@media (min-width: 640px) and (max-width: 1023px) {
  .dashboard {
    grid-template-columns: 1fr 1fr;
    grid-template-areas:
      "stats   stats"
      "chart   filters"
      "table   table";
  }
}

/* Desktop: three columns */
@media (min-width: 1024px) {
  .dashboard {
    grid-template-columns: 250px 1fr 300px;
    grid-template-areas:
      "filters stats   stats"
      "filters chart   chart"
      "filters table   table";
  }
}

.stats   { grid-area: stats; }
.chart   { grid-area: chart; }
.table   { grid-area: table; }
.filters { grid-area: filters; }
```

## Using Empty Cells

The dot (`.`) character denotes an empty cell. This is useful for creating gaps or asymmetric layouts:

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(3, 200px);
  gap: 8px;
  grid-template-areas:
    "hero hero . sidebar"
    "hero hero . sidebar"
    "a    b    c sidebar";
}

.hero    { grid-area: hero; }
.sidebar { grid-area: sidebar; }
.item-a  { grid-area: a; }
.item-b  { grid-area: b; }
.item-c  { grid-area: c; }
```

The empty cells in column 3 of rows 1 and 2 create a visual gap between the hero image and the sidebar without using explicit margins.

## Combining with `grid-template`

The `grid-template` shorthand combines `grid-template-rows`, `grid-template-columns`, and `grid-template-areas` into a single declaration:

```css
.layout {
  display: grid;
  grid-template:
    "header header header" 60px
    "nav    main   aside"  1fr
    "footer footer footer" 48px
    / 200px 1fr    250px;
}
```

This is equivalent to:

```css
.layout {
  display: grid;
  grid-template-rows: 60px 1fr 48px;
  grid-template-columns: 200px 1fr 250px;
  grid-template-areas:
    "header header header"
    "nav    main   aside"
    "footer footer footer";
}
```

## Implicit Line Names

Defining a named area automatically creates named lines. For an area named `header`, the browser generates:

- `header-start` — the starting row and column line
- `header-end` — the ending row and column line

These implicit line names can be used in other grid placement properties:

```css
.overlay {
  grid-row: header-start / footer-end;
  grid-column: sidebar-start / main-end;
}
```

## Accessibility Considerations

The visual order defined by `grid-template-areas` does not change the document source order. Screen readers and keyboard navigation follow the DOM order, not the visual layout.

If the visual order differs significantly from the source order, users navigating with a keyboard may experience confusing focus jumps. Ensure the HTML source order matches a logical reading order, and use grid placement purely for visual arrangement.

## Common Mistakes

### Mistake 1: Non-Rectangular Areas

```css
/* This is INVALID */
grid-template-areas:
  "a a b"
  "a c c"
  "d d c";
/* Area "a" is L-shaped, area "c" is L-shaped */
```

### Mistake 2: Mismatched Row Lengths

```css
/* This is INVALID — row 2 has 2 cells, row 1 has 3 */
grid-template-areas:
  "header header header"
  "main sidebar";
```

### Mistake 3: Forgetting `display: grid`

Named areas only work on grid containers. Without `display: grid`, the `grid-template-areas` property has no effect.

## Browser Compatibility

| Browser | Version | Date |
|---|---|---|
| Chrome | 57+ | March 2017 |
| Firefox | 52+ | March 2017 |
| Safari | 10.1+ | March 2017 |
| Edge | 16+ | October 2017 |
| Opera | 44+ | March 2017 |
| Chrome Android | 57+ | March 2017 |
| Firefox Android | 52+ | March 2017 |
| Safari iOS | 10.3+ | March 2017 |
| Samsung Internet | 6.0+ | August 2017 |

The `grid-template-areas` property is supported in all modern browsers. There are no known interoperability issues across current engine versions.

## Specifications

| Specification | Status |
|---|---|
| [CSS Grid Layout Module Level 2 — grid-template-areas](https://drafts.csswg.org/css-grid/#propdef-grid-template-areas) | Working Draft |
| [CSS Grid Layout Module Level 1 — grid-template-areas](https://www.w3.org/TR/css-grid-1/#propdef-grid-template-areas) | W3C Recommendation |

## See Also

- [`grid-template-columns`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-columns) — defines column track sizes
- [`grid-template-rows`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-rows) — defines row track sizes
- [`grid-template`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template) — shorthand for rows, columns, and areas
- [`grid-area`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-area) — places an item into a named area
- [CSS Grid Layout Guide](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout) — comprehensive grid reference
- [A Complete Guide to CSS Grid](https://css-tricks.com/snippets/css/complete-guide-grid/) — visual guide with examples
