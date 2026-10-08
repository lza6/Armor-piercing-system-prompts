# Audit Logging

Audit logging for compliance, security monitoring, and troubleshooting.

---

## Overview

Audit logging is **enabled by default** and captures all significant events for:

- Compliance requirements (SOC2, HIPAA, etc.)
- Security monitoring
- Debugging and troubleshooting
- Usage analytics

---

## Disabling Audit Logging

Audit logging is on by default. To disable it:

```bash
export LOKI_AUDIT_DISABLED=true
```

The legacy variable `LOKI_ENTERPRISE_AUDIT=true` still works and will force audit
logging on regardless of `LOKI_AUDIT_DISABLED`.

### Configuration File

```yaml
# .loki/config.yaml
enterprise:
  audit:
    enabled: true    # true is the default
    level: info
    retention_days: 90
```

---

## Logged Events

### Session Events

| Event | Description |
|-------|-------------|
| `session.start` | Session started |
| `session.stop` | Session stopped |
| `session.pause` | Session paused |
| `session.resume` | Session resumed |
| `session.complete` | Session completed successfully |
| `session.fail` | Session failed |

### API Events

| Event | Description |
|-------|-------------|
| `api.request` | API request received |
| `api.response` | API response sent |
| `api.error` | API error occurred |

### Authentication Events

| Event | Description |
|-------|-------------|
| `auth.token.create` | Token created |
| `auth.token.use` | Token used |
| `auth.token.revoke` | Token revoked |
| `auth.fail` | Authentication failed |

### Task Events

| Event | Description |
|-------|-------------|
| `task.create` | Task created |
| `task.start` | Task started |
| `task.complete` | Task completed |
| `task.fail` | Task failed |

---

## Log Format

Audit logs use JSON Lines format:

```json
{
  "timestamp": "2026-02-02T12:00:00.000Z",
  "event": "session.start",
  "level": "info",
  "actor": "user",
  "details": {
    "prd": "./prd.md",
    "provider": "claude",
    "parallel": false
  },
  "metadata": {
    "hostname": "dev-machine",
    "pid": 12345,
    "version": "5.25.0"
  }
}
```

---

## Log Location

```bash
# View audit log directory
ls ~/.loki/dashboard/audit/

# Files are rotated daily
audit-2026-02-01.jsonl
audit-2026-02-02.jsonl
```

---

## CLI Commands

### View Summary

```bash
loki enterprise audit summary
```

Output:
```
Audit Log Summary (Last 24 Hours)

Events by Type:
  session.start:    5
  session.complete: 4
  session.fail:     1
  api.request:     42
  auth.token.use:  15

Events by Level:
  info:    58
  warning:  3
  error:    1
```

### Tail Recent Entries

```bash
# Last 20 entries
loki enterprise audit tail

# Follow new entries
loki enterprise audit tail --follow

# Filter by event type
loki enterprise audit tail --event session.start
```

### Search Logs

```bash
# Search by event
loki enterprise audit search --event auth.fail

# Search by date range
loki enterprise audit search --from 2026-02-01 --to 2026-02-02

# Search by actor
loki enterprise audit search --actor ci-bot
```

### Export Logs

```bash
# Export to file
loki enterprise audit export --output audit-export.json

# Export with filters
loki enterprise audit export --from 2026-01-01 --level error
```

---

## API Endpoints

### Get Audit Entries

```bash
curl "http://localhost:57374/audit?limit=50"
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `start` | ISO date | Start timestamp |
| `end` | ISO date | End timestamp |
| `event` | string | Filter by event type |
| `level` | string | Filter by level |
| `actor` | string | Filter by actor |
| `limit` | number | Max results |
| `offset` | number | Pagination offset |

### Get Summary

```bash
curl http://localhost:57374/audit/summary
```

---

## Configuration

### Full Options

```yaml
# .loki/config.yaml
enterprise:
  audit:
    enabled: true
    level: info          # Minimum level: debug, info, warning, error
    retention_days: 90   # Days to keep logs
    max_file_size: 100   # MB per file before rotation
    compress: true       # Compress rotated files
    exclude_events:      # Events to exclude
      - api.request
    include_metadata:    # Additional metadata
      - environment
      - deployment_id
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `LOKI_AUDIT_DISABLED` | `false` | Set to `true` to disable audit logging |
| `LOKI_ENTERPRISE_AUDIT` | `false` | Force audit on (legacy, audit is now on by default) |
| `LOKI_AUDIT_LEVEL` | `info` | Minimum log level |
| `LOKI_AUDIT_RETENTION` | `90` | Retention in days |
| `LOKI_AUDIT_SYSLOG_HOST` | - | Syslog server hostname for audit forwarding |
| `LOKI_AUDIT_SYSLOG_PORT` | `514` | Syslog server port |
| `LOKI_AUDIT_SYSLOG_PROTO` | `udp` | Syslog protocol: `udp` or `tcp` |
| `LOKI_AUDIT_NO_INTEGRITY` | `false` | Disable SHA-256 chain hashing on audit entries |

