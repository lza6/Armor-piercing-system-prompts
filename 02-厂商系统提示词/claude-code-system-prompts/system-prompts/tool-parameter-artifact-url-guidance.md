<!--
name: "Tool Parameter: Artifact URL guidance"
description: "Explains when the Artifact url parameter must target an existing owned artifact and when it must be omitted for a new publish or same-conversation redeploy"
ccVersion: "2.1.239"
-->
Existing artifact URL to update in place. Pass whenever the user wants to update an artifact this conversation did not publish — "update my artifact", "keep the same link", a pasted artifact URL — and find the URL with action: "list" or ask the user for the link if you don't have it; without this, the publish creates a separate artifact instead of updating the existing one. Omit for new artifacts and same-conversation redeploys. Must be an artifact the user owns. For 'read' and the other url-addressed actions: the artifact to act on.
