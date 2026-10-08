<!--
name: "Data: SDK control cancel request schema"
description: "Schema description for withdrawing an in-flight SDK control request and ignoring any later response"
ccVersion: "2.1.234"
-->
Tells the other side that the sender no longer needs the answer to one of its own in-flight control_requests (for example a pending can_use_tool prompt after the turn was interrupted, or one that another client already answered). Either side may send it for a request it originated. The sender stops waiting at once and ignores any control_response that still arrives for that request_id; a receiver that can abort the work does so and may still reply (typically with an error), otherwise it simply completes the request. There is no reply to the cancel itself.
