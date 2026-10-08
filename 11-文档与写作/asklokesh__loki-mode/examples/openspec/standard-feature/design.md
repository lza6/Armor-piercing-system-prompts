## Context

The application currently uses hardcoded color values in CSS stylesheets. There is no theming infrastructure. The settings page has three sections (Account, Notifications, Privacy) arranged in a vertical layout.

## Goals / Non-Goals

**Goals:**
- Enable runtime theme switching with no page reload
- Respect OS-level color scheme preference
- Maintain accessible contrast ratios in both themes
- Keep the theming system simple and extensible

**Non-Goals:**
- Custom user-defined color themes (future consideration)
- Per-component theme overrides
- Server-side theme rendering

## Decisions

### Decision: React Context over Redux for Theme State
Theme state is a simple binary toggle (light/dark). React Context avoids adding a Redux dependency for such minimal state. The ThemeProvider wraps the app root and exposes current theme plus a toggle function.

### Decision: CSS Custom Properties over CSS-in-JS
CSS custom properties (variables) allow runtime theme switching by changing values on the document root element. This avoids the bundle size and runtime cost of CSS-in-JS libraries and works with the existing stylesheet architecture.

## Risks / Trade-offs

- **Risk:** Some third-party components may not support CSS variable theming and will need wrapper styles or overrides.
- **Trade-off:** Using localStorage for persistence means theme preference is per-device, not synced across devices. This is acceptable for the initial implementation.
- **Trade-off:** CSS custom properties are not supported in IE11, but IE11 is not in our browser support matrix.
