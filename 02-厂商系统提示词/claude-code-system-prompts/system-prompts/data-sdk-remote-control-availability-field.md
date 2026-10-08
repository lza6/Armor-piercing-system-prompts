<!--
name: "Data: SDK Remote Control availability field"
description: "Schema description for the optional internal remote_control_available initialize-response field, including stable deployment gates, IDE behavior, and the older-CLI default"
ccVersion: "2.1.251"
-->
@internal Whether Remote Control can be offered at all in this deployment (isRemoteControlDeploymentAvailable: not hard-disabled by managed settings, not nested in a remote environment, first-party provider). Only host-lifetime-stable conditions; transient ones (auth state, the GrowthBook rollout gate, the async org-compliance verdict) are deliberately excluded since hosts latch this from one initialize response — see the helper docstring. Lets IDE hosts hide their Remote Control affordance where it can never work. Absent (older CLI) → treat as available.
