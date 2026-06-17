---
type: Package Deep Dive
title: go-store — SQLite Key-Value Store with Multi-Layer Architecture
description: Complete documentation for go-store — SQLite-backed key-value store with TTL, namespace isolation, reactive events, DuckDB workspaces, InfluxDB journal, and cold archive
description: Pure Go SQLite KV store with TTL expiry (triple-layered), namespace isolation via ScopedStore, reactive Watch/OnChange events, DuckDB workspace buffering, InfluxDB time-series journal, compressed JSONL cold archive, and io.Medium transport abstraction
module: dappco.re/go/store
repo: core/go-store
tags: [storage, sqlite, kv, key-value, ttl, namespace, events, duckdb, influxdb, journal, archive, medium, io]
lang: go
author: Mistral Vibe
version: 1.0.0
created: 2026-06-18T05:00:00Z
---

# go-store — SQLite Key-Value Store with Multi-Layer Architecture

> **"An agent should be able to implement this library from this document alone."**

`dappco.re/go/store` is the **SQLite-backed key-value store** with **TTL expiry**, **namespace isolation**, **reactive events**, **DuckDB workspace buffering**, **InfluxDB time-series journal**, and **cold archive** capabilities for the Lethean agent platform. Pure Go (no CGO).

Used by `core/ide` for memory caching, by agents for workspace state, and across the platform for durable, portable storage that integrates with the CoreGO framework's `core.Result` and `io.Medium` transport abstraction.

---

## 🎯 Overview

### What it is

- **SQLite Key-Value Store** — CRUD operations on `(group, key)` compound primary keys with TTL expiry
- **Namespace Isolation** — `ScopedStore` wraps `*Store` with namespace prefixes and optional quota enforcement
- **Reactive Events** — `Watch()` channels (buffered, non-blocking) and `OnChange()` synchronous callbacks
- **TTL Expiry** — Triple-layered enforcement: lazy delete on `Get`, query-time filtering, background purge goroutine
- **DuckDB Workspaces** — Mutable accumulation buffers for work-in-progress, atomic commit to journal
- **InfluxDB Journal** — Time-series persistence for completed work units with Flux query support
- **Cold Archive** — Compressed JSONL (gzip/zstd) for aged journal entries
- **io.Medium Integration** — Hot-swap storage backends: local filesystem, S3, memory, encrypted DataCube, SFTP
- **SQL Injection Protection** — `escapeLike()` with `^` escape char for LIKE queries
- **Single-Connection Design** — `MaxOpenConns(1)` because SQLite pragmas (WAL, busy_timeout) are per-connection

### The Four-Layer Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      go-store LAYERS                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           LAYER 4: COLD ARCHIVE (Immutable)                │   │
│  │  ┌──────────────┐  ┌──────────────┐                       │   │
│  │  │ Compressed   │  │   JSONL      │                       │   │
│  │  │ JSONL.gz     │  │   Format     │                       │   │
│  │  │ or zstd      │  │              │                       │   │
│  │  └──────────────┘  └──────────────┘                       │   │
│  │  Long-term storage, training data, CDN publishing           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           LAYER 3: JOURNAL (Append-Only)                  │   │
│  │  ┌──────────────┐  ┌──────────────┐                       │   │
│  │  │ InfluxDB    │  │ Time-Series  │                       │   │
│  │  │ Client      │  │ Journal      │                       │   │
│  │  └──────────────┘  └──────────────┘                       │   │
│  │  Immutable record of completed work units                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           LAYER 2: WORKSPACE (Mutable)                      │   │
│  │  ┌──────────────┐  ┌──────────────┐                       │   │
│  │  │ DuckDB      │  │ Mutable      │                       │   │
│  │  │ Buffer      │  │ Accumulation │                       │   │
│  │  └──────────────┘  └──────────────┘                       │   │
│  │  Atomic commit → Journal + Identity Store                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           LAYER 1: IDENTITY STORE (Mutable)                │   │
│  │  ┌──────────────┐  ┌──────────────┐                       │   │
│  │  │ SQLite      │  │ Key-Value    │                       │   │
│  │  │ Database    │  │ with TTL     │                       │   │
│  │  └──────────────┘  └──────────────┘                       │   │
│  │  Permanent identity, current summary                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────┘

Data Flow:
  Workspace (DuckDB buffer) → Journal (InfluxDB time-series) → Identity (SQLite KV)
  │
  ┌─→ Cold Archive (compressed JSONL) ←── Journal (retention policy)
