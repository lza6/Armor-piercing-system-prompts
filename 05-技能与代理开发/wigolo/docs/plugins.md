# Plugins

wigolo loads two kinds of plugins from `~/.wigolo/plugins` (override with `WIGOLO_PLUGINS_DIR`): **search engines** that join the multi-engine dispatch, and **content extractors** that get first shot at turning a page into markdown for URLs they claim. Both are plain Node modules — no build step, no framework.

## Managing plugins

```bash
wigolo plugin add https://github.com/you/your-plugin   # clone; prompts before trusting
wigolo plugin list [--json]
wigolo plugin validate [--json]                        # do installed plugins load + export correctly?
wigolo plugin remove <name>
```

`plugin add` clones a git repository into the plugins dir and asks for confirmation before it does — a plugin is code that will run inside your wigolo process with your credentials and network access, so only install plugins you trust or have read. `--yes` skips the prompt for scripted setups.

`plugin validate` loads every installed plugin and reports whether its exports match the required contracts, with the exact reason when one doesn't.

## Package shape

A plugin is a directory with a `package.json` whose `main` points at a module:

```json
{
  "name": "my-wigolo-plugin",
  "version": "1.0.0",
  "main": "index.mjs"
}
```

The module exports `searchEngine`, `extractor`, or both. Exports are validated at load; an invalid plugin is reported and skipped — it never takes the server down.

## Search-engine plugins

The contract (from `src/plugins/` and `src/types.ts`):

```ts
interface SearchEngine {
  name: string;
  search(query: string, options?: SearchEngineOptions): Promise<RawSearchResult[]>;
}
```

A complete working engine — this is the whole of [`examples/plugin-search-engine`](../examples/plugin-search-engine/), copy it as your starting point:

```js
export const searchEngine = {
  name: 'example-search-engine',
  async search(query) {
    return [
      {
        title: `Example result for ${query}`,
        url: 'https://example.com/search-engine-example',
        snippet: 'Minimal search engine plugin example.',
        relevance_score: 1,
        engine: 'example-search-engine',
      },
    ];
  },
};
```

That's genuinely it — an internal wiki, a private index, a niche vertical engine lands in the dispatch pool in well under 100 lines. Results from your engine flow through the same fusion, dedup, and on-device reranking as the built-ins, and show up in `engines_used` / `engine_telemetry` like any other engine.

## Extractor plugins

```ts
interface Extractor {
  name: string;
  canHandle(url: string, html?: string): boolean;
  extract(html: string, url: string): ExtractionResult | null;
}
```

`canHandle` decides which URLs your extractor claims (e.g. your company's docs platform with a quirky DOM); `extract` returns the structured result, or `null` to hand the page back to the built-in extraction ensemble. Plugin extractors are consulted before the generic pipeline, so a site-specific extractor is how you fix a site that extracts poorly.

## Failure behavior

Plugin loading is defensive end to end: a missing `package.json`, a bad `main`, a throwing import, or an invalid export produces a per-plugin error in `plugin validate` (and in the server log) while every other plugin — and the server itself — keeps working.

[← Docs index](./README.md) · [Next: Troubleshooting](./troubleshooting.md)
