<!--
name: "Tool Description: Artifact unsupported supporting file error"
description: "Formats the Artifact publish error for a supporting file whose content type cannot be served"
ccVersion: "2.1.246"
variables:
  - "SUPPORTING_FILE_LABEL"
  - "SUPPORTING_FILE_OBJECT"
  - "JSON_STRINGIFY_FN"
-->
${SUPPORTING_FILE_LABEL} "${SUPPORTING_FILE_OBJECT.path}": contentType ${JSON_STRINGIFY_FN(SUPPORTING_FILE_OBJECT.contentType)} is not servable (nothing was published). Supporting files are assets the page itself loads — scripts, styles, images, media, JSON — and only standard web media types are served, so re-encode a data asset into one (e.g. JSON) or inline it. If the intent was instead to hand the viewer a file to keep, note that neither a served file nor a data:/blob: download link does that (the viewer blocks page-initiated downloads); offering a file to save is a runtime capability where available.
