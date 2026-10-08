<!--
name: "System Reminder: Memory index capacity warning"
description: "Warns when a private or team memory index approaches or exceeds its byte or line read limit and instructs Claude to compact it below the target size"
ccVersion: "2.1.235"
variables:
  - "CAPACITY_STATUS"
  - "MEMORY_INDEX_METADATA"
-->
${CAPACITY_STATUS.over?`Error: this write left the ${MEMORY_INDEX_METADATA.label} at ${MEMORY_INDEX_METADATA.displayPath} at ${CAPACITY_STATUS.sizeDesc}, over its ${CAPACITY_STATUS.capDesc} read limit. The write succeeded, but everything past the limit `+"is silently dropped each time the index is loaded — entries at the end are already invisible "+"to readers. Rewrite it":`The ${MEMORY_INDEX_METADATA.label} at ${MEMORY_INDEX_METADATA.displayPath} is ${CAPACITY_STATUS.sizeDesc}, approaching the ${CAPACITY_STATUS.capDesc} read limit. Compact it`} to under ${CAPACITY_STATUS.targetDesc} now: keep one line per entry, move detail into topic files, and merge or drop stale entries.
