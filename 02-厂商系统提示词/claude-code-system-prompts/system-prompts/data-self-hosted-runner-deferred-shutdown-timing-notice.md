<!--
name: "Data: Self-hosted runner deferred shutdown timing notice"
description: "Explains deferred shutdown release and drain timing, supervisor timeout sizing, forced-kill consequences, and second-signal behavior for self-hosted runners"
ccVersion: "2.1.238"
variables:
  - "FORMAT_DURATION_FN"
  - "DEFER_SHUTDOWN_MAX_MS"
  - "POST_CEILING_GRACE_MS"
  - "MATH_OBJECT"
  - "SHUTDOWN_BUDGET_SECONDS"
-->
, and the runner exits as soon as it holds no session; ${FORMAT_DURATION_FN(DEFER_SHUTDOWN_MAX_MS,{hideTrailingZeros:!0})} after the signal every remaining session is released at once and anything still attached ${FORMAT_DURATION_FN(POST_CEILING_GRACE_MS,{hideTrailingZeros:!0})} later is drained. If your supervisor's stop timeout ends first, every still-attached session is killed WITHOUT its post-session hook or deregister and is requeued to another runner about a minute later. Size the stop timeout to at least ${MATH_OBJECT.ceil((DEFER_SHUTDOWN_MAX_MS+POST_CEILING_GRACE_MS)/1000)+SHUTDOWN_BUDGET_SECONDS}s (M + post-ceiling grace + the budget above). A second signal drains immediately.
