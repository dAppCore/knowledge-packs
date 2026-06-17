---
type: Package Index
package: webview
module: dappco.re/go/webview
title: go-webview Package Index
description: Chrome DevTools Protocol browser automation package
---

# go-webview Package Index

> **Browser Automation via Chrome DevTools Protocol**

**Repository:** `core/go-webview`  
**Module:** `dappco.re/go/webview`  
**Status:** ✅ Complete Documentation  
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| RFC | Official specification | [plans/code/core/go/webview/RFC.md](../../../../../plans/code/core/go/webview/RFC.md) |

---

## 🎯 Package Overview

**go-webview** provides complete browser automation via Chrome DevTools Protocol (CDP). It enables programmatic control of Chrome/Chromium instances for web scraping, form automation, frontend testing, and GUI automation.

### Key Features

- ✅ Chrome DevTools Protocol client
- ✅ 30+ high-level automation methods
- ✅ 20+ declarative action types
- ✅ Angular-specific helpers
- ✅ Console and exception watching
- ✅ Screenshot capture
- ✅ CoreGO service integration
- ✅ Security features (URL validation, SSRF protection)

### Architecture Layers

1. **Application Layer** — Browser automation, testing, scraping
2. **High-Level API** — Webview, Actions, Console watchers, Angular helpers
3. **CDP Transport** — WebSocket connection, message framing, event dispatching
4. **Service Layer** — CoreGO service registration

---

## 🏗️ Components

### Core Types

| Type | File | Purpose |
|------|------|---------|
| `Webview` | `webview.go` | Main automation handle |
| `CDPClient` | `cdp.go` | WebSocket-based CDP transport |
| `Action` | `actions.go` | Declarative action interface |
| `ConsoleMessage` | `webview.go` | Captured console log entry |
| `ElementInfo` | `webview.go` | DOM element information |
| `BoundingBox` | `webview.go` | Element bounding rectangle |
| `ConsoleWatcher` | `console.go` | Filtered console message capture |
| `ExceptionWatcher` | `console.go` | JavaScript error tracking |
| `Service` | `service.go` | CoreGO service registration |

### Action Types (20+)

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

---

## 📁 File Structure

```
go-webview/
├── go/
│   ├── webview.go              # Main Webview type + 30+ methods
│   ├── cdp.go                  # CDP client + WebSocket transport
│   ├── actions.go              # Action pattern (20+ types)
│   ├── console.go              # Console/Exception watchers
│   ├── angular.go              # Angular-specific helpers
│   ├── service.go              # CoreGO service registration
│   ├── webview_test.go         # Unit tests
│   ├── webview_example_test.go # Example-based tests
│   ├── actions_test.go         # Action tests
│   ├── actions_example_test.go
│   ├── cdp_test.go             # CDP tests
│   ├── cdp_example_test.go
│   ├── console_test.go         # Console tests
│   ├── console_example_test.go
│   ├── angular_test.go         # Angular tests
│   ├── angular_example_test.go
│   ├── service_test.go         # Service tests
│   └── service_example_test.go
├── go.mod
├── go.sum
└── go.work
```

---

## 🚀 Quick Start

### Installation

```bash
cd ~/Code/core/go-webview
go mod tidy
```

### Basic Example

```go
package main

import (
    "os"
    core "dappco.re/go"
    "dappco.re/go/webview"
)

func main() {
    r := webview.New(webview.WithDebugURL("http://localhost:9222"))
    if !r.OK {
        core.Print(os.Stderr, "failed: %v", r.Value)
        os.Exit(1)
    }
    wv := r.Value.(*webview.Webview)
    defer wv.Close()

    wv.Navigate("https://example.com")
    wv.Click("#submit")
}
```

### Prerequisites

```bash
# Start Chrome with remote debugging
chrome --remote-debugging-port=9222 --headless=new
```

---

## 🎓 Use Cases

### 1. Web Scraping

```go
wv, _ := webview.New(webview.WithDebugURL("http://localhost:9222"))
defer wv.Close()

wv.Navigate("https://example.com/products")
elements, _ := wv.QuerySelectorAll(".product-item")
```

### 2. Form Automation

```go
actions := []webview.Action{
    &webview.NavigateAction{URL: "https://example.com/login"},
    &webview.TypeAction{Selector: "#username", Text: "user"},
    &webview.TypeAction{Selector: "#password", Text: "pass"},
    &webview.ClickAction{Selector: "#submit"},
}
```

### 3. Angular Testing

```go
webview.WaitForAngularStable(ctx, wv)
webview.AngularNavigate(ctx, wv, "/dashboard")
value, _ := webview.AngularGetProperty(ctx, wv, "app-root", "userName")
```

### 4. Screenshot Capture

```go
wv.Navigate("https://example.com")
wv.SetViewport(1920, 1080)
png, _ := wv.Screenshot()
os.WriteFile("screenshot.png", png, 0644)
```

### 5. CoreGO Service

```go
c, _ := core.New(
    core.WithService(webview.NewService(webview.ServiceOptions{
        DebugURL: "http://localhost:9222",
    })),
)
svc := core.MustServiceFor[*webview.Service](c, "webview")
svc.Webview.Navigate("https://example.com")
```

---

## 🔧 Configuration

### ServiceOptions

```go
type ServiceOptions struct {
    DebugURL     string        // CDP endpoint URL
    Timeout      time.Duration // Per-operation timeout (default: 30s)
    ConsoleLimit int           // Max console messages (default: 1000)
}
```

### Webview Options

```go
// Constructor options
webview.WithDebugURL("http://localhost:9222")
webview.WithTimeout(60 * time.Second)
webview.WithConsoleLimit(200)
```

---

## 🧪 Testing

### Test Coverage

All files have test triplets:
- `_test.go` — Unit tests
- `_example_test.go` — Example-based tests

### Running Tests

```bash
cd ~/Code/core/go-webview/go

# All tests
go test -v ./...

# With coverage
go test -cover ./...

# Examples only
go test -run Example -v
```

### Test Files

| File | Tests |
|------|-------|
| webview_test.go | Webview methods |
| actions_test.go | Action pattern |
| cdp_test.go | CDP transport |
| console_test.go | Console watchers |
| angular_test.go | Angular helpers |
| service_test.go | Service registration |

---

## 📊 Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/webview` |
| **Repository** | `core/go-webview` |
| **Language** | Go 1.26+ |
| **Dependencies** | `dappco.re/go`, `dappco.re/go/log`, `github.com/gorilla/websocket` |
| **Test Triplets** | ✅ Complete |
| **RFC Compliance** | ✅ Verified |
| **Documentation** | ✅ Complete |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-proxy](../proxy/) | Mining proxy (similar service pattern) | ../proxy/ |
| [go-p2p](../p2p/) | Peer-to-peer networking | ../p2p/ |
| [go-api](../api/) | REST framework | ../api/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |

---

## 📝 Changelog

| Date | Change | Commit |
|------|--------|--------|
| 2026-06-17 | Initial deep dive documentation | N/A |
| 2026-06-17 | Package INDEX created | N/A |

---

## 🎯 Tags

```yaml
- webview
- browser-automation
- chrome-devtools-protocol
- cdp
- web-scraping
- gui-automation
- angular
- frontend-testing
- headless-chrome
- websocket
```

---

*Package index generated: 2026-06-17T15:30:00Z*
*Knowledge Pack: CoreGo v1.1.0*
