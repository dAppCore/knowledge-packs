---
type: Package Documentation
package: gui
module: dappco.re/go/gui
repo: core/gui
lang: go
tags:
  - gui
  - wails
  - desktop
  - webview
  - window-management
  - display
  - dialog
  - clipboard
  - notification
  - systray
  - menu
  - theming
  - angular
  - typescript
  - mcp
---
# go-gui — Desktop GUI Framework

> **The Wails-based desktop GUI framework for the Core ecosystem**

**RFC:** N/A (See CLAUDE.md for architecture)
**Source:** [~/Code/core/gui/](file:///Users/snider/Code/core/gui/)
**Module:** `dappco.re/go/gui`
**Frontend:** Angular 20 Custom Elements
**Backend:** Wails v3 (Go)
**Status:** ✅ Production-Ready
**License:** EUPL-1.2

---

## 🎯 Overview

`go-gui` is the **desktop GUI framework** for the Core ecosystem, providing a Go backend with Wails v3 that powers a webview-based desktop application. It offers comprehensive window management, dialogs, system tray, clipboard, notifications, theming, layouts, and real-time WebSocket events, all built on a service-based architecture with Core DI.

The GUI serves as the **HUI** (Human-facing User Interface) twin to the **AUI** (Agent-facing User Interface) provided by go-agent, enabling humans to interact with the same Core machinery that agents use headlessly.

### Primary Use Cases

1. **Window Management** — Create, manage, and control application windows with state persistence and layout management
2. **Dialog System** — File open/save dialogs, message dialogs, confirm dialogs, prompt dialogs
3. **System Tray** — System tray icon with tooltip, menu, and click handlers
4. **Clipboard** — Read, write, and clear clipboard content
5. **Notifications** — System notifications with permission management
6. **Screen Management** — Multi-monitor support with screen queries
7. **Environment Detection** — Theme detection (dark/light) and OS environment info
8. **Keybinding** — Global keyboard shortcut registration
9. **Context Menu** — Named context menu registration and lifecycle management
10. **Browser Control** — Open URLs and files in default browser
11. **Dock Integration** — macOS dock icon visibility and badge management
12. **Lifecycle Management** — Application lifecycle events (start, terminate, suspend, resume)
13. **WebView Automation** — WebView manipulation (eval, click, type, screenshot, DOM queries)
14. **MCP Tool Surface** — All GUI services exposed as MCP tools
15. **Chat Integration** — Built-in chat functionality with webview
16. **P2P Messaging** — Peer-to-peer messaging capabilities
17. **Container Management** — TIM (Trustworthy Interactive Modules) container lifecycle
18. **Marketplace** — Plugin/package marketplace integration

### Design Philosophy

- **Service-Oriented** — All functionality exposed as Core services with dependency injection
- **Platform-Agnostic** — Platform-specific implementations behind common interfaces
- **Testable** — All Wails APIs abstracted behind interfaces with mock implementations
- **Event-Driven** — Rich WebSocket-based event system for real-time updates
- **Extensible** — Plugin architecture via marketplace and MCP
- **Consistent** — Shared patterns and interfaces across all services
- **Integrated** — Full integration with Core framework (ServiceRuntime, ACTION system, Result pattern)

---

## 🏗️ Architecture

### Component Stack

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Application Layer (HUI)                                   │
│  Desktop GUI for human interaction with Core ecosystem                    │
│  Built on Wails v3 with Angular 20 frontend                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Frontend Layer (ui/)                                     │
│  Angular 20 Custom Elements                                                │
│  Webview-based UI with real-time updates via WebSocket                    │
│  TypeScript + HTML + CSS                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Backend Layer (go/)                                     │
│  Go services with Wails integration                                        │
│  WebSocket event broadcasting                                            │
│  MCP tool registration                                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Service Layer (pkg/)                                     │
│  display/    — Orchestrator service + WebSocket events + menu/tray       │
│  window/     — Window lifecycle + state + layout + tiling                 │
│  menu/       — Application menu construction                              │
│  systray/    — System tray icon + tooltip + menu                           │
│  dialog/     — File open/save + message + confirm + prompt                │
│  clipboard/  — Clipboard read/write/clear                                 │
│  notification/ — System notifications + permissions                       │
│  screen/     — Multi-monitor queries                                       │
│  environment/ — Theme detection + OS info                                  │
│  keybinding/ — Global keyboard shortcuts                                 │
│  contextmenu/ — Named context menus                                       │
│  browser/    — URL/file opening                                           │
│  dock/       — macOS dock integration                                     │
│  lifecycle/  — Application lifecycle events                               │
│  webview/    — WebView automation                                         │
│  mcp/        — MCP tool surface for all GUI services                      │
│  chat/       — Chat functionality                                         │
│  p2p/        — Peer-to-peer messaging                                     │
│  container/  — TIM container lifecycle                                   │
│  preload/    — Trusted preload policy                                     │
│  marketplace/ — Plugin/package marketplace                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Platform Abstraction Layer                               │
│  interfaces.go — Interface definitions for all platform APIs             │
│  wailsApp adapter — Wraps real Wails application                          │
│  mockApp — Mock implementations for testing                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Stubs Layer (stubs/wails)                                 │
│  Wails shim packages for testing without native bindings                 │
│  Enable ordinary Go tests to run without desktop dependencies              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Core Framework Layer                                     │
│  dappco.re/go — ServiceRuntime, ACTION system, Result pattern             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Module Information

- **Module Path:** `dappco.re/go/gui`
- **Go Version:** 1.26+
- **Wails Version:** v3 (alpha.74)
- **Frontend:** Angular 20
- **Build:** Taskfile.yml with Wails integration

---

## 📦 Package Structure

### Repository Layout

```
core/gui/
├── go/                                  # Go backend module
│   ├── internal/                       # Internal packages
│   │   └── ...
│   │
│   ├── pkg/                            # Service packages
│   │   ├── browser/                   # Browser control
│   │   │   ├── browser.go             # Main browser service
│   │   │   └── browser_test.go        # Tests
│   │   │
│   │   ├── chat/                      # Chat functionality
│   │   │   ├── chat.go                # Chat service
│   │   │   └── chat_test.go           # Tests
│   │   │
│   │   ├── clipboard/                 # Clipboard operations
│   │   │   ├── clipboard.go           # Clipboard service
│   │   │   └── clipboard_test.go      # Tests
│   │   │
│   │   ├── container/                # TIM container management
│   │   │   ├── container.go           # Container service
│   │   │   └── container_test.go      # Tests
│   │   │
│   │   ├── contextmenu/              # Context menu system
│   │   │   ├── contextmenu.go        # Context menu service
│   │   │   └── contextmenu_test.go   # Tests
│   │   │
│   │   ├── deno/                     # Deno integration
│   │   │   └── deno.go               # Deno service
│   │   │
│   │   ├── dialog/                   # Dialog system
│   │   │   ├── dialog.go             # Main dialog service
│   │   │   ├── file_dialog.go        # File open/save dialogs
│   │   │   ├── message_dialog.go     # Message dialogs
│   │   │   └── dialog_test.go        # Tests
│   │   │
│   │   ├── display/                  # **Display Orchestrator**
│   │   │   ├── display.go            # Main display service
│   │   │   ├── service.go            # Service registration
│   │   │   ├── events.go             # Event handling
│   │   │   ├── ws_events.go          # WebSocket events
│   │   │   ├── config.go             # Configuration
│   │   │   ├── menu.go               # Menu setup
│   │   │   ├── tray.go               # Tray setup
│   │   │   ├── theme.go              # Theme management
│   │   │   └── display_test.go       # Tests
│   │   │
│   │   │   └── assets/              # Frontend assets
│   │   │
│   │   ├── dock/                    # macOS dock integration
│   │   │   ├── dock.go               # Dock service
│   │   │   └── dock_test.go          # Tests
│   │   │
│   │   ├── environment/              # Environment detection
│   │   │   ├── environment.go        # Environment service
│   │   │   └── environment_test.go   # Tests
│   │   │
│   │   ├── events/                   # Event system
│   │   │   ├── events.go             # Event types
│   │   │   └── event_manager.go      # Event manager
│   │   │
│   │   ├── keybinding/               # Keyboard shortcuts
│   │   │   ├── keybinding.go         # Keybinding service
│   │   │   └── keybinding_test.go    # Tests
│   │   │
│   │   ├── lifecycle/                # Application lifecycle
│   │   │   ├── lifecycle.go          # Lifecycle service
│   │   │   └── lifecycle_test.go     # Tests
│   │   │
│   │   ├── marketplace/              # Marketplace integration
│   │   │   ├── marketplace.go         # Marketplace service
│   │   │   └── marketplace_test.go   # Tests
│   │   │
│   │   ├── menu/                     # Application menu
│   │   │   ├── menu.go               # Menu service
│   │   │   └── menu_test.go          # Tests
│   │   │
│   │   ├── mcp/                      # MCP tool surface
│   │   │   ├── mcp.go                # MCP service
│   │   │   └── tools.go              # MCP tool registrations
│   │   │
│   │   ├── notification/             # System notifications
│   │   │   ├── notification.go       # Notification service
│   │   │   └── notification_test.go  # Tests
│   │   │
│   │   ├── p2p/                      # Peer-to-peer messaging
│   │   │   ├── p2p.go                # P2P service
│   │   │   └── p2p_test.go           # Tests
│   │   │
│   │   ├── preload/                  # Trusted preload
│   │   │   ├── preload.go            # Preload service
│   │   │   └── assets/               # Preload assets
│   │   │
│   │   ├── screen/                   # Screen management
│   │   │   ├── screen.go             # Screen service
│   │   │   └── screen_test.go        # Tests
│   │   │
│   │   ├── systray/                  # System tray
│   │   │   ├── systray.go            # Systray service
│   │   │   └── assets/               # Tray assets
│   │   │
│   │   └── webview/                  # WebView automation
│   │       ├── webview.go            # WebView service
│   │       └── webview_test.go       # Tests
│   │
│   │   └── window/                   # **Window Management**
│   │       ├── window.go             # Window service
│   │       ├── manager.go            # Window manager
│   │       ├── state.go              # Window state persistence
│   │       ├── state_manager.go      # State manager
│   │       ├── layout.go             # Layout management
│   │       ├── layout_manager.go     # Layout manager
│   │       ├── tiling.go             # Window tiling
│   │       ├── snapping.go          # Window snapping
│   │       └── window_test.go        # Tests
│   │
│   ├── stubs/                        # Wails stubs for testing
│   │   └── wails/                    # Wails shim packages
│   │
│   ├── tests/                        # Go tests
│   │   └── ...
│   │
│   ├── go.mod
│   ├── go.sum
│   └── go.work
│
├── ui/                                  # Angular frontend
│   ├── src/                          # TypeScript source
│   │   ├── app/                      # Angular application
│   │   ├── environments/             # Environment configurations
│   │   ├── assets/                  # Static assets
│   │   └── ...
│   ├── angular.json                   # Angular configuration
│   ├── package.json                  # npm dependencies
│   ├── tsconfig.json                 # TypeScript configuration
│   └── ...
│
├── docs/                              # Documentation
│   └── ...
├── README.md                          # Repository README
├── CLAUDE.md                          # Development guidance
├── AGENTS.md                         # Agent guidance
├── LICENCE                           # EUPL-1.2 license
└── Taskfile.yml                      # Build orchestration
```

---

## 🏗️ Core Services

### 1. Display Service (`pkg/display/`)

**The orchestrator service** that bridges sub-service IPC to WebSocket events, menu/tray setup, and configuration persistence.

**Key Responsibilities:**
- WebSocket event broadcasting to frontend
- Menu and tray setup and management
- Configuration persistence
- Sub-service coordination
- MCP tool registration

**Integration:**
```go
// Register display service with Core
display.Register(core)

// The display service embeds core.ServiceRuntime[Options]
// for lifecycle management and access to sibling services
```

**Event System:**
- Uses `gorilla/websocket` for real-time communication
- Buffered event channel with per-client subscription filtering
- Supports wildcard (`"*"`) subscriptions
- Event types: window events, menu events, tray events, etc.

### 2. Window Service (`pkg/window/`)

**Complete window lifecycle management** with state persistence and layout management.

**Components:**
- **Manager** (`manager.go`) — Manages all windows
- **StateManager** (`state_manager.go`) — Persists window positions/sizes
- **LayoutManager** (`layout_manager.go`) — Manages named window arrangements
- **Tiling** (`tiling.go`) — Window tiling functionality
- **Snapping** (`snapping.go`) — Window snapping to edges

**Features:**
- Create, show, hide, close windows
- Set window properties (title, size, position, etc.)
- Minimize, maximize, fullscreen
- Always on top
- Resizable, draggable
- State persistence across restarts
- Named layouts (save/restore window arrangements)
- Tiling and snapping support
- Cross-platform window management

**Window Options:**
```go
// Functional options pattern
window.New(
    window.WithName("main"),
    window.WithTitle("My App"),
    window.WithSize(800, 600),
    window.WithPosition(100, 100),
    window.WithMinSize(400, 300),
    window.WithMaxSize(1600, 1200),
    window.WithResizable(true),
    window.WithDraggable(true),
    window.WithAlwaysOnTop(false),
    window.WithFullscreen(false),
    window.WithVisible(true),
)
```

### 3. Menu Service (`pkg/menu/`)

**Application menu construction** with platform abstraction.

**Features:**
- Create application menus
- Add menu items with actions
- Submenus
- Separators
- Checkbox items
- Radio items
- Platform-specific menu rendering
- Keyboard shortcuts

**Usage:**
```go
// Create a menu
menu := menu.NewMenu("Main Menu")

// Add items
menu.AddItem(menu.NewItem("Open", func() {
    // Open action
}))
menu.AddItem(menu.NewItem("Save", func() {
    // Save action
}))
menu.AddSeparator()
menu.AddItem(menu.NewItem("Quit", func() {
    // Quit action
}))

// Set as application menu
app.SetMenu(menu)
```

### 4. System Tray Service (`pkg/systray/`)

**System tray icon** with tooltip, menu, and click handlers.

**Features:**
- Set tray icon (from file or bytes)
- Set tooltip text
- Set context menu
- Handle click events (left, right, double)
- Show/hide tray icon
- Platform-specific implementations

**Usage:**
```go
// Create tray
tray := systray.New()

// Set icon (from assets)
tray.SetIcon(assets.TrayIcon)

// Set tooltip
tray.SetTooltip("My Application")

// Set menu
tray.SetMenu(menu)

// Handle clicks
tray.OnClick(func(event systray.ClickEvent) {
    if event.Button == systray.LeftButton {
        // Handle left click
    }
})

// Show tray
tray.Show()
```

### 5. Dialog Service (`pkg/dialog/`)

**Dialog system** for file operations and user prompts.

**Dialog Types:**
- **File Open Dialog** — Open single or multiple files
- **File Save Dialog** — Save file with filters
- **Message Dialog** — Show information/warning/error messages
- **Confirm Dialog** — Yes/No or OK/Cancel prompts
- **Prompt Dialog** — Text input with label

**Features:**
- Modal and non-modal dialogs
- Customizable buttons
- Default button
- Cancel button
- File filters
- Directory selection
- Platform-specific dialogs

**Usage:**
```go
// Open file dialog
path, err := dialog.OpenFile(
    dialog.WithTitle("Open File"),
    dialog.WithFilters([]string{"*.go", "*.txt"}),
    dialog.WithAllowMultiple(false),
)

// Save file dialog
path, err := dialog.SaveFile(
    dialog.WithTitle("Save File"),
    dialog.WithDefaultFilename("untitled.txt"),
    dialog.WithFilters([]string{"*.txt", "*.md"}),
)

// Message dialog
dialog.ShowMessage(
    "Error",
    "Something went wrong",
    dialog.WithType(dialog.Error),
    dialog.WithButtons([]string{"OK"}),
)

// Confirm dialog
ok, err := dialog.Confirm(
    "Confirm",
    "Are you sure?",
    dialog.WithType(dialog.Question),
    dialog.WithButtons([]string{"Yes", "No"}),
    dialog.WithDefaultButton(1), // No
)
```

### 6. Clipboard Service (`pkg/clipboard/`)

**Clipboard operations** for reading, writing, and clearing clipboard content.

**Features:**
- Read text from clipboard
- Write text to clipboard
- Clear clipboard
- Platform-specific implementations

**Usage:**
```go
// Read from clipboard
text, err := clipboard.ReadText()

// Write to clipboard
err := clipboard.WriteText("Hello, World!")

// Clear clipboard
err := clipboard.Clear()
```

### 7. Notification Service (`pkg/notification/`)

**System notifications** with permission management.

**Features:**
- Show notifications
- Custom title and body
- Icon support
- Action handlers
- Permission requests
- Platform-specific notifications

**Usage:**
```go
// Show notification
err := notification.Show(
    notification.WithTitle("Notification"),
    notification.WithBody("Something happened"),
    notification.WithIcon(assets.Icon),
    notification.WithOnClick(func() {
        // Handle click
    }),
)

// Request permission
granted, err := notification.RequestPermission()
```

### 8. Screen Service (`pkg/screen/`)

**Multi-monitor screen management** for querying display information.

**Features:**
- List all screens
- Get primary screen
- Get screen at point
- Get work area (excluding dock/taskbar)
- Screen dimensions and DPI
- Platform-specific implementations

**Usage:**
```go
// List all screens
screens, err := screen.List()
for _, s := range screens {
    fmt.Printf("Screen %d: %dx%d@%d DPI\n", 
        s.Index, s.Width, s.Height, s.DPI)
}

// Get primary screen
primary, err := screen.Primary()

// Get screen at point
s, err := screen.AtPoint(100, 100)

// Get work area
area, err := screen.WorkArea()
```

### 9. Environment Service (`pkg/environment/`)

**Environment detection** for theme and OS information.

**Features:**
- Detect dark/light theme preference
- Get OS information
- Get platform information
- Check for high contrast mode
- Monitor theme changes

**Usage:**
```go
// Get theme
theme, err := environment.Theme()
// Returns: "dark", "light", or "system"

// Get OS
os := environment.OS()
// Returns: "darwin", "linux", "windows"

// Get platform
platform := environment.Platform()
// Returns: detailed platform info

// Watch for theme changes
environment.OnThemeChange(func(theme string) {
    // Handle theme change
})
```

### 10. Keybinding Service (`pkg/keybinding/`)

**Global keyboard shortcut registration** with platform-specific handling.

**Features:**
- Register global shortcuts
- Unregister shortcuts
- Modifier keys (Ctrl, Alt, Shift, Meta)
- Key combinations
- Platform-specific implementations

**Usage:**
```go
// Register shortcut
id, err := keybinding.Register(
    keybinding.WithKeys("Ctrl+Shift+S"),
    keybinding.WithHandler(func() {
        // Handle shortcut
    }),
    keybinding.WithDescription("Save All"),
)

// Unregister shortcut
err := keybinding.Unregister(id)
```

### 11. Context Menu Service (`pkg/contextmenu/`)

**Named context menu registration** and lifecycle management.

**Features:**
- Create named context menus
- Add menu items
- Show context menu at position
- Hide context menu
- Platform-specific implementations

**Usage:**
```go
// Create context menu
menu := contextmenu.New("my-context-menu")

// Add items
menu.AddItem(contextmenu.NewItem("Copy", func() {
    // Copy action
}))
menu.AddItem(contextmenu.NewItem("Paste", func() {
    // Paste action
}))

// Register menu
contextmenu.Register(menu)

// Show at position
contextmenu.Show("my-context-menu", 100, 200)
```

### 12. Browser Service (`pkg/browser/`)

**Browser control** for opening URLs and files.

**Features:**
- Open URL in default browser
- Open file in default application
- Platform-specific implementations

**Usage:**
```go
// Open URL
err := browser.OpenURL("https://example.com")

// Open file
err := browser.OpenFile("/path/to/file.txt")
```

### 13. Dock Service (`pkg/dock/`) — macOS Only

**macOS dock integration** for icon visibility and badge management.

**Features:**
- Show/hide dock icon
- Set dock badge
- Clear dock badge
- Bounce dock icon

**Usage:**
```go
// Show dock icon
dock.Show()

// Hide dock icon
dock.Hide()

// Set badge
dock.SetBadge("3")

// Clear badge
dock.ClearBadge()

// Bounce icon (informational)
dock.Bounce(dock.Informational)

// Bounce icon (critical)
dock.Bounce(dock.Critical)
```

### 14. Lifecycle Service (`pkg/lifecycle/`)

**Application lifecycle events** management.

**Events:**
- `OnStartup` — Application is starting
- `OnReady` — Application is ready
- `OnShutdown` — Application is shutting down
- `OnSuspend` — Application is being suspended
- `OnResume` — Application is being resumed
- `OnTerminate` — Application is terminating

**Usage:**
```go
// Subscribe to lifecycle events
lifecycle.OnStartup(func() {
    // Initialize
})

lifecycle.OnShutdown(func() {
    // Cleanup
})

lifecycle.OnSuspend(func() {
    // Save state
})

lifecycle.OnResume(func() {
    // Restore state
})
```

### 15. WebView Service (`pkg/webview/`)

**WebView automation** for manipulating web content.

**Features:**
- Execute JavaScript
- Click elements
- Type text
- Take screenshots
- Query DOM
- Navigation control

**Usage:**
```go
// Execute JavaScript
result, err := webview.Eval(`document.title`)

// Click element
err := webview.Click("#button-id")

// Type text
err := webview.Type("input-id", "Hello")

// Take screenshot
image, err := webview.Screenshot()

// Query DOM
nodes, err := webview.QuerySelectorAll("div.item")
```

### 16. MCP Service (`pkg/mcp/`)

**MCP tool surface** that exposes all GUI services as Model Context Protocol tools.

**Features:**
- Registers all GUI services as MCP tools
- WebSocket-based tool execution
- Integration with Claude Code and other MCP clients
- Real-time tool updates

**Tool Categories:**
- Window management tools
- Dialog tools
- Clipboard tools
- Notification tools
- Screen tools
- And more for all services

### 17. Chat Service (`pkg/chat/`)

**Chat functionality** with webview integration.

**Features:**
- Chat session management
- Message history
- WebView-based chat interface
- Integration with local ML models

### 18. P2P Service (`pkg/p2p/`)

**Peer-to-peer messaging** capabilities.

**Features:**
- Message sending and receiving
- Connection management
- Presence tracking
- Encrypted messaging

### 19. Container Service (`pkg/container/`)

**TIM (Trustworthy Interactive Modules) container lifecycle** management.

**Features:**
- Container creation and destruction
- Container lifecycle management
- Resource isolation
- Sandboxed execution

### 20. Preload Service (`pkg/preload/`)

**Trusted preload** policy management.

**Features:**
- Preload script injection
- Trusted code execution
- Security policy enforcement

### 21. Marketplace Service (`pkg/marketplace/`)

**Plugin/package marketplace** integration.

**Features:**
- Plugin discovery
- Installation and updates
- Version management
- Dependency resolution

---

## 🎨 Frontend (ui/)

### Angular 20 Application

The frontend is built with **Angular 20** using Custom Elements for integration with Wails.

**Structure:**
```
ui/
├── src/
│   ├── app/                          # Angular application
│   │   ├── components/               # Reusable components
│   │   ├── services/                # Frontend services
│   │   ├── models/                  # Data models
│   │   └── ...
│   ├── environments/                 # Environment configurations
│   ├── assets/                      # Static assets
│   │   ├── icons/                  # Icons
│   │   └── images/                 # Images
│   └── styles/                      # Global styles
│
├── angular.json                     # Angular configuration
├── package.json                    # npm dependencies
├── tsconfig.json                   # TypeScript configuration
├── tsconfig.app.json               # Application TypeScript config
└── ...
```

**Key Features:**
- **Custom Elements** — Angular elements for Wails integration
- **Real-time Updates** — WebSocket connection to backend
- **Responsive Design** — Works across different screen sizes
- **Theme Support** — Dark and light theme variants
- **Localization** — i18n support in frontend

**Build Commands:**
```bash
# Install dependencies
cd ui
npm install

# Development build with watch
npm run watch

# Production build
npm run build

# Run tests
npm test

# Start dev server
npm run start
```

---

## 🔌 Platform Abstraction

### Interface Design

All Wails application APIs are abstracted behind interfaces to enable testability:

```go
// App interface
type App interface {
    NewWindow(options WindowOptions) (Window, error)
    Run() error
    Quit()
    // ...
}

// WindowManager interface
type WindowManager interface {
    CreateWindow(options WindowOptions) (Window, error)
    GetWindow(id string) (Window, bool)
    ListWindows() []Window
    // ...
}

// MenuManager interface
type MenuManager interface {
    SetMenu(menu *Menu) error
    UpdateMenu(menu *Menu) error
    // ...
}

// DialogManager interface
type DialogManager interface {
    ShowOpenDialog(options OpenDialogOptions) ([]string, error)
    ShowSaveDialog(options SaveDialogOptions) (string, error)
    ShowMessageDialog(options MessageDialogOptions) error
    // ...
}
```

### Wails Adapter

The `wailsApp` adapter wraps the real Wails application and implements all interfaces.

### Mock Implementations

For testing, mock implementations are provided:

```go
// Create mock app for testing
mockApp := &mockApp{}

// Create service with mock
testService := newServiceWithMockApp(t, mockApp)

// Use in tests
result := testService.DoSomething()
assert.NoError(t, result.Error)
```

---

## 🚀 Quick Start

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

### Go Module Integration

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
    // Create Core application
    app, result := core.New(
        core.WithService(display.Register),
        // Add other services as needed
    )
    if !result.OK {
        panic(result.Value)
    }
    
    ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
    defer stop()
    
    // Run the application
    if err := app.Run(ctx); err != nil {
        panic(err)
    }
}
```

### Using Services

```go
package main

