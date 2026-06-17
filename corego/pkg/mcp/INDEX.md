---
type: Package Index
package: mcp
module: dappco.re/go/mcp
repo: core/mcp
title: go-mcp Package Index
description: Model Context Protocol server framework for CoreGO applications
tags:
  - mcp
  - model-context-protocol
  - server
  - ai-agent
  - claude-code
---

# go-mcp Package Index

> **Model Context Protocol Server Framework for CoreGO**

**Repository:** `core/mcp`  
**Module:** `dappco.re/go/mcp`  
**PHP Package:** `forge.lthn.ai/core/mcp`  
**Status:** ✅ Production-Ready  
**License:** EUPL-1.2  
**Test Pattern:** Good/Bad/Ugly  
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| CLAUDE.md | Development guidance for Claude Code | [~/Code/core/mcp/CLAUDE.md](file:///Users/snider/Code/core/mcp/CLAUDE.md) |
| Repository README | Original package README | [~/Code/core/mcp/README.md](file:///Users/snider/Code/core/mcp/README.md) |

---

## 🎯 Package Overview

`go-mcp` is the **Model Context Protocol (MCP) server framework** for CoreGO applications, providing a lightweight implementation that enables AI agents (Claude Code, Cursor, etc.) and IDEs to interact with codebases through a standardized protocol. It implements the MCP specification with comprehensive file operations, tool registration, notification broadcasting, and channel-based event streaming.

### Core Capabilities

1. **MCP Server** — Full Model Context Protocol implementation
2. **Multi-Transport** — Stdio (default), TCP, HTTP/REST, Unix Socket
3. **File Operations** — Sandboxed file system access with workspace restrictions
4. **Tool Registration** — Extensible tool system with custom tool support
5. **Subsystem Architecture** — Modular tool groups (brain, agentic, ide)
6. **Notification System** — Broadcast notifications to connected sessions
7. **Channel Events** — Pub/sub event system for cross-component communication
8. **Process Integration** — Optional go-process integration for process management
9. **WebSocket Integration** — Optional go-ws integration for real-time events
10. **Progress Tracking** — Progress notifications for long-running operations

### Repository Layout

```
core/mcp/
├── go/      — Go module (dappco.re/go/mcp)
├── php/     — PHP Laravel package
├── docs/    — Shared documentation
└── composer.json
```

---

## 🏗️ Architecture

### Component Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    MCP Server Layer                          │
│  Service — Main MCP server with transport management       │
│  Server — MCP protocol server (go-sdk)                     │
│  Implementation — Server capabilities and metadata           │
├─────────────────────────────────────────────────────────────┤
│                    Transport Layer                           │
│  Stdio — Standard I/O (default for Claude Code)              │
│  TCP — TCP socket transport                                  │
│  HTTP — REST API transport with Bearer auth                 │
│  Unix — Unix domain socket transport                        │
├─────────────────────────────────────────────────────────────┤
│                    Tool Layer                               │
│  File Operations — read, write, delete, list, etc.          │
│  Process Tools — start, stop, kill, list processes            │
│  Metrics Tools — record, query metrics                       │
│  RAG Tools — query, ingest, collections                      │
│  WebView Tools — connect, navigate, screenshot, etc.          │
│  WebSocket Tools — start, send, close connections            │
├─────────────────────────────────────────────────────────────┤
│                    Subsystem Layer                           │
│  Brain — OpenBrain integration (recall, remember, forget)    │
│  Agentic — Agent dispatch and workflow management           │
│  IDE — IDE bridge to Laravel backend                        │
├─────────────────────────────────────────────────────────────┤
│                    Notification Layer                        │
│  Notifications — Broadcast to all sessions                   │
│  Channels — Pub/sub event system                            │
│  Progress — Progress tracking for operations                 │
├─────────────────────────────────────────────────────────────┤
│                    Utility Layer                             │
│  Registry — Tool and resource registry                      │
│  IPC — Inter-process communication                         │
│  Compliance — Compliance checking utilities                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 Package Structure

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
│       ├── Core Files
│       │   ├── mcp.go                 # Main Service type + New()
│       │   ├── mcp_test.go             # Service tests
│       │   ├── mcp_example_test.go     # Service usage examples
│       │   ├── server.go              # MCP server wrapper
│       │   ├── server_test.go          # Server tests
│       │   ├── subsystem.go           # Subsystem interface
│       │   ├── subsystem_test.go       # Subsystem tests
│       │   └── subsystem_example_test.go # Subsystem examples
│       │
│       │   ├── register.go            # Tool/resource registration
│       │   ├── register_test.go        # Registration tests
│       │   ├── register_example_test.go # Registration examples
│       │   ├── ipc.go                 # Inter-process communication
│       │   ├── ipc_test.go             # IPC tests
│       │   ├── notify.go              # Notification broadcasting
│       │   ├── notify_test.go          # Notification tests
│       │   ├── notify_example_test.go # Notification examples
│       │   ├── progress.go            # Progress notification helpers
│       │   ├── progress_test.go        # Progress tests
│       │   ├── progress_example_test.go # Progress examples
│       │   ├── channel.go             # Channel event handling
│       │   ├── result_helpers.go      # Result handling utilities
│       │   ├── compliance_helpers.go  # Compliance checking
│       │   └── string_constants.go    # String constants
│       │
│       ├── Tools
│       │   ├── tools_files.go         # File operation tools
│       │   ├── tools_files_test.go     # File tools tests
│       │   ├── tools_process.go       # Process management tools
│       │   ├── tools_process_test.go   # Process tools tests
│       │   ├── tools_metrics.go       # Metrics tools
│       │   ├── tools_metrics_test.go   # Metrics tools tests
│       │   ├── tools_rag.go           # RAG tools
│       │   ├── tools_rag_test.go       # RAG tools tests
│       │   ├── tools_webview.go       # WebView tools
│       │   ├── tools_webview_test.go   # WebView tools tests
│       │   ├── tools_ws.go             # WebSocket tools
│       │   └── tools_ws_test.go       # WebSocket tools tests
│       │
│       ├── Transports
│       │   ├── transport_stdio.go      # Stdio transport
│       │   ├── transport_stdio_test.go  # Stdio transport tests
│       │   ├── transport_stdio_example_test.go
│       │   ├── transport_tcp.go        # TCP transport
│       │   ├── transport_tcp_test.go    # TCP transport tests
│       │   ├── transport_tcp_example_test.go
│       │   ├── transport_http.go       # HTTP transport
│       │   ├── transport_http_test.go   # HTTP transport tests
│       │   ├── transport_http_example_test.go
│       │   ├── transport_unix.go       # Unix socket transport
│       │   ├── transport_unix_test.go   # Unix socket tests
│       │   └── transport_unix_example_test.go
│       │
│       ├── Transformers
│       │   ├── transformer.go          # Request/response transformers
│       │   ├── transformer_test.go      # Transformer tests
│       │   ├── transformer_example_test.go
│       │   ├── transformer_anthropic.go # Anthropic-specific transformer
│       │   ├── transformer_anthropic_test.go
│       │   ├── transformer_anthropic_example_test.go
│       │   ├── transformer_openai.go   # OpenAI-specific transformer
│       │   ├── transformer_openai_test.go
│       │   ├── transformer_openai_example_test.go
│       │   ├── transformer_honeypot.go # Honeypot transformer
│       │   └── transformer_honeypot_test.go
│       │
│       ├── Upstream (Load Balancing)
│       │   ├── upstream_balancer.go    # Upstream load balancer
│       │   ├── upstream_balancer_test.go
│       │   ├── upstream_balancer_internal_test.go
│       │   ├── upstream_registry.go    # Upstream registry
│       │   ├── upstream_registry_test.go
│       │   ├── upstream_router.go       # Upstream router
│       │   ├── upstream_router_test.go
│       │   ├── upstream_router_internal_test.go
│       │   ├── upstream_transport.go   # Upstream transport
│       │   └── upstream_transport_test.go
│       │
│       ├── Agentic Subsystem
│       │   └── agentic/
│       │       ├── dispatch.go         # Agent dispatch
│       │       ├── dispatch_test.go     # Dispatch tests
│       │       ├── dispatch_example_test.go
│       │       ├── epic.go             # Epic management
│       │       ├── epic_test.go         # Epic tests
│       │       ├── epic_example_test.go
│       │       ├── issue.go            # Issue tracking
│       │       ├── issue_test.go        # Issue tests
│       │       ├── issue_example_test.go
│       │       ├── mirror.go           # Repository mirroring
│       │       ├── mirror_test.go       # Mirror tests
│       │       ├── mirror_example_test.go
│       │       ├── plan.go             # Plan management
│       │       ├── plan_test.go         # Plan tests
│       │       ├── plan_example_test.go
│       │       ├── prep.go             # PR preparation
│       │       ├── prep_test.go         # PR tests
│       │       ├── prep_example_test.go
│       │       ├── prep_template_test.go
│       │       ├── pr.go               # Pull request management
│       │       ├── pr_test.go           # PR tests
│       │       ├── pr_example_test.go
│       │       ├── queue.go            # Queue management
│       │       ├── queue_test.go        # Queue tests
│       │       ├── queue_example_test.go
│       │       ├── queue_status_test.go
│       │       ├── resume.go           # Resume tracking
│       │       ├── resume_test.go       # Resume tests
│       │       ├── resume_example_test.go
│       │       ├── review_queue.go      # Review queue
│       │       ├── review_queue_test.go
│       │       ├── review_queue_example_test.go
│       │       ├── scan.go             # Repository scanning
│       │       ├── scan_test.go         # Scan tests
│       │       ├── scan_example_test.go
│       │       ├── status.go           # Status tracking
│       │       ├── status_test.go       # Status tests
│       │       ├── status_example_test.go
│       │       ├── watch.go            # Watch functionality
│       │       ├── watch_test.go        # Watch tests
│       │       ├── watch_example_test.go
│       │       ├── write_atomic.go     # Atomic write
│       │       ├── write_atomic_test.go
│       │       ├── helpers_test.go      # Helper tests
│       │       └── repo_helpers.go     # Repository helpers
│       │
│       ├── Brain Subsystem
│       │   └── brain/
│       │       ├── brain.go             # Brain subsystem
│       │       ├── brain_test.go         # Brain tests
│       │       ├── brain_example_test.go
│       │       ├── bridge_events.go     # Bridge events
│       │       ├── bridge_events_test.go
│       │       ├── direct.go           # Direct operations
│       │       ├── direct_test.go       # Direct tests
│       │       ├── direct_example_test.go
│       │       ├── provider.go         # Brain provider
│       │       ├── provider_test.go     # Provider tests
│       │       ├── provider_example_test.go
│       │       ├── tools.go            # Brain tools
│       │       └── tools_test.go        # Brain tools tests
│       │       └── client/
│       │           ├── client.go         # Brain client
│       │           ├── client_test.go     # Client tests
│       │           ├── client_example_test.go
│       │           └── coreio_compat.go  # CoreIO compatibility
│       │
│       └── IDE Subsystem
│           └── ide/
│               ├── bridge.go           # IDE bridge
│               ├── bridge_test.go       # Bridge tests
│               ├── bridge_example_test.go
│               ├── config.go            # IDE configuration
│               ├── config_test.go        # Config tests
│               ├── config_example_test.go
│               ├── ide.go               # IDE subsystem
│               ├── ide_test.go           # IDE tests
│               ├── ide_example_test.go
│               ├── ide_data.go          # IDE data types
│               ├── ide_data_test.go      # IDE data tests
│               ├── tools_build.go       # Build tools
│               ├── tools_build_test.go  # Build tools tests
│               ├── tools_build_example_test.go
│               ├── tools_chat.go        # Chat tools
│               └── tools_chat_test.go   # Chat tools tests
│               └── tools_dashboard.go   # Dashboard tools (no test file)
│
├── go.mod
├── go.sum
├── go.work
├── AGENTS.md
├── CLAUDE.md
├── README.md
└── docs/
```

---

## 🚀 Quick Start

### Basic MCP Server

```go
svc, err := mcp.New(mcp.Options{
    WorkspaceRoot: ".",
})
if err != nil {
    panic(err)
}

ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
defer stop()

svc.Run(ctx)
```

### With TCP Transport

```go
os.Setenv("MCP_ADDR", "tcp://localhost:8080")
svc, _ := mcp.New(mcp.Options{WorkspaceRoot: "."})
svc.Run(context.Background())
```

### With HTTP Transport

```go
os.Setenv("MCP_HTTP_ADDR", "localhost:8081")
os.Setenv("MCP_AUTH_TOKEN", "your-secret-token")
svc, _ := mcp.New(mcp.Options{WorkspaceRoot: "."})
svc.Run(context.Background())
```

### With All Integrations

```go
app, _ := core.New(
    core.WithService(process.NewService(process.Options{})),
)
procSvc := core.MustServiceFor[*process.Service](app, "process")
wsHub := ws.NewHub()
go wsHub.Run(context.Background())

svc, _ := mcp.New(mcp.Options{
    WorkspaceRoot:  "/path/to/project",
    ProcessService: procSvc,
    WSHub:          wsHub,
    Subsystems:     []mcp.Subsystem{&mcp.BrainSubsystem{}, &mcp.AgenticSubsystem{}},
})
svc.Run(context.Background())
```

---

## 🔧 Core Components

### Service

Main MCP service type with transport management, tool registration, and lifecycle handling.

**File:** `mcp.go`

```go
type Service struct {
    *core.ServiceRuntime[struct{}]
    server         *mcp.Server
    workspaceRoot  string
    medium         *coreMedium
    subsystems     []Subsystem
    logger         *core.Log
    processService *process.Service
    wsHub          *ws.Hub
    wsServer       *http.Server
    wsAddr         string
    processMu      sync.Mutex
    processMeta    map[string]processRuntime
    tools          []ToolRecord
}
```

**Key Methods:**
- `New(opts Options) (*Service, error)`
- `Run(ctx context.Context) error`
- `Shutdown(ctx context.Context) error`
- `SendNotificationToAllClients(ctx, level, code string, data any)`
- `ChannelSend(ctx, name string, data any)`
- `ChannelSendToSession(ctx, session, name string, data any)`

### Options

```go
type Options struct {
    WorkspaceRoot  string           // Workspace directory (default: cwd)
    Unrestricted   bool             // Disable sandboxing (NOT RECOMMENDED)
    ProcessService *process.Service // Optional process management
    WSHub          *ws.Hub          // Optional WebSocket hub
    Subsystems     []Subsystem      // Additional subsystems
}
```

### Subsystem Interface

```go
type Subsystem interface {
    Name() string
    RegisterTools(server *mcp.Server)
}

type SubsystemWithNotifier interface {
    Subsystem
    Notify() chan any
}
```

---

## 📡 Built-in Tool Groups

### File Operations (tools_files.go)

| Tool | Description | Parameters |
|------|-------------|------------|
| `file_read` | Read file contents | path |
| `file_write` | Write file contents | path, content |
| `file_delete` | Delete a file | path |
| `file_rename` | Rename/move file | from, to |
| `file_exists` | Check file existence | path |
| `dir_list` | List directory | path |
| `dir_create` | Create directory | path |
| `lang_detect` | Detect language | path |
| `lang_list` | List languages | - |

### Process Management (tools_process.go)

| Tool | Description | Parameters |
|------|-------------|------------|
| `process_start` | Start process | command, args, dir, env |
| `process_stop` | Stop process | id |
| `process_kill` | Kill process | id, signal |
| `process_list` | List processes | - |
| `process_output` | Get output | id |
| `process_input` | Send input | id, input |

### Metrics (tools_metrics.go)

| Tool | Description | Parameters |
|------|-------------|------------|
| `metrics_record` | Record metric | name, value, tags |
| `metrics_query` | Query metrics | name, start, end |

### RAG (tools_rag.go)

| Tool | Description | Parameters |
|------|-------------|------------|
| `rag_query` | Query RAG | query, top_k |
| `rag_ingest` | Ingest document | content, metadata |
| `rag_collections` | List collections | - |

### WebView (tools_webview.go)

| Tool | Description | Parameters |
|------|-------------|------------|
| `webview_connect` | Connect | url |
| `webview_navigate` | Navigate | url |
| `webview_screenshot` | Screenshot | - |
| `webview_click` | Click element | selector |
| `webview_type` | Type text | text |
| `webview_close` | Close | - |

### WebSocket Client (tools_ws.go)

| Tool | Description | Parameters |
|------|-------------|------------|
| `ws_start` | Start connection | url, headers |
| `ws_info` | Get info | id |
| `ws_send` | Send message | id, message |
| `ws_close` | Close connection | id |

---

## 🧩 Subsystems

### Brain Subsystem (brain/)

OpenBrain integration for memory operations.

**Files:**
- `brain.go` — Main subsystem
- `tools.go` — Brain tools
- `provider.go` — Brain provider
- `direct.go` — Direct operations
- `bridge_events.go` — Bridge events
- `client/client.go` — Brain client
- `client/coreio_compat.go` — CoreIO compatibility

**Tools:**
- `brain_recall` — Recall from brain
- `brain_remember` — Store in brain
- `brain_forget` — Remove from brain
- `brain_search` — Search brain

### Agentic Subsystem (agentic/)

Agent dispatch and workflow management.

**Files:**
- `dispatch.go` — Task dispatch
- `epic.go` — Epic management
- `issue.go` — Issue tracking
- `mirror.go` — Repository mirroring
- `plan.go` — Plan management
- `prep.go` — PR preparation
- `pr.go` — Pull request management
- `queue.go` — Queue management
- `resume.go` — Resume tracking
- `review_queue.go` — Review queue
- `scan.go` — Repository scanning
- `status.go` — Status tracking
- `watch.go` — Watch functionality
- `write_atomic.go` — Atomic writes
- `repo_helpers.go` — Repository helpers

**Tools:**
- `agent_dispatch` — Dispatch agent
- `agent_status` — Agent status
- `agent_plans` — List plans
- `agent_prs` — List PRs
- `agent_scans` — List scans
- `agent_reviews` — List reviews

### IDE Subsystem (ide/)

IDE bridge for Laravel backend.

**Files:**
- `bridge.go` — IDE bridge
- `config.go` — Configuration
- `ide.go` — IDE subsystem
- `ide_data.go` — IDE data types
- `tools_build.go` — Build tools
- `tools_chat.go` — Chat tools
- `tools_dashboard.go` — Dashboard tools

**Tools:**
- `ide_build` — Build operations
- `ide_chat` — Chat integration
- `ide_dashboard` — Dashboard operations

---

## 🚀 Transport Layer

The MCP server supports multiple transports with automatic priority-based selection.

### Transport Priority

1. **Streamable HTTP** — If `MCP_HTTP_ADDR` is set
2. **TCP** — If `MCP_ADDR` is set
3. **Stdio** — Default
4. **Unix Socket** — If `MCP_UNIX_SOCKET` is set

### Transport Files

| File | Transport | Description |
|------|-----------|-------------|
| `transport_stdio.go` | Stdio | Standard I/O (default for Claude Code) |
| `transport_tcp.go` | TCP | TCP socket transport |
| `transport_http.go` | HTTP | REST API with Bearer auth |
| `transport_unix.go` | Unix | Unix domain socket |

### Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `MCP_HTTP_ADDR` | HTTP transport address | `localhost:8081` |
| `MCP_AUTH_TOKEN` | HTTP Bearer token | `your-secret` |
| `MCP_ADDR` | TCP transport address | `tcp://localhost:8080` |
| `MCP_UNIX_SOCKET` | Unix socket path | `/tmp/mcp.sock` |

---

## 🔔 Notification System

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

The `claude/channel` experimental capability is registered automatically.

**Event Types:**
- `monitor` — Monitoring updates
- `build.failed` — Build failures
- `process.crashed` — Process crashes
- `agent.complete` — Agent task completion

### Progress Tracking

```go
progress := mcp.NewProgress("scanning_repo", "Scanning repository", 100)
progress.Update(ctx, 50, "Scanned 50%")
progress.Complete(ctx, "Done")
progress.Fail(ctx, "Failed", err)
```

---

## 🔄 Adding Custom Tools

### Step 1: Define Input/Output

```go
type MyToolInput struct {
    Name string `json:"name"`
    Value int `json:"value"`
}

type MyToolOutput struct {
    Result string `json:"result"`
}
```

### Step 2: Write Handler

```go
func (s *Service) myTool(ctx context.Context, req *mcp.CallToolRequest, input MyToolInput) (*mcp.CallToolResult, MyToolOutput, error) {
    return nil, MyToolOutput{Result: "done"}, nil
}
```

### Step 3: Register Tool

```go
func (s *Service) registerCustomTools() {
    server := s.server
    addToolRecorded(s, server, "custom", &mcp.Tool{
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

---

## 📦 Adding a New Subsystem

### Implement Subsystem Interface

```go
type MySubsystem struct{}

func (m *MySubsystem) Name() string {
    return "my-sub"
}

func (m *MySubsystem) RegisterTools(server *mcp.Server) {
    addTool(server, "my_sub", &mcp.Tool{
        Name: "my_tool",
        Description: "My tool",
    }, m.myToolHandler)
}

// Optional: Add channel event support
func (m *MySubsystem) Notify() chan any {
    ch := make(chan any, 100)
    go m.handleEvents(ch)
    return ch
}
```

### Register Subsystem

```go
svc, _ := mcp.New(mcp.Options{
    Subsystems: []mcp.Subsystem{&MySubsystem{}},
})
```

---

## 🧪 Testing

### Test Structure

The package follows a specific naming convention:
- `_Good` — Happy path tests
- `_Bad` — Expected error cases
- `_Ugly` — Panics and edge cases

### Running Tests

```bash
cd ~/Code/core/mcp/go

# All tests
go test ./pkg/mcp/...

# With coverage
go test -cover ./pkg/mcp/...

# Specific subsystems
go test ./pkg/mcp/agentic/...
go test ./pkg/mcp/brain/...
go test ./pkg/mcp/ide/...

# Isolated module verification
GOWORK=off go test ./...
```

---

## 🌐 Integration

### Process Service Integration

```go
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
wsHub := ws.NewHub()
go wsHub.Run(context.Background())

svc, _ := mcp.New(mcp.Options{
    WSHub: wsHub,
})
```

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| **Total Go files** | ~150+ |
| **Test files** | ~120+ |
| **Example test files** | ~60+ |
| **Subsystems** | 3 built-in + extensible |
| **Tool groups** | 6 built-in + extensible |
| **Transports** | 4 (stdio, tcp, http, unix) |
| **Agentic tools** | 25+ |
| **Brain tools** | 10+ |
| **IDE tools** | 5+ |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-process](../process/) | Process management service | ../process/ |
| [go-ws](../ws/) | WebSocket hub | ../ws/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |
| [go-agent](../agent/) | Agent dispatch framework | ../agent/ |

---

## 📈 Quality Metrics

- ✅ **MCP Specification Compliance** — Full Model Context Protocol implementation
- ✅ **Test Coverage** — Good/Bad/Ugly pattern for all scenarios
- ✅ **Transport Flexibility** — Multiple transport backends
- ✅ **Security** — Workspace sandboxing and Bearer auth
- ✅ **Extensibility** — Subsystem architecture for custom tools
- ✅ **Documentation** — Complete README + INDEX
- ✅ **Core Integration** — Full CoreGO framework support

---

## 📝 Changelog

| Date | Change | Commit |
|------|--------|--------|
| 2026-06-17 | Complete knowledge pack documentation | N/A |
| 2026-05-XX | Brain subsystem finalized | N/A |
| 2026-04-30 | CLAUDE.md guidance created | N/A |

---

## 🎯 Tags

```yaml
- mcp
- model-context-protocol
- server
- ai-agent
- claude-code
- cursor
- ide-integration
- tools
- subsystems
- brain
- agentic
- ide
- transport
- stdio
- tcp
- http
- websocket
- file-operations
- process-management
- metrics
- rag
- webview
- notifications
- channel-events
- sandboxing
- authentication
- production-ready
- high-coverage
- extensible
```

---

## 📚 References

1. **Repository** — [~/Code/core/mcp/](file:///Users/snider/Code/core/mcp/)
2. **CLAUDE.md** — [~/Code/core/mcp/CLAUDE.md](file:///Users/snider/Code/core/mcp/CLAUDE.md)
3. **MCP Specification** — [modelcontextprotocol.io](https://modelcontextprotocol.io)
4. **Go SDK** — [github.com/modelcontextprotocol/go-sdk](https://github.com/modelcontextprotocol/go-sdk)
5. **CoreGO Framework** — [CoreGO INDEX](../../INDEX.md)

---

*Package index generated: 2026-06-17T19:00:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
