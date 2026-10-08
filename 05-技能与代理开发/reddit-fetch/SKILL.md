---
name: reddit-fetch
description: Fetch content from Reddit using Gemini CLI or curl JSON API fallback. Use when accessing Reddit URLs, researching topics on Reddit, or when Reddit returns 403/blocked errors.
---

# Reddit Fetch

## Method 1: Gemini CLI (Try First)

Use Gemini CLI via tmux. It can browse, summarize, and answer complex questions about Reddit content.

Pick a unique session name (e.g., `gemini_abc123`) and use it consistently throughout.

### Setup

```bash
tmux new-session -d -s <session_name> -x 200 -y 50
tmux send-keys -t <session_name> 'gemini -m gemini-3-pro-preview' Enter
sleep 3  # wait for Gemini CLI to load
```

### Send query and capture output

```bash
tmux send-keys -t <session_name> 'Your Reddit query here' Enter
sleep 30  # wait for response (adjust as needed, up to 90s for complex searches)
tmux capture-pane -t <session_name> -p -S -500  # capture output
```

If the captured output shows an API error (e.g., quota exceeded, model unavailable), kill the session and retry without the `-m` flag (just `gemini` with no model argument). This falls back to the default model.

### How to tell if Enter was sent

Look for YOUR QUERY TEXT specifically. Is it inside or outside the bordered box?

**Enter NOT sent** - your query is INSIDE the box:
```
╭─────────────────────────────────────╮
│ > Your actual query text here       │
╰─────────────────────────────────────╯
```

**Enter WAS sent** - your query is OUTSIDE the box, followed by activity:
```
> Your actual query text here

⠋ Our hamsters are working... (processing)

╭────────────────────────────────────────────╮
│ >   Type your message or @path/to/file     │
╰────────────────────────────────────────────╯
```

Note: The empty prompt `Type your message or @path/to/file` always appears in the box - that's normal. What matters is whether YOUR query text is inside or outside the box.

If your query is inside the box, run `tmux send-keys -t <session_name> Enter` to submit.

### Cleanup when done

```bash
tmux kill-session -t <session_name>
```

### If Gemini fails completely

If retrying without `-m` also fails, fall back to Method 2 below.

---

## Method 2: curl with Reddit JSON API (Fallback)

Reddit's public JSON API works by appending `.json` to any Reddit URL. Use this when Gemini is unavailable (quota exhausted, API errors, etc.).

### Listing hot/new/top posts

```bash
curl -s -L -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  "https://old.reddit.com/r/SUBREDDIT/hot.json?limit=15"
```

Replace `hot` with `new`, `top`, or `rising` as needed. For `top`, add `&t=day` (or `week`, `month`, `year`, `all`).

### Fetching a specific post + comments

```bash
curl -s -L -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  "https://old.reddit.com/r/SUBREDDIT/comments/POST_ID.json?limit=20"
```

The response is a JSON array: `[0]` is the post, `[1]` is the comment tree.

### Searching within a subreddit

```bash
curl -s -L -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  "https://old.reddit.com/r/SUBREDDIT/search.json?q=QUERY&restrict_sr=on&sort=new&limit=15"
```

### Parsing the JSON

Use jq to extract what you need:

```bash
# List posts
curl -s -L -o /tmp/reddit_result.txt -w "%{http_code}" \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  'https://old.reddit.com/r/SUBREDDIT/hot.json?limit=15'

jq -r '.data.children[] | .data | "\(.title)\n   \(.score) pts | \(.num_comments) comments | u/\(.author) | id: \(.id)\n"' /tmp/reddit_result.txt

# List comments from a specific post (the [1] element has comments)
jq -r '.[1].data.children[] | select(.kind == "t1") | .data | "u/\(.author) (\(.score) pts):\n  \(.body[:300])\n"' /tmp/reddit_thread.txt
```

Key details:
- Fetch to temp file first, then parse - avoids pipe-related encoding issues
- `-o /tmp/file` and `-w "%{http_code}"` saves the response and prints the HTTP status (useful for debugging empty responses)
- `-L` follows redirects (old.reddit.com sometimes redirects)
- Single-quoted URL avoids shell interpretation of `&` in query strings
- `.body[:300]` truncates long comment bodies (jq 1.7+)

### Rate limiting

Reddit's JSON API rate-limits aggressively:

- **Don't fire parallel requests.** Make them sequentially with `sleep 2` or `sleep 3` between each.
- If a request returns empty (0 bytes), wait 3-5 seconds and retry.
- If you get HTTP 429, back off for 10-15 seconds.
- A good pattern: fetch one search result listing, parse it, then fetch individual threads one at a time with delays.
