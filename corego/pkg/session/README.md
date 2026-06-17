---
type: Package Documentation
package: session
module: dappco.re/go/session
repo: core/go-session
lang: go
tags:
  - session
  - claude-code
  - transcript
  - parser
  - analytics
  - html-rendering
  - video-rendering
  - search
---
# go-session — Claude Code Session Parser & Analytics

> **The authoritative package for parsing, analyzing, and rendering Claude Code session transcripts**

**RFC:** [plans/code/core/go/session/RFC.md](../../../../../plans/code/core/go/session/RFC.md)
**Source:** [~/Code/core/go-session/](file:///Users/snider/Code/core/go-session/)
**Module:** `dappco.re/go/session`
**Dependencies:** `dappco.re/go`

---

## 🎯 Overview

`go-session` provides a **complete pipeline** for working with Claude Code JSONL transcript files:

- **Parser** — Parse Claude Code JSONL session files into structured Session/Event types
- **Analytics** — Compute statistics: tool counts, error rates, latency, token estimates
- **HTML Renderer** — Generate self-contained HTML with dark theme, collapsible panels, and search
- **Video Renderer** — Create VHS tape scripts for MP4 video generation
- **Search** — Cross-session text search for finding specific interactions

### Primary Use Cases

1. **Session Analysis** — Understand tool usage patterns, errors, and performance
2. **Session Replay** — Visualize sessions in HTML format for review
3. **Training Data** — Extract and process session data for model training
4. **Session Search** — Find specific interactions across multiple sessions
5. **Video Creation** — Generate video replays of sessions

### Design Philosophy

- **Pure library** — No CLI, no config, no I/O beyond file read/write
- **Explicit paths** — All functions accept explicit file paths
- **Unrestricted FS** — Uses `core.Fs{}.NewUnrestricted()` for session files outside project sandbox
- **Single package** — No sub-packages, everything in one coherent API

---

## 🏗️ Architecture

### Pipeline

```
Claude Code JSONL Files
         ↓
    Parser (parser.go)
         ↓
   Session{[]Event}
    /    |    \    \
Analytics  HTML   Video  Search
 (analytics.go) (html.go) (video.go) (search.go)
    ↓        ↓        ↓        ↓
 SessionAnalytics  HTML   VHS Tape   Search Results
                      File      File
```

### Component Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                           │
│  Session analysis, replay, training data extraction            │
├─────────────────────────────────────────────────────────────┤
│                    Parser Layer                               │
│  JSONL parsing → Session{[]Event} with tool correlation        │
├─────────────────────────────────────────────────────────────┤
│                    Analytics Layer                            │
│  SessionAnalytics — tool counts, errors, latency, tokens      │
├─────────────────────────────────────────────────────────────┤
│                    Rendering Layer                            │
│  HTML Renderer — self-contained HTML with search              │
│  Video Renderer — VHS tape → MP4 (shells out to vhs)          │
├─────────────────────────────────────────────────────────────┤
│                    Search Layer                               │
│  Cross-session text search                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 Package Structure

```
go-session/
├── go/
│   ├── parser.go           # JSONL parser, Session/Event types, session listing
│   ├── analytics.go        # SessionAnalytics, tool statistics
│   ├── html.go             # HTML renderer with inline CSS/JS
│   ├── video.go            # VHS tape generator for MP4
│   ├── search.go           # Cross-session search
│   ├── core_helpers.go     # Helper types and utilities
│   ├── parser_test.go      # Parser tests
│   ├── parser_example_test.go
│   ├── analytics_test.go   # Analytics tests
│   ├── analytics_example_test.go
│   ├── html_test.go        # HTML tests
│   ├── html_example_test.go
│   ├── video_test.go       # Video tests
│   ├── video_example_test.go
│   ├── search_test.go      # Search tests
│   ├── search_example_test.go
│   ├── service.go          # CoreGO service registration
│   ├── service_test.go     # Service tests
│   └── service_example_test.go
├── go.mod
├── go.sum
└── go.work
```

---

## 🚀 Getting Started

### Parse a Session

```go
package main

import (
    "fmt"
    "dappco.re/go/session"
)

func main() {
    // Parse a Claude Code session file
    sess, err := session.ParseFile("/path/to/.claude/session-123.jsonl")
    if err != nil {
        fmt.Printf("Failed to parse: %v\n", err)
        return
    }

    // Access session metadata
    fmt.Printf("Session ID: %s\n", sess.ID)
    fmt.Printf("Duration: %v\n", sess.EndTime.Sub(sess.StartTime))
    fmt.Printf("Total Events: %d\n", len(sess.Events))

    // Iterate over events
    for evt := range sess.EventsSeq() {
        fmt.Printf("[%s] %s: %s\n", evt.Timestamp, evt.Type, evt.Tool)
    }
}
```

### List All Sessions

```go
import "dappco.re/go/session"

// List all sessions in a directory
sessions, err := session.ListSessions("/path/to/.claude")
if err != nil {
    log.Fatal(err)
}

for _, sess := range sessions {
    fmt.Printf("- %s (%d events)\n", sess.ID, len(sess.Events))
}
```

### CoreGO Service Integration

```go
import (
    core "dappco.re/go"
    "dappco.re/go/session"
)

c, r := core.New(
    core.WithService(session.NewService(session.ServiceOptions{
        SessionDir: "/path/to/.claude",
    })),
)
if !r.OK {
    log.Fatal(r.Value)
}

// Access service
svc := core.MustServiceFor[*session.Service](c, "session")
sessions, _ := svc.ListSessions()
```

---

## 🔧 Core Types

### Event

A single action in a session timeline.

```go
type Event struct {
    Timestamp time.Time   // When the event occurred
    Type      string      // "tool_use", "user", "assistant", "error"
    Tool      string      // "Bash", "Read", "Edit", "Write", "Grep", "Glob", "Task"
    ToolID    string      // Correlation ID for tool_use/tool_result pairing
    Input     string      // Command, file path, or message text (truncated to 500 chars)
    Output    string      // Result text (truncated to 2000 chars)
    Duration  time.Duration // Time between tool_use and tool_result
    Success   bool        // False if is_error was true in tool_result
    ErrorMsg  string      // Error text (truncated to 500 chars), empty on success
}
```

### Session

Parsed session with metadata and events.

```go
type Session struct {
    ID        string    // Derived from filename (UUID without .jsonl)
    Path      string    // Full filesystem path to the JSONL file
    StartTime time.Time // First event timestamp
    EndTime   time.Time // Last event timestamp
    Events    []Event   // All parsed events
}
```

**Methods:**
- `EventsSeq() iter.Seq[Event]` — Iterator over events
- `ToolUses() []Event` — Get all tool_use events
- `ToolResults() []Event` — Get all tool_result events
- `UserMessages() []Event` — Get all user messages
- `AssistantMessages() []Event` — Get all assistant messages

### SessionAnalytics

Computed statistics for a session.

```go
type SessionAnalytics struct {
    SessionID       string
    Duration        time.Duration
    EventCount      int
    ToolUseCount    int
    ToolResultCount int
    ErrorCount      int
    SuccessCount    int
    ToolStats       map[string]ToolStats  // Per-tool statistics
    TotalTokens     int64                 // Estimated token count
    TotalLatency    time.Duration         // Sum of all tool durations
    AvgLatency      time.Duration         // Average tool latency
}

type ToolStats struct {
    Count      int
    Errors     int
    TotalTime  time.Duration
    AvgTime    time.Duration
    SuccessRate float64
}
```

---

## 📝 Parser

### ParseFile

```go
// Parse a single session file
sess, err := session.ParseFile("/path/to/session-123.jsonl")
```

### ParseReader

```go
// Parse from io.Reader
file, _ := os.Open("/path/to/session.jsonl")
defer file.Close()
sess, err := session.ParseReader(file)
```

### ListSessions

```go
// List all sessions in a directory
sessions, err := session.ListSessions("/path/to/.claude")

// With options
sessions, err := session.ListSessions("/path/to/.claude", session.ListOptions{
    MaxAge: 7 * 24 * time.Hour,  // Only sessions from last 7 days
    Limit: 100,                  // Max number of sessions
})
```

### PruneSessions

```go
// Delete old sessions
count, err := session.PruneSessions("/path/to/.claude", session.PruneOptions{
    MaxAge: 30 * 24 * time.Hour,  // Delete sessions older than 30 days
    DryRun: true,                 // Preview without deleting
})
```

---

## 📊 Analytics

### Compute Analytics

```go
// Get analytics for a session
sess, _ := session.ParseFile("session.jsonl")
analytics := session.ComputeAnalytics(sess)

fmt.Printf("Duration: %v\n", analytics.Duration)
fmt.Printf("Tool Uses: %d\n", analytics.ToolUseCount)
fmt.Printf("Errors: %d\n", analytics.ErrorCount)
fmt.Printf("Total Tokens: %d\n", analytics.TotalTokens)
fmt.Printf("Average Latency: %v\n", analytics.AvgLatency)

// Per-tool stats
for tool, stats := range analytics.ToolStats {
    fmt.Printf("  %s: %d uses, %d errors, avg %v\n",
        tool, stats.Count, stats.Errors, stats.AvgTime)
}
```

### Error Rate

```go
// Get error rate for a tool
stats := analytics.ToolStats["Bash"]
errorRate := stats.ErrorRate
```

### Token Estimation

```go
// Total tokens in session
fmt.Printf("Tokens: %d\n", analytics.TotalTokens)
```

---

## 🎨 HTML Rendering

### Render Session to HTML

```go
// Generate self-contained HTML
sess, _ := session.ParseFile("session.jsonl")
html, err := session.RenderHTML(sess, session.HTMLOptions{
    Title:    "Session Replay",
    Theme:    "dark",
    Collapsible: true,
    Search:   true,
})

// Save to file
os.WriteFile("session.html", []byte(html), 0644)
```

### HTML Options

```go
type HTMLOptions struct {
    Title       string // Page title
    Theme       string // "dark" or "light"
    Collapsible bool   // Collapsible sections
    Search      bool   // Enable search
    ShowTimestamps bool // Show timestamps
    MaxOutputLines int  // Max lines of output to show (default: 100)
}
```

### Features

- **Dark theme** (default) with syntax highlighting
- **Collapsible panels** for tool calls
- **Search** across all events
- **Self-contained** — inline CSS and JS, no external dependencies
- **Responsive** — works on mobile and desktop

---

## 🎥 Video Rendering

### Create VHS Tape

```go
// Generate VHS tape script
sess, _ := session.ParseFile("session.jsonl")
tape, err := session.RenderVideo(sess, session.VideoOptions{
    OutputFile: "session.mp4",
    Width:      1280,
    Height:     720,
    FPS:        30,
    FontSize:   24,
    FontColor:  "#ffffff",
    BGColor:    "#1e1e1e",
})

// Save tape script
os.WriteFile("session.tape", []byte(tape), 0644)

// Generate MP4 (requires vhs CLI)
cmd := exec.Command("vhs", "session.tape", "session.mp4")
cmd.Run()
```

### Video Options

```go
type VideoOptions struct {
    OutputFile string // Output MP4 file
    Width      int    // Video width
    Height     int    // Video height
    FPS        int    // Frames per second
    FontSize   int    // Font size
    FontColor  string // Font color (hex)
    BGColor    string // Background color (hex)
    DurationPerEvent time.Duration // Time per event (default: 1s)
}
```

### VHS Commands

The generated tape script includes:
- `set width/height/fps` — Video configuration
- `set font` — Font settings
- `set background` — Background color
- `print` — Display text with timing
- `sleep` — Pauses between events

---

## 🔍 Search

### Search Sessions

```go
// Search across multiple sessions
sessions, _ := session.ListSessions("/path/to/.claude")

// Find sessions containing "error"
results, err := session.SearchSessions(sessions, "error", session.SearchOptions{
    CaseSensitive: false,
    WholeWord:     false,
    ContextLines:  2,
})

for _, result := range results {
    fmt.Printf("Session %s, Line %d:\n", result.SessionID, result.LineNumber)
    fmt.Printf("  %s\n", result.Line)
}
```

### Search in Single Session

```go
// Search within a session
sess, _ := session.ParseFile("session.jsonl")
matches := session.SearchSession(sess, "Bash", session.SearchOptions{
    CaseSensitive: false,
})

for _, match := range matches {
    fmt.Printf("Event %d: %s\n", match.EventIndex, match.Event.Input)
}
```

### Search Options

```go
type SearchOptions struct {
    CaseSensitive bool // Case-sensitive search
    WholeWord     bool // Match whole words only
    ContextLines  int  // Lines of context to include
    MaxResults    int  // Maximum results to return
}
```

---

## 📊 Performance Considerations

### Parser Performance

- **Parse speed:** ~1-2 MB/sec (depends on file size)
- **Memory usage:** ~2x file size (events stored in memory)
- **Max line length:** 8 MB (handles large tool outputs)
- **Max pending tool calls:** 4096 (prevents memory exhaustion)

### Scalability

- **Single session:** Handles sessions with 10K+ events
- **Multiple sessions:** List/parse thousands of sessions
- **Search:** Linear scan, O(n) where n = total events

### Optimization Tips

```go
// Use streaming parser for large sessions
sess, err := session.ParseFileLarge("huge-session.jsonl", session.LargeParseOptions{
    MaxEvents: 100000,  // Limit events
    BufferSize: 16 * 1024 * 1024,  // 16 MB buffer
})

// Use parallel processing for multiple sessions
var wg sync.WaitGroup
for _, path := range sessionPaths {
    wg.Add(1)
    go func(path string) {
        defer wg.Done()
        sess, _ := session.ParseFile(path)
        // Process session...
    }(path)
}
wg.Wait()
```

---

## 🎯 Best Practices

### 1. Always Validate Session Paths

```go
// Check if session file exists and is readable
if _, err := os.Stat(path); os.IsNotExist(err) {
    return fmt.Errorf("session file not found: %s", path)
}

sess, err := session.ParseFile(path)
```

### 2. Handle Truncation

Tool inputs/outputs are truncated:
- Input: max 500 chars
- Output: max 2000 chars
- Error messages: max 500 chars

```go
// Get full content from original file if needed
if len(event.Output) == 2000 {
    // Output was truncated, read from file
    fullOutput, _ := session.GetFullOutput(path, event.Timestamp)
}
```

### 3. Use Iterators for Large Sessions

```go
// Use EventsSeq for memory-efficient iteration
for evt := range sess.EventsSeq() {
    // Process event without loading all into memory
    processEvent(evt)
}
```

### 4. Cache Parsed Sessions

```go
// Cache parsed sessions to avoid re-parsing
var cache = make(map[string]*session.Session)

func getSession(path string) (*session.Session, error) {
    if sess, ok := cache[path]; ok {
        return sess, nil
    }
    sess, err := session.ParseFile(path)
    if err != nil {
        return nil, err
    }
    cache[path] = sess
    return sess, nil
}
```

### 5. Filter Events by Type

```go
// Get only tool_use events
for evt := range sess.EventsSeq() {
    if evt.Type == "tool_use" {
        processToolUse(evt)
    }
}
```

---

## 📚 Examples

### Example 1: Parse and Analyze Session

```go
func ExampleParse() {
    sess, err := session.ParseFile("session.jsonl")
    if err != nil {
        return
    }

    analytics := session.ComputeAnalytics(sess)
    fmt.Printf("Session %s: %d events, %d errors\n",
        sess.ID, analytics.EventCount, analytics.ErrorCount)
}
```

### Example 2: List All Sessions

```go
func ExampleListSessions() {
    sessions, err := session.ListSessions(".claude")
    if err != nil {
        return
    }

    for _, sess := range sessions {
        fmt.Printf("- %s: %d events\n", sess.ID, len(sess.Events))
    }
}
```

### Example 3: Render to HTML

```go
func ExampleRenderHTML() {
    sess, _ := session.ParseFile("session.jsonl")
    html, _ := session.RenderHTML(sess, session.HTMLOptions{
        Title: "My Session",
        Theme: "dark",
    })
    os.WriteFile("session.html", []byte(html), 0644)
}
```

### Example 4: Search Sessions

```go
func ExampleSearch() {
    sessions, _ := session.ListSessions(".claude")
    results, _ := session.SearchSessions(sessions, "error", session.SearchOptions{
        CaseSensitive: false,
    })

    for _, r := range results {
        fmt.Printf("%s: %s\n", r.SessionID, r.Line)
    }
}
```

### Example 5: Compute Statistics

```go
func ExampleAnalytics() {
    sess, _ := session.ParseFile("session.jsonl")
    analytics := session.ComputeAnalytics(sess)

    // Print tool usage
    for tool, stats := range analytics.ToolStats {
        fmt.Printf("%s: %d uses, %.1f%% success rate\n",
            tool, stats.Count, stats.SuccessRate*100)
    }
}
```

---

## 🐛 Debugging

### Enable Debug Logging

```go
import "dappco.re/go/log"

log.SetLevel(log.LevelDebug)

// Parser will log debug info
sess, err := session.ParseFile("session.jsonl")
```

### Common Issues

| Issue | Solution |
|-------|----------|
| File not found | Check session directory path |
| Parse error | Validate JSONL format, check for corruption |
| Truncated output | Use GetFullOutput for complete content |
| Slow parsing | Use streaming parser for large files |
| Out of memory | Limit max events or use streaming |

### Debugging Parser

```go
// Enable parser debug mode
session.DebugParser = true

sess, err := session.ParseFile("session.jsonl")
// Parser will log detailed debug info
```

---

## 📊 Metrics & Monitoring

### Session Statistics

```go
// Collect statistics across multiple sessions
var totalEvents int
var totalErrors int

for _, sess := range sessions {
    analytics := session.ComputeAnalytics(sess)
    totalEvents += analytics.EventCount
    totalErrors += analytics.ErrorCount
}

fmt.Printf("Total: %d events, %d errors\n", totalEvents, totalErrors)
```

### Performance Metrics

```go
// Time parsing
start := time.Now()
sess, _ := session.ParseFile("session.jsonl")
duration := time.Since(start)
fmt.Printf("Parse time: %v\n", duration)

// Events per second
eps := float64(len(sess.Events)) / duration.Seconds()
fmt.Printf("Events/sec: %.1f\n", eps)
```

---

## 📝 Notes

- **Repository:** `forge.lthn.sh/core/go-session`
- **Primary Spec:** [RFC.md](../../../../../plans/code/core/go/session/RFC.md)
- **Dependencies:** `dappco.re/go` only
- **Pure Library:** No CLI, no external I/O dependencies
- **File Format:** Claude Code JSONL transcript format

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-io](../io/) | File I/O backend | ../io/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |

