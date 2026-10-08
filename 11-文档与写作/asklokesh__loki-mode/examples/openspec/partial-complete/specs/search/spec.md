## ADDED Requirements

### Requirement: Full-Text Search
The system SHALL provide full-text search across all user content, returning results ranked by relevance with query term highlighting in the result snippets.

#### Scenario: Basic keyword search
- **GIVEN** a user with 200 content items, 15 of which contain the word "deployment"
- **WHEN** the user searches for "deployment"
- **THEN** the 15 matching items are returned, ranked by relevance
- **AND** the search term is highlighted in each result snippet

#### Scenario: No results found
- **GIVEN** a user searching for a term that matches no content
- **WHEN** the search query is submitted
- **THEN** an empty result set is returned with a clear "no results" message
- **AND** search suggestions are offered based on similar terms

### Requirement: Search Filters
The system SHALL support filtering search results by content type, date range, and author. Filters MUST be combinable and update the result count in real time.

#### Scenario: Filter by content type and date
- **GIVEN** a search for "quarterly report"
- **WHEN** the user applies filters for content type "document" and date range "last 30 days"
- **THEN** only documents created in the last 30 days matching "quarterly report" are returned
- **AND** the result count updates to reflect the applied filters