```

### The Problem & Solution

**Problem:** Writing every micro-event directly to time-series makes deltas meaningless — 4000 writes of "+1" produces noise. A mutable buffer accumulates work, then commits once as a complete unit. The time-series only sees finished work, so deltas between entries represent real change.

**Solution:** Four-layer architecture:
1. **Identity Store (SQLite):** "this thing exists" — identity, current summary
2. **Workspace Buffer (DuckDB):** "this thing is working" — mutable temp state, atomic
3. **Journal (InfluxDB):** "this thing completed" — immutable, delta-ready
4. **Cold Archive (JSONL.gz):** "this thing is archived" — compressed, portable

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     go-store PACKAGE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐         │
│  │   store.go    │    │   events.go   │    │   scope.go    │         │
│  │   Core KV     │    │   Watch/      │    │   ScopedStore │         │
│  │   + TTL       │    │   OnChange    │    │   + Quota     │         │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘         │
│         │                  │                  │                    │
│  ┌──────▼───────┐    ┌──────▼───────┐    ┌──────▼───────┐      │
│  │  workspace.go │    │  journal.go   │    │  compact.go   │      │
│  │  DuckDB      │    │  InfluxDB    │    │  JSONL.gz     │      │
│  │  Buffer      │    │  Journal     │    │  Archive      │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    io.Medium Integration                        │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐   │   │
│  │  │ Local  │ │   S3    │ │ Memory │ │  Cube   │ │  SFTP   │   │   │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📦 Quick Start

### Basic Store Usage

```go
import "dappco.re/go/store"

// Create an in-memory store
result := store.New(":memory:")
if !result.OK {
    log.Fatal(result.Error())
}
st := result.Value.(*Store)
defer st.Close()

// Set and Get values
st.Set("config", "colour", "blue")
value, result := st.Get("config", "colour")
if result.OK {
    fmt.Println(value) // "blue"
}

// Set with TTL (5 minutes)
st.SetWithTTL("session", "token", "abc123", 5*time.Minute)

// Get with automatic TTL expiry (lazy delete)
value, result = st.Get("session", "token") // Returns value if not expired
```

### With Full Configuration

```go
import "dappco.re/go/store"

// Configure store with journal and custom settings
result := store.NewConfigured(store.StoreConfig{
    DatabasePath:            "/path/to/store.db",
    Journal:                 store.JournalConfiguration{
        EndpointURL:  "http://localhost:8086",
        Organisation: "core",
        BucketName:   "events",
    },
    PurgeInterval:           30 * time.Second,
    WorkspaceStateDirectory: ".core/state/",
})
if !result.OK {
    log.Fatal(result.Error())
}
st := result.Value.(*Store)
defer st.Close()
```

### With io.Medium Backend

```go
import (
    "dappco.re/go/store"
    "dappco.re/go/core/io"
)

// Store backed by S3
s3Medium := io.S3("garage.lthn.io/bucket")
result := store.New(":memory:", store.WithMedium(s3Medium))
if !result.OK {
    log.Fatal(result.Error())
}
st := result.Value.(*Store)

// Store backed by encrypted DataCube
cubeMedium := io.Cube("app.cube", key)
result = store.New("app.db", store.WithMedium(cubeMedium))
```

---

## 🏗️ Architecture

### Core Components

#### Store Type

The central SQLite key-value store with TTL, events, and journal integration:

```go
type Store struct {
    db                     *sql.DB          // SQLite connection (single, WAL mode)
    journal                influxdb2.Client // InfluxDB client (optional)
    bucket, org            string           // InfluxDB config
    purgeInterval         time.Duration     // Background purge interval (default: 60s)
    workspaceStateDirectory string           // DuckDB workspace directory
    medium                 Medium           // Transport abstraction (optional)
    watchers              map[string][]chan Event
    callbacks             []changeCallbackRegistration
    // Lifecycle management
    lifecycleLock  sync.Mutex
    isClosed, isClosing bool
}

// Event represents a change in the store
type Event struct {
    Type      EventType   // EventSet, EventDelete, EventDeleteGroup
    Group     string      // Group name (e.g., "config")
    Key       string      // Key name (e.g., "colour")
    Value     string      // New value (empty if delete)
    Timestamp time.Time   // When the event occurred
}
```

#### Event Types

```go
type EventType int

const (
    EventSet EventType = iota        // Key value was set
    EventDelete                       // Key was deleted
    EventDeleteGroup                  // Entire group deleted
)
```

### Constructor Options

```go
// Simple constructor with path
func New(databasePath string, options ...StoreOption) (r core.Result)

// Full configuration constructor
func NewConfigured(storeConfig StoreConfig) (r core.Result)

// Configuration struct
type StoreConfig struct {
    DatabasePath            string
    Journal                 JournalConfiguration
    PurgeInterval           time.Duration
    WorkspaceStateDirectory string
    Medium                  Medium
}

type JournalConfiguration struct {
    EndpointURL  string
    Organisation string
    BucketName   string
}

