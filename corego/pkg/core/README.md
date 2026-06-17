---
type: Deep Dive
title: CoreGo Framework
package: core
description: The zero-dependency foundation framework for dAppCore — dependency injection, service lifecycle, Result pattern, IPC, and universal primitives
domain: dappco.re/go
repository: https://github.com/dappcore/go
---

# CoreGo Framework — The Foundation

> **Zero dependencies. Zero exceptions.**
> The universal substrate that ~30 downstream dAppCore packages build upon.

---

## 🎯 Overview

**CoreGo** (`core/go` or `dappco.re/go`) is a zero-dependency Go framework that provides:

- **Dependency Injection** — Central `Core` struct with typed accessors for all subsystems
- **Service Lifecycle** — Startup, shutdown, reload hooks with ordering guarantees
- **Result Pattern** — Universal error handling with panic recovery
- **IPC Message Bus** — ACTION (broadcast), QUERY (first-respondent), QUERYALL (collect all)
- **Named Action Registry** — Capability discovery and invocation
- **Universal Types** — `Options` for input, `Result` for output
- **SPOR Compliance** — Single Point Of Responsibility for all stdlib wrappers

**Design Philosophy:** Every pattern in CoreGo is intentionally shaped for **Agent Experience (AX)** — AI agents replicate these patterns without a judgement layer. The framework IS the example.

---

## 🏗️ Core Architecture

### The Core Struct

The central object that orchestrates everything:

```go
c := core.New(core.WithOption("name", "my-app"))
```

```
┌─────────────────────────────────────────────────────────────────┐
│                        Core Struct                                  │
├─────────────────────────────────────────────────────────────────┤
│  Accessors:                                                            │
│  ┌ c.Options()  → *Options     (input configuration)               │
│  ├ c.App()      → *App         (identity: name, version, runtime)    │
│  ├ c.Data()     → *Data        (embedded assets registry)           │
│  ├ c.Drive()    → *Drive       (transport handle registry)          │
│  ├ c.Fs()       → *Fs          (sandboxed filesystem)              │
│  ├ c.Config()   → *Config      (runtime settings + feature flags)   │
│  ├ c.Error()    → *ErrorPanic  (panic recovery + crash reporting)   │
│  ├ c.Log()      → *ErrorLog    (structured logging)                 │
│  ├ c.Context()  → Context      (lifecycle context)                  │
│  ├ c.IPC()      → *Ipc         (message bus)                        │
│  ├ c.Info()     → *SysInfo     (read-only system/env info)         │
│  ├ c.I18n()     → *I18n       (internationalisation)                │
│  ├ c.Cli()      → *Cli         (command framework)                  │
│  ├ c.Command()  → *Command     (command tree access)               │
│  └ c.Service()  → *Service     (service registry)                   │
│                                                                     │
│  Lifecycle:                                                         │
│  ┌ c.Run()        → Start services → Run CLI → Shutdown           │
│  ├ c.RunResult() → Same but returns Result for programmatic use   │
│  ├ c.ServiceStartup()                                            │
│  └ c.ServiceShutdown()                                          │
│                                                                     │
│  IPC (Uppercase aliases):                                          │
│  ┌ c.ACTION(msg)  → Broadcast to all handlers (fire-and-forget)   │
│  ├ c.QUERY(q)     → First handler to return OK wins               │
│  └ c.QUERYALL(q)  → Collect all OK responses                         │
└─────────────────────────────────────────────────────────────────┘
```

### Universal Types

#### Options — Universal Input

Structured key-value configuration. Every Core operation accepts Options.

```go
// Creating options
opts := core.NewOptions(
    core.Option{Key: "name", Value: "brain"},
    core.Option{Key: "port", Value: 8080},
)

// Typed accessors
name := opts.String("name")           // "brain"
port := opts.Int("port")              // 8080
 enabled := opts.Bool("enabled")      // false (default)

// Nested options
nested := core.NewOptions(
    core.Option{Key: "database", Value: core.NewOptions(
        core.Option{Key: "host", Value: "localhost"},
        core.Option{Key: "port", Value: 5432},
    )},
)
dbHost := nested.Options("database").String("host")
```

#### Result — Universal Output

Every Core operation returns a Result. Replaces Go's `(value, error)` pattern.