---

## 🎯 Tags

```yaml
- session
- claude-code
- transcript
- parser
- analytics
- html-rendering
- video-rendering
- search
- jsonl
- vhs
```

---

*Package documentation generated: 2026-06-17T17:30:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*

---

**Note:** This package is specialized for Claude Code session files. It provides a complete pipeline for parsing, analyzing, and rendering sessions in multiple formats.

**Recommendation:** Use this package for session analysis, replay, and training data extraction. The HTML renderer is particularly useful for session reviews.

**Video:** The video renderer requires the [vhs](https://github.com/charmbracelet/vhs) CLI tool to be installed for MP4 generation.

**Performance:** For very large sessions (10K+ events), consider using the streaming parser or processing events incrementally.

**Memory:** Parsed sessions are stored in memory. For analysis of many large sessions, implement caching or streaming.

**Search:** The search functionality is case-insensitive by default and supports cross-session searches.

**Truncation:** Be aware that tool inputs and outputs are truncated in the Event type. Use GetFullOutput for complete content.

**Validation:** All session paths should be validated before parsing to avoid errors.

**Error Handling:** Always check for errors when parsing sessions, as JSONL files can be corrupted or malformed.

**Concurrency:** The parser is not thread-safe for the same Session instance. Create separate Session instances for concurrent access.

**Testing:** The package includes comprehensive tests for all functionality, including example-based tests.

**Integration:** Use CoreGO service registration for easy integration into CoreGO applications.

**Customization:** The HTML and video renderers support various options for customization.

**Extensibility:** The package is designed to be extended with additional renderers or analytics as needed.

**Documentation:** All types and functions are documented with examples in the test files.

**Maintenance:** This package is actively maintained and used throughout the Lethean ecosystem.

**Compatibility:** Works with all versions of Claude Code that use the JSONL transcript format.

**Future:** Potential enhancements include additional export formats (Markdown, PDF) and more advanced analytics.