// Store options
func WithJournal(endpointURL, organisation, bucketName string) StoreOption
func WithWorkspaceStateDirectory(directory string) StoreOption
func WithPurgeInterval(interval time.Duration) StoreOption
func WithMedium(medium Medium) StoreOption
```

### CRUD Operations

```go
// Get retrieves a value by group and key
// Lazy-deletes expired entries on access
func (s *Store) Get(group, key string) (string, core.Result)

// Set stores a value with no TTL (permanent)
func (s *Store) Set(group, key, value string) core.Result

// SetWithTTL stores a value with time-to-live
// After expiry, Get returns NotFoundError and background purge removes it
func (s *Store) SetWithTTL(group, key, value string, ttl time.Duration) core.Result

// Delete removes a key from a group
func (s *Store) Delete(group, key string) core.Result

// DeleteGroup removes all keys in a group
func (s *Store) DeleteGroup(group string) core.Result

// DeletePrefix removes all keys in groups matching a prefix
func (s *Store) DeletePrefix(groupPrefix string) core.Result

// Exists checks if a key exists (respects TTL)
func (s *Store) Exists(group, key string) (bool, core.Result)

// GroupExists checks if a group has any keys
func (s *Store) GroupExists(group string) (bool, core.Result)

// GetAll retrieves all key-value pairs in a group
func (s *Store) GetAll(group string) (map[string]string, core.Result)

// GetPage retrieves a paginated subset of keys from a group
func (s *Store) GetPage(group string, offset, limit int) (map[string]string, core.Result)
```

### Iteration

```go
// AllSeq returns all key-value pairs in a group as an iterator
// Uses iter.Seq2 for memory-efficient iteration
func (s *Store) AllSeq(group string) iter.Seq2[string, string]

// GroupsSeq returns all group names as an iterator
func (s *Store) GroupsSeq() iter.Seq2[string, struct{}]

// Count returns the number of keys in a group
func (s *Store) Count(group string) (int, core.Result)

// CountAll returns the total number of keys across all groups
func (s *Store) CountAll() (int, core.Result)
```

### Template Rendering

```go
// Render executes a text/template with store values
// Templates can access store values using {{.Get "group" "key"}}
func (s *Store) Render(templateText string) (string, core.Result)

// RenderWithData combines template data with store values
func (s *Store) RenderWithData(templateText string, data map[string]any) (string, core.Result)
```

---

## 🔔 Event System

### Watch API (Channel-Based)

```go
// Watch returns a buffered channel (cap 16) for events on a specific group
// Non-blocking sends: if buffer is full, events are dropped
// Pass "*" to watch all groups
func (s *Store) Watch(group string) <-chan Event

// Unwatch removes a watcher channel
func (s *Store) Unwatch(group string, events <-chan Event)

// Example: Watch for config changes
ch := st.Watch("config")
defer st.Unwatch("config", ch)

go func() {
    for event := range ch {
        fmt.Printf("[%s] %s/%s = %s\n", 
            event.Type, event.Group, event.Key, event.Value)
    }
}()
```

### OnChange API (Callback-Based)

```go
// OnChange registers a synchronous callback for all events
// Callbacks are invoked after event dispatch, outside locks
// Returns an unregister function
// WARNING: Deadlock risk - callbacks see notifications before watches complete
// Avoid blocking I/O in callbacks
func (s *Store) OnChange(callback func(Event)) func()

// Example: Callback for all changes
unregister := st.OnChange(func(event store.Event) {
    log.Printf("Change detected: %s %s/%s", 
        event.Type, event.Group, event.Key)
})
defer unregister()
```

---

## 🗂️ ScopedStore - Namespace Isolation

### Overview

`ScopedStore` wraps a `*Store` with namespace prefixing and optional quota enforcement. All groups are prefixed with `namespace:`. This provides multi-tenant isolation within a single SQLite database.

```go
type ScopedStore struct {
    store     *Store
    namespace string  // Validated: ^[a-zA-Z0-9-]+$
    MaxKeys   int     // 0 = unlimited
    MaxGroups int     // 0 = unlimited
}
```

### Constructors

```go
// NewScoped creates a scoped store with namespace
// Prefer NewScopedConfigured when config is known at call site
func NewScoped(storeInstance *Store, namespace string) *ScopedStore

// NewScopedConfigured creates with full configuration
func NewScopedConfigured(storeInstance *Store, scopedConfig ScopedStoreConfig) (*ScopedStore, core.Result)

type ScopedStoreConfig struct {
    Namespace string
    Quota     QuotaConfig
}

type QuotaConfig struct {
    MaxKeys   int  // Max keys per namespace (0 = unlimited)
    MaxGroups int  // Max groups per namespace (0 = unlimited)
}

