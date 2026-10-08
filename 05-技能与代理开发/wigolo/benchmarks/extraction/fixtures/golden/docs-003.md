# Configuration - ReadTheDocs Style

## Overview

The configuration system uses a layered approach where settings from multiple sources are merged in a defined priority order.

## Configuration Sources

Settings are loaded from the following sources, in order of increasing priority:

1. Built-in defaults
2. System-wide config file (`/etc/myapp/config.yaml`)
3. User config file (`~/.config/myapp/config.yaml`)
4. Project config file (`./myapp.config.yaml`)
5. Environment variables (`MYAPP_*`)
6. Command-line arguments

## Configuration File Format

### YAML Format

```yaml
# myapp.config.yaml
server:
  host: "0.0.0.0"
  port: 8080
  workers: 4
  timeout: 30s
  max_request_size: "10MB"

database:
  driver: postgres
  host: localhost
  port: 5432
  name: myapp_production
  pool:
    min_connections: 5
    max_connections: 20
    idle_timeout: 300s

logging:
  level: info
  format: json
  output: stderr
  file:
    enabled: false
    path: /var/log/myapp/app.log
    max_size: "100MB"
    max_backups: 5
    compress: true

cache:
  backend: redis
  redis:
    url: "redis://localhost:6379/0"
    prefix: "myapp:"
    ttl: 3600
  memory:
    max_items: 10000
    max_size: "256MB"

auth:
  jwt:
    secret_env: "MYAPP_JWT_SECRET"
    expiration: 3600
    refresh_expiration: 86400
    algorithm: HS256
  oauth:
    enabled: false
    providers: []

rate_limit:
  enabled: true
  window: 60s
  max_requests: 100
  by: ip
  whitelist:
    - "127.0.0.1"
    - "::1"
```

### TOML Format

```toml
[server]
host = "0.0.0.0"
port = 8080
workers = 4
timeout = "30s"

[database]
driver = "postgres"
host = "localhost"
port = 5432
name = "myapp_production"

[database.pool]
min_connections = 5
max_connections = 20
idle_timeout = "300s"
```

## Environment Variables

All configuration options can be set via environment variables using the `MYAPP_` prefix with underscores replacing dots:

| Config Path | Environment Variable | Example |
|-------------|---------------------|---------|
| `server.host` | `MYAPP_SERVER_HOST` | `0.0.0.0` |
| `server.port` | `MYAPP_SERVER_PORT` | `8080` |
| `database.host` | `MYAPP_DATABASE_HOST` | `db.example.com` |
| `database.pool.max_connections` | `MYAPP_DATABASE_POOL_MAX_CONNECTIONS` | `50` |
| `logging.level` | `MYAPP_LOGGING_LEVEL` | `debug` |
| `cache.backend` | `MYAPP_CACHE_BACKEND` | `redis` |
| `auth.jwt.expiration` | `MYAPP_AUTH_JWT_EXPIRATION` | `7200` |

## Command-Line Arguments

```bash
myapp serve --server.port=9090 --logging.level=debug --database.host=remote-db
```

## Programmatic Access

```python
from myapp.config import Config

config = Config.load()

# Access with dot notation
port = config.server.port  # 8080
db_host = config.database.host  # "localhost"

# Access with dictionary syntax
port = config["server"]["port"]  # 8080

# Get with default
workers = config.get("server.workers", default=2)

# Check if key exists
has_cache = config.has("cache.backend")  # True
```

## Validation

The configuration is validated on load against a schema:

```python
from myapp.config import Config, ValidationError

try:
    config = Config.load()
except ValidationError as e:
    print(f"Configuration error: {e}")
    for error in e.errors:
        print(f"  - {error.path}: {error.message}")
```

### Validation Rules

| Field | Type | Required | Default | Constraints |
|-------|------|----------|---------|-------------|
| `server.host` | string | No | `"0.0.0.0"` | Valid IP or hostname |
| `server.port` | integer | No | `8080` | 1-65535 |
| `server.workers` | integer | No | CPU count | 1-256 |
| `server.timeout` | duration | No | `"30s"` | 1s-300s |
| `database.driver` | string | Yes | - | `postgres`, `mysql`, `sqlite` |
| `database.host` | string | Yes | - | Valid hostname |
| `database.port` | integer | No | Driver default | 1-65535 |
| `database.name` | string | Yes | - | Non-empty |
| `database.pool.min_connections` | integer | No | `5` | 1-max_connections |
| `database.pool.max_connections` | integer | No | `20` | min_connections-1000 |
| `logging.level` | string | No | `"info"` | `debug`, `info`, `warn`, `error` |
| `cache.backend` | string | No | `"memory"` | `memory`, `redis`, `memcached` |

## Profiles

Configuration profiles allow different settings for different environments:

```yaml
# myapp.config.yaml
profiles:
  development:
    server:
      port: 3000
    logging:
      level: debug
      format: text
    database:
      name: myapp_dev

  staging:
    server:
      port: 8080
    logging:
      level: info
    database:
      name: myapp_staging

  production:
    server:
      workers: 8
    logging:
      level: warn
      format: json
    database:
      pool:
        max_connections: 50
```

Activate a profile:

```bash
MYAPP_PROFILE=production myapp serve
# or
myapp serve --profile=production
```

## Hot Reloading

Configuration changes can be applied without restarting:

```python
config = Config.load(watch=True)

@config.on_change("logging.level")
def on_log_level_change(old_value, new_value):
    logger.setLevel(new_value)
    logger.info(f"Log level changed from {old_value} to {new_value}")
```

Supported hot-reload fields:

- `logging.level`
- `logging.format`
- `rate_limit.*`
- `cache.ttl`

Fields that require restart:

- `server.host`
- `server.port`
- `database.*`

## Secrets Management

Sensitive values should never be stored in config files. Use environment variable references:

```yaml
database:
  password_env: "MYAPP_DB_PASSWORD"

auth:
  jwt:
    secret_env: "MYAPP_JWT_SECRET"
```

Or use a secrets backend:

```yaml
secrets:
  backend: vault
  vault:
    address: "https://vault.example.com"
    path: "secret/data/myapp"
    token_env: "VAULT_TOKEN"
```

## See Also

- [Getting Started Guide](../getting-started)
- [Deployment Guide](../deployment)
- [API Reference](../api-reference)
- [Troubleshooting](../troubleshooting)
