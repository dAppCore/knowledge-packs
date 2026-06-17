---
type: Package Index
package: process
module: dappco.re/go/process
repo: core/go-process
title: go-process Package Index
description: Process orchestration, daemon management, and pipeline execution for CoreGO
tags:
  - process-management
  - daemon
  - lifecycle
  - pidfile
  - ipc
---

# go-process Package Index

> **Process Orchestration & Daemon Management for CoreGO**

**Repository:** `core/go-process`  
**Module:** `dappco.re/go/process`  
**Status:** ✅ Production-Ready  
**Windows Support:** Phase 1 (compile-compatible)  
**Test Triplets:** ✅ Complete  
**RFC:** [plans/code/core/go/process/RFC.md](../../../../../plans/code/core/go/process/RFC.md)
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| RFC | Windows support specification | [plans/code/core/go/process/RFC.md](../../../../../plans/code/core/go/process/RFC.md) |
| Repository README | Original package README | [~/Code/core/go-process/README.md](file:///Users/snider/Code/core/go-process/README.md) |

---

## 🎯 Package Overview

`go-process` is the **process orchestration framework** for CoreGO applications, providing comprehensive management of external commands with full lifecycle support. It enables spawning, monitoring, and controlling processes with Core IPC integration, output streaming, PID file management, health endpoints, and REST/WebSocket provider support.

### Core Capabilities

1. **Process Management** — Start, stop, kill, signal processes with full lifecycle tracking
2. **Daemon Lifecycle** — PID file management, graceful shutdown, health checks
3. **Pipeline Orchestration** — Run dependent processes with `After` dependencies
4. **Output Capture** — Ring buffer streaming with configurable size
5. **Event Broadcasting** — Core ACTION system integration for process events
6. **Registry Tracking** — JSON-based daemon registry for multi-instance coordination
7. **Cross-Platform** — Unix (POSIX) + Windows (Phase 1 stubs, Phase 2 Job Objects)
8. **REST API** — Comprehensive HTTP endpoints via `pkg/api`
9. **WebSocket Streaming** — Real-time process events via WebSocket

### Architecture Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                           │
│  Process orchestration, daemon management, pipeline execution │
├─────────────────────────────────────────────────────────────┤
│                    Service Layer                              │
│  Service — Process registry + Core ACTION integration        │
│  Runner — Pipeline orchestration with dependencies           │
│  Daemon — PID file + health server + graceful shutdown      │
├─────────────────────────────────────────────────────────────┤
│                    Process Layer                              │
│  ManagedProcess — Individual process with lifecycle state   │
│  Process — Alias for backward compatibility                 │
├─────────────────────────────────────────────────────────────┤
│                    Registry Layer                             │
│  Registry — JSON-based daemon tracking                        │
│  DaemonEntry — Individual daemon metadata                     │
├─────────────────────────────────────────────────────────────┤
│                    Platform Layer                             │
│  platform.go — Platform-agnostic helper signatures           │
│  platform_unix.go — POSIX signal and process group ops      │
│  platform_windows.go — Windows stubs (Phase 1)               │
├─────────────────────────────────────────────────────────────┤
│                    API Layer (pkg/api)                         │
│  ProcessProvider — REST + WebSocket provider                 │
│  Response types — Standardized API responses                  │
│  WebSocket bridge — Real-time process events                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 🏗️ Components

### Core Types

| Type | File | Purpose |
|------|------|---------|
| `ManagedProcess` | `process.go` | Individual managed process with lifecycle |
| `Process` | `process.go` | Alias for ManagedProcess (compatibility) |
| `Service` | `service.go` | Main service with Core integration |
| `Runner` | `runner.go` | Pipeline runner with dependency support |
| `Daemon` | `daemon.go` | Daemon lifecycle manager |
| `Registry` | `registry.go` | JSON-based daemon registry |
| `RingBuffer` | `buffer.go` | Output capture ring buffer |

### Status Types

| Type | Description | Values |
|------|-------------|--------|
| `Status` | Process lifecycle state | `pending`, `running`, `exited`, `failed`, `killed` |
| `Stream` | Output stream identifier | `stdout`, `stderr` |

### Configuration Types

| Type | File | Purpose |
|------|------|---------|
| `Options` | `service.go` | Service configuration (BufferSize) |
| `RunOptions` | `types.go` | Process execution options |
| `RunSpec` | `runner.go` | Pipeline specification with dependencies |
| `RunResult` | `runner.go` | Pipeline execution result |
| `DaemonOptions` | `daemon.go` | Daemon configuration |
| `DaemonEntry` | `registry.go` | Registry entry metadata |
| `HealthCheck` | `daemon.go` | Health check function |

### ACTION Message Types

| Type | File | Purpose |
|------|------|---------|
| `ActionProcessStarted` | `actions.go` | Process has started |
| `ActionProcessOutput` | `actions.go` | Process output line |
| `ActionProcessExited` | `actions.go` | Process has exited |
| `ActionProcessKilled` | `actions.go` | Process was killed |

### Error Types

| Error | File | Purpose |
|-------|------|---------|
| `ErrProcessNotFound` | `service.go` | Process not in registry |
| `ErrProcessNotRunning` | `service.go` | Process already completed |
| `ErrStdinNotAvailable` | `service.go` | stdin closed/unavailable |
| `ErrContextRequired` | `service.go` | Context is nil |
| `ErrUncatchableSignal` | `service.go` | Signal cannot be caught (SIGKILL, SIGSTOP) |
| `ErrRunnerNoService` | `runner.go` | Runner created without service |
| `ErrRunnerInvalidSpecName` | `runner.go` | Invalid RunSpec name |
| `ErrRunnerInvalidDependencyName` | `runner.go` | Invalid dependency name |
| `ErrRunnerContextRequired` | `runner.go` | Runner context is nil |

### Platform Helpers

| Function | Purpose |
|----------|---------|
| `processSignal(pid, sig)` | Send signal to process |
| `processKillGroup(pid)` | Kill process group |
| `processSignalGroup(pid, sig)` | Signal process group |
| `applyProcessGroup(cmd)` | Configure process group |
| `exitWasSignaled(status)` | Check if exit was by signal |
| `exitSignalName(status)` | Get signal name from status |
| `signalCannotBeCaught(sig)` | Check if signal is catchable |

---

## 📁 File Structure

```
go-process/
├── go/
│   ├── types.go                 # Core types (Status, Stream, RunOptions, etc.)
│   ├── process.go               # ManagedProcess type + core methods
│   ├── process_lifecycle.go    # Lifecycle: Start, Stop, Kill, Signal, Wait
│   ├── process_global.go       # Global process management helpers
│   ├── process_test.go         # Process tests
│   ├── process_example_test.go # Process usage examples
│   ├── process_global_test.go  # Global process tests
│   ├── process_global_example_test.go # Global examples
│   │
│   ├── service.go              # Service type + Core integration
│   ├── service_test.go         # Service tests
│   ├── service_example_test.go # Service usage examples
│   ├── service_handlers_test.go # Handler tests
│   ├── service_handletask_test.go # Task handler tests
│   │
│   ├── daemon.go                # Daemon lifecycle management
│   ├── daemon_test.go           # Daemon tests
│   ├── daemon_example_test.go  # Daemon usage examples
│   │
│   ├── runner.go                # Pipeline runner with dependencies
│   ├── runner_test.go           # Runner tests
│   ├── runner_example_test.go  # Runner usage examples
│   │
│   ├── registry.go              # JSON-based daemon registry
│   ├── registry_test.go         # Registry tests
│   ├── registry_example_test.go # Registry usage examples
│   ├── registry_corrupt_test.go # Corruption handling tests
│   │
│   ├── actions.go               # ACTION message types
│   ├── actions_test.go          # ACTION tests
│   ├── actions_example_test.go # ACTION usage examples
│   ├── actions_parse_test.go    # ACTION parsing tests
│   │
│   ├── health.go                # Health check server
│   ├── health_test.go           # Health tests
│   ├── health_example_test.go  # Health usage examples
│   │
│   ├── buffer.go                # Ring buffer for output capture
│   ├── buffer_test.go           # Buffer tests
│   ├── buffer_example_test.go  # Buffer usage examples
│   │
│   ├── errors.go                # Package-specific errors
│   ├── errors_test.go           # Error tests
│   ├── errors_example_test.go  # Error usage examples
│   │
│   ├── pidfile.go               # PID file management
│   ├── pidfile_test.go          # PID file tests
│   ├── pidfile_example_test.go  # PID file usage examples
│   ├── pidfile_unix.go          # Unix PID file operations
│   ├── pidfile_windows.go       # Windows PID file operations
│   ├── pidfile_ioerror_test.go  # PID file I/O error tests
│   │
│   ├── platform.go              # Platform helper signatures
│   ├── platform_unix.go         # Unix platform implementations
│   ├── platform_windows.go      # Windows platform implementations
│   ├── platform_unix_test.go    # Unix platform tests
│   ├── platform_windows_test.go # Windows platform tests
│   │
│   ├── program.go               # Program execution helpers
│   ├── program_test.go          # Program tests
│   ├── program_example_test.go # Program usage examples
│   │
│   ├── helpers_internal_test.go # Internal test helpers
│   ├── test_helpers_test.go     # Shared test utilities
│   │
│   ├── os_exec_link.go          # os/exec compatibility layer
│   │
│   ├── exec/
│   │   └── go/
│   │       ├── doc.go                # Package documentation
│   │       ├── exec.go              # Extended exec functionality
│   │       ├── exec_test.go          # Exec tests
│   │       ├── exec_example_test.go # Exec usage examples
│   │       ├── exec_internal_test.go # Internal exec tests
│   │       ├── logger.go             # Exec logger
│   │       ├── logger_test.go        # Logger tests
│   │       └── logger_example_test.go # Logger usage examples
│   │
│   └── pkg/
│       └── api/
│           ├── provider.go          # REST + WebSocket provider
│           ├── provider_test.go     # Provider tests
│           ├── provider_example_test.go # Provider usage examples
│           ├── provider_validation_test.go # Validation tests
│           ├── provider_kill_stop_test.go # Kill/stop tests
│           ├── response.go           # API response types
│           ├── response_test.go     # Response tests
│           ├── websocket.go          # WebSocket bridge
│           ├── websocket_test.go     # WebSocket tests
│           ├── websocket_bridge_test.go # Bridge tests
│           ├── test_helpers_test.go # Test utilities
│           ├── platform_unix.go     # Unix-specific API helpers
│           ├── platform_windows.go  # Windows-specific API helpers
│           ├── platform_unix_test.go # Unix API tests
│           └── ui/                   # GUI integration assets
│
├── go.mod
├── go.sum
├── go.work
├── RFC.md                     # Windows support RFC
├── README.md                  # Package README
├── AGENTS.md                  # Agent guidance
├── CLAUDE.md                  # Claude-specific notes
└── specs/                      # Additional specifications
```

---

## 🚀 Quick Start

### Basic Process Execution

```go
app, _ := core.New(
    core.WithService(process.NewService(process.Options{})),
)
svc := core.MustServiceFor[*process.Service](app, "process")
proc, _ := svc.Start(context.Background(), "echo", "hello")
<-proc.Done()
fmt.Println(proc.Output())
```

### Daemon Management

```go
daemon := process.NewDaemon(process.DaemonOptions{
    PIDFile:       "/var/run/myapp.pid",
    HealthAddr:    "127.0.0.1:8080",
    ShutdownTimeout: 30 * time.Second,
})
ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
defer stop()
daemon.Start(ctx, "myapp")
```

### Pipeline with Dependencies

```go
runner := process.NewRunner(svc)
specs := []process.RunSpec{
    {Name: "lint", Command: "golangci-lint", Args: []string{"run", "./..."}},
    {Name: "test", Command: "go", Args: []string{"test", "./..."}, After: []string{"lint"}},
    {Name: "build", Command: "go", Args: []string{"build", "./..."}, After: []string{"test"}},
}
results, _ := runner.Run(context.Background(), specs)
```

### Registry Tracking

```go
reg := process.NewRegistry("/tmp/daemons")
reg.Register(process.DaemonEntry{
    Code: "myapp", Daemon: "worker", PID: 12345,
    Health: "http://localhost:8080/health",
})
entries, _ := reg.List()
```

---

## 🌐 API Endpoints (pkg/api)

### Daemon Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/process/daemons` | List all registered daemons |
| GET | `/api/process/daemons/{code}/{daemon}` | Get specific daemon info |
| POST | `/api/process/daemons/{code}/{daemon}/stop` | Stop daemon |
| GET | `/api/process/daemons/{code}/{daemon}/health` | Health check |

### Process Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/process/processes` | List all running processes |
| POST | `/api/process/processes` | Start a new process |
| POST | `/api/process/processes/run` | Run a process pipeline |
| GET | `/api/process/processes/{id}` | Get process info |
| GET | `/api/process/processes/{id}/output` | Get process output |
| POST | `/api/process/processes/{id}/wait` | Wait for process completion |
| POST | `/api/process/processes/{id}/input` | Send input to stdin |
| POST | `/api/process/processes/{id}/close-stdin` | Close stdin |
| POST | `/api/process/processes/{id}/kill` | Kill process |

### WebSocket Events

| Channel | Description |
|---------|-------------|
| `process.daemon.started` | Daemon started |
| `process.daemon.stopped` | Daemon stopped |
| `process.daemon.health` | Daemon health status update |
| `process.started` | Process started |
| `process.output` | Process output line |
| `process.exited` | Process exited |
| `process.killed` | Process killed |

---

## 🎓 Use Cases

### 1. Mining Pool Daemon

```go
// Start mining pool with PID file and health checks
daemon := process.NewDaemon(process.DaemonOptions{
    PIDFile: "/var/run/go-pool.pid",
    HealthAddr: ":8080",
    HealthChecks: []process.HealthCheck{
        {Name: "redis", Check: checkRedisHealth},
        {Name: "daemon", Check: checkDaemonConnection},
    },
})
daemon.Start(ctx, "go-pool")
```

### 2. Build Pipeline

```go
// Run build pipeline: lint → test → build
specs := []process.RunSpec{
    {Name: "lint", Command: "golangci-lint", Args: []string{"run"}},
    {Name: "test", Command: "go", Args: []string{"test", "./..."}, After: []string{"lint"}},
    {Name: "build", Command: "go", Args: []string{"build"}, After: []string{"test"}},
}
results, _ := runner.Run(ctx, specs)
```

### 3. Service with WebSocket Events

```go
// Provider setup with WebSocket bridge
hub := ws.NewHub()
provider := api.NewProvider(process.DefaultRegistry(), svc, hub)
provider.Register(router)

// Clients receive real-time events via WebSocket
// Events: process.started, process.output, process.exited, etc.
```

### 4. Health Check Server

```go
// Daemon with health server
daemon := process.NewDaemon(process.DaemonOptions{
    HealthAddr: ":9090",
    HealthChecks: []process.HealthCheck{
        {Name: "database", Check: isDatabaseHealthy},
        {Name: "cache", Check: isCacheHealthy},
    },
})
daemon.Start(ctx, "myapp")
// Access health at: http://localhost:9090/health
```

### 5. Process Group Management

```go
// Start process in its own process group
proc, _ := svc.Start(ctx, "my-server", "--port", "8080",
    process.WithDetach(true),  // Detached process group
    process.WithKillGroup(true), // Kill entire group on stop
)
// Process survives parent death, entire group can be killed together
```

---

## 🔧 Configuration

### Service Options

```go
type Options struct {
    BufferSize int  // Ring buffer size for output capture (default: 1MB)
}

// Create service with custom buffer size
svc := process.NewService(process.Options{
    BufferSize: 2 * 1024 * 1024, // 2MB
})
```

### Daemon Options

```go
type DaemonOptions struct {
    PIDFile         string        // PID file path
    ShutdownTimeout time.Duration // Graceful shutdown timeout (default: 30s)
    HealthAddr      string        // Health server address (e.g., ":8080")
    HealthChecks    []HealthCheck // Custom health check functions
    Registry        *Registry     // Daemon registry
    RegistryEntry   DaemonEntry  // Entry for registry
}
```

### RunOptions

```go
type RunOptions struct {
    Command      string        // Executable to run
    Args         []string      // Arguments
    Dir          string        // Working directory
    Env          []string      // Environment variables
    DisableCapture bool        // Disable output capture
    Detach       bool        // Detached process group
    Timeout      time.Duration // Maximum run duration
}
```

---

## 🪟 Platform Support

### Unix (Linux, macOS, BSD)

✅ **Fully Supported**
- POSIX signal support (SIGTERM, SIGKILL, etc.)
- Process group management via Setpgid
- Signal-based process termination
- Exit status detection (signaled vs normal exit)
- Full process lifecycle control

### Windows (Phase 1)

✅ **Compile-Compatible**
- No syscall compilation errors
- Best-effort process termination
- Process group leader kill
- Stub implementations for unavailable features

⚠️ **Limitations (Phase 1)**
- No process group signals (stubbed to no-op)
- No exit signal detection (always returns false)
- Process group kill only affects leader

📋 **Phase 2 (Planned)**
- Job Object lifecycle management
- Full descendant tree termination
- Proper Windows process group semantics

---

## 🧪 Testing

### Test Structure

All components follow the AX standard with **test triplets**:
- `*_test.go` — Unit and integration tests
- `*_example_test.go` — Usage examples as tests

### Running Tests

```bash
# All tests
cd ~/Code/core/go-process/go
go test -v ./...

# With coverage
go test -cover ./...

# With race detector
go test -race ./...

# Benchmarks
go test -bench . -benchmem

# Specific package
go test -v ./pkg/api/...
```

### Test Coverage Areas

- ✅ Process lifecycle (start, stop, kill, signal)
- ✅ Service integration
- ✅ Daemon management
- ✅ Runner pipeline orchestration
- ✅ Registry operations
- ✅ Platform-specific behavior (Unix + Windows)
- ✅ API provider endpoints
- ✅ WebSocket bridge
- ✅ Health check server
- ✅ PID file management
- ✅ Buffer operations
- ✅ ACTION system integration

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| **Total Go files** | ~45 |
| **Test files** | ~35 |
| **Example test files** | ~20 |
| **Lines of code** | ~5,000+ |
| **Test coverage** | High |
| **Platform support** | Unix + Windows |
| **API endpoints** | 15+ |
| **WebSocket channels** | 7 |
| **Core ACTION handlers** | 5 |
| **Configuration options** | 3 types |
| **Error types** | 9 |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-p2p](../p2p/) | P2P networking with process management | ../p2p/ |
| [go-proxy](../proxy/) | Mining proxy (uses go-process) | ../proxy/ |
| [go-pool](../pool/) | Mining pool backend (uses go-process) | ../pool/ |
| [go-io](../io/) | I/O abstraction layer | ../io/ |
| [CoreGO INDEX](../../INDEX.md) | Complete package catalog | ../../INDEX.md |

