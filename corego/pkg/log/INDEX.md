---
type: Package Index
package: log
module: dappco.re/go/log
title: go-log Package Index
description: Structured logging and error handling package
---

# go-log Package Index

> **Structured Logging & Error Handling**

**Repository:** `core/go-log`  
**Module:** `dappco.re/go/log`  
**Status:** ✅ Complete Documentation  
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| RFC | Official specification | [plans/code/core/go/log/RFC.md](../../../../../plans/code/core/go/log/RFC.md) |

---

## 🎯 Package Overview

**go-log** provides unified structured logging and error handling for Core applications. It is the **MANDATORY** error handling package for the entire Lethean ecosystem.

### Key Features

- ✅ **Structured Logging** — Key-value pairs for machine-readable logs
- ✅ **Log Levels** — Quiet, Error, Warn, Info, Debug
- ✅ **Structured Errors** — Operation context, codes, recovery metadata
- ✅ **Automatic Redaction** — Sensitive key masking
- ✅ **Log Rotation** — File rotation with size/age/compression
- ✅ **Zero Dependencies** — Only stdlib for production
- ✅ **CoreGO Integration** — Re-exported as core.E(), core.Wrap(), etc.

### Architecture Layers

1. **Application Layer** — Logging and error handling in app code
2. **Logger Layer** — Level control, output, styling, redaction, rotation
3. **Error Layer** — Structured errors with context and introspection

---

## 🏗️ Components

### Logger Types

| Type | File | Purpose |
|------|------|---------|
| `Logger` | `log.go` | Main logging type |
| `Level` | `log.go` | Log level enum |
| `Options` | `log.go` | Logger configuration |
| `RotationOptions` | `log.go` | File rotation config |

### Error Types

| Type | File | Purpose |
|------|------|---------|
| `Err` | `errors.go` | Structured error with context |

### Styling Hooks

| Hook | Purpose |
|------|---------|
| `StyleTimestamp` | Format timestamp prefix |
| `StyleDebug` | Format debug level prefix |
| `StyleInfo` | Format info level prefix |
| `StyleWarn` | Format warning level prefix |
| `StyleError` | Format error level prefix |
| `StyleSecurity` | Format security event prefix |

---

## 📁 File Structure

```
go-log/
├── go/
│   ├── log.go                  # Logger type, levels, key-value formatting, styling
│   ├── errors.go               # Err type, error creators, introspection, recovery
│   ├── service.go              # CoreGO service registration
│   ├── log_test.go             # Logger tests
│   ├── log_example_test.go    # Example-based tests
│   ├── errors_test.go          # Error tests
│   ├── errors_example_test.go # Error examples
│   ├── service_test.go         # Service tests
│   └── service_example_test.go # Service examples
├── go.mod
├── go.sum
└── go.work
```

---

## 🚀 Quick Start

### Basic Setup

```go
import "dappco.re/go/log"

// Configure global logger
log.SetLevel(log.LevelInfo)

// Use package functions
log.Info("server started", "port", 8080)
log.Error("failed", "error", err)
```

### Custom Logger

```go
logger := log.New(log.Options{
    Level:      log.LevelDebug,
    Output:     os.Stdout,
    RedactKeys: []string{"password", "token"},
})

logger.Info("user login", "user", "alice")
logger.Error("auth failed", "error", err)
```

### Error Creation (MANDATORY)

```go
// ALWAYS use E() or Wrap(), never fmt.Errorf
return log.E("user.Save", "failed to save user", err)
return log.Wrap(err, "database.Query", "query failed")
return log.WrapCode(err, "api.Handle", "invalid request", "ERR_INVALID_REQUEST")
```

---

## 🎓 Use Cases

### 1. Application Logging

```go
log.SetLevel(log.LevelInfo)
log.Info("application started", "version", "1.0.0", "env", "production")
log.Error("database error", "error", err, "retryable", true)
```

### 2. Error Handling

```go
func saveUser(user User) error {
    if err := db.Save(user); err != nil {
        return log.Wrap(err, "saveUser", "failed to save")
    }
    return nil
}
```

### 3. HTTP Handler Logging

```go
func handler(w http.ResponseWriter, r *http.Request) {
    log.Debug("request received", "method", r.Method, "path", r.URL.Path)
    
    user, err := getUser(r)
    if err != nil {
        log.Error("request failed", "error", err, "status", 500)
        http.Error(w, "Internal Server Error", 500)
        return
    }
    
    log.Info("request succeeded", "user", user.ID)
}
```

### 4. Redaction

```go
logger := log.New(log.Options{
    RedactKeys: []string{"password", "token", "apikey"},
})

logger.Info("login", "user", "alice", "password", "secret123")
// Output: password=[REDACTED]
```

### 5. Log Rotation

```go
logger := log.New(log.Options{
    Rotation: &log.RotationOptions{
        Filename:   "/var/log/app.log",
        MaxSize:    100,  // MB
        MaxAge:     30,   // days
        MaxBackups: 10,
        Compress:   true,
    },
})
```

---

## 🔧 Configuration

### Options

```go
type Options struct {
    Level      Level           // Log level (default: Info)
    Output     core.Writer     // Output writer (default: stderr)
    Rotation   *RotationOptions // File rotation config
    RedactKeys []string        // Keys to redact
}
```

### RotationOptions

```go
type RotationOptions struct {
    Filename    string // Log file path
    MaxSize     int    // Max size in MB (default: 100)
    MaxAge      int    // Max age in days (default: 28)
    MaxBackups  int    // Max backup files (default: 5)
    Compress    bool   // Gzip compress (default: true)
}
```

---

## 📊 Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/log` |
| **Repository** | `core/go-log` |
| **Language** | Go 1.26+ |
| **Dependencies** | `dappco.re/go` (stdlib only for errors) |
| **Test Triplets** | ✅ Complete |
| **RFC Compliance** | ✅ Verified |
| **Documentation** | ✅ Complete |

---

## ⚠️ Important Notes

### MANDATORY Usage

> **ALL errors MUST use `E()` from go-log, never `fmt.Errorf`**

This is enforced throughout the Lethean codebase. Using `fmt.Errorf` or `errors.New` is considered a bug.

### Re-exports

The `core` package re-exports all logging functions:
- `core.E(op, msg, err)` → `log.E(op, msg, err)`
- `core.Wrap(err, op, msg)` → `log.Wrap(err, op, msg)`
- `core.Print(w, format, args...)` → `log.Print(w, format, args...)`
- `core.Fatal(code, format, args...)` → `log.Fatal(code, format, args...)`

### Zero Dependencies

Production code has zero external dependencies. Only stdlib is used for error handling.

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [core/go](../../) | Re-exports logging functions | ../../ |
| [go-io](../io/) | Log rotation backend | ../io/ |
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
- zero-dependencies
- mandatory
```

---

*Package index generated: 2026-06-17T16:30:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
