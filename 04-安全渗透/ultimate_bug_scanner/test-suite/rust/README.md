# Rust UBS Samples

| File | Category |
|------|----------|
| `buggy/buggy_unwrap.rs` | Panic-prone unwrap chains |
| `buggy/async_block.rs` | Spawned tasks never awaited |
| `buggy/blocking_async.rs` | Blocking sleep/fs/thread operations inside async functions |
| `buggy/resource_lifecycle.rs` | Missing JoinHandle cleanup |
| `buggy/security_injection.rs` | Command injection + exposed secrets |
| `buggy/archive_extraction.rs` | Archive member paths joined into extraction destinations |
| `buggy/temp_file_race.rs` | Predictable temp-file writes in shared temp directories |
| `buggy/open_redirect.rs` | Request/header/host redirect targets sent to redirects or Location headers |
| `buggy/header_injection.rs` | Request/header values sent to non-Location response headers without CR/LF validation |
| `buggy/ssrf.rs` | Request/header/env/CLI URL values sent through outbound HTTP clients |
| `buggy/tls_verification.rs` | TLS certificate or hostname verification disabled directly or through true constants |
| `buggy/sql_injection.rs` | Request-derived values interpolated into raw SQL strings that reach execution sinks |
| `buggy/cors_credentials.rs` | Credentialed wildcard or reflected-origin CORS policies |
| `buggy/math_precision.rs` | Float equality for money |
| Clean files (`clean/*.rs`) | `Result` handling, JoinHandle waiting, integer cents, safe temp-file creation, safe redirect validation, safe response-header values, safe outbound URL validation, TLS verification kept enabled, parameterized SQL, safe CORS origin allow-lists |

```bash
ubs --only=rust --fail-on-warning test-suite/rust/buggy
ubs --only=rust test-suite/rust/clean
```
