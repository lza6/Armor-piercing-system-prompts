# Forwarding audit events

The audit file (`audit.jsonl`, see [audit.md](audit.md)) is always written and is the source of
truth. Sinks forward the same events, as they happen, to a syslog server or an HTTP collector for
installs that have no log shipper on the Hermes machine.

## Configuration

`plugins.entries.netbox.settings.audit_sinks` is a list. Any mix of sinks works, and each event goes
to all of them.

```yaml
plugins:
  entries:
    netbox:
      settings:
        audit_sinks:
          - type: syslog
            host: 10.0.0.5
            port: 514
            protocol: udp        # udp | tcp | tls
            format: json         # json | cef
            facility: local0     # any standard facility name
            app_name: hermes-netbox
          - type: http
            url: https://splunk.example.com:8088/services/collector/event
            headers:
              Authorization: "Splunk ${SPLUNK_HEC_TOKEN}"
            format: hec          # json | hec
            timeout: 5
```

`${VAR}` in a header value is replaced from the environment at send time, so the token lives in
`~/.hermes/.env` and never in `config.yaml`.

| Sink option | Meaning |
|---|---|
| `protocol` | `udp` (default), `tcp`, or `tls`. TLS uses the system trust store; `ca_file` points at a private CA, `verify_ssl: false` disables verification for a lab. |
| `framing` | TCP framing: `newline` (default for tcp, what rsyslog, syslog-ng, Graylog and Wazuh accept) or `octet-counting` (RFC 6587, default and required for tls per RFC 5425). |
| `format` | `json`: the audit record as the syslog message body. `cef`: ArcSight Common Event Format, below. |
| `facility` | Syslog facility name, default `local0`. |
| `timeout` | Connect and send timeout in seconds, default 5. |
| `verify_ssl` | For `http` sinks over HTTPS and `tls` syslog. Default true. |
| `sourcetype` | HEC only, default `hermes:netbox`. |

## Delivery rules

- Events are queued and sent from one background thread. A NetBox operation never waits for a sink.
- Each event is tried three times per sink with a short backoff, then dropped for that sink. The
  file still has it.
- The queue holds 1,000 events. If a sink is down for long enough to fill it, further events are
  dropped for the sinks, never for the file. Failures are logged once per sink every five minutes,
  not once per event.
- On shutdown the plugin waits up to two seconds for the queue to drain.

Sinks are for convenience and near-real-time alerting. If you need guaranteed delivery, ship the
file with Filebeat, Fluent Bit, Vector or a forwarder, which buffer to disk.

## Syslog details

Messages are RFC 5424:

```
<134>1 2026-09-08T19:35:50Z hermes-01 hermes-netbox 4242 step_done - {"ts":"2026-09-08T19:35:50Z","event":"step_done",…}
```

`PRI` is facility × 8 + severity. `MSGID` is the event name. The message body is the full JSON
record, so a receiver that parses JSON gets every field.

Severity mapping:

| Severity | Events |
|---|---|
| warning (4) | `plan_rejected`, `apply_refused`, `rollback_refused`, `step_failed`, `step_conflict`, `revert_failed`, `revert_conflict`, `apply_finished` with a failed outcome, `rollback_finished` that is not fully rolled back, `changelog_linked` that could not link |
| notice (5) | `apply_started`, `rollback_started`, `plans_pruned` |
| informational (6) | everything else |

## CEF

With `format: cef` the body is one Common Event Format line, which ArcSight, Splunk, QRadar,
Sentinel, Graylog and Wazuh parse natively:

```
CEF:0|andrewvieyra|hermes-plugin-netbox|0.4.0|apply_refused|apply refused|7|rt=1757360150000 dvchost=hermes-01 suser=Andrew cs1Label=plan_id cs1=nbp-… cs2Label=actor_kind cs2=model cs3Label=via cs3=tool cs4Label=platform cs4=signal cs5Label=chat cs5=Andrew msg=write_mode is read_only
```

| CEF key | Source |
|---|---|
| `rt` | event time, epoch milliseconds |
| `dvchost`, `dvcpid` | host and pid |
| `request` | `netbox_url` |
| `suser` | actor user name or id |
| `act` | step action (`create`, `update`, `delete`) |
| `externalId` | NetBox object id |
| `cn1` | HTTP status |
| `outcome` | apply outcome or rollback status |
| `msg` | error, reason or description |
| `cs1` … `cs7` | plan id, actor kind, via, platform, chat, endpoint, request text |

CEF severity is 7 for warnings, 4 for notices, 2 for informational.

## HTTP details

`format: json` posts the record as the body. `format: hec` wraps it for Splunk's HTTP Event
Collector: `{"time": <epoch>, "sourcetype": "hermes:netbox", "host": …, "event": <record>}`. Any
collector that accepts a JSON POST with a static header works with `format: json`.

## Receiver examples

- **rsyslog**: `module(load="imudp") input(type="imudp" port="514")`, or `imtcp` for TCP. The
  message body starts with `{`, so `mmjsonparse` can parse it when preceded by `@cee: ` … simpler:
  store the raw line and let your search tool parse the JSON.
- **Graylog**: a Syslog UDP or TCP input; enable "Store full message". Add a JSON extractor on the
  `message` field.
- **Wazuh**: a `<remote>` syslog block on the manager; then a decoder for `hermes-netbox`.
- **Splunk HEC**: create an HEC token with sourcetype `hermes:netbox`; use `format: hec`.
- **Elastic**: point Filebeat at the file instead; it maps `ts` to `@timestamp` with one processor.

TLS syslog is implemented with the standard library's certificate verification. It is not
covered by an automated test; verify it against your receiver before relying on it.
