# `relaydeck` — a publishable agent skill

The **`relaydeck` skill**: a self-contained, portable **agent skill** that
teaches *any* capable CLI agent (Claude Code, and any harness that reads
`SKILL.md`) to install and drive [relaydeck](https://relaydeck.ai) — a
general-purpose control plane for a **fleet of CLI agents**. Drop it into a
skill directory and your agent can register workspaces, spin up, supervise,
coordinate, unblock, collect from, and tear down a team of background agents.

It's **general-purpose**: relaydeck manages the fleet; the *purpose* is the
user's — parallel implementation, review/audit, research, large migrations,
cross-repo fan-out, monitoring/ops. Not "coding orchestration only."

> Packaging note: shipped as the **`relaydeck` skill** (not an "orchestrator
> plugin") so it can be published as a standalone skill package — including to
> an npm skill registry — under that name.

This bundle stands alone — it is **not** a relaydeck-internal skill. The
in-fleet skills (`relaydeck-fleet`, `relaydeck-cli`, …) are injected into
agents that already run *inside* relaydeck; this one is for the agent
*outside*, looking in. The source repository's
[skills catalog](https://github.com/relaydeck/relaydeck/blob/main/skills/README.md)
lists the in-fleet bundles separately.

## What's in here

```
relaydeck/
├── SKILL.md                     # the skill itself (frontmatter + body)
├── README.md                    # this file (install + publish + positioning)
├── scripts/
│   └── relaydeck-bootstrap.sh   # idempotent detect → install → daemon-up
└── reference/
    ├── commands.md              # full, verified command cheat-sheet
    ├── permissions.md           # autonomy, autopilot, unblock, escalate, auth
    ├── monitoring.md            # events, status, context/usage/limits, manager, hooks
    ├── extending.md             # skills management + plugins (install/configure/author)
    └── recipes.md               # fan-out, pipeline, quorum, migration, cross-repo, monitoring
```

The frontmatter stays lean because its metadata is always visible; the
`SKILL.md` body loads when the skill triggers, and `reference/` files load on
demand. Every command is checked against the real CLI to keep an agent's
tool-call failure rate low.

## Install (for an end user)

**Claude Code** — with relaydeck installed, use the built-in installer:

```sh
relaydeck skills install            # → ~/.claude/skills/relaydeck
relaydeck skills install --target both   # Claude + Codex roots
```

Or copy the folder into a skills directory manually:

```sh
cp -r relaydeck ~/.claude/skills/        # personal (all projects)
cp -r relaydeck .claude/skills/          # or project-scoped
```

The agent discovers it by `name` + `description`; when a task smells like
"run several agents" or "manage these workspaces/agents", it reads `SKILL.md`
and follows it — installing relaydeck on first use via
`scripts/relaydeck-bootstrap.sh`.

**Any other harness** that supports SKILL.md-style skills: place the folder in
that tool's skills path. The skill only depends on a shell and `relaydeck`
(which it installs itself).

> `SKILL.md` references `$CLAUDE_SKILL_DIR` for the script path. If your
> harness exposes the skill's directory under a different variable, substitute
> it (or run the absolute path to `scripts/relaydeck-bootstrap.sh`).

## Self-awareness built in (inside vs outside)

The skill's first step checks `RELAYDECK_AGENT_ID`:

- **unset** → external operator: install + bootstrap + drive a fleet.
- **set** → already a managed agent: do *not* bootstrap a second fleet;
  coordinate as a peer (use the `relaydeck-fleet` / `relaydeck-cli` skills).
  `RELAYDECK_ORCHESTRATION_DEPTH` (injected on every spawn) lets a nested
  agent see how deep it is and refuse a runaway fleet-of-fleets.

The bootstrap script enforces the same guard and exits early (code 2) if run
from inside relaydeck. So the skill is safe to publish broadly: it behaves
correctly whether it lands in a plain terminal or inside a relaydeck worker.

## Why this is the wedge

The CLI-agent world is exploding, but everyone hits the same wall the moment
one agent isn't enough: **how do I run, see, and coordinate many?** Today
people hand-roll tmux panes, `&` jobs, and bespoke scripts — fragile,
invisible, unsupervisable. relaydeck already solved the hard parts (per-agent
PTYs, cross-harness status, a message bus, durable results, a live dashboard,
prompt auto-answering). This skill is the **zero-to-fleet on-ramp** that makes
all of it reachable from inside the agent the user already trusts.

The flywheel:

1. **Distribution rides the host.** The skill installs into the agent the
   user *already* runs. No new app to adopt.
2. **One folder, instant capability.** Copy a directory → the agent can
   command a fleet. The "wow" is one prompt away.
3. **Harness-agnostic by design.** It drives *any* harness (Claude Code,
   Codex, Cursor, opencode, antigravity, pi, relaydeck-native). It multiplies
   the agent that loaded it instead of competing with it.
4. **The dashboard is the share moment.** The first time a human watches
   several agents work in parallel on one screen, they screenshot it.
5. **Safe-by-default earns trust.** Inside/outside guards,
   look-before-you-unblock, conservative auto-answer, escalate-the-rest.

### Go-to-market checklist (for the maintainers)

- [ ] Publish `relaydeck` to PyPI so `uv tool install relaydeck` works cold.
- [ ] List this skill in the public skill marketplaces the target harnesses
      index, with the demo GIF (several agents, one dashboard) as the hero.
- [ ] Ship a 60-second "zero to fleet" screencast.
- [x] Ship the bundle *inside* the `relaydeck` package plus a
      `relaydeck skills install` command that copies it into the user's skill
      roots.
- [ ] Publish the `relaydeck` skill as a standalone package (npm skill
      registry) for harnesses that install skills from npm.
- [x] Keep `SKILL.md` lean and the `reference/` deep — and **verified**
      against the CLI so tool calls don't fail.

## License

MIT, same as relaydeck.
