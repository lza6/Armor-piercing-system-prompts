# Architecture

This document captures the current technical shape and boundaries.

## Current Shape

Describe the main runtime pieces:

- entrypoint or app shell;
- data store;
- API or service boundaries;
- UI surface;
- background jobs, if any;
- external integrations, if any.

## Data Model

Describe the main entities and fields at a high level.

## Boundaries

The project should not silently cross these boundaries:

- data it must not collect;
- network calls it must not make;
- user actions that require consent;
- features that are future work.

## Decisions

Important architecture decisions should be recorded in
[`docs/DECISIONS.md`](./DECISIONS.md) or `docs/adr/`.

## Out Of Scope

- Broad feature 1.
- Broad feature 2.
- Risky automation that should wait until the MVP works.
