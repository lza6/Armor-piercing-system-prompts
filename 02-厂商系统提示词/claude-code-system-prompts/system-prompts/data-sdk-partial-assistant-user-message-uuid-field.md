<!--
name: "Data: SDK partial assistant user message UUID field"
description: "Schema description for the optional user_message_uuid join key on non-ping SDK partial assistant stream events, including mid-turn ownership changes, placement, and backward compatibility"
ccVersion: "2.1.265"
-->
Client uuid of the user message this turn is answering (submitMessage options.uuid), stamped on a non-ping stream event each time that send changes: the turn's FIRST non-ping stream event (normally the frame that triggers the turn's initial ack), and, for a turn started by a synthetic (meta) prompt, the first non-ping stream event after each queued user message folded in mid-turn takes the echo over (see SDKAssistantMessage.user_message_uuid for the rule) — so a consumer can bind the reply stream to the send it answers without waiting for the result. A turn started by a typed prompt stamps its first non-ping stream event only. Absent on every other stream event of the turn, on turns that neither had a client uuid nor folded a user message in, and from older producers.