```go
// Structure
type Result struct {
    Value any
    OK    bool
}

// Constructors
r := core.Ok("success")              // Result{Value: "success", OK: true}
r := core.Fail(errors.New("failed")) // Result{Value: err, OK: false}
r := core.ResultOf(file, err)        // Adapts (T, error) → Result

// Consumption
if !r.OK {
    log.Error("operation failed", "error", r.Error())
    return r
}
value := r.Value.(string)  // Type assertion

// Common patterns
// Must - panic on failure (for init/test only)
config := core.JSONUnmarshal(data).Must().(*Config)

// Or - fallback on failure
value := core.Env("PORT").Or("8080").Value.(string)

// Code - stable error codes for grepping
if r.Code() == "fs.notfound" {
    // Handle specific error
}
```

---

## ⚡ Key Features

### 1. Service Lifecycle Management

Services are the building blocks of CoreGo applications:

```go
// Define a service
myService := core.Service{
    Name: "database",
    OnStart: func() core.Result {
        // Initialize database connection
        return core.Ok(nil)
    },
    OnStop: func() core.Result {
        // Close database connection
        return core.Ok(nil)
    },
    OnReload: func() core.Result {
        // Reconnect to database
        return core.Ok(nil)
    },
}

// Register service
c.Service("database", myService)

// Or register with auto-discovery of Startable/Stoppable interfaces
c.RegisterService("database", dbInstance)

// Service options
core.WithService(myService.Register)        // Register during Core creation
core.WithServiceLock()                      // Lock service registry (no new services)
```

**Lifecycle Order:**
1. `OnStart` called in registration order
2. CLI runs (if configured)
3. `OnStop` called in reverse order on shutdown
4. `OnReload` for configuration changes

**Service Lock:** When `WithServiceLock()` is used, no new services can be registered and admin modifications are prevented. This creates a secure enclave.

### 2. IPC Message Bus

Three messaging patterns for inter-service communication:

```go
// ACTION - Fire-and-forget broadcast
c.ACTION(messages.AgentCompleted{Agent: "codex", Status: "completed"})

// QUERY - First respondent wins
r := c.QUERY(MyQuery{Name: "brain"})
if r.OK {
    result := r.Value.(MyResponse)
}

// QUERYALL - Collect all responses
r := c.QUERYALL(countQuery{})
results := r.Value.([]any)

// Register handlers
c.RegisterAction(func(_ *core.Core, msg core.Message) core.Result {
    if agentMsg, ok := msg.(messages.AgentCompleted); ok {
        log.Info("Agent completed", "agent", agentMsg.Agent)
    }
    return core.Result{OK: true}
})

c.RegisterQuery(func(_ *core.Core, q core.Query) core.Result {
    if countQ, ok := q.(countQuery); ok {
        return core.Ok(countQ.Count)
    }
    return core.Result{OK: false}
})
```

**Built-in Messages:**
- `ActionServiceStartup` — Broadcast when a service finishes startup
- `ActionServiceShutdown` — Broadcast when Core begins shutdown
- `ActionTaskStarted` — Broadcast when async task begins
- `ActionTaskProgress` — Broadcast for task progress updates
- `ActionTaskCompleted` — Broadcast when task finishes

### 3. Named Action Registry

The action system is the **capability map** — it answers "what can this application do?"

```go
// Register an action
c.Action("git.log", func(ctx context.Context, opts core.Options) core.Result {
    dir := opts.String("dir")
    return c.Process().RunIn(ctx, dir, "git", "log", "-1")
})

// Invoke by name
r := c.Action("git.log").Run(ctx, core.NewOptions(
    core.Option{Key: "dir", Value: "/path/to/repo"},
))

// Check capability
if c.Action("process.run").Exists() {
    // Action is available
}

// List all actions
names := c.Actions()  // ["process.run", "agent.dispatch", ...]

// Disable/enable actions
c.Action("dangerous.purge").Disable()
c.Action("dangerous.purge").Enable()
```

**Action Features:**
- Named and discoverable
- Typed input via `Options`
- Panic recovery built-in
- Entitlement checking (permission boundaries)
- Enable/disable toggles
- Schema declaration for expected inputs

### 4. Context Propagation

Core manages context lifecycles carefully:

```go
// Derived context for request-scoped operations
rc := c.WithContext(core.WithValue(c.Context(), userKey{}, user))

// Context cancellation propagates correctly:
// - Parent shutdown → derived context
// - Derived cancel does NOT affect parent

// Task-scoped context
taskCtx, taskCancel := core.WithTimeout(c.Context(), 30*time.Second)
defer taskCancel()

// Use in services
go func() {
    select {
    case <-taskCtx.Done():
        // Cleanup on timeout or parent cancellation
    case <-done:
        // Normal completion
    }
}()
```

