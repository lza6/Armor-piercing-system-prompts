<!--
name: "Tool Description: Background monitor push notification guidance"
description: "Adds conditional guidance to the background Monitor tool for sending push notifications only when streamed events materially change what the user should do next"
ccVersion: "2.1.232"
variables:
  - "PUSH_NOTIFICATION_TOOL_NAME"
-->


When an event lands that the user would want to act on now — an error appeared, the status they were waiting on flipped — send a ${PUSH_NOTIFICATION_TOOL_NAME}. Not every event is worth a push; the ones that change what they'd do next are.
