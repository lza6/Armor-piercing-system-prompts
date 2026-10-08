<!--
name: "Tool Description: Durable artifact watch and status guidance"
description: "Explains durable artifact wake subscriptions, optional comment wakes, status checks, and watch registration semantics"
ccVersion: "2.1.257"
variables:
  - "HAS_ARTIFACT_COMMENTS"
  - "COMMENTS_OFF_SENTENCE"
-->
**Watching for republishes**: in this remote session a watch is a durable wake subscription held by the artifact service, not a live connection: this session is woken with a new turn when the watched artifact is republished elsewhere${HAS_ARTIFACT_COMMENTS?", or when a comment on it is sent to Claude":""}; nothing streams in between, so on a wake re-read the artifact${HAS_ARTIFACT_COMMENTS?" (and its comments, on a comment wake)":""} before editing.${HAS_ARTIFACT_COMMENTS?' Plain comments never wake this session — read them with `action: "comments"` when the user asks.':COMMENTS_OFF_SENTENCE} Publishing an artifact starts registering its watch in the background, and the result line says whether that began, was skipped, or was already registered; `action: "status"` lists the watches that actually registered and what wakes each (pass `url` to check one). To watch an artifact you did not just publish, pass `action: "watch"` with its `url`; `action: "unwatch"` with `url` stops one. Do not claim you are watching an artifact unless a watch result, `status`, or a publish result's "already registered" line says so — its "arming" line is not yet a watch.
