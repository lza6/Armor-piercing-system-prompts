# one-shot CLI

Every wigolo tool doubles as a plain terminal command. No server to start, no
session to manage — `wigolo <tool> <args>`, results on stdout, logs on stderr,
exit code you can gate on.

![demo](demo.gif)

## Run it

```bash
npx wigolo search "css grid layout guide" --max-results 5
npx wigolo fetch https://developer.mozilla.org/en-US/docs/Web/API/structuredClone
npx wigolo research "how does sqlite full text search work" --depth quick --json | jq -r '.brief.highlights[0].text'
```

Or run all three at once:

```bash
./run.sh                                  # uses npx wigolo
WIGOLO="wigolo" ./run.sh                  # or a global install
```

Requires node >= 20 and `jq`.

## What you'll see

Search returns ranked results with per-result evidence scores:

```text
Search: "css grid layout guide" (5 results, 3ms, engines: mdn, duckduckgo)

  [1] Basic concepts of grid layout - developer.mozilla.org (score: 1.00)
      CSS grid layout introduces a two-dimensional grid system to CSS. Grids can be
      used to lay out major page areas or small user interface elements. ...

  [2] Relationship of grid layout to other layout methods - developer.mozilla.org (score: 0.91)
      CSS grid layout is designed to work alongside other parts of CSS, as part of
      a complete system for doing the layout. ...
```

That `3ms` is real: repeat queries answer from the local knowledge cache. The
first run goes to the engines (a couple of seconds), and everything it learns
is kept on disk for next time.

Fetch turns any URL into clean markdown (JS-rendered pages included — a
browser engine kicks in only when needed):

```text
Fetch: https://developer.mozilla.org/en-US/docs/Web/API/structuredClone

  # Window: structuredClone() method

  Baseline Widely available

  [cached: true, 9570 chars]
```

Research runs multi-step question decomposition and returns a structured
brief. `--json` makes the whole thing pipeable:

```console
$ npx wigolo research "how does sqlite full text search work" --depth quick --json \
    | jq -r '.brief.highlights[0].text'
At its core, full-text search in SQLite is designed to split text into terms or
phrases and index these for fast search retrieval. When a full-text search is
queried, it doesn't scan rows linearly. Instead, it uses indexed tokens to match
search criteria, making full-text search much quicker than traditional LIKE
queries on large datasets.
```

## The contract

- Result → stdout. Logs → stderr. Pipes stay clean.
- `--json` emits the tool's full machine-readable output (same shape MCP
  clients get: `results`, `evidence`, `citations`, `engine_telemetry`, ...).
- Failure → exit code 1 (and under `--json`, a parseable error envelope).
- `--help` on any tool prints its full flag set: `npx wigolo search --help`.

## Where to go next

- Pipe many commands through one process: [shell-ndjson-pipeline](../shell-ndjson-pipeline/)
- The same tools over HTTP: [rest-curl](../rest-curl/)
- Typed clients: [sdk-typescript-research](../sdk-typescript-research/), [sdk-python-agent](../sdk-python-agent/)