---

## Log Levels

| Level | Description | Examples |
|-------|-------------|----------|
| `debug` | Detailed debugging | Internal state changes |
| `info` | Normal operations | Session start/stop |
| `warning` | Potential issues | Rate limiting, retries |
| `error` | Errors | Auth failures, task failures |

---

## SIEM Integration

### Forwarding to External Systems

Export logs for ingestion by SIEM systems:

```bash
# Export in SIEM-compatible format
loki enterprise audit export --format syslog > /var/log/loki-audit.log

# Stream to syslog
loki enterprise audit tail --format syslog | logger -t loki-mode
```

### Splunk

```bash
# Configure Splunk forwarder to monitor
/opt/splunk/bin/splunk add monitor ~/.loki/dashboard/audit/
```

### Datadog

```yaml
# datadog.yaml
logs:
  - type: file
    path: /home/user/.loki/dashboard/audit/*.jsonl
    source: loki-mode
    service: loki-mode
```

### CloudWatch

```bash
# Install CloudWatch agent
aws logs create-log-group --log-group-name loki-mode-audit

# Configure agent to push logs
```

---

## Log Integrity (v5.38.0)

Audit entries are chain-hashed with SHA-256 for tamper detection.

### How It Works

Each audit entry includes a `chain_hash` field:
1. First entry hashes against a genesis hash (`0` * 64)
2. Each subsequent entry hashes: `SHA256(previous_hash + current_entry_json)`
3. Any modification to a past entry invalidates all subsequent hashes

### Verification

```python
from dashboard.audit import verify_log_integrity

result = verify_log_integrity("/path/to/audit.jsonl")
print(f"Valid: {result['valid']}")
print(f"Entries checked: {result['entries_checked']}")
if not result['valid']:
    print(f"First tampered line: {result['first_tampered_line']}")
```

### Disabling Chain Hashing

```bash
export LOKI_AUDIT_NO_INTEGRITY=true
```

---

## Syslog Forwarding (v5.37.1)

Forward audit events to external syslog servers for SIEM integration.

### Enable Syslog

```bash
export LOKI_AUDIT_SYSLOG_HOST=syslog.example.com
export LOKI_AUDIT_SYSLOG_PORT=514
export LOKI_AUDIT_SYSLOG_PROTO=udp
```

### Details

- Uses Python stdlib `logging.handlers.SysLogHandler`
- Facility: `LOG_LOCAL0`
- Security actions forwarded at `WARNING` level
- Fire-and-forget: syslog failures do not block audit writes
- Supports both UDP and TCP protocols

### Example syslog-ng Configuration

```
source s_loki {
    network(port(514) transport("udp"));
};

destination d_loki_audit {
    file("/var/log/loki-mode-audit.log");
};

filter f_loki {
    facility(local0);
};

log {
    source(s_loki);
    filter(f_loki);
    destination(d_loki_audit);
};
```

---

## Agent Action Audit (v5.38.0)

In addition to the dashboard audit log, agent actions are tracked in a separate JSONL file.

### Location

`.loki/logs/agent-audit.jsonl`

### Tracked Actions

| Action | Description |
|--------|-------------|
| `cli_invoke` | CLI command executed by agent |
| `git_commit` | Git commit performed by agent |
| `session_start` | Agent session started |
| `session_stop` | Agent session stopped |

### Entry Format

```json
{
  "timestamp": "2026-02-12T18:30:00Z",
  "action": "git_commit",
  "agent": "development",
  "details": {"message": "Add auth module", "files_changed": 3}
}
```

### CLI Commands

```bash
loki audit log      # View recent entries
loki audit count    # Count total entries
loki audit help     # Show help
```

---

## Compliance

### SOC2

Audit logging supports SOC2 requirements:

- **CC6.1** - Logical access security
- **CC7.2** - System monitoring
- **CC7.3** - Incident response

### HIPAA

For healthcare applications:

- Enable all authentication events
- Set retention to minimum 6 years
- Enable log encryption

```yaml
enterprise:
  audit:
    enabled: true
    retention_days: 2190  # 6 years
    encrypt: true
```

---

## Troubleshooting

### Logs Not Being Created

```bash
# Check if enabled
loki enterprise status

# Verify directory permissions
ls -la ~/.loki/dashboard/audit/

# Check disk space
df -h ~/.loki/
```

### Missing Events

```bash
# Check minimum level
loki enterprise audit summary

# Lower level to capture more
export LOKI_AUDIT_LEVEL=debug
```

### Disk Space Issues

```bash
# Check current usage
du -sh ~/.loki/dashboard/audit/

# Manually clean old logs
find ~/.loki/dashboard/audit/ -name "*.jsonl" -mtime +30 -delete
```

---

## See Also

- [[Enterprise Features]] - All enterprise features
- [[Security]] - Security best practices
- [[API Reference]] - Audit API endpoints
- [[Network Security]] - Network egress control
