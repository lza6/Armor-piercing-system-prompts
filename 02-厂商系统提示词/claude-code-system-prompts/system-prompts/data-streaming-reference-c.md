<!--
name: "Data: Streaming reference — C#"
description: "C# streaming reference including streaming events and the RawMessageStreamEvent TryPick methods"
ccVersion: "2.1.260"
-->
# Streaming - C#

## Streaming

```csharp
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = "{{OPUS_ID}}",
    MaxTokens = 64000,
    Messages = [new() { Role = Role.User, Content = "Write a haiku" }]
};

await foreach (RawMessageStreamEvent streamEvent in client.Messages.CreateStreaming(parameters))
{
    if (streamEvent.TryPickContentBlockDelta(out var delta) &&
        delta.Delta.TryPickText(out var text))
    {
        Console.Write(text.Text);
    }
}
```

**`RawMessageStreamEvent` TryPick methods** (naming drops the `Message`/`Raw` prefix): `TryPickStart`, `TryPickDelta`, `TryPickStop`, `TryPickContentBlockStart`, `TryPickContentBlockDelta`, `TryPickContentBlockStop`. There is no `TryPickMessageStop` - use `TryPickStop`.

---

