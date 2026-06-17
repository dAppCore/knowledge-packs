---
type: Package Documentation
package: process
module: dappco.re/go/process
repo: core/go-process
lang: go
tags:
  - process-management
  - daemon
  - lifecycle
  - pidfile
  - ipc
  - health-check
  - cross-platform
  - windows
  - unix
---
# go-process — Process Orchestration & Daemon Management

> **The authoritative process management framework for CoreGO applications**

**RFC:** [plans/code/core/go/process/RFC.md](../../../../../plans/code/core/go/process/RFC.md)
**Source:** [~/Code/core/go-process/](file:///Users/snider/Code/core/go-process/)
**Module:** `dappco.re/go/process`
**Dependencies:** `dappco.re/go`
**Status:** ✅ Production-Ready (Windows Phase 1 compatible)

---

## 🎯 Overview

`go-process` provides **comprehensive process orchestration** for CoreGO applications, enabling spawning, monitoring, and controlling external commands with full lifecycle management. It wraps external command execution with Core IPC integration, output streaming, PID file management, health endpoints, and REST/WebSocket provider support.

### Primary Use Cases

1. **Daemon management** — Start/stop/restart long-running processes with PID files
2. **Command pipelines** — Run dependent processes with orchestration
3. **Output capture** — Stream and buffer process stdout/stderr
4. **Health monitoring** — HTTP health check endpoints for process state
5. **Cross-platform** — Unix (POSIX signals) + Windows (Phase 1 stubs, Phase 2 Job Objects planned)
6. **CoreGO integration** — Native ACTION system for process events
7. **Registry tracking** — JSON-based daemon registry for multi-instance coordination

### Design Philosophy

- **Single responsibility** — Each component handles one aspect of process management
- **Core-native** — Full integration with Core ACTION system and Result pattern
- **Platform-agnostic** — Abstract platform differences behind helper functions
- **Observable** — Rich event streaming via Core ACTIONs
- **Testable** — Complete test triplet coverage (_test.go + _example_test.go)

---

## 🏗️ Architecture

### Component Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                           │
│  Process orchestration, daemon management, pipeline execution │
├─────────────────────────────────────────────────────────────┤
│                    Service Layer                              │
│  Service — Process registry, Core ACTION integration        │
│  Runner — Pipeline orchestration with dependencies           │
│  Daemon — PID file, health server, graceful shutdown         │
├─────────────────────────────────────────────────────────────┤
│                    Process Layer                              │
│  ManagedProcess — Individual process with lifecycle state   │
│  Process — Alias for ManagedProcess (compatibility)          │
├─────────────────────────────────────────────────────────────┤
│                    Registry Layer                             │
│  Registry — JSON-based daemon tracking                        │
│  DaemonEntry — Individual daemon metadata                     │
├─────────────────────────────────────────────────────────────┤
│                    Platform Layer                             │
│  platform.go — Platform-agnostic helper signatures           │
│  platform_unix.go — POSIX signal and process group ops       │
│  platform_windows.go — Windows stubs (Phase 1)               │
├─────────────────────────────────────────────────────────────┤
│                    API Layer (pkg/api)                         │
│  ProcessProvider — REST + WebSocket provider                 │
│  Response types — Standardized API responses                  │
│  WebSocket bridge — Real-time process events                 │
└─────────────────────────────────────────────────────────────┘
```

### Core Integration

The package integrates with CoreGO via:
- **ServiceRuntime** — Standard Core service lifecycle
- **ACTION system** — Process events broadcast to registered handlers
- **Result pattern** — All operations return `core.Result`
- **IPC bus** — Internal communication via Core ACTIONs

---

## 📦 Package Structure

```
go-process/
├── go/
│   ├── process.go              # ManagedProcess type + core methods
│   ├── process_lifecycle.go    # Lifecycle management (Start, Stop, Kill)
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
│   ├── types.go                 # Core types (Status, Stream, RunOptions, etc.)
│   ├── types_test.go            # Type tests
│   ├── types_example_test.go   # Type usage examples
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
│   │       ├── exec.go              # Extended exec functionality
│   │       ├── exec_test.go          # Exec tests
│   │       ├── exec_example_test.go # Exec usage examples
│   │       ├── exec_internal_test.go # Internal exec tests
│   │       ├── logger.go             # Exec logger
│   │       ├── logger_test.go        # Logger tests
│   │       ├── logger_example_test.go # Logger usage examples
│   │       └── doc.go                # Package documentation
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
│           └── ui/                   # GUI integration
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

## 🚀 Getting Started

### Basic Process Execution

```go
package main

import (
    "context"
    "fmt"

    core "dappco.re/go"
    "dappco.re/go/process"
)

func main() {
    // Create Core application
    app, _ := core.New(
        core.WithName("myapp"),
        core.WithService(process.NewService(process.Options{})),
    )

    // Get process service
    svc := core.MustServiceFor[*process.Service](app, "process")

    // Start a process
    ctx := context.Background()
    proc, r := svc.Start(ctx, "echo", "hello", "world")
    if !r.OK {
        fmt.Println("Error:", r.Value)
        return
    }

    // Wait for completion
    <-proc.Done()

    // Get output
    output := proc.Output()
    fmt.Println("Output:", output)

    // Get process info
    info := proc.Info()
    fmt.Printf("Exit code: %d\n", info.ExitCode)
}
```

### Daemon Management

```go
package main

import (
    "context"
    "signal"
    "syscall"
    "time"

    "dappco.re/go/process"
)

func main() {
    // Create daemon with PID file and health server
    daemon := process.NewDaemon(process.DaemonOptions{
        PIDFile:       "/var/run/myapp.pid",
        HealthAddr:    "127.0.0.1:8080",
        ShutdownTimeout: 30 * time.Second,
    })

    // Set up health checks
    daemon.Opts.HealthChecks = append(daemon.Opts.HealthChecks, process.HealthCheck{
        Name: "database",
        Check: func() bool {
            return checkDatabaseHealth()
        },
    })

    // Start daemon
    ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
    defer stop()

    if r := daemon.Start(ctx, "myapp"); !r.OK {
        panic(r.Value)
    }

    // Your application logic here
    // ...

    // Daemon will handle graceful shutdown on signal
}
```

### Pipeline Execution with Dependencies

```go
package main

import (
    "context"
    "fmt"

    core "dappco.re/go"
    "dappco.re/go/process"
)

func main() {
    app, _ := core.New(
        core.WithService(process.NewService(process.Options{})),
    )

    svc := core.MustServiceFor[*process.Service](app, "process")
    runner := process.NewRunner(svc)

    // Define pipeline with dependencies
    specs := []process.RunSpec{
        {
            Name:    "lint",
            Command: "golangci-lint",
            Args:    []string{"run", "./..."},
        },
        {
            Name:    "test",
            Command: "go",
            Args:    []string{"test", "./..."},
            After:   []string{"lint"}, // Run after lint
        },
        {
            Name:    "build",
            Command: "go",
            Args:    []string{"build", "./..."},
            After:   []string{"test"}, // Run after test
        },
    }

    // Run pipeline
    results, r := runner.Run(context.Background(), specs)
    if !r.OK {
        fmt.Println("Error:", r.Value)
        return
    }

    // Check results
    for _, result := range results {
        if result.Passed() {
            fmt.Printf("✓ %s passed\n", result.Name)
        } else {
            fmt.Printf("✗ %s failed: %s\n", result.Name, result.Output)
        }
    }
}
```

### Core ACTION Integration

```go
package main

import (
    "fmt"

    core "dappco.re/go"
    "dappco.re/go/process"
)

func main() {
    app, _ := core.New(
        core.WithService(process.NewService(process.Options{})),
    )

    // Register custom ACTION handler for process events
    app.RegisterAction(func(c *core.Core, msg core.Message) core.Result {
        switch m := msg.(type) {
        case process.ActionProcessOutput:
            fmt.Printf("[%s] %s: %s\n", m.ID, m.Stream, m.Line)
        case process.ActionProcessExited:
            fmt.Printf("[%s] exited with code %d\n", m.ID, m.ExitCode)
        case process.ActionProcessStarted:
            fmt.Printf("[%s] started (PID: %d)\n", m.ID, m.PID)
        }
        return core.Ok(nil)
    })

    // Start service
    svc := core.MustServiceFor[*process.Service](app, "process")
    
    // Start a process - events will be sent via ACTION system
    proc, _ := svc.Start(context.Background(), "sleep", "5")
    <-proc.Done()
}
```

### Registry Tracking

```go
package main

import (
    "fmt"

    "dappco.re/go/process"
)

func main() {
    // Create registry
    reg := process.NewRegistry("/tmp/daemons")

    // Register a daemon
    r := reg.Register(process.DaemonEntry{
        Code:    "myapp",
        Daemon:  "worker",
        PID:     12345,
        Health:  "http://localhost:8080/health",
        Project: "myproject",
        Binary:  "/usr/local/bin/myapp",
    })
    if !r.OK {
        fmt.Println("Registration failed:", r.Value)
        return
    }

    // List all daemons
    entries, r := reg.List()
    if r.OK {
        for _, entry := range entries {
            fmt.Printf("%s/%s (PID: %d)\n", entry.Code, entry.Daemon, entry.PID)
        }
    }

    // Find specific daemon
    entry, r := reg.Find("myapp", "worker")
    if r.OK {
        fmt.Printf("Found: %s/%s\n", entry.Code, entry.Daemon)
    }
}
```

---

## 🔧 Core Types

### ManagedProcess

Represents a managed external process with full lifecycle tracking.

```go
type ManagedProcess struct {
    ID        string
    Command   string
    Args      []string
    Dir       string
    Env       []string
    StartedAt time.Time
    Status    Status        // pending, running, exited, failed, killed
    ExitCode  int
    Duration  time.Duration
    
    // Internal
    cmd          *core.Cmd
    ctx          context.Context
    cancel       context.CancelFunc
    output       *RingBuffer
    stdin        io.WriteCloser
    done         chan struct{}
    mu           sync.RWMutex
    gracePeriod  time.Duration
    killGroup    bool
    killNotified bool
    killSignal   string
}

// Process is an alias for compatibility
type Process = ManagedProcess
```

**Key Methods:**
- `Info() Info` — Get process snapshot
- `Output() string` — Get captured output
- `OutputBytes() []byte` — Get output as bytes
- `SendInput(data []byte) core.Result` — Send data to stdin
- `CloseStdin() core.Result` — Close stdin
- `Kill() core.Result` — Kill process
- `Signal(sig syscall.Signal) core.Result` — Send signal
- `Wait() core.Result` — Wait for completion

### Status Constants

```go
const (
    StatusPending Status = "pending"
    StatusRunning Status = "running"
    StatusExited  Status = "exited"
    StatusFailed  Status = "failed"
    StatusKilled  Status = "killed"
)
```

### Stream Constants

```go
const (
    StreamStdout Stream = "stdout"
    StreamStderr Stream = "stderr"
)
```

### RunOptions

```go
type RunOptions struct {
    Command    string
    Args       []string
    Dir        string
    Env        []string
    DisableCapture bool
    Detach     bool  // Process survives parent death
    Timeout    time.Duration
}
```

### RunSpec

Pipeline specification with dependency support.

```go
type RunSpec struct {
    Name         string
    Command      string
    Args         []string
    Dir          string
    Env          []string
    After        []string  // Dependencies
    AllowFailure bool
}
```

### RunResult

```go
type RunResult struct {
    Name     string
    Spec     RunSpec
    ExitCode int
    Duration time.Duration
    Output   string
    Error    error
    Skipped  bool
}

func (r RunResult) Passed() bool {
    return !r.Skipped && r.Error == nil && r.ExitCode == 0
}
```

### DaemonOptions

```go
type DaemonOptions struct {
    PIDFile         string
    ShutdownTimeout time.Duration
    HealthAddr      string
    HealthChecks    []HealthCheck
    Registry        *Registry
    RegistryEntry   DaemonEntry
}
```

### DaemonEntry

```go
type DaemonEntry struct {
    Code    string
    Daemon  string
    PID     int
    Health  string
    Project string
    Binary  string
    Started time.Time
}
```

### HealthCheck

```go
type HealthCheck struct {
    Name string
    Check func() bool
}
```

---

## 🌐 ACTION System Integration

Process events are broadcast via Core ACTION system with the following message types:

### Action Types

| Type | Description | Fields |
|------|-------------|--------|
| `ActionProcessStarted` | Process has started | ID, PID, Command, Args, StartedAt |
| `ActionProcessOutput` | Process output line | ID, Stream, Line, Timestamp |
| `ActionProcessExited` | Process has exited | ID, ExitCode, Duration, Error |
| `ActionProcessKilled` | Process was killed | ID, Signal, ExitCode |

### Registration

The service automatically registers these ACTION handlers on startup:
- `process.run` — Run a process
- `process.start` — Start a process
- `process.kill` — Kill a process
- `process.list` — List all processes
- `process.get` — Get process info

---

## 🎛️ Service Configuration

### Options

```go
type Options struct {
    // BufferSize is the ring buffer size for output capture.
    // Default: 1MB (1024 * 1024 bytes).
    BufferSize int
}
```

### Default Configuration

```go
svc := process.NewService(process.Options{})
// Uses 1MB buffer, integrates with Core ACTION system
```

### Custom Configuration

```go
svc := process.NewService(process.Options{
    BufferSize: 2 * 1024 * 1024, // 2MB buffer
})
```

---

## 🔌 REST API (pkg/api)

The `pkg/api` package provides HTTP REST endpoints and WebSocket streaming for process management.

### Provider Configuration

```go
import "dappco.re/go/process/pkg/api"

// Create provider with registry and service
provider := api.NewProvider(
    process.DefaultRegistry(),
    svc,
    hub, // Optional WebSocket hub
)

// Register with Gin router
router := gin.Default()
provider.Register(router)
```

### Routes

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/process/daemons` | List all daemons |
| GET | `/api/process/daemons/{code}/{daemon}` | Get daemon info |
| POST | `/api/process/daemons/{code}/{daemon}/stop` | Stop daemon |
| GET | `/api/process/daemons/{code}/{daemon}/health` | Health check |
| GET | `/api/process/processes` | List all processes |
| POST | `/api/process/processes` | Start a process |
| POST | `/api/process/processes/run` | Run a pipeline |
| GET | `/api/process/processes/{id}` | Get process info |
| GET | `/api/process/processes/{id}/output` | Get process output |
| POST | `/api/process/processes/{id}/wait` | Wait for process |
| POST | `/api/process/processes/{id}/input` | Send input to stdin |
| POST | `/api/process/processes/{id}/close-stdin` | Close stdin |
| POST | `/api/process/processes/{id}/kill` | Kill process |

### WebSocket Events

Real-time events via WebSocket:
- `process.daemon.started` — Daemon started
- `process.daemon.stopped` — Daemon stopped
- `process.daemon.health` — Daemon health status
- `process.started` — Process started
- `process.output` — Process output
- `process.exited` — Process exited
- `process.killed` — Process killed

---

## 🪟 Platform Support

### Unix (Linux, macOS, BSD)

- ✅ Full POSIX signal support
- ✅ Process group management (Setpgid)
- ✅ Signal-based process termination
- ✅ Exit status detection (signaled vs exited)

### Windows (Phase 1)

- ✅ Compile compatibility (no syscall errors)
- ✅ Best-effort process termination
- ✅ Process group leader kill
- ⚠️ No process group signals (stubbed)
- ⚠️ No exit signal detection (returns false)

**Phase 2 (Planned):**
- Job Object lifecycle management
- Full descendant tree termination
- Proper Windows process group semantics

See [RFC.md](../../../../../plans/code/core/go/process/RFC.md) for details.

---

## 🧪 Testing

All subsystems have comprehensive test coverage following the AX standard (test triplets).

### Running Tests

```bash
cd ~/Code/core/go-process/go
go test -v ./...
go test -cover ./...
go test -bench . -benchmem
```

### Test Coverage

- Process lifecycle tests
- Service integration tests
- Daemon management tests
- Runner pipeline tests
- Registry tests
- Platform-specific tests (Unix + Windows)
- API provider tests
- WebSocket bridge tests

---

## 📊 Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/process` |
| **Repository** | `core/go-process` |
| **Language** | Go 1.26+ |
| **Dependencies** | `dappco.re/go`, `github.com/gin-gonic/gin` (pkg/api) |
| **Test Triplets** | ✅ Complete |
| **RFC Compliance** | ✅ Verified |
| **Documentation** | ✅ Complete |
| **Windows Support** | Phase 1 (compile-compatible) |
| **Platform Abstracted** | ✅ Unix + Windows |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-service](../service/) | Service management patterns | (future) |
| [go-io](../io/) | I/O abstraction layer | ../io/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |
| [go-p2p](../p2p/) | P2P process management | ../p2p/ |
| [go-proxy](../proxy/) | Mining proxy processes | ../proxy/ |

---

## 📝 Changelog

| Date | Change | Commit |
|------|--------|--------|
| 2026-06-17 | Complete knowledge pack documentation | N/A |
| 2026-05-12 | Windows RFC published | N/A |
| 2026-04-30 | Package structure finalized | N/A |

---

## 🎯 Tags

```yaml
- process-management
- daemon
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
- production-ready
```

---

## 📈 Statistics

| Metric | Value |
|--------|-------|
| Total Go files | ~45 |
| Test files | ~35 |
| Example test files | ~20 |
| Lines of code | ~5,000+ |
| Test coverage | High |
| Platform support | Unix + Windows |
| API endpoints | 15+ |
| WebSocket channels | 7 |

---

*Package documentation generated: 2026-06-17T18:30:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