### 5. Configuration Management

Hierarchical configuration with multiple sources:

```go
// Access configuration
c.Config().String("database.host")
c.Config().Int("database.port")
c.Config().Bool("debug.enabled")

// Feature flags
c.Config().Enable("experimental.rag")
c.Config().Disable("legacy.api")
c.Config().IsEnabled("experimental.rag")

// Config sources (checked in order):
// 1. CLI flags
// 2. Environment variables
// 3. Config files
// 4. Defaults

// Config watching (for reloads)
c.Config().Watch(func(key string, value any) {
    log.Info("Config changed", "key", key, "value", value)
})
```

### 6. Embedded Data Registry

Asset management for embedded files:

```go
// Embed data at init
data := core.AddAsset("prompts/coding.md", codingPromptContent)

// Read embedded data
content := c.Data().ReadString("prompts/coding.md")

// List all embedded assets
names := c.Data().Names()

// Check if asset exists
if c.Data().Has("prompts/coding.md") {
    // Asset is available
}
```

### 7. Transport Registry (Drive)

Resource handle management:

```go
// Get a transport handle
r := c.Drive().Get("forge")
if r.OK {
    handle := r.Value.(*core.DriveHandle)
    // Use handle for network operations
}

// Register a transport
c.Drive().Set("custom", myTransport)
```

### 8. Structured Logging

Mandatory error creation with structured fields:

```go
// Log with fields
c.Log().Info("request started", "method", "GET", "path", "/api")
c.Log().Error(err, "request failed", "method", "GET", "status", 500)
c.Log().Warn(err, "retrying", "attempt", 3)
c.Log().Debug("cache hit", "key", cacheKey)

// LogError/Warn helpers return Result
r := c.LogError(err, "service.Start", "database connection failed")
if !r.OK {
    return r
}
```

**Log Levels:** Info, Error, Warn, Debug (configurable)

**Error Logging:** Mandatory — all errors MUST be created through Core's error system for proper tracking.

---

## 📋 Framework Primitives Deep Dive

### Error System

**Mandatory Error Creation:** All errors flow through Core's error system.

```go
// Create errors
err := core.E("fs.read", "file not found", nil)
err := core.NewError("validation failed")
err := core.NewCode("http.timeout", "request timed out")

// Wrap errors
err := core.Wrap(err, "fs.read", "cannot read config")
err := core.WrapCode(err, "config.load", "failed to load")

// Error codes for grepping
// All error codes form a flat keyspace:
// - "fs.notfound", "fs.permission"
// - "json.invalid", "json.marshal"
// - "http.timeout", "http.refused"
// - "crypto.algo.unsupported"
// - "core.Service", "action.Run"
```

**Error Type:**
```go
type Err struct {
    Code    string      // Stable error code (for grepping)
    Message string      // Human-readable message
    Cause   error       // Underlying error (wrapped)
    Fields  []string    // Additional context fields
}
```

### Registry Pattern

Thread-safe named collections with three lock modes:

```go
// Create registry
r := core.NewRegistry[*MyService]()

// Set item
r.Set("brain", brainService)

// Get item
result := r.Get("brain")
if result.OK {
    svc := result.Value.(*MyService)
}

// Check existence
if r.Has("brain") { ... }

// List all names (insertion order)
names := r.Names()

// Iterate
r.Each(func(name string, svc *MyService) {
    fmt.Println(name, svc)
})

// Lock modes
r.Seal()   // No new keys, existing can be updated
r.Lock()   // Fully frozen, no writes at all
r.Unlock() // Return to open mode
```

**Lock Modes:**
- **Open** (default) — Anything goes
- **Sealed** — No new keys, existing keys CAN be updated
- **Locked** — Fully frozen, no writes at all

### Command Framework

Hierarchical CLI command system:

```go
// Register commands
c.Command("deploy", "Deploy application")
c.Command("deploy.to", "Deploy to target")
c.Command("deploy.to.homelab", "Deploy to homelab")

// Command structure
cmd := c.Command("deploy.to.homelab")
cmd.Description  // "Deploy to homelab"
cmd.Usage       // "deploy to homelab [options]"
cmd.Aliases     // ["dh", "deploy-homelab"]

// Execute command
r := c.Cli().Run("deploy", "to", "homelab")

// Command tree navigation
parent := c.Command("deploy.to")
children := parent.Children()
```

