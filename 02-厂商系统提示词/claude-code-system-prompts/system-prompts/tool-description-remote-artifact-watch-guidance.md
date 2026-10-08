<!--
name: "Tool Description: Remote artifact watch guidance"
description: "Explains durable remote artifact wake subscriptions, registration, comment wakes, and truthful watch-status reporting"
ccVersion: "2.1.257"
variables:
  - "HAS_ARTIFACT_COMMENTS"
  - "ARTIFACT_WATCH_STATUS_GUIDANCE"
  - "REMOTE_ARTIFACT_WATCH_NOTE"
-->
**Watching for republishes**: in this remote session a watch is a durable wake subscription held by the artifact service, not a live connection: this session is woken with a new turn when the watched artifact is republished elsewhere${HAS_ARTIFACT_COMMENTS?", or when a comment on it is sent to Claude":""}; nothing streams in between, so on a wake re-read the artifact before editing. Publishing an artifact starts registering its watch in the background, and the result line says whether that began, was skipped, or was already registered. ${ARTIFACT_WATCH_STATUS_GUIDANCE} Do not claim you are watching an artifact unless a watch result or a publish result's "already registered" line says so — its "arming" line is not yet a watch.${REMOTE_ARTIFACT_WATCH_NOTE}
