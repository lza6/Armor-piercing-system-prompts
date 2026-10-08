<!--
name: "Data: SDK query result modelUsage field"
description: "Schema description for the cumulative modelUsage field on SDK query results, including per-model accounting, streaming-session lifecycle, excluded helper calls, and estimate caveats"
ccVersion: "2.1.223"
-->
Per-model totals for every model call made through the query pipeline during this query() call — main loop, Task subagents, sidechains, and internal calls such as compaction and Workflow agents. Cumulative across turns in streaming-input sessions: each result carries the running total so far, so read the latest result rather than summing across results. Internal helper calls outside the query pipeline (e.g. the permission classifier, token-count probes) are excluded; crash/startup-error results may carry zeroed usage, resumed sessions start fresh, and a mid-session /clear resets the running total. The correct field for token/cost accounting; treat it as an estimate, not a billing statement.
