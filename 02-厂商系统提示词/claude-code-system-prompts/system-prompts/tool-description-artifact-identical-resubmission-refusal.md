<!--
name: "Tool Description: Artifact identical resubmission refusal"
description: "Formats the Artifact publish refusal for unchanged content previously rejected against an unseen or newer live version and requires rereading and merging before retrying"
ccVersion: "2.1.247"
variables:
  - "PUBLISH_REFUSED_PREFIX"
  - "LIVE_VERSION"
  - "FORCE_REFUSAL_SUFFIX_FN"
  - "IS_FORCE_REFUSED"
-->
${PUBLISH_REFUSED_PREFIX}. Merge your edits onto ${LIVE_VERSION===void 0?"the live version's":"that version's"} source (handed to you or read in the turn that refused this content; if neither, fetch the artifact's URL first) and publish the merged result. If your content genuinely already includes that version's changes, fetch the artifact's URL again to confirm it (re-Reading a file an earlier refusal handed you does not count; if that fetch's result says the version counts as viewed only once its saved file is Read, Read every line of that file first) and, once you have that fetch's result, publish again${FORCE_REFUSAL_SUFFIX_FN(IS_FORCE_REFUSED)}