---

## 📈 Quality Metrics

- ✅ **SPOR Compliance** — Single Point Of Responsibility for all types
- ✅ **Test Triplets** — Every public type has _test.go + _example_test.go
- ✅ **RFC Verified** — Windows support specification complete
- ✅ **Core Integration** — Full Core ACTION system support
- ✅ **Cross-Platform** — Unix + Windows support
- ✅ **Documentation** — Complete README + INDEX

---

## 📝 Changelog

| Date | Change | Commit |
|------|--------|--------|
| 2026-06-17 | Complete knowledge pack documentation | N/A |
| 2026-05-12 | Windows Phase 1 RFC published | N/A |
| 2026-04-30 | Package structure finalized | N/A |
| 2026-03-XX | Initial package creation | N/A |

---

## 🎯 Tags

```yaml
- process-management
- daemon-management
- lifecycle
- pidfile
- ipc
- health-check
- cross-platform
- windows
- unix
- posix
- signals
- process-group
- job-object
- corego
- action-system
- rest-api
- websocket
- registry
- orchestration
- pipeline
- dependencies
- ring-buffer
- stdout
- stderr
- production-ready
- high-coverage
```

---

## 📚 References

1. **RFC Specification** — [plans/code/core/go/process/RFC.md](../../../../../plans/code/core/go/process/RFC.md)
2. **Repository** — [~/Code/core/go-process/](file:///Users/snider/Code/core/go-process/)
3. **CoreGO Framework** — [CoreGO INDEX](../../INDEX.md)
4. **AX Standard** — Single Point Of Responsibility pattern

---

*Package index generated: 2026-06-17T18:30:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
