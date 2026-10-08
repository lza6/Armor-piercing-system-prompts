<!--
name: "System Reminder: Artifact auto-replies resumed"
description: "Reports that Artifact comment auto-replies resumed, explains which stopped-period comments are handled based on the stop cause, and warns that the stop remains until the watch reconnects"
ccVersion: "2.1.238"
variables:
  - "FORMAT_ARTIFACT_URL_FN"
  - "RESUME_REPLIES_RESULT"
-->
Auto-replies resumed on ${FORMAT_ARTIFACT_URL_FN(RESUME_REPLIES_RESULT.url)} — the live watch is re-armed; the stop clears with a visible notice when the watch connects. Once connected, new to-Claude comments are answered; ${RESUME_REPLIES_RESULT.stop_kind==="interrupt"?"comments sent to Claude while replies were paused (since the interrupt) are answered too":RESUME_REPLIES_RESULT.stop_kind==="user"?"comments sent to Claude while the watch was killed or unwatched stay unanswered history":"comments sent to Claude while replies were stopped are picked up too if the stop was a session interrupt (Ctrl+C or Stop), and stay unanswered history if the watch had been killed or unwatched"}. If the watch fails to connect, this turn is interrupted before it does, or the user stops auto-replies again before it connects, the stop stays in place — check action "status" and resume again if the user still wants it.
