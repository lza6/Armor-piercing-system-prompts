<!--
name: "Data: SDK initialize plugins parameter"
description: "Schema description for the optional SDK initialize plugins parameter, its launch-only loading through --await-initialize, and plugins_applied response semantics"
ccVersion: "2.1.261"
-->
Plugins to load for the session, in the same shape as the SDK `plugins` option: the stdin form of one --plugin-dir flag per entry (--plugin-dir-no-mcp when skipMcpDiscovery is set), so the launch command line does not grow with the plugin count. Loaded only by a CLI launched with --await-initialize, which reads this request during startup before any plugin work. Without that flag, on a repeated initialize, or over a remote session transport the field loads nothing; plugins_applied in the response reports whether the listed plugins are in fact loaded.
