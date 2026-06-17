---
type: Package Documentation
package: ide
module: dappco.re/go/ide
repo: core/ide
lang: go
tags:
  - ide
  - desktop
  - lethean-desktop
  - wails
  - editor
  - mcp-server
  - chat
  - subagent
  - workspace
  - brain
  - vi-control-panel
  - control-panel
---
# go-ide — Lethean Desktop IDE

> **The compile target for Lethean Desktop — native umbrella product across Darwin/Linux/Windows/iOS/iPadOS**

**RFC:** [plans/project/lthn/desktop/RFC.md](file:///Users/snider/Code/meowmix/plans/project/lthn/desktop/RFC.md)
**Source:** [~/Code/core/ide/](file:///Users/snider/Code/core/ide/)
**Module:** `dappco.re/go/ide`
**Binary:** `core-ide` (ships as **Lethean Desktop**)
**Frontend:** Angular 20+ (embedded via `//go:embed`)
**Backend:** Wails v3 + Go
**Status:** ✅ Production-Ready
**License:** EUPL-1.2

---

## 🎯 Overview

`go-ide` is the **Lethean Desktop** compile target — a native umbrella product that provides a comprehensive IDE experience across macOS (Darwin), Linux, Windows, iOS, and iPadOS. It composes multiple Core ecosystem packages into a unified desktop application with a thin Wails shell.

The IDE serves as both:
- **GUI Mode** — Full desktop application with Vi Control Panel shell
- **MCP Mode** — Stdio MCP server for Claude Code and other editor integrations
- **HTTP Mode** — HTTP MCP daemon for JetBrains, remote agents, and local agent communication

### What This Binary IS

`core/ide` owns **composition** — which packages get pulled in, how they wire at boot, and how Vi (the Control Panel) surfaces them. The **implementation** of each capability lives in its canonical home:

| Capability | Canonical Spec |
|------------|----------------|
| Lethean Desktop convergence + product framing | `plans/project/lthn/desktop/RFC.md` |
| Visual system + native profiles + Vi Control Panel | `plans/ops/hostuk/website/_design/lethean-3/` |
| Blockchain / Mining / Encryption / Filesystem | `plans/project/lthn/{blockchain,mining}/RFC.md` |
| CoreGUI / CoreTS / CoreApp / core/agent / core/mcp | `plans/code/core/{gui,ts,app,agent,mcp}/` |

**Rule:** If a capability isn't defined in the plans tree, it isn't real.

### Brand Identity

- **Default Brand:** **Lethean** (cool indigo, hue 270)
- **Host UK Variant:** Can flip via `[data-brand="hostuk"]` at runtime
- **Vi (Violet) Mascot:** The spine of the Control Panel (raven character)
  - Character canon: `plans/ops/hostuk/website/_design/lethean-3/uploads/mascot-raven.md`
  - Voice samples: `plans/ops/hostuk/website/_design/lethean-3/uploads/mascot-voice-samples.md`
  - Brand voice: `plans/ops/hostuk/website/_design/lethean-3/uploads/BRAND-VOICE.md`
  - Character: *The chill chick — a raven watching the tower for people, letting them know when the weather is changing or trouble is at the gates.*
  - Style: Italic Instrument Serif "*Quiet night.*" — used **sparingly** (one phrase per surface max)

### Vi Control Panel

Vi is the **spine of the desktop shell** — not a chat widget. It's a Control Panel that surfaces what matters without performing. The same logical component tree works across:
- **Web:** `control.host.uk.com`
- **Native:** This binary (core-ide)

**Brand/Platform Swapping:**
- Brand swap via `[data-brand]` attribute
- Platform swap via `[data-platform]` attribute

---

## 🏗️ Architecture

### Thin Wails Shell

The IDE uses a **thin Wails shell** that wires ecosystem packages via `core.Core` dependency injection. Three operating modes are supported:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Lethean Desktop Binary                                   │
│  core-ide — The single binary that ships as Lethean Desktop                │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Wails v3 Shell                                           │
│  Thin wrapper that embeds Angular frontend and manages native windowing    │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Core Framework (dappco.re/go)                            │
│  Service container with dependency injection                              │
│  ACTION system for event-driven communication                             │
│  Result pattern for error handling                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Composed Services                                        │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│  │  display    │ │   mcp       │ │   ide       │ │   ws        │ │
│  │  (core/gui) │ │ (core/mcp)  │ │  bridge     │ │  (go-ws)    │ │
│  │             │ │             │ │             │ │             │ │
│  │ 74 MCP tools│ │ GUI subsys  │ │ Laravel     │ │ WebSocket   │ │
│  │ 14 categories│ │ Brain      │ │ backend     │ │ hub         │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘ │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Frontend (Embedded Angular 20+)                          │
│  ┌─────────────────────────┐ ┌─────────────────────────┐ │
│  │   /tray route           │ │    /ide route            │ │
│  │  System tray panel       │ │  Full IDE layout         │ │
│  │  380x480 frameless       │ │  Complete desktop         │ │
│  └─────────────────────────┘ └─────────────────────────┘ │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Display Modes (Runtime-Switchable)                        │
│  ClientHub (default) | ServerHub | GatewayHub | DeveloperHub | AdminHub    │
│  Same binary, runtime-switchable, fully customisable                       │
│  Users can flip without restart via core/config persistence                │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Operating Modes

#### 1. GUI Mode (Default)

**Entry Point:** `main()` → Wails 3 application with embedded Angular frontend

**Features:**
- System tray (macOS: accessory app, no Dock icon)
- Window management via `core/gui` display service
- 74 MCP tools across 14 categories
- WebSocket bridge to Angular frontend
- IDE bridge to Laravel core-agentic backend
- WebSocket hub for Angular communication

**Process Flow:**
```
main()
  → Wails 3 initialization
  → Embed Angular frontend
  → Register Core services
  → Start WebSocket hub
  → Show system tray
  → Mount IDE routes (/tray, /ide)
  → Enable GUI mode
```

#### 2. MCP Mode (`--mcp`)

**Entry Point:** `core-ide --mcp` → Stdio MCP server

**Features:**
- No GUI, no HTTP
- Stdio transport for Claude Code integration
- Full MCP tool set available
- Configure in `.claude/.mcp.json`

**Configuration:**
```json
{
  "mcpServers": {
    "core-ide": {
      "type": "stdio",
      "command": "core-ide",
      "args": ["--mcp"]
    }
  }
}
```

#### 3. HTTP Mode (`--no-gui --http`)

**Entry Point:** `core-ide --no-gui --http 127.0.0.1:9880 --token $TOK`

**Features:**
- HTTP MCP daemon for JetBrains and remote agents
- Bearer token authentication (required)
- Loopback-only binding (localhost or explicit loopback IP)
- Wildcard addresses (`:9880`, `0.0.0.0:9880`) are **rejected**

**Usage:**
```bash
# Start HTTP mode
core-ide --no-gui \
  --http 127.0.0.1:9880 \
  --token your-secret-token

# Connect with bearer token
curl -H "Authorization: Bearer your-secret-token" \
  http://127.0.0.1:9880/mcp/tools/list
```

#### 4. Headless Mode

**Entry Point:** Core framework runs without Wails

**Features:**
- All services run without GUI
- MCP transport determined by `MCP_ADDR` environment variable
- TCP if `MCP_ADDR` is set, stdio otherwise
- No Wails dependencies

---

## 📦 Package Structure

### Repository Layout

```
core/ide/
├── go/                                  # Go backend module
│   ├── cmd/core-ide/                   # Binary entry point
│   │   ├── main.go                    # Main entry with mode detection
│   │   ├── wails_boot.go             # Wails bootstrap
│   │   ├── flags.go                  # CLI flags parsing
│   │   └── embed.go                  # Embedded frontend assets
│   │
│   ├── service.go                     # Main service registration
│   │
│   ├── third_party/                   # Vendored dependencies
│   │   ├── core_api/                 # Core API provider
│   │   │   └── pkg/provider/         # Provider implementations
│   │   ├── gui/                      # GUI components
│   │   │   └── pkg/
│   │   │       ├── chat/             # Chat service
│   │   │       │   └── register.go   # Chat registration
│   │   │       └── mcp/              # MCP tools
│   │   │           └── mcp.go         # MCP service
│   │   └── process/                  # Process management
│   │       └── pkg/api/             # API provider
│   │           ├── provider.go       # Provider implementation
│   │           └── embed.go           # Embedded assets
│   │
│   ├── .lintdeps/                     # Lint dependencies
│   │   └── dappco.re/
│   │       └── go/
│   │           └── gui@v0.9.0/
│   │               └── pkg/
│   │                   └── ...
│   │
│   ├── go.mod
│   ├── go.sum
│   └── go.work
│
├── frontend/                          # Angular frontend
│   ├── src/                          # TypeScript source
│   │   ├── app/                      # Angular application
│   │   │   ├── components/           # UI components
│   │   │   │   ├── tray/            # System tray panel
│   │   │   │   └── ide/             # Full IDE layout
│   │   │   ├── services/            # Frontend services
│   │   │   └── ...
│   │   └── ...
│   ├── angular.json                   # Angular configuration
│   ├── package.json                  # npm dependencies
│   └── ...
│
├── build/                             # Build configurations
│   ├── windows/                      # Windows-specific
│   ├── linux/                       # Linux-specific
│   └── macos/                       # macOS-specific
│
├── dist/                              # Built binaries
│   └── ...
│
├── .core/                             # Runtime configuration
│   └── ...
│
├── .forgejo/                          # Forgejo CI configuration
│   └── workflows/
│       ├── ci.yml                   # CI pipeline
│       └── security-scan.yml        # Security scanning
│
├── docs/                              # Documentation
│   └── ...
│
├── README.md                          # Repository README
├── CLAUDE.md                          # Development guidance
├── AGENTS.md                         # Agent guidance
├── LICENCE                           # EUPL-1.2 license
└── Taskfile.yml                      # Build orchestration
```

### Frontend Routes

| Route | Component | Description |
|-------|-----------|-------------|
| `/tray` | TrayPanel | System tray panel (380x480 frameless) |
| `/ide` | IDELayout | Full IDE layout |

---

## 🎯 Running Modes

### Mode Comparison

| Mode | Command | Use Case | Transport | Authentication |
|------|---------|----------|-----------|----------------|
| GUI | `core-ide` | Local desktop with chat | Wails | None |
| Stdio MCP | `core-ide --mcp` | Claude Code, Cursor, Continue | Stdio | None |
| HTTP MCP | `core-ide --no-gui --http <addr> --token <tok>` | JetBrains, remote agents | HTTP | Bearer Token (required) |
| Headless | `CORE_DAEMON=1 core-ide` | Background service | TCP/Stdio | Optional |

### HTTP Mode Security

**Security Hardening:**
- HTTP mode **refuses to start without a bearer token**
- REST and MCP-over-HTTP requests require `Authorization: Bearer <token>`
- Missing or wrong tokens return `401 Unauthorized`
- HTTP and relay listeners **must bind to localhost or loopback IP**
- Wildcard addresses (`:9880`, `0.0.0.0:9880`) are **rejected**
- HTTP read-header timeout: bounded
- HTTP read timeout: bounded
- HTTP write timeout: bounded
- HTTP idle timeout: bounded
- HTTP headers capped at **1 MiB**
- HTTP request bodies capped at **10 MiB**
- Relay listener only enabled when: relay path + loopback bind + bearer token configured

### Command Examples

```bash
# GUI mode (default)
core-ide

# Stdio MCP mode (Claude Code)
core-ide --mcp

# HTTP MCP mode (JetBrains, remote)
core-ide --no-gui --http 127.0.0.1:9880 --token my-secret-token

# Headless mode
CORE_DAEMON=1 core-ide
```

---

## 🚀 MCP Tools (19 Tools)

The IDE exposes **19 MCP tools** with **action parity** enforced in `pkg/server/integration_action_parity_test.go`.

### Brain Tools (Semantic Memory)

| Tool | Description | Parameters |
|------|-------------|------------|
| `brain_recall` | Recall from OpenBrain | query, tags, limit, workspace |
| `brain_remember` | Store in OpenBrain | content, tags, metadata, workspace |
| `brain_forget` | Remove from OpenBrain | id, workspace |
| `brain_list` | List OpenBrain entries | tags, limit, workspace |
| `brain_context` | Get context from OpenBrain | query, workspace |

### Workspace Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `workspace_status` | Get workspace status | path |
| `workspace_conventions` | Get workspace conventions | path |
| `workspace_impact` | Analyze workspace impact | path, changes |
| `workspace_scan` | Scan workspace for issues | path, depth, patterns |

### Subagent Tools (AI Assistant Relay)

| Tool | Description | Parameters |
|------|-------------|------------|
| `subagent_guide` | Get guidance from subagent | query, context |
| `subagent_ask` | Ask subagent a question | question, context |
| `subagent_progress` | Get subagent progress | session_id |
| `subagent_watch` | Watch subagent activity | cursor, limit, nextCursor, hasMore |
| `subagent_answer` | Get subagent answer | question, context |
| `subagent_dispatch_guided` | Dispatch guided task | task, context |

### Navigation & Package Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `core_navigate` | Navigate to resource | path, resource |
| `pkg_search` | Search for packages | query, limit |
| `pkg_info` | Get package info | package |
| `pkg_install` | Install package | package, version |

### Tool Parity

All 19 tools maintain **MCP/action parity** — they can be called via:
- MCP protocol (stdio/HTTP)
- Core ACTION system
- CLI projection (where applicable)

This ensures consistent behavior across all interfaces.

---

## 🏗️ Core Services

The IDE composes multiple Core services via dependency injection:

### 1. Display Service (`core/gui/pkg/display/`)

**Window management, webview automation, 74 MCP tools across 14 categories**

- Window lifecycle management
- WebSocket event broadcasting
- Menu and tray setup
- Configuration persistence

### 2. MCP Service (`core/mcp`)

**Model Context Protocol server** with:
- File operations tools
- Brain subsystem (semantic memory)
- GUI subsystem (window/dialog/notification)
- Agentic subsystem (dispatch/workspace)
- IDE bridge subsystem

### 3. IDE Bridge (`core/mcp/pkg/mcp/ide/`)

**WebSocket bridge to Laravel core-agentic backend**

- Connects frontend to Laravel backend
- Real-time bidirectional communication
- Agent task management
- Workspace operations

### 4. WebSocket Hub (`core/go-ws`)

**WebSocket hub for Angular frontend communication**

- Manages frontend connections
- Broadcasts events to all connected clients
- Handles message routing
- Supports subscription filtering

---

## 🔌 Configuration

### Environment Variables

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `MCP_ADDR` | MCP TCP address | No | - |
| `CORE_BRAIN_KEY` | OpenBrain API key | No | - |
| `CORE_DAEMON` | Run in daemon mode | No | `0` |
| `CORE_MODE` | Display mode | No | `ClientHub` |
| `CORE_BRAND` | Brand override | No | `lethean` |

### Configuration File

**Location:** `~/.core/config.yaml` (or project `.core/config.yaml`)

```yaml
# Display mode (runtime-switchable)
mode: ClientHub
# Options: ClientHub, ServerHub, GatewayHub, DeveloperHub, AdminHub

# Brand
brand: lethean
# Options: lethean, hostuk

# MCP settings
mcp:
  addr: 127.0.0.1:9880
  token: your-secret-token

# Brain integration
brain:
  enabled: true
  key: ${CORE_BRAIN_KEY}

# GUI settings
gui:
  enabled: true
  system_tray: true
  start_minimized: false

# IDE settings
ide:
  default_workspace: ~/Code
  show_hidden_files: false
```

---

## 🚀 Build & Development

### Prerequisites

- Go 1.26+
- Node.js (for Angular frontend)
- Wails v3
- Task

### Build Commands

```bash
# Development mode (hot-reload GUI + Go rebuild)
wails3 dev

# Production build (preferred)
core build

# Frontend-only development
cd frontend
npm install
npm run dev

# Frontend-only production build
cd frontend
npm install
npm run build
```

### Test Commands

```bash
# All Go tests
core go test

# Single test
core go test --run TestName

# With coverage
core go cov

# Coverage in browser
core go cov --open

# Full QA pipeline
core go qa

# Full QA with extras
core go qa full

# Format only
core go fmt

# Lint only
core go lint

# Frontend tests
cd frontend
npm run test
```

### Live OpenBrain Test

The live OpenBrain integration test is **build-tagged** and skipped by default:

```bash
# Enable integration tests
CORE_BRAIN_INTEGRATION=1 \
  CORE_BRAIN_KEY=$CORE_BRAIN_KEY \
  go test -tags integration -run TestLive ./pkg/brain/...
```

---

## 🎨 Vi Control Panel

### Design System

The **Lethean-3 unified design system** defines the visual appearance and behavior:

**Canonical Location:** `plans/ops/hostuk/website/_design/lethean-3/`

**Native Profiles:**
- Wails Darwin (macOS) native profile
- Wails iOS native profile
- Wails iPadOS native profile
- Windows native profile
- Linux native profile

**Components:**
- Vi mascot (raven character)
- Brand voice guidelines
- Color palette (cool indigo, hue 270)
- Typography (Italic Instrument Serif for Vi)
- Layout patterns
- Interaction patterns

### Brand Switching

**Lethean Brand:**
```html
<div data-brand="lethean">...</div>
```

**Host UK Brand:**
```html
<div data-brand="hostuk">...</div>
```

**Platform Switching:**
```html
<div data-platform="darwin">...</div>
<div data-platform="linux">...</div>
<div data-platform="windows">...</div>
```

### Display Modes

All modes are **runtime-switchable** and **fully customisable**:

| Mode | Target Audience | Features |
|------|----------------|----------|
| **ClientHub** | End users | Full IDE, chat, workspace management |
| **ServerHub** | Server operators | Server monitoring, configuration |
| **GatewayHub** | Gateway operators | Gateway management, routing |
| **DeveloperHub** | Developers | Full development toolchain |
| **AdminHub** | System administrators | Admin tools, user management |

**Custom Modes:**
```html
<div data-mode="custom-foo">...</div>
```

**State Persistence:** Per-user mode preferences persist via `core/config`.

---

## 🌐 Integration

### Claude Code Integration

Configure in `.claude/.mcp.json`:

```json
{
  "mcpServers": {
    "core-ide": {
      "type": "stdio",
      "command": "core-ide",
      "args": ["--mcp"]
    }
  }
}
```

**Features Available:**
- All 19 MCP tools
- File operations
- Brain semantic memory
- Workspace management
- Subagent relay
- Package marketplace

### JetBrains Integration

Configure as MCP client:

```bash
# Start IDE in HTTP MCP mode
core-ide --no-gui --http 127.0.0.1:9880 --token your-token

# Configure JetBrains to connect to localhost:9880
# with Bearer token authentication
```

### Remote Agent Integration

```bash
# Start IDE for remote agent access
core-ide --no-gui \
  --http 127.0.0.1:9880 \
  --token remote-agent-token

# Remote agent connects via HTTP MCP
curl -H "Authorization: Bearer remote-agent-token" \
  http://127.0.0.1:9880/mcp/tools/call \
  -d '{"name": "brain_recall", "arguments": {"query": "test"}}'
```

---

## 🧪 Testing

### Test Structure

The IDE follows the AX Standard with test triplets:
- `*_test.go` — Unit and integration tests
- `*_example_test.go` — Usage examples

**Naming Convention:**
- `TestGood*` — Happy path tests
- `TestBad*` — Expected error conditions
- `TestUgly*` — Panics and edge cases

### Parity Testing

The `integration_action_parity_test.go` enforces MCP/action parity for all 19 tools:

```go
// Each tool is tested for:
// 1. MCP tool registration
// 2. ACTION system registration
// 3. Consistent parameter handling
// 4. Consistent return types
// 5. Error handling
```

### Running Tests

```bash
# All tests
cd ~/Code/core/ide/go
go test -count=1 ./...

# Specific test
go test -run TestActionParity ./pkg/server/

# With race detector
go test -race ./...

# Integration tests (build-tagged)
CORE_BRAIN_INTEGRATION=1 CORE_BRAIN_KEY=$KEY \
  go test -tags integration ./...

# Frontend tests
cd frontend
npm run test
```

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| **MCP Tools** | 19 |
| **Action Parity** | 19 (100%) |
| **GUI Routes** | 2 (/tray, /ide) |
| **Display Modes** | 5 (ClientHub, ServerHub, GatewayHub, DeveloperHub, AdminHub) |
| **Brands** | 2 (Lethean, Host UK) |
| **Platforms** | 5 (Darwin, Linux, Windows, iOS, iPadOS) |
| **Transport Modes** | 4 (GUI, Stdio MCP, HTTP MCP, Headless) |
| **Frontend** | Angular 20+ |
| **Backend** | Wails v3 + Go |
| **Dependencies** | Core framework + Wails + gorilla/websocket + MCP SDK |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-gui](../gui/) | GUI framework (display service) | [../gui/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/gui/) |
| [go-mcp](../mcp/) | MCP server framework | [../mcp/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/mcp/) |
| [go-ws](../ws/) | WebSocket hub | [../ws/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/ws/) |
| [go-agent](../agent/) | Agent orchestration | [../agent/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/agent/) |
| [go-cli](../cli/) | Command-line interface | [../cli/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/cli/) |
| [CoreGO INDEX](../../INDEX.md) | Complete catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## 📈 Quality Metrics

- ✅ **MCP Specification Compliance** — Full MCP v1.0 implementation
- ✅ **Action Parity** — 19 tools with MCP/action consistency
- ✅ **Security Hardened** — Bearer auth, loopback-only, size limits, timeouts
- ✅ **Test Coverage** — Good/Bad/Ugly pattern
- ✅ **Parity Testing** — Integration tests for all tools
- ✅ **Documentation** — Complete README + INDEX
- ✅ **Core Integration** — Full CoreGO framework support
- ✅ **Multi-Platform** — macOS, Linux, Windows, iOS, iPadOS
- ✅ **Production Ready** — Deployed as Lethean Desktop

---

## 📝 Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-06-17 | Complete knowledge pack documentation | Purberus |
| 2026-05-XX | All MCP tools finalized | Maintainer |
| 2026-04-XX | Action parity testing added | Maintainer |
| 2026-03-XX | HTTP mode security hardening | Maintainer |
| 2026-02-XX | Initial IDE composition | Maintainer |

---

## 🎯 Tags

```yaml
# Core Identity
- ide
- desktop
- lethean-desktop
- lethean
- binary
- compile-target

# Modes
- gui-mode
- mcp-mode
- http-mode
- headless-mode
- stdio
- tcp

# Capabilities
- mcp-server
- model-context-protocol
- chat
- subagent
- relay
- workspace
- brain
- openbrain
- semantic-memory
- navigation
- package-management
- pkg-search
- pkg-install

# Architecture
- wails
- angular
- thin-shell
- composition
- dependency-injection
- service-container
- action-system
- result-pattern

# Security
- bearer-auth
- loopback-only
- size-limits
- timeouts
- hardened

# UI
- vi-control-panel
- vi
- violet
- raven
- tray-panel
- ide-layout
- responsive
- theming

# Platforms
- macos
- darwin
- linux
- windows
- ios
- ipados

# Brands
- lethean
- hostuk

# Quality
- production-ready
- high-coverage
- parity-tested
- documented
- multi-platform

# Ecosystem
- corego
- dappcore
- forgejo
- claude-code
- jetbrains
```

---

## 📚 References

1. **Repository** — [~/Code/core/ide/](file:///Users/snider/Code/core/ide/)
2. **CLAUDE.md** — [~/Code/core/ide/CLAUDE.md](file:///Users/snider/Code/core/ide/CLAUDE.md)
3. **README.md** — [~/Code/core/ide/README.md](file:///Users/snider/Code/core/ide/README.md)
4. **RFC (Lethean Desktop)** — [plans/project/lthn/desktop/RFC.md](file:///Users/snider/Code/meowmix/plans/project/lthn/desktop/RFC.md)
5. **Design System** — [plans/ops/hostuk/website/_design/lethean-3/](file:///Users/snider/Code/meowmix/plans/ops/hostuk/website/_design/lethean-3/)
6. **Vi Mascot Canon** — [plans/ops/hostuk/website/_design/lethean-3/uploads/mascot-raven.md](file:///Users/snider/Code/meowmix/plans/ops/hostuk/website/_design/lethean-3/uploads/mascot-raven.md)
7. **Brand Voice** — [plans/ops/hostuk/website/_design/lethean-3/uploads/BRAND-VOICE.md](file:///Users/snider/Code/meowmix/plans/ops/hostuk/website/_design/lethean-3/uploads/BRAND-VOICE.md)
8. **Wails** — [wails.io](https://wails.io)
9. **Angular** — [angular.io](https://angular.io)
10. **CoreGO INDEX** — [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md)

---

*Package documentation generated: 2026-06-17T23:00:00Z*
*Knowledge Pack: CoreGo v1.2.0*
*Binary: Lethean Desktop (core-ide)*
*Maintainer: Purberus <purberus@lthn.ai>*
