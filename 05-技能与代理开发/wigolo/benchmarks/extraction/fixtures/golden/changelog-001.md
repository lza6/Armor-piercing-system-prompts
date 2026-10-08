# Vortex Changelog

All notable changes to Vortex are documented here. This project follows [Semantic Versioning](https://semver.org/).

## [4.0.0] - 2026-04-10

### Breaking Changes

- **Dropped Node.js 18 support.** The minimum required version is now Node.js 20.11.0. Node 18 reached end-of-life in April 2025 and is no longer maintained. If you are still on Node 18, upgrade before updating Vortex.
- **Removed `legacy` transport.** The `createLegacyTransport()` function and the `LegacyTransport` class have been removed. Use `createTransport()` instead, which has been the default since v3.2.0.
- **Changed default serialization format from JSON to MessagePack.** This reduces payload sizes by approximately 30% on average. To continue using JSON, set `serializer: "json"` in your configuration:

  ```typescript
  const client = createClient({
    serializer: "json", // opt back into JSON
  });
  ```

- **Renamed `onError` to `onFailure` in middleware API.** The previous name conflicted with the built-in error event. Update your middleware:

  ```typescript
  // Before
  middleware({ onError: (err) => console.error(err) });

  // After
  middleware({ onFailure: (err) => console.error(err) });
  ```

- **Config file renamed from `.vortexrc` to `vortex.config.ts`.** The old `.vortexrc` JSON format is no longer read. Run `npx vortex migrate-config` to convert automatically.

### Added

- **HTTP/3 transport.** New transport layer using HTTP/3 (QUIC) for significantly lower latency on high-packet-loss networks. Enable with `transport: "h3"`.
- **Plugin system.** Vortex now supports plugins via the `plugins` configuration array. Plugins can hook into the request lifecycle, modify payloads, and add custom transports:

  ```typescript
  import { definePlugin } from "vortex";

  export const loggingPlugin = definePlugin({
    name: "logging",
    onRequest(ctx) {
      console.log(`[${ctx.method}] ${ctx.url}`);
    },
    onResponse(ctx, response) {
      console.log(`[${response.status}] ${ctx.url} (${ctx.duration}ms)`);
    },
  });
  ```

- **Retry with exponential backoff.** Failed requests are now retried up to 3 times with jittered exponential backoff. Configure via `retry` option:

  ```typescript
  const client = createClient({
    retry: {
      maxAttempts: 5,
      baseDelay: 200,
      maxDelay: 10000,
    },
  });
  ```

- **Request deduplication.** Identical in-flight GET requests are automatically deduplicated. The second caller receives the same promise as the first, avoiding redundant network calls.
- **OpenTelemetry integration.** Built-in span creation for distributed tracing. Requires `@opentelemetry/api` as a peer dependency:

  ```typescript
  import { withTracing } from "vortex/otel";

  const client = createClient({
    plugins: [withTracing({ serviceName: "my-api" })],
  });
  ```

- **`client.stream()` method.** Server-Sent Events (SSE) and streaming responses are now first-class:

  ```typescript
  const stream = client.stream("/events", { topic: "deployments" });

  for await (const event of stream) {
    console.log(event.type, event.data);
  }
  ```

### Changed

- **Default timeout increased from 10s to 30s.** The previous 10-second default caused excessive timeouts for users on slower networks. Override with `timeout: 10_000` to restore the old behavior.
- **Improved TypeScript inference for route parameters.** Path parameters like `/users/:id` now infer the parameter type automatically when using the typed client.
- **Connection pooling uses HTTP/2 multiplexing by default.** Multiple concurrent requests to the same host share a single TCP connection, reducing handshake overhead.
- **Error messages now include the request URL and method.** Previously, transport errors only contained the status code.

### Fixed

- Fixed memory leak when using `client.stream()` with large payloads. The internal buffer was not flushed after each event, causing memory to grow unbounded over long-lived connections. ([#1847](https://github.com/example/vortex/issues/1847))
- Fixed race condition in request deduplication where two requests with the same key but different headers could return incorrect cached results. ([#1852](https://github.com/example/vortex/issues/1852))
- Fixed `Content-Type` header not being set correctly for `FormData` payloads on Node.js. The boundary string was missing from the header value. ([#1861](https://github.com/example/vortex/issues/1861))
- Fixed crash when calling `client.close()` while requests were in-flight. Pending requests now resolve with an `AbortError` instead of crashing the process. ([#1870](https://github.com/example/vortex/issues/1870))

### Deprecated

- **`client.rawFetch()` is deprecated.** Use `client.request()` with `raw: true` instead. `rawFetch()` will be removed in v5.0.0.
- **`interceptors` config option is deprecated.** Use `plugins` instead. Interceptors will be removed in v5.0.0.

### Migration Guide

For a detailed migration guide from v3 to v4, see [https://vortex.example.com/docs/migration/v4](https://vortex.example.com/docs/migration/v4).

---

## [3.8.2] - 2026-03-28

### Fixed

- Fixed incorrect URL encoding for query parameters containing special characters (`+`, `&`, `=`). ([#1839](https://github.com/example/vortex/issues/1839))
- Fixed TypeScript type for `headers` option not accepting `Headers` instances. ([#1841](https://github.com/example/vortex/issues/1841))
- Fixed `keepAlive` option being ignored when set to `false`. ([#1843](https://github.com/example/vortex/issues/1843))

---

## [3.8.1] - 2026-03-14

### Fixed

- Fixed regression where `baseURL` with a trailing slash produced double slashes in the final URL. ([#1835](https://github.com/example/vortex/issues/1835))
- Fixed `AbortController` not being cleaned up after successful requests, causing a slow memory leak in long-running processes. ([#1836](https://github.com/example/vortex/issues/1836))

---

## [3.8.0] - 2026-03-01

### Added

- **`client.head()` method.** Convenience method for HEAD requests.
- **`throwOnError` option.** When set to `false`, HTTP error responses resolve normally instead of throwing. Useful for APIs that use 4xx codes for expected conditions:

  ```typescript
  const response = await client.get("/users/maybe-exists", {
    throwOnError: false,
  });

  if (response.status === 404) {
    // Handle missing user without try/catch
  }
  ```

- **Request timing information.** Response objects now include a `timing` property with DNS, connect, TLS, and transfer durations in milliseconds.

### Changed

- Improved error messages for network failures. Errors now distinguish between DNS resolution failures, connection refused, and timeouts.
- The `baseURL` option now accepts a `URL` object in addition to strings.

### Fixed

- Fixed `onDownloadProgress` not being called for small responses that fit in a single chunk. ([#1828](https://github.com/example/vortex/issues/1828))
- Fixed incorrect `Content-Length` calculation for string bodies containing multi-byte UTF-8 characters. ([#1830](https://github.com/example/vortex/issues/1830))

---

## [3.7.0] - 2026-02-01

### Added

- **Proxy support.** HTTP and SOCKS5 proxies can be configured via the `proxy` option or the `HTTPS_PROXY` environment variable:

  ```typescript
  const client = createClient({
    proxy: "http://proxy.internal.example.com:8080",
  });
  ```

- **`client.options()` method** for CORS preflight requests.
- **Custom DNS resolver.** Override DNS resolution for testing or service mesh integration:

  ```typescript
  const client = createClient({
    dns: {
      resolve: async (hostname) => "127.0.0.1",
    },
  });
  ```

### Changed

- Reduced bundle size by 18% (42KB to 34KB gzipped) by replacing internal URL parsing with the native `URL` API.
- `client.get()` and `client.head()` now reject with a `TypeError` if a request body is provided, matching the `fetch` specification.

### Fixed

- Fixed `client.patch()` sending the wrong `Content-Type` header when the body was a plain object. ([#1812](https://github.com/example/vortex/issues/1812))
- Fixed redirect loop detection not accounting for URL fragments. ([#1815](https://github.com/example/vortex/issues/1815))

---

## [3.6.0] - 2026-01-15

### Added

- **Response caching.** GET responses can be cached in memory with configurable TTL:

  ```typescript
  const client = createClient({
    cache: {
      ttl: 60_000,
      maxEntries: 500,
    },
  });
  ```

- **`client.paginate()` helper.** Automatically follows paginated API responses:

  ```typescript
  const allUsers = await client.paginate("/users", {
    getNextUrl: (response) => response.data.next_page_url,
  });
  ```

### Changed

- Default `User-Agent` header now includes the Vortex version (`Vortex/3.6.0`).
- JSON parsing errors now include the first 200 characters of the response body for easier debugging.

---

## [3.5.0] - 2025-12-01

### Added

- **Middleware stack.** Request and response middleware for cross-cutting concerns like authentication, logging, and metrics:

  ```typescript
  client.use(async (ctx, next) => {
    ctx.headers.set("X-Request-ID", crypto.randomUUID());
    const response = await next(ctx);
    console.log(`${ctx.method} ${ctx.url} -> ${response.status}`);
    return response;
  });
  ```

- **File upload progress.** `onUploadProgress` callback for monitoring large file uploads.
- **Brotli compression support.** Responses compressed with Brotli are automatically decompressed.

### Fixed

- Fixed `timeout` option not applying to the TLS handshake phase. ([#1790](https://github.com/example/vortex/issues/1790))
- Fixed incorrect `Content-Type` detection for `.mjs` file uploads. ([#1793](https://github.com/example/vortex/issues/1793))

---

## Links

- [Documentation](https://vortex.example.com/docs)
- [GitHub Repository](https://github.com/example/vortex)
- [Issue Tracker](https://github.com/example/vortex/issues)
- [Migration Guides](https://vortex.example.com/docs/migration)

[4.0.0]: https://github.com/example/vortex/compare/v3.8.2...v4.0.0
[3.8.2]: https://github.com/example/vortex/compare/v3.8.1...v3.8.2
[3.8.1]: https://github.com/example/vortex/compare/v3.8.0...v3.8.1
[3.8.0]: https://github.com/example/vortex/compare/v3.7.0...v3.8.0
[3.7.0]: https://github.com/example/vortex/compare/v3.6.0...v3.7.0
[3.6.0]: https://github.com/example/vortex/compare/v3.5.0...v3.6.0
[3.5.0]: https://github.com/example/vortex/releases/tag/v3.5.0
