<!--
name: "Data: Streaming reference — Ruby"
description: "Ruby streaming reference including basic streamed text handling"
ccVersion: "2.1.246"
-->
# Streaming - Ruby

## Streaming

```ruby
stream = client.messages.stream(
  model: :"{{OPUS_ID}}",
  max_tokens: 64000,
  messages: [{ role: "user", content: "Write a haiku" }]
)

stream.text.each { |text| print(text) }
```

---

