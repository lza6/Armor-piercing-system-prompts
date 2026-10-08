---
name: loom-topics
description: Coordinate bounded work that spans multiple Turns, days, or CodexLoom Agents through a shared Topic. Use when a workstream needs one Responsible Agent, scoped Participants, a versioned current brief, waiting conditions, evidence links, or recovery after time has passed; when receiving loom_topic_context; or when Message, Needs You, Trigger, Goal, Turn, or Artifact work should remain connected without sharing Agent Threads. Do not use for a one-Turn task, a permanent Domain, or a private Agent process.
---

# Loom Topics

A Topic is a thin shared coordination record for one bounded workstream. It preserves the current brief, responsibility, waiting condition, important evidence, and causal activity while every Agent keeps its detailed work in its own Thread.

A Topic is not a shared model Session, another Goal, a group chat, a task board, or a second source of truth. Provider state, code, artifacts, and Agent Threads remain authoritative for their own facts.

## Decide whether a Topic is warranted

Use a Topic when work has a completion boundary and at least one of these is true:

- it spans multiple Turns or days;
- it waits on external or human conditions;
- multiple Agents need a stable shared brief;
- the Owner needs to recover the work without reconstructing several Threads.

Keep a one-Turn request in the Agent Thread. Keep permanent responsibility in the Agent Profile. Use a Goal for one Agent's long-running execution lifecycle.

## Establish the coordination boundary

Create one Responsible Agent and only the Participants needed now:

```bash
loom topic create \
  --title "Parall Clip release closure" \
  --responsible parall-dev-lead \
  --purpose "Coordinate the bounded release stage across Lead, Edge, and Platform." \
  --completion "The responsible Agent publishes the verified release-stage result." \
  --summary "Candidate facts are being re-verified before the next gate." \
  --participant 'parall-edge-dev::Own packaged client verification.' \
  --participant 'parall-platform-dev::Own deployment and environment evidence.'
```

The Responsible Agent owns purpose, scope, Participants, versioned brief, waiting state, and final resolution. A Participant owns only its stated topic-scoped responsibility. The Owner confirms material scope, authorization, and organization changes.

Inspect before acting:

```bash
loom topic list --agent AGENT
loom topic get TOPIC_ID
```

## Work through causal links

The Responsible Agent routes scoped work with the Topic ID:

```bash
loom msg PARTICIPANT --from RESPONSIBLE --topic TOPIC_ID \
  --subject "Verify the current packaged candidate" \
  --body "Read current provider facts and return evidence or a blocking condition."
```

Participants report to the Responsible Agent using the same Topic ID:

```bash
loom msg RESPONSIBLE --from PARTICIPANT --topic TOPIC_ID \
  --subject "Re: Verify the current packaged candidate" \
  --body "Current result, limitation, and evidence IDs."
```

Do not send ordinary Topic work to an Agent outside the Topic. Add a Participant with a narrow responsibility first. Do not copy another Agent's full Thread into the Topic.

Link only durable anchors that help recovery:

```bash
loom topic link TOPIC_ID pull-request OWNER/REPO#1970 \
  --from RESPONSIBLE --label "Historical contract candidate"
loom topic link TOPIC_ID artifact ARTIFACT_ID \
  --from RESPONSIBLE --label "Packaged smoke report"
```

`loom msg --topic`, `loom ask-user --topic`, and `loom trigger add --topic` create causal Topic activity automatically. Replies inherit the Topic. Use explicit links for external Inbox / Outbox items, Artifacts, provider facts, or an existing native Goal that did not originate inside the Topic. Identify a Goal version as `<thread-id>@<goal-created-at>` because a Thread can receive a later Goal.

## Maintain the shared brief

Only the Responsible Agent updates the operational brief and publishes results:

```bash
loom topic update TOPIC_ID --from RESPONSIBLE --if-version VERSION \
  --summary "Current cross-domain conclusion." \
  --state "What is true now." \
  --next "The first next action." \
  --limitations "Known uncertainty."
```

When work must wait, record why, the stable reference, and what to do after wake-up:

```bash
loom topic update TOPIC_ID --from RESPONSIBLE \
  --waiting "Waiting for the target PR to enter main." \
  --waiting-kind github-pr \
  --waiting-ref OWNER/REPO#1970 \
  --resume-action "Re-read main and verify the expected contract."
```

Arm the related Trigger with `--topic TOPIC_ID`, then end the Turn. Clear waiting only after re-reading authoritative state.

Publish a stage result only after integrating Participant evidence:

```bash
loom topic update TOPIC_ID --from RESPONSIBLE --if-version VERSION \
  --summary "Verified stage result." --state "Ready for Owner review." --result
loom topic resolve TOPIC_ID --from RESPONSIBLE
```

Participants must not mutate the shared brief or declare the Topic resolved. They return facts, limits, and proposals to the Responsible Agent.

## Handle Owner input and intervention

`loom topic send TOPIC_ID ...` always starts a Turn for the Responsible Agent. Scope, priority, completion, and Participant assignment changes go through that path.

The Owner may guide or interrupt an exact active Participant Turn from the Topic UI or CLI:

```bash
loom topic intervene TOPIC_ID --agent PARTICIPANT --action steer --text "Re-check the current HEAD, not the archived candidate."
loom topic intervene TOPIC_ID --agent PARTICIPANT --action interrupt --reason "The candidate was superseded."
```

An intervention is an audited process correction. It does not change Topic status, reassign work, or silently change scope. If it changes the overall plan, the Responsible Agent must update the brief and replan.

## Read supplied context

`<loom_topic_context>` contains a bounded current brief, your Topic responsibility, waiting state, key links, and activity delta since your last Topic delivery. Treat it as coordination context, not a replacement for current provider reads or your own Thread history. Do not poll the Topic; linked Messages and Triggers resume the right Agent.