### Entitlement System

Permission boundaries for actions and services:

```go
// Check entitlement
if c.Entitled("agent.dispatch").Allowed {
    // Action is permitted
}

// Custom entitlement checker
c.SetEntitlementChecker(func(action string) core.EntitlementResult {
    if action == "admin.purge" {
        return core.EntitlementResult{Allowed: false, Reason: "reserved for admin"}
    }
    return core.EntitlementResult{Allowed: true}
})

// Action-level entitlement
if e := c.Action("admin.purge").Entitled(); !e.Allowed {
    return core.Fail(core.NewCode("not.entitled", e.Reason))
}
```

---

## 🛡️ Core Patterns

### Pattern 1: Service with Lifecycle

```go
type Database struct {
    conn *sql.DB
}

func (db *Database) OnStartup(ctx context.Context) core.Result {
    var err error
    db.conn, err = sql.Open("sqlite3", ":memory:")
    if err != nil {
        return core.Fail(err)
    }
    return core.Ok(nil)
}

func (db *Database) OnShutdown(ctx context.Context) core.Result {
    if err := db.conn.Close(); err != nil {
        return core.Fail(err)
    }
    return core.Ok(nil)
}

// Register
c.RegisterService("database", &Database{})

// Usage in other services
func (s *MyService) OnStartup(ctx context.Context) core.Result {
    db, ok := core.ServiceFor[*Database](c, "database")
    if !ok {
        return core.Fail(core.NewError("database service required"))
    }
    s.db = db
    return core.Ok(nil)
}
```

### Pattern 2: IPC Communication

```go
// Service A: Broadcast events
func (s *ServiceA) Process() {
    s.core.ACTION(messages.TaskStarted{
        TaskID: "123",
        Type:   "inference",
    })
    
    // Do work...
    
    s.core.ACTION(messages.TaskCompleted{
        TaskID: "123",
        Status: "success",
    })
}

// Service B: Handle events
func (s *ServiceB) OnStartup(ctx context.Context) core.Result {
    s.core.RegisterAction(func(_ *core.Core, msg core.Message) core.Result {
        switch m := msg.(type) {
        case messages.TaskStarted:
            s.trackTaskStart(m.TaskID, m.Type)
        case messages.TaskCompleted:
            s.trackTaskComplete(m.TaskID, m.Status)
        }
        return core.Result{OK: true}
    })
    return core.Ok(nil)
}
```

### Pattern 3: Query-Based Data Fetching

```go
// Service providing data
func (s *DataService) OnStartup(ctx context.Context) core.Result {
    s.core.RegisterQuery(func(_ *core.Core, q core.Query) core.Result {
        if req, ok := q.(GetUserRequest); ok {
            user, err := s.getUser(req.ID)
            if err != nil {
                return core.Fail(err)
            }
            return core.Ok(user)
        }
        return core.Result{OK: false}
    })
    return core.Ok(nil)
}

// Service consuming data
func (s *APIService) GetUserHandler(w http.ResponseWriter, r *http.Request) {
    userID := r.URL.Query().Get("id")
    r := s.core.QUERY(GetUserRequest{ID: userID})
    if !r.OK {
        http.Error(w, r.Error(), http.StatusNotFound)
        return
    }
    user := r.Value.(*User)
    json.NewEncoder(w).Encode(user)
}
```

### Pattern 4: Result Chain Handling

```go
// Chaining operations with Result
func (s *Service) ProcessFile(path string) core.Result {
    // Read file
    dataR := s.core.Fs().Read(path)
    if !dataR.OK {
        return dataR
    }
    
    // Parse JSON
    var config Config
    parseR := core.JSONUnmarshal(dataR.Value.([]byte), &config)
    if !parseR.OK {
        return parseR
    }
    
    // Validate
    if config.Version != "2" {
        return core.Fail(core.NewCode("config.unsupported", "version 2 required"))
    }
    
    // Success
    return core.Ok(config)
}

// With LogError for automatic logging
func (s *Service) ProcessFileLogged(path string) core.Result {
    dataR := s.core.Fs().Read(path)
    if !dataR.OK {
        return s.core.LogError(dataR.Error(), "fs.Read", "failed to read config")
    }
    return dataR
}
```

### Pattern 5: Options-Based Configuration

