<!--
name: "Data: Query result pending command count"
description: "Describes the query result field that counts queued user sends still awaiting turns and clarifies when it is absent or zero"
ccVersion: "2.1.242"
-->
User-initiated sends still waiting in the command queue when this result was produced. Greater than 0 means at least one more user turn (and result) follows without further input, barring cancellation; 0 means none is pending, or the session is ending (end_session or a shutdown latched mid-turn discards the backlog). Queued sends may coalesce into fewer turns, so this counts pending sends, not remaining results. System-generated queue entries are not counted. Absent on fatal startup results and on surfaces without a command queue.
