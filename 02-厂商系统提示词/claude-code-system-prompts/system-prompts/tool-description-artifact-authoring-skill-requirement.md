<!--
name: "Tool Description: Artifact authoring skill requirement"
description: "Requires the artifact design skill before authoring and describes workshop, diagramming, and contextual exceptions"
ccVersion: "2.1.257"
variables:
  - "ARTIFACT_DESIGN_SKILL_NAME"
  - "WORKSHOP_SKILL_NAME"
  - "ARTIFACT_DIAGRAMMING_SKILL_NAME"
  - "ARTIFACT_AUTHORING_CONTEXT_NOTE"
-->
**Before writing the file — a skill-instructed `.md` included — you MUST load the `${ARTIFACT_DESIGN_SKILL_NAME}` skill**: it carries the page contract — author HTML (Markdown only when a loaded skill instructs it), the publish-time skeleton, the title, which libraries a page may load, browser storage, the size cap, responsive layout, theming and the favicon — and calibrates how much design investment this particular request warrants; Markdown is never a shortcut past it. The one exception to loading it is a workshop document from the `${WORKSHOP_SKILL_NAME}` skill — both its lanes carry their own design: skip `${ARTIFACT_DESIGN_SKILL_NAME}` there, and load `${ARTIFACT_DIAGRAMMING_SKILL_NAME}` for a template page's diagrams instead. Then write the content to a file (via Write/Edit) and call Artifact with its path. ${ARTIFACT_AUTHORING_CONTEXT_NOTE}
