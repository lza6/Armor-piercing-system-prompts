---
name: pierre-guard
description: Guard against breaking the @pierre/diffs integration in Plannotator's code review UI. Use this skill whenever modifying DiffViewer.tsx, upgrading the @pierre/diffs package, changing unsafeCSS injection, adding new props to FileDiff, or touching shadow DOM selectors or CSS variables that cross into Pierre's shadow boundary. Also trigger when someone asks "will this break the diff viewer", "is this safe to change", or when reviewing PRs that touch the review-editor package.
---

# Pierre Integration Guard

Plannotator's code review UI wraps `@pierre/diffs` — an open-source diff renderer that uses Shadow DOM. The integration is concentrated in a single file but relies on undocumented internals (shadow DOM selectors, CSS variable names, grid layout assumptions). This skill helps verify changes don't break that contract.

## Source of Truth

- **Upstream repo**: https://github.com/pierrecomputer/pierre/tree/main/packages/diffs
- **Local types**: `node_modules/@pierre/diffs/dist/` (`.d.ts` files)
- **Integration point**: `packages/review-editor/components/DiffViewer.tsx`
- **Current version**: check `packages/review-editor/package.json` for the pinned version

Always verify against the upstream repo or local `.d.ts` files — don't rely on memory of the API shape.

## What We Import

```typescript
import { FileDiff } from '@pierre/diffs/react';
import { getSingularPatch, processFile } from '@pierre/diffs';
```

These are the only three imports. `DiffViewer.tsx` is the only file that touches Pierre.

## API Surface to Guard

### 1. Component Props (`FileDiff`)

Read the current prop types from `node_modules/@pierre/diffs/dist/react/index.d.ts` or the upstream source. The props we use:

| Prop | Type | Notes |
|------|------|-------|
| `fileDiff` | `FileDiffMetadata` | From `getSingularPatch()` or `processFile()` |
| `options` | `FileDiffOptions<T>` | See options table below |
| `lineAnnotations` | `DiffLineAnnotation<T>[]` | `{ side, lineNumber, metadata }` |
| `selectedLines` | `SelectedLineRange \| null` | `{ start, end, side }` |
| `renderAnnotation` | `(ann) => ReactNode` | Custom inline annotation renderer |
| `renderHoverUtility` | `(getHoveredLine) => ReactNode` | The `+` button on hover (deprecated upstream — watch for removal) |

### 2. Options Object

| Option | Value We Pass | Risk |
|--------|--------------|------|
| `themeType` | `'dark' \| 'light'` | Low — standard enum |
| `unsafeCSS` | CSS string | **High** — targets internal selectors |
| `diffStyle` | `'split' \| 'unified'` | Low — standard enum |
| `diffIndicators` | `'bars'` | Low |
| `hunkSeparators` | `'line-info'` | Low |
| `enableLineSelection` | `true` | Low |
| `enableHoverUtility` | `true` | Medium — deprecated prop |
| `onLineSelectionEnd` | callback | Medium — signature could change |

### 3. Shadow DOM Selectors (via `unsafeCSS`)

These are the selectors we inject CSS rules against. They target `data-*` attributes inside Pierre's shadow DOM. If Pierre renames or removes any of these, our styling breaks silently.

**Currently used:**
- `:host` — shadow root
- `[data-diff]` — root diff container
- `[data-file]` — file wrapper
- `[data-diffs-header]` — header bar
- `[data-error-wrapper]` — error display
- `[data-virtualizer-buffer]` — virtual scroll buffer
- `[data-file-info]` — file metadata row
- `[data-column-number]` — line number gutter
- `[data-diffs-header] [data-title]` — title (we hide it)
- `[data-diff-type='split']` — split layout mode
- `[data-overflow='scroll']` / `[data-overflow='wrap']` — overflow mode

### 4. CSS Variables We Override

We override these `--diffs-*` variables to theme Pierre:

- `--diffs-bg`, `--diffs-fg` — base colors
- `--diffs-dark-bg`, `--diffs-light-bg` — theme-specific backgrounds
- `--diffs-dark`, `--diffs-light` — theme-specific foregrounds

### 5. CSS Variables We Inject (Custom)

We set these on a wrapper div outside the shadow DOM, relying on CSS custom property inheritance:

- `--split-left`, `--split-right` — control the split pane grid ratio

The `unsafeCSS` grid override references these: `grid-template-columns: var(--split-left, 1fr) var(--split-right, 1fr)`. The `1fr` fallback ensures the layout is safe if the variables aren't set.

### 6. Grid Layout Assumption

Pierre's split view uses CSS Grid with `grid-template-columns: 1fr 1fr`. We override this for the resizable split pane. If Pierre changes its layout engine (e.g., to flexbox or a different grid structure), the override will stop working.

**How to verify:** In the upstream source, search for `grid-template-columns` in the diff component styles.

## Verification Checklist

When reviewing changes that touch the Pierre integration, check:

### Props & Types
- [ ] Read the current `.d.ts` files to confirm prop names and types haven't changed
- [ ] Check if `renderHoverUtility` is still supported (it's deprecated — may be removed)
- [ ] Verify `DiffLineAnnotation` still uses `side: 'deletions' | 'additions'` (not `'old' | 'new'`)
- [ ] Confirm `SelectedLineRange` shape: `{ start, end, side? }`

### Shadow DOM Selectors
- [ ] Grep the upstream source for each `data-*` attribute we target in `unsafeCSS`
- [ ] If upgrading the package version, diff the old and new CSS/HTML output for renamed attributes
- [ ] Test both `split` and `unified` views — selectors are layout-dependent

### CSS Variables
- [ ] Grep upstream for `--diffs-bg`, `--diffs-fg`, and other variables we override
- [ ] Verify the variable names haven't been renamed or removed
- [ ] Check that `!important` is still needed (Pierre may change specificity)

### Theme Compliance
- [ ] New UI elements must use theme tokens (`bg-border`, `bg-primary`, etc.), not hardcoded colors like `bg-blue-500`
- [ ] The existing `ResizeHandle` component in `packages/ui/components/ResizeHandle.tsx` sets the visual convention — match it

### Build & Runtime
- [ ] Run `bun run dev:review` and verify the diff renders in both split and unified modes
- [ ] Check the browser console for Pierre warnings (e.g., `parseLineType: Invalid firstChar`)
- [ ] Test with add-only and delete-only files (Pierre doesn't render split grid for these)
- [ ] If changing UI code, remember build order: `bun run --cwd apps/review build && bun run build:hook`

## When Upgrading @pierre/diffs

1. Check the upstream changelog / commit history at https://github.com/pierrecomputer/pierre
2. Diff the `.d.ts` files between old and new versions:
   ```bash
   # Before upgrading, snapshot current types
   cp -r node_modules/@pierre/diffs/dist /tmp/pierre-old
   # After upgrading
   diff -r /tmp/pierre-old node_modules/@pierre/diffs/dist
   ```
3. Search for renamed/removed data attributes in the new version
4. Run through the full verification checklist above
5. Test the resizable split pane — it depends on grid layout internals
