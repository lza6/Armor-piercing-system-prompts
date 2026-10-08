<!--
name: "Data: SDK set max thinking tokens request schema"
description: "Schema description for the SDK set_max_thinking_tokens control request, including token-budget reset behavior and session-scoped thinking display overrides"
ccVersion: "2.1.241"
-->
Sets the maximum number of thinking tokens for extended thinking. When max_thinking_tokens is omitted or null, thinking resets to the session default: any mid-session budget override is cleared (back to the spawn-time budget, if one was set), and thinking stays off for sessions that have it disabled. thinking_display optionally sets the thinking display mode for the rest of the session: a value replaces the session display mode, null clears that override so Claude Code's default display handling applies again, and when omitted the display mode from session start (--thinking-display) is kept.