// NewScopedWithQuota convenience constructor
func NewScopedWithQuota(storeInstance *Store, namespace string, quota QuotaConfig) (*ScopedStore, core.Result)
```

### Scoped Operations

```go
// Set stores in the default group (namespace:default)
func (ss *ScopedStore) Set(key, value string) core.Result

// SetIn stores in an explicit group (namespace:group)
func (ss *ScopedStore) SetIn(group, key, value string) core.Result

// Get retrieves from the default group
func (ss *ScopedStore) Get(key string) (string, core.Result)

// GetFrom retrieves from an explicit group
func (ss *ScopedStore) GetFrom(group, key string) (string, core.Result)

// SetWithTTL with expiry
func (ss *ScopedStore) SetWithTTL(group, key, value string, ttl time.Duration) core.Result

// Delete, DeleteGroup, DeletePrefix - scoped versions
func (ss *ScopedStore) Delete(group, key string) core.Result
func (ss *ScopedStore) DeleteGroup(group string) core.Result
func (ss *ScopedStore) DeletePrefix(groupPrefix string) core.Result

// GetAll, Exists, GroupExists - scoped versions
func (ss *ScopedStore) GetAll(group string) (map[string]string, core.Result)
func (ss *ScopedStore) Exists(key string) (bool, core.Result)
func (ss *ScopedStore) ExistsIn(group, key string) (bool, core.Result)
func (ss *ScopedStore) GroupExists(group string) (bool, core.Result)

// Namespace returns the namespace string
func (ss *ScopedStore) Namespace() string

// Config returns the full configuration
func (ss *ScopedStore) Config() ScopedStoreConfig
```

### Scoped Watch

```go
// Watch returns a channel for scoped events
// Internally bridges to the parent store's watcher system
func (ss *ScopedStore) Watch(group string) <-chan Event

// Unwatch removes a scoped watcher
func (ss *ScopedStore) Unwatch(group string, events <-chan Event)

// OnChange registers a callback for scoped events
func (ss *ScopedStore) OnChange(callback func(Event)) func()
```

### Scoped Transactions

```go
// Transaction executes operations atomically with quota checking
func (ss *ScopedStore) Transaction(fn func(*ScopedStoreTransaction) core.Result) core.Result

type ScopedStoreTransaction struct {
    scopedStore *ScopedStore
    // Transactional methods that don't commit until fn returns
}
```

---

## 💼 Workspace - DuckDB Buffer System

### Overview

A **Workspace** is a named DuckDB buffer for mutable work-in-progress. It accumulates entries and commits them atomically to the journal and identity store. This solves the problem of noise in time-series data.

**Key Insight:** Instead of writing 4000 individual "+1" events to the journal, accumulate them in a workspace, then commit once with the final count. The journal only sees meaningful deltas.

### Workspace Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│                        WORKSPACE LIFECYCLE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Created   ───► Active ───► Committed ───► Closed                    │
│       │              │              │              │               │
│       │              │              │              │               │
│  NewWorkspace()   Put()        Commit()       Discard()             │
│       │         Accumulate      Atomic        File Deleted          │
│       │         Entries         Write         (or kept if Close)    │
│       ▼              ▼              ▼              ▼               │
│  .duckdb       .duckdb       Journal        File Deleted          │
│  File          File           Write          (on Discard)          │
│  Created       Active         Complete                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────┘

File States:
- Created:   workspace opens → .core/state/{name}.duckdb
- Active:    Put() accumulates entries
- Committed: Commit() → journal write → identity update → file deleted
- Discarded: Discard() → file deleted
- Crashed:   Orphaned .duckdb files detected on next New() call
```

### Workspace API

```go
type Workspace struct {
    name   string
    store  *Store      // Parent store for identity updates + journal config
    db     *sql.DB     // DuckDB via database/sql driver
    filesystem *core.Fs
}

// NewWorkspace creates a new workspace buffer
// Creates .core/state/{name}.duckdb file
func (s *Store) NewWorkspace(name string) (*Workspace, core.Result)

// Put accumulates an entry in the workspace buffer
func (ws *Workspace) Put(kind string, data map[string]any) error

// Aggregate returns a summary of the current workspace state
// Example: {"like": 4000, "profile_match": 12}
func (ws *Workspace) Aggregate() map[string]any

// Commit writes aggregated state to journal and updates identity store
// Atomic operation: either all succeeds or none does
func (ws *Workspace) Commit() core.Result

// Discard drops the workspace without committing
// Deletes the .duckdb file
func (ws *Workspace) Discard()

// Close keeps the .duckdb file for recovery
// Use RecoverOrphans() to reopen crashed workspaces
func (ws *Workspace) Close() core.Result

// Query runs SQL against the workspace buffer
// Returns []map[string]any (rows as maps)
func (ws *Workspace) Query(sql string) core.Result

// Name returns the workspace name
func (ws *Workspace) Name() string

// DatabasePath returns the .duckdb file path
func (ws *Workspace) DatabasePath() string
```

