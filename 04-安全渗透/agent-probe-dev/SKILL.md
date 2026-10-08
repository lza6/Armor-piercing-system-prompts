---
name: agent-probe-dev
description: Develop, test and ship the Go agent probe (apps/agent) — build/test commands, protocol versioning, the nekoagent service wrapper, and constraints of router environments (OpenWrt/tmpfs). Use for any change under apps/agent.
---

# Agent Probe Development (apps/agent)

The agent is a Go binary deployed on routers/servers. It polls the local gateway (mihomo/Surge) and reports to the collector over **HTTP** (gzip JSON POST to `/api/agent/report|heartbeat|config|policy-state`) — not WebSocket.

## Layout

- `main.go` — entrypoint, flag parsing
- `internal/agent/runner.go` — main loop: report/heartbeat/config-sync tickers, pending queue, retry batch
- `internal/gateway/` — mihomo & Surge API clients
- `internal/config/config.go` — flags, `AgentProtocolVersion`
- `nekoagent` — POSIX-sh service manager wrapper (install/upgrade/systemd/procd/launchd)
- `install.sh` — curl-pipe installer

## Verify

```bash
cd apps/agent
go vet ./... && go build ./... && go test ./...
sh -n nekoagent && sh -n install.sh   # shell syntax gate — both scripts must parse with POSIX sh
```

## Hard rules

1. **Protocol versioning.** Payload shape changes require bumping `AgentProtocolVersion` (Go) AND the collector's minimum accepted version (agent auth in `apps/collector/src/modules/app/app.ts`); the collector rejects too-old agents with HTTP 426 + structured error. The Go structs and the collector's TS payload types are mirrored by hand — change both sides in one commit.
2. **Data-loss protections are contracts:** single in-flight retry batch, `requestId` idempotency dedup, counter-regression handling (gateway restart counts current value as new traffic), bounded pending queue with `dropped` accounting. Don't weaken these when refactoring the loop.
3. **Router constraints.** Target environments include OpenWrt/BusyBox: `nekoagent` must stay POSIX sh (no bashisms), `/tmp` is tmpfs (RAM) — clean up temp dirs (accumulate EXIT traps, don't overwrite them), and network calls in shell must carry timeouts + retries (`curl --connect-timeout 10 --max-time 120 --retry 3`).
4. **Self-replacement.** Never overwrite a running script/binary in place — download to a sibling `.tmp`/`.new` path and `mv` (same-filesystem rename) so the running inode survives.
5. **Service management.** systemd unit: `StartLimitIntervalSec/Burst` belong in `[Unit]`, not `[Service]`. procd (OpenWrt) respawn has a finite retry budget — treat "cannot start" as exit non-zero so supervisors behave correctly.

## Release

Tag-driven, independent from the main product: `git tag agent-vX.Y.Z && git push origin agent-vX.Y.Z` → `.github/workflows/agent-release.yml` publishes multi-arch binaries (~1 min). The binary version comes from the tag via ldflags — no version file. Full flow: `docs/agent/release.en.md`.

Known deferred issues and their context live in `docs/dev/deep-review-2026-07.md` (agent P0/P1 section) — check it before starting a fix batch so related items ship together.
