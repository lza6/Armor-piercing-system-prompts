## Why

Users have no way to personalize their profiles with a photo. This leads to a generic experience where users cannot visually identify each other in comments, reviews, or team views. Avatar support is one of the most requested features in our feedback tracker.

## What Changes

- Add avatar upload capability to user profiles
- Support image validation (format, size) on upload
- Display avatars across the application where user identity is shown

## Capabilities

### New Capabilities
- `users`: Avatar upload, validation, and display for user profiles

### Modified Capabilities

## Impact

- User profile API endpoints gain upload/retrieve avatar routes
- Frontend profile page needs upload widget
- Comment and team views need avatar display components
- Storage backend needs image file handling
