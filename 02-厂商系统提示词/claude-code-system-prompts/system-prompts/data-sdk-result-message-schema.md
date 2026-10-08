<!--
name: "Data: SDK result message schema"
description: "Schema description for the turn-complete SDK result message, including success, API-error, and early-stop outcomes"
ccVersion: "2.1.234"
-->
The outcome of a turn. The CLI emits exactly one result message per turn, after that turn's assistant, user and stream_event messages; treat it as the turn-complete signal (informational system messages such as task notifications, session state changes or prompt suggestions may still follow it). subtype "success" carries the final assistant text in result — or, with is_error true, the error text when the turn ended on an API error; the error subtypes say why the turn stopped early. In single-prompt (non-streaming-input) mode the process exits after the turn.
