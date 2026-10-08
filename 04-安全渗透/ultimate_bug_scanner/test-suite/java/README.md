# Java UBS Samples

| File | Focus |
|------|-------|
| `buggy/BuggyService.java` | Blocking I/O, TLS issues |
| `buggy/BuggyConcurrency.java` | ExecutorService leaks, missing shutdown |
| `buggy/ResourceLifecycle.java` | Streams not closed |
| `buggy/BuggySecurity.java` | SQL/command injection |
| `security/path_traversal_buggy.java` | request parameters, annotated headers, and upload filenames reaching file read/write/delete sinks without containment checks |
| `security/path_traversal_clean.java` | canonical root containment and basename extraction before file sinks, including annotated header paths |
| `security/SsrfBuggy.java` | request parameters, headers, annotated request-header parameters, and servlet host accessors reaching outbound HTTP clients |
| `security/SsrfClean.java` | safe outbound URL helpers with scheme and host allow-list validation, including annotated headers |
| `security/OpenRedirectBuggy.java` | request parameters, headers, and annotated params reaching servlet/Spring redirect sinks without validation |
| `security/OpenRedirectClean.java` | redirect targets routed through same-origin/allow-list helpers before servlet/Spring redirect sinks |
| `security/HeaderInjectionBuggy.java` | request parameters, headers, and annotated params reaching servlet/Spring response headers without CR/LF safety |
| `security/HeaderInjectionClean.java` | CR/LF stripping, URL-encoded filename fragments, and reject-on-newline guards before response headers |
| `security/HeaderInjectionMultilineBuggy.java` | multiline-only servlet response header sink fed by request data |
| `security/HeaderInjectionMultilineClean.java` | multiline-only servlet response header sink after CR/LF stripping |
| `security/ArchiveExtractionBuggy.java` | Archive extraction security |
| `security/ArchiveExtractionClean.java` | normalize + startsWith destination checks |
| Clean files | try-with-resources, prepared statements, ProcessBuilder argv |

```bash
ubs --only=java --fail-on-warning test-suite/java/buggy
ubs --only=java test-suite/java/clean
```