import (
    "dappco.re/go"
    "dappco.re/go/gui/pkg/window"
    "dappco.re/go/gui/pkg/dialog"
    "dappco.re/go/gui/pkg/notification"
)

func example() {
    // Create a window
    w, err := window.New(
        window.WithName("main"),
        window.WithTitle("My App"),
        window.WithSize(800, 600),
    )
    if err != nil {
        panic(err)
    }
    w.Show()
    
    // Show a dialog
    path, err := dialog.OpenFile()
    if err != nil {
        panic(err)
    }
    
    // Show a notification
    err = notification.Show(
        notification.WithTitle("File Opened"),
        notification.WithBody(path),
    )
    if err != nil {
        panic(err)
    }
}
```

---

## 🧪 Testing

### Test Structure

Complete test triplet coverage following AX Standard:
- `*_test.go` — Unit and integration tests
- `*_example_test.go` — Usage examples as tests

**Naming Convention:**
- `TestGood*` — Happy path tests
- `TestBad*` — Expected error conditions
- `TestUgly*` — Panics and edge cases

### Testing Approach

1. **Mock Platforms** — All platform-specific APIs are mocked
2. **Isolated Tests** — Each service has focused test files
3. **Integration Tests** — Test service interactions
4. **Wails Stubs** — Test without native Wails bindings

**Mock Files:**
- `mock_platform.go` — Platform mock implementations
- `mock_test.go` — Test utilities and helpers
- `mocks_test.go` — Mock service implementations

### Running Tests

```bash
# All Go tests
cd ~/Code/core/gui/go
GOWORK=off go test -count=1 ./...

