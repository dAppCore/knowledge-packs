---
type: Knowledge Pack
title: CoreGo Framework
description: Knowledge pack for CoreGo — the zero-dependency Go framework for the Lethean ecosystem
author: Mistral Vibe
version: 1.0.0
created: 2026-06-17T14:00:00Z
tags: [framework, golang, zero-dependency, core, lethean]
---

# CoreGo knowledge pack

> **Official Site:** [dappco.re/go/](https://dappco.re/go/)

This knowledge pack contains everything an agent needs to understand, use, and contribute to CoreGo — the foundational Go framework for the Lethean ecosystem.

**Official documentation:** All content in this knowledge pack is sourced from [dappco.re/go/](https://dappco.re/go/), the official CoreGo documentation site.

---

## Overview

`dappco.re/go` is the umbrella primitives package. Four universal types form the surface every operation flows through. The package has zero external dependencies — `go.mod` stays at three lines.

**Official description from [dappco.re/go/](https://dappco.re/go/):**

### Key characteristics

- **Zero external dependencies** — `go.mod` stays at three lines
- **Universal types** — Result, Options, Actions, Errors form the foundation
- **Design discipline:** Predictable names, comments as examples, path as documentation
- **Universal types everywhere** — No bespoke error idioms per-package
- **Lib never imports consumer** — Primitives stay zero-dependency

### Key statistics

- **Total packages:** 50+ (see [INDEX.md](INDEX.md))
- **Stdlib packages wrapped:** 44
- **SPOR Rule:** Single Point Of Responsibility — each stdlib package has exactly ONE owner file
- **Downstream impact:** ~30+ repositories depend on CoreGo
- **Current version:** v0.9.0 (last breaking-change window before v1.0.0-beta.1)
- **Status:** Patch releases (v0.9.x) for fixes only

### Official install

```bash
# From dappco.re/go/
go get dappco.re/go@latest
```

---

## Documentation structure

```
knowledge-packs/corego/
├── README.md              # This file — knowledge pack overview
├── INDEX.md               # Complete package catalog
├── SPEC.md                # CoreGo specification (if not in plans)
├── okf/
│   ├── index.md           # OKF bundle root
│   ├── log.md             # Change history
│   ├── framework/         # Framework-level concepts
│   │   ├── result.md      # core.Result pattern
│   │   ├── panic-recovery.md # Panic handling
│   │   └── ax-standard.md # AX Standard compliance
│   ├── packages/          # Individual package specs
│   │   ├── io.md          # I/O package
│   │   ├── json.md        # JSON utilities
│   │   ├── error.md       # Error handling
│   │   └── ... (50+ packages)
│   └── patterns/          # Common patterns
│       ├── spor.md       # Single Point Of Responsibility
│       ├── triplets.md    # Test triplet pattern
│       └── comments.md   # Comments-for-agents
└── examples/
    ├── basic-usage.md     # Getting started
    ├── error-handling.md  # Using core.Result
    └── testing.md        # Test triplet examples
```

---

## Source of truth

The **canonical specification** lives in:
- [`~/Code/meowmix/plans/code/core/go/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)
- [`~/Code/core/go/AGENTS.md`](file:///Users/snider/Code/core/go/AGENTS.md)

This knowledge pack aggregates and organises that information for agent consumption.

---

## Core concepts

All content sourced from [dappco.re/go/](https://dappco.re/go/)

### The four universal primitives

From the official documentation: "Four universal types form the surface every operation flows through."

| Primitive | Purpose | Source |
|-----------|---------|--------|
| **[Result](#1-the-result-pattern)** | `{Value any, OK bool}` — replaces the `(T, error)` pair | [dappco.re/go/result/](https://dappco.re/go/result/) |
| **[Options](#2-the-options-pattern)** | Typed key/value bag — the universal input | [dappco.re/go/options/](https://dappco.re/go/options/) |
| **[Actions](#3-the-actions-pattern)** | Named, registered, invokable unit of work | [dappco.re/go/action/](https://dappco.re/go/action/) |
| **[Errors](#4-the-errors-pattern)** | Structured `*Err` with operation context, stable codes | [dappco.re/go/error/](https://dappco.re/go/error/) |

Every component in the ecosystem accepts and returns the same primitive types. An agent processing any level of the tree sees identical shapes.

---

### 1. The Result pattern

**Purpose:** Standard error handling with logging and panic recovery

**Official description:** `Result` is the universal output type. Every Core operation returns one. It collapses the `(T, error)` pair into a single value with `OK` discrimination.

```go
// Type definition
type Result struct {
    Value any
    OK    bool
}
```

**Constructors:**

| Constructor | When to use | Example |
|-------------|--------------|---------|
| `core.Ok(v)` | Happy path | `return core.Ok(parsed)` |
| `core.Fail(err)` | Sad path | `if err := decode(b); err != nil { return core.Fail(err) }` |
| `core.ResultOf(v, err)` | Adapt a stdlib `(T, error)` pair | `r := core.ResultOf(os.ReadFile(path))` |
| `core.Try(fn)` | Wrap a function that may panic | `r := core.Try(func() any { return riskyParse(input) })` |

**Unwrap methods:**

| Method | When to use | Example |
|--------|--------------|---------|
| `r.OK` | Branch on success | `if !r.OK { return r }` |
| `r.Or(fallback)` | Get value or default | `port := opts.Get("port").Or("8080").(string)` |
| `r.Must()` | Panic on failure (init/test only) | `cfg := core.MustCast[*Config](...)` |
| `core.Cast[T](r)` | Typed extract `(T, ok)` | `if user, ok := core.Cast[*User](r); ok { use(user) }` |
| `core.MustCast[T](r)` | Panicking generic variant | Hot config paths |

**Inspect methods:**

| Method | Returns | Example |
|--------|---------|---------|
| `r.Error()` | Error message string when `!r.OK` | `if r.Error() != "" { log.Error(r.Error()) }` |
| `r.Code()` | Stable error code when failure is `*core.Err` | `switch r.Code() { case "fs.notfound": firstRun() }` |

**Why a single shape:** Every operation in `core/go` returns `Result`. Every operation in every consumer package returns `Result`. Agents reading the codebase never have to learn a per-package error idiom — the shape is universal.

**AX Standard:** Every function returns `core.Result[T]` instead of `(T, error)`

---

### 2. The Options pattern

**Purpose:** Typed key/value bag — the universal input

**Official description:** `Options` is the universal input type. Every Core operation that takes parameters takes one. A structured collection of key/value pairs with typed accessors.

```go
// Type definition
type Option struct {
    Key   string
    Value any
}

type Options struct { /* opaque */ }
```

**Construct:**

```go
opts := core.NewOptions(
    core.Option{Key: "name", Value: "brain"},
    core.Option{Key: "port", Value: 8080},
    core.Option{Key: "debug", Value: true},
)
```

**Mutate:**

```go
opts.Set("key", value)  // Add or update a key. Mutates in place.
```

**Typed accessors:** Each accessor returns the type's zero value when the key is missing or the stored value isn't assignable to that type.

| Accessor | Returns | Zero | Example |
|----------|---------|------|---------|
| `opts.String(key)` | `string` | `""` | `name := opts.String("name")` |
| `opts.Int(key)` | `int` | `0` | `port := opts.Int("port")` |
| `opts.Bool(key)` | `bool` | `false` | `debug := opts.Bool("debug")` |
| `opts.Float64(key)` | `float64` | `0` | `weight := opts.Float64("weight")` (promotes int/int64/float32) |
| `opts.Duration(key)` | `core.Duration` | `0` | `timeout := opts.Duration("timeout")` (parses string via `ParseDuration`) |

**Strict reads (Result-shaped):** For cases where missing-vs-typed-zero matters:

```go
r := opts.Get("port")
if !r.OK { return core.Fail(core.NewError("port required")) }
port := r.Value.(int)
```

**Inspect:**

| Method | Returns | Example |
|--------|---------|---------|
| `opts.Has(key)` | `true` if the key exists, regardless of value type | `if opts.Has("debug") { ... }` |
| `opts.Len()` | Number of options | `if opts.Len() > 0 { ... }` |
| `opts.Items()` | Copy of the underlying option slice | `for _, opt := range opts.Items() { ... }` |

---

### 3. The Actions pattern

**Purpose:** Named, registered, invokable unit of work

**Official description:** Actions are the atomic unit of work in `core/go`. Named, registered, invokable, inspectable. The Action registry **is** the capability map.

**Register:**

```go
c.Action("git.log", func(ctx core.Context, opts core.Options) core.Result {
    dir := opts.String("dir")
    return c.Process().RunIn(ctx, dir, "git", "log")
})
```

The signature is invariant across every action:

```go
type ActionHandler func(core.Context, core.Options) core.Result
```

**Invoke:**

```go
r := c.Action("git.log").Run(ctx, core.NewOptions(
    core.Option{Key: "dir", Value: "/path/to/repo"},
))
```

**Lifecycle — enable / disable:**

```go
c.Action("dangerous.purge").Disable()
// ...later...
c.Action("dangerous.purge").Enable()

if c.Action("dangerous.purge").Enabled() { /* will fire */ }
```

`Run()` on a disabled action returns `Result{OK: false}` with code `"action.disabled"`. The capability stays queryable via `Exists()`.

**Inspect:**

| Method | Returns | Example |
|--------|---------|---------|
| `c.Action(name).Exists()` | `true` if a handler is registered | `if c.Action("git.log").Exists() { ... }` |
| `c.Action(name).Enabled()` | `true` if it will run when invoked | `if c.Action("git.log").Enabled() { ... }` |
| `c.Actions()` | All registered action names in registration order | `for _, name := range c.Actions() { ... }` |

**Background — PerformAsync:** For long-running work that shouldn't block the request path:

```go
r := c.PerformAsync("agentic.dispatch", opts)
taskID := r.Value.(string)
// ActionTaskStarted / ActionTaskProgress / ActionTaskCompleted broadcast on IPC
```

**Progress updates:**

```go
c.Progress(taskID, 0.5, "halfway done", "agentic.dispatch")
```

**Composition — Tasks:** A `Task` is a named sequence of action steps:

```go
c.Task("agent.completion", core.Task{
    Steps: []core.Step{
        {Action: "agentic.qa"},
        {Action: "agentic.auto-pr"},
        {Action: "agentic.verify", Input: "previous"},
        {Action: "agentic.poke", Async: true},
    },
})

r := c.Task("agent.completion").Run(ctx, c, opts)
```

Sync steps run sequentially — failure stops the chain. Async steps fire-and-forget. `Input: "previous"` pipes the last sync step's output as `_input` on the next.

**Why named actions:** The registry IS the capability map. Auditable, testable, swappable. Permission boundaries (entitlements) and observability (broadcast events) attach at a single choke point — `Action.Run` — instead of being threaded through every call site.

---

### 4. The Errors pattern

**Purpose:** Structured `*Err` with operation context, stable codes, uniform introspection

**Constructors:**

| Constructor | When to use |
|-------------|--------------|
| `core.E(err)` | Create error Result |
| `core.NewError(msg)` | Create new error with message |
| `core.NewCode(code, msg)` | Create error with stable code |
| `core.Wrap(err, msg)` | Wrap existing error with context |

**Codes form a flat keyspace agents grep on:**

```go
switch r.Code() {
case "fs.notfound":
    firstRun()
case "http.timeout":
    retry()
case "http.refused":
    fallback()
}
```

See the stable codespace for all available error codes.

### 3. SPOR (Single Point Of Responsibility)

Each stdlib package is wrapped by **exactly one file** in CoreGo:

| Stdlib Package | Owner File | CoreGO Wrapper |
|----------------|------------|----------------|
| `fmt` | `format.go` | `core.Sprintf` |
| `io` | `io.go` | `core.Reader` |
| `json` | `json.go` | `core.JSONMarshal` |
| `http` | `api.go` | `core.Request` |

**Why it matters:** Prevents duplicate wrappers and ensures consistency across the ecosystem.

### 4. Test triplets

Every package has three files:

```
pkg/
├── file.go              # Implementation
├── file_test.go         # Unit tests (Good/Bad/Ugly pattern)
└── file_example_test.go # Example tests
```

**Pattern:** Good tests (happy path), Bad tests (error cases), Ugly tests (edge cases)

---

## Design principles

From [dappco.re/go/](https://dappco.re/go/design):

Every shape decision in `core/go` propagates to ~30 downstream consumer repos. The discipline:

- **Predictable names over short names** — agents grep, not autocomplete
- **Comments as usage examples** — every public symbol shows a copy-pastable call
- **Path is documentation** — folder structure equals API shape
- **Universal types** — Result, Options, Action are everywhere; no bespoke pairs
- **Lib never imports consumer** — primitives stay zero-dependency

These principles form the **AX Standard** (Agent Experience) — designing code for AI agents as first-class consumers.

---

## Package catalog

See [INDEX.md](INDEX.md) for the complete list of 50+ packages.

### Core packages

| Package | Purpose | Files |
|---------|---------|-------|
| `action` | Action execution framework | action.go, action_test.go |
| `api` | HTTP client/server | api.go, api_test.go |
| `context` | Context utilities | context.go |
| `error` | Error handling | error.go |
| `format` | String formatting | format.go |
| `io` | I/O operations | io.go |
| `json` | JSON encoding/decoding | json.go |
| `log` | Structured logging | log.go |
| `result` | Result type and panic recovery | result.go |

### I/O packages

| Package | Purpose | Backend |
|---------|---------|---------|
| `go-io` | Storage-agnostic I/O | 8 backends |
| `go-cache` | Caching layer | go-io medium |
| `go-store` | SQLite key-value store | |

### I/O packages

| Package | Purpose | Backends |
|---------|---------|----------|
| `go-io` | **Mandatory I/O abstraction** | 10+ (Local, Memory, S3, GitHub, etc.) |
| `go-cache` | Caching layer | go-io medium |
| `go-store` | SQLite key-value store | |

### Network packages

| Package | Purpose | Protocol | Deep Dive |
|---------|---------|----------|-----------|
| `go-dns` | .lthn DNS Resolution | DNS (RFC 1035) | **[Available](./pkg/dns/README.md)** |
| `go-p2p` | Peer-to-peer networking | Levin, UEPS, WebSocket | **[Available](./pkg/p2p/README.md)** |
| `go-proxy` | Stratum mining proxy | TCP/TLS, Stratum JSON-RPC | **[Available](./pkg/proxy/README.md)** |
| `go-netops` | UniFi network controller | HTTP API | Coming soon |

### Internationalisation packages

| Package | Purpose | Features | Deep Dive |
|---------|---------|----------|-----------|
| `go-i18n` | Grammar-aware i18n | Semantic intent, GrammarImprint, dual-class, CLDR | **[Available](./pkg/i18n/README.md)** |

### Blockchain packages

| Package | Purpose | Status | Deep Dive |
|---------|---------|--------|-----------|
| `go-blockchain` | Blockchain implementation | Production | **[Available](./pkg/blockchain/README.md)** |
| `go-lns` | Lethean Name System | Production | **[Available](./pkg/lns/README.md)** |
| `go-dns` | .lthn DNS Resolution | Production | **[Available](./pkg/dns/README.md)** |
| `go-io` | **Mandatory I/O abstraction** | Production | **[Available](./pkg/io/README.md)** |
| `go-miner` | Mining operations | Production | Coming soon |

### AI/ML packages

| Package | Purpose | Backend |
|---------|---------|---------|
| `go-ai` | AI operations | |
| `go-ml` | ML inference | CPU |
| `go-mlx` | Apple Metal GPU | Production |
| `go-rocm` | AMD GPU | Production |
| `go-cuda` | NVIDIA GPU | Scaffold |
| `go-tpu` | Google TPU | Scaffold |
| `go-inference` | Shared ML interfaces | |

---

## Getting started

### For agents

1. **Read the RFC:** Start with [`plans/code/core/go/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)
2. **Explore AGENTS.md:** [`core/go/AGENTS.md`](file:///Users/snider/Code/core/go/AGENTS.md)
3. **Browse packages:** See [INDEX.md](INDEX.md)
4. **Use Result:** Always return `core.Result[T]`
5. **Follow SPOR:** Each stdlib package has one owner

### For developers

1. **Clone the repo:** `git clone forge.lthn.sh/core/go`
2. **Read the RFC:** Understand the contract
3. **Use core.Result:** For all error handling
4. **Write triplets:** Implementation + tests + examples
5. **Follow AX Standard:** Comments for agents, not humans

---

## Exploring the codebase

### Key files

- [`core/go/AGENTS.md`](file:///Users/snider/Code/core/go/AGENTS.md) — Agent-specific guidance
- [`core/go/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md) — Primary specification
- [`core/go/result.go`](file:///Users/snider/Code/core/go/result.go) — Result type implementation
- [`core/go/api.go`](file:///Users/snider/Code/core/go/api.go) — HTTP utilities

### Discovery commands

```bash
# List all packages
ls -1 /Users/snider/Code/core/go/*.go | sed 's/.go$//' | sort

# Count files
ls /Users/snider/Code/core/go/*.go | wc -l

# Find test triplets
for file in /Users/snider/Code/core/go/*.go; do
    base=$(basename $file .go)
    if [ -f "${file}_test.go" ] && [ -f "${file}_example_test.go" ]; then
        echo "✓ $base has complete triplet"
    fi
done
```

---

## Quick reference

### Result constructors

| Constructor | Purpose | Example |
|-------------|---------|---------|
| `core.Ok(v)` | Success with value | `return core.Ok(data)` |
| `core.Fail(err)` | Failure with error | `return core.Fail(err)` |
| `core.ResultOf(v, err)` | Adapt `(T, error)` | `r := core.ResultOf(os.ReadFile(path))` |
| `core.Try(fn)` | Panic-safe wrapper | `r := core.Try(func() any { ... })` |

### Result methods

| Method | Returns | Use case |
|--------|---------|----------|
| `r.OK` | `bool` | Check success |
| `r.Value` | `any` | Access the value |
| `r.Error()` | `string` | Get error message |
| `r.Code()` | `string` | Get stable error code |
| `r.Or(fallback)` | `any` | Value or fallback |
| `r.Must()` | `any` | Panic on error |
| `core.Cast[T](r)` | `(T, bool)` | Type-safe extract |
| `core.MustCast[T](r)` | `T` | Panic on type mismatch |

### Options accessors

| Accessor | Returns | Zero value |
|----------|---------|------------|
| `opts.String(key)` | `string` | `""` |
| `opts.Int(key)` | `int` | `0` |
| `opts.Bool(key)` | `bool` | `false` |
| `opts.Float64(key)` | `float64` | `0` |
| `opts.Duration(key)` | `core.Duration` | `0` |

### Action methods

| Method | Returns | Use case |
|--------|---------|----------|
| `c.Action(name).Run(ctx, opts)` | `Result` | Invoke action |
| `c.Action(name).Exists()` | `bool` | Check if registered |
| `c.Action(name).Enabled()` | `bool` | Check if enabled |
| `c.Action(name).Disable()` | - | Disable action |
| `c.Action(name).Enable()` | - | Enable action |
| `c.Actions()` | `[]string` | List all actions |
| `c.PerformAsync(name, opts)` | `Result` | Run in background |
| `c.Progress(taskID, pct, msg, action)` | - | Update progress |

---

## Use cases

### When to use CoreGo

- **New dAppCore repository** — Always start with CoreGo
- **Error handling** — Use `core.Result[T]` instead of `(T, error)`
- **Panic recovery** — Use `core.Recover` in handlers
- **I/O operations** — Use `go-io` backends instead of raw file operations
- **HTTP clients** — Use `core.Request` instead of raw `http.Client`
- **Universal input** — Use `core.Options` for all function parameters
- **Named capabilities** — Use `core.Action` for all invokable operations

### When not to use CoreGo

- **Performance-critical code** — CoreGo adds minimal overhead but may not be suitable for extreme performance requirements
- **External libraries** — Don't wrap external library calls in CoreGo (use the library directly)
- **Stdlib packages already wrapped** — Don't re-wrap; use the existing CoreGo wrapper
- **Binary size-sensitive projects** — CoreGo is designed for clarity over minimalism

---

## Related knowledge packs

- [CoreGUI](../coregui/README.md) — GUI framework (Wails v2)
- [CoreTS](../corets/README.md) — TypeScript framework
- [CoreCLI](../corecli/README.md) — CLI framework
- [CorePlay](../coreplay/README.md) — Play framework (Go backend + TS frontend)
- [CorePHP](../corephp/README.md) — PHP framework

---

## Agent tips

### Core principles

1. **Always check the RFC first** — The spec in [`plans/code/core/go/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md) drives the code
2. **Follow SPOR** — Don't duplicate stdlib wrappers; each has exactly one owner
3. **Use core.Result** — Never return raw `(T, error)`; always use `core.Result[T]`
4. **Write triplets** — Every new package needs `file.go`, `file_test.go`, and `file_example_test.go`
5. **Comments for agents** — Write documentation that agents can parse (show usage, not describe)

### AX-specific guidance

6. **Universal shapes** — Every operation returns `Result`, takes `Options`
7. **Named everything** — Use descriptive names; abbreviations add mapping overhead
8. **Path navigation** — Directory structure tells you the intent before reading files
9. **Greppable codes** — Error codes form a flat keyspace for easy searching
10. **Capability map** — The Action registry IS the API; it's auditable and testable

### Learning resources

- **[Official site](https://dappco.re/go/)** — Always start here
- **[Result deep dive](https://dappco.re/go/result/)** — Master the universal output type
- **[Options deep dive](https://dappco.re/go/options/)** — Understand the universal input
- **[Actions deep dive](https://dappco.re/go/action/)** — Learn named capabilities
- **[GitHub repository](https://github.com/dappcore/go)** — Source code and issues
- **[AGENTS.md](file:///Users/snider/Code/core/go/AGENTS.md)** — Agent-specific guidance

---

## Maintenance

This knowledge pack is maintained by Mistral Vibe. Updates are triggered by:

- Changes to `plans/code/core/go/`
- Changes to `core/go/` repository
- New downstream repository requirements
- AX Standard updates

---

*Knowledge Pack v1.0.0*
*Created: 2026-06-17T14:00:00Z*
*Author: Mistral Vibe*
*Source: Lethean CoreGo Framework*
