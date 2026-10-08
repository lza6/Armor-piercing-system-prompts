# C++ UBS Samples

| File | Focus |
|------|-------|
| `buggy/buggy_raii.cpp` | Missing deletes, manual memory, `new`/`delete` mismatch |
| `buggy/buggy_concurrency.cpp` | Detached threads, shared data races |
| `buggy/resource_lifecycle.cpp` | FILE* leaks, missing `close()` |
| `buggy/security_overflow.cpp` | `strcpy`, `system()` usage, leaks |
| `buggy/math_precision.cpp` | Integer overflow + float equality |
| `security/path_traversal_buggy.cpp` | Request/query/header/path filenames joined into file read/write/delete sinks without containment checks |
| `security/path_traversal_clean.cpp` | Request/header paths passed through canonical root checks or reduced to a basename before file sinks |
| `security/open_redirect_buggy.cpp` | Request/query/header/host redirect targets reach redirects or `Location` headers without local-path or host allow-list validation |
| `security/open_redirect_clean.cpp` | Request redirect targets passed through safe helper validation or inline local-path checks before redirect sinks |
| `security/ssrf_buggy.cpp` | CGI query/header URL values configured as libcurl request targets without scheme/host allow-list checks |
| `security/ssrf_clean.cpp` | CGI-derived outbound URLs passed through a safe helper with explicit scheme and host allow-list validation before libcurl |
| `archive_extraction_buggy/zip_slip.cpp` | libarchive/libzip/minizip/miniz entry names joined to destination paths without containment checks |
| `archive_extraction_clean/zip_slip_safe.cpp` | Archive entries canonicalized and checked against the extraction root before writes |
| Clean files (`clean/*.cpp`) | RAII, smart pointers, bounded math |

Run C++ scans with:

```bash
ubs --only=cpp --fail-on-warning test-suite/cpp/buggy
ubs --only=cpp test-suite/cpp/clean
```
