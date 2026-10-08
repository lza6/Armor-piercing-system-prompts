## ADDED Requirements

### Requirement: Avatar Upload
The system SHALL allow users to upload a profile avatar image in JPEG, PNG, or WebP format, with a maximum file size of 5MB.

#### Scenario: Successful avatar upload
- **GIVEN** an authenticated user on the profile settings page
- **WHEN** the user selects a valid JPEG image under 5MB
- **THEN** the image is uploaded and stored
- **AND** the user's profile displays the new avatar

#### Scenario: Rejected oversized file
- **GIVEN** an authenticated user on the profile settings page
- **WHEN** the user selects an image larger than 5MB
- **THEN** the upload is rejected with a clear error message
- **AND** the existing avatar (or default) remains unchanged

### Requirement: Avatar Display
The system SHALL display user avatars in all contexts where user identity is shown, falling back to a generated default when no avatar has been uploaded.

#### Scenario: Avatar shown in comments
- **GIVEN** a user who has uploaded an avatar
- **WHEN** that user's comment is displayed in any thread
- **THEN** the avatar appears next to the comment author name
