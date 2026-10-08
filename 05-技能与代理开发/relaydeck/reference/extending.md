# Skills & plugins — managing and extending the fleet

Two extension surfaces:
- **Skills** = SKILL.md bundles injected into the *agents* (what an agent
  knows how to do).
- **Plugins** = code loaded into the *daemon* (new harnesses, providers,
  policy, UI, CLI, event reactors).

This file is the operator's view: managing both from the CLI. To *author* a
plugin in depth, use the **relaydeck-plugin-dev** skill.

## Skills — what agents can do

### Inventory & inspection
```sh
relaydeck skills list                  # every discovered skill across sources
relaydeck skills show <name>           # one skill's metadata + which agents see it
relaydeck skills doctor                # health summary across all sources
relaydeck skills hubs                  # curated public skill discovery sources
```

### How skills reach an agent (the injection contract)
An agent sees a skill from one of three places:
- **Runtime/plugin skills** — materialized by a plugin into a workspace
  (e.g. `messaging` injects `relaydeck-cli`, `telegram` injects
  `relaydeck-telegram`). Always injected when the plugin is enabled.
- **User skills** — skills you import into a workspace. Gated for the whole
  workspace by the `skills` harness gate in `agent.toml`; every newly spawned
  agent in an enabled workspace sees the valid user skills.
- **Native tool skills** — whatever the harness already reads from its own
  skill root (`~/.claude/skills`, `~/.codex/skills`).

So enabling a plugin on a workspace is how you give *every* agent there a
capability; importing a user skill + listing it in an agent's `skills` is how
you give *one* agent a capability.

### Importing skills into a workspace
```sh
# From any supported source (git, npm, local, hub) — discover + import:
relaydeck skills add <source> [-w <ws>] [--skill <name>] \
    [--mode symlink|copy|reference] [--dry-run]

# A local skill directory → a workspace:
relaydeck skills link ./my-skill -w proj --alias my-skill

# A managed catalog skill → a workspace:
relaydeck skills deploy <alias> -w proj

relaydeck skills unlink <name> -w proj         # remove from a workspace
relaydeck skills rescan                         # refresh the inventory cache
relaydeck skills refresh-imports                # check managed imports for drift
```

`--mode`: `symlink` (default, edits track the source), `copy` (snapshot),
`reference` (point at it in place).

### Installing THIS skill for an external agent
```sh
relaydeck skills install                       # → ~/.claude/skills/relaydeck
relaydeck skills install --target both          # Claude + Codex roots
relaydeck skills install --target codex --force # replace an existing install
```

## Plugins — what the daemon can do

### Inventory & provenance
```sh
relaydeck plugin list                  # loaded plugins + Source + Trust
relaydeck plugin show <name>           # metadata + current settings (with sources)
relaydeck plugin info <name>           # install lock / provenance details
relaydeck plugin bundle [<name>]       # recommended bundles (default/minimal)
relaydeck plugin search <query>        # installable plugins in the registry
```

**Trust ladder** (high→low): `bundled > curated > local > untrusted`. An
`untrusted` plugin won't load unless approved on install or
`RELAYDECK_ALLOW_UNTRUSTED_PLUGINS=1`. `Source` tells you where it came from
(`builtin`/`installed`/`user`).

### Enable / disable / configure
```sh
relaydeck plugin enable autopilot
relaydeck plugin disable telegram
relaydeck plugin set autopilot mode benign         # one setting: NAME KEY VALUE
relaydeck plugin set manager on_context_critical compact
relaydeck plugin unset autopilot mode              # back to env/schema default
```

Per-workspace enablement (vs global) is on the workspace:
```sh
relaydeck workspace plugins proj                   # show enabled
relaydeck workspace plugins proj --add messaging --add skills
relaydeck workspace plugins proj --add fleet-context
```

### Install / update / remove
```sh
relaydeck plugin install ./my-plugin --editable    # local dir, live edits
relaydeck plugin install git+https://…@<sha>       # pinned git
relaydeck plugin install <registry-name>           # curated registry entry
relaydeck plugin update <name>                      # reinstall from recorded source
relaydeck plugin uninstall <name>
```

### Authoring (quick start — deep guide: relaydeck-plugin-dev skill)
```sh
# Private plugin, just for you (plain dir, never pushed upstream):
relaydeck plugin new my-plugin --local
# Workspace-scoped private plugin:
relaydeck plugin new my-plugin --workspace proj
# Publishable package (community/PyPI or core PR), pick a starting pattern:
relaydeck plugin new my-plugin --pattern reactor   # reactor|workflow|harness|provider|ui|cli|skill

relaydeck plugin lint ./relaydeck-plugin-my-plugin/plugin.toml
relaydeck plugin test ./relaydeck-plugin-my-plugin
relaydeck plugin publish-check ./relaydeck-plugin-my-plugin
relaydeck plugin dev [path]                         # set up an editable checkout
```

A plugin reacts to the same events you see on `events tail` (the
`reactor`/`workflow` patterns subscribe to event families), exposes settings
(typed, shown by `plugin show`), and can register CLI commands, API routes,
harnesses, providers, or UI tabs. The `skill` pattern ships a SKILL.md that
gets materialized into workspaces — that's how `messaging`/`telegram` inject
their skills. Pick the tier (private / community / core) first; the API is
identical, only packaging differs.

## Hooks — reacting to fleet events

Two distinct "hook" mechanisms, don't confuse them:

1. **Integration telemetry**: the always-on engine derives status from every
   PTY; `relaydeck integration install claude` can add a deterministic
   vendor-side hook for Claude Code. This is about *status accuracy* — see
   `reference/monitoring.md`.
2. **Plugin event reactors** (daemon-side): a plugin that subscribes to
   `events.subscribe` and runs code when an event fires (e.g. `manager`
   reacting to `agent.context`). This is how you automate policy. Scaffold
   one with `relaydeck plugin new <name> --pattern reactor`.

To *emit* an event a reactor (or a human watching the stream) can hook onto,
use `relaydeck events emit <type> --data k=v` from anywhere in your flow.
