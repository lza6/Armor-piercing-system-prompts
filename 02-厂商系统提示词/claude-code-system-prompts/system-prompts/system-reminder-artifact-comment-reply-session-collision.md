<!--
name: "System Reminder: Artifact comment reply session collision"
description: "Reports how concurrent live sessions resolved or may duplicate automatic replies to comments on the same Artifact"
ccVersion: "2.1.242"
variables:
  - "ESCAPE_EVENT_TEXT_FN"
  - "OTHER_SESSION_YIELDED_REPLIES"
  - "ARTIFACT_URL"
  - "THIS_SESSION_YIELDED_REPLIES"
-->

<event>${ESCAPE_EVENT_TEXT_FN(OTHER_SESSION_YIELDED_REPLIES?`Another live session of this same conversation was also armed to reply to comments on ${ARTIFACT_URL}; it paused its replies at this session's request, so only this session answers them now. Nothing to do; do not stop a watch on your own.`:THIS_SESSION_YIELDED_REPLIES?`Another live session of this same conversation claimed the replies to comments on ${ARTIFACT_URL} a moment after this one; this session paused its own at that session's request, so only that session answers them now. Nothing to do — a publish the user asks for here takes them back; do not republish or stop a watch on your own.`:`Another live session of this same conversation is running. If it is also replying to comments on ${ARTIFACT_URL}, every comment will get a reply from both sessions until one stops. Tell the user; they can end either session's live-updates task in /tasks. Do not stop a watch on your own.`)}</event>
