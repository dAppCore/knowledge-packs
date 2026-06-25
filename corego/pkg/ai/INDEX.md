# go-ai Package Index

> **Package:** `dappco.re/go/ai`  
> **Documentation:** [README.md](./README.md)  
> **Repository:** [`github.com/dappcore/go-ai`](https://github.com/dappcore/go-ai)  
> **Spec:** [`plans/code/core/go/ai/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>

---

## Table of contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Packages](#packages)
- [Commands](#commands)
- [Usage patterns](#usage-patterns)
- [Testing](#testing)
- [API reference](#api-reference)
- [Related documentation](#related-documentation)

---

## Overview

**go-ai** is the AI host surface in the Core CLI ecosystem, providing a composition and serving layer for AI capabilities. It serves as the facade between the Core CLI and heavy ML packages (go-ml, go-inference, go-rag).

### Key features

| Feature | Description |
|---------|-------------|
| **AI Facade** | Canonical entry point for AI operations |
| **Metrics Logging** | JSONL event storage with 365-day retention |
| **RAG Integration** | Qdrant + Ollama integration for context retrieval |
| **Provider Routing** | Multi-provider model routing with preferences and fallbacks |
| **Differential Loading** | Base/fine-tune pair staging for workflows |
| **MCP Server** | Model Context Protocol with stdio/TCP/Unix transports |
| **Content Welfare** | Per-turn safety detection and filtering |
| **Session Management** | Persistent session state with KV storage |
| **Chat History** | Conversation history tracking |
| **Security Scanning** | GitHub security integration |

---

## Architecture

### Repository structure

```
core/go-ai/
├── go/                          # Go module root (dappco.re/go/ai)
│   ├── ai/                      # Library facade package (6 files)
│   │   ├── ai.go                # Package docs + facade
│   │   ├── metrics.go           # JSONL event recording (150+ lines)
│   │   ├── rag.go               # RAG query facade (100 lines)
│   │   ├── context.go           # RAG context assembly
│   │   ├── provider_router.go   # Provider routing
│   │   └── differential_loader.go # Differential loading
│   │
│   ├── mcp/                     # MCP tool server (20+ files)
│   │   ├── service.go           # MCP service host (10KB)
│   │   ├── jsonrpc.go           # JSON-RPC transport (8KB)
│   │   ├── tools_core.go        # Core tools - file, dir, language, process, http, log (20+ tools)
│   │   ├── tools_external.go    # External tools - RAG, metrics, process, WebSocket, browser, IDE, dashboard (30+ tools)
│   │   ├── transport_stdio.go   # stdio transport
│   │   ├── transport_tcp.go     # TCP transport
│   │   └── transport_unix.go    # Unix socket transport
│   │
│   ├── pkg/                     # Extended packages (40+ subdirectories)
│   │   ├── welfare/            # Content welfare (4 files)
│   │   ├── sessionkv/           # Session KV storage
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
│   ├── cmd/                     # CLI commands (11 subpackages)
│   │   ├── ai/                  # Unified AI commands
│   │   ├── metrics/            # Metrics viewer command
│   │   ├── rag/                # RAG commands (re-export from go-rag)
│   │   ├── security/           # GitHub security scanning (5 subcommands)
│   │   ├── lab/                # Homelab monitoring dashboard
│   │   ├── embed-bench/        # Embedding model benchmarks
│   │   ├── daemon/             # Background daemon
│   │   ├── lthn-ai/            # Lethean AI host
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

### Module paths

| Module | Path | Purpose |
|--------|------|---------|
| Main | `dappco.re/go/ai` | Library module |
| MCP | `dappco.re/go/ai/mcp` | MCP server |

### Dependencies

**CoreGo Dependencies:**
- `dappco.re/go` — Core framework
- `dappco.re/go/io` — Storage abstractions
- `dappco.re/go/log` — Logging
- `dappco.re/go/rag` — RAG utilities
- `dappco.re/go/scm` — SCM utilities
- `dappco.re/go/i18n` — Internationalization

**External Dependencies:**
- `github.com/urfave/cli/v3` — CLI framework
- `golang.org/x/oauth2` — OAuth2 support

**Sibling Modules (Forgejo):**
- `forge.lthn.ai/core/mcp` — MCP server, transports, tool registration
- `forge.lthn.ai/core/go-rag` — Qdrant vector DB + Ollama embeddings
- `forge.lthn.ai/core/go-ml` — Scoring engine, heuristics, probes
- `forge.lthn.ai/core/go-inference` — Shared ML backend interfaces
- `forge.lthn.ai/core/cli` — CLI framework
- `forge.lthn.ai/core/go-i18n` — Internationalization

---

## Packages

### ai/ — Library Facade Package

The core library package providing the AI facade surface.

#### Files

| File | Size | Purpose | Key Types/Functions |
|------|------|---------|---------------------|
| `ai.go` | 14 lines | Package documentation | Package comment |
| `metrics.go` | 200+ lines | JSONL event recording | `Event`, `Record()`, `ReadEvents()`, `Summary()` |
| `rag.go` | 100 lines | RAG query facade | `TaskInfo`, `QueryRAGForTask()` |
| `context.go` | - | RAG context assembly | `RAGContextAssembler` |
| `provider_router.go` | - | Provider routing | `ProviderRoute`, `RouteRequest()` |
| `differential_loader.go` | - | Differential loading | `DifferentialLoadAction` |

#### Types

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

**TaskInfo Type:**
```go
type TaskInfo struct {
    Title       string  // Task title for query construction
    Description string  // Task description, truncated to 500 runes
}
```

**ProviderRoute Type:**
```go
type ProviderRoute struct {
    ID          string
    Name        string
    Provider    string      // local, external
    Model       string
    Endpoint    string
    APIKey      string
    Priority    int
    Capabilities []string
}
```

**DifferentialLoadAction Type:**
```go
type DifferentialLoadAction struct {
    BaseModel   string
    FineTuneID  string
    LoadPath    string
}
```

#### Constants

```go
const (
    // Metrics
    recentEventLimit          = 10
    maxMetricsReadWindowDays  = 365
    maxMetricsLineBytes       = 1 << 20  // 1MB
    metricsFileMode           = 0o600
    metricsDirMode            = 0o700
    
    // RAG
    ragTaskCollection         = "hostuk-docs"
    ragTaskResultLimit        = 3
    ragTaskSimilarityThreshold = 0.5
    ragTaskQueryRuneLimit     = 500
)
```

#### Functions

| Function | Description |
|----------|-------------|
| `Record(event Event) core.Result` | Append event to today's JSONL file |
| `ReadEvents(since time.Time) core.Result` | Read events from `since` to today |
| `Summary(events []Event) map[string]any` | Aggregate events into summary statistics |
| `QueryRAGForTask(task TaskInfo) core.Result` | Query RAG for task context |
| `RouteRequest(request RequestInfo) ProviderRoute` | Route request to appropriate provider |
| `Execute(ctx context.Context) core.Result` | Execute differential load action |

### mcp/ — Model Context Protocol Server

**Purpose:** MCP tool server providing JSON-RPC interface.

#### Files

| File | Size | Purpose |
|------|------|---------|
| `service.go` | 10KB | MCP service host |
| `jsonrpc.go` | 8KB | JSON-RPC transport |
| `tools_core.go` | - | Core tools (20+) |
| `tools_external.go` | - | External tools (30+) |
| `transport_stdio.go` | - | stdio transport |
| `transport_tcp.go` | - | TCP transport |
| `transport_unix.go` | - | Unix socket transport |

#### Service type

```go
type Service struct {
    Name       string
    Version    string
    Tools      []Tool
    Transports []Transport
}

func NewService() *Service
func (s *Service) Run(ctx context.Context) core.Result
func (s *Service) RegisterTool(tool Tool)
func (s *Service) RegisterTools(tools ...Tool)
```

#### Tool type

```go
type Tool struct {
    Name        string
    Description string
    Handler     func(ctx context.Context, args map[string]any) core.Result
}
```

#### Transport types

```go
// stdio Transport
type StdioTransport struct {
    Stdin  io.Reader
    Stdout io.Writer
}

// TCP Transport
type TCPTransport struct {
    Address string
}

// Unix Transport
type UnixTransport struct {
    Path string
}

// Interface
type Transport interface {
    Run(ctx context.Context) core.Result
}
```

#### Core tools (20+)

**File Operations:**
- `files_read` — Read file contents
- `files_write` — Write file contents
- `files_list` — List files in directory
- `files_delete` — Delete files
- `files_move` — Move/rename files

**Directory Operations:**
- `dir_create` — Create directory
- `dir_delete` — Delete directory
- `dir_list` — List directory contents

**Language Tools:**
- `language_detect` — Detect programming language
- `language_format` — Format code
- `language_lint` — Lint code
- `language_test` — Run tests

**Process Tools:**
- `process_run` — Run system processes
- `process_list` — List running processes
- `process_kill` — Kill processes

**HTTP Tools:**
- `http_get` — HTTP GET requests
- `http_post` — HTTP POST requests

**Logging Tools:**
- `log_message` — Log messages
- `log_error` — Log errors

#### External tools (30+)

**RAG Tools (5):**
- `rag_query` — Query vector database
- `rag_index` — Index documents
- `rag_update` — Update indexed documents
- `rag_delete` — Delete from index
- `rag_stats` — Get index statistics

**Metrics Tools (3):**
- `metrics_record` — Record metrics event
- `metrics_summary` — Get metrics summary
- `metrics_read` — Read metrics events

**Process Tools (4):**
- `process_start` — Start background task
- `process_status` — Check task status
- `process_stop` — Stop background task
- `process_list` — List background tasks

**WebSocket Tools (3):**
- `websocket_connect` — Connect to WebSocket
- `websocket_send` — Send WebSocket message
- `websocket_close` — Close WebSocket connection

**Browser Tools (4):**
- `browser_page_load` — Load web page
- `browser_page_content` — Get page content
- `browser_navigate` — Navigate to URL
- `browser_screenshot` — Capture screenshot

**IDE Tools (4):**
- `ide_project_analyze` — Analyze project structure
- `ide_file_outline` — Get file outline
- `ide_symbol_search` — Search symbols
- `ide_refactor` — Code refactoring

**Dashboard Tools (3):**
- `dashboard_metrics` — Display metrics
- `dashboard_status` — Show service status
- `dashboard_resources` — Show resource usage

### pkg/ — Extended Packages (40+)

#### pkg/welfare/ — Content welfare

**Files:**
- `welfare.go` — Main welfare logic
- `detect.go` — Content detection
- `guard.go` — Content guarding
- `mediate.go` — Content mediation
- `feedback.go` — User feedback
- `slurs/` — Slur lexicon

**Types:**
```go
type DetectResult struct {
    IsSafe  bool
    Reasons []string
    Score   float64
}

func Detect(content string) DetectResult
```

**Stages:**
1. **Detect** — Identify potentially harmful content
2. **Guard** — Block/allow based on detection
3. **Mediate** — Modify/redact problematic content
4. **Feedback** — Provide user feedback on decisions

#### pkg/sessionkv/ — Session KV storage

**Files:**
- `session.go` — Session type and methods
- `kv.go` — KV storage implementation
- `memory.go` — In-memory storage

**Types:**
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

**Files:**
- `history.go` — History type and methods
- `message.go` — Message type
- `storage.go` — Storage implementation

**Types:**
```go
type Message struct {
    Role      string    // system, user, assistant
    Content   string
    Timestamp time.Time
    Metadata  map[string]any
}

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

**Files:**
- `driver.go` — Driver interface
- `local.go` — Local model driver
- `remote.go` — Remote API driver
- `batch.go` — Batch driver

**Driver Interface:**
```go
type Driver interface {
    Name() string
    Generate(ctx context.Context, request GenerateRequest) core.Result
    Stream(ctx context.Context, request GenerateRequest, callback func(string))
    Chat(ctx context.Context, messages []Message) core.Result
}
```

#### pkg/api/ — HTTP API surface

**Files:**
- `api.go` — HTTP handler
- `routes.go` — Route definitions
- `middleware.go` — Middleware

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

**Files:**
- `batch.go` — Batch processor
- `task.go` — Task type

**Types:**
```go
type BatchProcessor struct {
    Concurrency int
    Timeout    time.Duration
}

type Task struct {
    ID      string
    Input   map[string]any
    Output  map[string]any
}

type BatchResult struct {
    Completed int
    Failed    int
    Results   []Task
}

func (bp *BatchProcessor) Process(ctx context.Context, tasks []Task) BatchResult
```

#### pkg/budget/ — Budget management

**Files:**
- `budget.go` — Budget type
- `tracker.go` — Usage tracking

**Types:**
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

**Files:**
- `chat.go` — Chat processor
- `options.go` — Chat options

**Types:**
```go
type ChatProcessor struct {
    Model       string
    Temperature float64
    TopP        float64
}

type ChatOptions struct {
    Model       string
    Temperature float64
    TopP        float64
    MaxTokens   int
}

func (cp *ChatProcessor) Process(ctx context.Context, messages []Message) core.Result
```

#### pkg/creds/ — Credential management

**Files:**
- `creds.go` — Credential store
- `storage.go` — Storage backend

**Types:**
```go
type CredentialStore struct {
    Path string
}

type Credential struct {
    Name     string
    APIKey   string
    Secret   string
    Metadata map[string]any
}

func (cs *CredentialStore) Save(name string, creds Credential) core.Result
func (cs *CredentialStore) Get(name string) (Credential, core.Result)
func (cs *CredentialStore) Delete(name string) core.Result
```

#### pkg/embed/ — Embedding utilities

**Files:**
- `embed.go` — Embedder interface
- `batch.go` — Batch embedding

**Types:**
```go
type Embedder interface {
    Embed(ctx context.Context, text string) ([]float64, error)
    EmbedBatch(ctx context.Context, texts []string) ([][]float64, error)
}
```

#### pkg/fusion/ — Model fusion

**Files:**
- `fusion.go` — Fusion strategies
- `ensemble.go` — Ensemble methods

**Fusion strategies:**
- Ensemble voting
- Weighted averaging
- Majority voting
- Confidence-based selection

#### pkg/kvtier/ — KV tier storage

**Files:**
- `kvtier.go` — Multi-tier KV storage
- `memory.go` — Memory tier
- `disk.go` — Disk tier
- `remote.go` — Remote tier

**Tiers:**
1. **Memory** — Fastest, ephemeral
2. **Disk** — Persistent, local
3. **Remote** — Distributed, scalable

#### pkg/lora/ — LoRA adapters

**Files:**
- `lora.go` — LoRA configuration
- `loader.go` — Adapter loading

**Types:**
```go
type LoRAConfig struct {
    BaseModel   string
    AdapterPath string
    Rank        int
    Alpha       float64
}
```

#### pkg/modality/ — Multi-modal support

**Files:**
- `modality.go` — Modality types
- `text.go` — Text modality
- `image.go` — Image modality
- `audio.go` — Audio modality
- `video.go` — Video modality
- `structured.go` — Structured data

**Modalities:**
- Text
- Image
- Audio
- Video
- Structured data

#### pkg/pipeline/ — Processing pipelines

**Files:**
- `pipeline.go` — Pipeline type
- `step.go` — Step type

**Types:**
```go
type Pipeline struct {
    Name    string
    Steps   []Step
    Context context.Context
}

type Step struct {
    Name string
    Run  func(ctx context.Context, input any) (any, error)
}

func (p *Pipeline) Run() core.Result
```

#### pkg/prompt/ — Prompt engineering

**Files:**
- `prompt.go` — Prompt builder
- `templates.go` — Prompt templates
- `variables.go` — Template variables

**Types:**
```go
type PromptBuilder struct {
    Template   string
    Variables  map[string]string
}

func (pb *PromptBuilder) Build() string
func (pb *PromptBuilder) WithContext(context string) string
```

#### pkg/session/ — Session management

**Files:**
- `session.go` — Session type
- `manager.go` — Session manager

**Types:**
```go
type Session struct {
    ID        string
    Messages  []Message
    State     map[string]any
    CreatedAt time.Time
    UpdatedAt time.Time
}

type SessionManager struct {
    Sessions map[string]*Session
}

func (sm *SessionManager) Create() *Session
func (sm *SessionManager) Get(id string) (*Session, bool)
func (sm *SessionManager) Delete(id string)
```

#### pkg/stream/ — Streaming

**Files:**
- `stream.go` — Stream handler
- `buffer.go` — Stream buffering

**Types:**
```go
type StreamHandler struct {
    Callback func(string)
}

type Stream struct {
    Chunk chan string
    Done chan error
}

func (sh *StreamHandler) Handle(ctx context.Context, stream Stream) error
```

#### pkg/structured/ — Structured outputs

**Files:**
- `structured.go` — Output formatting
- `json.go` — JSON output
- `yaml.go` — YAML output
- `xml.go` — XML output
- `csv.go` — CSV output
- `markdown.go` — Markdown tables

**Output types:**
- JSON
- YAML
- XML
- CSV
- Markdown tables

#### pkg/tools/ — Tool utilities

**Files:**
- `tools.go` — Tool manager
- `invoker.go` — Tool invoker

**Types:**
```go
type ToolManager struct {
    Tools map[string]Tool
}

type Tool struct {
    Name        string
    Description string
    Handler     func(ctx context.Context, args map[string]any) core.Result
}

func (tm *ToolManager) Register(name string, tool Tool)
func (tm *ToolManager) Invoke(name string, args map[string]any) core.Result
```

#### pkg/transform/ — Transformations

**Files:**
- `transform.go` — Transformation interface
- `text.go` — Text transformations
- `json.go` — JSON transformations

#### pkg/usage/ — Usage tracking

**Files:**
- `usage.go` — Usage tracker
- `metrics.go` — Usage metrics

---

### providers/ — Provider Implementations

#### providers/openai/ — OpenAI-compatible provider

**Files:**
- `client.go` — Client implementation
- `config.go` — Configuration
- `types.go` — Request/response types

**Types:**
```go
type Client struct {
    APIKey    string
    Endpoint  string
    Model     string
    Timeout   time.Duration
}

type GenerateRequest struct {
    Prompt      string
    MaxTokens   int
    Temperature float64
    TopP        float64
}

type ChatRequest struct {
    Messages []Message
    Model    string
}

func (c *Client) Generate(ctx context.Context, request GenerateRequest) core.Result
func (c *Client) Chat(ctx context.Context, messages []Message) core.Result
```

---

### cmd/ — CLI Commands

**Total Commands:** 11 command groups

#### cmd/ai/ — Unified AI commands

**Purpose:** Unified entry point for AI operations.

**Registration:** `AddAICommands(root *cli.Command)`

**Subcommands:**
- `ai generate` — Generate text from prompt
- `ai chat` — Start interactive chat session
- `ai complete` — Text completion
- `ai embed` — Generate text embeddings

**Flags (common):**
```
--model string       Model to use
--temperature float  Sampling temperature (default: 0.7)
--top-p float        Top-p sampling (default: 0.9)
--max-tokens int     Maximum tokens to generate
```

#### cmd/metrics/ — Metrics viewer command

**Purpose:** Display recorded AI metrics.

**Registration:** `AddMetricsCommand(parent *cli.Command)`

**Flags:**
```
--since duration    How far back to read events (default: 168h = 7 days)
--json             Output raw JSON instead of summary
--limit int        Limit number of events (default: 100)
```

**Flow:**
1. Parse `--since` flag to time.Duration
2. Call `ai.ReadEvents(time.Now().Add(-since))`
3. Call `ai.Summary(events)`
4. Render table (default) or JSON to stdout

**Example:**
```bash
# View metrics from last 24 hours
core ai metrics --since 24h

# View metrics as JSON
core ai metrics --json

# View metrics from last week
core ai metrics --since 168h
```

#### cmd/rag/ — RAG commands

**Purpose:** Re-export RAG commands from `go-rag`.

**Registration:** `var AddRAGSubcommands = ragcmd.AddRAGSubcommands`

**Subcommands:**

| Command | Description | Flags |
|---------|-------------|-------|
| `rag query` | Query vector database | `--collection`, `--limit`, `--threshold` |
| `rag index` | Index documents | `--collection`, `--file`, `--recursive` |
| `rag update` | Update indexed documents | `--collection`, `--id`, `--content` |
| `rag delete` | Delete from index | `--collection`, `--id` |
| `rag stats` | Show index statistics | `--collection` |

**Example:**
```bash
# Query the hostuk-docs collection
core ai rag query --collection hostuk-docs "build system"

# Index documents
core ai rag index --collection my-docs --file ./docs/*.md --recursive

# Show statistics
core ai rag stats --collection hostuk-docs
```

#### cmd/security/ — GitHub security scanning (5 subcommands)

**Purpose:** GitHub security scanning via `gh` CLI.

**Registration:** `AddSecurityCommand(parent *cli.Command)`

**Dependencies:** Requires `gh` CLI to be installed and authenticated.

**Subcommands:**

**1. Alerts:**
```bash
core ai security alerts [--repo OWNER/REPO] [--state open|closed|fixed] [--json]
```
- Lists security alerts for repositories
- Filters by state (open, closed, fixed)
- Outputs as JSON if `--json` flag is set
- Defaults to current repository if `--repo` not specified

**2. Dependencies:**
```bash
core ai security deps [--repo OWNER/REPO] [--manifest PATH] [--vulnerabilities] [--json]
```
- Scans dependencies for vulnerabilities
- Can specify manifest file with `--manifest`
- Shows only vulnerabilities with `--vulnerabilities`
- Outputs as JSON if `--json` flag is set

**3. Secrets:**
```bash
core ai security secrets [--repo OWNER/REPO] [--push] [--all] [--json]
```
- Scans for exposed secrets
- Pushes results to GitHub with `--push`
- Scans all repositories with `--all`
- Outputs as JSON if `--json` flag is set

**4. Scanning:**
```bash
core ai security scan [--repo OWNER/REPO] [--sarif OUTPUT] [--severity low|medium|high|critical] [--json]
```
- Runs security scans
- Outputs SARIF format to file with `--sarif`
- Filters by severity level
- Outputs as JSON if `--json` flag is set

**5. Jobs:**
```bash
core ai security jobs [--repo OWNER/REPO] [--status queued|in_progress|completed] [--json]
```
- Lists security scan jobs
- Filters by status
- Outputs as JSON if `--json` flag is set

**Example Usage:**
```bash
# Check alerts for current repo
core ai security alerts

# Check all dependencies
core ai security deps --all

# Run full security scan
core ai security scan --repo myorg/myrepo --sarif scan.sarif

# Push secrets scan results
core ai security secrets --push
```

#### cmd/lab/ — Homelab monitoring dashboard

**Purpose:** Homelab monitoring dashboard.

**Registration:** `AddLabCommand(parent *cli.Command)`

**Note:** Tagged with `build:ignore` to exclude from binary builds.

**Features:**
- Real-time metrics display
- Resource monitoring (CPU, memory, disk, GPU)
- Model serving status
- Performance metrics
- Historical data visualization

**Flags:**
```
--port int        HTTP server port (default: 8080)
--interval duration Refresh interval (default: 5s)
--theme string     UI theme (light, dark, system)
```

**Example:**
```bash
# Start lab dashboard on port 8080
core ai lab

# Start on custom port
core ai lab --port 3000
```

#### cmd/embed-bench/ — Embedding model benchmarks

**Purpose:** Benchmark embedding models for performance and accuracy.

**Registration:** Main entry point at `cmd/embed-bench/main.go`

**Features:**
- Compare embedding model performance
- Measure latency and throughput
- Test accuracy on various datasets
- Generate benchmark reports

**Flags:**
```
--models strings    Comma-separated model names
--dataset string    Dataset to use (default: internal)
--iterations int    Number of iterations (default: 100)
--output string     Output file for results
--json             Output as JSON
```

**Example:**
```bash
# Run benchmark on default models
core ai embed-bench

# Benchmark specific models
core ai embed-bench --models ollama:llama3,ollama:mistral

# Save results to file
core ai embed-bench --output benchmark-results.json
```

#### cmd/daemon/ — Background daemon

**Purpose:** Run AI services as background daemon.

**Registration:** `AddDaemonCommand(parent *cli.Command)`

**Features:**
- Long-running AI service
- Automatic restart on failure
- Logging and monitoring
- Configuration reload
- Health checks

**Flags:**
```
--config string     Configuration file path
--log-level string  Log level (debug, info, warn, error)
--pidfile string    PID file path
--umask string      File mode creation mask
```

**Example:**
```bash
# Start daemon with default config
core ai daemon

# Start with custom config
core ai daemon --config /etc/ai-daemon.yaml

# Start with debug logging
core ai daemon --log-level debug
```

#### cmd/lthn-ai/ — Lethean AI host

**Purpose:** Lethean AI host with MCP + embeddings + vector + State.

**Features:**
- MCP server hosting
- Embedding service integration
- Vector database access
- Session state management
- Multi-model support
- REST API

**Flags:**
```
--mcp-host string     MCP server host (default: localhost)
--mcp-port int        MCP server port (default: 8080)
--embed-host string    Embedding service host (default: localhost)
--embed-port int      Embedding service port (default: 11434)
--vector-host string   Vector database host (default: localhost)
--vector-port int     Vector database port (default: 6334)
```

**Example:**
```bash
# Start Lethean AI host
core ai lthn-ai

# Start with custom endpoints
core ai lthn-ai --mcp-port 9090 --embed-port 12345 --vector-port 7334
```

#### cmd/lem-runtime/ — Contained model service

**Purpose:** Contained model service with curated catalogue.

**Features:**
- Pre-packaged models
- Easy deployment
- Model catalogue management
- Version management
- Health monitoring

**Note:** Includes `lem-runtime` binary (144MB)

**Flags:**
```
--model-path string   Path to model directory
--catalog-path string  Path to model catalogue
--port int             HTTP server port (default: 8080)
```

**Example:**
```bash
# Start model runtime
core ai lem-runtime

# Start with custom model path
core ai lem-runtime --model-path /models

# Start on custom port
core ai lem-runtime --port 9000
```

#### cmd/lem-desktop/ — Wails desktop app

**Purpose:** Desktop application with Wails framework.

**Features:**
- Dashboard interface
- Control panel
- Workbench
- Multi-platform support (Windows, macOS, Linux)

**UI Components:**
- Angular 20 frontend
- Wails v3 backend
- Multiple services
- MCP tool integration

**Note:** Builds as native desktop application

---

## Usage patterns

### Pattern 1: Basic metrics recording

```go
package main

import (
    "time"
    "dappco.re/go/ai"
)

func main() {
    // Record a task completion
    err := ai.Record(ai.Event{
        Type: "task.complete",
        Repo: "myorg/myrepo",
        AgentID: "my-agent",
        Duration: 2 * time.Minute,
        Data: map[string]any{
            "task_type": "code_review",
            "files": 10,
        },
    })
    if err != nil {
        panic(err)
    }
}
```

### Pattern 2: RAG-enhanced task

```go
package main

import (
    "dappco.re/go/ai"
)

func main() {
    // Get context for a task
    contextResult := ai.QueryRAGForTask(ai.TaskInfo{
        Title: "Debug build issue",
        Description: "Build fails with undefined reference error in linker",
    })
    
    if !contextResult.OK {
        panic(contextResult.Err)
    }
    
    context := contextResult.Value.(string)
    println("Relevant context:", context)
    
    // Use context in AI model call
    // ...
}
```

### Pattern 3: MCP server with custom tools

```go
package main

import (
    "context"
    "os"
    "dappco.re/go/ai/mcp"
)

func main() {
    service := mcp.NewService()
    
    // Register custom tools
    service.RegisterTool(mcp.Tool{
        Name: "get_time",
        Description: "Get current time",
        Handler: func(ctx context.Context, args map[string]any) core.Result {
            return core.Ok(map[string]any{"time": time.Now().String()})
        },
    })
    
    service.RegisterTool(mcp.Tool{
        Name: "read_file",
        Description: "Read file contents",
        Handler: func(ctx context.Context, args map[string]any) core.Result {
            path := args["path"].(string)
            content := core.ReadFile(path)
            return core.Ok(map[string]any{"content": content})
        },
    })
    
    // Use stdio transport
    transport := &mcp.StdioTransport{
        Stdin: os.Stdin,
        Stdout: os.Stdout,
    }
    
    service.Transports = []mcp.Transport{transport}
    
    // Start service
    if err := service.Run(context.Background()); err != nil {
        panic(err)
    }
}
```

### Pattern 4: Full AI workflow with metrics and RAG

```go
package main

import (
    "context"
    "time"
    "dappco.re/go/ai"
    "dappco.re/go/ai/mcp"
)

func processTask(task ai.TaskInfo) core.Result {
    // Start timer
    start := time.Now()
    
    // Record task start
    if err := ai.Record(ai.Event{
        Type: "task.start",
        Repo: task.Repo,
        Data: map[string]any{"task": task.Title},
    }); err != nil {
        return err
    }
    
    // Get RAG context
    contextResult := ai.QueryRAGForTask(task)
    if !contextResult.OK {
        return contextResult
    }
    context := contextResult.Value.(string)
    
    // Process task with context
    // ...
    
    // Record task completion
    return ai.Record(ai.Event{
        Type: "task.complete",
        Repo: task.Repo,
        Duration: time.Since(start),
        Data: map[string]any{
            "task": task.Title,
            "status": "success",
            "context_used": len(context) > 0,
        },
    })
}
```

### Pattern 5: Session management with KV storage

```go
package main

import (
    "dappco.re/go/ai/pkg/sessionkv"
)

func main() {
    // Create new session
    session := sessionkv.NewSession()
    session.ID = "user-123"
    
    // Set session state
    session.Set("theme", "dark")
    session.Set("language", "en-US")
    session.Set("last_activity", time.Now())
    
    // Save session
    if err := session.Save(); err != nil {
        panic(err)
    }
    
    // Load session
    loaded := sessionkv.NewSession()
    if err := loaded.Load("user-123"); err != nil {
        panic(err)
    }
    
    theme := loaded.Get("theme")
    println("Theme:", theme)
}
```

### Pattern 6: Chat history tracking

```go
package main

import (
    "dappco.re/go/ai/pkg/chathistory"
)

func main() {
    // Create new chat history
    history := &chathistory.History{
        SessionID: "chat-456",
    }
    
    // Add messages
    history.AddMessage(chathistory.Message{
        Role: "system",
        Content: "You are a helpful AI assistant",
    })
    
    history.AddMessage(chathistory.Message{
        Role: "user",
        Content: "What is CoreGo?",
    })
    
    history.AddMessage(chathistory.Message{
        Role: "assistant",
        Content: "CoreGo is a Go framework...",
    })
    
    // Get last N messages for context
    context := history.GetContext(10)
    
    // Save history
    if err := history.Save("chat-456.json"); err != nil {
        panic(err)
    }
}
```

### Pattern 7: Content welfare check

```go
package main

import (
    "dappco.re/go/ai/pkg/welfare"
)

func checkContent(content string) bool {
    result := welfare.Detect(content)
    
    if !result.IsSafe {
        println("Unsafe content detected:")
        for _, reason := range result.Reasons {
            println("  -", reason)
        }
        return false
    }
    
    return true
}
```

---

## Configuration

### Environment variables

| Variable | Description | Default | Used By |
|----------|-------------|---------|---------|
| `CORE_HOME` | Core home directory | `~/.core` | Metrics storage |
| `HOME` | User home directory | System default | Fallback for CORE_HOME |
| `USERPROFILE` | Windows user profile | System default | Windows fallback |

### Metrics storage

**Location:** `~/.core/ai/metrics/`

**Structure:**
```
~/.core/ai/metrics/
├── 2026-06-01.jsonl
├── 2026-06-02.jsonl
├── 2026-06-03.jsonl
└── ...
```

**File Format:**
- Plain text JSONL (JSON Lines)
- Each line: JSON-encoded `Event` struct
- Line endings: LF (`\n`)
- Max line size: 1MB

**Retention:**
- Default: 365 days (1 year)
- Auto-rotation: Daily files
- Cleanup: Old files not automatically deleted (manual cleanup recommended)

### Default endpoints

| Service | Endpoint | Port | Purpose |
|---------|----------|------|---------|
| Qdrant | localhost | 6334 | Vector database |
| Ollama | localhost | 11434 | Embedding models |
| MCP | localhost | 8080 | Model Context Protocol |

---

## Testing

### Test structure

All packages follow the **AX-7 Triplet Pattern**:

```
ai/
├── metrics.go                # Main implementation
├── metrics_test.go           # Unit tests (Good/Bad/Ugly)
├── metrics_example_test.go   # Usage examples
└── stdlib_assert_test.go     # Standard library compliance

mcp/
├── service.go
├── service_test.go
├── service_example_test.go
└── stdlib_assert_test.go

cmd/metrics/
├── cmd.go
├── cmd_test.go
├── cmd_example_test.go
└── stdlib_assert_test.go
```

### Test categories

| Category | Pattern | Purpose | Count |
|----------|---------|---------|-------|
| Good | `Test*_Good` | Happy path, expected inputs | 50+ |
| Bad | `Test*_Bad` | Error conditions, invalid inputs | 25+ |
| Ugly | `Test*_Ugly` | Edge cases, unusual conditions | 10+ |
| Example | `Test*_Example` | Usage examples | 15+ |

### Key test files

| File | Purpose |
|------|---------|
| `ai/metrics_test.go` | Metrics recording and reading |
| `ai/rag_test.go` | RAG query functionality |
| `mcp/service_test.go` | MCP service testing |
| `mcp/jsonrpc_test.go` | JSON-RPC transport |
| `mcp/tools_core_test.go` | Core tools |
| `mcp/tools_external_test.go` | External tools |
| `cmd/metrics/cmd_test.go` | Metrics CLI command |
| `cmd/security/cmd_alerts_test.go` | Security alerts command |
| `cmd/security/cmd_deps_test.go` | Security dependencies command |
| `cmd/security/cmd_secrets_test.go` | Security secrets command |
| `cmd/security/cmd_scan_test.go` | Security scanning command |
| `cmd/security/cmd_jobs_test.go` | Security jobs command |

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

# Specific test
cd go
go test ./ai -run TestRecord_Good

# Verbose mode
cd go
go test -v ./ai
```

### Benchmark examples

```bash
# Run all benchmarks
cd go
go test -bench=. ./...

# Run specific benchmark
cd go
go test -bench=BenchmarkRecord ./ai

# Run with memory profiling
cd go
go test -bench=. -memprofile=mem.prof ./mcp

# Run with CPU profiling
cd go
go test -bench=. -cpuprofile=cpu.prof ./mcp
```

---

## API reference

### Type index

| Type | Package | Description |
|------|---------|-------------|
| `Event` | `ai` | Metrics event structure |
| `TaskInfo` | `ai` | Task information for RAG queries |
| `ProviderRoute` | `ai` | Provider routing information |
| `DifferentialLoadAction` | `ai` | Differential loading action |
| `Service` | `mcp` | MCP service host |
| `Tool` | `mcp` | MCP tool definition |
| `Transport` | `mcp` | Transport interface |
| `StdioTransport` | `mcp` | stdio transport |
| `TCPTransport` | `mcp` | TCP transport |
| `UnixTransport` | `mcp` | Unix socket transport |
| `Session` | `pkg/sessionkv` | Session state |
| `Message` | `pkg/chathistory` | Chat message |
| `History` | `pkg/chathistory` | Chat history |
| `Driver` | `pkg/driver` | Model driver interface |
| `DetectResult` | `pkg/welfare` | Welfare detection result |

### Function index

| Function | Package | Description |
|----------|---------|-------------|
| `Record()` | `ai` | Record metrics event |
| `ReadEvents()` | `ai` | Read events from metrics storage |
| `Summary()` | `ai` | Aggregate events into summary |
| `QueryRAGForTask()` | `ai` | Query RAG for task context |
| `NewService()` | `mcp` | Create MCP service |
| `Run()` | `mcp` | Start MCP service |
| `RegisterTool()` | `mcp` | Register MCP tool |
| `HandleRequest()` | `mcp` | Handle JSON-RPC request |
| `Detect()` | `pkg/welfare` | Detect unsafe content |
| `NewSession()` | `pkg/sessionkv` | Create new session |
| `AddMessage()` | `pkg/chathistory` | Add message to history |
| `Generate()` | `pkg/driver` | Generate text with driver |

### Constant index

| Constant | Package | Value | Description |
|----------|---------|-------|-------------|
| `recentEventLimit` | `ai` | 10 | Recent events limit in summary |
| `maxMetricsReadWindowDays` | `ai` | 365 | Max days to read events |
| `maxMetricsLineBytes` | `ai` | 1<<20 | Max bytes per line |
| `metricsFileMode` | `ai` | 0o600 | File permissions |
| `metricsDirMode` | `ai` | 0o700 | Directory permissions |
| `ragTaskCollection` | `ai` | "hostuk-docs" | Default RAG collection |
| `ragTaskResultLimit` | `ai` | 3 | Max RAG results |
| `ragTaskSimilarityThreshold` | `ai` | 0.5 | Similarity threshold |
| `ragTaskQueryRuneLimit` | `ai` | 500 | Query truncation limit |

---

## Related documentation

### RFC documents

| Document | Location | Description | Size |
|----------|----------|-------------|------|
| Main RFC | [`RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.md) | Complete specification | 93KB |
| Models RFC | [`RFC.models.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.models.md) | Data model definitions | 7KB |
| Commands RFC | [`RFC.commands.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.commands.md) | Command specifications | 400 bytes |
| Catalog RFC | [`RFC.catalog.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.catalog.md) | Tool catalog | 93KB |
| Imports RFC | [`RFC.imports.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.imports.md) | Import specifications | 479 bytes |
| CLAUDE.md | [`CLAUDE.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ai/CLAUDE.md) | Claude-specific notes | 3KB |

### Repository documentation

| Document | Location | Description |
|----------|----------|-------------|
| README.md | [`README.md`](file:///Users/snider/Code/core/go-ai/README.md) | Repository overview | 1KB |
| AGENTS.md | [`AGENTS.md`](file:///Users/snider/Code/core/go-ai/AGENTS.md) | Agent guidance | 1.6KB |
| CLAUDE.md | [`CLAUDE.md`](file:///Users/snider/Code/core/go-ai/CLAUDE.md) | Claude-specific notes | 3.3KB |
| GOAL.md | [`GOAL.md`](file:///Users/snider/Code/core/go-ai/GOAL.md) | Goals and objectives | 24KB |
| CONTRIBUTING.md | [`CONTRIBUTING.md`](file:///Users/snider/Code/core/go-ai/CONTRIBUTING.md) | Contribution guide | 1KB |
| TEST-RESULTS.md | [`TEST-RESULTS.md`](file:///Users/snider/Code/core/go-ai/TEST-RESULTS.md) | Test results | 9KB |
| LICENCE | [`LICENCE`](file:///Users/snider/Code/core/go-ai/LICENCE) | EUPL-1.2 license | 14KB |

### Knowledge pack documentation

| Document | Location | Description |
|----------|----------|-------------|
| README.md | [`README.md`](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/ai/README.md) | This document's companion | 35KB |
| CoreGo INDEX | [`INDEX.md`](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) | CoreGo package catalog | 240 |

---

## Metadata

### Package information

```yaml
name: go-ai
module: dappco.re/go/ai
repository: github.com/dappcore/go-ai
spec: file:///Users/snider/Code/meowmix/plans/code/core/go/ai/RFC.md
version: v1.0.0
license: EUPL-1.2
status: Production
maintainer: Purberus <purberus@lthn.ai>
```

### Document information

```yaml
title: go-ai Package Index
type: INDEX
description: Complete index for the go-ai package with all types, functions, and concepts
file: knowledge-packs/corego/pkg/ai/INDEX.md
size: 36KB
lines: ~1000
generated_by: Purberus
commit: 3cc71f7
date: 2026-06-17
```

### Related packages

| Package | Relationship | Purpose |
|---------|--------------|---------|
| `go-ml` | Sibling | Scoring engine, heuristics, probes |
| `go-inference` | Sibling | Shared ML backend interfaces |
| `go-rag` | Sibling | Qdrant vector DB + Ollama embeddings |
| `go-config` | Dependency | Configuration management |
| `go-io` | Dependency | Storage abstractions |
| `go-log` | Dependency | Logging |
| `go-scm` | Dependency | SCM utilities |
| `go-i18n` | Dependency | Internationalization |

---
