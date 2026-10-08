# Ruby UBS Samples

| File | Category |
|------|----------|
| `buggy/security_issues.rb` | eval, unsafe YAML |
| `buggy/resource_lifecycle.rb` | File/thread cleanup |
| `buggy/buggy_scripts.rb` | Shelling out, missing rescue |
| `buggy/performance.rb` | Thread leaks, backticks |
| `archive_extraction_buggy/zip_slip.rb` | RubyZip/TarReader/Minitar path traversal |
| `archive_extraction_clean/zip_slip_safe.rb` | `File.expand_path`/`Pathname` + `start_with?` containment checks |
| `open_redirect_buggy/redirects.rb` | Rack/Rails params, headers, referers, and host values flowing to redirect sinks |
| `open_redirect_clean/redirects.rb` | Safe redirect helpers, local path checks, `url_from`, and `allow_other_host: false` |
| `header_injection_buggy/headers.rb` | Rack/Rails params, headers, cookies, and env values flowing to non-`Location` response headers |
| `header_injection_clean/headers.rb` | CR/LF stripping, CR/LF rejection, encoded filenames, and `Location` coverage routed to open-redirect checks |
| `path_traversal_buggy/request_paths.rb` | Rack/Rails params and upload names flowing to file read/write/serve/delete sinks |
| `path_traversal_clean/request_paths.rb` | `File.basename` upload names and `File.expand_path` + `start_with?` containment checks |
| `ssrf_buggy/request_urls.rb` | Rack/Rails params, headers, and request host accessors flowing into Ruby outbound HTTP clients |
| `ssrf_clean/request_urls.rb` | `URI.parse` + scheme/host allow-list validation before outbound HTTP clients, including URLs assembled from inbound host values |
| Clean files | Managed threads, Open3 argv |

```bash
ubs --only=ruby --fail-on-warning test-suite/ruby/buggy
ubs --only=ruby test-suite/ruby/clean
```
