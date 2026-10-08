# WebAssembly

**WebAssembly** (abbreviated **Wasm**) is a binary instruction format for a stack-based virtual machine. It is designed as a portable compilation target for programming languages, enabling deployment on the web for client and server applications.

WebAssembly was first announced in 2015 and became a W3C Recommendation on December 5, 2019, alongside HTML, CSS, and JavaScript as the fourth language to run natively in web browsers.

## History

### Origins

The development of WebAssembly began as a collaborative effort between Mozilla, Google, Microsoft, and Apple. It grew out of earlier projects including Mozilla's asm.js and Google's Native Client (NaCl), both of which aimed to run high-performance code in the browser.

- **2013** — Mozilla releases asm.js, a strict subset of JavaScript optimized for ahead-of-time compilation
- **2015** — WebAssembly Community Group formed at W3C; initial design announced jointly by browser vendors
- **2017** — All four major browsers (Chrome, Firefox, Safari, Edge) ship WebAssembly support; MVP specification finalized
- **2019** — W3C officially designates WebAssembly as a web standard
- **2022** — Component Model proposal gains traction, enabling language interoperability
- **2024** — Garbage collection (WasmGC) proposal reaches phase 4 and ships in Chrome and Firefox

### Design Goals

The original design goals of WebAssembly as stated by the W3C Community Group were:

1. **Fast execution** — approaching native speed through hardware-level optimizations
2. **Compact representation** — binary format smaller than minified JavaScript
3. **Safe** — memory-safe sandboxed execution environment
4. **Open** — human-readable text format for debugging and development
5. **Part of the web platform** — interoperable with JavaScript and the DOM

## Technical Overview

### Architecture

WebAssembly defines a virtual instruction set architecture (ISA) based on a structured stack machine. Unlike register-based architectures used by most physical CPUs, Wasm instructions implicitly operate on values at the top of a virtual stack.

A WebAssembly module consists of:

- **Functions** — typed sequences of instructions
- **Tables** — typed arrays of references (used for indirect calls)
- **Memories** — contiguous, byte-addressable linear memory regions
- **Globals** — typed mutable or immutable global variables

### Value Types

WebAssembly 1.0 supports four value types:

| Type  | Description                  | Size    |
|-------|------------------------------|---------|
| `i32` | 32-bit integer               | 4 bytes |
| `i64` | 64-bit integer               | 8 bytes |
| `f32` | 32-bit IEEE 754 float        | 4 bytes |
| `f64` | 64-bit IEEE 754 float        | 8 bytes |

Later proposals added reference types (`funcref`, `externref`) and the SIMD proposal added 128-bit vector types (`v128`).

### Text Format

While the primary distribution format is binary (`.wasm`), WebAssembly also defines a human-readable text format (`.wat`) using S-expressions:

```wasm
(module
  (func $add (param $a i32) (param $b i32) (result i32)
    local.get $a
    local.get $b
    i32.add
  )
  (export "add" (func $add))
)
```

This module defines a single function that adds two 32-bit integers and exports it under the name "add".

### Memory Model

WebAssembly linear memory is a contiguous, resizable byte array. It starts at a specified initial size and can grow up to a declared maximum. Memory is accessed through load and store instructions with static alignment hints:

```wasm
(memory 1 10)  ;; initial 1 page (64KB), max 10 pages (640KB)

(func $write_byte (param $addr i32) (param $val i32)
  local.get $addr
  local.get $val
  i32.store8
)
```

All memory accesses are bounds-checked at runtime. Out-of-bounds access traps immediately rather than producing undefined behavior.

### Security Model

WebAssembly executes within a sandboxed environment with several security guarantees:

- **No raw memory access** — all memory accesses are bounds-checked against linear memory
- **Control flow integrity** — indirect calls are type-checked at runtime through function tables
- **No ambient authority** — Wasm modules cannot access the file system, network, or other system resources without explicit host imports
- **Same-origin policy** — in browsers, Wasm modules are subject to the same security policies as JavaScript

## Language Support

Numerous programming languages can compile to WebAssembly:

### Tier 1 (Mature Support)

- **C/C++** — via Emscripten or LLVM directly
- **Rust** — via `wasm32-unknown-unknown` target, with `wasm-pack` and `wasm-bindgen`
- **AssemblyScript** — TypeScript-like language designed specifically for Wasm

### Tier 2 (Production-Ready)

- **Go** — via TinyGo compiler (standard Go compiler support is experimental)
- **C#/.NET** — via Blazor WebAssembly and the .NET runtime
- **Kotlin** — via Kotlin/Wasm compiler backend
- **Swift** — via SwiftWasm project

### Tier 3 (Experimental)

- **Python** — via Pyodide (CPython compiled to Wasm) or componentize-py
- **Ruby** — via ruby.wasm project
- **PHP** — via php-wasm
- **Haskell** — via GHC WebAssembly backend

## Use Cases

### Web Applications

WebAssembly is used extensively in performance-critical web applications:

