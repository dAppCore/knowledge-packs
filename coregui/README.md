---
type: Knowledge Pack
title: CoreGUI Framework
description: Wails v2-based GUI framework for Lethean desktop applications
author: Mistral Vibe
version: 1.0.0
created: 2026-06-17T14:30:00Z
tags: [framework, gui, wails, desktop, lethean, wails3]
---

# CoreGUI knowledge pack

**Official site:** [dappco.re/go/gui/](https://dappco.re/go/gui/)

CoreGUI is the **Wails v2** framework for Lethean desktop applications, providing a native GUI wrapper with webview-based frontend and Go backend.

**Official description:** Native GUI runtime — Wails-shaped windows over core/go primitives.

---

## Overview

CoreGUI serves as the **gateway binary** for provider-hosted actions.

### Gateway role

From the [CoreGUI RFC](file:///Users/snider/Code/meowmix/plans/code/core/gui/RFC.md):

CoreGUI serves as the gateway binary that:
- Mounts all providers (go-ai, go-mlx, go-html, go-io, go-p2p, plus pending providers) inside the CoreGUI desktop process
- Exposes their action surface over local HTTP on **loopback only**
- Manages desktop process lifecycle
- Provides window management and orchestration

### Security considerations

**Blast-radius implication:** Single-process compromise. A webview-driven privilege escalation can directly compromise mounted providers and their state. The Cerberus DREAD context is documented in `code/core/api/RFC.providers.md § gateway-binary`.

**CRITICAL:** CoreGUI MUST implement a per-action allow-list (severity: CRITICAL post-#1044 PATH A) before shipping providers in production builds.

### Key facts

- **Framework:** Wails v3.0.0-alpha.91
- **Display package:** 30+ files, 67KB main implementation
- **Test coverage:** 100% test triplet pattern
- **Ports:** Vite (9245), Bridge (9879) for lthn-desktop
- **Status:** Page in flight — content fills in as the package converges

### Official install

```bash
go get dappco.re/go/gui@latest
```

### Official import

```go
import "dappco.re/go/gui"
```

---

## Source of truth

- **Primary spec:** [`plans/code/core/gui/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/gui/RFC.md)
- **Implementation:** [`core/gui/go/pkg/display/`](file:///Users/snider/Code/core/gui/go/pkg/display/)
- **Agent guide:** [`core/gui/AGENTS.md`](file:///Users/snider/Code/core/gui/AGENTS.md)

---

## Architecture

```
CoreGUI (gateway binary)
├── Provider Mounting
│   ├── go-ai (AI operations)
│   ├── go-mlx (Apple Metal GPU)
│   ├── go-html (HTML rendering)
│   ├── go-io (I/O operations)
│   └── go-p2p (P2P networking)
├── HTTP Server
│   └── Loopback only (127.0.0.1)
├── Window Management
│   └── Wails v2 integration
└── Action Surface
    └── REST/JSON API for all providers
```

**Built on:** `dappco.re/go/gui` — Wails-shaped windows over core/go primitives

---

## How-tos

### 1. Starting CoreGUI development

```bash
# Clone the repository
git clone forge.lthn.sh/core/gui
cd gui

# Start development server (Wails v2)
task dev

# Or use the helper script
gui.sh dev
```

### 2. Building CoreGUI

```bash
# Build the binary
go build -o lthn-desktop ./...

# Or use the helper
gui.sh build
```

### 3. Managing lifecycle

```bash
# Check status
gui.sh status

# Stop the application
gui.sh stop

# Restart with delay
gui.sh restart 90  # Wait 90 seconds

# Tail logs
gui.sh log 100  # Last 100 lines
```

### 4. Using the bridge

The bridge exposes CoreGUI capabilities to external tools:

```bash
# Wait for bridge to be ready
gui.sh wait

# Then connect via lthn-bridge skill
# The bridge runs on port 9879 by default
```

### 5. Running tests

```bash
# Run all tests
go test -count=1 ./...

# Run display package tests
go test -count=1 ./pkg/display/

# Run example tests
go test -run Example ./...

# Full audit (v0.9.0 + go/frontend tests)
gui.sh audit
```

---

## Core components

### 1. Display package (`pkg/display/`)

The display package manages:

| Component | Purpose | Files |
|-----------|---------|-------|
| `display.go` | Main display service | 67KB |
| `api.go` | Display API core | 18KB |
| `background.go` | Background management | 5.5KB |
| `events.go` | Event handling | 16KB |
| `preload.go` | Preload functionality | 44KB |
| `scheme.go` | Scheme handling | 26KB |
| `storage.go` | Storage management | 9KB |
| `window/` | Window lifecycle | 12KB |

**Test triplets:** Each major component has `_test.go` + `_example_test.go`

### 2. Window package (`pkg/window/`)

Manages window lifecycle:

- **Window management:** Create, destroy, minimize, maximize, restore
- **State persistence:** Save/load window positions, sizes, states
- **Layout engine:** Smart layout, tiling, snapping, stacking
- **WebView integration:** Chrome DevTools Protocol client

From GUI challenges:
- 25 files, 12,575+ lines
- Complete window management system
- Smart layout engine
- State persistence
- WebView integration

### 3. Browser package (`pkg/browser/`)

Handles browser operations:

- **Browser control:** Open URLs, navigate, execute JavaScript
- **Console access:** Read browser console output
- **Evaluation:** Run JavaScript in browser context

---

## Key features

### Window lifecycle

Verbs from lethean-gui skill:

```bash
gui.sh status              # What's running (task dev, lthn proc, vite, bridge)
gui.sh dev                 # Start task dev backgrounded
gui.sh stop                # Clean shutdown
gui.sh restart [SECS]      # Stop + dev + wait (default 90s)
gui.sh log [N]             # Tail last N lines of dev log
gui.sh wait [SECS]         # Block until connected or failed
gui.sh build               # Go build only (compile sanity)
gui.sh audit               # Full v0.9.0 + go/frontend tests
```

### Display orchestration

30+ files covering:
- API wrappers and handlers
- Background tasks
- Clipboard operations
- Context menus
- Display lifecycle
- Event management
- HTML content
- IPC (Inter-Process Communication)
- Manifest handling
- Marketplace integration
- Messages
- ML integration
- Network functionality
- P2P integration
- Scheme handlers
- Sidecar management
- Storage

### Provider integration

CoreGUI mounts and manages:

| Provider | Purpose | Status |
|----------|---------|--------|
| go-ai | AI agent operations | Mounted |
| go-mlx | Apple Metal GPU inference | Mounted |
| go-html | HTML rendering | Mounted |
| go-io | Storage-agnostic I/O | Mounted |
| go-p2p | P2P networking | Mounted |

---

## Getting started

### For agents

1. **Read the RFC:** [`plans/code/core/gui/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/gui/RFC.md)
2. **Explore the display package:** `ls core/gui/go/pkg/display/*.go`
3. **Run tests:** `go test -count=1 ./...`
4. **Check examples:** `go test -run Example ./...`

### For developers

1. **Clone the repo:** `git clone forge.lthn.sh/core/gui`
2. **Start dev:** `task dev` (Wails v2 orchestrator)
3. **Use gui.sh:** Helper script for lifecycle management
4. **Drive via bridge:** Use `lthn-bridge` skill after `gui.sh wait`

---

## Exploring the codebase

### Key files

- [`core/gui/go/pkg/display/display.go`](file:///Users/snider/Code/core/gui/go/pkg/display/display.go) — Main display service
- [`core/gui/go/pkg/display/api.go`](file:///Users/snider/Code/core/gui/go/pkg/display/api.go) — Display API
- [`core/gui/go/pkg/window/`](file:///Users/snider/Code/core/gui/go/pkg/window/) — Window management
- [`core/gui/CLAUDE.md`](file:///Users/snider/Code/core/gui/CLAUDE.md) — Claude-specific notes

### Discovery commands

```bash
# List all display files
ls /Users/snider/Code/core/gui/go/pkg/display/*.go

# Run display tests
go test -count=1 ./pkg/display/

# Run example tests
go test -run Example ./pkg/display/

# Count test triplets
find /Users/snider/Code/core/gui/go/pkg/display/ -name "*_test.go" | wc -l
```

---

## Quick reference

### gui.sh commands

| Command | Description | Example |
|---------|-------------|---------|
| `gui.sh status` | Check running processes | `gui.sh status` |
| `gui.sh dev` | Start development server | `gui.sh dev &` |
| `gui.sh stop` | Clean shutdown | `gui.sh stop` |
| `gui.sh restart [N]` | Stop + dev + wait | `gui.sh restart 90` |
| `gui.sh log [N]` | Tail log lines | `gui.sh log 100` |
| `gui.sh wait [N]` | Block until ready | `gui.sh wait 60` |
| `gui.sh build` | Build binary | `gui.sh build` |
| `gui.sh audit` | Full test suite | `gui.sh audit` |

### Key packages

| Package | Purpose | Location |
|---------|---------|----------|
| `display` | Main display service | `pkg/display/` |
| `window` | Window management | `pkg/window/` |
| `browser` | Browser control | `pkg/browser/` |
| `lifecycle` | Process lifecycle | `pkg/lifecycle/` |
| `clipboard` | Clipboard operations | `pkg/clipboard/` |
| `contextmenu` | Context menus | `pkg/contextmenu/` |
| `events` | Event handling | `pkg/events/` |
| `keybinding` | Keyboard shortcuts | `pkg/keybinding/` |

---

## Use cases

### When to use CoreGUI

- **Desktop applications** — native desktop apps
- **Provider integration** — mount providers for unified action surface
- **Window management** — use display package for window orchestration
- **WebView apps** — Chrome DevTools Protocol integration

### When not to use CoreGUI

- **Server-only applications** — use core/api instead
- **CLI tools** — use core/cli instead
- **Mobile applications** — not supported (Wails is desktop-only)

---

## Related knowledge packs

- [CoreGo](../corego/README.md) — Core Go framework (underlying primitives)
- [CoreTS](../corets/README.md) — TypeScript framework (frontend)
- [CorePlay](../coreplay/README.md) — Play framework (Go + TS)
- [CoreCLI](../corecli/README.md) — CLI framework
- [CorePHP](../corephp/README.md) — PHP framework

---

## Agent tips

1. **Check gui.sh first** — Most lifecycle operations are wrapped
2. **Use bridge for running apps** — Don't poll bridge as build detector
3. **Read dev log** — `gui.sh log` for debugging
4. **Wait for ready** — `gui.sh wait` blocks until app is ready
5. **Restart for Go changes** — Vite hot-reloads frontend, not Go

---

## Maintenance

This knowledge pack is maintained by Mistral Vibe. Updates triggered by:

- Changes to `plans/code/core/gui/`
- Changes to `core/gui/` repository
- New provider integrations
- Wails version updates

---

*Knowledge Pack v1.0.0*
*Created: 2026-06-17T14:30:00Z*
*Author: Mistral Vibe*
*Source: Lethean CoreGUI Framework*
