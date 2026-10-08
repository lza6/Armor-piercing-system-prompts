<!--
name: "Data: SDK plugin warnings field"
description: "Schema description for optional plugin_warnings on SDK system init frames, distinguishing advisory feedback for loaded plugins from synthetic-source notices for content that did not load"
ccVersion: "2.1.248"
-->
@internal Plugin authoring feedback (e.g., a default folder shadowed by a manifest key). When `plugin` matches an entry in `plugins[]`, that plugin loaded and the warning is advisory; warnings with a synthetic `plugin` source (no matching `plugins[]` entry, e.g. workspace-level suppression notices) describe content that did NOT load. The key is omitted when there are no warnings. On connections that persist frames server-side (the local bridge-worker lane) this key is always omitted — plugin diagnostics stay in the local log, so an omitted key there does NOT assert a clean load.