```go
type MyService struct {
    host string
    port int
}

func (s *MyService) Configure(opts core.Options) core.Result {
    s.host = opts.String("host")
    s.port = opts.Int("port")
    
    if s.host == "" {
        return core.Fail(core.NewCode("config.missing", "host is required"))
    }
    if s.port == 0 {
        s.port = 8080  // Default
    }
    
    return core.Ok(nil)
}

// Usage
c := core.New(
    core.WithService(myService.Register),
    core.WithOption("host", "localhost"),
    core.WithOption("port", 3000),
)
```

---

## 🔌 CoreGo Service Integration

### WithServiceLock() — Secure Enclave

Creates an immutable service bundle:

```go
// Services registered before lock are fixed
c := core.New(
    core.WithService(dbService.Register),
    core.WithService(cacheService.Register),
    core.WithServiceLock(),  // ⬅ Locks here
    // core.WithService(newService) would fail now
)

// Service registry is now frozen
// No new services can be registered
// No admin modifications allowed
```

### Service Discovery

```go
// Get service by type
svc, ok := core.ServiceFor[*Database](c, "database")
if !ok {
    // Handle missing service
}

// Get service by name (returns interface{})
r := c.Service("database")
if r.OK {
    db := r.Value.(*Database)
}

// Check if service exists
if c.Service("database").OK {
    // Service is registered
}
```

---

## 📊 Performance Considerations

### Lock-Free Dispatch

IPC dispatch uses atomic pointers for lock-free reads:

```go
// actionHandlers and queryHandlers are stored behind AtomicPointer
handlers := c.ipc.ipcActions.Load()  // Lock-free read
if handlers != nil {
    for _, h := range *handlers {
        h(c, msg)  // Execute handler
    }
}
```

Registration takes a short mutex while copying-on-write to a new slice.

### Registry Efficiency

`Registry[T]` uses:
- `RWMutex` for thread-safety
- `map[string]T` for O(1) lookups
- `[]string` for insertion-order iteration
- Three modes (Open, Sealed, Locked) for progressive freezing

### Result Ergonomics

Result methods collapse common patterns:

```go
// Instead of:
if !r.OK {
    if err, ok := r.Value.(error); ok {
        return err.Error()
    }
    return "unknown error"
}

// Use:
return r.Error()  // Handles both error and string types

// Instead of:
if !r.OK {
    return r
}
value := r.Value.(string)

// Use:
value := r.Must().(string)  // Panics with context on failure
```

---

## 🎯 When to Use CoreGo

### ✅ Perfect for:

- **Agent Orchestration** — The entire dAppCore ecosystem is built on CoreGo
- **Microservices** — Service lifecycle + IPC makes service composition natural
- **CLI Tools** — Command framework + configuration management
- **Daemons** — Long-running processes with clean shutdown
- **Plugin Systems** — Action registry enables plugin discovery
- **AI Services** — RAG, inference backends all use CoreGo

### 📦 What You Get:

- **Zero external dependencies** — Pure Go stdlib
- **Panic recovery** — Built into every operation
- **Structured logging** — Consistent error reporting
- **Configuration** — Hierarchical with multiple sources
- **I18n** — Grammar-aware internationalization
- **IPC** — Service communication without tight coupling
- **Testing** — Test triplets (example + unit + bench) for every package

### 📝 What You Must Do:

- Use `core.Result` for all operations
- Use `core.Options` for all configuration
- Use `core.E()` / `core.NewError()` for all errors
- Use `core.Log()` for all logging
- Never import stdlib directly — use CoreGo wrappers

---

## 🏆 AX (Agent Experience) Principles in Practice

The 10 AX principles that shaped CoreGo:

1. **Predictable names** — `Config`, not `Cfg`; `Service`, not `Svc`
2. **Comments as usage examples** — All godoc shows real call sites
3. **Path is documentation** — `path/file.go` describes content from path alone
4. **Templates over freeform** — When a shape recurs, provide a template
5. **Declarative over imperative** — YAML/JSON for orchestration, Go for implementation
6. **Universal types** — `Options` in, `Result` out, `Service` for registration
7. **Directory as semantics** — Top-level dirs are categories, not bins
8. **Lib never imports consumer** — Dependencies flow one direction
9. **Issues are N+(rounds) deep** — Iteration is the discovery process
10. **CLI tests as artifact validation** — Binary tested against fixtures

**Example:** If `fmt.Sprintf` appears in CoreGo, agents will use it everywhere. So CoreGo uses `core.Sprintf` and bans direct `fmt` imports.

