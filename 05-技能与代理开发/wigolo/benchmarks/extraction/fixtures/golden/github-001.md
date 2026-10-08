# fasthttp

Fast HTTP implementation for Go.

[![Build Status](https://github.com/valyala/fasthttp/actions/workflows/ci.yml/badge.svg)](https://github.com/valyala/fasthttp/actions)
[![Go Report Card](https://goreportcard.com/badge/github.com/valyala/fasthttp)](https://goreportcard.com/report/github.com/valyala/fasthttp)
[![GoDoc](https://godoc.org/github.com/valyala/fasthttp?status.svg)](https://godoc.org/github.com/valyala/fasthttp)

## Overview

fasthttp is a high-performance HTTP library for Go designed to handle thousands of small to medium requests per second. It is up to 10x faster than `net/http` for common workloads.

## Features

- Optimized HTTP request/response handling
- Zero-allocation design for hot paths
- Connection pooling and reuse
- Built-in support for HTTP pipelining
- Streaming request/response bodies
- WebSocket support via [fasthttp/websocket](https://github.com/fasthttp/websocket)
- TLS support
- Timeouts for all operations

## Installation

```bash
go get -u github.com/valyala/fasthttp
```

## Quick Start

### Server

```go
package main

import (
    "fmt"
    "log"

    "github.com/valyala/fasthttp"
)

func main() {
    requestHandler := func(ctx *fasthttp.RequestCtx) {
        fmt.Fprintf(ctx, "Hello, world!\n\n")

        fmt.Fprintf(ctx, "Request method: %s\n", ctx.Method())
        fmt.Fprintf(ctx, "Request URI:    %s\n", ctx.RequestURI())
        fmt.Fprintf(ctx, "Request path:   %s\n", ctx.Path())
        fmt.Fprintf(ctx, "Host:           %s\n", ctx.Host())
        fmt.Fprintf(ctx, "User-Agent:     %s\n", ctx.UserAgent())

        ctx.SetContentType("text/plain; charset=utf8")
        ctx.Response.Header.Set("X-Custom-Header", "fasthttp")
    }

    if err := fasthttp.ListenAndServe(":8080", requestHandler); err != nil {
        log.Fatalf("Error in ListenAndServe: %v", err)
    }
}
```

### Client

```go
package main

import (
    "fmt"
    "log"

    "github.com/valyala/fasthttp"
)

func main() {
    status, body, err := fasthttp.Get(nil, "https://httpbin.org/get")
    if err != nil {
        log.Fatalf("Error: %v", err)
    }
    fmt.Printf("Status: %d\n", status)
    fmt.Printf("Body:\n%s\n", body)
}
```

## Benchmarks

Comparison with `net/http` on a 4-core machine:

| Library | Requests/sec | Latency p99 | Allocs/op |
|---------|-------------|-------------|-----------|
| fasthttp | 245,000 | 1.2ms | 0 |
| net/http | 38,000 | 8.5ms | 11 |

## API Reference

### Server

| Method | Description |
|--------|-------------|
| `ListenAndServe(addr, handler)` | Start HTTP server |
| `ListenAndServeTLS(addr, cert, key, handler)` | Start HTTPS server |
| `Serve(ln, handler)` | Serve on existing listener |

### RequestCtx

| Method | Description |
|--------|-------------|
| `ctx.Method()` | Get HTTP method |
| `ctx.Path()` | Get request path |
| `ctx.QueryArgs()` | Get query parameters |
| `ctx.PostBody()` | Get POST body |
| `ctx.SetStatusCode(code)` | Set response status |
| `ctx.SetContentType(ct)` | Set Content-Type header |
| `ctx.Write(data)` | Write response body |

### Client

| Method | Description |
|--------|-------------|
| `Get(dst, url)` | Send GET request |
| `Post(dst, url, body)` | Send POST request |
| `Do(req, resp)` | Send custom request |
| `DoTimeout(req, resp, t)` | Send with timeout |

## Common Patterns

### Routing with fasthttp

```go
func requestHandler(ctx *fasthttp.RequestCtx) {
    switch string(ctx.Path()) {
    case "/":
        handleIndex(ctx)
    case "/api/users":
        handleUsers(ctx)
    case "/api/health":
        handleHealth(ctx)
    default:
        ctx.Error("Not found", fasthttp.StatusNotFound)
    }
}
```

### Middleware Pattern

```go
type Middleware func(fasthttp.RequestHandler) fasthttp.RequestHandler

func LogMiddleware(next fasthttp.RequestHandler) fasthttp.RequestHandler {
    return func(ctx *fasthttp.RequestCtx) {
        start := time.Now()
        next(ctx)
        log.Printf("%s %s - %d (%v)",
            ctx.Method(), ctx.Path(),
            ctx.Response.StatusCode(),
            time.Since(start))
    }
}
```

## Migration from net/http

1. Replace `http.HandleFunc` with `fasthttp.RequestHandler`
2. Replace `http.Request` with `fasthttp.RequestCtx`
3. Replace `http.ResponseWriter` with `ctx` methods
4. Replace `r.URL.Path` with `ctx.Path()`
5. Replace `w.Write()` with `ctx.Write()`

## License

MIT License. See [LICENSE](LICENSE) for details.

## Contributing

1. Fork the repo
2. Create your feature branch (`git checkout -b feature/amazing`)
3. Commit your changes (`git commit -am 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing`)
5. Open a Pull Request

## Links

- [Documentation](https://pkg.go.dev/github.com/valyala/fasthttp)
- [Examples](https://github.com/valyala/fasthttp/tree/master/examples)
- [FAQ](https://github.com/valyala/fasthttp/wiki/FAQ)