- **Figma** — the collaborative design tool runs its rendering engine in Wasm, compiled from C++
- **Google Earth** — ported from a native desktop application to the web using Wasm
- **AutoCAD** — Autodesk ported their CAD application to the browser via Emscripten
- **Photoshop** — Adobe brought Photoshop to the web using Wasm and Emscripten

### Server-Side and Edge Computing

The WebAssembly System Interface (WASI) extends Wasm beyond the browser:

- **Cloudflare Workers** — supports Wasm modules for edge computing
- **Fastly Compute** — built entirely on a Wasm runtime (Wasmtime)
- **Fermyon Spin** — framework for building server-side Wasm applications
- **Docker + Wasm** — Docker integrated WasmEdge runtime for running Wasm containers

### Blockchain

Several blockchain platforms use Wasm as their smart contract execution environment:

- **Polkadot** — uses Wasm for its runtime and parachain validation
- **NEAR Protocol** — smart contracts compile to Wasm
- **Cosmos/CosmWasm** — Wasm-based smart contract platform

## Proposals and Future Development

WebAssembly development is governed by a phased proposal process (Phase 0 through Phase 5):

### Finalized Proposals

- **Multi-value returns** — functions can return multiple values
- **Reference types** — first-class references to host objects
- **Bulk memory operations** — efficient memory copying and filling
- **SIMD** — 128-bit packed SIMD instructions for data parallelism

### Active Proposals

- **Garbage Collection (WasmGC)** — native GC support enabling efficient compilation of managed languages like Java, Kotlin, and Dart
- **Component Model** — high-level composition of Wasm modules across language boundaries using WIT (Wasm Interface Type) definitions
- **Exception Handling** — zero-cost exception support using try/catch semantics
- **Threads** — shared linear memory and atomic operations for parallel execution
- **Tail Calls** — guaranteed tail call optimization for functional languages
- **Memory64** — 64-bit memory addresses for applications requiring more than 4GB of linear memory

### WASI

The WebAssembly System Interface (WASI) defines a standardized set of APIs for Wasm modules running outside the browser. WASI follows a capability-based security model where modules only have access to resources explicitly granted by the host.

WASI Preview 2, released in early 2024, introduced the Component Model as its foundation, replacing the POSIX-like API surface of Preview 1 with a more modular, composable interface based on WIT.

## Runtimes

Major WebAssembly runtimes outside the browser include:

| Runtime    | Developer  | Language | Focus                     |
|------------|------------|----------|---------------------------|
| Wasmtime   | Bytecode Alliance | Rust | Standards-compliant reference |
| Wasmer     | Wasmer Inc | Rust     | Universal Wasm runtime    |
| WasmEdge   | CNCF       | C++      | Cloud-native and edge     |
| wazero     | Tetrate    | Go       | Zero-dependency Go runtime|
| V8         | Google     | C++      | Browser and Node.js       |
| SpiderMonkey | Mozilla  | C++/Rust | Firefox browser engine    |

## Performance

Benchmarks consistently show WebAssembly executing within 10-30% of native code performance for compute-intensive workloads. Factors affecting performance include:

- **Startup time** — Wasm modules can be compiled ahead of time or streamed and compiled in parallel with download
- **Predictable performance** — unlike JavaScript, Wasm does not suffer from JIT deoptimization or garbage collection pauses (unless using WasmGC)
- **SIMD** — packed vector instructions provide significant speedups for graphics, audio, and scientific computing

## Criticism

Critics of WebAssembly have raised several concerns:

- **Obfuscation** — the binary format makes it harder to audit web page behavior compared to readable JavaScript
- **Security research challenges** — analyzing Wasm binaries for vulnerabilities is more difficult than analyzing source code
- **Complexity** — adding a fourth language to the web platform increases the surface area for browser bugs
- **Toolchain maturity** — debugging support and development tools are less mature than the JavaScript ecosystem

## See Also

- [asm.js](https://en.wikipedia.org/wiki/Asm.js)
- [Emscripten](https://emscripten.org/)
- [WASI](https://wasi.dev/)
- [Bytecode Alliance](https://bytecodealliance.org/)

## References

1. Haas, A., et al. "Bringing the Web up to Speed with WebAssembly." *ACM SIGPLAN Notices*, vol. 52, no. 6, 2017, pp. 185-200.
2. W3C. "WebAssembly Core Specification." W3C Recommendation, 5 December 2019. [https://www.w3.org/TR/wasm-core-1/](https://www.w3.org/TR/wasm-core-1/)
3. Clark, L. "A cartoon intro to WebAssembly." Mozilla Hacks, 2017. [https://hacks.mozilla.org/2017/02/a-cartoon-intro-to-webassembly/](https://hacks.mozilla.org/2017/02/a-cartoon-intro-to-webassembly/)
4. Bytecode Alliance. "WASI: The WebAssembly System Interface." [https://wasi.dev/](https://wasi.dev/)
5. Google. "WebAssembly Garbage Collection (WasmGC)." V8 Blog, 2023. [https://v8.dev/blog/wasm-gc](https://v8.dev/blog/wasm-gc)
