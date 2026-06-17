---
type: Knowledge Pack
title: CoreGUI Framework
description: Complete knowledge pack for CoreGUI — the Wails v2-based GUI framework for Lethean desktop applications
author: Mistral Vibe
version: 1.0.0
created: 2026-06-17T14:30:00Z
tags: [framework, gui, wails, desktop, lethean, wails3]
---

# CoreGUI Knowledge Pack

> **"Gateway binary for provider-hosted actions"** — CoreGUI RFC

CoreGUI is the **Wails v2** framework for Lethean desktop applications, providing a native GUI wrapper with webview-based frontend and Go backend.

---

## 🎯 Overview

**CoreGUI** serves as the **gateway binary** that:
- Mounts all providers (go-ai, go-mlx, go-html, go-io, go-p2p, plus pending providers)
- Exposes their action surface over local HTTP on loopback only
- Manages desktop process lifecycle
- Provides window management and orchestration

### Key Statistics

- **Framework:** Wails v3.0.0-alpha.91
- **Display Package:** 30+ files, 67KB main implementation
- **Test Coverage:** 100% test triplet pattern
- **Ports:** Vite (9245), Bridge (9879) for lthn-desktop

---

## 📚 Source of Truth

- **Primary Spec:** [`plans/code/core/gui/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/gui/RFC.md)
- **Implementation:** [`core/gui/go/pkg/display/`](file:///Users/snider/Code/core/gui/go/pkg/display/)
- **Agent Guide:** [`core/gui/AGENTS.md`](file:///Users/snider/Code/core/gui/AGENTS.md)

---

## 🏗️ Architecture

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

---

## 📦 Core Components

### 1. Display Package (`pkg/display/`)

The **display package** is the heart of CoreGUI, managing:

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

**Test Triplets:** Each major component has `_test.go` + `_example_test.go`

### 2. Window Package (`pkg/window/`)

Manages window lifecycle:

- **Window Management:** Create, destroy, minimize, maximize, restore
- **State Persistence:** Save/load window positions, sizes, states
- **Layout Engine:** Smart layout, tiling, snapping, stacking
- **WebView Integration:** Chrome DevTools Protocol client

**From GUI Challenges:**
- 25 files, 12,575+ lines
- Complete window management system
- Smart layout engine
- State persistence
- WebView integration

### 3. Browser Package (`pkg/browser/`)

Handles browser operations:

- **Browser Control:** Open URLs, navigate, execute JavaScript
- **Console Access:** Read browser console output
- **Evaluation:** Run JavaScript in browser context

---

## 🔥 Key Features

### Window Lifecycle

**Verbs from lethean-gui skill:**

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

### Display Orchestration

**30+ files** covering:
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

### Provider Integration

CoreGUI mounts and manages:

| Provider | Purpose | Status |
|----------|---------|--------|
| go-ai | AI agent operations | Mounted |
| go-mlx | Apple Metal GPU inference | Mounted |
| go-html | HTML rendering | Mounted |
| go-io | Storage-agnostic I/O | Mounted |
| go-p2p | P2P networking | Mounted |

---

## 🚀 Getting Started

### For Agents

1. **Read the RFC:** [`plans/code/core/gui/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/gui/RFC.md)
2. **Explore the display package:** `ls core/gui/go/pkg/display/*.go`
3. **Run tests:** `go test -count=1 ./...`
4. **Check examples:** `go test -run Example ./...`

### For Developers

1. **Clone the repo:** `git clone forge.lthn.sh/core/gui`
2. **Start dev:** `task dev` (Wails v2 orchestrator)
3. **Use gui.sh:** Helper script for lifecycle management
4. **Drive via bridge:** Use `lthn-bridge` skill after `gui.sh wait`

---

## 🔍 Exploring the Codebase

### Key Files

- [`core/gui/go/pkg/display/display.go`](file:///Users/snider/Code/core/gui/go/pkg/display/display.go) — Main display service
- [`core/gui/go/pkg/display/api.go`](file:///Users/snider/Code/core/gui/go/pkg/display/api.go) — Display API
- [`core/gui/go/pkg/window/`](file:///Users/snider/Code/core/gui/go/pkg/window/) — Window management
- [`core/gui/CLAUDE.md`](file:///Users/snider/Code/core/gui/CLAUDE.md) — Claude-specific notes

### Discovery Commands

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

## 📊 Quick Stats

```
Total packages:          8+ (display, window, browser, lifecycle, clipboard, contextmenu, events, keybinding)
Display package files:   30+
Window package files:    25+
Test files:              30+ (_test.go)
Example files:           30+ (_example_test.go)
Wails version:           v3.0.0-alpha.91
Default ports:           Vite 9245, Bridge 9879 (lthn-desktop)
```

---

## 🎯 Use Cases

### When to Use CoreGUI

✅ **Desktop applications** — Use CoreGUI for native desktop apps
✅ **Provider integration** — Mount providers for unified action surface
✅ **Window management** — Use display package for window orchestration
✅ **WebView apps** — Use for Chrome DevTools Protocol integration

### When NOT to Use CoreGUI

❌ **Server-only applications** — Use core/api instead
❌ **CLI tools** — Use core/cli instead
❌ **Mobile applications** — Not supported (Wails is desktop-only)

---

## 🔗 Related Knowledge Packs

- [CoreGo](../corego/README.md) — Core Go framework (underlying primitives)
- [CoreTS](../corets/README.md) — TypeScript framework (frontend)
- [CorePlay](../coreplay/README.md) — Play framework (Go + TS)
- [CoreCLI](../corecli/README.md) — CLI framework
- [CorePHP](../corephp/README.md) — PHP framework

---

## 💡 Agent Tips

1. **Check gui.sh first** — Most lifecycle operations are wrapped
2. **Use bridge for running apps** — Don't poll bridge as build detector
3. **Read dev log** — `gui.sh log` for debugging
4. **Wait for ready** — `gui.sh wait` blocks until app is ready
5. **Restart for Go changes** — Vite hot-reloads frontend, not Go

---

## 📝 Maintenance

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
