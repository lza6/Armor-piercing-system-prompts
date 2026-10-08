<!--
name: "Data: SDK assistant user message UUIDs field"
description: "Schema description for the optional user_message_uuids join-key list on the first SDK assistant reply frame, including prompt-batch ordering, bounds, placement, and backward compatibility"
ccVersion: "2.1.265"
-->
Client uuids of every user message whose prompt this turn has consumed so far, in consumption order — all members of a prompt batch the host merged into this one turn (several messages sent close together run as one turn whose user_message_uuid is the LAST member's), then any user message folded into the turn before this frame — so a consumer that sent any of them can bind this reply to its own send by finding its uuid anywhere in the list. Always contains user_message_uuid; at most 64 entries. Present exactly when user_message_uuid is, on the same frames; absent from older producers (fall back to user_message_uuid).
