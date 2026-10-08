# Maintain Skill

Periodic brain health checks and cleanup.

## Workflow

1. **Run health check.** Check gbrain health to get the dashboard.
2. **Check each dimension:**

### Stale pages
Pages where compiled_truth is older than the latest timeline entry. The assessment hasn't been updated to reflect recent evidence.
- Check the health output for stale page count
- For each stale page: read the page from gbrain, review timeline, determine if compiled_truth needs rewriting

### Orphan pages
Pages with zero inbound links. Nobody references them.
- Review orphans: are they genuinely isolated or just missing links?
- Add links in gbrain from related pages or flag for deletion

### Dead links
Links pointing to pages that don't exist.
- Remove dead links in gbrain

### Missing cross-references
Pages that mention entity names but don't have formal links.
- Read compiled_truth from gbrain, extract entity mentions, create links in gbrain

### Tag consistency
Inconsistent tagging (e.g., "vc" vs "venture-capital", "ai" vs "artificial-intelligence").
- Standardize to the most common variant using gbrain tag operations

### Embedding freshness
Chunks without embeddings, or chunks embedded with an old model.
- For large embedding refreshes (>1000 chunks), use nohup:
  `nohup gbrain embed refresh > /tmp/gbrain-embed.log 2>&1 &`
- Then check progress: `tail -1 /tmp/gbrain-embed.log`

### Security (RLS verification)
Run `gbrain doctor --json` and check the RLS status.
All tables should show RLS enabled. If not, run `gbrain init` again.

### Schema health
Check that the schema version is up to date. `gbrain doctor --json` reports
the current version vs expected. If behind, `gbrain init` runs migrations
automatically.

### Open threads
Timeline items older than 30 days with unresolved action items.
- Flag for review

## Heartbeat Integration

For production agents running on a schedule, integrate gbrain health checks into
your operational heartbeat.

### On every heartbeat (hourly or per-session)

Run `gbrain doctor --json` and check for degradation. Report any failing checks
to the user. Key signals: connection health, schema version, RLS status, embedding
staleness.

### Weekly maintenance

Run `gbrain embed --stale` to refresh embeddings for pages that have changed since
their last embedding. For large brains (>5000 pages), run this with nohup:
```bash
nohup gbrain embed --stale > /tmp/gbrain-embed.log 2>&1 &
```

### Daily verification

Verify sync is running: check `gbrain stats` and confirm `last_sync` is within
the last 24 hours. If sync has stopped, the brain is drifting from the repo.

### Stale compiled truth detection

Flag pages where compiled truth is >30 days old but the timeline has recent entries.
This means new evidence exists that hasn't been synthesized. These pages need a
compiled truth rewrite (see the maintain workflow above).

## Quality Rules

- Never delete pages without confirmation
- Log all changes via timeline entries
- Check gbrain health before and after to show improvement

## Tools Used

- Check gbrain health (get_health)
- List pages in gbrain with filters (list_pages)
- Read a page from gbrain (get_page)
- Check backlinks in gbrain (get_backlinks)
- Link entities in gbrain (add_link)
- Remove links in gbrain (remove_link)
- Tag a page in gbrain (add_tag)
- Remove a tag in gbrain (remove_tag)
- View timeline in gbrain (get_timeline)
