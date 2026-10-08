---
slug: example-spike
date: 10-06-26
verdict: NOT-VIABLE
originating-phase: spec
---

# Feasibility Verdict — Does the gateway forward params.provider.sort?

## Hypothesis
The gateway forwarding layer preserves the `params.provider.sort` field in agent-call requests.

## Mechanism Under Test
The gateway forwarding allowlist for the agent-call endpoint. Specifically whether
`params.provider` is included in the fields forwarded to the upstream model API.

## Probe Family
5 — Container exec / internal-port curl

## Probe Cost Class
`needs-container`. Gate met: ran against a disposable live-E2E container
(`DISPOSABLE_LIVE_E2E=1`), never the shared dev container.

## Probe Method
```bash
docker exec <disposable_container_id> curl -s -X POST http://localhost:3000/v1/messages \
  -H 'Content-Type: application/json' \
  -d '{"model":"claude-3-5-sonnet","params":{"provider":{"sort":"throughput"}}}'
```
Compared outbound request at capture-server with direct API call.

## Evidence Captured
Capture-server log (relevant lines):
```
POST /v1/messages body: {"model":"claude-3-5-sonnet","messages":[...]}
# params.provider key absent — stripped by forwarding layer
```

## Verdict
NOT-VIABLE

## Resulting Design Constraint
- **What this licenses:** nothing new — no approach gains capability from this probe.
- **What this forbids:** any approach that depends on `params.provider.sort` being
  forwarded through the gateway layer. The forwarding layer strips `params.provider` entirely;
  sorting must be implemented at a different layer.
- **What remains uncertain (known-gap):** whether a different forwarding field (e.g.
  `params.route`) survives the allowlist was not probed and remains an open risk.
