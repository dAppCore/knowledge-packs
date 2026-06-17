---
type: Package Documentation
package: cache
module: dappco.re/go/cache
repo: core/go-cache
lang: go
tags:
  - cache
  - storage-agnostic
  - json-cache
  - ttl
  - invalidation
  - http-cache
  - path-traversal-protection
---
# go-cache — Storage-Agnostic JSON Caching

> **The authoritative caching layer for CoreGO applications**

**RFC:** [plans/code/core/go/cache/RFC.md](../../../../../plans/code/core/go/cache/RFC.md)
**Source:** [~/Code/core/go-cache/](file:///Users/snider/Code/core/go-cache/)
**Module:** `dappco.re/go/cache`
**Dependencies:** `dappco.re/go`, `dappco.re/go/io`

---

## 🎯 Overview

`go-cache` provides a **storage-agnostic JSON-based caching layer** backed by any `io.Medium`. It implements:

- **JSON serialization** — Values stored as JSON with metadata
- **TTL support** — Automatic expiry for all entries
- **Path-traversal protection** — Safe key validation
- **Invalidation hooks** — Automatic cache invalidation on triggers
- **HTTP caching** — Specialized helpers for HTTP response caching
- **Multiple backends** — Local FS, SQLite, S3, in-memory via `go-io`

### Primary Use Cases

1. **General caching** — Store any JSON-serializable data
2. **HTTP response caching** — Cache API responses with request matching
3. **Scoped caching** — Namespace isolation with collision resistance
4. **GitHub API caching** — Rate limit reduction with smart keys
5. **Performance optimization** — Reduce expensive computations

### Design Philosophy

- **Storage-agnostic** — All I/O delegated to `io.Medium`
- **Security-first** — Path-traversal protection on every operation
- **Lazy eviction** — Expired entries remain until overwritten
- **No external deps** — Pure Go, only core dependencies
- **TTL simplicity** — Single TTL for all entries in a cache instance

---

## 🏗️ Architecture

### Component Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                           │
│  Caching of data, HTTP responses, API results                  │
├─────────────────────────────────────────────────────────────┤
│                    Cache Layer                                 │
│  Cache — Main cache type with Get/Set/Delete operations      │
│  Entry — Serialized cache record with TTL metadata            │
│  ScopedCache — Namespaced cache with collision resistance      │
├─────────────────────────────────────────────────────────────┤
│                    Storage Layer                               │
│  io.Medium — Pluggable storage backend (Local, SQLite, S3)  │
│  CacheStorage — Medium wrapper for cache-specific operations   │
├─────────────────────────────────────────────────────────────┤
│                    HTTP Cache Layer                            │
│  HTTPCache — HTTP response caching with request matching      │
│  CachedRequest — Serialized HTTP request                      │
│  CachedResponse — Serialized HTTP response                    │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Application │────▶│    Cache    │────▶│ io.Medium   │
│  Code        │     │  Instance   │     │ (Local/S3/  │
└─────────────┘     └─────────────┘     │  SQLite)    │
                                          └─────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │   Entry (JSON)  │
                    │  - Data         │
                    │  - CachedAt     │
                    │  - ExpiresAt    │
                    └─────────────────┘
```

---

## 📦 Package Structure

```
go-cache/
├── go/
│   ├── cache.go           # Cache type, Entry, BinaryMeta, core methods
│   ├── service.go         # CoreGO service registration
│   ├── cache_test.go      # Unit tests
│   ├── cache_example_test.go
│   ├── service_test.go    # Service tests
│   └── service_example_test.go
├── go.mod
├── go.sum
└── go.work
```

---

## 🚀 Getting Started

### Basic Cache Setup

```go
package main

import (
    "time"

    core "dappco.re/go"
    coreio "dappco.re/go/io"
    "dappco.re/go/cache"
)

func main() {
    // Create cache with local filesystem backend
    c, r := cache.New(coreio.Local, "/tmp/my-cache", 5*time.Minute)
    if !r.OK {
        core.Fatal(1, "failed to create cache: %v", r.Value)
    }

    // Store a value
    if err := c.Set("my-key", map[string]string{"hello": "world"}); err != nil {
        core.Fatal(1, "failed to set: %v", err)
    }

    // Retrieve a value
    var result map[string]string
    found, err := c.Get("my-key", &result)
    if err != nil {
        core.Fatal(1, "failed to get: %v", err)
    }
    if found {
        fmt.Printf("Got: %+v\n", result)
    }
}
```

### With Different Backends

```go
// SQLite backend
import "dappco.re/go/io/sqlite"

c, _ := cache.New(sqlite.New("cache.db"), "/cache", 1*time.Hour)

// S3 backend
import "dappco.re/go/io/s3"

c, _ := cache.New(s3.New(s3.Config{Bucket: "my-bucket"}), "cache/