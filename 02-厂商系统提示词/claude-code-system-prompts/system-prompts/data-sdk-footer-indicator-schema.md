<!--
name: "Data: SDK footer indicator schema"
description: "Schema description for the server-configured SDK footer indicator carried on system/init and initialize responses so host UIs can render the terminal status pill"
ccVersion: "2.1.248"
-->
@internal The server-configured session indicator (bootstrap client_data.footer_indicator) that the terminal renders as a "◆ <text>" pill in the prompt footer — an opaque status note operators set per cohort (e.g. to prove a test config reached the session). Carried on `system/init` and the `initialize` response so a host UI (Claude Desktop, IDE webviews) can render the same pill. Absent when nothing is configured; hosts should then render nothing. Read from the CLI's cached bootstrap data, so a label configured after that cache was last written first appears on a later `system/init`.
