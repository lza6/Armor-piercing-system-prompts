# Python SDK — autonomous gathering

The [`wigolo` PyPI package](https://pypi.org/project/wigolo/) is a thin,
stdlib-only Python client for the wigolo daemon. `local_client()` gives you
**embedded local mode**: it reuses a healthy daemon that's already listening,
or spawns `wigolo serve` itself and stops it on exit. Zero setup, nothing left
running.

## Run it

```bash
python3 -m venv .venv          # throwaway venv, stays inside this directory
.venv/bin/pip install wigolo
.venv/bin/python gather.py
```

Requires Python >= 3.9 and a wigolo CLI the SDK can spawn (`npm i -g wigolo`,
or set `WIGOLO_CLI` to a path / JSON argv list). The `.venv/` dir is
git-ignored — delete it when you're done.

## What you'll see (real output, trimmed)

```text
daemon status: healthy
pages fetched: 2  steps: 5  time: 7416ms

## Results: What is the latest SQLite release version and release date?

Gathered from 2 source(s):

### [1] SQLite Home Page
**URL:** https://sqlite.org/
...
### Latest Release

[Version 3.53.3](https://sqlite.org/releaselog/3_53_3.html) (2026-06-26). ...

sources:
  - https://sqlite.org/
  - https://sqlite.org/releaselog/current.html
```

## The shape of `result`

`gather.py` passes a small JSON Schema:

```python
res = client.agent(
    prompt="What is the latest SQLite release version and release date?",
    schema={
        "type": "object",
        "properties": {
            "latest_version": {"type": "string"},
            "release_date": {"type": "string"},
        },
    },
    urls=["https://sqlite.org/"],   # optional seed; the agent also plans its own searches
    max_pages=3,
    max_time_ms=90_000,
)
```

By contract `res["result"]` is **`str | dict`**: when the gatherer can shape
values to your schema it returns a dict; otherwise you get the full readable
report of what it gathered (that's what this run produced, and the answer is
right there in it). Handle both branches — `gather.py` shows the pattern. In
keyless mode the shaping is heuristic; wiring an optional local or cloud
language model makes it considerably more schema-faithful.

## Beyond `agent`

One method per tool, sync (`Client`) and async (`AsyncClient`) with the same
surface:

```python
from wigolo import local_client

with local_client() as client:
    client.search(query="css container queries", max_results=5)
    client.fetch(url="https://example.com")
    client.research(question="...", depth="quick")
```

Full client docs: the [wigolo PyPI page](https://pypi.org/project/wigolo/).
TypeScript version of this example: [sdk-typescript-research](../sdk-typescript-research/).
