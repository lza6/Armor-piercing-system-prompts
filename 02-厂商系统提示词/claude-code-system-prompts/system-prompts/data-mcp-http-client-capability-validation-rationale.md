<!--
name: "Data: MCP HTTP client capability validation rationale"
description: "Explains where required client capabilities are validated in the modern MCP HTTP request ladder and how the pre-dispatch gate affects observable precedence"
ccVersion: "2.1.221"
-->
The capability requirement is checked by the HTTP entry, pre-dispatch, against the validated envelope the classifier produced — pinning the spec-mandated HTTP 400 independently of how dispatch- and handler-produced errors are mapped. The documented order (after method resolution and params validation) is preserved observably only while the requirement table is empty: once a served method gains a requirement entry, a request that is missing the capability and would also fail a dispatch rung is answered by this gate first, so the entry must consult the method registry before the gate if the documented precedence is to stay observable.
