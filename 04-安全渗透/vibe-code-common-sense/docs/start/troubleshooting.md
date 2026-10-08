# Troubleshooting

## The Agent Starts Coding Too Soon

Say:

```text
Do not build product features yet. First create the project rails.
```

## The Agent Skips The Tracker

Say:

```text
Read docs/PRODUCT_TIMELINE.md and update the completed item with the PR number.
```

## Claude Code Does Not Show The Skill

Run:

```text
/reload-plugins
```

Then try:

```text
/vc-sense:vc-sense
```

## The Agent Keeps Re-Deciding Architecture

Tell the agent:

```text
Record this in docs/DECISIONS.md so future agents do not re-decide it.
```