### Workspace Example

```go
// Create a workspace for a scroll session
ws, result := st.NewWorkspace("scroll-session-2026-03-30")
if !result.OK {
    log.Fatal(result.Error())
}
defer ws.Discard() // Clean up if we don't commit

// Accumulate entries
ws.Put("like", map[string]any{"user": "@alice", "post": "video_123"})
ws.Put("like", map[string]any{"user": "@bob", "post": "video_123"})
ws.Put("profile_match", map[string]any{"user": "@alice"})

// Get summary
summary := ws.Aggregate()
// summary = {"like": 2, "profile_match": 1}

// Run ad-hoc SQL query
result = ws.Query("SELECT kind, COUNT(*) as n FROM entries GROUP BY kind")
rows := result.Value.([]map[string]any)
// rows = [{"kind": "like", "n": 2}, {"kind": "profile_match", "n": 1}]

// Commit atomically to journal
commitResult := ws.Commit()
if commitResult.OK {
    // Workspace file is deleted, journal has the aggregated data
}
```

### Orphan Recovery

```go
// RecoverOrphans scans for leftover .duckdb files
// Returns []*Workspace that can be inspected and committed or discarded
func (s *Store) RecoverOrphans(stateDirectory string) []*Workspace

// Example: Recover after crash
orphans := st.RecoverOrphans(".core/state/")
for _, orphan := range orphans {
    summary := orphan.Aggregate()
    log.Printf("Recovered workspace %s: %v", orphan.Name(), summary)
    
    // Inspect and decide
    if shouldRecover(summary) {
        orphan.Commit()
    } else {
        orphan.Discard()
    }
}
```

---

## 📊 Journal - InfluxDB Time-Series

### Overview

The **Journal** provides time-series persistence for completed work units. Each commit writes a single InfluxDB point representing the aggregated state of a workspace. This ensures queries naturally produce meaningful results without complex aggregation.

### Journal API

```go
// CommitToJournal writes aggregated state as a single InfluxDB point
// Called by Workspace.Commit() internally, but exported for testing
func (s *Store) CommitToJournal(measurement string, fields map[string]any, tags map[string]string) core.Result

// QueryJournal runs a Flux query against the time-series
// Returns []map[string]any (rows as maps)
func (s *Store) QueryJournal(flux string) core.Result
```

### Flux Query Support

```go
// Raw SQL queries (prefixed with SELECT, WITH, EXPLAIN, PRAGMA)
result := st.QueryJournal("SELECT measurement, committed_at FROM journal_entries ORDER BY committed_at")

// Flux queries
result := st.QueryJournal(`from(bucket: "core") |> range(start: -7d) |> filter(fn: (r) => r.measurement == "scroll-session")`)

// Supported Flux features:
// - from(bucket: "...") - bucket selection
// - range(start: -7d, stop: now()) - time range
// - filter() with ==, !=, >, <, >=, <=
// - Strings, integers, floats, booleans
// - r._measurement, r._bucket, r.column_name
```

### Flux-to-SQL Translation

The store automatically translates Flux queries to SQLite-compatible SQL:

| Flux | SQLite |
|------|---------|
| `from(bucket: "core")` | `WHERE bucket_name = 'core'` |
| `range(start: -7d)` | `WHERE committed_at >= <timestamp>` |
| `filter(fn: (r) => r.workspace == "a")` | `WHERE tags_json LIKE '%"workspace":"a"%'` |

---

## 🗃️ Cold Archive

### Overview

When journal entries age past retention, they compact to cold storage as compressed JSONL files. Each line is a complete unit of work — ready for training data ingestion, CDN publishing, or long-term analytics.

### Archive API

```go
type CompactOptions struct {
    Before time.Time // Archive entries before this time
    Output string    // Output directory (default: .core/archive/)
    Format string    // "gzip" or "zstd" (default: gzip)
    Medium Medium    // Transport backend (optional)
}

// Compact archives journal entries to compressed JSONL
func (s *Store) Compact(opts CompactOptions) core.Result
```

### Archive Example

```go
// Compact entries older than 90 days to S3
result := st.Compact(store.CompactOptions{
    Before: time.Now().Add(-90 * 24 * time.Hour),
    Output: "/archive/",
    Format: "gzip",
    Medium: io.S3("garage.lthn.io/archive/"),
})

// Compact to encrypted DataCube
result = st.Compact(store.CompactOptions{
    Before: time.Now().Add(-30 * 24 * time.Hour),
    Medium: io.Cube("cold-archive.cube", archiveKey),
})
```

### Archive Format

