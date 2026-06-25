---
type: Package Documentation
package: mcp
module: dappco.re/go/mcp
repo: core/mcp
lang: go
tags:
  - mcp
  - model-context-protocol
  - server
  - ai-agent
  - tools
  - subsystems
  - transport
  - websocket
  - stdio
  - tcp
  - http
---
# go-mcp — Model Context Protocol Server Framework

**RFC:** [plans/code/core/mcp/RFC.md](../../../../../plans/code/core/mcp/RFC.md) (if exists)
**Source:** [~/Code/core/mcp/](file:///Users/snider/Code/core/mcp/)
**Module:** `dappco.re/go/mcp`
**PHP Package:** `forge.lthn.ai/core/mcp` (Laravel package in php/)
**Dependencies:** `dappco.re/go`, `dappco.re/go/process`, `dappco.re/go/ws`, `github.com/modelcontextprotocol/go-sdk/mcp`
**Status:** ✅ Production-Ready
**License:** EUPL-1.2

---

## Overview

`go-mcp` provides a **lightweight MCP (Model Context Protocol) server** implementation for CoreGO applications, enabling AI agents and IDEs to interact with codebases, tools, and resources through a standardized protocol. The package implements the MCP specification with file operations, tool registration, notification broadcasting, and channel-based event streaming.

### Primary Use Cases

1. **AI Agent Integration** — Enable AI assistants (Claude Code, Cursor, etc.) to interact with your codebase
2. **Tool Registration** — Expose custom tools and commands to MCP clients
3. **File Operations** — Sandboxed file system access for MCP clients
4. **Notification Broadcasting** — Real-time notifications to connected sessions
5. **Channel Events** — Pub/sub event system for cross-component communication
6. **Multi-Transport Support** — Stdio, TCP, HTTP, WebSocket transports
7. **Subsystem Architecture** — Modular tool groups (brain, agentic, ide)

### Design Philosophy

- **Lightweight** — Minimal overhead, designed for CLI use
- **Extensible** — Subsystem architecture for adding tool groups
- **Secure** — Sandboxed file operations with workspace restrictions
- **Observable** — Rich notification and channel event system
- **Transport-agnostic** — Multiple transport backends (stdio, TCP, HTTP, WebSocket)
- **Core-native** — Full integration with CoreGO framework

### Repository Layout

```
core/mcp/
├── go/      — Go module (dappco.re/go/mcp)
├── php/     — PHP Laravel package
├── docs/    — Shared documentation
└── composer.json
```

---

## Architecture

### Component Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    MCP Server Layer                          │
│  Service — Main MCP server with transport management       │
│  Server — MCP protocol server (go-sdk)                     │
│  Implementation — Server capabilities and metadata           │
├─────────────────────────────────────────────────────────────┤
│                    Transport Layer                           │
│  transport_stdio.go — Standard I/O transport                │
│  transport_tcp.go — TCP socket transport                     │
│  transport_http.go — HTTP/REST transport                    │
│  transport_unix.go — Unix domain socket transport           │
├─────────────────────────────────────────────────────────────┤
│                    Tool Layer                               │
│  tools_files.go — File operations (read, write, delete)     │
│  tools_process.go — Process management tools               │
│  tools_metrics.go — Metrics recording and querying          │
│  tools_rag.go — RAG (Retrieval-Augmented Generation) tools    │
│  tools_webview.go — WebView integration tools               │
│  tools_ws.go — WebSocket client tools                       │
├─────────────────────────────────────────────────────────────┤
│                    Subsystem Layer                           │
│  subsystem.go — Subsystem interface                         │
│  brain/ — OpenBrain integration (recall, remember, forget)    │
│  agentic/ — Agent dispatch and management                   │
│  ide/ — IDE bridge to Laravel backend                       │
├─────────────────────────────────────────────────────────────┤
│                    Notification Layer                        │
│  notify.go — Notification broadcasting                      │
│  channel.go — Channel event handling                        │
│  progress.go — Progress notification helpers                │
├─────────────────────────────────────────────────────────────┤
│                    Utility Layer                             │
│  registry.go — Tool and resource registry                   │
│  ipc.go — Inter-process communication                       │
│  compliance_helpers.go — Compliance checking utilities       │
│  result_helpers.go — Result handling utilities              │
└─────────────────────────────────────────────────────────────┘
```

### Integration with CoreGO

The package integrates with CoreGO via:
- **ServiceRuntime** — Standard Core service lifecycle
- **Process Service** — Optional integration with go-process for process management
- **WebSocket Hub** — Optional integration with go-ws for real-time events
- **Result Pattern** — All operations return `core.Result`

---

## Package structure

```
core/mcp/go/
├── cmd/
│   ├── core-mcp/
│   │   └── main.go           # Main MCP server binary
│   ├── openbrain-mcp/
│   │   └── main.go           # OpenBrain MCP server
│   └── brain-seed/
│       └── main.go           # Brain seed utility
│
├── pkg/
│   └── mcp/
│       ├── mcp.go                 # Main Service type
│       ├── mcp_test.go             # Service tests
│       ├── mcp_example_test.go     # Service usage examples
│       │
│       ├── server.go              # MCP server wrapper
│       ├── server_test.go          # Server tests
│       │
│       ├── subsystem.go           # Subsystem interface
│       ├── subsystem_test.go       # Subsystem tests
│       ├── subsystem_example_test.go # Subsystem examples
│       │
│       ├── register.go            # Tool/resource registration
│       ├── register_test.go        # Registration tests
│       ├── register_example_test.go # Registration examples
│       │
│       ├── ipc.go                 # Inter-process communication
│       ├── ipc_test.go             # IPC tests
│       │
│       ├── notify.go              # Notification broadcasting
│       ├── notify_test.go          # Notification tests
│       ├── notify_example_test.go # Notification examples
│       │
│       ├── progress.go            # Progress notification helpers
│       ├── progress_test.go        # Progress tests
│       ├── progress_example_test.go # Progress examples
│       │
│       ├── channel.go             # Channel event handling
│       │
│       ├── result_helpers.go      # Result handling utilities
│       │
│       ├── compliance_helpers.go  # Compliance checking
│       │
│       ├── string_constants.go    # String constants
│       │
│       ├── test_constants_test.go # Test constants
│       │
│       ├── tools_files.go         # File operation tools
│       ├── tools_files_test.go     # File tools tests
│       │
│       ├── tools_process.go       # Process management tools
│       ├── tools_process_test.go   # Process tools tests
│       │
│       ├── tools_metrics.go       # Metrics tools
│       ├── tools_metrics_test.go   # Metrics tools tests
│       │
│       ├── tools_rag.go           # RAG tools
│       ├── tools_rag_test.go       # RAG tools tests
│       │
│       ├── tools_webview.go       # WebView tools
│       ├── tools_webview_test.go   # WebView tools tests
│       │
│       ├── tools_ws.go             # WebSocket tools
│       ├── tools_ws_test.go       # WebSocket tools tests
│       │
│       ├── transport_stdio.go      # Stdio transport
│       ├── transport_stdio_test.go  # Stdio transport tests
│       │
│       ├── transport_tcp.go        # TCP transport
│       ├── transport_tcp_test.go    # TCP transport tests
│       │
│       ├── transport_http.go       # HTTP transport
│       ├── transport_http_test.go   # HTTP transport tests
│       │
│       ├── transport_unix.go       # Unix socket transport
│       ├── transport_unix_test.go   # Unix socket tests
│       │
│       ├── transformer.go          # Request/response transformers
│       ├── transformer_test.go      # Transformer tests
│       │
│       ├── transformer_anthropic.go # Anthropic-specific transformer
│       ├── transformer_anthropic_test.go
│       │
│       ├── transformer_openai.go   # OpenAI-specific transformer
│       ├── transformer_openai_test.go
│       │
│       ├── transformer_honeypot.go # Honeypot transformer
│       ├── transformer_honeypot_test.go
│       │
│       ├── upstream_balancer.go    # Upstream load balancer
│       ├── upstream_balancer_test.go
│       │
│       ├── upstream_registry.go    # Upstream registry
│       ├── upstream_registry_test.go
│       │
│       ├── upstream_router.go       # Upstream router
│       ├── upstream_router_test.go
│       │
│       ├── upstream_transport.go   # Upstream transport
│       ├── upstream_transport_test.go
│       │
│       └── agentic/
│           ├── dispatch.go         # Agent dispatch
│           ├── dispatch_test.go     # Dispatch tests
│           ├── epic.go             # Epic management
│           ├── epic_test.go         # Epic tests
│           ├── issue.go            # Issue tracking
│           ├── issue_test.go        # Issue tests
│           ├── mirror.go           # Repository mirroring
│           ├── mirror_test.go       # Mirror tests
│           ├── plan.go             # Plan management
│           ├── plan_test.go         # Plan tests
│           ├── prep.go             # PR preparation
│           ├── prep_test.go         # PR tests
│           ├── pr.go               # Pull request management
│           ├── pr_test.go           # PR tests
│           ├── queue.go            # Queue management
│           ├── queue_test.go        # Queue tests
│           ├── resume.go           # Resume tracking
│           ├── resume_test.go       # Resume tests
│           ├── review_queue.go      # Review queue
│           ├── review_queue_test.go
│           ├── scan.go             # Repository scanning
│           ├── scan_test.go         # Scan tests
│           ├── status.go           # Status tracking
│           ├── status_test.go       # Status tests
│           ├── watch.go            # Watch functionality
│           ├── watch_test.go        # Watch tests
│           ├── write_atomic.go     # Atomic write
│           ├── helpers_test.go      # Helper tests
│           └── repo_helpers.go     # Repository helpers
│       │
│       └── brain/
│           ├── brain.go             # Brain subsystem
│           ├── brain_test.go         # Brain tests
│           ├── client/
│           │   ├── client.go         # Brain client
│           │   ├── client_test.go     # Client tests
│           │   └── coreio_compat.go  # CoreIO compatibility
│           ├── bridge_events.go     # Bridge events
│           ├── direct.go           # Direct operations
│           ├── direct_test.go       # Direct tests
│           ├── provider.go         # Brain provider
│           ├── provider_test.go     # Provider tests
│           ├── tools.go            # Brain tools
│           └── tools_test.go        # Brain tools tests
│       │
│       └── ide/
│           ├── bridge.go           # IDE bridge
│           ├── bridge_test.go       # Bridge tests
│           ├── config.go            # IDE configuration
│           ├── config_test.go        # Config tests
│           ├── ide.go               # IDE subsystem
│           ├── ide_test.go           # IDE tests
│           ├── ide_data.go          # IDE data types
│           ├── ide_data_test.go      # IDE data tests
│           ├── tools_build.go       # Build tools
│           ├── tools_build_test.go  # Build tools tests
│           ├── tools_chat.go        # Chat tools
│           ├── tools_chat_test.go   # Chat tools tests
│           └── tools_dashboard.go   # Dashboard tools
│
├── go.mod
├── go.sum
├── go.work
├── AGENTS.md
├── CLAUDE.md
├── README.md
├── RFC.md (if exists)
└── docs/
```

---

## Getting started

### Basic MCP Server

```go
package main

import (
    "context"
    "os"
    "os/signal"
    "syscall"

    "dappco.re/go/mcp"
)

func main() {
    // Create MCP service with default options
    svc, err := mcp.New(mcp.Options{
        WorkspaceRoot: ".", // Restrict to current directory
    })
    if err != nil {
        panic(err)
    }

    // Handle graceful shutdown
    ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
    defer stop()

    // Run server (uses stdio by default for Claude Code integration)
    if err := svc.Run(ctx); err != nil {
        os.Exit(1)
    }
}
```

### With Custom Configuration

```go
package main

import (
    "context"

    core "dappco.re/go"
    "dappco.re/go/mcp"
    "dappco.re/go/process"
    "dappco.re/go/ws"
)

func main() {
    // Create process service for process management tools
    app, _ := core.New(
        core.WithService(process.NewService(process.Options{})),
    )
    procSvc := core.MustServiceFor[*process.Service](app, "process")

    // Create WebSocket hub for real-time events
    wsHub := ws.NewHub()
    go wsHub.Run(context.Background())

    // Create MCP service with all options
    svc, err := mcp.New(mcp.Options{
        WorkspaceRoot:  "/path/to/project",
        Unrestricted:   false, // Keep sandboxing enabled
        ProcessService: procSvc,
        WSHub:          wsHub,
        Subsystems:     []mcp.Subsystem{
            &mcp.BrainSubsystem{},
            &mcp.AgenticSubsystem{},
        },
    })
    if err != nil {
        panic(err)
    }

    // Run with specific transport
    if err := svc.Run(context.Background()); err != nil {
        panic(err)
    }
}
```

### With TCP Transport

```go
package main

import (
    "context"
    "os"

    "dappco.re/go/mcp"
)

func main() {
    svc, err := mcp.New(mcp.Options{
        WorkspaceRoot: ".",
    })
    if err != nil {
        panic(err)
    }

    // Set environment variable for TCP transport
    os.Setenv("MCP_ADDR", "tcp://localhost:8080")

    if err := svc.Run(context.Background()); err != nil {
        panic(err)
    }
}
```

### With HTTP Transport (REST API)

```go
package main

import (
    "context"
    "os"

    "dappco.re/go/mcp"
)

func main() {
    svc, err := mcp.New(mcp.Options{
        WorkspaceRoot: ".",
    })
    if err != nil {
        panic(err)
    }

    // Set environment variables for HTTP transport
    os.Setenv("MCP_HTTP_ADDR", "localhost:8081")
    os.Setenv("MCP_AUTH_TOKEN", "your-secret-token")

    if err := svc.Run(context.Background()); err != nil {
        panic(err)
    }
}
```

---

## Core types

### Service

The main MCP service type that manages the server, transports, and subsystems.

```go
type Service struct {
    *core.ServiceRuntime[struct{}] // Core access via s.Core()
    
    server         *mcp.Server
    workspaceRoot  string
    medium         *coreMedium
    subsystems     []Subsystem
    logger         *core.Log
    processService *process.Service
    wsHub          *ws.Hub
    wsServer       *http.Server
    wsAddr         string
    wsMu           sync.Mutex
    processMu      sync.Mutex
    processMeta    map[string]processRuntime
    tools          []ToolRecord
}
```

**Key Methods:**
- `New(opts Options) (*Service, error)` — Create new MCP service
- `Run(ctx context.Context) error` — Run the MCP server
- `Shutdown(ctx context.Context) error` — Graceful shutdown
- `SendNotificationToAllClients(ctx context.Context, level, code string, data any)` — Broadcast notification
- `ChannelSend(ctx context.Context, name string, data any)` — Send channel event
- `ChannelSendToSession(ctx context.Context, session string, name string, data any)` — Send to specific session

### Options

Configuration for creating an MCP service.

```go
type Options struct {
    WorkspaceRoot  string           // Restrict file ops to this directory (empty = cwd)
    Unrestricted   bool             // Disable sandboxing entirely (not recommended)
    ProcessService *process.Service // Optional process management
    WSHub          *ws.Hub          // Optional WebSocket hub for real-time streaming
    Subsystems     []Subsystem      // Additional tool groups registered at startup
}
```

### Subsystem Interface

Interface for adding modular tool groups to the MCP server.

```go
type Subsystem interface {
    Name() string
    RegisterTools(server *mcp.Server)
}

// SubsystemWithNotifier extends Subsystem for channel event support
type SubsystemWithNotifier interface {
    Subsystem
    Notify() chan any
}
```

### Tool Record

Internal registration of tools for REST bridge and management.

```go
type ToolRecord struct {
    Name        string
    Description string
    Group       string
    Handler     any
}
```

---

## Tool groups

The MCP server provides several built-in tool groups:

### File Operations

Tools for file system access (sandboxed to workspace root).

| Tool | Description | Parameters |
|------|-------------|------------|
| `file_read` | Read file contents | path |
| `file_write` | Write file contents | path, content |
| `file_delete` | Delete a file | path |
| `file_rename` | Rename/move a file | from, to |
| `file_exists` | Check if file exists | path |
| `dir_list` | List directory contents | path |
| `dir_create` | Create a directory | path |
| `lang_detect` | Detect file language | path |
| `lang_list` | List available languages | - |

### Process Management

Tools for managing external processes.

| Tool | Description | Parameters |
|------|-------------|------------|
| `process_start` | Start a process | command, args, dir, env |
| `process_stop` | Stop a running process | id |
| `process_kill` | Kill a process | id, signal |
| `process_list` | List running processes | - |
| `process_output` | Get process output | id |
| `process_input` | Send input to process | id, input |

### Metrics

Tools for recording and querying metrics.

| Tool | Description | Parameters |
|------|-------------|------------|
| `metrics_record` | Record a metric | name, value, tags |
| `metrics_query` | Query metrics | name, start, end |

### RAG (Retrieval-Augmented Generation)

Tools for RAG operations.

| Tool | Description | Parameters |
|------|-------------|------------|
| `rag_query` | Query RAG index | query, top_k |
| `rag_ingest` | Ingest document | content, metadata |
| `rag_collections` | List collections | - |

### WebView

Tools for WebView integration.

| Tool | Description | Parameters |
|------|-------------|------------|
| `webview_connect` | Connect to WebView | url |
| `webview_navigate` | Navigate WebView | url |
| `webview_screenshot` | Take screenshot | - |
| `webview_click` | Click element | selector |
| `webview_type` | Type text | text |
| `webview_close` | Close WebView | - |

### WebSocket Client

Tools for WebSocket client operations.

| Tool | Description | Parameters |
|------|-------------|------------|
| `ws_start` | Start WebSocket connection | url, headers |
| `ws_info` | Get WebSocket info | id |
| `ws_send` | Send message | id, message |
| `ws_close` | Close connection | id |

---

## Subsystems

### Brain Subsystem

OpenBrain integration for memory and recall operations.

**Tools:**
- `brain_recall` — Recall information from brain
- `brain_remember` — Store information in brain
- `brain_forget` — Remove information from brain
- `brain_search` — Search brain contents

**Configuration:**
```go
brainSubsystem := &mcp.BrainSubsystem{
    Client: brainClient, // Optional custom client
}
```

### Agentic Subsystem

Agent dispatch and management for automated workflows.

**Tools:**
- `agent_dispatch` — Dispatch agent to perform task
- `agent_status` — Get agent status
- `agent_plans` — List agent plans
- `agent_prs` — List agent pull requests
- `agent_scans` — List agent scans
- `agent_reviews` — List agent reviews

**Components:**
- `Dispatch` — Task dispatch and queue management
- `Epic` — Epic-level task tracking
- `Issue` — Issue tracking and management
- `Mirror` — Repository mirroring
- `Plan` — Plan creation and management
- `PR` — Pull request management
- `Prep` — PR preparation helpers
- `Queue` — Task queue management
- `Resume` — Resume tracking
- `ReviewQueue` — Review queue management
- `Scan` — Repository scanning
- `Status` — Status tracking
- `Watch` — Watch functionality

### IDE Subsystem

IDE bridge for Laravel backend integration.

**Tools:**
- `ide_build` — Run build operations
- `ide_chat` — IDE chat integration
- `ide_dashboard` — IDE dashboard operations

**Configuration:**
```go
ideSubsystem := &mcp.IDESubsystem{
    Config: ideConfig,
}
```

---

## Transport configuration

The MCP server supports multiple transports, selected in priority order:

### Priority Order

1. **Streamable HTTP** — If `MCP_HTTP_ADDR` is set
2. **TCP** — If `MCP_ADDR` is set
3. **Stdio** — Default (used by Claude Code / IDEs)
4. **Unix Socket** — If `MCP_UNIX_SOCKET` is set

### Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `MCP_HTTP_ADDR` | HTTP transport address | `localhost:8081` |
| `MCP_AUTH_TOKEN` | HTTP Bearer auth token | `your-secret-token` |
| `MCP_ADDR` | TCP transport address | `tcp://localhost:8080` |
| `MCP_UNIX_SOCKET` | Unix socket path | `/tmp/mcp.sock` |

### Transport Selection

The `Run()` method automatically selects the highest-priority available transport.

```go
// Force specific transport by setting only that environment variable
os.Setenv("MCP_ADDR", "tcp://localhost:8080")
svc.Run(ctx)
```

---

## Notification system

The MCP server provides a rich notification and channel event system.

### Notification Methods

```go
// Broadcast to all connected sessions
svc.SendNotificationToAllClients(ctx, "info", "monitor", data)

// Push a named channel event
svc.ChannelSend(ctx, "agent.complete", map[string]any{"repo": "go-io"})

// Push to a specific session
svc.ChannelSendToSession(ctx, sessionID, "build.failed", data)
```

### Channel Events

The `claude/channel` experimental capability is registered automatically, enabling channel-based communication between components.

### Notification Types

| Level | Code | Description |
|-------|------|-------------|
| info | monitor | Monitoring updates |
| warning | build.failed | Build failure |
| error | process.crashed | Process crash |
| success | agent.complete | Agent task completed |

---

## Progress tracking

Progress notification helpers for long-running operations.

```go
// Create progress tracker
progress := mcp.NewProgress("scanning_repo", "Scanning repository", 100)

// Update progress
progress.Update(ctx, 50, "Scanned 50% of files")

// Complete progress
progress.Complete(ctx, "Scan completed")

// Fail progress
progress.Fail(ctx, "Scan failed", err)
```

---

## Compliance & security

### Workspace Sandboxing

By default, all file operations are restricted to the `WorkspaceRoot` directory:

```go
// Restricted to specific directory
svc, _ := mcp.New(mcp.Options{
    WorkspaceRoot: "/path/to/project",
})

// Unrestricted (NOT RECOMMENDED)
svc, _ := mcp.New(mcp.Options{
    Unrestricted: true,
})
```

### Authentication

HTTP transport supports Bearer token authentication:

```go
os.Setenv("MCP_AUTH_TOKEN", "your-secret-token")
svc.Run(ctx)
```

### Compliance Helpers

The package includes compliance checking utilities for validating tool inputs and operations.

---

## Adding custom tools

### Step 1: Define Input/Output Structs

```go
type MyToolInput struct {
    Name string `json:"name"`
    Value int `json:"value"`
}

type MyToolOutput struct {
    Result string `json:"result"`
    Count int `json:"count"`
}
```

### Step 2: Write Handler

```go
func (s *Service) myTool(ctx context.Context, req *mcp.CallToolRequest, input MyToolInput) (*mcp.CallToolResult, MyToolOutput, error) {
    // Your implementation here
    result := fmt.Sprintf("Processed %s with value %d", input.Name, input.Value)
    
    return nil, MyToolOutput{
        Result: result,
        Count: 1,
    }, nil
}
```

### Step 3: Register Tool

```go
func (s *Service) registerTools() {
    server := s.server
    
    // Register your tool
    mcp.AddToolRecorded(s, server, "custom", &mcp.Tool{
        Name:        "my_tool",
        Description: "Does something useful",
        InputSchema: &mcp.ToolInputSchema{
            Type: "object",
            Properties: map[string]*mcp.ToolInputSchema{
                "name": {Type: "string"},
                "value": {Type: "integer"},
            },
            Required: []string{"name"},
        },
    }, s.myTool)
}
```

### Step 4: Add to Service Initialization

```go
func New(opts Options) (*Service, error) {
    s := &Service{...}
    
    // Register built-in tools
    s.registerTools()
    
    // Register custom tools
    s.registerCustomTools()
    
    // Register subsystems
    for _, sub := range opts.Subsystems {
        sub.RegisterTools(s.server)
    }
    
    return s, nil
}
```

---

## Adding a new subsystem

### Step 1: Implement Subsystem Interface

```go
type MySubsystem struct {
    name string
}

func (m *MySubsystem) Name() string {
    return m.name
}

func (m *MySubsystem) RegisterTools(server *mcp.Server) {
    // Register your subsystem's tools
    mcp.AddTool(server, "my_sub", &mcp.Tool{
        Name:        "my_sub_tool",
        Description: "A tool from my subsystem",
    }, m.myToolHandler)
}
```

### Step 2: Add Channel Event Support (Optional)

```go
func (m *MySubsystem) Notify() chan any {
    ch := make(chan any, 100)
    go m.handleEvents(ch)
    return ch
}

func (m *MySubsystem) handleEvents(ch chan any) {
    for event := range ch {
        // Handle channel events
        fmt.Printf("Received event: %v\n", event)
    }
}
```

### Step 3: Register Subsystem

```go
svc, err := mcp.New(mcp.Options{
    Subsystems: []mcp.Subsystem{
        &MySubsystem{name: "my-sub"},
    },
})
```

---

## Command-line usage

### Build and Run

```bash
# Build the MCP server
cd ~/Code/core/mcp/go
GOWORK=off go build ./cmd/core-mcp/

# Run with stdio (for Claude Code)
./core-mcp

# Run with TCP
MCP_ADDR=tcp://localhost:8080 ./core-mcp

# Run with HTTP
MCP_HTTP_ADDR=localhost:8081 MCP_AUTH_TOKEN=secret ./core-mcp
```

### Using Core CLI

```bash
# Run tests
cd ~/Code/core/mcp/go
core go test

# Run QA (fmt + vet + lint + test)
core go qa
```

---

## Testing

The package follows the AX standard with comprehensive test coverage.

### Running Tests

```bash
cd ~/Code/core/mcp/go

# All tests
go test ./pkg/mcp/...

# With coverage
go test -cover ./pkg/mcp/...

# With race detector
go test -race ./pkg/mcp/...

# Isolated module verification
GOWORK=off go test ./...

# Specific subsystems
go test ./pkg/mcp/agentic/...
go test ./pkg/mcp/brain/...
go test ./pkg/mcp/ide/...
```

### Test Naming Convention

The package uses a specific naming convention for tests:
- `_Good` — Happy path tests
- `_Bad` — Expected error cases
- `_Ugly` — Panics and edge cases

Example:
```go
func TestMyTool_Good(t *testing.T) { ... }
func TestMyTool_Bad(t *testing.T) { ... }
func TestMyTool_Ugly(t *testing.T) { ... }
```

---

## Integration with other packages

### Process Service Integration

```go
import (
    core "dappco.re/go"
    "dappco.re/go/mcp"
    "dappco.re/go/process"
)

app, _ := core.New(
    core.WithService(process.NewService(process.Options{})),
)
procSvc := core.MustServiceFor[*process.Service](app, "process")

svc, _ := mcp.New(mcp.Options{
    ProcessService: procSvc,
})
```

### WebSocket Hub Integration

```go
import (
    "dappco.re/go/mcp"
    "dappco.re/go/ws"
)

wsHub := ws.NewHub()
go wsHub.Run(context.Background())

svc, _ := mcp.New(mcp.Options{
    WSHub: wsHub,
})
```

---

## Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/mcp` |
| **Repository** | `core/mcp` |
| **Language** | Go 1.26+ |
| **PHP Package** | `forge.lthn.ai/core/mcp` |
| **Dependencies** | `dappco.re/go`, `dappco.re/go/process`, `dappco.re/go/ws`, `github.com/modelcontextprotocol/go-sdk/mcp` |
| **Test Triplets** | ✅ Complete (Good/Bad/Ugly pattern) |
| **Documentation** | ✅ Complete |
| **Transports** | Stdio, TCP, HTTP, Unix Socket |
| **Tool Groups** | 6 built-in + extensible |
| **Subsystems** | 3 built-in (brain, agentic, ide) + extensible |

---

## Related packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-process](../process/) | Process management service | ../process/ |
| [go-ws](../ws/) | WebSocket hub for real-time events | ../ws/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |
| [go-api](../../pkg/api/) | REST API framework (different package) | ../../pkg/api/ |
| [go-agent](../agent/) | Agent dispatch framework | ../agent/ |

---

## Tags

```yaml
- mcp
- model-context-protocol
- server
- ai-agent
- tools
- subsystems
- transport
- websocket
- stdio
- tcp
- http
- file-operations
- process-management
- metrics
- rag
- webview
- notifications
- channel-events
- sandboxing
- authentication
- bearer-token
- claude-code
- ide-integration
- openbrain
- agent-dispatch
- production-ready
- high-coverage
```

---

## References

1. **Repository** — [~/Code/core/mcp/](file:///Users/snider/Code/core/mcp/)
2. **CLAUDE.md** — [~/Code/core/mcp/CLAUDE.md](file:///Users/snider/Code/core/mcp/CLAUDE.md)
3. **MCP Specification** — [modelcontextprotocol.io](https://modelcontextprotocol.io)
4. **Go SDK** — [github.com/modelcontextprotocol/go-sdk](https://github.com/modelcontextprotocol/go-sdk)
5. **CoreGO Framework** — [CoreGO INDEX](../../INDEX.md)

---

*Package documentation generated: 2026-06-17T19:00:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
