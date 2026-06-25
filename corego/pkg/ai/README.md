# go-ai — AI Integration Layer

> **Package:** `dappco.re/go/ai`  
> **Repository:** [`github.com/dappcore/go-ai`](https://github.com/dappcore/go-ai)  
> **Spec:** [`plans/code/core/go/ai/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>

---

## Overview

**go-ai** is the AI host surface in the Core CLI ecosystem, serving as the composition and serving point for AI capabilities. It provides a facade layer, JSONL metrics logging, RAG query wrappers, provider routing, differential model loading, and integrates with the MCP (Model Context Protocol) server.

### Design principle

Library subsystems wrap specialized packages with minimal API surfaces. Heavy model logic remains in:
- `go-ml` — Scoring engine, heuristics, probes
- `go-inference` — Shared ML backend interfaces  
- `go-rag` — Qdrant vector DB + Ollama embeddings

### Architecture

**Module Path:** `dappco.re/go/ai`  
**Repository Structure:**

```
core/go-ai/
├── go/                          # Go module root
│   ├── ai/                      # Library facade package
│   │   ├── ai.go                # Canonical AI facade
│   │   ├── metrics.go           # JSONL event recording
│   │   ├── rag.go               # RAG query facade
│   │   ├── context.go           # RAG context assembly
│   │   ├── provider_router.go   # Provider routing
│   │   └── differential_loader.go # Base/fine-tune pair staging
│   │
│   ├── mcp/                     # MCP tool server
│   │   ├── service.go           # MCP service host
│   │   ├── jsonrpc.go           # JSON-RPC transport
│   │   ├── tools_core.go        # Core tools (20+)
│   │   ├── tools_external.go    # External tools (30+)
│   │   ├── transport_stdio.go   # stdio transport
│   │   ├── transport_tcp.go     # TCP transport
│   │   └── transport_unix.go    # Unix socket transport
│   │
│   ├── pkg/                     # Extended packages (40+)
│   │   ├── welfare/            # Content welfare (detect, guard, mediate, feedback)
│   │   ├── sessionkv/           # session.kv (State) memory
│   │   ├── chathistory/         # Conversation history
│   │   ├── driver/              # Model drivers
│   │   ├── api/                 # HTTP API surface
│   │   ├── batch/               # Batch processing
│   │   ├── budget/              # Budget management
│   │   ├── chat/                # Chat processing
│   │   ├── creds/               # Credential management
│   │   ├── embed/               # Embedding utilities
│   │   ├── fusion/              # Model fusion
│   │   ├── kvtier/              # KV tier storage
│   │   ├── lora/                # LoRA adapters
│   │   ├── modality/            # Multi-modal support
│   │   ├── ngram/               # N-gram utilities
│   │   ├── obs/                 # Observability
│   │   ├── parse/               # Parsing utilities
│   │   ├── pipeline/            # Processing pipelines
│   │   ├── prompt/              # Prompt engineering
│   │   ├── radix/               # Radix utilities
│   │   ├── residency/           # Model residency
│   │   ├── respcache/           # Response caching
│   │   ├── retry/               # Retry logic
│   │   ├── safety/              # Safety checks
│   │   ├── schedule/            # Scheduling
│   │   ├── session/             # Session management
│   │   ├── sessionkv/           # Session KV storage
│   │   ├── specctl/             # Spec control
│   │   ├── stream/              # Streaming
│   │   ├── structured/          # Structured outputs
│   │   ├── tools/              # Tool utilities
│   │   ├── transform/           # Transformations
│   │   └── usage/               # Usage tracking
│   │
│   ├── providers/              # Provider implementations
│   │   └── openai/             # OpenAI-compatible provider
│   │
│   ├── cmd/                     # CLI commands (11 commands)
│   │   ├── ai/                  # Unified AI commands
│   │   ├── metrics/            # Metrics viewer
│   │   ├── rag/                # RAG commands (re-export from go-rag)
│   │   ├── security/           # GitHub security scanning (5 subcommands)
│   │   ├── lab/                # Homelab monitoring dashboard
│   │   ├── embed-bench/        # Embedding model benchmarks
│   │   ├── daemon/             # Background daemon
│   │   ├── lthn-ai/            # Lethean AI host (MCP + embeddings + vector + State)
│   │   ├── lem-runtime/        # Contained model service
│   │   └── lem-desktop/        # Wails desktop app
│   │
│   ├── demos/                  # Demonstration code
│   ├── docs/                    # Documentation
│   ├── external/               # External resources
│   ├── ui/                      # UI assets
│   ├── README.md
│   ├── AGENTS.md
│   ├── CLAUDE.md
│   ├── GOAL.md
│   └── LICENCE
│
├── swift/                     # Swift bindings
├── .core/                     # Core configuration
└── .forgejo/                  # Forgejo CI configuration
```

---

## Package deep dive

### ai/ — Library Facade Package

The core library package providing the AI facade surface.

#### ai.go — Canonical AI facade

```go
// Package ai provides the canonical AI facade for the core CLI.
package ai

// Usage:
//   contextText, err := ai.QueryRAGForTask(ai.TaskInfo{
//       Title:       "Investigate build failure",
//       Description: "CI compile step fails",
//   })
//   if err != nil { return err }
//
//   if err := ai.Record(ai.Event{
//       Type: "security.scan",
//       Repo: "wailsapp/wails"
//   }); err != nil { return err }
```

This file serves as the package documentation entry point with usage examples.

#### metrics.go — JSONL event logging

**Purpose:** Append-only JSONL event storage for agent activity and security scan results.

**Storage Location:** `~/.core/ai/metrics/YYYY-MM-DD.jsonl`

**Thread Safety:** Uses `sync.Mutex` via `core.New().Lock("ai.metrics.write")`

**Key Constants:**
```go
const (
    recentEventLimit       = 10
    maxMetricsReadWindowDays = 365
    maxMetricsLineBytes    = 1 << 20  // 1MB
    metricsFileMode        = 0o600
    metricsDirMode          = 0o700
)
```

**Event Type:**
```go
type Event struct {
    Type      string         `json:"type"`              // Event category
    Timestamp time.Time      `json:"timestamp"`         // When the event occurred
    AgentID   string         `json:"agent_id,omitempty"`  // Which agent produced this
    Repo      string         `json:"repo,omitempty"`    // Target repository
    Duration  time.Duration  `json:"duration,omitempty"` // Operation duration
    Data      map[string]any `json:"data,omitempty"`     // Arbitrary payload
}
```

**Functions:**

```go
// Record appends one Event as JSON to today's JSONL file
// Creates daily file and parent directory if missing
// Uses metricsWriteLock mutex for concurrent write safety
func Record(event Event) core.Result

// ReadEvents iterates daily JSONL files from `since` to today
// Skips missing days silently (not every day has events)
// Parses each line as an Event
// Returns all events in chronological order
func ReadEvents(since time.Time) core.Result

// Summary aggregates events into maps: counts by Type, by Repo, by AgentID
// Also returns a `recent` slice of the last 10 events
func Summary(events []Event) map[string]any
```

**Event Flow:**
1. Determine metrics directory from env vars: `CORE_HOME` → `HOME` → `USERPROFILE`
2. Ensure directory exists via `coreio.Local.EnsureDir()`
3. Open today's JSONL file for append
4. Marshal event to JSON
5. Write JSON + newline
6. Close file

**Example Usage:**
```go
// Record a security scan event
if err := ai.Record(ai.Event{
    Type: "security.scan",
    Repo: "dappcore/go-build",
    Data: map[string]any{
        "findings": 5,
        "severity": "high",
    },
}); err != nil {
    return err
}

// Read events from the last 7 days
eventsResult := ai.ReadEvents(time.Now().Add(-168 * time.Hour))
if !eventsResult.OK {
    return eventsResult.Err
}
events := eventsResult.Value.([]ai.Event)

// Get summary statistics
summary := ai.Summary(events)
```

#### rag.go — RAG query facade

**Purpose:** Thin wrapper around `go-rag` Qdrant vector search with Ollama embeddings.

**Design Note:** Breaks import cycle — `pkg/agentic` imports `ai`, so `ai` cannot import `pkg/agentic` directly.

**Configuration Defaults:**

```go
const (
    ragTaskCollection          = "hostuk-docs"
    ragTaskResultLimit         = 3
    ragTaskSimilarityThreshold = 0.5
    ragTaskQueryRuneLimit      = 500
)
```

**TaskInfo Type:**
```go
// Carries minimal task data needed for RAG queries
type TaskInfo struct {
    Title       string  // Task title for query construction
    Description string  // Task description, truncated to 500 runes
}
```

**QueryRAGForTask Function:**

```go
// Constructs query from Title + ": " + Description (truncated to 500 runes)
// Creates Qdrant client and Ollama embeddings client
// Runs rag.Query against "hostuk-docs" collection
// Returns formatted context string or empty string on failure
func QueryRAGForTask(task TaskInfo) core.Result
```

**Query Process:**
1. Build query text: `task.Title + ": " + task.Description`
2. Truncate to 500 runes
3. Create Qdrant client with default config (localhost:6334)
4. Create Ollama client with default config (localhost:11434)
5. Execute query with config:
   - Collection: `hostuk-docs`
   - Limit: 3 results
   - Threshold: 0.5 similarity
6. Format results as context string
7. Return context or empty string on failure

**Example Usage:**
```go
// Query for build failure documentation
contextResult := ai.QueryRAGForTask(ai.TaskInfo{
    Title:       "Investigate build failure",
    Description: "CI compile step fails with linker error",
})
if !contextResult.OK {
    return contextResult.Err
}
context := contextResult.Value.(string)
// Use context in LLM prompt
```

#### context.go — RAG context assembly

**Purpose:** Adapts package RAG helper to provider context injection.

**RAGContextAssembler:**
```go
type RAGContextAssembler struct {
    // Assembly configuration
}

func (a *RAGContextAssembler) Assemble(ctx context.Context, task TaskInfo) core.Result
```

Ensures retrieved documentation lands in the prompt the provider sees.

#### provider_router.go — Provider routing

**Purpose:** Describes local or external models that can satisfy chat requests via shared inference contract.

**ProviderRoute Type:**
```go
type ProviderRoute struct {
    ID          string      // Unique route identifier
    Name        string      // Human-readable name
    Provider    string      // Provider type (local, external)
    Model       string      // Model identifier
    Endpoint    string      // API endpoint URL
    APIKey      string      // API key (optional)
    Priority    int         // Routing priority
    Capabilities []string   // Supported capabilities
}

func RouteRequest(request RequestInfo) ProviderRoute
func GetRoutes() []ProviderRoute
```

**Routing features:**
- Preferences management
- Fallback chains
- Batching support
- Load balancing

#### differential_loader.go — Differential model loading

**Purpose:** Stages base/fine-tune model pairs before research or agentic workflows.

**DifferentialLoadAction:**
```go
type DifferentialLoadAction struct {
    BaseModel   string  // Base model identifier
    FineTuneID  string  // Fine-tune identifier
    LoadPath    string  // Path to load models
}

func (a *DifferentialLoadAction) Execute(ctx context.Context) core.Result
```

Consumes the base/adapter diff that `go-ml`'s LQL `diff` path produces.

### mcp/ — Model Context Protocol Server

**Purpose:** MCP tool server providing JSON-RPC interface with stdio, TCP, and Unix socket transports.

**Module:** `dappco.re/go/ai/mcp`

#### service.go — MCP service host

**Service Type:**
```go
type Service struct {
    Name        string
    Version     string
    Tools       []Tool
    Transports  []Transport
}

func NewService() *Service
func (s *Service) Run(ctx context.Context) core.Result
```

**Tool Registration:**
```go
func (s *Service) RegisterTool(tool Tool)
func (s *Service) RegisterTools(tools ...Tool)
```

#### jsonrpc.go — JSON-RPC transport

**JSONRPC Handler:**
```go
type JSONRPCHandler struct {
    Service *Service
}

func (h *JSONRPCHandler) HandleRequest(request jsonrpc.Request) jsonrpc.Response
func (h *JSONRPCHandler) HandleStreamRequest(request jsonrpc.StreamRequest) jsonrpc.StreamResponse
```

#### tools_core.go — Core tools (20+)

Core tools provide fundamental MCP capabilities:

| Tool | Description |
|------|-------------|
| `files_read` | Read file contents |
| `files_write` | Write file contents |
| `files_list` | List files in directory |
| `files_delete` | Delete files |
| `files_move` | Move/rename files |
| `dir_create` | Create directory |
| `dir_delete` | Delete directory |
| `dir_list` | List directory contents |
| `search_files` | Search files by pattern |
| `language_detect` | Detect programming language |
| `language_format` | Format code |
| `language_lint` | Lint code |
| `language_test` | Run tests |
| `process_run` | Run system processes |
| `process_list` | List running processes |
| `process_kill` | Kill processes |
| `http_get` | HTTP GET requests |
| `http_post` | HTTP POST requests |
| `log_message` | Log messages |
| `log_error` | Log errors |

#### tools_external.go — External tools (30+)

External tools provide specialized AI capabilities:

| Category | Tools | Count |
|----------|-------|-------|
| **RAG** | Qdrant search, Ollama embeddings, document indexing | 5 |
| **Metrics** | Event recording, summary, analysis | 3 |
| **Process** | Background tasks, monitoring | 4 |
| **WebSocket** | Client connections, messaging | 3 |
| **Browser** | Page loading, navigation, screenshots | 4 |
| **IDE** | Editor integration, project analysis | 4 |
| **Dashboard** | Metrics display, status monitoring | 3 |

**RAG Tools:**
- `rag_query` — Query vector database
- `rag_index` — Index documents
- `rag_update` — Update indexed documents
- `rag_delete` — Delete from index
- `rag_stats` — Get index statistics

**Browser Tools:**
- `browser_page_load` — Load web page
- `browser_page_content` — Get page content
- `browser_navigate` — Navigate to URL
- `browser_screenshot` — Capture screenshot

#### Transport layers

**stdio Transport:**
```go
type StdioTransport struct {
    Stdin  io.Reader
    Stdout io.Writer
}

func (t *StdioTransport) Run(ctx context.Context) core.Result
```

**TCP Transport:**
```go
type TCPTransport struct {
    Address string
}

func (t *TCPTransport) Run(ctx context.Context) core.Result
```

**Unix Socket Transport:**
```go
type UnixTransport struct {
    Path string
}

func (t *UnixTransport) Run(ctx context.Context) core.Result
```

### pkg/ — Extended Packages (40+)

#### pkg/welfare/ — Content welfare

**Purpose:** Per-turn content welfare for served chats.

**Detection:**
```go
type DetectResult struct {
    IsSafe    bool
    Reasons   []string
    Score     float64
}

func Detect(content string) DetectResult
```

**Welfare Stages:**
1. **Detect** — Identify potentially harmful content
2. **Guard** — Block/allow based on detection
3. **Mediate** — Modify/redact problematic content
4. **Feedback** — Provide user feedback on decisions

**Slur Lexicon:** Located at `pkg/welfare/slurs/`

#### pkg/sessionkv/ — Session KV storage

**Purpose:** session.kv (State) memory for AI sessions.

**Session Type:**
```go
type Session struct {
    ID        string
    State     map[string]any
    CreatedAt time.Time
    UpdatedAt time.Time
}

func NewSession() *Session
func (s *Session) Get(key string) (any, bool)
func (s *Session) Set(key string, value any)
func (s *Session) Delete(key string)
func (s *Session) Save() core.Result
func (s *Session) Load(id string) core.Result
```

#### pkg/chathistory/ — Conversation history

**Purpose:** Maintain chat conversation history.

**Message Type:**
```go
type Message struct {
    Role    string    // system, user, assistant
    Content string
    Timestamp time.Time
    Metadata map[string]any
}

func (m *Message) String() string
```

**History Type:**
```go
type History struct {
    SessionID string
    Messages  []Message
}

func (h *History) AddMessage(msg Message)
func (h *History) GetContext(windowSize int) []Message
func (h *History) Save(path string) core.Result
func (h *History) Load(path string) core.Result
```

#### pkg/driver/ — Model drivers

**Purpose:** Unified driver interface for different AI providers.

**Driver Interface:**
```go
type Driver interface {
    Name() string
    Generate(ctx context.Context, request GenerateRequest) core.Result
    Stream(ctx context.Context, request GenerateRequest, callback func(string))
    Chat(ctx context.Context, messages []Message) core.Result
}

// Driver types
type LocalDriver struct {}       // Local models (Ollama, etc.)
type RemoteDriver struct {}     // Remote APIs (OpenAI, etc.)
type BatchDriver struct {}      // Batch processing
```

#### pkg/api/ — HTTP API surface

**Purpose:** REST API for AI services.

**Endpoints:**
```
POST /api/ai/generate      - Generate text
POST /api/ai/chat          - Chat completion
GET  /api/ai/models        - List available models
GET  /api/ai/metrics      - Get AI metrics
POST /api/ai/rag/query     - RAG query
POST /api/ai/session       - Create session
GET  /api/ai/session/{id}  - Get session
```

#### pkg/batch/ — Batch processing

**Purpose:** Batch AI operations.

**BatchProcessor:**
```go
type BatchProcessor struct {
    Concurrency int
    Timeout    time.Duration
}

func (bp *BatchProcessor) Process(ctx context.Context, tasks []Task) BatchResult
```

#### pkg/budget/ — Budget management

**Purpose:** Track and manage AI usage budgets.

**Budget Type:**
```go
type Budget struct {
    TotalTokens  int
    UsedTokens   int
    Remaining    int
    ResetAt      time.Time
}

func (b *Budget) CanUse(tokens int) bool
func (b *Budget) Use(tokens int) error
```

#### pkg/chat/ — Chat processing

**Purpose:** Chat message processing utilities.

**ChatProcessor:**
```go
type ChatProcessor struct {
    Model      string
    Temperature float64
    TopP        float64
}

func (cp *ChatProcessor) Process(ctx context.Context, messages []Message) core.Result
```

#### pkg/creds/ — Credential management

**Purpose:** Secure credential storage and retrieval.

**CredentialStore:**
```go
type CredentialStore struct {
    Path string
}

func (cs *CredentialStore) Save(name string, creds Credential) core.Result
func (cs *CredentialStore) Get(name string) (Credential, core.Result)
func (cs *CredentialStore) Delete(name string) core.Result
```

#### pkg/embed/ — Embedding utilities

**Purpose:** Text embedding operations.

**Embedder Interface:**
```go
type Embedder interface {
    Embed(ctx context.Context, text string) ([]float64, error)
    EmbedBatch(ctx context.Context, texts []string) ([][]float64, error)
}
```

#### pkg/fusion/ — Model fusion

**Purpose:** Combine multiple model outputs.

**Fusion Strategies:**
- Ensemble voting
- Weighted averaging
- Majority voting
- Confidence-based selection

#### pkg/kvtier/ — KV tier storage

**Purpose:** Multi-tier key-value storage.

**Tiers:**
- Memory (fastest, ephemeral)
- Disk (persistent, local)
- Remote (distributed, scalable)

#### pkg/lora/ — LoRA adapters

**Purpose:** Low-rank adaptation support.

**LoRAConfig:**
```go
type LoRAConfig struct {
    BaseModel   string
    AdapterPath string
    Rank        int
    Alpha       float64
}
```

#### pkg/modality/ — Multi-modal support

**Purpose:** Handle different input/output modalities.

**Modalities:**
- Text
- Image
- Audio
- Video
- Structured data

#### pkg/pipeline/ — Processing pipelines

**Purpose:** Define and execute AI processing pipelines.

**Pipeline:**
```go
type Pipeline struct {
    Name     string
    Steps    []Step
    Context  context.Context
}

func (p *Pipeline) Run() core.Result
```

#### pkg/prompt/ — Prompt engineering

**Purpose:** Prompt construction and optimization.

**PromptBuilder:**
```go
type PromptBuilder struct {
    Template   string
    Variables  map[string]string
}

func (pb *PromptBuilder) Build() string
func (pb *PromptBuilder) WithContext(context string) string
```

#### pkg/session/ — Session management

**Purpose:** Manage AI chat sessions.

**SessionManager:**
```go
type SessionManager struct {
    Sessions map[string]*Session
}

func (sm *SessionManager) Create() *Session
func (sm *SessionManager) Get(id string) (*Session, bool)
func (sm *SessionManager) Delete(id string)
```

#### pkg/stream/ — Streaming

**Purpose:** Handle streaming AI responses.

**StreamHandler:**
```go
type StreamHandler struct {
    Callback func(string)
}

func (sh *StreamHandler) Handle(ctx context.Context, stream Stream) error
```

#### pkg/structured/ — Structured outputs

**Purpose:** Generate structured outputs from AI models.

**Output Types:**
- JSON
- YAML
- XML
- CSV
- Markdown tables

#### pkg/tools/ — Tool utilities

**Purpose:** Tool invocation and management.

**ToolManager:**
```go
type ToolManager struct {
    Tools map[string]Tool
}

func (tm *ToolManager) Invoke(name string, args map[string]any) core.Result
```

### providers/ — Provider Implementations

#### providers/openai/ — OpenAI-compatible provider

**Purpose:** OpenAI API compatible provider implementation.

**Client:**
```go
type Client struct {
    APIKey    string
    Endpoint  string
    Model     string
    Timeout   time.Duration
}

func (c *Client) Generate(ctx context.Context, request GenerateRequest) core.Result
func (c *Client) Chat(ctx context.Context, messages []Message) core.Result
```

### cmd/ — CLI Commands (11 commands)

#### cmd/ai/ — Unified AI commands

**Purpose:** Unified entry point for AI operations.

**Registration:** `AddAICommands(root *cli.Command)`

**Subcommands:**
- `ai generate` — Generate text
- `ai chat` — Interactive chat
- `ai complete` — Text completion
- `ai embed` — Generate embeddings

#### cmd/metrics/ — Metrics viewer

**Purpose:** Display recorded AI metrics.

**Registration:** `AddMetricsCommand(parent *cli.Command)`

**Flags:**
```
--since duration    How far back to read events (default: 168h = 7 days)
--json             Output raw JSON instead of summary
```

**Flow:**
1. Calls `ai.ReadEvents(since)`
2. Calls `ai.Summary(events)`
3. Renders table or JSON to stdout

#### cmd/rag/ — RAG commands

**Purpose:** Re-export RAG commands from `go-rag`.

**Registration:** `var AddRAGSubcommands = ragcmd.AddRAGSubcommands`

**Subcommands:**
- `rag query` — Query vector database
- `rag index` — Index documents
- `rag update` — Update documents
- `rag delete` — Delete from index
- `rag stats` — Show index statistics

#### cmd/security/ — GitHub security scanning (5 subcommands)

**Purpose:** GitHub security scanning via `gh` CLI.

**Registration:** `AddSecurityCommand(parent *cli.Command)`

**Subcommands:**

**1. Alerts:**
```bash
core ai security alerts [--repo OWNER/REPO] [--state open|closed|fixed]
```
- Lists security alerts for repositories
- Filters by state

**2. Dependencies:**
```bash
core ai security deps [--repo OWNER/REPO] [--manifest PATH] [--vulnerabilities]
```
- Scans dependencies for vulnerabilities
- Can specify manifest file
- Shows vulnerability details

**3. Secrets:**
```bash
core ai security secrets [--repo OWNER/REPO] [--push] [--all]
```
- Scans for exposed secrets
- Pushes results to GitHub
- Scans all repositories

**4. Scanning:**
```bash
core ai security scan [--repo OWNER/REPO] [--sarif OUTPUT] [--severity low|medium|high|critical]
```
- Runs security scans
- Outputs SARIF format
- Filters by severity

**5. Jobs:**
```bash
core ai security jobs [--repo OWNER/REPO] [--status queued|in_progress|completed]
```
- Lists security scan jobs
- Filters by status

#### cmd/lab/ — Homelab monitoring dashboard

**Purpose:** Homelab monitoring dashboard (build:ignore).

**Registration:** `AddLabCommand(parent *cli.Command)`

**Note:** Tagged with `build:ignore` to exclude from binary builds.

**Features:**
- Real-time metrics display
- Resource monitoring
- Model serving status
- Performance metrics

#### cmd/embed-bench/ — Embedding model benchmarks

**Purpose:** Benchmark embedding models.

**Registration:** Main entry point at `cmd/embed-bench/main.go`

**Features:**
- Compare embedding model performance
- Measure latency and throughput
- Test accuracy on various datasets

#### cmd/daemon/ — Background daemon

**Purpose:** Run AI services as background daemon.

**Registration:** `AddDaemonCommand(parent *cli.Command)`

**Features:**
- Long-running AI service
- Automatic restart on failure
- Logging and monitoring

#### cmd/lthn-ai/ — Lethean AI host

**Purpose:** Lethean AI host with MCP + embeddings + vector + State.

**Features:**
- MCP server hosting
- Embedding service
- Vector database
- Session state management
- Multi-model support

#### cmd/lem-runtime/ — Contained model service

**Purpose:** Contained model service with curated catalogue.

**Features:**
- Pre-packaged models
- Easy deployment
- Model catalogue management
- Version management

**Note:** Includes `lem-runtime` binary (144MB)

#### cmd/lem-desktop/ — Wails desktop app

**Purpose:** Desktop application with Wails framework.

**Features:**
- Dashboard interface
- Control panel
- Workbench
- Multi-platform support

**UI Components:**
- Angular 20 frontend
- Wails v3 backend
- Multiple services
- MCP tool integration

---

## Usage patterns

### Pattern 1: Basic RAG query

```go
package main

import (
    "dappco.re/go/ai"
)

func main() {
    // Query RAG for task context
    contextResult := ai.QueryRAGForTask(ai.TaskInfo{
        Title:       "Build system investigation",
        Description: "Need to understand the CoreGo build orchestration",
    })
    
    if !contextResult.OK {
        return
    }
    
    context := contextResult.Value.(string)
    println("Context:", context)
}
```

### Pattern 2: Metrics recording

```go
package main

import (
    "time"
    "dappco.re/go/ai"
)

func main() {
    // Record a security scan
    if err := ai.Record(ai.Event{
        Type: "security.scan",
        Repo: "dappcore/go-build",
        AgentID: "security-scanner-v1",
        Duration: 15 * time.Second,
        Data: map[string]any{
            "findings": []map[string]any{
                {"severity": "high", "type": "vulnerability", "count": 2},
                {"severity": "medium", "type": "warning", "count": 5},
            },
        },
    }); err != nil {
        return
    }
    
    // Read events from last 24 hours
    eventsResult := ai.ReadEvents(time.Now().Add(-24 * time.Hour))
    if !eventsResult.OK {
        return
    }
    
    events := eventsResult.Value.([]ai.Event)
    summary := ai.Summary(events)
}
```

### Pattern 3: MCP server

```go
package main

import (
    "context"
    "dappco.re/go/ai/mcp"
)

func main() {
    // Create MCP service
    service := mcp.NewService()
    
    // Register custom tools
    service.RegisterTool(mcp.Tool{
        Name: "my_tool",
        Description: "My custom tool",
        Handler: func(ctx context.Context, args map[string]any) core.Result {
            return core.Ok("tool executed")
        },
    })
    
    // Run with stdio transport
    transport := &mcp.StdioTransport{
        Stdin:  os.Stdin,
        Stdout: os.Stdout,
    }
    
    service.Transports = []mcp.Transport{transport}
    
    // Start service
    if err := service.Run(context.Background()); err != nil {
        panic(err)
    }
}
```

### Pattern 4: Full AI pipeline

```go
package main

import (
    "context"
    "dappco.re/go/ai"
    "dappco.re/go/ai/mcp"
)

func main() {
    // Query RAG for context
    contextResult := ai.QueryRAGForTask(ai.TaskInfo{
        Title: "Code analysis",
        Description: "Analyze this Go code for issues",
    })
    
    // Record the operation
    ai.Record(ai.Event{
        Type: "code.analysis",
        Repo: "myorg/myrepo",
        Data: map[string]any{"status": "started"},
    })
    
    // Use MCP for tool execution
    service := mcp.NewService()
    
    // Execute analysis
    // ...
    
    // Record completion
    ai.Record(ai.Event{
        Type: "code.analysis",
        Repo: "myorg/myrepo",
        Data: map[string]any{"status": "completed", "findings": 3},
    })
}
```

---

## Configuration

### Environment variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CORE_HOME` | Core home directory | `~/.core` |
| `HOME` | User home directory | System default |
| `USERPROFILE` | Windows user profile | System default |

### Metrics storage

**Location:** `~/.core/ai/metrics/`

**File Format:**
- Daily files: `YYYY-MM-DD.jsonl`
- Each line: JSON-encoded `Event` struct
- Compression: None (plain text)
- Retention: 365 days max

### Default endpoints

| Service | Endpoint | Port |
|---------|----------|------|
| Qdrant | localhost | 6334 |
| Ollama | localhost | 11434 |

---

## Dependencies

### CoreGo dependencies

```go
import (
    "dappco.re/go"                    // Core framework
    "dappco.re/go/ai"                  // This package
    "dappco.re/go/ai/mcp"             // MCP server
    "dappco.re/go/io"                 // Storage abstractions
    "dappco.re/go/log"                // Logging
    "dappco.re/go/rag"                // RAG utilities
    "dappco.re/go/scm"                // SCM utilities
    "dappco.re/go/i18n"               // Internationalization
)
```

### External dependencies

```
# From go.mod
github.com/urfave/cli/v3           - CLI framework
golang.org/x/oauth2                 - OAuth2 support
```

### Sibling modules

| Module | Purpose |
|--------|---------|
| `forge.lthn.ai/core/mcp` | MCP server, transports, tool registration |
| `forge.lthn.ai/core/go-rag` | Qdrant vector DB + Ollama embeddings |
| `forge.lthn.ai/core/go-ml` | Scoring engine, heuristics, probes |
| `forge.lthn.ai/core/go-inference` | Shared ML backend interfaces |
| `forge.lthn.ai/core/cli` | CLI framework |
| `forge.lthn.ai/core/go-i18n` | Internationalization |

---

## Testing

### Test structure

All packages follow the **AX-7 Triplet Pattern**:

```
ai/
├── metrics.go                # Main implementation
├── metrics_test.go           # Unit tests
├── metrics_example_test.go   # Usage examples
└── stdlib_assert_test.go     # Standard library compliance
```

### Test coverage

**Key Test Files:**
- `ai/metrics_test.go` — Metrics recording and reading
- `ai/rag_test.go` — RAG query functionality
- `mcp/service_test.go` — MCP service testing
- `mcp/jsonrpc_test.go` — JSON-RPC transport
- `mcp/tools_core_test.go` — Core tools
- `mcp/tools_external_test.go` — External tools
- `cmd/metrics/cmd_test.go` — Metrics CLI command
- `cmd/security/*_test.go` — Security commands

### Running tests

```bash
# All tests
cd go
go test ./...

# Specific package
cd go
go test ./ai/...

# MCP package
cd go
go test ./mcp/...

# With coverage
cd go
go test -cover ./...

# Benchmark tests
cd go
go test -bench=. ./mcp/
```

---

## API reference

### Key types

| Type | Package | Description |
|------|---------|-------------|
| `Event` | `ai` | Metrics event structure |
| `TaskInfo` | `ai` | Task information for RAG queries |
| `Service` | `mcp` | MCP service host |
| `Tool` | `mcp` | MCP tool definition |
| `Transport` | `mcp` | Transport interface |
| `Session` | `pkg/sessionkv` | Session state |
| `Message` | `pkg/chathistory` | Chat message |
| `Driver` | `pkg/driver` | Model driver interface |

### Key functions

| Function | Package | Description |
|----------|---------|-------------|
| `Record()` | `ai` | Record metrics event |
| `ReadEvents()` | `ai` | Read events from metrics storage |
| `Summary()` | `ai` | Aggregate events into summary |
| `QueryRAGForTask()` | `ai` | Query RAG for task context |
| `NewService()` | `mcp` | Create MCP service |
| `Run()` | `mcp` | Start MCP service |
| `RegisterTool()` | `mcp` | Register MCP tool |

---

## Related documentation

### RFC documents

| Document | Location | Description |
|----------|----------|-------------|
| Main RFC | [`RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.md) | Complete specification |
| Models RFC | [`RFC.models.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.models.md) | Data model definitions |
| Commands RFC | [`RFC.commands.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.commands.md) | Command specifications |
| Catalog RFC | [`RFC.catalog.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.catalog.md) | Tool catalog |
| Imports RFC | [`RFC.imports.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.imports.md) | Import specifications |
| CLAUDE.md | [`CLAUDE.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/CLAUDE.md) | Claude-specific notes |

### Repository documentation

| Document | Location | Description |
|----------|----------|-------------|
| README.md | [`README.md`](file:///Users/snider/Code/core/go-ai/README.md) | Repository overview |
| AGENTS.md | [`AGENTS.md`](file:///Users/snider/Code/core/go-ai/AGENTS.md) | Agent guidance |
| CLAUDE.md | [`CLAUDE.md`](file:///Users/snider/Code/core/go-ai/CLAUDE.md) | Claude-specific notes |
| GOAL.md | [`GOAL.md`](file:///Users/snider/Code/core/go-ai/GOAL.md) | Goals and objectives |
| CONTRIBUTING.md | [`CONTRIBUTING.md`](file:///Users/snider/Code/core/go-ai/CONTRIBUTING.md) | Contribution guide |
| LICENCE | [`LICENCE`](file:///Users/snider/Code/core/go-ai/LICENCE) | EUPL-1.2 license |

---

## Metadata

```yaml
name: go-ai
module: dappco.re/go/ai
repository: github.com/dappcore/go-ai
spec: file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.md
maintainer: Purberus <purberus@lthn.ai>
license: EUPL-1.2
status: Production
version: v1.0.0
last_updated: 2026-06-17
```

---
