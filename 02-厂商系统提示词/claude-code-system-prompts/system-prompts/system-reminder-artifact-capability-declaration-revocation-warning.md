<!--
name: "System Reminder: Artifact capability declaration revocation warning"
description: "Warns that publishing a replacement capability declaration would implicitly revoke stored capabilities and explains how to preserve or intentionally revoke them"
ccVersion: "2.1.257"
variables:
  - "PLURALIZE_FN"
  - "OMITTED_CAPABILITY_ENTRIES"
  - "OMITTED_CAPABILITY_NAMES"
  - "CAPABILITY_DECLARATION_UNION"
-->
your capabilities declaration omits the stored ${PLURALIZE_FN(OMITTED_CAPABILITY_ENTRIES.length,"capability","capabilities")} ${OMITTED_CAPABILITY_NAMES.join(", ")} while adding new ones — a sent declaration replaces the stored one, so this publish would have silently revoked ${OMITTED_CAPABILITY_ENTRIES.length===1?"it":"them"}. To keep ${OMITTED_CAPABILITY_ENTRIES.length===1?"it":"them"}, republish declaring the union${CAPABILITY_DECLARATION_UNION.length<600?`: ${CAPABILITY_DECLARATION_UNION}`:" (republish with capabilities omitted to read the stored declaration back, then resend it plus your additions)"}. To revoke on purpose, publish that union first, then republish without the revoked names (a declaration that adds no new name goes out as sent); capabilities: {} clears everything.
