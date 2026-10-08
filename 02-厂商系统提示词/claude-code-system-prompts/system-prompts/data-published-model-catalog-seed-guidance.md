<!--
name: "Data: Published model catalog seed guidance"
description: "Documents the compiled model catalog seed's role, provenance, refresh process, and minimum accepted version"
ccVersion: "2.1.257"
-->
Compiled-in seed of the published model-catalog document (utils/model/servedCatalog/published/seed.ts): what a session reads until its first fetch of https://downloads.claude.ai/model-catalog/v1/catalog.json has been cached. HAND-BUILT for now from the public ids in model-catalog.json, in the document's envelope shape; a bot PR will refresh this file from the published document once the publisher is live, so do not hand-edit rows here on a model launch — the hosted document is what launches a model, this file only has to be valid and public. Version 0 is the floor below which no published document is accepted.
