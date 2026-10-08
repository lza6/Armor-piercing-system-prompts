## Why

Users frequently work in low-light environments and have requested a dark mode option. Currently the application only supports a light theme, which causes eye strain during extended use and does not respect operating system appearance preferences. Dark mode is a standard expectation for modern web applications.

## What Changes

- Add a theme toggle allowing users to switch between light and dark modes
- Detect and respect the operating system color scheme preference on first visit
- Persist the user's theme preference across sessions
- Ensure all UI components meet WCAG 2.1 contrast ratios in both themes
- Update the settings page layout to accommodate the new theme section

## Capabilities

### New Capabilities
- `ui`: Theme switching, system preference detection, and dark mode rendering
- `accessibility`: Contrast ratio compliance for dark mode components

### Modified Capabilities
- `settings`: Settings page layout updated to include theme configuration section

## Impact

- All CSS must migrate to custom properties for dynamic theming
- Settings page gains a new "Appearance" section
- localStorage used for theme persistence (no backend changes)
- Component library needs dark variants for all interactive elements
