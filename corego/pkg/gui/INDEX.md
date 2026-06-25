---
type: Package Index
package: gui
module: dappco.re/go/gui
repo: core/gui
title: go-gui Package Index
description: Desktop GUI framework for the Core ecosystem — Wails v3 backend with Angular 20 frontend
lang: go
tags:
  - gui
  - desktop
  - wails
  - webview
  - window-management
  - angular
  - typescript
---
# go-gui Package Index

**Desktop GUI framework for the Core ecosystem**

**Repository:** `core/gui`  
**Module:** `dappco.re/go/gui`  
**Frontend:** Angular 20 Custom Elements  
**Backend:** Wails v3 (Go)  
**Status:** Production-Ready  
**License:** EUPL-1.2  
**Test Pattern:** Good/Bad/Ugly (AX Standard)  
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| Repository README | Original package README | [~/Code/core/gui/README.md](file:///Users/snider/Code/core/gui/README.md) |
| CLAUDE.md | Development guidance | [~/Code/core/gui/CLAUDE.md](file:///Users/snider/Code/core/gui/CLAUDE.md) |

---

## Package overview

`go-gui` is the HUI (Human-facing User Interface) twin to the AUI (Agent-facing User Interface) provided by go-agent. It is a desktop GUI framework built on Wails v3 (Go backend) with an Angular 20 frontend that provides window management, dialogs, system tray, clipboard, notifications, theming, layouts, and real-time WebSocket events.

### Core capabilities

1. **Window Management** — Complete window lifecycle with state persistence, layout management, tiling, and snapping
2. **Dialog System** — File open/save, message, confirm, and prompt dialogs
3. **System Tray** — Icon with tooltip, menu, and click handlers
4. **Clipboard** — Read, write, and clear operations
5. **Notifications** — System notifications with permission management
6. **Screen Management** — Multi-monitor queries and work area calculations
7. **Environment Detection** — Theme (dark/light) and OS information
8. **Keybinding** — Global keyboard shortcut registration
9. **Context Menu** — Named context menu registration
10. **Browser Control** — Open URLs and files
11. **Dock Integration** — macOS dock visibility and badge management
12. **Lifecycle Management** — Application lifecycle events
13. **WebView Automation** — JavaScript execution, click, type, screenshot, DOM queries
14. **MCP Tool Surface** — All services exposed as MCP tools
15. **Chat Integration** — WebView-based chat with ML models
16. **P2P Messaging** — Peer-to-peer communication
17. **Container Management** — TIM container lifecycle
18. **Marketplace** — Plugin/package discovery and installation
19. **Preload** — Trusted preload policy

### Design principles

- **Service-Oriented Architecture** — All functionality as Core services with dependency injection
- **Platform-Agnostic** — Platform-specific code behind common interfaces
- **Testable** — All Wails APIs abstracted with mock implementations
- **Event-Driven** — WebSocket-based real-time communication
- **Extensible** — Plugin architecture via marketplace and MCP
- **HUI/AUI Symmetry** — Human and Agent interfaces for the same Core machinery

---

## Architecture

### Component layers

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    HUI Layer (Human-Facing Interface)                        │
│  Desktop GUI for human interaction with Core ecosystem                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Frontend (ui/)                                             │
│  Angular 20 Custom Elements | TypeScript | HTML | CSS | Responsive Design   │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Backend (go/)                                             │
│  Wails v3 Application | WebSocket Events | MCP Tool Registration             │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Service Layer (pkg/)                                     │
│  display/    | Orchestrator + WebSocket + menu/tray + config              │
│  window/     | Window lifecycle + state + layout + tiling + snapping        │
│  menu/       | Application menu construction                                │
│  systray/    | System tray icon + tooltip + menu                            │
│  dialog/     | File open/save + message + confirm + prompt                   │
│  clipboard/  | Clipboard read/write/clear                                  │
│  notification/ | System notifications + permissions                         │
│  screen/     | Multi-monitor queries                                        │
│  environment/ | Theme detection + OS info                                   │
│  keybinding/ | Global keyboard shortcuts                                  │
│  contextmenu/ | Named context menus                                         │
│  browser/    | URL/file opening                                            │
│  dock/       | macOS dock integration                                      │
│  lifecycle/  | Application lifecycle events                                │
│  webview/    | WebView automation                                          │
│  mcp/        | MCP tool surface for all services                           │
│  chat/       | Chat functionality                                          │
│  p2p/        | Peer-to-peer messaging                                      │
│  container/  | TIM container lifecycle                                     │
│  preload/    | Trusted preload policy                                      │
│  marketplace/ | Plugin/package marketplace                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Platform Abstraction                                    │
│  interfaces.go | Platform interfaces for all Wails APIs                     │
│  wailsApp adapter | Real Wails implementation                                 │
│  mockApp | Mock implementations for testing                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Stubs Layer                                              │
│  stubs/wails/ | Wails shim packages for testing without native bindings    │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Core Framework                                          │
│  dappco.re/go | ServiceRuntime + ACTION system + Result pattern           │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Repository structure

### Go backend (go/)

```
go/
├── internal/                       # Internal packages
│   └── ...
├── pkg/                            # 21 Service packages
│   ├── browser/                   # Browser control
│   ├── chat/                      # Chat functionality
│   ├── clipboard/                 # Clipboard operations
│   ├── container/                # TIM container management
│   ├── contextmenu/              # Context menu system
│   ├── deno/                     # Deno integration
│   ├── dialog/                   # Dialog system
│   ├── display/                  # **Display Orchestrator**
│   │   ├── *.go                   # Service + events + config
│   │   └── assets/              # Frontend assets
│   ├── dock/                    # macOS dock integration
│   ├── environment/              # Environment detection
│   ├── events/                   # Event system
│   ├── keybinding/               # Keyboard shortcuts
│   ├── lifecycle/                # Application lifecycle
│   ├── marketplace/              # Marketplace integration
│   ├── menu/                     # Application menu
│   ├── mcp/                      # MCP tool surface
│   ├── notification/             # System notifications
│   ├── p2p/                      # Peer-to-peer messaging
│   ├── preload/                  # Trusted preload
│   ├── screen/                   # Screen management
│   ├── systray/                  # System tray
│   │   └── assets/               # Tray assets
│   ├── webview/                  # WebView automation
│   └── window/                   # **Window Management**
│       ├── *.go                  # Manager + state + layout
│       ├── tiling.go             # Tiling functionality
│       └── snapping.go          # Snapping functionality
├── stubs/wails/                 # Wails stubs for testing
├── tests/                        # Go tests
├── go.mod
├── go.sum
└── go.work
```

### Angular frontend (ui/)

```
ui/
├── src/
│   ├── app/                      # Angular application
│   │   ├── components/           # Reusable components
│   │   ├── services/            # Frontend services
│   │   ├── models/              # Data models
│   │   └── ...
│   ├── environments/             # Environment configurations
│   ├── assets/                  # Static assets
│   │   └── icons/
│   └── styles/                  # Global styles
├── angular.json                   # Angular configuration
├── package.json                  # npm dependencies
├── tsconfig.json                 # TypeScript configuration
└── ...
```

---

## Services (21 total)

### Display Service
- WebSocket event broadcasting to frontend
- Menu and tray setup
- Configuration persistence
- Sub-service coordination
- MCP tool registration

### Window Service
- Window lifecycle management
- State persistence
- Layout management
- Window tiling
- Window snapping
- Cross-platform support

### Menu Service
- Application menu construction
- Platform abstraction
- Keyboard shortcuts

### System Tray Service
- Tray icon with tooltip
- Context menu
- Click handlers

### Dialog Service
- File open/save
- Message dialogs
- Confirm dialogs
- Prompt dialogs

### Clipboard Service
- Read/write/clear text
- Platform-specific

### Notification Service
- System notifications
- Permission management
- Icons and actions

### Screen Service
- Multi-monitor support
- Screen queries
- Work area calculations
- DPI information

### Environment Service
- Theme detection (dark/light/system)
- OS information
- High contrast mode
- Theme change monitoring

### Keybinding Service
- Global shortcut registration
- Platform-specific handling

### Context Menu Service
- Named context menus
- Position-based display

### Browser Service
- Open URLs in default browser
- Open files in default application

### Dock Service (macOS)
- Icon visibility
- Badge management
- Icon bouncing

### Lifecycle Service
- Startup, ready, shutdown, suspend, resume, terminate events

### WebView Service
- JavaScript execution
- Element clicking and typing
- Screenshots
- DOM queries

### MCP Service
- All GUI services as MCP tools
- WebSocket-based execution
- Claude Code integration

### Chat Service
- WebView-based chat
- Session management
- ML model integration

### P2P Service
- Message sending/receiving
- Connection management
- Presence tracking
- Encrypted messaging

### Container Service
- TIM container lifecycle
- Resource isolation
- Sandboxed execution

### Preload Service
- Trusted preload injection
- Security policy enforcement

### Marketplace Service
- Plugin discovery
- Installation and updates
- Version management

---

## Frontend features

### Angular 20 custom elements
- Custom elements for Wails integration
- Reusable component architecture
- Type-safe TypeScript
- Reactive programming with RxJS

### Real-time updates
- WebSocket connection to backend
- Event-driven architecture
- Bidirectional communication

### Theming
- Dark and light theme support
- Automatic theme detection
- Theme change monitoring

---

## Quick start

### Installation

```bash
# Clone repository
cd ~/Code/core/gui
git pull origin dev

# Build backend
cd go
go build -o core-gui .

# Build frontend
cd ../ui
npm install
npm run build

# Run application
cd ..
./go/core-gui
```

### Go integration

```go
package main

import (
    "context"
    "signal"
    "syscall"
    
    core "dappco.re/go"
    "dappco.re/go/gui/pkg/display"
)

func main() {
    app, result := core.New(
        core.WithService(display.Register),
    )
    if !result.OK {
        panic(result.Value)
    }
    
    ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
    defer stop()
    
    if err := app.Run(ctx); err != nil {
        panic(err)
    }
}
```

### Service usage

```go
// Window
w, _ := window.New(window.WithTitle("My App"))
w.Show()

// Dialog
path, _ := dialog.OpenFile()

// Notification
notification.Show(notification.WithTitle("Hello"))

// WebView
webview.Eval(`alert('Hello')`)
```

---

## Testing

### Test structure
- `*_test.go` — Unit and integration tests
- `*_example_test.go` — Usage examples
- Naming: `TestGood*`, `TestBad*`, `TestUgly*`

### Testing approach
1. Mock all platform APIs
2. Isolated tests per service
3. Integration tests for service interactions
4. Wails stubs for desktop-independent testing

### Running tests

```bash
# All Go tests
cd ~/Code/core/gui/go
GOWORK=off go test -count=1 ./...

# Specific package
go test -v ./pkg/window/...

# Frontend tests
cd ui
npm test
```

---

## Related packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-agent](../agent/) | AUI twin (Agent-facing) | [../agent/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/agent/) |
| [go-cli](../cli/) | Command-line interface | [../cli/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/cli/) |
| [CoreGO Framework](file:///Users/snider/Code/core/go/) | Foundation | N/A |
| [CoreGO INDEX](../../INDEX.md) | Complete catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## Tags

```yaml
# Core
- gui
- desktop
- wails
- webview
- hui
- human-interface

# Services
- window-management
- dialog
- systray
- clipboard
- notification
- screen
- environment
- keybinding
- contextmenu
- browser
- dock
- lifecycle
- webview
- mcp
- chat
- p2p
- container
- preload
- marketplace

# Frontend
- angular
- typescript
- custom-elements
- responsive
- theming

# Backend
- go
- wails
- websocket
- platform-abstraction

# Architecture
- service-oriented
- event-driven
- plugin-architecture

# Quality
- production-ready
- high-coverage
- testable
- documented
- cross-platform

# Ecosystem
- corego
- dappcore
- lethean
- forgejo
```

---

## References

1. **Repository** — [~/Code/core/gui/](file:///Users/snider/Code/core/gui/)
2. **CLAUDE.md** — [~/Code/core/gui/CLAUDE.md](file:///Users/snider/Code/core/gui/CLAUDE.md)
3. **README.md** — [~/Code/core/gui/README.md](file:///Users/snider/Code/core/gui/README.md)
4. **Wails** — [wails.io](https://wails.io)
5. **Angular** — [angular.io](https://angular.io)
6. **CoreGO INDEX** — [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md)

