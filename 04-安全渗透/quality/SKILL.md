---
name: quality
description: "Score a report draft before submission. Usage: /quality <draft-path-or-finding-description>"
disable-model-invocation: false
---
Quality-check a report draft: $ARGUMENTS

1. If a file path is given, read the draft file.
2. Launch `quality-check` agent with the draft content.
3. Show the score and any issues to fix.
4. If score < 7, recommend specific improvements.
5. If score >= 7, confirm the report is ready for submission.

## Top-Tier Quality Bar

Block reports that are merely plausible.

Score against:
- reproducibility: exact steps, accounts, URLs, headers, and stable artifacts
- impact proof: real capability demonstrated without prohibited access
- severity fit: CVSS/vector matches the proven final state, not the hoped-for chain
- novelty: dupcheck considered and prior art handled
- evidence hygiene: screenshots, recordings, PoC paths, and request/response pairs exist on disk
- triage empathy: title, summary, and remediation let the program act fast

If any artifact is missing, say exactly what command or capture step fixes it. Do not let a fluent report compensate for weak proof.
