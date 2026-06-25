---
type: Package Documentation
package: log
module: dappco.re/go/log
repo: core/go-log
lang: go
tags:
  - logging
  - structured-logs
  - error-handling
  - severity-levels
  - redaction
  - rotation
  - slog
---
# go-log — Structured Logging & Error Handling

**RFC:** [plans/code/core/go/log/RFC.md](../../../../../plans/code/core/go/log/RFC.md)
**Source:** [~/Code/core/go-log/](file:///Users/snider/Code/core/go-log/)
**Module:** `dappco.re/go/log`
**Dependencies:** `dappco.re/go` (stdlib only for errors)
**Re-exported as:** `core.E()`, `core.Wrap()`, `core.Print()`, etc.

---

## Overview

`go-log` provides **unified structured logging and error handling** for Core applications. It combines:

- **Structured Logging** — JSON/text key-value output with automatic redaction
- **Log Levels** — Quiet, Error, Warn, Info, Debug
- **Error Handling** — Structured errors with operation context, codes, recovery metadata
- **Zero Dependencies** — Only stdlib (except test utilities)

### Primary Use Cases

1. **Application logging** — Structured logs with key-value pairs
2. **Error creation** — `E()` and `Wrap()` for operational context
3. **Log redaction** — Automatic masking of sensitive keys
4. **Log rotation** — File rotation with size/age limits
5. **CoreGO integration** — Re-exported as `core.E()`, `core.Print()`, etc.

### Design Philosophy

- **MANDATORY:** All errors MUST use `E()` from go-log, never `fmt.Errorf`
- **Structured:** Key-value pairs for machine-readable logs
- **Safe:** Automatic redaction of sensitive data
- **Flexible:** Custom styling hooks for colors/icons
- **Zero deps:** Only stdlib for production code

---

## Architecture

### Component Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                           │
│  Structured logging, error handling in application code       │
├─────────────────────────────────────────────────────────────┤
│                    Logger Layer                                │
│  Logger type — Level control, output writer, styling hooks     │
│  Key-value formatting — Structured output with redaction       │
│  Rotation — File rotation with size/age/compression           │
├─────────────────────────────────────────────────────────────┤
│                    Error Layer                                 │
│  Err type — Structured error with Op, Msg, Err, Code           │
│  Error creators — E(), Wrap(), WrapCode(), etc.               │
│  Error introspection — Op(), Code(), Root(), AllOps(), etc.    │
│  Recovery metadata — Retryable, RetryAfter, NextAction         │
└─────────────────────────────────────────────────────────────┘
```

### Integration with Core Framework

```
┌─────────────────────────────────────────────────────────────┐
│                    core package                                │
│  Re-exports: E(), Wrap(), Print(), Fatal(), etc.                │
│  Default logger: Used by core.Result for panic recovery        │
└─────────────────────────────────────────────────────────────┘
```

---

## Package structure

```
go-log/
├── go/
│   ├── log.go           # Logger type, levels, key-value formatting, styling
│   ├── errors.go        # Err type, error creators, introspection, recovery
│   ├── service.go       # CoreGO service registration
│   ├── log_test.go      # Logger tests
│   ├── log_example_test.go
│   ├── errors_test.go   # Error tests
│   ├── errors_example_test.go
│   ├── service_test.go  # Service tests
│   └── service_example_test.go
├── go.mod
├── go.sum
└── go.work
```

---

## Getting started

### Basic Logging

```go
package main

import (
    "os"
    
    core "dappco.re/go"
    "dappco.re/go/log"
)

func main() {
    // Create a logger
    logger := log.New(log.Options{
        Level: log.LevelInfo,
        Output: os.Stdout,
    })

    // Log at different levels
    logger.Debug("debug message", "key", "value")  // Won't print (level is Info)
    logger.Info("server starting", "port", 8080)
    logger.Warn("low disk space", "available", "10%")
    logger.Error("connection failed", "error", err)
}
```

### Using Package-Level Functions

```go
import "dappco.re/go/log"

// Configure global logger
log.SetLevel(log.LevelDebug)

// Use package functions
log.Info("server started", "port", 8080)
log.Error("failed", "error", err)
```

### Error Creation

```go
// ALWAYS use E() or Wrap(), never fmt.Errorf
if err != nil {
    return log.E("user.Save", "failed to save user", err)
}

// Wrap an existing error
if err != nil {
    return log.Wrap(err, "database.Query", "query failed")
}

// With error code
if err != nil {
    return log.WrapCode(err, "api.Handle", "invalid request", "ERR_INVALID_REQUEST")
}
```

---

## Core types

### Level

Log level enum (ordered by verbosity).

```go
type Level int

const (
    LevelQuiet Level = iota  // No output
    LevelError               // Errors only
    LevelWarn                // Warnings + errors
    LevelInfo                // Info + warnings + errors
    LevelDebug               // All messages (most verbose)
)
```

### Logger

Structured logger with flexible configuration.

```go
type Logger struct {
    mu          sync.RWMutex
    writeMu     sync.Mutex
    level       Level          // Current log level
    output      core.Writer     // Output destination
    redactKeys  []string        // Keys to redact
    
    // Styling hooks (for colors/icons)
    StyleTimestamp func(string) string
    StyleDebug     func(string) string
    StyleInfo      func(string) string
    StyleWarn      func(string) string
    StyleError     func(string) string
    StyleSecurity  func(string) string
}
```

**Constructor:**
```go
func New(opts Options) *Logger
```

**Options:**
```go
type Options struct {
    Level      Level           // Log level
    Output     core.Writer     // Output writer (stderr by default)
    Rotation   *RotationOptions // File rotation config
    RedactKeys []string        // Keys to redact
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `SetLevel(level)` | Change log level |
| `SetOutput(w)` | Change output writer |
| `SetRedactKeys(keys...)` | Set keys to redact |
| `Level()` | Get current level |
| `Debug(msg, keyvals...)` | Log debug message |
| `Info(msg, keyvals...)` | Log info message |
| `Warn(msg, keyvals...)` | Log warning message |
| `Error(msg, keyvals...)` | Log error message |
| `Print(w, format, args...)` | Formatted print (used by core.Result) |

### RotationOptions

File rotation configuration.

```go
type RotationOptions struct {
    Filename    string // Log file path
    MaxSize     int    // Max size in MB (default: 100)
    MaxAge      int    // Max age in days (default: 28)
    MaxBackups  int    // Max backup files (default: 5)
    Compress    bool   // Compress backups (default: true)
}
```

### Err

Structured error with operational context.

```go
type Err struct {
    Op          string        // Operation (e.g., "user.Save")
    Msg         string        // Human-readable message
    Err         error         // Underlying error
    Code        string        // Error code
    Retryable   bool          // Can be retried
    RetryAfter  *time.Duration // Suggested retry delay
    NextAction  string        // Alternative action
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `Error()` | String representation |
| `Unwrap()` | Underlying error for error chains |
| `Is(target error)` | Check error equality |

---

## Logging API

### Level-Based Logging

```go
logger := log.New(log.Options{Level: log.LevelInfo})

// These will print (level <= Info)
logger.Info("starting", "service", "api")
logger.Warn("config missing", "key", "database.url")
logger.Error("connection failed", "error", err)

// This won't print (Debug > Info)
logger.Debug("internal state", "value", 42)
```

### Key-Value Pairs

```go
// Multiple key-value pairs
log.Info("user logged in",
    "user", "alice",
    "ip", "192.168.1.1",
    "duration_ms", 150,
)

// Structured data
log.Info("request completed",
    "method", "GET",
    "path", "/api/users",
    "status", 200,
    "duration", time.Second,
)
```

### Output Format

```
[15:04:05] [INF] user logged in user=alice ip=192.168.1.1 duration_ms=150
[15:04:05] [ERR] connection failed error="connection refused"
```

---

## Error handling API

### Error Creation

**MANDATORY:** Always use these functions, never `fmt.Errorf` or `errors.New`.

```go
// Basic error with operation context
return log.E("user.Save", "failed to save user", err)

// Wrap existing error with context
return log.Wrap(err, "database.Query", "query failed")

// With error code
return log.WrapCode(err, "api.Handle", "invalid request", "ERR_INVALID_REQUEST")
```

### Error Wrapping

```go
// Wrap with additional context
func readConfig() core.Result {
    data, err := os.ReadFile("config.json")
    if err != nil {
        return log.Wrap(err, "readConfig", "failed to read config file")
    }
    
    var config Config
    if err := json.Unmarshal(data, &config); err != nil {
        return log.Wrap(err, "readConfig", "failed to parse config")
    }
    
    return core.Ok(config)
}
```

### Error Introspection

```go
// Get operation
op := log.Op(err)  // Returns "user.Save"

// Get error code
code := log.Code(err)  // Returns "ERR_INVALID_REQUEST"

// Get root cause
root := log.Root(err)  // Returns the innermost error

// Get all operations in chain
ops := log.AllOps(err)  // Returns ["api.Handle", "user.Save", "database.Query"]

// Get stack trace
stack := log.FormatStackTrace(err)
```

### Error Chaining

```go
// Check if error is of specific type
if os.IsNotExist(err) {
    // Handle file not found
}

// Check error code
if log.Code(err) == "ERR_INVALID_REQUEST" {
    // Handle specific error
}

// Unwrap for errors.Is/As
var notFoundErr *NotFoundError
if errors.As(err, &notFoundErr) {
    // Handle not found
}
```

---

## Recovery metadata

### Retryable Errors

```go
// Create retryable error
return log.EWithRecovery(
    "api.Call",
    "rate limited",
    err,
    true,  // Retryable
    core.Ptr(5 * time.Second),  // Retry after
    "",     // No next action
)

// Check if retryable
if log.Retryable(err) {
    // Safe to retry
}

// Get retry delay
if delay, ok := log.RetryAfter(err); ok {
    time.Sleep(delay)
}
```

### Next Action

```go
// Suggest alternative action
return log.EWithRecovery(
    "database.Connect",
    "connection refused",
    err,
    false,  // Not retryable
    nil,    // No retry after
    "fallback_to_cache",  // Next action
)

// Get suggested action
if action := log.NextAction(err); action != "" {
    // Try alternative
}
```

---

## Security features

### Automatic Redaction

```go
// Create logger with redaction
logger := log.New(log.Options{
    Level: log.LevelInfo,
    RedactKeys: []string{"password", "token", "apikey", "secret"},
})

// Sensitive values are automatically redacted
logger.Info("login attempt",
    "user", "alice",
    "password", "secret123",  // Will print as [REDACTED]
    "token", "abc123",        // Will print as [REDACTED]
)

// Output: [15:04:05] [INF] login attempt user=alice password=[REDACTED] token=[REDACTED]
```

### Secure Formatting

```go
// All string values are properly quoted
logger.Info("message", "value", "hello\nworld")
// Output: value="hello\nworld"

// Prevents log injection
logger.Info("user input", "input", "\n[INF] fake log entry")
// Output: input="\n[INF] fake log entry" (not a new log line)
```

---

## Log rotation

### Configuration

```go
// Enable file rotation
logger := log.New(log.Options{
    Level: log.LevelInfo,
    Rotation: &log.RotationOptions{
        Filename:    "/var/log/myapp/app.log",
        MaxSize:     100,  // 100 MB
        MaxAge:      28,   // 28 days
        MaxBackups:  5,    // Keep 5 backups
        Compress:    true, // Gzip compress
    },
})
```

### With External Writer Factory

```go
// Set custom rotation writer factory
log.RotationWriterFactory = func(opts log.RotationOptions) core.WriteCloser {
    // Your custom implementation
    return NewCustomRotatingWriter(opts)
}

logger := log.New(log.Options{
    Rotation: &log.RotationOptions{
        Filename: "/var/log/myapp/app.log",
    },
})
```

---

## Styling & customisation

### Level Styling

```go
logger := log.New(log.Options{
    Level: log.LevelInfo,
    Output: os.Stdout,
})

// Custom styling hooks
logger.StyleTimestamp = func(s string) string {
    return "[" + s + "]"
}
logger.StyleDebug = func(s string) string {
    return "[" + ansi.Blue(s) + "]"
}
logger.StyleInfo = func(s string) string {
    return "[" + ansi.Green(s) + "]"
}
logger.StyleWarn = func(s string) string {
    return "[" + ansi.Yellow(s) + "]"
}
logger.StyleError = func(s string) string {
    return "[" + ansi.Red(s) + "]"
}
```

### Package-Level Defaults

```go
// Set global styling
log.StyleTimestamp = func(s string) string {
    return ansi.Gray(s)
}
log.StyleError = func(s string) string {
    return ansi.Red(s)
}

// Now all loggers use these styles
log.Info("styled message")
```

---

## CoreGO integration

### Re-exported Functions

The `core` package re-exports all logging functions:

```go
import core "dappco.re/go"

// These are equivalent to log.*
core.Print(os.Stderr, "error: %v", err)
core.Fatal(1, "fatal error: %v", err)

// Error creation
core.E("op", "message", err)
core.Wrap(err, "op", "message")
```

### Default Logger

The core framework uses go-log internally:

```go
// core.Result uses go-log for panic recovery
r := core.Ok(value)
if !r.OK {
    // Logs error with stack trace
}

// With context
r := core.Fail(log.E("myOp", "failed", err))
```

### Service Registration

```go
import (
    core "dappco.re/go"
    "dappco.re/go/log"
)

c, r := core.New(
    core.WithService(log.NewService(log.ServiceOptions{
        Level: log.LevelDebug,
        Output: os.Stdout,
    })),
)

// Access logger from core
svc := core.MustServiceFor[*log.Service](c, "log")
svc.Logger.Info("service started")
```

---

## Testing

### Test Triplets

Each file has `_test.go` + `_example_test.go`:

```
go/
├── log_test.go              # Logger tests
├── log_example_test.go     # Example-based tests
├── errors_test.go           # Error tests
├── errors_example_test.go # Error examples
├── service_test.go         # Service tests
└── service_example_test.go # Service examples
```

### Running Tests

```bash
cd ~/Code/core/go-log/go

# All tests
go test -v ./...

# With coverage
go test -cover ./...

# Examples only
go test -run Example -v
```

---

## Best practices

### 1. ALWAYS Use Structured Errors

❌ **WRONG:**
```go
return fmt.Errorf("failed to save: %w", err)
```

✅ **RIGHT:**
```go
return log.E("user.Save", "failed to save user", err)
```

### 2. Use Appropriate Log Levels

```go
// Debug: Development/debugging info
log.Debug("cache hit", "key", key, "size", len(data))

// Info: Normal operation messages
log.Info("server started", "port", 8080, "env", "production")

// Warn: Potentially harmful but recoverable
log.Warn("high latency", "duration", 500*time.Millisecond, "threshold", 100*time.Millisecond)

// Error: Failed operations
log.Error("database connection failed", "error", err, "retryable", true)
```

### 3. Always Include Context

❌ **WRONG:**
```go
log.Error("failed")
```

✅ **RIGHT:**
```go
log.Error("failed to connect", "host", host, "port", port, "error", err)
```

### 4. Redact Sensitive Data

```go
logger := log.New(log.Options{
    RedactKeys: []string{"password", "token", "apikey", "secret", "authorization"},
})
```

### 5. Use Error Codes

```go
return log.WrapCode(err, "api.Handle", "invalid request", "ERR_INVALID_REQUEST")
```

### 6. Set Recovery Metadata

```go
return log.EWithRecovery(
    "database.Query",
    "connection timeout",
    err,
    true,                          // Retryable
    core.Ptr(5 * time.Second),     // Retry after
    "use_cache",                   // Next action
)
```

### 7. Configure Log Level for Environment

```go
// Development
log.SetLevel(log.LevelDebug)

// Production
log.SetLevel(log.LevelInfo)  // or LevelWarn for noisy environments

// Quiet mode (errors only)
log.SetLevel(log.LevelError)
```

---

## Examples

### Example 1: Basic Application Logging

```go
package main

import (
    "os"
    "time"

    "dappco.re/go/log"
)

func main() {
    log.SetLevel(log.LevelInfo)
    
    log.Info("starting application", "version", "1.0.0")
    
    if err := run(); err != nil {
        log.Error("application failed", "error", err)
        os.Exit(1)
    }
    
    log.Info("application stopped")
}

func run() error {
    log.Debug("initializing components")
    
    if err := initializeDatabase(); err != nil {
        return log.E("run", "database initialization failed", err)
    }
    
    log.Info("database connected")
    return nil
}
```

### Example 2: Error Handling with Context

```go
func processUser(userID string) error {
    user, err := getUser(userID)
    if err != nil {
        return log.Wrap(err, "processUser", "failed to get user")
    }

    if err := validateUser(user); err != nil {
        return log.WrapCode(err, "processUser", "invalid user", "ERR_INVALID_USER")
    }

    if err := saveUser(user); err != nil {
        return log.E("processUser", "failed to save user", err)
    }

    return nil
}
```

### Example 3: HTTP Handler with Logging

```go
func userHandler(w http.ResponseWriter, r *http.Request) {
    userID := r.URL.Query().Get("id")
    
    log.Debug("handling user request",
        "method", r.Method,
        "path", r.URL.Path,
        "user_id", userID,
    )

    user, err := getUser(userID)
    if err != nil {
        log.Error("user request failed",
            "user_id", userID,
            "error", err,
            "status", 404,
        )
        http.Error(w, "User not found", 404)
        return
    }

    log.Info("user request succeeded", "user_id", userID)
    json.NewEncoder(w).Encode(user)
}
```

### Example 4: Error Introspection

```go
func handleError(err error) {
    // Get operation
    op := log.Op(err)
    log.Error("error occurred", "op", op, "error", err)

    // Check if retryable
    if log.Retryable(err) {
        if delay, ok := log.RetryAfter(err); ok {
            log.Info("retrying after delay", "delay", delay)
            time.Sleep(delay)
            retry()
        }
        return
    }

    // Check next action
    if nextAction := log.NextAction(err); nextAction != "" {
        log.Info("trying alternative", "action", nextAction)
        doAlternative(nextAction)
        return
    }

    // Get all operations in chain
    ops := log.AllOps(err)
    log.Error("error chain", "operations", ops)
}
```

### Example 5: Rotating Logs

```go
func main() {
    logger := log.New(log.Options{
        Level: log.LevelInfo,
        Rotation: &log.RotationOptions{
            Filename:    "/var/log/myapp/app.log",
            MaxSize:     100,  // 100 MB
            MaxAge:      30,   // 30 days
            MaxBackups:  10,  // Keep 10 backups
            Compress:    true,
        },
    })

    // Use logger
    logger.Info("application started")
}
```

---

## Debugging

### Enable Debug Logging

```go
// Enable debug level
log.SetLevel(log.LevelDebug)

// Now all levels will print
log.Debug("debug info", "value", 123)
log.Info("info message")
log.Warn("warning")
log.Error("error")
```

### Check Error Chain

```go
func debugError(err error) {
    // Print full error chain
    fmt.Println("Error:", err)
    fmt.Println("Operations:", log.AllOps(err))
    fmt.Println("Root cause:", log.Root(err))
    fmt.Println("Stack trace:")
    fmt.Println(log.FormatStackTrace(err))
}
```

### Common Issues

| Issue | Solution |
|-------|----------|
| No logs appearing | Check log level is set correctly |
| Sensitive data in logs | Add keys to RedactKeys |
| Error chain lost | Use log.Wrap(), not fmt.Errorf |
| Log injection | go-log prevents this automatically |
| File rotation not working | Set RotationWriterFactory |

---

## Performance considerations

### Logging Overhead

- Debug logging: ~1-2 μs per call
- Info logging: ~500 ns per call
- Error creation: ~200 ns per call
- With redaction: ~500 ns per call (depends on key count)

### Memory Usage

- Logger: ~100 bytes base + redact keys
- Error: ~200 bytes (with stack trace)

### Tips for Production

```go
// Reduce overhead in production
log.SetLevel(log.LevelWarn)  // Only warnings and errors

// Or disable debug completely
log.SetLevel(log.LevelInfo)

// Use buffered writer for file output
logger := log.New(log.Options{
    Output: bufio.NewWriter(os.Stderr),
})
```

---

## Notes

- **Repository:** `forge.lthn.sh/core/go-log`
- **Primary Spec:** [RFC.md](../../../../../plans/code/core/go/log/RFC.md)
- **MANDATORY:** All errors MUST use `E()` from go-log
- **Re-exported:** Available as `core.E()`, `core.Wrap()`, etc.
- **Zero Dependencies:** Only stdlib for production code

---

## Related packages

| Package | Relationship | Path |
|---------|--------------|------|
| [core/go](../../) | Re-exports logging functions | ../../ |
| [go-io](../io/) | Log rotation backend | ../io/ |
| [go-testing](../) | Test utilities | (planned) |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |

---

## Tags

```yaml
- logging
- structured-logs
- error-handling
- severity-levels
- redaction
- rotation
- slog
- structured-errors
- error-chaining
- operational-context
```

---

*Package documentation generated: 2026-06-17T16:30:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
