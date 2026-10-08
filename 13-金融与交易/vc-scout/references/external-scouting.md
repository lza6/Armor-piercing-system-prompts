# External Scouting with Gemini/OpenCode

Use external agentic tools for faster searches with large context windows (1M+ tokens).

## Tool Selection

```
SCALE <= 3  → gemini CLI
SCALE 4-5   → opencode CLI
SCALE >= 6  → Use internal scouting instead
```

## Configuration

Read from `.claude/.vc.json` (falls back to legacy `.claude/.ck.json` if present):
```json
{
  "gemini": {
    "model": "gemini-3-flash-preview"
  }
}
```

Default model: `gemini-3-flash-preview`

## Gemini CLI (SCALE <= 3)

### Command
```bash
gemini -y -m <model> "[prompt]"
```

### Example
```bash
gemini -y -m gemini-3-flash-preview "Search src/ for authentication files. List paths with brief descriptions."
```

## OpenCode CLI (SCALE 4-5)

### Command
```bash
opencode run "[prompt]" --model opencode/grok-code
```

### Example
```bash
opencode run "Find all payment-related files in lib/ and api/" --model opencode/grok-code
```

## Installation Check

Before using, verify tools installed:
```bash
which gemini
which opencode
```

If not installed, ask user:
1. **Yes** - Provide installation instructions (may need manual auth steps)
2. **No** - Fall back to internal shell scouting (`internal-scouting.md`)

## Optional Parallel Workers

If your runtime supports parallel worker delegation, split external CLI prompts across multiple workers. Otherwise run the commands sequentially in the main session.

```
Worker 1: gemini -y -m gemini-3-flash-preview "[prompt1]"
Worker 2: gemini -y -m gemini-3-flash-preview "[prompt2]"
Worker 3: gemini -y -m gemini-3-flash-preview "[prompt3]"
```

Aggregate the resulting file lists before moving on to deeper reads.

## Prompt Guidelines

- Be specific about directories to search
- Request file paths with descriptions
- Set clear scope boundaries
- Ask for patterns/relationships if relevant

## Example Workflow

User: "Find database migration files"

Run 3 parallel external searches if worker delegation is available:
```
Worker 1: gemini -y -m gemini-3-flash-preview "Search db/, migrations/ for migration files"
Worker 2: gemini -y -m gemini-3-flash-preview "Search lib/, src/ for database schema files"
Worker 3: gemini -y -m gemini-3-flash-preview "Search config/ for database configuration"
```

## Reading File Content

When needing to read file content, use chunking to stay within context limits (<150K tokens safe zone).

### Step 1: Get Line Counts
```bash
wc -l path/to/file1.ts path/to/file2.ts path/to/file3.ts
```

### Step 2: Calculate Chunks
- **Target:** ~500 lines per chunk (safe for most files)
- **Max files per agent:** 3-5 small files OR 1 large file chunked

**Chunking formula:**
```
chunks = ceil(total_lines / 500)
lines_per_chunk = ceil(total_lines / chunks)
```

### Step 3: Run Parallel Reads If Available

**Small files (<500 lines each):**
```
Worker 1: cat file1.ts file2.ts
Worker 2: cat file3.ts file4.ts
```

**Large file (>500 lines) - use sed for ranges:**
```
Worker 1: sed -n '1,500p' large-file.ts
Worker 2: sed -n '501,1000p' large-file.ts
Worker 3: sed -n '1001,1500p' large-file.ts
```

### Chunking Decision Tree
```
File < 500 lines     → Read entire file
File 500-1500 lines  → Split into 2-3 chunks
File > 1500 lines    → Split into ceil(lines/500) chunks
```

If worker delegation is unavailable, execute the same chunks sequentially in the main session.

## Timeout and Error Handling

- Set 3-minute timeout per bash call
- Skip timed-out agents
- Don't restart failed agents
- On persistent failures, fall back to internal scouting
