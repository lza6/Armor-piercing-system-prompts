# Migrate Skill

Universal migration from any wiki, note tool, or brain system into GBrain.

## Supported Sources

| Source | Format | Strategy |
|--------|--------|----------|
| Obsidian | Markdown + `[[wikilinks]]` | Direct import, convert wikilinks to gbrain links |
| Notion | Exported markdown or CSV | Parse Notion's export structure |
| Logseq | Markdown with `((block refs))` | Convert block refs to page links |
| Plain markdown | Any .md directory | Import directory into gbrain directly |
| CSV | Tabular data | Map columns to frontmatter fields |
| JSON | Structured data | Map keys to page fields |
| Roam | JSON export | Convert block structure to pages |

## General Workflow

1. **Assess the source.** What format? How many files? What structure?
2. **Plan the mapping.** How do source fields map to gbrain fields (type, title, tags, compiled_truth, timeline)?
3. **Test with a sample.** Import 5-10 files, verify by reading them back from gbrain and exporting.
4. **Bulk import.** Import the full directory into gbrain.
5. **Verify.** Check gbrain health and statistics, spot-check pages.
6. **Build links.** Extract cross-references from content and create typed links in gbrain.

## Obsidian Migration

1. Import the vault directory into gbrain (Obsidian vaults are markdown directories)
2. Convert `[[wikilinks]]` to gbrain links:
   - Read each page from gbrain
   - For each `[[Name]]` found, resolve to a slug and create a link in gbrain
   - `[[Name|alias]]` uses the alias for context

Obsidian-specific:
- Tags (`#tag`) become gbrain tags
- Frontmatter properties map to gbrain frontmatter
- Attachments (images, PDFs) are noted but handled separately via file storage

## Notion Migration

1. Export from Notion: Settings > Export > Markdown & CSV
2. Notion exports nested directories with UUIDs in filenames
3. Strip UUIDs from filenames for clean slugs
4. Map Notion's database properties to frontmatter
5. Import the cleaned directory into gbrain

## CSV Migration

For tabular data (e.g., CRM exports, contact lists):
1. For each row in the CSV, create a page with column values as frontmatter
2. Use a designated column as the slug (e.g., name)
3. Use another column as compiled_truth (e.g., notes)
4. Store each page in gbrain

## Verification

After any migration:
1. Check gbrain statistics to verify page count matches source
2. Check gbrain health for orphans and missing embeddings
3. Export pages from gbrain for round-trip verification
4. Spot-check 5-10 pages by reading them from gbrain
5. Test search: search gbrain for "someone you know is in the data"

## Tools Used

- Store/update pages in gbrain (put_page)
- Read pages from gbrain (get_page)
- Link entities in gbrain (add_link)
- Tag pages in gbrain (add_tag)
- Get gbrain statistics (get_stats)
- Check gbrain health (get_health)
- Search gbrain (query)
