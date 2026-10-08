<!--
name: "Data: SDK register device hooks request schema"
description: "Schema description for the internal register_device_hooks request used by Claude cloud device clients to register forwarded hooks and vetted templates with a session worker"
ccVersion: "2.1.236"
-->
@internal Registers this device's hooks with the worker — unrelated to device registration or bind. Sent by a claude --cloud device client after it attaches to a cloud session (never as an initial event of the create): names, by opaque id, which of the user's own hooks the worker should ask this device to run over hook_callback, and which vetted templates to run in the container. The worker rejects requests over 64 KiB with invalid_registration. Error replies use the standard error response with a message that starts with a code token and a colon: hook_forwarding_disabled (off for this session; not retryable, nothing stored), hook_forwarding_not_ready (retry after a short backoff), invalid_registration, or stale_worker_epoch (re-read the worker epoch and re-send). A worker that predates the request answers with the usual unsupported-subtype error.
