<!--
name: "Tool Description: Live artifact watch guidance"
description: "Explains live artifact change subscriptions, comment wake behavior, session scope, and truthful watch-status reporting"
ccVersion: "2.1.257"
variables:
  - "HAS_ARTIFACT_COMMENTS"
  - "ARTIFACT_WATCH_STATUS_GUIDANCE"
  - "LIVE_ARTIFACT_WATCH_NOTE"
-->
**Watching for republishes**: publishing an artifact starts subscribing this session to its live changes in the background, and the result line says whether that began, was skipped, or was already connected; you are told if it cannot connect, and watches reconnect on their own if the connection drops. A later republish from elsewhere — another session, or someone saving from a page that can publish new versions of itself — arrives as a notification telling you to re-read it before editing.${HAS_ARTIFACT_COMMENTS?" A comment on a watched artifact that is sent to Claude also wakes this session while that artifact's auto-replies are armed (when comment auto-replies are on for this session, a publish arms them).":""} ${ARTIFACT_WATCH_STATUS_GUIDANCE} Watches are session-local, and the user can see and stop them in /tasks. Do not claim you are watching an artifact unless a watch result or a publish result's "already connected" line says so — its "arming" line is not yet a watch. Only an interactive or SDK main-loop session holds a watch (not a subagent, teammate, background, or print session).${LIVE_ARTIFACT_WATCH_NOTE}
