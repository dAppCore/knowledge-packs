---
type: Package Documentation
package: agent
module: dappco.re/go/agent
repo: core/agent
lang: go
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
  - container
  - chat-archive
  - monitoring
---
# go-agent — AI agent orchestration platform

**RFC:** [plans/code/core/go/agent/RFC.md](../../../../../plans/code/core/go/agent/RFC.md)
**Source:** [~/Code/core/agent/](file:///Users/snider/Code/core/agent/)
**Module:** `dappco.re/go/agent`
**Binary:** `core-agent` / `lthn-agent`
**Dependencies:** `dappco.re/go` (CoreGO framework)
**Status:** ✅ Production-Ready
**License:** EUPL-1.2

---

## Overview

`go-agent` is the AI agent orchestration platform for the Core ecosystem, providing a single Go binary (`core-agent` or `lthn-agent`) that runs as an MCP server and CLI tool. It dispatches AI coding agents (Claude, Codex, Gemini, opencode) into sandboxed containers, runs an opencode-backed agent fleet, serves an MCP + hub control plane, and carries shared semantic memory (OpenBrain).

### Primary use cases

1. **Agent Dispatch** — Fan out tasks to sandboxed workers (Claude, Codex, Hermes, Google) running in `.core/workspace/`
2. **MCP Server** — Model Context Protocol server for Claude Code, Cursor, and other IDE integrations
3. **HTTP Daemon** — Cross-agent communication via HTTP MCP for CI and remote use
4. **Fleet Management** — Pull/merge/push across Core ecosystem repos via `agents.yaml`
5. **OpenBrain Integration** — Durable memory and cross-agent messaging via Postgres + Qdrant + Ollama
6. **Provider Integrations** — First-class support for Claude Code, Codex, Hermes, and Google Gemini
7. **Hub Control Plane** — Loopback control plane with Bearer auth for opencode control/proxy groups
8. **Chat Archive** — Per-user portable DuckDB chat archive for conversation history
9. **Container Runtime** — Local and container-based runners with dispatch queue

### Design philosophy

- **Single Binary** — One binary (`core-agent`) with multiple modes (mcp, serve, hub, chat)
- **Core-Native** — Full integration with CoreGO framework (ServiceRuntime, ACTION system, Result pattern)
- **Provider-Agnostic** — Abstract provider differences behind common interfaces
- **Sandboxed** — All agent work runs in isolated workspaces with restricted access
- **Observable** — Rich event streaming via Core ACTIONs and channel events
- **Testable** — Complete test triplet coverage (_test.go + _example_test.go)
- **Extensible** — Subsystem architecture for custom tool groups

---

## Architecture

### Component stack

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Application Layer                                      │
│  core-agent binary: mcp, serve, hub, chat, models-download, run flow       │
├─────────────────────────────────────────────────────────────────────────────┤
│                        Service Layer                                           │
│  Agentic Service — Dispatch, prep, verify, scan, remote, mirror, plans       │
│  Brain Service — OpenBrain client (recall, remember, forget, list, msg)    │
│  Lemma Service — Local lthn-mlx client (chat sessions + /v1/admin control) │
│  Monitor Service — Background monitoring + repo sync                         │
│  Runner Service — Local + container runners + dispatch queue                │
│  Setup Service — Project detection + .core/ scaffolding                      │
│  MCP Service — Model Context Protocol server framework                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                        Package Layer                                           │
│  pkg/agentic/ — Dispatch, workspace, plans/phases/sessions, fleet sync       │
│  pkg/brain/ — OpenBrain client with semantic memory operations              │
│  pkg/lemma/ — Local ML inference client (lthn-mlx)                            │
│  pkg/chathistory/ — Portable DuckDB chat archive                              │
│  pkg/monitor/ — Background monitoring + repository synchronization           │
│  pkg/runner/ — Process runners (local + container) + dispatch queue         │
│  pkg/setup/ — Project detection + workspace scaffolding                      │
│  pkg/messages/ — Typed IPC message definitions                               │
│  pkg/agentcompat/ — Agent compatibility utilities                            │
│  pkg/opencode/ — OpenCode integration (sandboxed host)                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                        Provider Layer                                          │
│  provider/claude/ — Claude Code plugin (8 marketplace plugins)               │
│  provider/codex/ — Codex CLI integration                                        │
│  provider/hermes/ — Hermes plugin sources + skills                              │
│  provider/google/ — Google Gemini integration scaffold                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                        Data Layer                                              │
│  .core/agents.yaml — Runtime configuration                                    │
│  .core/workspace.yaml — Workspace templates                                   │
│  ~/Lethean/workspace/ — Task workspaces                                       │
│  ~/Lethean/data/ — Persistent storage (DuckDB chat archives)                  │
│  ~/Lethean/conf/ — Configuration files                                        │
│  ~/Lethean/log/ — Agent logs                                                  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Binary modes

The `core-agent` binary supports multiple operation modes:

| Mode | Description | Use Case |
|------|-------------|----------|
| `mcp` | Stdio MCP server for coding-agent hosts | Claude Code integration (default) |
| `serve` | HTTP MCP daemon | Cross-agent communication, CI, remote use |
| `hub` | Loopback control plane with Bearer auth | Opencode control/proxy groups + brain |
| `chat --user=<id>` | REPL against local LEM engine (lthn-mlx) | Interactive chat with auto-capture to archive |
| `serve-status` | Inspect local model engine | Check running model profiles |
| `serve-reload` | Hot-swap local model engine | Reload model configurations |
| `serve-profiles` | List model engine profiles | View available models |
| `models-download` | Queue Hugging Face model downloads | Download new models |
| `models-job` | Poll model download status | Check download progress |
| `run flow <path>` | Execute YAML workflow | Run predefined workflows |

### Core integration

The package integrates with CoreGO via:
- **ServiceRuntime** — Standard Core service lifecycle for all subsystems
- **ACTION system** — Rich event broadcasting for process and agent events
- **Result pattern** — All operations return `core.Result` with panic recovery
- **IPC bus** — Internal communication via Core ACTIONs
- **Path helpers** — Shared path utilities in `pkg/agentic/paths.go`

---

## Package structure

### Repository layout

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
│   │   ├── main_test.go               # Main tests
│   │   ├── main_example_test.go       # Main usage examples
│   │   ├── update.go                  # Update functionality
│   │   └── update_*_test.go           # Update tests
│   │
│   ├── pkg/
│   │   ├── agentcompat/               # Agent compatibility utilities
│   │   │   ├── agentcompat.go         # Compatibility layer
│   │   │   └── agentcompat_*_test.go  # Compatibility tests
│   │   │
│   │   ├── agentic/                  # **Core agentic orchestration**
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
│   │   │   ├── brain_client.go        # Brain client
│   │   │   ├── provider_manager.go    # Provider management
│   │   │   └── sanitise.go            # Input sanitization
│   │   │
│   │   │   └── .core/                 # Runtime state
│   │   │       └── state/              # State management
│   │   │
│   │   ├── audit/                     # Audit logging
│   │   │   └── audit.go               # Audit trail
│   │   │
│   │   ├── brain/                     # **OpenBrain integration**
│   │   │   ├── brain.go               # Main brain client
│   │   │   ├── bridge_events.go       # Bridge event handling
│   │   │   ├── client/                # Brain client
│   │   │   │   ├── client.go          # Client implementation
│   │   │   │   └── coreio_compat.go   # CoreIO compatibility
│   │   │   ├── direct.go             # Direct operations
│   │   │   ├── provider.go           # Brain provider
│   │   │   └── tools.go              # Brain tools
│   │   │
│   │   ├── chathistory/               # **DuckDB chat archive**
│   │   │   ├── chathistory.go         # Main chat history
│   │   │   ├── export.go              # Export functionality
│   │   │   └── migrations/            # Database migrations
│   │   │
│   │   ├── lemma/                     # **Local ML inference**
│   │   │   └── lemma.go               # LEM (lthn-mlx) client
│   │   │
│   │   ├── lib/                       # **Embedded templates**
│   │   │   ├── flow/                  # Flow templates
│   │   │   │   └── upgrade/           # Upgrade flows
│   │   │   ├── persona/               # Persona definitions
│   │   │   │   ├── code/              # Code personas
│   │   │   │   ├── secops/            # Security personas
│   │   │   │   └── testing/           # Testing personas
│   │   │   ├── prompt/                # Prompt templates
│   │   │   ├── task/                  # Task templates
│   │   │   │   ├── code/              # Code tasks
│   │   │   │   │   ├── review/        # Code review tasks
│   │   │   │   │   └── simplifier/    # Code simplifier tasks
│   │   │   │   └── ...
│   │   │   └── workspace/             # Workspace templates
│   │   │       └── default/          # Default workspace
│   │   │
│   │   ├── messages/                  # **IPC message definitions**
│   │   │   └── messages.go            # Message types
│   │   │
│   │   ├── monitor/                   # **Background monitoring**
│   │   │   ├── monitor.go             # Main monitor
│   │   │   ├── sync.go                # Repository sync
│   │   │   ├── harvest.go             # Data harvesting
│   │   │   ├── register.go            # Monitor registration
│   │   │   └── sync_*_test.go         # Sync tests
│   │   │
│   │   ├── opencode/                  # **OpenCode integration**
│   │   │   ├── internal/             # Internal utilities
│   │   │   │   ├── paths/            # Path utilities
│   │   │   │   └── sigkeys/           # Signature keys
│   │   │   └── opencode.go            # OpenCode client
│   │   │
│   │   ├── runner/                    # **Process runners**
│   │   │   ├── runner.go              # Main runner
│   │   │   ├── queue.go               # Dispatch queue
│   │   │   └── paths.go               # Runner paths
│   │   │
│   │   └── setup/                     # **Workspace setup**
│   │       ├── config.go             # Setup configuration
│   │       ├── detect.go              # Project detection
│   │       ├── service.go             # Setup service
│   │       └── setup.go               # Main setup
│   │
│   ├── version.go                     # Version information
│   ├── version_*_test.go              # Version tests
│   ├── go.mod
│   ├── go.sum
│   └── go.work
│
├── php/                                # PHP Laravel package
├── provider/                          # Provider integrations
│   ├── claude/                        # Claude Code plugins
│   │   ├── core/                      # Core plugins
│   │   ├── core-go/                   # Core-Go plugin
│   │   ├── core-php/                  # Core-PHP plugin
│   │   ├── devops/                    # DevOps plugin
│   │   ├── infra/                     # Infrastructure plugin
│   │   ├── research/                  # Research plugin
│   │   ├── hermes_runner_mcp/         # Hermes runner MCP
│   │   ├── camofox_mcp/               # Camofox MCP
│   │   └── plugins/                   # Marketplace plugins
│   ├── codex/                         # Codex integration
│   ├── hermes/                        # Hermes integration
│   └── google/                        # Google Gemini integration
│
├── .core/                             # Runtime configuration
│   ├── agents.yaml                    # Agent configuration
│   ├── workspace.yaml                 # Workspace templates
│   └── ...
├── docs/                              # Documentation
├── vm/                                # Virtual machine configurations
├── scripts/                           # Utility scripts
├── README.md                          # Repository README
├── CLAUDE.md                          # Development guidance
├── GOAL.md                           # Implementation goals
├── RFC.md                            # Request for Comments
├── AGENTS.md                         # Agent guidance
├── .mcp.json                          # MCP server configuration
└── Taskfile.yaml                      # Build orchestration
```

---

## Binary modes

### MCP mode (`core-agent mcp`)

The stdio MCP server for coding-agent host integration. This is the default mode for Claude Code.

**Features:**
- Full Model Context Protocol specification compliance
- File operations with workspace sandboxing
- Tool registration for agent capabilities
- Notification broadcasting to connected sessions
- Channel-based event system for cross-component communication

**Integration:**
```go
// Registered automatically via .claude-plugin/marketplace.json
// or manually via:
svc, _ := mcp.New(mcp.Options{WorkspaceRoot: "."})
svc.Run(context.Background())
```

### Serve mode (`core-agent serve`)

HTTP MCP daemon for cross-agent communication, CI, and remote use.

**Features:**
- REST API transport with Bearer authentication
- Multiple concurrent sessions
- Process isolation
- Resource quotas

**Environment Variables:**
- `MCP_HTTP_ADDR` — HTTP transport address (e.g., `localhost:8081`)
- `MCP_AUTH_TOKEN` — Bearer token for authentication

### Hub mode (`core-agent hub`)

Loopback control plane serving opencode control/proxy groups and brain.

**Features:**
- Bearer-authenticated HTTP endpoint at `127.0.0.1:9201`
- Fail-closed MCP HTTP+SSE plane at `127.0.0.1:9202`
- Non-optional audit edge recording every request
- Opencode control and proxy groups
- Brain integration with semantic memory

### Chat mode (`core-agent chat --user=<id>`)

REPL against the local LEM engine (lthn-mlx / lthn-ai driver).

**Features:**
- Interactive chat with local models
- Auto-capture to user's portable DuckDB archive
- Session persistence
- Multi-turn conversations

### Model management

**Commands:**
- `serve-status` — Inspect local model engine status
- `serve-reload` — Hot-swap model engine configurations
- `serve-profiles` — List available model profiles
- `models-download` — Queue Hugging Face model downloads
- `models-job` — Poll download job status

---

## Core services

### Agentic service (`pkg/agentic/`)

The core orchestration engine for agent dispatch and workspace management.

#### Key components:

**Workspace Management (`paths.go`):**
- `WorkspaceRoot()` — Root workspace directory (`~/Lethean/workspace/`)
- `CoreRoot()` — Core data root (`~/Lethean/data/`)
- `LetheanHome()` — Lethean home directory (`~/Lethean/`)
- `AgentName()` — Current agent name (cladius/charon)
- `GitHubOrg()` — Default GitHub organization (dAppCore)
- `PlansRoot()` — Plans directory path
- `WorkspaceStatusPaths()` — All workspace status.json paths
- `WorkspaceStatusPath()` — Status path for specific workspace
- `WorkspaceName()` — Extract name from workspace path
- `WorkspaceRepoDir()` — Repository directory in workspace
- `WorkspaceMetaDir()` — Metadata directory in workspace
- `WorkspaceBlockedPath()` — BLOCKED.md path
- `WorkspaceAnswerPath()` — ANSWER.md path
- `WorkspaceLogFiles()` — Agent log files

**Dispatch Pipeline:**
```
Task → Queue → Concurrency Gate → Rate Gate → Workspace Prep → Container Spawn → Agent Runs → Completion Pipeline
```

**Workspace Preparation (`prep.go`):**
1. Clone repo into `repo/` from local mirror `~/Code/{org}/{repo}`
2. Create working branch `agent/{task-slug}`
3. Clone workspace dependencies
4. Copy spec tree (`plans/.../RFC*.md`) into `specs/`
5. Copy org docs repo into `.core/reference/docs/`
6. Build agent prompt and write snapshot

**Prompt Building:**
- Combines task description, repository context, and agent persona
- Supports prompt templates from `pkg/lib/prompt/`
- Creates immutable prompt versions

**Dispatch Flow:**
```
dispatch → agent works → closeout sequence (review → fix → simplify → re-review)
    → commit → auto PR → inline tests → pass → auto-merge on Forge
    → push to GitHub → CodeRabbit reviews → merge or dispatch fix agent
```

#### Agent types supported:

| Agent | Command | Use Case |
|-------|---------|----------|
| `claude:opus` | Claude Code | Complex coding, architecture |
| `claude:sonnet` | Claude Code | Standard tasks |
| `claude:haiku` | Claude Code | Quick/cheap tasks, discovery |
| `gemini` | Gemini CLI | Fast batch operations |
| `codex` | Codex CLI | Autonomous coding |
| `codex:review` | Codex review | Deep security analysis |
| `coderabbit` | CodeRabbit CLI | Code quality review |
| `opencode` | `opencode run` | Sandboxed agent with local/free-compute models |
| `local` | Codex + ollama bridge | Local OSS model via host `ollama` |

### Brain service (`pkg/brain/`)

OpenBrain integration for semantic memory and cross-agent messaging.

**Features:**
- `brain_recall` — Recall from brain with tags and filters
- `brain_remember` — Store knowledge in brain with metadata
- `brain_forget` — Remove entries from brain
- `brain_search` — Semantic search across knowledge
- `brain_list` — List brain entries
- Messaging between agents
- Supersession chains linking new knowledge to what it replaces

**Data Model:**
- `BrainMemory` — Semantic knowledge entry with tags, type, confidence, vector indexing
- Scoped by workspace and agent
- Persistent storage via Postgres + Qdrant + Ollama (homelab stack)

### Lemma service (`pkg/lemma/`)

Local LEM engine client (lthn-mlx / lthn-ai driver).

**Features:**
- Chat sessions with local models
- `/v1/admin` control endpoints
- Model profile management
- Inference with GPU acceleration

### Chat history service (`pkg/chathistory/`)

Per-user portable DuckDB chat archive.

**Features:**
- Portable DuckDB database per user
- Chat session storage with metadata
- Export functionality for analysis
- Migration support for schema changes

**Database Schema:**
- Conversations with timestamps
- Messages with role (user/assistant)
- Session metadata (model, temperature, etc.)
- Search and filter capabilities

### Monitor service (`pkg/monitor/`)

Background monitoring and repository synchronisation.

**Features:**
- Repository health monitoring
- Automatic sync from upstream
- Data harvesting from workspaces
- State tracking for running agents
- Notification on state changes

**Components:**
- `monitor.go` — Main monitor service
- `sync.go` — Repository synchronization
- `harvest.go` — Data harvesting from workspaces
- `register.go` — Service registration

### Runner service (`pkg/runner/`)

Local and container process runners with dispatch queue.

**Features:**
- Local command execution
- Container-based runners (Docker)
- Dispatch queue with priority
- Concurrency control
- Result collection and aggregation

**Path Utilities:**
- `paths.go` — Path resolution for runners
- Docker volume mounts
- Workspace path mapping

### Setup service (`pkg/setup/`)

Project detection and workspace scaffolding.

**Features:**
- Project type detection
- `.core/` directory scaffolding
- Configuration file generation
- Dependency analysis
- Workspace initialization

---

## MCP tools

The agent package exposes MCP tools across multiple subsystems.

### Agentic tools (dispatch & workspace)

| Tool | Description | Parameters |
|------|-------------|------------|
| `agentic_dispatch` | Dispatch agent to task | repo, org, task, agent, issue, template |
| `agentic_dispatch_remote` | Remote dispatch | host, repo, task, agent |
| `agentic_status` | Get dispatch status | - |
| `agentic_status_remote` | Get remote status | host |
| `agentic_prep_workspace` | Prepare workspace | repo, org, task, agent, issue |
| `agentic_resume` | Resume previous session | session_id |
| `agentic_watch` | Watch workspace | path |

### Plan management tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `agentic_plan_create` | Create new plan | title, description, phases |
| `agentic_plan_read` | Read plan details | plan_id |
| `agentic_plan_update` | Update plan | plan_id, changes |
| `agentic_plan_delete` | Delete plan | plan_id |
| `agentic_plan_list` | List all plans | - |

### PR & review tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `agentic_create_pr` | Create pull request | repo, title, description, branch |
| `agentic_list_prs` | List pull requests | repo, filter |
| `agentic_create_epic` | Create epic | title, description, issues |
| `agentic_review_queue` | Get review queue | - |

### Mirror & scan tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `agentic_mirror` | Mirror repository | source, target |
| `agentic_scan` | Scan repository | repo, depth |

### Brain tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `brain_recall` | Recall from brain | query, tags, limit |
| `brain_remember` | Store in brain | content, tags, metadata |
| `brain_forget` | Remove from brain | id |
| `brain_search` | Search brain | query, limit |
| `brain_list` | List brain entries | tags, limit |

### Messaging tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `agent_send` | Send message to agent | agent, message, channel |
| `agent_inbox` | Get agent inbox | agent |
| `agent_conversation` | Get conversation | conversation_id |

### File tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `file_read` | Read file contents | path |
| `file_write` | Write file contents | path, content |
| `file_edit` | Edit file | path, old, new |
| `file_delete` | Delete file | path |
| `file_rename` | Rename/move file | from, to |
| `file_exists` | Check file existence | path |
| `dir_list` | List directory | path |
| `dir_create` | Create directory | path |

### Language tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `lang_detect` | Detect language | path |
| `lang_list` | List languages | - |

---

## Subsystems

### Agentic subsystem

Dispatch and workflow management.

**Files:**
- `dispatch.go` — Task dispatch logic
- `epic.go` — Epic management
- `issue.go` — Issue tracking
- `mirror.go` — Repository mirroring
- `plan.go` — Plan management
- `prep.go` — PR preparation
- `pr.go` — Pull request management
- `queue.go` — Queue management
- `resume.go` — Session resume
- `review_queue.go` — Review queue
- `scan.go` — Repository scanning
- `status.go` — Status tracking
- `watch.go` — Workspace watching
- `write_atomic.go` — Atomic write operations
- `repo_helpers.go` — Repository helper utilities

### Brain subsystem

OpenBrain integration for semantic memory.

**Files:**
- `brain.go` — Main brain subsystem
- `tools.go` — Brain tools
- `provider.go` — Brain provider
- `direct.go` — Direct operations
- `bridge_events.go` — Bridge event handling
- `client/client.go` — Brain client implementation
- `client/coreio_compat.go` — CoreIO compatibility

### IDE subsystem

IDE bridge for Laravel backend (defined in go-mcp, referenced here).

---

## Provider integrations

### Claude Code provider (`provider/claude/`)

Eight marketplace plugins:
- `core` — Core capabilities
- `core-go` — Core Go integration
- `core-php` — Core PHP integration
- `devops` — DevOps tools
- `infra` — Infrastructure management
- `research` — Research tools
- `hermes_runner_mcp` — Hermes runner MCP
- `camofox_mcp` — Camofox MCP

**Features:**
- MCP server registration via `.claude-plugin/marketplace.json`
- Hooks for inbox notifications, auto-format, debug warnings
- Agents for code review and simplification
- Commands for dispatch, status, review, recall, remember, scan
- Skills for security, architecture, test review

### Codex provider (`provider/codex/`)

Codex CLI integration.

### Hermes provider (`provider/hermes/`)

Hermes plugin with skills for homelab agent.

### Google provider (`provider/google/`)

Google Gemini integration scaffolding.

---

## Configuration

### Environment variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CORE_WORKSPACE` | Workspace root override | `~/Lethean/workspace` |
| `CORE_HOME` | Core home directory | `~/Code/.core` |
| `AGENT_NAME` | Agent name | `cladius` (Snider's Mac) / `charon` |
| `GITHUB_ORG` | GitHub organization | `dAppCore` |
| `LETHEAN_HOME` | Lethean home | `~/Lethean` |
| `MCP_ADDR` | MCP TCP address | - |
| `MCP_HTTP_ADDR` | MCP HTTP address | - |
| `MCP_AUTH_TOKEN` | MCP Bearer token | - |
| `MCP_UNIX_SOCKET` | MCP Unix socket path | - |

### Configuration Files

**`~/.Lethean/conf/agents.yaml`:**
```yaml
# Agent configuration
concurrency: 10
default_agent: claude:sonnet
container_runtime: docker
image: ghcr.io/dappcore/agent-worker:latest

# GPU passthrough
gpu:
  enabled: true
  devices: ["all"]

# Per-provider quotas
quotas:
  claude:
    daily: 1000
    hourly: 100
  codex:
    daily: 500
```

**`~/.Lethean/data/workspace.yaml`:**
```yaml
# Workspace templates
templates:
  default:
    structure:
      - repo/
      - specs/
      - .meta/
      - .core/reference/docs/
    prompts:
      - coding
      - review
      - security
```

---

## Quick start

### Basic usage

```bash
# Build the binary
cd ~/Code/core/agent/go
go build -o core-agent ./cmd/core-agent/

# Run as MCP server (Claude Code)
./core-agent mcp

# Run as HTTP daemon
MCP_HTTP_ADDR=localhost:8081 MCP_AUTH_TOKEN=secret ./core-agent serve

# Run hub control plane
./core-agent hub --http 127.0.0.1:9201 --mcp-http 127.0.0.1:9202

# Start chat session
./core-agent chat --user=purberus

# Check models
./core-agent serve-profiles
```

### As a Go module

```go
package main

import (
    "context"
    "signal"
    "syscall"
    
    "dappco.re/go"
    "dappco.re/go/agent/pkg/agentic"
    "dappco.re/go/agent/pkg/brain"
    "dappco.re/go/agent/pkg/monitor"
)

func main() {
    // Initialize Core application
    app, _ := core.New(
        core.WithService(agentic.Register),
        core.WithService(brain.Register),
        core.WithService(monitor.Register),
    )
    
    ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
    defer stop()
    
    // Run the application
    app.Run(ctx)
}
```

### Dispatch example

```go
// Dispatch a task to an agent
result := agentic.Dispatch(context.Background(), agentic.DispatchOptions{
    Repo:    "go-io",
    Org:     "dAppCore",
    Task:    "fix-memory-leak",
    Agent:   "claude:sonnet",
    Issue:   42,
    Template: "coding",
    Persona:  "senior-dev",
})

if result.OK {
    workspaceDir := result.Value.(string)
    fmt.Println("Workspace:", workspaceDir)
}
```

### Workspace Preparation

```go
// Prepare a workspace for an agent
prepResult := agentic.PrepWorkspace(context.Background(), agentic.PrepInput{
    Repo:    "go-process",
    Org:     "dAppCore",
    Task:    "windows-support",
    Agent:   "claude:opus",
    Issue:   123,
    PR:      456,
    Branch:  "dev",
    Tag:     "",
    Template: "RFC.md",
    Persona:  "architect",
    Variables: map[string]string{
        "target": "windows",
    },
    DryRun: false,
})

if prepResult.OK {
    output := prepResult.Value.(agentic.PrepOutput)
    fmt.Printf("Workspace: %s\n", output.WorkspaceDir)
    fmt.Printf("Prompt: %s\n", output.Prompt)
}
```

### Brain operations

```go
// Store knowledge in brain
brain.Remember(context.Background(), brain.Memory{
    Content:    "The go-process package uses RingBuffer for output capture",
    Tags:       []string{"go-process", "architecture", "buffer"},
    Type:       "fact",
    Confidence:  0.95,
    Workspace:   "core/go-process",
    Agent:      "purberus",
})

// Recall from brain
results, _ := brain.Recall(context.Background(), brain.Query{
    Query:  "output capture",
    Tags:   []string{"go-process"},
    Limit:  5,
})

for _, result := range results {
    fmt.Printf("%s: %s\n", result.Tags, result.Content)
}
```

---

## Testing

### Test structure

The package follows the AX standard with test triplets:
- `*_test.go` — Unit and integration tests (Good/Bad/Ugly pattern)
- `*_example_test.go` — Usage examples as tests

**Naming convention:**
- `_Good` — Happy path tests
- `_Bad` — Expected error conditions
- `_Ugly` — Panics and edge cases

### Running tests

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

# Specific package
go test -v ./pkg/agentic/...
go test -v ./pkg/brain/...

# Isolated module verification
GOWORK=off go test ./...
```

### Test coverage areas

- ✅ Agentic dispatch and workspace prep
- ✅ Brain recall/remember/forget operations
- ✅ Chat history storage and retrieval
- ✅ Monitor sync and harvesting
- ✅ Runner local and container execution
- ✅ Setup project detection
- ✅ All MCP tool registrations
- ✅ Transport layers (stdio, tcp, http, unix)
- ✅ Provider integrations
- ✅ Path utilities and helpers

---

## API endpoints

### Hub control plane endpoints

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/api/agent/status` | Get agent status | Bearer |
| GET | `/api/agent/dispatch` | List dispatches | Bearer |
| POST | `/api/agent/dispatch` | Create dispatch | Bearer |
| GET | `/api/agent/brain/recall` | Recall from brain | Bearer |
| POST | `/api/agent/brain/remember` | Store in brain | Bearer |
| GET | `/api/agent/workspaces` | List workspaces | Bearer |
| GET | `/api/agent/models` | List available models | Bearer |

### MCP Endpoints

All MCP endpoints follow the Model Context Protocol specification:
- Tool calls via `mcp.CallToolRequest`
- Resource access via `mcp.ReadResourceRequest`
- Notification delivery via `mcp.Notification`
- Progress tracking via `mcp.ProgressNotification`

---

## Related packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-mcp](../mcp/) | MCP server framework (used by agent) | ../mcp/ |
| [go-process](../process/) | Process orchestration (used by runner) | ../process/ |
| [go-io](../io/) | I/O abstraction layer | ../io/ |
| [go-api](../api/) | REST framework | ../api/ |
| [go-ws](../ws/) | WebSocket hub | ../ws/ |
| [go-cache](../cache/) | Caching layer | ../cache/ |
| [go-session](../session/) | Session parsing | ../session/ |
| [CoreGO INDEX](../../INDEX.md) | Complete package catalog | ../../INDEX.md |

---

## Tags

```yaml
- ai-agent
- orchestration
- dispatch
- mcp-server
- model-context-protocol
- claude-code
- codex
- gemini
- hermes
- opencode
- brain
- openbrain
- semantic-memory
- workspace
- sandboxing
- container
- docker
- chat-archive
- duckdb
- monitoring
- fleet
- repository-sync
- pull-request
- code-review
- issue-tracking
- persona
- prompt
- flow
- production-ready
- high-coverage
- extensible
- corego
- action-system
- service-runtime
```

---

## References

1. **RFC Specification** — [plans/code/core/go/agent/RFC.md](../../../../../plans/code/core/go/agent/RFC.md)
2. **Repository** — [~/Code/core/agent/](file:///Users/snider/Code/core/agent/)
3. **CLAUDE.md** — [~/Code/core/agent/CLAUDE.md](file:///Users/snider/Code/core/agent/CLAUDE.md)
4. **GOAL.md** — [~/Code/core/agent/GOAL.md](file:///Users/snider/Code/core/agent/GOAL.md)
5. **MCP Specification** — [modelcontextprotocol.io](https://modelcontextprotocol.io)
6. **CoreGO Framework** — [CoreGO INDEX](../../INDEX.md)
7. **Lethean Project** — [Lethean Documentation](https://docs.lthn.io)

