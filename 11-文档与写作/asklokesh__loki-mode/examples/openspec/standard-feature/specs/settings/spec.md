## MODIFIED Requirements

### Requirement: Settings Page Layout
The settings page SHALL include an "Appearance" section containing the theme toggle, positioned before the "Notifications" section. Previously the settings page contained only Account, Notifications, and Privacy sections.

#### Scenario: Appearance section visible
- **GIVEN** an authenticated user
- **WHEN** the user navigates to the settings page
- **THEN** the Appearance section is visible between Account and Notifications
- **AND** the section contains the theme toggle control