```jsonl
{"bucket":"core","measurement":"scroll-session","fields":{"like":4000,"profile_match":12},"tags":{"workspace":"session-a"},"committed_at":1711234567890}
{"bucket":"core","measurement":"scroll-session","fields":{"like":2500},"tags":{"workspace":"session-b"},"committed_at":1711234567891}
```

Compressed with gzip or zstd, each line is a self-contained JSON object.

---

## 🔌 io.Medium Integration

### Overview

go-store accepts `io.Medium` as its storage backend. The store API stays the same — only the transport changes. Hot-swap local filesystem for S3, memory, encrypted DataCube, or SFTP with one constructor option.

### Medium Backends

| Backend | Use Case | Example |
|---------|----------|---------|
| Local | Local development | `io.Local("~/.core/data/")` |
| S3 | Production (Garage) | `io.S3("garage.lthn.io/bucket")` |
| Memory | Unit tests, CI/CD | `io.Memory()` |
| Cube | Distributed agents | `io.Cube("app.cube", key)` |
| SFTP | Backup to homelab | `io.SFTP("homelab.lthn.sh/backup/")` |

### Medium Usage

```go
// Constructor with Medium
func WithMedium(medium Medium) StoreOption

// Store creation with Medium
st, result := store.New("app.db", store.WithMedium(io.S3("garage.lthn.io/bucket")))

// Medium-backed DuckDB workspace
workspaceMedium := io.Memory()
// Import/export through Medium
store.Import(ws, workspaceMedium, "data.json")
store.Export(ws, io.S3("garage.lthn.io/export/"), "results.json")

// Cold archive through Medium
st.Compact(store.CompactOptions{
    Before: time.Now().Add(-90 * 24 * time.Hour),
    Medium: io.S3("garage.lthn.io/archive/"),
})
```

### Medium-Handled Operations

The Medium handles all database file I/O:
- Reads/writes for SQLite database files
- WAL and journal file management
- DuckDB workspace files
- Cold archive files

The store never touches the filesystem directly — everything goes through the Medium interface.

---

## 🔧 Import/Export

### Import Data

```go
// Import reads data from a Medium into a workspace buffer
// Format determined by file extension (CSV, JSON, JSONL)
func Import(ws *Workspace, medium Medium, path string) error

// Example: Import CSV from S3
s3Medium := io.S3("garage.lthn.io/data/")
store.Import(ws, s3Medium, "dataset.csv")

// Example: Import JSON from GitHub
githubMedium := github.New(github.Options{Owner: "org", Repo: "data"})
store.Import(ws, githubMedium, "data.json")

// Example: Import from encrypted DataCube
cubeMedium, _ := cube.Open("archive.cube", key)
store.Import(ws, cubeMedium, "entries.jsonl")
```

### Export Data

```go
// Export writes workspace query results to a Medium
func Export(ws *Workspace, medium Medium, path string) error

// Example: Export to S3
s3Medium := io.S3("garage.lthn.io/reports/")
store.Export(ws, s3Medium, "report.json")

// Example: Export training data
store.Export(ws, io.Local("/data/training/"), "dataset.jsonl")
```

---

## 📝 Transactions

### Transaction API

```go
// Transaction executes operations atomically on the underlying SQLite store
func (s *Store) Transaction(fn func(*Transaction) core.Result) core.Result

type Transaction struct {
    store *Store
}

// Transaction methods mirror Store methods but execute within a transaction
func (t *Transaction) Get(group, key string) (string, core.Result)
func (t *Transaction) Set(group, key, value string) core.Result
func (t *Transaction) SetWithTTL(group, key, value string, ttl time.Duration) core.Result
func (t *Transaction) Delete(group, key string) core.Result
func (t *Transaction) DeleteGroup(group string) core.Result
func (t *Transaction) DeletePrefix(groupPrefix string) core.Result
func (t *Transaction) GetPage(group string, offset, limit int) (map[string]string, core.Result)
func (t *Transaction) All(group string) (map[string]string, core.Result)
func (t *Transaction) GroupsSeq() iter.Seq2[string, struct{}]
func (t *Transaction) Render(templateText string) (string, core.Result)
```

### Transaction Example

```go
result := st.Transaction(func(tx *store.Transaction) core.Result {
    // All operations execute atomically
    tx.Set("user", "balance", "100")
    tx.Set("user", "last_update", time.Now().Format(time.RFC3339))
    
    // If any operation fails, the entire transaction rolls back
    oldBalance, _ := tx.Get("user", "balance")
    if oldBalance != "" {
        return core.Fail(core.E("tx", "balance already set", nil))
    }
    
    return core.Ok(nil)
})
```

---

## 🔒 Lifecycle Management

### Store Lifecycle

