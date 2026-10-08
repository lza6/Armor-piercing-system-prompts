<!--
name: "Tool Parameter: Artifact title"
description: "Describes concise Artifact titles, HTML title-tag precedence, Markdown filename identity, and conditional naming behavior for creation from an Artifact type"
ccVersion: "2.1.260"
variables:
  - "IS_ARTIFACT_TYPE_CREATION_ENABLED"
-->
Title for the artifact — the name shown in the browser tab and gallery. A short, distinctive noun-phrase name — not a generic label, a summary, or a name with an appended explainer. Prefer a <title> tag at the top of the HTML itself; this parameter fills in only when the file lacks one in the first 8KB of the file, and never overrides the tag. HTML publishes only — Markdown pages keep their filename identity. Content always comes from file_path — there is no inline content parameter.${IS_ARTIFACT_TYPE_CREATION_ENABLED?' On a `type_url` create there is no HTML file and none of that applies: `title` is simply the new Artifact's name — what the user called it, or a short descriptive name; left out, it is named after the type (e.g. just "Slides").':""}
