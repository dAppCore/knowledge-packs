---
type: Package Index
package: agent
module: dappco.re/go/agent
repo: core/agent
title: go-agent Package Index
description: AI agent orchestration platform for the Core ecosystem with MCP server, dispatch, and fleet management
tags:
  - ai-agent
  - orchestration
  - dispatch
  - mcp-server
  - fleet
  - claude-code
  - codex
  - gemini
  - opencode
  - brain
  - workspace
  - sandboxing
---
# go-agent Package Index

> **AI Agent Orchestration Platform for the Core Ecosystem**

**Repository:** `core/agent`  
**Module:** `dappco.re/go/agent`  
**Binary:** `core-agent` / `lthn-agent`  
**PHP Package:** `forge.lthn.ai/core/agent`  
**Status:** ✅ Production-Ready  
**License:** EUPL-1.2  
**Test Pattern:** Good/Bad/Ugly (AX Standard)  
**RFC:** [plans/code/core/go/agent/RFC.md](../../../../../plans/code/core/go/agent/RFC.md)
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| RFC | Request for Comments specification | [plans/code/core/go/agent/RFC.md](../../../../../plans/code/core/go/agent/RFC.md) |
| GOAL | Implementation goals and parity gate | [~/Code/core/agent/GOAL.md](file:///Users/snider/Code/core/agent/GOAL.md) |
| CLAUDE.md | Development guidance for Claude Code | [~/Code/core/agent/CLAUDE.md](file:///Users/snider/Code/core/agent/CLAUDE.md) |
| AGENTS.md | Agent guidance and conventions | [~/Code/core/agent/AGENTS.md](file:///Users/snider/Code/core/agent/AGENTS.md) |
| Repository README | Original package README | [~/Code/core/agent/README.md](file:///Users/snider/Code/core/agent/README.md) |

---

## 🎯 Package Overview

`go-agent` is the **AI agent orchestration platform** for the Core ecosystem, providing a single Go binary (`core-agent` or `lthn-agent`) that runs as a Model Context Protocol (MCP) server and CLI tool. It serves as the **AUI** (Agent-facing User Interface) where agents wield the system headlessly, with `lthn/desktop` as its **HUI** (Human-facing User Interface) twin.

### Core Capabilities

1. **Agent Dispatch** — Fan out tasks to sandboxed workers (Claude, Codex, Hermes, Google) running in isolated workspaces
2. **MCP Server** — Full Model Context Protocol implementation for IDE integrations (Claude Code, Cursor, etc.)
3. **HTTP Daemon** — Cross-agent communication via HTTP MCP for CI and remote use
4. **Hub Control Plane** — Loopback control plane with Bearer authentication for opencode control/proxy groups
5. **Fleet Management** — Pull/merge/push across Core ecosystem repos via `agents.yaml` configuration
6. **OpenBrain Integration** — Durable semantic memory and cross-agent messaging via Postgres + Qdrant + Ollama
7. **Provider Integrations** — First-class support for Claude Code, Codex, Hermes, and Google Gemini
8. **Chat Archive** — Per-user portable DuckDB chat archive for conversation history
9. **Container Runtime** — Local and container-based runners with dispatch queue
10. **Workspace Orchestration** — Full workspace lifecycle management with spec tree copying

### Design Principles

- **Single Binary** — One binary with multiple modes (mcp, serve, hub, chat)
- **Core-Native** — Full integration with CoreGO framework (ServiceRuntime, ACTION system, Result pattern)
- **Provider-Agnostic** — Abstract provider differences behind common interfaces
- **Sandboxed** — All agent work runs in isolated workspaces with restricted file system access
- **Observable** — Rich event streaming via Core ACTIONs and channel-based event system
- **Testable** — Complete test triplet coverage (_test.go + _example_test.go)
- **Extensible** — Subsystem architecture for custom tool groups
- **Portable** — Works across Unix and Windows (where applicable)

### Binary Name Detection

The binary detects its invocation name from `argv[0]`:
- `core-agent` — Legacy name
- `lthn-agent` — Forward-going family-consistent name (lthn-{mlx,cuda,amd,agent} family)
- Either produces the same behavior; `lthn-agent` is preferred for consistency

---

## 🏗️ Architecture

### Component Layers

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Binary Layer                                             │
│  core-agent / lthn-agent binary                                            │
│  Mode detection from argv[0]                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Command Layer                                            │
│  mcp, serve, hub, chat, serve-status, serve-reload, serve-profiles         │
│  models-download, models-job, run flow                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Service Layer                                             │
│  Agentic Service — Dispatch, prep, verify, scan, remote, mirror, plans    │
│  Brain Service — OpenBrain client (recall, remember, forget, list, msg)     │
│  Lemma Service — Local lthn-mlx client (chat + /v1/admin control)         │
│  Monitor Service — Background monitoring + repo sync                        │
│  Runner Service — Local + container runners + dispatch queue               │
│  Setup Service — Project detection + .core/ scaffolding                     │
│  MCP Service — Model Context Protocol server framework                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Package Layer (pkg/)                                      │
│  agentic/ — Dispatch, workspace, plans/phases/sessions, fleet sync          │
│  brain/ — OpenBrain client with semantic memory operations                 │
│  chathistory/ — Portable DuckDB chat archive                               │
│  lemma/ — Local ML inference client (lthn-mlx)                             │
│  lib/ — Embedded personas, prompt + flow + workspace templates              │
│  messages/ — Typed IPC message definitions                                  │
│  monitor/ — Background monitoring + repository synchronization            │
│  opencode/ — OpenCode integration (sandboxed host)                        │
│  agentcompat/ — Agent compatibility utilities                             │
│  runner/ — Process runners (local + container) + dispatch queue           │
│  setup/ — Project detection + workspace scaffolding                         │
│  audit/ — Audit logging for all operations                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Provider Layer (provider/)                                │
│  claude/ — Claude Code plugin (8 marketplace plugins)                       │
│  codex/ — Codex CLI integration                                               │
│  hermes/ — Hermes plugin sources + skills                                    │
│  google/ — Google Gemini integration scaffold                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Data Layer                                                │
│  ~/Lethean/workspace/ — Task workspaces                                      │
│  ~/Lethean/data/ — Persistent storage (DuckDB, workspace state)            │
│  ~/Lethean/conf/ — Configuration (agents.yaml, workspace.yaml)              │
│  ~/Lethean/log/ — Agent logs                                                │
│  .core/ — Runtime configuration and scaffolding                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Repository Layout

```
core/agent/
├── go/                                  # Go module root (dappco.re/go/agent)
│   ├── cmd/core-agent/                 # Binary entry point
│   │   ├── main.go                    # Main entry + Core integration
│   │   ├── commands.go                # CLI command registration
│   │   ├── commands_chat.go           # Chat command handlers
│   │   ├── commands_forge.go          # Forge integration commands
│   │   ├── commands_hub.go            # Hub control plane commands
│   │   ├── commands_message.go        # Message handling commands
│   │   ├── commands_models.go         # Model management commands
│   │   ├── commands_opencode.go       # OpenCode integration commands
│   │   ├── commands_phase.go          # Phase management commands
│   │   ├── commands_serve.go          # Serve command handlers
│   │   ├── lemma_mcp.go               # Lemma MCP server integration
│   │   ├── update.go                  # Update functionality
│   │   └── *_{test,example_test}.go  # Test triplets
│   │
│   ├── pkg/
│   │   ├── agentcompat/               # Agent compatibility utilities
│   │   │   ├── agentcompat.go         # Compatibility layer
│   │   │   └── *_{test,example_test}.go
│   │   │
│   │   ├── agentic/                  # **Core orchestration engine**
│   │   │   ├── paths.go               # Shared path helpers
│   │   │   ├── register.go            # Service registration
│   │   │   ├── runner.go              # Dispatch runner
│   │   │   ├── deps.go                # Dependency management
│   │   │   ├── epic.go                # Epic management
│   │   │   ├── issue.go               # Issue tracking
│   │   │   ├── mirror.go              # Repository mirroring
│   │   │   ├── plan.go                # Plan management
│   │   │   ├── prep.go                # Workspace preparation
│   │   │   ├── qa_analysis.go         # QA analysis utilities
│   │   │   ├── qa_cluster.go          # QA cluster management
│   │   │   ├── queue.go               # Task queue management
│   │   │   ├── remote_status.go       # Remote status tracking
│   │   │   ├── resume.go              # Session resume
│   │   │   ├── review_queue.go        # Review queue management
│   │   │   ├── scan.go                # Repository scanning
│   │   │   ├── status.go              # Agent status tracking
│   │   │   ├── watch.go               # Workspace watching
│   │   │   ├── fetch_loop.go          # Fetch loop for updates
│   │   │   ├── branch_cleanup.go       # Branch cleanup utilities
│   │   │   ├── pipeline_*.go          # Pipeline management
│   │   │   ├── commands_*.go          # Command handlers
│   │   │   ├── transport.go           # Transport utilities
│   │   │   ├── forge_client.go        # Forge API client
│   │   │   ├── brain_client.go        # Brain client wrapper
│   │   │   ├── provider_manager.go    # Provider management
│   │   │   └── sanitise.go            # Input sanitization
│   │   │
│   │   │   └── .core/                 # Runtime state
│   │   │       └── state/              # State management
│   │   │
│   │   ├── audit/                     # Audit logging
│   │   │   └── audit.go
│   │   │
│   │   ├── brain/                     # **OpenBrain integration**
│   │   │   ├── brain.go               # Main brain client
│   │   │   ├── bridge_events.go       # Bridge event handling
│   │   │   ├── client/                # Brain client
│   │   │   │   ├── client.go
│   │   │   │   └── coreio_compat.go   # CoreIO compatibility
│   │   │   ├── direct.go             # Direct operations
│   │   │   ├── provider.go           # Brain provider
│   │   │   └── tools.go              # Brain MCP tools
│   │   │
│   │   ├── chathistory/               # **DuckDB chat archive**
│   │   │   ├── chathistory.go         # Main chat history operations
│   │   │   ├── export.go              # Export functionality
│   │   │   └── migrations/            # Database schema migrations
│   │   │
│   │   ├── lemma/                     # **Local ML inference**
│   │   │   └── lemma.go               # LEM (lthn-mlx) client
│   │   │
│   │   ├── lib/                       # **Embedded templates**
│   │   │   ├── flow/                  # Flow templates
│   │   │   │   └── upgrade/           # Upgrade flow definitions
│   │   │   ├── persona/               # Persona definitions
│   │   │   │   ├── code/              # Code review personas
│   │   │   │   ├── secops/            # Security personas
│   │   │   │   └── testing/           # Testing personas
│   │   │   ├── prompt/                # Prompt templates
│   │   │   ├── task/                  # Task templates
│   │   │   │   └── code/              # Code task templates
│   │   │   │       ├── review/        # Code review subtasks
│   │   │   │       └── simplifier/    # Code simplification
│   │   │   └── workspace/             # Workspace templates
│   │   │       └── default/          # Default workspace template
│   │   │
│   │   ├── messages/                  # **IPC message definitions**
│   │   │   └── messages.go            # Message type definitions
│   │   │
│   │   ├── monitor/                   # **Background monitoring**
│   │   │   ├── monitor.go             # Main monitor service
│   │   │   ├── sync.go                # Repository synchronization
│   │   │   ├── harvest.go             # Data harvesting from workspaces
│   │   │   ├── register.go            # Monitor service registration
│   │   │   └── *_{test,example_test}.go
│   │   │
│   │   ├── opencode/                  # **OpenCode integration**
│   │   │   ├── opencode.go            # OpenCode client
│   │   │   └── internal/             # Internal utilities
│   │   │       ├── paths/            # Path utilities
│   │   │       └── sigkeys/           # Signature key management
│   │   │
│   │   ├── runner/                    # **Process runners**
│   │   │   ├── runner.go              # Main runner implementation
│   │   │   ├── queue.go               # Dispatch queue with priority
│   │   │   └── paths.go               # Runner path utilities
│   │   │
│   │   └── setup/                     # **Workspace setup**
│   │       ├── config.go             # Setup configuration types
│   │       ├── detect.go              # Project type detection
│   │       ├── service.go             # Setup service implementation
│   │       └── setup.go               # Main setup logic
│   │
│   ├── version.go                     # Version information
│   ├── *_{test,example_test}.go      # Version test triplets
│   ├── go.mod
│   ├── go.sum
│   └── go.work
│
├── php/                                # PHP Laravel package
│   ├── src/                           # PHP source
│   └── composer.json                  # PHP dependencies
│
├── provider/                          # Provider plugin sources
│   ├── claude/                        # Claude Code plugins
│   │   ├── core/                      # Core plugin
│   │   ├── core-go/                   # Core-Go plugin
│   │   ├── core-php/                  # Core-PHP plugin
│   │   ├── devops/                    # DevOps plugin
│   │   ├── infra/                     # Infrastructure plugin
│   │   ├── research/                  # Research plugin
│   │   ├── hermes_runner_mcp/         # Hermes runner MCP plugin
│   │   ├── camofox_mcp/               # Camofox MCP plugin
│   │   └── plugins/                   # Marketplace-flavoured subset
│   ├── codex/                         # Codex CLI integration
│   │   └── ...
│   ├── hermes/                        # Hermes plugin
│   │   └── ...
│   └── google/                        # Google Gemini integration
│       └── ...
│
├── .core/                             # Runtime configuration
│   ├── agents.yaml                    # Agent configuration (concurrency, quotas, etc.)
│   ├── workspace.yaml                 # Workspace templates
│   └── ...
│
├── vm/                                # Virtual machine configurations
│   └── docker/                       # Docker configurations
│
├── docs/                              # Documentation
│   ├── RFCs/                          # Request for Comments
│   └── ...
│
├── scripts/                           # Utility scripts
│   └── ...
│
├── .claude-plugin/                    # Claude Code plugin directory
│   └── marketplace.json                # Marketplace plugin manifest
│
├── .mcp.json                          # MCP server configuration
├── .gitmodules                        # Git submodules
├── .gitleaks.toml                    # Security scanning config
├── .gitignore
├── AGENTS.md                         # Agent guidance
├── CLAUDE.md                         # Development guidance
├── GOAL.md                           # Implementation goals
├── LICENCE                           # EUPL-1.2 license
├── module-graph.json                 # Dependency snapshot
├── README.md                         # Repository README
├── Taskfile.yaml                     # Build orchestration
└── sonar-project.properties          # SonarCloud configuration
```

---

## 🎯 Binary Modes

The `core-agent` binary supports multiple operation modes, each serving a specific purpose in the agent ecosystem.

### 1. MCP Mode (`core-agent mcp`)

**Purpose:** Stdio MCP server for coding-agent host integration (Claude Code, Cursor, etc.)

**Features:**
- Full Model Context Protocol specification compliance
- Stdio transport (default for Claude Code)
- File operations with workspace sandboxing
- Tool registration for all agent capabilities
- Notification broadcasting to connected sessions
- Channel-based event system for cross-component communication
- Automatic plugin registration via `.claude-plugin/marketplace.json`

**Usage:**
```bash
# Default for Claude Code integration
core-agent mcp

# Or as lthn-agent
lthn-agent mcp
```

### 2. Serve Mode (`core-agent serve`)

**Purpose:** HTTP MCP daemon for cross-agent communication, CI, and remote use

**Features:**
- REST API transport with Bearer authentication
- Multiple concurrent sessions
- Process isolation per session
- Resource quotas and rate limiting
- Streaming responses (SSE)

**Environment Variables:**
- `MCP_HTTP_ADDR` — HTTP transport address (e.g., `localhost:8081`)
- `MCP_AUTH_TOKEN` — Bearer token for authentication

**Usage:**
```bash
MCP_HTTP_ADDR=localhost:8081 \
  MCP_AUTH_TOKEN=your-secret-token \
  core-agent serve
```

### 3. Hub Mode (`core-agent hub`)

**Purpose:** Loopback control plane for opencode control/proxy groups and brain

**Features:**
- Bearer-authenticated HTTP endpoint at `127.0.0.1:9201`
- Fail-closed MCP HTTP+SSE plane at `127.0.0.1:9202`
- Non-optional audit edge recording every request
- Opencode control and proxy groups
- Brain integration with semantic memory
- Fleet management capabilities

**Flags:**
- `--http <addr>` — HTTP control plane address (default: `127.0.0.1:9201`)
- `--mcp-http <addr>` — MCP HTTP+SSE address (default: `127.0.0.1:9202`)

**Usage:**
```bash
core-agent hub --http 127.0.0.1:9201 --mcp-http 127.0.0.1:9202
```

### 4. Chat Mode (`core-agent chat --user=<id>`)

**Purpose:** REPL against the local LEM engine (lthn-mlx / lthn-ai driver)

**Features:**
- Interactive chat with local models
- Auto-capture to user's portable DuckDB archive
- Session persistence across restarts
- Multi-turn conversation support
- Model profile switching

**Usage:**
```bash
# Start chat session for user
core-agent chat --user=purberus

# With specific model
core-agent chat --user=purberus --model=gemma3-1b
```

### 5. Model Management Commands

**Purpose:** Manage local model engine and downloads

| Command | Description |
|---------|-------------|
| `serve-status` | Inspect local model engine status |
| `serve-reload` | Hot-swap model engine configurations |
| `serve-profiles` | List available model profiles |
| `models-download` | Queue Hugging Face model downloads |
| `models-job` | Poll download job status |

**Usage:**
```bash
# List available models
core-agent serve-profiles

# Check engine status
core-agent serve-status

# Queue model download
core-agent models-download --model gemma3-1b --quant q4_0

# Check download progress
core-agent models-job --id <job-id>
```

### 6. Flow Execution (`core-agent run flow <path>`)

**Purpose:** Execute predefined YAML workflows

**Features:**
- YAML-based workflow definitions
- Step-by-step execution with error handling
- Artifact collection and storage
- Progress reporting

**Usage:**
```bash
# Run a workflow
core-agent run flow /path/to/workflow.yaml

# With variables
core-agent run flow /path/to/workflow.yaml --var key=value
```

---

## 🚀 Core Services

### 1. Agentic Service (`pkg/agentic/`)

**The core orchestration engine** for agent dispatch and workspace management.

#### Key Responsibilities:
- Task dispatch to AI agents
- Workspace preparation and management
- Plan, phase, and session lifecycle
- Fleet synchronization across repositories
- Repository mirroring and scanning
- QA analysis and review queue management

#### Path Utilities (`paths.go`):

All paths use environment variables with sensible defaults:

| Function | Description | Default |
|----------|-------------|---------|
| `WorkspaceRoot()` | Root workspace directory | `~/Lethean/workspace/` |
| `CoreRoot()` | Core data root | `~/Lethean/data/` |
| `LetheanHome()` | Lethean home directory | `~/Lethean/` |
| `ConfDir()` | Configuration directory | `~/Lethean/conf/` |
| `DataDir()` | Data directory | `~/Lethean/data/` |
| `LogDir()` | Log directory | `~/Lethean/log/` |
| `AgentName()` | Current agent name | `cladius` (Snider's Mac) / `charon` |
| `GitHubOrg()` | Default GitHub organization | `dAppCore` |
| `PlansRoot()` | Plans directory | `<CoreRoot>/plans` |
| `AgentsConfigPath()` | Agents config path | `<ConfDir>/agents.yaml` |
| `WorkspaceRepoDir()` | Repo dir in workspace | `<workspace>/repo` |
| `WorkspaceMetaDir()` | Meta dir in workspace | `<workspace>/.meta` |
| `WorkspaceBlockedPath()` | BLOCKED.md path | `<repo>/BLOCKED.md` |
| `WorkspaceAnswerPath()` | ANSWER.md path | `<repo>/ANSWER.md` |
| `WorkspaceStatusPath()` | Status path | `<workspace>/status.json` |
| `WorkspaceLogFiles()` | Agent log files | `.meta/agent-*.log` |

#### Environment Variable Overrides:
- `CORE_WORKSPACE` — Override workspace root
- `LETHEAN_HOME` — Override Lethean home
- `AGENT_NAME` — Override agent name
- `GITHUB_ORG` — Override GitHub organization
- `CORE_HOME` — Override Core home (fallback to `HOME` or `DIR_HOME`)

#### Workspace Preparation Flow:

```
Input: PrepInput{Repo, Org, Task, Agent, Issue, PR, Branch, Tag, Template, Persona, Variables, DryRun}
    ↓
1. Resolve workspace directory under WorkspaceRoot()
   └── Format: ~/Lethean/workspace/{org}/{repo}/{task-N | pr-N | branch | tag}
    ↓
2. Clone repository from local mirror
   └── Source: ~/Code/{org}/{repo} (fast, kept fresh by post-completion sync)
   └── Re-prep pulls --ff-only instead of cloning
    ↓
3. Create working branch
   └── Branch: agent/{task-slug}
    ↓
4. Clone workspace dependencies
    ↓
5. Copy spec tree
   └── plans/.../RFC*.md → specs/
   └── org docs repo → .core/reference/docs/
    ↓
6. Build agent prompt
   └── Combines task, context, persona
   └── Writes prompt snapshot
    ↓
Output: PrepOutput{Success, WorkspaceDir, RepoDir, Branch, Prompt, PromptVersion, Memories, Consumers, Resumed}
```

#### Dispatch Flow:

```
Task → Queue
    ↓
Concurrency Gate → Rate Gate
    ↓
Workspace Prep (see above)
    ↓
Container Spawn
    ↓
Agent Runs
    ↓
Completion Pipeline:
    ├─► Review
    ├─► Fix
    ├─► Simplify
    └─► Re-review
    ↓
Commit → Auto PR → Inline Tests → Pass → Auto-merge on Forge
    ↓
Push to GitHub → CodeRabbit reviews → Merge or dispatch fix agent
```

#### Agent Types:

| Agent Identifier | Provider | Use Case | Speed | Cost |
|-----------------|----------|----------|-------|------|
| `claude:opus` | Claude Code | Complex coding, architecture | Slow | High |
| `claude:sonnet` | Claude Code | Standard tasks | Medium | Medium |
| `claude:haiku` | Claude Code | Quick/cheap tasks, discovery | Fast | Low |
| `gemini` | Google | Fast batch operations | Fast | Medium |
| `codex` | Codex | Autonomous coding | Medium | High |
| `codex:review` | Codex | Deep security analysis | Medium | High |
| `coderabbit` | CodeRabbit | Code quality review | Fast | Medium |
| `opencode` | OpenCode | Sandboxed agent (local/free-compute) | Variable | Free |
| `local` | Ollama bridge | Local OSS models | Fast | Free |

#### Provider Management:

The `provider_manager.go` handles:
- Provider registration and discovery
- Capability matching
- Authentication and rate limiting
- Error handling and retry logic
- Provider-specific configuration

### 2. Brain Service (`pkg/brain/`)

**OpenBrain integration** for semantic memory and cross-agent messaging.

#### Capabilities:
- `brain_recall` — Recall from brain with tags, type, and confidence filters
- `brain_remember` — Store knowledge in brain with metadata and supersession chains
- `brain_forget` — Remove entries from brain (soft delete with supersession)
- `brain_search` — Semantic search across all knowledge
- `brain_list` — List brain entries with pagination
- Cross-agent messaging via `agent_send`, `agent_inbox`, `agent_conversation`

#### Data Model:

```go
type BrainMemory struct {
    ID          string    // Unique identifier
    Content    string    // Knowledge content
    Tags       []string  // Categorization tags
    Type       string    // fact, insight, pattern, bug, feature, etc.
    Confidence float64   // 0.0-1.0 confidence score
    Workspace  string    // Workspace context
    Agent      string    // Creating agent
    CreatedAt  time.Time // Creation timestamp
    UpdatedAt  time.Time // Last update timestamp
    Vector     []float64 // Embedding vector (Qdrant)
    SupersededBy string  // Supersession chain link
}
```

#### Storage Backend:
- **Postgres** — Primary relational storage for metadata
- **Qdrant** — Vector database for semantic search
- **Ollama** — Embedding generation (homelab stack)

#### Client Architecture:

```
pkg/brain/
├── brain.go           # Main service + MCP tool registration
├── bridge_events.go   # Event bridging between brain and MCP
├── direct.go          # Direct brain operations (non-MCP)
├── provider.go        # Brain provider for service integration
├── tools.go           # MCP tool implementations
└── client/
    ├── client.go      # Brain client with connection pooling
    └── coreio_compat.go # CoreIO compatibility layer
```

### 3. Lemma Service (`pkg/lemma/`)

**Local LEM engine client** (lthn-mlx / lthn-ai driver).

#### Capabilities:
- Chat sessions with local models
- `/v1/admin` control endpoints for model management
- Model profile management (list, load, unload)
- Inference with GPU acceleration (MLX for Apple Silicon, CUDA, ROCm)
- Streaming responses
- Session state persistence

#### Integration:
- Connects to local `lthn-mlx` service
- Manages model lifecycle
- Handles GPU resource allocation
- Provides REST API for chat operations

### 4. Chat History Service (`pkg/chathistory/`)

**Per-user portable DuckDB chat archive** for conversation persistence.

#### Features:
- **Portable** — DuckDB database can be moved between systems
- **Per-user** — Separate database per user/agent
- **Searchable** — Full-text search across conversations
- **Exportable** — Export conversations to JSON/Markdown
- **Versioned** — Schema migrations for backward compatibility

#### Database Schema:

```sql
-- Conversations table
CREATE TABLE conversations (
    id TEXT PRIMARY KEY,
    user_id TEXT NOT NULL,
    model TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    metadata JSON
);

-- Messages table
CREATE TABLE messages (
    id TEXT PRIMARY KEY,
    conversation_id TEXT NOT NULL,
    role TEXT NOT NULL,  -- 'user' or 'assistant'
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    token_count INTEGER,
    FOREIGN KEY (conversation_id) REFERENCES conversations(id)
);

-- Indexes for performance
CREATE INDEX idx_messages_conversation ON messages(conversation_id);
CREATE INDEX idx_messages_created ON messages(created_at);
```

#### Export Functionality:

The `export.go` provides:
- `ExportConversation(conversationID, format)` — Export single conversation
- `ExportAll(userID, format)` — Export all user conversations
- Format options: JSON, Markdown, HTML
- Metadata inclusion options

### 5. Monitor Service (`pkg/monitor/`)

**Background monitoring and repository synchronization**.

#### Responsibilities:
- Monitor repository health and status
- Automatic sync from upstream (Forgejo → local)
- Data harvesting from active workspaces
- State tracking for running agents
- Notification on state changes via Core ACTIONs
- Periodic health checks

#### Components:

| File | Purpose |
|------|---------|
| `monitor.go` | Main monitor service with lifecycle |
| `sync.go` | Repository synchronization logic |
| `harvest.go` | Data harvesting from workspaces |
| `register.go` | Service registration and configuration |

#### Monitoring Targets:
- Local repository mirrors (`~/Code/{org}/{repo}`)
- Workspace directories (`~/Lethean/workspace/**`)
- Agent process health
- Container runtime status
- GPU resource utilization

### 6. Runner Service (`pkg/runner/`)

**Local and container process runners** with dispatch queue.

#### Capabilities:
- **Local execution** — Run commands directly on host
- **Container execution** — Run in Docker containers with:
  - Volume mounts for workspace access
  - GPU passthrough
  - Environment variable injection
  - Resource limits
- **Dispatch queue** — Priority-based task queue with:
  - Concurrency control
  - Rate limiting
  - Dependency resolution
  - Result aggregation

#### Path Utilities:
- `paths.go` — Resolve container paths, volume mounts, workspace mappings
- Docker socket communication
- Cross-platform path handling

#### Queue Management:
- Priority levels (high, medium, low)
- Concurrency limits per priority
- Task timeout and retry
- Result collection and error handling

### 7. Setup Service (`pkg/setup/`)

**Project detection and workspace scaffolding**.

#### Capabilities:
- **Project detection** — Identify project type and structure
- **Scaffolding** — Create `.core/` directory structure
- **Configuration** — Generate configuration files
- **Dependency analysis** — Detect and document dependencies
- **Workspace initialization** — Setup initial workspace state

#### Detection:
- Identifies Go modules via `go.mod`
- Detects PHP projects via `composer.json`
- Recognizes CoreGO packages via module path
- Analyzes project structure and dependencies

#### Scaffolding:
Creates the following structure:
```
.core/
├── agents.yaml          # Agent configuration
├── workspace.yaml       # Workspace templates
├── hooks/               # Git hooks
│   ├── pre-commit       # Pre-commit hook
│   └── commit-msg       # Commit message hook
└── reference/           # Reference documentation
    └── docs/            # Copied from org docs repo
```

---

## 📡 MCP Tools

The agent package exposes **50+ MCP tools** across multiple tool groups, organized into subsystems.

### Agentic Tools (Dispatch & Workspace)

**Tool Prefix:** `agentic_`

| Tool | Description | Parameters |
|------|-------------|------------|
| `agentic_dispatch` | Dispatch agent to task | repo, org, task, agent, issue, pr, branch, tag, template, persona, variables |
| `agentic_dispatch_remote` | Dispatch to remote host | host, repo, task, agent, ... |
| `agentic_status` | Get dispatch status | - |
| `agentic_status_remote` | Get remote dispatch status | host |
| `agentic_prep_workspace` | Prepare workspace | repo, org, task, agent, issue, ... |
| `agentic_resume` | Resume previous session | session_id |
| `agentic_watch` | Watch workspace for changes | path, recursive |

### Plan Management Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `agentic_plan_create` | Create new plan | title, description, phases, dependencies |
| `agentic_plan_read` | Read plan details | plan_id |
| `agentic_plan_update` | Update plan | plan_id, title, description, phases, status |
| `agentic_plan_delete` | Delete plan | plan_id |
| `agentic_plan_list` | List all plans | status, limit, offset |

### Phase Management Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `agentic_phase_create` | Create phase in plan | plan_id, title, description, tasks |
| `agentic_phase_update` | Update phase | plan_id, phase_id, changes |
| `agentic_phase_complete` | Mark phase complete | plan_id, phase_id |

### Epic & Issue Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `agentic_create_epic` | Create epic | title, description, issues, labels |
| `agentic_list_epics` | List epics | filter, limit |
| `agentic_issue_create` | Create issue | repo, title, description, labels, priority |
| `agentic_issue_list` | List issues | repo, filter, limit |

### PR & Review Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `agentic_create_pr` | Create pull request | repo, title, description, branch, target |
| `agentic_list_prs` | List pull requests | repo, filter, limit |
| `agentic_create_epic` | Create epic | title, description, issues |
| `agentic_review_queue` | Get review queue | - |

### Mirror & Scan Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `agentic_mirror` | Mirror repository | source, target, force |
| `agentic_scan` | Scan repository for issues | repo, depth, patterns |
| `agentic_scan_remote` | Remote scan | host, repo, depth |

### Brain Tools

**Tool Prefix:** `brain_`

| Tool | Description | Parameters |
|------|-------------|------------|
| `brain_recall` | Recall from semantic memory | query, tags, type, limit, confidence |
| `brain_remember` | Store in brain | content, tags, type, metadata, workspace |
| `brain_forget` | Remove from brain | id, workspace |
| `brain_search` | Semantic search | query, tags, limit |
| `brain_list` | List brain entries | tags, type, limit, offset |

### Messaging Tools

**Tool Prefix:** `agent_`

| Tool | Description | Parameters |
|------|-------------|------------|
| `agent_send` | Send message to agent | agent, message, channel |
| `agent_inbox` | Get agent inbox | agent, limit |
| `agent_conversation` | Get conversation | conversation_id, limit |
| `agent_conversations` | List conversations | agent, limit |

### File Tools

**Tool Prefix:** `file_`

| Tool | Description | Parameters |
|------|-------------|------------|
| `file_read` | Read file contents | path |
| `file_write` | Write file contents | path, content |
| `file_edit` | Edit file (replace) | path, old, new |
| `file_delete` | Delete file | path |
| `file_rename` | Rename/move file | from, to |
| `file_exists` | Check file existence | path |
| `file_stat` | Get file stats | path |

### Directory Tools

**Tool Prefix:** `dir_`

| Tool | Description | Parameters |
|------|-------------|------------|
| `dir_list` | List directory contents | path, recursive, pattern |
| `dir_create` | Create directory | path, parents |
| `dir_delete` | Delete directory | path, recursive |
| `dir_exists` | Check directory existence | path |

### Language Tools

**Tool Prefix:** `lang_`

| Tool | Description | Parameters |
|------|-------------|------------|
| `lang_detect` | Detect file language | path |
| `lang_list` | List supported languages | - |

---

## 🎯 Domain Model

### Struct Types

| Type | Package | Description |
|------|---------|-------------|
| `AgentPlan` | agentic | Structured work plan with phases |
| `AgentPhase` | agentic | Phase within a plan with tasks |
| `AgentSession` | agentic | Agent work session with context |
| `AgentMessage` | messages | Direct agent-to-agent message |
| `BrainMemory` | brain | Semantic knowledge entry |
| `DispatchOptions` | agentic | Options for agent dispatch |
| `PrepInput` | agentic | Input for workspace preparation |
| `PrepOutput` | agentic | Output from workspace preparation |
| `WorkspaceState` | agentic | Typed key-value state per plan |
| `Sandbox` | agentic | Running opencode container state |

### Status Enums

**AgentPlan Status:**
- `draft` — Plan is being created
- `active` — Plan is ready for work
- `in_progress` — Plan is being worked on
- `needs_verification` — Plan requires verification
- `verified` — Plan has been verified
- `completed` — Plan is complete
- `archived` — Plan is archived (soft-deleted)

**AgentPhase Status:**
- `pending` — Phase not started
- `in_progress` — Phase being worked on
- `blocked` — Phase is blocked
- `completed` — Phase is complete
- `skipped` — Phase was skipped

**Sandbox Status:**
- `pending` — Container not yet created
- `creating` — Container being created
- `running` — Container is running
- `stopped` — Container is stopped
- `failed` — Container creation failed

---

## 🌐 Provider Integrations

### Claude Code Provider (`provider/claude/`)

**8 Marketplace Plugins:**

| Plugin | ID | Description |
|--------|----|-------------|
| Core | `core` | Core capabilities for the Core ecosystem |
| Core-Go | `core-go` | Core Go integration and tools |
| Core-PHP | `core-php` | Core PHP Laravel integration |
| DevOps | `devops` | DevOps automation tools |
| Infra | `infra` | Infrastructure management |
| Research | `research` | Research and analysis tools |
| Hermes Runner MCP | `hermes_runner_mcp` | Hermes runner as MCP server |
| Camofox MCP | `camofox_mcp` | Camofox integration |

**Plugin Structure:**
```
provider/claude/
├── .claude-plugin/
│   └── marketplace.json          # Plugin manifest for marketplace
├── core/
│   ├── mcp.json                 # MCP server configuration
│   ├── hooks.json               # Git hooks configuration
│   ├── agents/                  # Agent definitions
│   │   ├── agent-task-code-review.json
│   │   └── agent-task-code-simplifier.json
│   └── commands/                # Command definitions
│       ├── dispatch.json
│       ├── status.json
│       ├── review.json
│       ├── recall.json
│       ├── remember.json
│       └── scan.json
├── skills/                     # Skill definitions
│   ├── security-review.md
│   ├── architecture-review.md
│   ├── test-review.md
│   └── ...
└── README.md
```

**Marketplace Integration:**
```bash
# Add repository as plugin source
claude plugin marketplace add https://github.com/dappcore/agent

# Install core-agent plugin
claude plugin install core-agent
```

### Codex Provider (`provider/codex/`)

**Codex CLI integration** for autonomous coding.

**Features:**
- Autonomous task execution
- Code generation and modification
- Multi-file operations
- Error recovery and retry

**Configuration:**
```yaml
# .core/agents.yaml
providers:
  codex:
    enabled: true
    api_key: ${CODEX_API_KEY}
    default_model: codex-pro
    quotas:
      daily: 500
      hourly: 50
```

### Hermes Provider (`provider/hermes/`)

**Hermes plugin** for homelab agent integration.

**Features:**
- Chat interface at `chat.lthn.sh`
- Skill-based task execution
- Custom tool support
- WebSocket-based communication

### Google Provider (`provider/google/`)

**Google Gemini integration** scaffolding for future development.

---

## 📁 Configuration

### Environment Variables

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| `CORE_WORKSPACE` | Workspace root override | `~/Lethean/workspace` | `/srv/agent/workspace` |
| `CORE_HOME` | Core home directory | `~/Code/.core` | `/opt/core` |
| `AGENT_NAME` | Agent name | `cladius` / `charon` | `purberus` |
| `GITHUB_ORG` | GitHub organization | `dAppCore` | `my-org` |
| `LETHEAN_HOME` | Lethean home | `~/Lethean` | `/opt/lethean` |
| `MCP_ADDR` | MCP TCP address | - | `tcp://localhost:8080` |
| `MCP_HTTP_ADDR` | MCP HTTP address | - | `localhost:8081` |
| `MCP_AUTH_TOKEN` | MCP Bearer token | - | `secret-token` |
| `MCP_UNIX_SOCKET` | MCP Unix socket | - | `/tmp/mcp.sock` |
| `DIR_HOME` | Home directory fallback | - | `/home/user` |

### Configuration Files

#### `~/.Lethean/conf/agents.yaml`

```yaml
# Agent Configuration
---

# Global settings
concurrency: 10              # Max concurrent agents
default_agent: claude:sonnet  # Default agent for new tasks
container_runtime: docker    # Container runtime (docker, podman)
image: ghcr.io/dappcore/agent-worker:latest

# Container configuration
container:
  timeout: 3600             # 1 hour default timeout
  memory_limit: 8G         # Memory limit
  cpu_limit: 4            # CPU limit
  
# GPU configuration
gpu:
  enabled: true
  devices: ["all"]
  passthrough: true
  
# Workspace configuration
workspace:
  root: "~/Lethean/workspace"
  cleanup: true            # Cleanup old workspaces
  retention_days: 30       # Keep workspaces for 30 days
  
# Per-provider quotas
quotas:
  claude:
    daily: 1000
    hourly: 100
    rate_limit: 10        # Requests per minute
  codex:
    daily: 500
    hourly: 50
  hermes:
    daily: 1000
    hourly: 200
  
# Provider-specific settings
providers:
  claude:
    enabled: true
    plugins:
      - core
      - core-go
      - core-php
      - devops
      - infra
  codex:
    enabled: true
    api_key: "${CODEX_API_KEY}"
  hermes:
    enabled: true
    endpoint: "wss://chat.lthn.sh"
  google:
    enabled: false
    api_key: "${GOOGLE_API_KEY}"

# Notification settings
notifications:
  slack:
    enabled: false
    webhook: "${SLACK_WEBHOOK}"
  discord:
    enabled: false
    webhook: "${DISCORD_WEBHOOK}"
```

#### `~/.Lethean/data/workspace.yaml`

```yaml
# Workspace Templates
---

templates:
  default:
    description: "Default workspace template for CoreGO packages"
    structure:
      - repo/                           # Repository clone
      - specs/                         # Spec tree (RFCs)
      - .meta/                        # Metadata and state
      - .core/                        # Core configuration
      │   └── reference/docs/         # Reference documentation
    prompts:
      - name: coding
        template: pkg/lib/prompt/coding.md
      - name: review
        template: pkg/lib/prompt/review.md
      - name: security
        template: pkg/lib/prompt/security.md
      - name: verify
        template: pkg/lib/prompt/verify.md
      - name: simplify
        template: pkg/lib/prompt/simplify.md
    
  go-package:
    extends: default
    structure:
      - go/
      - go.mod
      - go.sum
    prompts:
      - name: go-coding
        template: pkg/lib/prompt/go/coding.md
      - name: go-review
        template: pkg/lib/prompt/go/review.md
    
  php-package:
    extends: default
    structure:
      - src/
      - composer.json
    prompts:
      - name: php-coding
        template: pkg/lib/prompt/php/coding.md
```

#### `.core/agents.yaml` (Repository-level)

```yaml
# Repository-specific agent configuration
---

# Override global settings for this repository
concurrency: 5

# Repository-specific providers
disabled_providers:
  - codex:review  # Don't use codex:review for this repo

# Custom personas
personas:
  maintainer:
    description: "Senior maintainer for this repository"
    prompt: pkg/lib/persona/maintainer.md
    
# Custom templates
templates:
  rfc-implementation:
    description: "Template for RFC implementation tasks"
    prompt: pkg/lib/prompt/rfc-implementation.md
    workflow: pkg/lib/flow/rfc-implementation.yaml
```

---

## 🚀 Quick Start Examples

### Basic Usage

```bash
# Clone and build
cd ~/Code/core/agent
git pull origin dev
cd go
go build -o core-agent ./cmd/core-agent/

# Run as MCP server (Claude Code)
./core-agent mcp

# Run as HTTP daemon
MCP_HTTP_ADDR=localhost:8081 \
  MCP_AUTH_TOKEN=your-secret \
  ./core-agent serve

# Run hub control plane
./core-agent hub --http 127.0.0.1:9201 --mcp-http 127.0.0.1:9202

# Start chat session
./core-agent chat --user=purberus

# List available models
./core-agent serve-profiles

# Check engine status
./core-agent serve-status
```

### Go Module Integration

```go
package main

import (
    "context"
    "signal"
    "syscall"
    
    core "dappco.re/go"
    "dappco.re/go/agent/pkg/agentic"
    "dappco.re/go/agent/pkg/brain"
    "dappco.re/go/agent/pkg/monitor"
)

func main() {
    // Create Core application with agent services
    app, result := core.New(
        core.WithService(agentic.Register),
        core.WithService(brain.Register),
        core.WithService(monitor.Register),
    )
    if !result.OK {
        panic(result.Value)
    }
    
    ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
    defer stop()
    
    // Run the application
    if err := app.Run(ctx); err != nil {
        core.Error("agent failed", "err", err)
        core.Exit(1)
    }
}
```

### Dispatch Example

```go
package main

import (
    "context"
    "fmt"
    
    "dappco.re/go/agent/pkg/agentic"
)

func main() {
    ctx := context.Background()
    
    // Dispatch a task to an agent
    result := agentic.Dispatch(ctx, agentic.DispatchOptions{
        Repo:    "go-io",
        Org:     "dAppCore",
        Task:    "add-s3-backend",
        Agent:   "claude:sonnet",
        Issue:   123,
        Template: "RFC.md",
        Persona:  "senior-dev",
        Variables: map[string]string{
            "backend": "s3",
            "priority": "high",
        },
    })
    
    if result.OK {
        workspaceDir := result.Value.(string)
        fmt.Printf("Dispatched to workspace: %s\n", workspaceDir)
    } else {
        fmt.Printf("Dispatch failed: %v\n", result.Error)
    }
}
```

### Workspace Preparation

```go
package main

import (
    "context"
    "fmt"
    
    "dappco.re/go/agent/pkg/agentic"
)

func main() {
    ctx := context.Background()
    
    // Prepare a workspace
    prepResult := agentic.PrepWorkspace(ctx, agentic.PrepInput{
        Repo:    "go-process",
        Org:     "dAppCore",
        Task:    "windows-support",
        Agent:   "claude:opus",
        Issue:   456,
        PR:      789,
        Branch:  "dev",
        Tag:     "",
        Template: "RFC.md",
        Persona:  "architect",
        Variables: map[string]string{
            "target": "windows",
            "phase":  "1",
        },
        DryRun: false,
    })
    
    if prepResult.OK {
        output := prepResult.Value.(agentic.PrepOutput)
        fmt.Printf("Workspace: %s\n", output.WorkspaceDir)
        fmt.Printf("Repo: %s\n", output.RepoDir)
        fmt.Printf("Branch: %s\n", output.Branch)
        fmt.Printf("Prompt: %s\n", output.Prompt)
    }
}
```

### Brain Operations

```go
package main

import (
    "context"
    "fmt"
    
    "dappco.re/go/agent/pkg/brain"
)

func main() {
    ctx := context.Background()
    
    // Store knowledge in brain
    err := brain.Remember(ctx, brain.Memory{
        Content:    "The go-process package uses RingBuffer with 1MB default size for output capture",
        Tags:       []string{"go-process", "architecture", "buffer", "output"},
        Type:       "fact",
        Confidence:  0.95,
        Workspace:   "core/go-process",
        Agent:      "purberus",
    })
    if err != nil {
        panic(err)
    }
    
    // Recall from brain
    results, err := brain.Recall(ctx, brain.Query{
        Query:  "output capture",
        Tags:   []string{"go-process"},
        Type:   "fact",
        Limit:  5,
        MinConfidence: 0.8,
    })
    if err != nil {
        panic(err)
    }
    
    for i, result := range results {
        fmt.Printf("Result %d (confidence: %.2f):\n", i+1, result.Confidence)
        fmt.Printf("  Tags: %v\n", result.Tags)
        fmt.Printf("  Content: %s\n\n", result.Content)
    }
}
```

### MCP Server Integration

```go
package main

import (
    "context"
    "signal"
    "syscall"
    
    core "dappco.re/go"
    "dappco.re/go/agent/pkg/agentic"
    coremcp "dappco.re/go/mcp/pkg/mcp"
)

func main() {
    ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
    defer stop()
    
    // Initialize Core with agent services
    app, _ := core.New(
        core.WithService(agentic.Register),
        core.WithService(coremcp.Register),
    )
    
    // Configure MCP
    os.Setenv("MCP_HTTP_ADDR", "localhost:8081")
    os.Setenv("MCP_AUTH_TOKEN", "secret-token")
    
    // Run the application
    app.Run(ctx)
}
```

---

## 🧪 Testing

### Test Structure

The package follows the **AX Standard** with complete test triplet coverage:

- `*_test.go` — Unit and integration tests
- `*_example_test.go` — Usage examples as tests

**Naming Convention:**
- `TestGood*` — Happy path tests
- `TestBad*` — Expected error conditions
- `TestUgly*` — Panics and edge cases

### Running Tests

```bash
# All tests
cd ~/Code/core/agent/go
go test -v ./...

# With coverage
go test -cover ./...

# With race detector
go test -race ./...

# Benchmarks
go test -bench . -benchmem

# Specific packages
go test -v ./pkg/agentic/...
go test -v ./pkg/brain/...
go test -v ./pkg/chathistory/...
go test -v ./pkg/monitor/...

# Isolated module verification (no workspace)
GOWORK=off go test ./...

# With timeout (recommended for slow tests)
go test -timeout 60s ./...

# Short mode (skip long-running tests)
go test -short ./...
```

### Test Coverage Areas

All major components have comprehensive test coverage:

- ✅ **Agentic Service** — Dispatch, prep, verify, scan, remote, mirror, plans
- ✅ **Brain Service** — Recall, remember, forget, search, list, messaging
- ✅ **Chat History** — Store, retrieve, search, export, migrations
- ✅ **Monitor Service** — Sync, harvest, state tracking, notifications
- ✅ **Runner Service** — Local execution, container execution, queue management
- ✅ **Setup Service** — Project detection, scaffolding, configuration
- ✅ **MCP Integration** — All tool registrations and transports
- ✅ **Provider Integrations** — Claude, Codex, Hermes, Google
- ✅ **Path Utilities** — All path helpers and overrides
- ✅ **Messages** — IPC message serialization and deserialization

### Example Test Patterns

```go
// Good path test
func TestGoodDispatch(t *testing.T) {
    ctx := context.Background()
    result := agentic.Dispatch(ctx, agentic.DispatchOptions{
        Repo: "test-repo",
        Org:  "test-org",
        Task: "test-task",
    })
    assert.True(t, result.OK)
    assert.NotEmpty(t, result.Value)
}

// Bad path test (expected error)
func TestBadDispatch(t *testing.T) {
    ctx := context.Background()
    result := agentic.Dispatch(ctx, agentic.DispatchOptions{
        Repo: "", // Empty repo should fail
    })
    assert.False(t, result.OK)
    assert.NotNil(t, result.Error)
}

// Ugly path test (panic recovery)
func TestUglyDispatch(t *testing.T) {
    ctx := context.Background()
    // Setup conditions that would cause panic
    defer func() {
        if r := recover(); r != nil {
            assert.NotNil(t, r)
        }
    }()
    // Trigger panic scenario
    agentic.Dispatch(ctx, agentic.DispatchOptions{
        Repo: "invalid-repo",
        // ... conditions that cause panic
    })
}
```

---

## 🌐 API Endpoints

### Hub Control Plane API

**Base URL:** `http://127.0.0.1:9201` (configurable via `--http` flag)

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/api/agent/status` | Get overall agent status | Bearer |
| GET | `/api/agent/dispatch` | List active dispatches | Bearer |
| POST | `/api/agent/dispatch` | Create new dispatch | Bearer |
| GET | `/api/agent/dispatch/{id}` | Get dispatch details | Bearer |
| POST | `/api/agent/dispatch/{id}/cancel` | Cancel dispatch | Bearer |
| GET | `/api/agent/brain/recall` | Recall from brain | Bearer |
| POST | `/api/agent/brain/remember` | Store in brain | Bearer |
| GET | `/api/agent/brain/list` | List brain entries | Bearer |
| GET | `/api/agent/workspaces` | List workspaces | Bearer |
| GET | `/api/agent/workspaces/{name}` | Get workspace details | Bearer |
| GET | `/api/agent/models` | List available models | Bearer |
| GET | `/api/agent/models/{name}` | Get model details | Bearer |
| POST | `/api/agent/models/{name}/load` | Load model | Bearer |
| POST | `/api/agent/models/{name}/unload` | Unload model | Bearer |

### MCP HTTP API

**Base URL:** `http://127.0.0.1:9202` (configurable via `--mcp-http` flag)

Follows the [Model Context Protocol specification](https://modelcontextprotocol.io):

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/mcp/initialize` | Initialize MCP session |
| POST | `/mcp/tools/list` | List available tools |
| POST | `/mcp/tools/call` | Call a tool |
| POST | `/mcp/resources/list` | List available resources |
| POST | `/mcp/resources/read` | Read a resource |
| POST | `/mcp/sampling/createMessage` | Create message (streaming) |

### Response Formats

All endpoints return JSON responses:

**Success Response:**
```json
{
    "ok": true,
    "value": { ... },
    "duration": "123.456ms"
}
```

**Error Response:**
```json
{
    "ok": false,
    "error": "detailed error message",
    "code": "error_code",
    "duration": "123.456ms"
}
```

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| **Total Go files** | ~400+ |
| **Test files** | ~300+ |
| **Example test files** | ~150+ |
| **Lines of code** | ~100,000+ |
| **Test coverage** | High |
| **Binary size** | ~50MB |
| **Subsystems** | 7 (agentic, brain, lemma, chathistory, monitor, runner, setup) |
| **MCP tools** | 50+ |
| **Provider integrations** | 4 (Claude, Codex, Hermes, Google) |
| **MCP transports** | 4 (stdio, tcp, http, unix) |
| **Binary modes** | 8 (mcp, serve, hub, chat, serve-status, serve-reload, serve-profiles, models-download, models-job, run flow) |
| **Configuration files** | 2 (agents.yaml, workspace.yaml) |
| **Environment variables** | 15+ |
| **Agent types** | 9 |
| **Personas** | 50+ (embedded in pkg/lib/persona/) |
| **Prompt templates** | 20+ (embedded in pkg/lib/prompt/) |
| **Flow templates** | 10+ (embedded in pkg/lib/flow/) |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-mcp](../mcp/) | MCP server framework (foundation for agent's MCP mode) | [../mcp/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/mcp/) |
| [go-process](../process/) | Process orchestration (used by runner service) | [../process/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/process/) |
| [go-io](../io/) | I/O abstraction layer (used throughout) | [../io/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/io/) |
| [go-api](../api/) | REST framework (used by hub control plane) | [../api/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/api/) |
| [go-ws](../ws/) | WebSocket hub (used for real-time events) | [../ws/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/ws/) |
| [go-cache](../cache/) | Caching layer (used for model caching) | [../cache/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/cache/) |
| [go-session](../session/) | Session parsing (used for chat history) | [../session/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/session/) |
| [go-webview](../webview/) | WebView automation (used for browser tasks) | [../webview/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/webview/) |
| [CoreGO INDEX](../../INDEX.md) | Complete package catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## 📈 Quality Metrics

- ✅ **MCP Specification Compliance** — Full Model Context Protocol v1.0 implementation
- ✅ **Test Coverage** — Good/Bad/Ugly pattern for all scenarios (100% of public APIs)
- ✅ **Transport Flexibility** — Multiple transport backends (stdio, tcp, http, unix)
- ✅ **Security** — Workspace sandboxing, Bearer authentication, input sanitization
- ✅ **Extensibility** — Subsystem architecture for custom tools and providers
- ✅ **Documentation** — Complete README + INDEX + inline documentation
- ✅ **Core Integration** — Full CoreGO framework support (ServiceRuntime, ACTION, Result)
- ✅ **Cross-Platform** — Unix + Windows support (where applicable)
- ✅ **Production Ready** — Deployed in production environments
- ✅ **AX Standard Compliance** — Single Point Of Responsibility for all types

---

## 📝 Changelog

| Date | Change | Commit | Author |
|------|--------|--------|--------|
| 2026-06-17 | Complete knowledge pack documentation | N/A | Purberus |
| 2026-05-XX | Brain subsystem finalized with full MCP integration | N/A | Cladius |
| 2026-04-30 | CLAUDE.md guidance created with sprint intel collection | N/A | Cladius |
| 2026-04-XX | Opencode integration added with sandboxed host support | N/A | Cladius |
| 2026-03-XX | Initial package creation with basic dispatch capabilities | N/A | Cladius |
| 2026-02-XX | Repository structure established (go/, php/, provider/) | N/A | Cladius |

---

## 🎯 Tags

```yaml
# Core Capabilities
- ai-agent
- orchestration
- dispatch
- mcp-server
- model-context-protocol
- agent-dispatch
- workspace-management
- sandboxing
- container
- docker

# Providers
- claude-code
- codex
- gemini
- google-gemini
- hermes
- opencode

# Services
- brain
- openbrain
- semantic-memory
- chat-archive
- duckdb
- monitoring
- fleet-management
- repository-sync

# Integrations
- github
- forgejo
- huggingface
- ollama
- postgresql
- qdrant

# Features
- pull-request
- code-review
- issue-tracking
- epic-management
- plan-management
- persona
- prompt-engineering
- flow-automation
- yml-workflows

# Architecture
- microservices
- event-driven
- pubsub
- channel-events
- rest-api
- websocket
- stdio
- tcp
- unix-socket

# Quality
- production-ready
- high-coverage
- extensible
- testable
- observable
- secure
- documented

# Ecosystem
- corego
- action-system
- service-runtime
- result-pattern
- xor-pattern
- agent-persona
- lthn
- lethean
- dappcore
```

---

## 📚 References

### Internal Documentation

1. **RFC Specification** — [plans/code/core/go/agent/RFC.md](../../../../../plans/code/core/go/agent/RFC.md)
   - Complete contract for all subsystems
   - Present-tense specification
2. **Repository** — [~/Code/core/agent/](file:///Users/snider/Code/core/agent/)
   - Full source code
   - Examples and tests
3. **CLAUDE.md** — [~/Code/core/agent/CLAUDE.md](file:///Users/snider/Code/core/agent/CLAUDE.md)
   - Development guidance
   - Sprint intel collection
   - Coding standards
4. **GOAL.md** — [~/Code/core/agent/GOAL.md](file:///Users/snider/Code/core/agent/GOAL.md)
   - Implementation goals
   - Parity gate specification
5. **AGENTS.md** — [~/Code/core/agent/AGENTS.md](file:///Users/snider/Code/core/agent/AGENTS.md)
   - Agent guidance
   - Best practices
6. **Repository README** — [~/Code/core/agent/README.md](file:///Users/snider/Code/core/agent/README.md)
   - Quickstart guide
   - Build and test instructions

### External References

1. **Model Context Protocol** — [modelcontextprotocol.io](https://modelcontextprotocol.io)
   - Official MCP specification
   - SDK documentation
2. **Claude Code** — [claude.ai](https://claude.ai)
   - Claude Code documentation
   - Plugin marketplace
3. **Codex** — [codex.ai](https://codex.ai)
   - Codex CLI documentation
4. **Gemini** — [deepmind.google](https://deepmind.google)
   - Google Gemini documentation
5. **DuckDB** — [duckdb.org](https://duckdb.org)
   - Embedded database documentation
6. **Qdrant** — [qdrant.tech](https://qdrant.tech)
   - Vector database documentation
7. **Lethean Project** — [docs.lthn.io](https://docs.lthn.io)
   - Project documentation
   - Architecture overviews

---

*Package index generated: 2026-06-17T20:00:00Z*
*Knowledge Pack: CoreGo v1.2.0*
*Repository: dappco.re/go/agent*
*Maintainer: Purberus <purberus@lthn.ai>*
