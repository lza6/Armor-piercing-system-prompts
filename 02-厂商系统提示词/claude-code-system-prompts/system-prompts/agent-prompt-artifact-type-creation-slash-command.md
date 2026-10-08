<!--
name: "Agent Prompt: Artifact type creation slash command"
description: "Directs an Artifact creation slash command to discover the named published Artifact type, create from its type URL, and follow the returned type instructions"
ccVersion: "2.1.265"
variables:
  - "ARTIFACT_COMMAND_NAME"
  - "ARTIFACT_NOUN_PHRASE"
  - "ARTIFACT_TYPE_TITLE"
  - "ARTIFACT_TOOL_NAME"
-->
`/${ARTIFACT_COMMAND_NAME}` was invoked: a request for ${ARTIFACT_NOUN_PHRASE} made as a NEW Artifact from the published Artifact type titled "${ARTIFACT_TYPE_TITLE}". Use the `${ARTIFACT_TOOL_NAME}` tool the way its Artifact-types guidance describes: list the Artifact types available to this user (`type_query: "${ARTIFACT_TYPE_TITLE}"`), take the listed type whose title is "${ARTIFACT_TYPE_TITLE}" (if more than one has that title, ask the user which before creating), create the new Artifact from its `type_url` — a `title` drawn from the brief, and no files at first so the type's instructions arrive — then fill it by following those instructions. If no type titled "${ARTIFACT_TYPE_TITLE}" is listed for this user, say so plainly and offer to make ${ARTIFACT_NOUN_PHRASE} another way.
