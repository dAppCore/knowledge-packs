---
type: Package Index
package: session
module: dappco.re/go/session
title: go-session Package Index
description: Claude Code session parser and analytics package
---

# go-session Package Index

**Repository:** `core/go-session`
**Module:** `dappco.re/go/session`
**Status:** Complete documentation
**Last Updated:** 2026-06-17
**Maintainer:** Purberus <purberus@lthn.ai>

---

## Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| RFC | Official specification | [plans/code/core/go/session/RFC.md](../../../../../plans/code/core/go/session/RFC.md) |

---

## Package overview

**go-session** provides a complete pipeline for parsing, analysing, and rendering Claude Code JSONL transcript files. It implements parsing, analytics, HTML rendering, video rendering, and search functionality.

### Key features

- Claude Code JSONL transcript parser
- Session and Event types with tool correlation
- Analytics engine (tool counts, error rates, latency, tokens)
- HTML renderer (dark theme, collapsible panels, search)
- Video renderer (VHS tape → MP4)
- Cross-session text search
- Session listing and pruning
- Pure library (no CLI, no external deps)

### Architecture layers

1. **Parser layer** — JSONL parsing with tool_use/tool_result correlation
2. **Analytics layer** — Statistics computation and metrics
3. **Rendering layer** — HTML and video output generation
4. **Search layer** — Cross-session text search

---

## Components

### Parser types

| Type | File | Purpose |
|------|------|---------|
| `Event` | `parser.go` | Single action in session timeline |
| `Session` | `parser.go` | Parsed session with metadata and events |
| `rawEntry` | `parser.go` | Raw JSONL entry structure |

### Analytics types

| Type | File | Purpose |
|------|------|---------|
| `SessionAnalytics` | `analytics.go` | Computed statistics for a session |
| `ToolStats` | `analytics.go` | Per-tool statistics |

### Rendering types

| Type | File | Purpose |
|------|------|---------|
| `HTMLOptions` | `html.go` | HTML rendering options |
| `VideoOptions` | `video.go` | Video rendering options |

### Search types

| Type | File | Purpose |
|------|------|---------|
| `SearchResult` | `search.go` | Search result with context |
| `SearchOptions` | `search.go` | Search configuration |

---

## File structure

```
go-session/
├── go/
│   ├── parser.go              # JSONL parser, Session/Event types
│   ├── analytics.go           # Analytics computation
│   ├── html.go                # HTML renderer
│   ├── video.go               # Video renderer
│   ├── search.go              # Cross-session search
│   ├── core_helpers.go        # Helper types and utilities
│   ├── parser_test.go         # Parser tests
│   ├── parser_example_test.go
│   ├── analytics_test.go      # Analytics tests
│   ├── analytics_example_test.go
│   ├── html_test.go           # HTML tests
│   ├── html_example_test.go
│   ├── video_test.go          # Video tests
│   ├── video_example_test.go
│   ├── search_test.go         # Search tests
│   ├── search_example_test.go
│   ├── service.go             # CoreGO service registration
│   ├── service_test.go        # Service tests
│   └── service_example_test.go
├── go.mod
├── go.sum
└── go.work
```

---

## Quick start

### Parse a session

```go
import "dappco.re/go/session"

sess, err := session.ParseFile("/path/to/.claude/session-123.jsonl")
for evt := range sess.EventsSeq() {
    fmt.Printf("[%s] %s: %s\n", evt.Timestamp, evt.Type, evt.Tool)
}
```

### List all sessions

```go
sessions, err := session.ListSessions("/path/to/.claude")
for _, sess := range sessions {
    fmt.Printf("- %s (%d events)\n", sess.ID, len(sess.Events))
}
```

### Compute analytics

```go
analytics := session.ComputeAnalytics(sess)
fmt.Printf("Duration: %v, Errors: %d\n", analytics.Duration, analytics.ErrorCount)
```

### Render to HTML

```go
html, _ := session.RenderHTML(sess, session.HTMLOptions{
    Title: "Session Replay",
    Theme: "dark",
})
os.WriteFile("session.html", []byte(html), 0644)
```

### Search sessions

```go
results, _ := session.SearchSessions(sessions, "error", session.SearchOptions{})
for _, r := range results {
    fmt.Printf("%s: %s\n", r.SessionID, r.Line)
}
```

---

## Use cases

### 1. Session analysis

```go
sess, _ := session.ParseFile("session.jsonl")
analytics := session.ComputeAnalytics(sess)
fmt.Printf("Tool usage: %+v\n", analytics.ToolStats)
```

### 2. Session replay (HTML)

```go
html, _ := session.RenderHTML(sess, session.HTMLOptions{Theme: "dark"})
os.WriteFile("replay.html", []byte(html), 0644)
```

### 3. Video generation

```go
tape, _ := session.RenderVideo(sess, session.VideoOptions{Width: 1280})
os.WriteFile("session.tape", []byte(tape), 0644)
// Run: vhs session.tape session.mp4
```

### 4. Training data extraction

```go
for _, sess := range sessions {
    for evt := range sess.EventsSeq() {
        if evt.Type == "tool_use" {
            // Extract training data
            extractForTraining(evt)
        }
    }
}
```

### 5. Session search

```go
results, _ := session.SearchSessions(sessions, "Bash", session.SearchOptions{
    CaseSensitive: false,
    ContextLines: 2,
})
```

---

## Configuration

### Parser options

```go
// Default parsing
sess, err := session.ParseFile("session.jsonl")

// With validation
sess, err := session.ParseFileValidated("session.jsonl", session.ValidateOptions{
    MaxEvents: 10000,
})
```

### List options

```go
type ListOptions struct {
    MaxAge time.Duration  // Only sessions newer than this
    Limit  int            // Maximum number of sessions
    SortBy string         // "newest", "oldest", "size"
}
```

### Prune options

```go
type PruneOptions struct {
    MaxAge time.Duration  // Delete sessions older than this
    DryRun bool           // Preview without deleting
}
```

---

## Testing

### Test coverage

All files have test triplets (`_test.go` + `_example_test.go`):

| File | Tests |
|------|-------|
| parser_test.go | Parser tests |
| parser_example_test.go | Parser examples |
| analytics_test.go | Analytics tests |
| analytics_example_test.go | Analytics examples |
| html_test.go | HTML tests |
| html_example_test.go | HTML examples |
| video_test.go | Video tests |
| video_example_test.go | Video examples |
| search_test.go | Search tests |
| search_example_test.go | Search examples |
| service_test.go | Service tests |
| service_example_test.go | Service examples |

### Running tests

```bash
cd ~/Code/core/go-session/go

# All tests
go test -v ./...

# With coverage
go test -cover ./...

# Examples only
go test -run Example -v
```

---

## Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/session` |
| **Repository** | `core/go-session` |
| **Language** | Go 1.26+ |
| **Dependencies** | `dappco.re/go` |
| **Test triplets** | Complete |
| **RFC compliance** | Verified |
| **Documentation** | Complete |

---

## Related packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-io](../io/) | File I/O backend | ../io/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |

---

## Performance figures

| Metric | Value |
|--------|-------|
| Parse speed | ~1-2 MB/sec |
| Memory usage | ~2x file size |
| Max line length | 8 MB |
| Max pending tool calls | 4096 |
| Max events per session | 10K+ |

---

*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