---

## 📚 File Structure

```
core/go/
├── core.go          # Main Core struct + accessors + lifecycle
├── options.go       # Options + Result types
├── result.go        # Result methods + constructors
├── service.go       # Service registration + lifecycle
├── action.go        # Named action registry
├── ipc.go           # Message bus (ACTION/QUERY/QUERYALL)
├── contract.go      # Message/Query types + interfaces
├── registry.go      # Thread-safe named collections
├── app.go           # Application identity
├── error.go         # Error system + mandatory error creation
├── log.go           # Structured logging
├── config.go        # Configuration management
├── data.go          # Embedded data registry
├── drive.go         # Transport handle registry
├── fs.go            # Filesystem abstraction
├── process.go       # Process execution
├── command.go       # CLI command tree
├── i18n.go          # Internationalization
├── context.go       # Context utilities
├── AGENTS.md        # Agent orientation + AX principles
├── ... (272 files total)
```

---

## 🔗 Related Packages

CoreGo is extended by ~48 `go-*` packages in the dAppCore ecosystem:

| Category | Key Packages |
|----------|--------------|
| I/O | go-io, go-store, go-cache, go-stream |
| Blockchain | go-blockchain, go-lns, go-miner, go-p2p |
| AI/ML | go-ml, go-mlx, go-rocm, go-cuda, go-inference, go-rag |
| Network | go-proxy, go-netops, go-dns |
| Build | go-build, go-devops, go-git, go-forge |
| GUI | go-gui, go-ide, go-webview |
| Agent | go-agent, go-mcp, go-ansible |

All follow the same AX principles and SPOR compliance.

---

## 🚀 Getting Started

### Minimal Application

```go
package main

import "dappco.re/go"

func main() {
    c := core.New(
        core.WithOption("name", "my-app"),
        core.WithOption("version", "1.0.0"),
    )
    
    // Register a service
    c.Service("greet", core.Service{
        OnStart: func() core.Result {
            core.Println("Hello from greet service!")
            return core.Result{OK: true}
        },
    })
    
    // Run
    c.Run()
}
```

### With CLI Commands

```go
func main() {
    c := core.New(
        core.WithOption("name", "my-cli"),
    )
    
    // Register command
    c.Action("greet", func(ctx context.Context, opts core.Options) core.Result {
        name := opts.String("name")
        if name == "" {
            name = "World"
        }
        core.Println(core.Sprintf("Hello, %s!", name))
        return core.Result{OK: true}
    })
    
    c.Run()
}
```

### With IPC

```go
func main() {
    c := core.New()
    
    // Service A: Broadcast messages
    c.Service("publisher", core.Service{
        OnStart: func() core.Result {
            go func() {
                for i := 0; i < 10; i++ {
                    c.ACTION(messages.Progress{Percent: i * 10})
                    time.Sleep(time.Second)
                }
            }()
            return core.Result{OK: true}
        },
    })
    
    // Service B: Receive messages
    c.RegisterAction(func(_ *core.Core, msg core.Message) core.Result {
        if p, ok := msg.(messages.Progress); ok {
            core.Println(core.Sprintf("Progress: %d%%", p.Percent))
        }
        return core.Result{OK: true}
    })
    
    c.Run()
}
```

---

## 📖 References

- **Source:** `https://github.com/dappcore/go`
- **Module:** `dappco.re/go`
- **Previous Path:** `dappco.re/go/core` (migration in progress)
- **Go Version:** 1.26.0+
- **Dependencies:** Zero external dependencies
- **AGENTS.md:** [`core/go/AGENTS.md`](file:///Users/snider/Code/core/go/AGENTS.md)
- **RFC:** [`plans/code/core/go/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)
- **SPOR Table:** [`docs/pkg/PACKAGE_STANDARDS.md`](file:///Users/snider/Code/meowmix/plans/docs/pkg/PACKAGE_STANDARDS.md)

---

## 🏷️ Metadata

```yaml
package: core
category: Framework
status: Production
stability: Stable
test_coverage: High (test triplets for all packages)
spor_compliance: 100%
zero_deps: true
go_version: 1.26.0
maintainer: dAppCore Team
knowledge_pack: CoreGo v1.3.0
documented_by: Purberus <purberus@lthn.ai>
documented_at: 2026-06-17
```

---

*Purrr... This is the foundation. Everything else builds on this.* 🐱
