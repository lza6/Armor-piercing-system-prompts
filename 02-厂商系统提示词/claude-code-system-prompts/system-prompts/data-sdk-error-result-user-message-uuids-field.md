<!--
name: "Data: SDK error result user message UUIDs field"
description: "Schema description for the optional user_message_uuids join-key list on SDK error results, including prompt batches, queued-message folding, ordering, bounds, exclusions, and backward compatibility"
ccVersion: "2.1.259"
-->
Client uuids of every user message whose prompt this turn consumed, in consumption order — all members of a prompt batch the host merged into this one turn (several messages sent close together run as one turn whose user_message_uuid is the LAST member's), then any queued user message folded into the running turn between tool rounds, once taken off the queue — so a consumer that sent any of them can bind this result to its own send by finding its uuid anywhere in the list. Always contains user_message_uuid; at most 64 entries; can be longer than the list on the turn's first reply frame. Present when a headless turn that ran echoes user_message_uuid; absent on delivery-failure and zeroed results and from older producers (fall back to user_message_uuid).