```go
// Close the store
func (s *Store) Close() core.Result

// Check if store is closed
func (s *Store) IsClosed() bool

// Get configuration
func (s *Store) Config() StoreConfig
func (s *Store) JournalConfiguration() JournalConfiguration
func (s *Store) DatabasePath() string
func (s *Store) WorkspaceStateDirectory() string
func (s *Store) JournalConfigured() bool
```

### Background Purge

```go
// Background purge goroutine runs every `purgeInterval` (default: 60s)
// Removes expired entries from the database
// Started automatically on New() / NewConfigured()

// Custom purge interval
st, _ := store.New(":memory:", store.WithPurgeInterval(30*time.Second))
```

---

## 📊 Use Cases

| Scenario | Configuration | Use Case |
|----------|--------------|----------|
| Local development | `io.Local("~/.core/data/")` | Fast, simple, default |
| Production | `io.S3("garage.lthn.io/bucket")` | Garage on own infrastructure, durable |
| Unit tests | `io.Memory()` | No disk, no cleanup, fast |
| Offline mobile | `io.Memory()` | Accumulate in RAM, flush to Local on reconnect |
| Distributed agents | `io.Cube("app.cube", key)` | Encrypted, signed, portable between nodes |
| Backup | `io.SFTP("homelab.lthn.sh/backup/")` | Nightly sync to homelab |
| CI/CD | `io.Memory()` | Ephemeral test runs, no state leaks |

---

## 🎯 Consumers

| Package | Usage |
|---------|-------|
| `core/ide` | Memory caching for agent workspaces |
| `go-agent` | Workspace state management |
| `go-io` | Transport-agnostic storage (via Medium) |
| Custom agents | Durable state, workspace buffering |

---

## 🏆 Key Features Summary

### ✅ What go-store Provides

1. **TTL Expiry** — Three-layer enforcement (lazy delete, query filter, background purge)
2. **Namespace Isolation** — `ScopedStore` with quota enforcement
3. **Reactive Events** — `Watch()` channels + `OnChange()` callbacks
4. **Workspace Buffering** — DuckDB mutable accumulation, atomic commit
5. **Time-Series Journal** — InfluxDB integration with Flux query support
6. **Cold Archive** — Compressed JSONL (gzip/zstd) for long-term storage
7. **Transport Abstraction** — `io.Medium` for hot-swapping storage backends
8. **SQL Injection Protection** — `escapeLike()` for LIKE queries
9. **Single-Connection Design** — `MaxOpenConns(1)` for SQLite pragmas
10. **Orphan Recovery** — Automatic detection and recovery of crashed workspaces
11. **Atomic Transactions** — Full SQLite transaction support
12. **Template Rendering** — `text/template` integration with store values

### ❌ What go-store Does NOT Do

1. **No CGO** — Pure Go implementation (uses `modernc.org/sqlite`)
2. **No connection pooling** — Single-connection by design for SQLite
3. **No distributed coordination** — Single-instance; use go-p2p for multi-node
4. **No schema migrations** — Schema is fixed at initialization
5. **No external dependencies** — Except `modernc.org/sqlite` and `influxdb-client-go`

---

## 📚 Testing

### Test Patterns

All tests follow the **Good/Bad/Ugly** triplet pattern:
- `TestFile_Function_Good_*` — Happy path, valid inputs
- `TestFile_Function_Bad_*` — Error conditions, invalid inputs
- `TestFile_Function_Ugly_*` — Edge cases, boundary conditions

### Running Tests

```bash
# Run all tests
go test ./... -count=1

# Run with race detector (must pass before commit)
go test -race ./...

# Run with coverage
go test -cover ./...

# Run specific test
go test -v -run TestStore_Set_Good_.* ./...
```

### Test File Structure

```
store_test.go           # Store CRUD tests
store_example_test.go   # Usage examples as tests
scope_test.go           # ScopedStore tests
scope_example_test.go   # ScopedStore examples
workspace_test.go        # Workspace tests
workspace_example_test.go # Workspace examples
journal_test.go         # Journal tests
journal_example_test.go # Journal examples
compact_test.go         # Compact/archive tests
compact_example_test.go # Compact examples
events_test.go           # Event system tests
events_example_test.go   # Event examples
transaction_test.go      # Transaction tests
transaction_example_test.go # Transaction examples
```

---

## 🎨 Code Examples

### Example: Session Management

```go
// Create session store with TTL
sessionStore, _ := store.New(":memory:")

// Store session token with 1 hour TTL
sessionStore.SetWithTTL("sessions", "user:123:token", "abc123", time.Hour)

// Check session on each request
func checkSession(token string) bool {
    value, result := sessionStore.Get("sessions", "user:123:token")
    return result.OK && value == token
}

// Sessions automatically expire after 1 hour
```

### Example: Configuration Store

