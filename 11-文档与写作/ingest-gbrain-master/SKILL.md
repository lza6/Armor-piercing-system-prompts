# Ingest Skill

Ingest meetings, articles, documents, and conversations into the brain.

## Workflow

1. **Parse the source.** Extract people, companies, dates, and events from the input.
2. **For each entity mentioned:**
   - Read the entity's page from gbrain to check if it exists
   - If exists: update compiled_truth (rewrite State section with new info, don't append)
   - If new: store the page in gbrain with the appropriate type and slug
3. **Append to timeline.** Add a timeline entry in gbrain for each event, with date, summary, and source.
4. **Create cross-reference links.** Link entities in gbrain for every entity pair mentioned together, using the appropriate relationship type.
5. **Timeline merge.** The same event appears on ALL mentioned entities' timelines. If Alice met Bob at Acme Corp, the event goes on Alice's page, Bob's page, and Acme Corp's page.

## Entity Detection on Every Message

Production agents should detect entity mentions on EVERY inbound message. This is
the signal detection loop that makes the brain compound over time.

### Protocol

1. **Scan the message** for entity mentions: people, companies, concepts, original
   thinking. Fire on every message (no exceptions unless purely operational).
2. **For each entity detected:**
   - `gbrain search "name"` -- does a page already exist?
   - **If yes:** load context with `gbrain get <slug>`. Use the compiled truth to
     inform your response. Update the page if the message contains new information.
   - **If no:** assess notability. If the entity is worth tracking (will come up
     again, is relevant to the user's world), create a new page with
     `gbrain put <type/slug>` and populate with what you know.
3. **After creating or updating pages:** commit changes to the brain repo, then
   sync to gbrain:
   ```bash
   git add brain/ && git commit -m "update entity pages"
   gbrain sync --no-pull --no-embed
   ```
4. **Don't block the conversation.** Entity detection and enrichment should happen
   alongside the response, not before it. The user shouldn't wait for brain writes
   to get an answer.

### What counts as notable

- People the user interacts with or discusses (not random mentions)
- Companies relevant to the user's work, investments, or interests
- Concepts or frameworks the user references or creates
- The user's own original thinking (ideas, theses, observations) -- highest value

## Quality Rules

- Executive summary in compiled_truth must be updated, not just timeline appended
- State section is REWRITTEN, not appended to. Current best understanding only.
- Timeline entries are reverse-chronological (newest first)
- Every person/company mentioned gets a page if one doesn't exist
- Link types: knows, works_at, invested_in, founded, met_at, discussed
- Source attribution: every timeline entry includes the source (meeting, article, email, etc.)

## Tools Used

- Read a page from gbrain (get_page)
- Store/update a page in gbrain (put_page)
- Add a timeline entry in gbrain (add_timeline_entry)
- Link entities in gbrain (add_link)
- List tags for a page (get_tags)
- Tag a page in gbrain (add_tag)