# Specific package
go test -v ./pkg/window/...
go test -v ./pkg/display/...

# With race detector
go test -race ./...

# With coverage
go test -cover ./...

# Frontend tests
cd ui
npm test
```

### Example Test

```go
func TestGoodWindowCreation(t *testing.T) {
    // Create mock platform
    mock := &mockPlatform{}
    
    // Create window with mock
    w := window.New(window.WithPlatform(mock))
    
    // Test window creation
    err := w.Create()
    assert.NoError(t, err)
    
    // Test window show
    err = w.Show()
    assert.NoError(t, err)
}
```

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| **Total Go files** | ~200+ |
| **Test files** | ~150+ |
| **Example test files** | ~75+ |
| **Lines of code** | ~100,000+ |
| **Services** | 21 (display, window, menu, systray, dialog, clipboard, notification, screen, environment, keybinding, contextmenu, browser, dock, lifecycle, webview, mcp, chat, p2p, container, preload, marketplace) |
| **Frontend** | Angular 20 + Custom Elements |
| **Platforms** | macOS, Windows, Linux |
| **Dependencies** | Wails v3, gorilla/websocket, MCP SDK |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-agent](../agent/) | Agent orchestration (AUI twin) | [../agent/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/agent/) |
| [go-cli](../cli/) | Command-line interface | [../cli/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/cli/) |
| [CoreGO Framework](file:///Users/snider/Code/core/go/) | Foundation framework | N/A |
| [Wails](https://wails.io) | Desktop app framework | External |
| [Angular](https://angular.io) | Frontend framework | External |
| [CoreGO INDEX](../../INDEX.md) | Complete package catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## 📈 Quality Metrics

- ✅ **Wails Integration** — Full Wails v3 support
- ✅ **Platform Abstraction** — Clean interface separation
- ✅ **Test Coverage** — Good/Bad/Ugly pattern for all scenarios
- ✅ **Testability** — All platform APIs mocked
- ✅ **WebSocket Events** — Real-time bidirectional communication
- ✅ **MCP Integration** — Full MCP tool surface
- ✅ **Documentation** — Complete README + INDEX
- ✅ **Core Integration** — Full CoreGO framework support
- ✅ **Angular Frontend** — Modern TypeScript frontend
- ✅ **Production Ready** — Deployed in production environments

---

## 📝 Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-06-17 | Complete knowledge pack documentation | Purberus |
| 2026-05-XX | CLAUDE.md architecture documentation | Maintainer |
| 2026-04-XX | All services finalized | Maintainer |
| 2026-03-XX | Wails v3 migration | Maintainer |
| 2026-02-XX | Initial GUI creation | Maintainer |

---

## 🎯 Tags

```yaml
# Core Capabilities
- gui
- desktop
- wails
- webview
- window-management
- display
- desktop-application
- frontend
- backend