```go
configStore, _ := store.New("config.db")

// Get configuration with default
func getConfig(key string, defaultValue string) string {
    value, result := configStore.Get("config", key)
    if !result.OK {
        return defaultValue
    }
    return value
}

// Set configuration
configStore.Set("config", "theme", "dark")
configStore.Set("config", "language", "en-GB")

// Watch for config changes
ch := configStore.Watch("config")
go func() {
    for event := range ch {
        log.Printf("Config changed: %s = %s", event.Key, event.Value)
    }
}()
```

### Example: Multi-Tenant with ScopedStore

```go
mainStore, _ := store.New("multi-tenant.db")

// Create tenant-specific stores
tenantA := store.NewScoped(mainStore, "tenant-a")
tenantB := store.NewScoped(mainStore, "tenant-b")

// Each tenant has isolated namespace
tenantA.Set("config", "theme", "dark")      // Stored as "tenant-a:config/theme"
tenantB.Set("config", "theme", "light")     // Stored as "tenant-b:config/theme"

// With quotas
tenantC, _ := store.NewScopedWithQuota(mainStore, "tenant-c", store.QuotaConfig{
    MaxKeys:   100,
    MaxGroups: 10,
})
```

### Example: Analytics Workspace

```go
analyticsStore, _ := store.New("analytics.db")

// Create workspace for today's analytics
ws, _ := analyticsStore.NewWorkspace("analytics-2026-06-18")

// Track events
ws.Put("page_view", map[string]any{"url": "/home", "user": "alice"})
ws.Put("page_view", map[string]any{"url": "/about", "user": "bob"})
ws.Put("click", map[string]any{"button": "cta", "page": "/home"})

// Get real-time summary
summary := ws.Aggregate()
// summary = {"page_view": 2, "click": 1}

// Commit to journal at end of day
ws.Commit()

// Later: Query historical analytics
result := analyticsStore.QueryJournal(`from(bucket: "analytics") |> range(start: -30d) |> filter(fn: (r) => r.measurement == "analytics")`)
```

### Example: Training Data Pipeline

```go
// Create store with S3 medium
trainingStore, _ := store.New(":memory:", store.WithMedium(io.S3("s3.amazonaws.com/training/")))

// Create workspace for training data
ws, _ := trainingStore.NewWorkspace("training-run-001")

// Import data from various sources
store.Import(ws, io.S3("bucket/dataset1.csv"), "dataset1.csv")
store.Import(ws, io.Local("/local/data/"), "dataset2.json")

// Process data in workspace
result := ws.Query(`SELECT * FROM entries WHERE kind = 'valid'`)// Clean data
cleanRows := result.Value.([]map[string]any)

// Export processed data
store.Export(ws, io.S3("bucket/processed/"), "cleaned.jsonl")

// Commit metadata to journal
trainingStore.CommitToJournal("training-metadata", map[string]any{
    "total_records": len(cleanRows),
    "processing_time": 3600,
}, map[string]string{"run_id": "001"})

// Archive old training data
 trainingStore.Compact(store.CompactOptions{
    Before: time.Now().Add(-90 * 24 * time.Hour),
    Medium: io.S3("bucket/archive/"),
})
```

---

## 📖 References

### RFC Specification

- **[RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/store/RFC.md)** — Complete technical specification

### Related Documentation

- **[CLAUDE.md](file:///Users/snider/Code/core/go-store/CLAUDE.md)** — Implementation details, conventions, banned imports
- **[AGENTS.md](file:///Users/snider/Code/core/go-store/AGENTS.md)** — Agent guidance for working with this package
- **[CoreGO RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)** — Framework specification
- **[IO RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/io/RFC.md)** — Medium transport specification

### Source Code

- **Repository:** `github.com/dAppCore/go-store` / `dappco.re/go/store`
- **Module:** `dappco.re/go/store`
- **Package:** `dappco.re/go/store`
- **Go Version:** 1.21+

### Dependencies

```
modernc.org/sqlite          # SQLite driver (pure Go)
github.com/influxdata/influxdb-client-go/v2  # InfluxDB client
dappco.re/go               # CoreGO framework
```

---

## 🏷️ Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/store` |
| **Repository** | `core/go-store` |
| **Type** | Library |
| **Status** | Production |
| **Tier** | lib (foundation) |
| **Lines** | ~15,000+ (source + tests) |
| **Files** | 20+ source files |
| **Test Coverage** | High |
| **Licence** | EUPL-1.2 |
| **Language** | Go 1.21+ |
| **CGO** | Not required (pure Go) |
| **Author** | Purberus <purberus@lthn.ai> |
| **Created** | 2026-06-18T05:00:00Z |
| **Version** | 1.0.0 |

---

*Documentation generated from source code analysis and RFC specification.*
*Last updated: 2026-06-18T05:00:00Z*
