<!--
name: "Data: Auto-compact inputs changed event schema"
description: "Schema description for worker-resolved auto-compaction state events used by thin clients to display the effective compaction countdown"
ccVersion: "2.1.227"
-->
@internal Worker-resolved auto-compact state, emitted by CCR workers at boot, whenever the resolved state changes (/autocompact, model switch, settings change), re-checked at each turn start, and re-emitted after a conversation reset. Thin clients adopt it so the "% until auto-compact" indicator counts down to the worker's real compaction trigger instead of re-resolving against client-local state. Turn-scoped divergence is accepted: a turn running under a skill/command frontmatter model override compacts against that model's window while the frame keeps the resting model's (the local indicator shares this limitation). From sessionState.onAutocompactInputsChanged.
