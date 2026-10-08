<!--
name: "Data: SDK remote tool call request schema"
description: "Schema description for the internal remote_tool_call request used by cloud session workers to forward one tool-call leg, approval leg, detached decline, or outcome query to the attached machine that announced the tool"
ccVersion: "2.1.247"
-->
@internal One leg of a tool call this session forwards to the attached machine that announced it serves the tool: the first leg, the approval leg (envelope.approval), a detached decline, or an outcome query (envelope.op = 'outcome_of'). Answered with a control_response whose success payload is the served CallToolResult; cancelled with control_cancel_request. An error-shaped control_response is not an answer (an older attached client error-replies subtypes it does not know); the worker keeps waiting.
