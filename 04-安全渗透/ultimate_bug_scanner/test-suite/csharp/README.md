# C# Fixtures

- `buggy/` contains intentionally unsafe patterns for `ubs-csharp.sh`.
- `clean/` provides counterexamples that should stay free of critical/warning findings.
- `security/ArchiveExtractionBuggy.cs` and `security/ArchiveExtractionClean.cs` cover zip/tar archive entry path containment.
- `security/OpenRedirectBuggy.cs` and `security/OpenRedirectClean.cs` cover ASP.NET request/header/cookie/route values flowing into redirect and `Location` header sinks.
- `security/HeaderInjectionBuggy.cs` and `security/HeaderInjectionClean.cs` cover ASP.NET request/query/header/form and annotated action values flowing into response headers.
- `security/RequestPathTraversalBuggy.cs` and `security/RequestPathTraversalClean.cs` cover ASP.NET request/header/upload values flowing into file read/write/serve/delete sinks.
- `security/SsrfBuggy.cs` and `security/SsrfClean.cs` cover ASP.NET request/header values flowing into outbound HTTP clients.
- `tests/test_helper_scanners.py` covers the helper-backed type narrowing, resource lifecycle, and async task-handle analyzers directly.
- `manifest.json` now also includes a shimmed ast-grep regression case so the AST rule pack stays testable even when `ast-grep` is not installed globally.
- Manifest cases run with `--no-dotnet` so scanner regressions stay stable even when the .NET SDK is absent.
