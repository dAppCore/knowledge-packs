---
type: Package Documentation
package: webview
module: dappco.re/go/webview
repo: core/go-webview
lang: go
tags:
  - webview
  - browser-automation
  - chrome-devtools-protocol
  - cdp
  - web-scraping
  - gui-automation
  - angular
  - frontend-testing
---
# go-webview — Browser Automation via Chrome DevTools Protocol

> **The authoritative Chrome DevTools Protocol client for interactive browser automation in the Lethean ecosystem**

**RFC:** [plans/code/core/go/webview/RFC.md](../../../../../plans/code/core/go/webview/RFC.md)
**Source:** [~/Code/core/go-webview/](file:///Users/snider/Code/core/go-webview/)
**Module:** `dappco.re/go/webview`
**Dependencies:** `dappco.re/go`, `dappco.re/go/log`, `github.com/gorilla/websocket`

---

## 🎯 Overview

`go-webview` provides **complete browser automation** via Chrome DevTools Protocol (CDP). It enables programmatic control of Chrome/Chromium instances for:

- **Web scraping** — Extract data from dynamic web pages
- **Form automation** — Fill and submit forms programmatically
- **Frontend testing** — Test Angular, React, and other SPAs
- **Screenshot capture** — Capture PNG screenshots of pages
- **Console monitoring** — Capture and filter browser console output
- **JavaScript execution** — Run arbitrary JS in page context
- **Element interaction** — Click, type, scroll, hover, and more

### Primary Use Cases

1. **CoreGUI integration** — Webview components for desktop applications
2. **lthn.ai portal testing** — Automated testing of web interfaces
3. **Agent-driven web scraping** — Data extraction for AI agents
4. **SPA interaction** — Testing Angular, React, Vue applications
5. **Form automation** — Automated form filling and submission

### Design Philosophy

- **Chrome-only** — Focused on Chrome/Chromium (no Playwright, no Firefox)
- **Zero external launch** — Requires Chrome running with `--remote-debugging-port=9222`
- **Single dependency** — Only gorilla/websocket beyond standard library
- **Security-first** — URL validation, size-bounded responses, SSRF protection
- **Action pattern** — 20+ concrete action types for declarative automation
- **Angular-aware** — Specialized helpers for Angular applications

---

## 🏗️ Architecture

### Component Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                           │
│  Browser automation, testing, scraping, form filling          │
├─────────────────────────────────────────────────────────────┤
│                    High-Level API Layer                        │
│  Webview type — Navigate, Click, Type, Query, Screenshot       │
│  Action pattern — 20+ action types, sequence builder          │
│  Console watchers — ConsoleWatcher, ExceptionWatcher          │
│  Angular helpers — Zone.js detection, router navigation      │
├─────────────────────────────────────────────────────────────┤
│                    CDP Transport Layer                         │
│  CDPClient — WebSocket connection management                 │
│  Message framing — Request/response correlation              │
│  Event dispatching — Console, exception, network events       │
│  Target management — Multiple tabs/pages                     │
├─────────────────────────────────────────────────────────────┤
│                    Service Layer                              │
│  Service registration — CoreGO service pattern              │
│  ServiceOptions — DebugURL, Timeout, ConsoleLimit            │
└─────────────────────────────────────────────────────────────┘
```

### Core Components

| Component | File | Purpose |
|-----------|------|---------|
| `Webview` | `webview.go` | Main automation handle with 30+ methods |
| `CDPClient` | `cdp.go` | WebSocket-based CDP transport |
| `Action` interface | `actions.go` | Declarative action pattern (20+ types) |
| `ConsoleWatcher` | `console.go` | Filtered console message capture |
| `ExceptionWatcher` | `console.go` | JavaScript error tracking |
| `Angular` helpers | `angular.go` | Angular-specific automation |
| `Service` | `service.go` | CoreGO service registration |

---

## 📦 Package Structure

```
go-webview/
├── go/
│   ├── webview.go           # Main Webview type + high-level API
│   ├── cdp.go               # CDP client + WebSocket transport
│   ├── actions.go           # Action pattern implementation
│   ├── console.go           # Console/Exception watchers
│   ├── angular.go           # Angular-specific helpers
│   ├── service.go           # CoreGO service registration
│   ├── webview_test.go      # Unit tests
│   ├── webview_example_test.go
│   ├── actions_test.go
│   ├── actions_example_test.go
│   ├── cdp_test.go
│   ├── cdp_example_test.go
│   ├── console_test.go
│   ├── console_example_test.go
│   ├── angular_test.go
│   ├── angular_example_test.go
│   ├── service_test.go
│   └── service_example_test.go
├── go.mod
├── go.sum
└── go.work
```

---

## 🚀 Getting Started

### Prerequisites

```bash
# Start Chrome with remote debugging enabled
chrome --remote-debugging-port=9222 --headless=new

# Or with Docker
docker run -p 9222:9222 -e CHROME_REMOTE_DEBUGGING_PORT=9222 chromium
```

### Basic Usage

```go
package main

import (
    "os"

    core "dappco.re/go"
    "dappco.re/go/webview"
)

func main() {
    r := run()
    if !r.OK {
        core.Print(os.Stderr, "webview example: %v", r.Value)
        os.Exit(1)
    }
}

func run() core.Result {
    // Connect to Chrome DevTools
    wv, err := webview.New(
        webview.WithDebugURL("http://localhost:9222"),
        webview.WithTimeout(30*time.Second),
        webview.WithConsoleLimit(1000),
    )
    if err != nil {
        return core.Fail(err)
    }
    defer wv.Close()

    // Navigate to a page
    if err := wv.Navigate("https://example.com"); err != nil {
        return core.Fail(err)
    }

    // Click a button
    if err := wv.Click("#submit-button"); err != nil {
        return core.Fail(err)
    }

    // Get console output
    logs := wv.GetConsole()
    for _, log := range logs {
        fmt.Printf("[%s] %s\n", log.Type, log.Text)
    }

    return core.Ok(nil)
}
```

### Action Pattern Usage

```go
// Declarative action sequence
actions := []webview.Action{
    &webview.NavigateAction{URL: "https://example.com/login"},
    &webview.TypeAction{Selector: "#username", Text: "agent"},
    &webview.TypeAction{Selector: "#password", Text: "secret"},
    &webview.ClickAction{Selector: "#submit"},
    &webview.WaitAction{Duration: 2 * time.Second},
}

for _, action := range actions {
    if err := action.Execute(ctx, wv); err != nil {
        return err
    }
}
```

### CoreGO Service Integration

```go
c, r := core.New(
    core.WithService(webview.NewService(webview.ServiceOptions{
        DebugURL:     "http://localhost:9222",
        Timeout:      30 * time.Second,
        ConsoleLimit: 1000,
    })),
)
if !r.OK {
    log.Fatal(r.Value)
}

// Access the service
svc := core.MustServiceFor[*webview.Service](c, "webview")

// Use the webview
if err := svc.Webview.Navigate("https://example.com"); err != nil {
    log.Fatal(err)
}
```

---

## 🔧 Core Types

### Webview

The main automation handle.

```go
type Webview struct {
    mu           sync.RWMutex
    client       *CDPClient
    ctx          context.Context
    cancel       context.CancelFunc
    timeout      time.Duration      // Default: 30s
    consoleLogs  []ConsoleMessage
    consoleLimit int                // Default: 1000
}
```

**Constructor:**
```go
func New(opts ...Option) core.Result
```

**Options:**
- `WithDebugURL(url string)` — CDP endpoint URL
- `WithTimeout(d time.Duration)` — Per-operation timeout
- `WithConsoleLimit(n int)` — Max console messages to retain

**Key Methods:**

| Method | Description |
|--------|-------------|
| `Navigate(url)` | Navigate to URL, wait for page load |
| `Click(selector)` | Click element by CSS selector |
| `Type(selector, text)` | Type text into element |
| `QuerySelector(selector)` | Get single element info |
| `QuerySelectorAll(selector)` | Get all matching elements |
| `Screenshot()` | Capture PNG screenshot |
| `Evaluate(script)` | Execute JavaScript |
| `GetURL()` | Get current page URL |
| `GetTitle()` | Get page title |
| `GetHTML(selector)` | Get outer HTML |
| `WaitForSelector(selector)` | Wait for element to appear |
| `SetViewport(w, h)` | Set viewport size |
| `SetUserAgent(ua)` | Override user agent |
| `Reload()` | Refresh page |
| `GoBack()` | Navigate back |
| `GoForward()` | Navigate forward |
| `GetConsole()` | Get captured console messages |
| `ClearConsole()` | Clear console buffer |
| `Close()` | Close connection |

### ConsoleMessage

Captured browser console output.

```go
type ConsoleMessage struct {
    Type      string    // "log", "warn", "error", "info", "debug"
    Text      string    // Message text
    Timestamp time.Time // When logged
    URL       string    // Source URL
    Line      int       // Source line
    Column    int       // Source column
}
```

### ElementInfo

DOM element information.

```go
type ElementInfo struct {
    NodeID      int
    TagName     string
    Attributes  map[string]string
    InnerHTML   string
    InnerText   string
    BoundingBox *BoundingBox
}

type BoundingBox struct {
    X      float64
    Y      float64
    Width  float64
    Height float64
}
```

### CDPClient

Low-level CDP transport.

```go
type CDPClient struct {
    conn     *websocket.Conn
    debugURL string
    // Message tracking
    messageID atomic.Int64
    pending   map[int64]chan *cdpResponse
    // Event handlers
    handlers map[string][]func(map[string]any)
    // Lifecycle
    ctx    context.Context
    cancel context.CancelFunc
}
```

**Methods:**
- `Call(ctx, method, params)` — Send CDP command, await response
- `On(method, handler)` — Register event handler
- `Close()` — Close connection

---

## ⚡ Action Pattern

Declarative automation with 20+ action types.

### Action Interface

```go
type Action interface {
    Execute(ctx context.Context, wv *Webview) error
}
```

### Available Actions

| Action | Description |
|--------|-------------|
| `ClickAction` | Click element by selector |
| `TypeAction` | Type text into element |
| `NavigateAction` | Navigate to URL |
| `WaitAction` | Wait for duration |
| `WaitForSelectorAction` | Wait for element to appear |
| `ScrollAction` | Scroll to coordinates |
| `ScrollIntoViewAction` | Scroll element into view |
| `PressAction` | Press keyboard key |
| `HoverAction` | Hover over element |
| `DragDropAction` | Drag and drop |
| `UploadFileAction` | Upload file to input |
| `SelectAction` | Select dropdown option |
| `CheckAction` | Check checkbox |
| `UncheckAction` | Uncheck checkbox |
| `FocusAction` | Focus element |
| `BlurAction` | Blur element |
| `SetValueAction` | Set input value |
| `GetValueAction` | Get input value |
| `ScreenshotAction` | Capture screenshot |
| `EvaluateAction` | Execute JavaScript |

### Example: Form Submission

```go
actions := []webview.Action{
    &webview.NavigateAction{URL: "https://example.com/login"},
    &webview.WaitForSelectorAction{Selector: "#username"},
    &webview.TypeAction{Selector: "#username", Text: "agent@example.com"},
    &webview.TypeAction{Selector: "#password", Text: "secret123"},
    &webview.ClickAction{Selector: "#submit"},
    &webview.WaitAction{Duration: 2 * time.Second},
}

for _, action := range actions {
    if err := action.Execute(ctx, wv); err != nil {
        return err
    }
}
```

---

## 🎭 Angular Integration

Specialized helpers for Angular applications.

### Angular-Specific Features

```go
// Wait for Angular Zone.js to stabilize
if err := webview.WaitForAngularStable(ctx, wv); err != nil {
    return err
}

// Navigate using Angular router
if err := webview.AngularNavigate(ctx, wv, "/dashboard"); err != nil {
    return err
}

// Get Angular component property
value, err := webview.AngularGetProperty(ctx, wv, "app-root", "userName")

// Set Angular ngModel binding
if err := webview.AngularSetModel(ctx, wv, "[ngModel]='email'", "test@example.com"); err != nil {
    return err
}

// Trigger change detection
if err := webview.AngularTriggerChangeDetection(ctx, wv); err != nil {
    return err
}
```

### Angular Helper Types

```go
// AngularComponent represents an Angular component
type AngularComponent struct {
    Selector    string
    Properties  map[string]any
    Directives  []string
    ChangeDetector bool
}
```

---

## 📡 Console & Exception Watching

### ConsoleWatcher

Filter and capture console messages.

```go
// Create a watcher for error messages only
watcher := webview.NewConsoleWatcher(
    webview.ConsoleWatcherOptions{
        Types: []string{"error", "warn"},
        Limit: 100,
    },
)

// Start watching
watcher.Start(wv)
defer watcher.Stop()

// Get filtered messages
for msg := range watcher.Chan() {
    fmt.Printf("Error: %s at %s:%d\n", msg.Text, msg.URL, msg.Line)
}
```

### ExceptionWatcher

Track JavaScript errors.

```go
exceptionWatcher := webview.NewExceptionWatcher(
    webview.ExceptionWatcherOptions{
        Limit: 50,
    },
)

exceptionWatcher.Start(wv)
defer exceptionWatcher.Stop()

for exc := range exceptionWatcher.Chan() {
    fmt.Printf("JS Error: %s in %s\n", exc.Message, exc.URL)
}
```

---

## 🛡️ Security Features

### URL Validation

All navigation URLs are validated:

```go
func validateNavigationURL(url string) error {
    // Only allow http, https, about:blank
    parsed, err := url.Parse(url)
    if err != nil {
        return err
    }
    
    switch parsed.Scheme {
    case "http", "https", "about":
        return nil
    default:
        return errors.New("invalid URL scheme")
    }
}
```

### Size Bounds

- Max debug response: 1 MB
- Max CDP message: 16 MB
- Console message limit: configurable (default 1000)

### SSRF Protection

- DevTools endpoint must be localhost or loopback
- No external network calls from CDP

---

## 🧪 Testing

### Test Triplets

Each file has `_test.go` + `_example_test.go`:

```
go/
├── webview_test.go              # Unit tests
├── webview_example_test.go     # Example-based tests
├── actions_test.go
├── actions_example_test.go
├── cdp_test.go
├── cdp_example_test.go
├── console_test.go
├── console_example_test.go
├── angular_test.go
├── angular_example_test.go
├── service_test.go
└── service_example_test.go
```

### Running Tests

```bash
cd ~/Code/core/go-webview/go

# Run all tests
go test -v ./...

# Run with coverage
go test -cover ./...

# Run examples
go test -run Example -v
```

---

## 📊 Performance Considerations

### Connection Overhead

- WebSocket connection establishment: ~100ms
- Each CDP call: ~1-10ms (depends on page complexity)

### Memory Usage

- Base: ~10 MB per Webview instance
- Console buffer: ~1 KB per message (configurable limit)

### Parallelism

- Multiple Webview instances can run concurrently
- Each CDP call is serialized per connection
- Use separate connections for parallel operations

---

## 🔗 Integration Points

### CoreGO Framework

```go
import (
    core "dappco.re/go"
    "dappco.re/go/webview"
)

// Service registration
c, r := core.New(
    core.WithService(webview.Register),
    // or with options
    core.WithService(webview.NewService(webview.ServiceOptions{
        DebugURL: "http://localhost:9222",
    })),
)

// Access from core
svc := core.MustServiceFor[*webview.Service](c, "webview")
```

### With Other Packages

```go
// Use with go-log for structured logging
import "dappco.re/go/log"

wv, err := webview.New(webview.WithDebugURL("http://localhost:9222"))
if err != nil {
    log.Error("webview creation failed", "error", err)
    return err
}

// Use with go-io for storage
import "dappco.re/go/io"

png, err := wv.Screenshot()
if err != nil {
    return err
}

r := io.WriteFile("screenshot.png", png)
if !r.OK {
    return r.Value
}
```

---

## 📚 Examples

### Example 1: Simple Navigation and Click

```go
func ExampleWebview_basic() {
    r := webview.New(webview.WithDebugURL("http://localhost:9222"))
    if !r.OK {
        return
    }
    wv := r.Value.(*webview.Webview)
    defer wv.Close()

    wv.Navigate("https://example.com")
    wv.Click("a[href='/about']")
}
```

### Example 2: Form Automation

```go
func ExampleWebview_form() {
    wv, err := webview.New(webview.WithDebugURL("http://localhost:9222"))
    if err != nil {
        return
    }
    defer wv.Close()

    wv.Navigate("https://example.com/login")
    wv.Type("#username", "myuser")
    wv.Type("#password", "mypass")
    wv.Click("#submit")

    // Wait for redirect
    wv.WaitForSelector(".dashboard")
}
```

### Example 3: Screenshot Capture

```go
func ExampleWebview_screenshot() {
    wv, err := webview.New(webview.WithDebugURL("http://localhost:9222"))
    if err != nil {
        return
    }
    defer wv.Close()

    wv.Navigate("https://example.com")
    wv.SetViewport(1920, 1080)

    png, err := wv.Screenshot()
    if err != nil {
        return
    }

    // Save to file
    os.WriteFile("screenshot.png", png, 0644)
}
```

### Example 4: Angular Application Testing

```go
func ExampleWebview_angular() {
    wv, err := webview.New(webview.WithDebugURL("http://localhost:9222"))
    if err != nil {
        return
    }
    defer wv.Close()

    wv.Navigate("https://angular-app.com")

    // Wait for Angular to stabilize
    webview.WaitForAngularStable(context.Background(), wv)

    // Navigate using Angular router
    webview.AngularNavigate(context.Background(), wv, "/products")

    // Get component property
    value, err := webview.AngularGetProperty(
        context.Background(), wv, "app-product-list", "items",
    )
}
```

---

## 🐛 Debugging

### Enable Debug Logging

```go
import "dappco.re/go/log"

// Enable debug logging
log.SetLevel(log.LevelDebug)

wv, err := webview.New(webview.WithDebugURL("http://localhost:9222"))
```

### Common Issues

| Issue | Solution |
|-------|----------|
| Connection refused | Start Chrome with `--remote-debugging-port=9222` |
| WebSocket error | Check if Chrome is running and port is correct |
| Timeout | Increase timeout with `WithTimeout(60*time.Second)` |
| Element not found | Use `WaitForSelector` before interacting |
| SSRF blocked | Use localhost or loopback address |

### Debugging CDP Messages

```go
// Enable CDP message logging
webview.EnableCDPDebug(true)

wv, err := webview.New(webview.WithDebugURL("http://localhost:9222"))
// All CDP messages will be logged
```

---

## 📈 Metrics & Monitoring

### Console Metrics

```go
// Get console message statistics
logs := wv.GetConsole()
stats := make(map[string]int)
for _, log := range logs {
    stats[log.Type]++
}

fmt.Printf("Console stats: %+v\n", stats)
```

### Performance Timing

```go
start := time.Now()
wv.Navigate("https://example.com")
duration := time.Since(start)
fmt.Printf("Navigation took: %v\n", duration)
```

---

## 🎯 Best Practices

### 1. Always Close Connections

```go
wv, err := webview.New(...)
if err != nil {
    return err
}
defer wv.Close() // Always close
```

### 2. Use Timeouts

```go
// Set reasonable timeouts
wv, err := webview.New(
    webview.WithDebugURL("http://localhost:9222"),
    webview.WithTimeout(30*time.Second), // Per-operation timeout
)
```

### 3. Wait for Elements

```go
// Always wait for elements before interacting
wv.Navigate("https://example.com")
wv.WaitForSelector("#submit-button")
wv.Click("#submit-button")
```

### 4. Limit Console Buffer

```go
// Prevent memory exhaustion
wv, err := webview.New(
    webview.WithDebugURL("http://localhost:9222"),
    webview.WithConsoleLimit(1000), // Max messages to retain
)
```

### 5. Use Action Pattern for Complex Workflows

```go
// Declarative automation is more maintainable
actions := []webview.Action{
    &webview.NavigateAction{URL: "https://example.com/login"},
    &webview.TypeAction{Selector: "#username", Text: "user"},
    &webview.TypeAction{Selector: "#password", Text: "pass"},
    &webview.ClickAction{Selector: "#submit"},
}
```

---

## 📝 Notes

- **Repository:** `forge.lthn.sh/core/go-webview`
- **Primary Spec:** [RFC.md](../../../../../plans/code/core/go/webview/RFC.md)
- **Dependencies:** gorilla/websocket for WebSocket transport
- **Security:** URL validation, size bounds, SSRF protection
- **Compatibility:** Chrome/Chromium with DevTools Protocol enabled

---

## 🔍 See Also

- [go-p2p](../p2p/README.md) — Peer-to-peer networking
- [go-proxy](../proxy/README.md) — Mining proxy
- [go-api](../api/README.md) — REST framework
- [CoreGO INDEX](../../INDEX.md) — Complete package catalog
