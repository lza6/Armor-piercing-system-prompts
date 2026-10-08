# Enrich Skill

Enrich person and company pages from external APIs.

## Sources

| Source | Data | API |
|--------|------|-----|
| Crustdata | LinkedIn profiles, company data | REST API |
| Happenstance | Career history, connections | REST API |
| Exa | Web mentions, articles | REST API |

Note: enrichment requires separate API credentials for each service. No client
integrations ship in v1. This skill guides the agent to make API calls directly.

## Workflow

1. **Select target pages.** List person or company pages in gbrain.
2. **For each page:**
   - Read the page from gbrain to understand what we already know
   - Call external APIs for fresh data
   - Store raw API responses in gbrain (put_raw_data) to preserve provenance
   - Distill highlights into compiled_truth updates
   - Store the updated page in gbrain
3. **Validation rules:**
   - Connection count < 20 on LinkedIn = likely wrong person, skip
   - Name mismatch between brain and API = skip, flag for manual review
   - Don't overwrite human-written assessments with API boilerplate

## Quality Rules

- Raw data goes to gbrain's raw_data store (preserves provenance)
- Only distilled, useful info goes to compiled_truth
- Always add a timeline entry in gbrain: "Enriched from [source] on [date]"
- Don't enrich the same page more than once per week unless requested
- Rate limit: respect API rate limits, use exponential backoff

## Tools Used

- Read a page from gbrain (get_page)
- Store/update a page in gbrain (put_page)
- Add a timeline entry in gbrain (add_timeline_entry)
- List pages in gbrain by type (list_pages)
- Store raw API data in gbrain (put_raw_data)
- Retrieve raw data from gbrain (get_raw_data)
