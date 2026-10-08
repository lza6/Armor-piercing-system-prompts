## ADDED Requirements

### Requirement: Theme Toggle
The system SHALL provide a toggle control that allows users to switch between light and dark visual themes without requiring a page reload.

#### Scenario: User switches to dark mode
- **GIVEN** the application is displaying in light mode
- **WHEN** the user activates the theme toggle
- **THEN** all UI elements transition to the dark color scheme
- **AND** the transition completes within 200ms

#### Scenario: User switches back to light mode
- **GIVEN** the application is displaying in dark mode
- **WHEN** the user activates the theme toggle
- **THEN** all UI elements revert to the light color scheme

### Requirement: System Preference Detection
The system SHALL detect the operating system color scheme preference on first visit and apply the matching theme automatically.

#### Scenario: System prefers dark mode
- **GIVEN** a new user whose operating system is set to dark mode
- **WHEN** the user visits the application for the first time
- **THEN** the application renders in dark mode
- **AND** no manual toggle action is required

#### Scenario: System preference changes while app is open
- **GIVEN** a user who has not manually set a theme preference
- **WHEN** the operating system color scheme changes
- **THEN** the application updates to match the new system preference

### Requirement: Theme Persistence
The system SHALL persist the user's theme preference in local storage so that it is restored on subsequent visits.

#### Scenario: Preference survives page reload
- **GIVEN** a user who has selected dark mode via the toggle
- **WHEN** the user reloads the page or returns in a new session
- **THEN** the application loads in dark mode
- **AND** the toggle reflects the saved preference
