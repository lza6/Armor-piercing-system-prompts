<!--
name: "Tool Description: Finding artifacts from earlier sessions"
description: "Explains how to list and recover artifacts from earlier sessions and conditionally provides client-specific follow-up guidance"
ccVersion: "2.1.257"
variables:
  - "HAS_ARTIFACT_GALLERY_GUIDANCE"
  - "ARTIFACT_GALLERY_GUIDANCE"
  - "ARTIFACT_LIST_FALLBACK_GUIDANCE"
-->
**To find artifacts from earlier sessions**: pass `action: "list"` (optionally with `limit` and `scope`) to enumerate the user's published artifacts — title, URL, favicon, and last-updated, newest first. Use it when the user refers to a published artifact whose URL you don't have, then follow the update flow above with the URL you found. Artifacts published earlier in THIS session need neither `action: "list"` nor `url` — calling again with the same file path redeploys them. ${HAS_ARTIFACT_GALLERY_GUIDANCE?ARTIFACT_GALLERY_GUIDANCE:ARTIFACT_LIST_FALLBACK_GUIDANCE}
