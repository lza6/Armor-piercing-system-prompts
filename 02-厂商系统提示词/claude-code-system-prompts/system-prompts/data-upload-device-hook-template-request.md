<!--
name: "Data: Upload device hook template request"
description: "Describes the cloud device request that uploads a vetted hook template to a worker before device hooks are registered"
ccVersion: "2.1.242"
-->
@internal Sent by a claude --cloud device client BEFORE register_device_hooks for each vetted template it wants run in the container; a register that arrives first reports awaiting_upload and the device registers again after uploading. The worker keeps the bytes for its own life only after hashing them against its own table; nothing executable is taken from the wire. Idempotent. Errors: hook_forwarding_disabled | hook_forwarding_not_ready (retryable) | stale_worker_epoch | invalid_upload | template_refused: unknown_template, version_mismatch, too_large or digest_mismatch.
