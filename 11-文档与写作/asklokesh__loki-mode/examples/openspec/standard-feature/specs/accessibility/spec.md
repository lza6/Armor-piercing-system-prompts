## ADDED Requirements

### Requirement: Dark Mode Contrast Ratios
All text and interactive elements in dark mode SHALL meet WCAG 2.1 AA contrast ratio requirements: at least 4.5:1 for normal text and 3:1 for large text and UI components.

#### Scenario: Body text contrast in dark mode
- **GIVEN** the application is in dark mode
- **WHEN** body text is rendered on the dark background
- **THEN** the contrast ratio between text and background is at least 4.5:1

#### Scenario: Button contrast in dark mode
- **GIVEN** the application is in dark mode
- **WHEN** a primary action button is rendered
- **THEN** the contrast ratio between button text and button background is at least 4.5:1
- **AND** the contrast ratio between button background and page background is at least 3:1