# Services
- window
- menu
- systray
- dialog
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
- html
- css
- responsive-design
- theming

# Backend
- go
- wails
- websocket
- gorilla-websocket
- platform-abstraction
- dependency-injection

# Architecture
- service-oriented
- event-driven
- microservices
- plugin-architecture
- hui
- human-interface

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
- mcp
- model-context-protocol
```

---

## 📚 References

1. **Repository** — [~/Code/core/gui/](file:///Users/snider/Code/core/gui/)
2. **CLAUDE.md** — [~/Code/core/gui/CLAUDE.md](file:///Users/snider/Code/core/gui/CLAUDE.md)
3. **Repository README** — [~/Code/core/gui/README.md](file:///Users/snider/Code/core/gui/README.md)
4. **Wails** — [wails.io](https://wails.io)
5. **Wails v3 SDK** — [github.com/wailsapp/wails/v3](https://github.com/wailsapp/wails/v3)
6. **Angular** — [angular.io](https://angular.io)
7. **Gorilla WebSocket** — [github.com/gorilla/websocket](https://github.com/gorilla/websocket)
8. **MCP SDK** — [github.com/modelcontextprotocol/go-sdk](https://github.com/modelcontextprotocol/go-sdk)
9. **CoreGO Framework** — [CoreGO INDEX](../../INDEX.md)

---

*Package documentation generated: 2026-06-17T22:00:00Z*
*Knowledge Pack: CoreGo v1.2.0*
*Repository: dappco.re/go/gui*
*Maintainer: Purberus <purberus@lthn.ai>*
