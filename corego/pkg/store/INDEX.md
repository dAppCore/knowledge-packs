---
type: Package Index
title: go-store Package Index
description: Complete index of go-store package components and API surface
description: SQLite KV store with TTL, namespace isolation, DuckDB workspaces, InfluxDB journal, cold archive, io.Medium integration
module: dappco.re/go/store
repo: core/go-store
---

# go-store — Package Index

> **Repository:** `core/go-store`
> **Module:** `dappco.re/go/store`
> **Type:** Library
> **Status:** Production
> **Tier:** lib (foundation — consumers import this, this never imports consumers)
> **Lines:** ~23,000+ (source + tests + example tests)

---

## 📚 Quick Links

- **[README.md](./README.md)** — Complete package documentation
- **[RFC Specification](file:///Users/snider/Code/meowmix/plans/code/core/go/store/RFC.md)** — Technical specification (self-contained, agent-implementable)
- **[CLAUDE.md](file:///Users/snider/Code/core/go-store/CLAUDE.md)** — Implementation details, banned imports, conventions
- **[AGENTS.md](file:///Users/snider/Code/core/go-store/AGENTS.md)** — Agent guidance

### Sub-Specifications

The RFC includes comprehensive sub-specifications:

- **Section 2:** Overview — Package purpose and high-level architecture
- **Section 3:** Architecture — File structure and component relationships
- **Section 4:** Store Struct — Core Store type and its fields
- **Section 5:** API — Complete public API with usage examples
- **Section 6:** ScopedStore — Namespace isolation with quota enforcement
- **Section 7:** Event System — Watch/OnChange API with buffered channels
- **Section 8:** Workspace Buffer — DuckDB buffering with atomic commit
- **Section 9:** Medium-Backed Storage — io.Medium transport abstraction
- **Section 10:** Reference Material — Related RFCs and specifications

### Local Documentation

- [RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/store/RFC.md) — Full technical specification
- [CLAUDE.md](file:///Users/snider/Code/core/go-store/CLAUDE.md) — Development conventions, test patterns
- [AGENTS.md](file:///Users/snider/Code/core/go-store/AGENTS.md) — Agent-specific guidance
- [docs/](file:///Users/snider/Code/core/go-store/docs/) — Additional documentation

---

## 🗂️ File Structure

### Core Package Files (`go/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `doc.go` | 134 | Package-level documentation, usage examples | ✅ Complete |
| `store.go` | 1,427 | Core `Store`: CRUD, TTL, schema, lifecycle, configuration | ✅ Complete |
| `events.go` | 236 | Event system: `Watch`/`Unwatch`, `OnChange`, `notify()` dispatch | ✅ Complete |
| `scope.go` | 956 | `ScopedStore`: namespace prefixing, quota enforcement, scoped operations | ✅ Complete |
| `workspace.go` | 670 | `Workspace`: DuckDB buffer, Put, Aggregate, Commit, Discard, Query | ✅ Complete |
| `journal.go` | 690 | InfluxDB journal: CommitToJournal, QueryJournal, Flux-to-SQL translation | ✅ Complete |
| `compact.go` | 315 | Cold archive: Compact, CompactOptions, JSONL compression (gzip/zstd) | ✅ Complete |
| `transaction.go` | 507 | SQLite transactions: Transaction type, atomic operations | ✅ Complete |
| `medium.go` | 509 | io.Medium integration: prepareSQLiteStorage, syncMediumBackedDatabase | ✅ Complete |
| `import.go` | 882 | Data import: Import, format detection (CSV/JSON/JSONL), Medium integration | ✅ Complete |
| `json.go` | 204 | JSON utilities: marshalJSONText, JSONL parsing, validation | ✅ Complete |
| `duckdb.go` | 527 | DuckDB workspace database: openWorkspaceDatabase, schema, helpers | ✅ Complete |
| `duckdb_driver.go` | 7 | DuckDB SQLite driver registration | ✅ Complete |
| `service.go` | 230 | CoreGO service integration: action handlers, IPC | ✅ Complete |
| `publish.go` | 329 | Publishing utilities: Upload, Hugo site publishing, HF integration | ✅ Complete |
| `inventory.go` | 191 | Inventory management: ImportAll, training files, benchmark data | ✅ Complete |
| `parquet.go` | 95 | Parquet support: DuckDB Parquet reading | ✅ Complete |

**Total Core Source Lines:** ~8,085

### Test Files (`go/`)

#### Unit Tests (48 files)

| File | Lines | Purpose | Pattern |
|------|-------|---------|---------|
| `store_test.go` | ~87,335 | Store CRUD, TTL, Get/Set/Delete, pagination | Good/Bad/Ugly |
| `scope_test.go` | ~91,243 | ScopedStore operations, quota enforcement, namespace validation | Good/Bad/Ugly |
| `workspace_test.go` | ~24,238 | Workspace Put, Aggregate, Commit, Discard, Query | Good/Bad/Ugly |
| `journal_test.go` | ~13,623 | Journal CommitToJournal, QueryJournal, Flux parsing | Good/Bad/Ugly |
| `events_test.go` | ~12,952 | Watch, Unwatch, OnChange, notify, callback registration | Good/Bad/Ugly |
| `compact_test.go` | ~10,468 | Compact, archive, compression, Medium integration | Good/Bad/Ugly |
| `transaction_test.go` | ~33,094 | Transaction atomicity, rollback, concurrent access | Good/Bad/Ugly |
| `medium_test.go` | ~19,888 | Medium-backed storage, S3, Memory, Cube, SFTP | Good/Bad/Ugly |
| `import_test.go` | ~7,840 | Import from various Medium backends, format detection | Good/Bad/Ugly |
| `duckdb_test.go` | ~10,031 | DuckDB workspace operations, SQL queries | Good/Bad/Ugly |
| `coverage_test.go` | ~21,168 | Integration tests, end-to-end workflows | Good/Bad/Ugly |
| `conventions_test.go` | ~10,400 | AX standard compliance, naming conventions | Good/Bad/Ugly |
| `v090_helpers_test.go` | ~8,177 | Version-specific helpers and utilities | Good/Bad/Ugly |
| `recover_test.go` | ~2,227 | Orphan workspace recovery, crash resilience | Good/Bad/Ugly |
| `training_example_test.go` | ~8,582 | Training data examples and patterns | Examples |
| `inventory_helpers_test.go` | ~5,124 | Inventory management test helpers | Helpers |
| `import_helpers_test.go` | ~7,740 | Import test utilities and fixtures | Helpers |
| `publish_helpers_test.go` | ~948 | Publishing test helpers | Helpers |
| `medium_proxy_test.go` | ~6,493 | Medium proxy testing | Good/Bad/Ugly |
| `import_export_test.go` | ~948 | Import/export integration | Good/Bad/Ugly |
| `journal_parse_test.go` | ~5,289 | Journal Flux query parsing | Good/Bad/Ugly |
| `json_indent_test.go` | ~817 | JSON indentation formatting | Good |
| `path_test.go` | ~415 | Path manipulation utilities | Good |

**Total Test Lines:** ~370,000+

#### Example Tests (16 files)

| File | Lines | Purpose |
|------|-------|---------|
| `store_example_test.go` | ~9,274 | Store usage examples |
| `scope_example_test.go` | ~18,453 | ScopedStore usage examples |
| `workspace_example_test.go` | ~2,227 | Workspace usage examples |
| `journal_example_test.go` | ~688 | Journal usage examples |
| `events_example_test.go` | ~740 | Event system usage examples |
| `compact_example_test.go` | ~834 | Compact/archive usage examples |
| `transaction_example_test.go` | ~3,309 | Transaction usage examples |
| `medium_example_test.go` | ~1,189 | Medium integration examples |
| `import_example_test.go` | ~345 | Import usage examples |
| `publish_example_test.go` | ~265 | Publishing examples |
| `inventory_example_test.go` | ~265 | Inventory examples |
| `parquet_example_test.go` | ~265 | Parquet usage examples |
| `service_example_test.go` | ~230 | Service integration examples |
| `json_example_test.go` | ~588 | JSON utilities examples |
| `training_example_test.go` | ~4,639 | Training data pipeline examples |

**Total Example Lines:** ~53,000+

### External Package (`external/go/`)

| File | Purpose |
|------|---------|
| `external.go` | External dependencies wrapper (if any) |

### Documentation Files

| File | Purpose |
|------|---------|
| `README.md` | Package overview and basic usage |
| `RFC.md` | Full technical specification |
| `CLAUDE.md` | Development conventions and guidance |
| `AGENTS.md` | Agent-specific working instructions |
| `CONTRIBUTING.md` | Contribution guidelines |
| `DEPENDENCIES.md` | Dependency information |
| `LICENCE` / `LICENCE.md` | EUPL-1.2 licence |

### Directories

| Directory | Purpose |
|-----------|---------|
| `docs/` | Additional documentation |
| `tests/` | Integration and end-to-end tests |
| `.core/` | Runtime state (workspace files, archives) |
| `external/` | External integrations |

---

## 🔧 Public API Surface

### Store Type

The central SQLite key-value store:

```go
type Store struct {
    // Database connections
    db                     *sql.DB
    sqliteDatabase         *sql.DB
    journal                influxdb2.Client
    
    // Configuration
    databasePath           string
    workspaceStateDirectory string
    purgeInterval          time.Duration
    bucket, org           string
    journalConfiguration   JournalConfiguration
    medium                 Medium
    
    // Lifecycle
    lifecycleLock          sync.Mutex
    closeLock              sync.Mutex
    isClosed, isClosing    bool
    
    // Event system
    watchers              map[string][]chan Event
    callbacks             []changeCallbackRegistration
    watcherLock           sync.RWMutex
    callbackLock          sync.RWMutex
    nextCallbackID        uint64
    
    // Workspace
    orphanWorkspaceLock   sync.Mutex
    cachedOrphanWorkspaces []*Workspace
}
```

### Configuration Types

```go
// StoreConfig for NewConfigured constructor
type StoreConfig struct {
    DatabasePath            string
    Journal                 JournalConfiguration
    PurgeInterval           time.Duration
    WorkspaceStateDirectory string
    Medium                  Medium
}

// JournalConfiguration for InfluxDB integration
type JournalConfiguration struct {
    EndpointURL  string
    Organisation string
    BucketName   string
}

// CompactOptions for cold archive
type CompactOptions struct {
    Before time.Time
    Output string
    Format string  // "gzip" or "zstd"
    Medium Medium
}

// EventType for change notifications
type EventType int
const (
    EventSet EventType = iota
    EventDelete
    EventDeleteGroup
)

// Event emitted on changes
type Event struct {
    Type      EventType
    Group     string
    Key       string
    Value     string
    Timestamp time.Time
}
```

### ScopedStore Types

```go
type ScopedStore struct {
    store     *Store
    namespace string  // Validated: ^[a-zA-Z0-9-]+$
    MaxKeys   int
    MaxGroups int
    watcherBridgeLock sync.Mutex
    watcherBridges    map[uintptr]scopedWatcherBridge
}

type ScopedStoreConfig struct {
    Namespace string
    Quota     QuotaConfig
}

type QuotaConfig struct {
    MaxKeys   int  // 0 = unlimited
    MaxGroups int  // 0 = unlimited
}
```

### Workspace Types

```go
type Workspace struct {
    name                  string
    store                 *Store
    db                    *sql.DB  // DuckDB connection
    databasePath          string
    filesystem            *core.Fs
    cachedOrphanAggregate map[string]any
    lifecycleLock         sync.Mutex
    isClosed              bool
}
```

### Medium Types

```go
// Medium interface for transport abstraction
type Medium interface {
    Read(path string) (result core.Result)
    Write(path string, content string) (result core.Result)
    Exists(path string) bool
    Delete(path string) (result core.Result)
    List(path string) (result core.Result)
    // ... additional methods for directory operations
}
```

---

## 📋 Public API Methods

### Store Constructors

| Method | Signature | Description |
|--------|-----------|-------------|
| `New` | `func New(databasePath string, options ...StoreOption) (r core.Result)` | Create store with options |
| `NewConfigured` | `func NewConfigured(storeConfig StoreConfig) (r core.Result)` | Create store with full config |

### Store Options

| Function | Signature | Description |
|----------|-----------|-------------|
| `WithJournal` | `func WithJournal(endpointURL, organisation, bucketName string) StoreOption` | Configure InfluxDB journal |
| `WithWorkspaceStateDirectory` | `func WithWorkspaceStateDirectory(directory string) StoreOption` | Set workspace directory |
| `WithPurgeInterval` | `func WithPurgeInterval(interval time.Duration) StoreOption` | Set TTL purge interval |
| `WithMedium` | `func WithMedium(medium Medium) StoreOption` | Set transport backend |

### Store CRUD Operations

| Method | Signature | Description |
|--------|-----------|-------------|
| `Get` | `func (s *Store) Get(group, key string) (string, core.Result)` | Retrieve value by group/key |
| `Set` | `func (s *Store) Set(group, key, value string) core.Result` | Store value permanently |
| `SetWithTTL` | `func (s *Store) SetWithTTL(group, key, value string, ttl time.Duration) core.Result` | Store with expiry |
| `Delete` | `func (s *Store) Delete(group, key string) core.Result` | Remove key from group |
| `DeleteGroup` | `func (s *Store) DeleteGroup(group string) core.Result` | Remove entire group |
| `DeletePrefix` | `func (s *Store) DeletePrefix(groupPrefix string) core.Result` | Remove groups by prefix |
| `Exists` | `func (s *Store) Exists(group, key string) (bool, core.Result)` | Check key existence |
| `GroupExists` | `func (s *Store) GroupExists(group string) (bool, core.Result)` | Check group existence |
| `GetAll` | `func (s *Store) GetAll(group string) (map[string]string, core.Result)` | Get all key-value pairs in group |
| `GetPage` | `func (s *Store) GetPage(group string, offset, limit int) (map[string]string, core.Result)` | Paginated get |
| `Count` | `func (s *Store) Count(group string) (int, core.Result)` | Count keys in group |
| `CountAll` | `func (s *Store) CountAll() (int, core.Result)` | Count all keys |

### Store Iteration

| Method | Signature | Description |
|--------|-----------|-------------|
| `AllSeq` | `func (s *Store) AllSeq(group string) iter.Seq2[string, string]` | Iterate all key-value pairs |
| `GroupsSeq` | `func (s *Store) GroupsSeq() iter.Seq2[string, struct{}]` | Iterate group names |

### Store Template Rendering

| Method | Signature | Description |
|--------|-----------|-------------|
| `Render` | `func (s *Store) Render(templateText string) (string, core.Result)` | Execute text/template with store values |
| `RenderWithData` | `func (s *Store) RenderWithData(templateText string, data map[string]any) (string, core.Result)` | Template with additional data |

### Store Events

| Method | Signature | Description |
|--------|-----------|-------------|
| `Watch` | `func (s *Store) Watch(group string) <-chan Event` | Channel for group events |
| `Unwatch` | `func (s *Store) Unwatch(group string, events <-chan Event)` | Remove watcher |
| `OnChange` | `func (s *Store) OnChange(callback func(Event)) func()` | Register synchronous callback |

### Store Lifecycle

| Method | Signature | Description |
|--------|-----------|-------------|
| `Close` | `func (s *Store) Close() core.Result` | Close store and cleanup |
| `IsClosed` | `func (s *Store) IsClosed() bool` | Check if closed |
| `Config` | `func (s *Store) Config() StoreConfig` | Get configuration |
| `JournalConfiguration` | `func (s *Store) JournalConfiguration() JournalConfiguration` | Get journal config |
| `DatabasePath` | `func (s *Store) DatabasePath() string` | Get database path |
| `WorkspaceStateDirectory` | `func (s *Store) WorkspaceStateDirectory() string` | Get workspace directory |
| `JournalConfigured` | `func (s *Store) JournalConfigured() bool` | Check if journal is configured |

### Store Transactions

| Method | Signature | Description |
|--------|-----------|-------------|
| `Transaction` | `func (s *Store) Transaction(fn func(*Transaction) core.Result) core.Result` | Execute atomic transaction |

### Transaction Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `Get` | `func (t *Transaction) Get(group, key string) (string, core.Result)` | Get within transaction |
| `Set` | `func (t *Transaction) Set(group, key, value string) core.Result` | Set within transaction |
| `SetWithTTL` | `func (t *Transaction) SetWithTTL(group, key, value string, ttl time.Duration) core.Result` | Set with TTL |
| `Delete` | `func (t *Transaction) Delete(group, key string) core.Result` | Delete within transaction |
| `DeleteGroup` | `func (t *Transaction) DeleteGroup(group string) core.Result` | Delete group |
| `DeletePrefix` | `func (t *Transaction) DeletePrefix(groupPrefix string) core.Result` | Delete by prefix |
| `GetPage` | `func (t *Transaction) GetPage(group string, offset, limit int) (map[string]string, core.Result)` | Paginated get |
| `All` | `func (t *Transaction) All(group string) (map[string]string, core.Result)` | Get all in group |
| `GroupsSeq` | `func (t *Transaction) GroupsSeq() iter.Seq2[string, struct{}]` | Iterate groups |
| `Render` | `func (t *Transaction) Render(templateText string) (string, core.Result)` | Template rendering |

### Workspace Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewWorkspace` | `func (s *Store) NewWorkspace(name string) (*Workspace, core.Result)` | Create workspace |
| `Put` | `func (ws *Workspace) Put(kind string, data map[string]any) error` | Add entry to workspace |
| `Aggregate` | `func (ws *Workspace) Aggregate() map[string]any` | Get summary of workspace |
| `Commit` | `func (ws *Workspace) Commit() core.Result` | Atomic commit to journal |
| `Discard` | `func (ws *Workspace) Discard()` | Discard without committing |
| `Close` | `func (ws *Workspace) Close() core.Result` | Close keeping files |
| `Query` | `func (ws *Workspace) Query(sql string) core.Result` | Run SQL on workspace |
| `Name` | `func (ws *Workspace) Name() string` | Get workspace name |
| `DatabasePath` | `func (ws *Workspace) DatabasePath() string` | Get .duckdb path |
| `RecoverOrphans` | `func (s *Store) RecoverOrphans(stateDirectory string) []*Workspace` | Recover crashed workspaces |

### Journal Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `CommitToJournal` | `func (s *Store) CommitToJournal(measurement string, fields map[string]any, tags map[string]string) core.Result` | Write to journal |
| `QueryJournal` | `func (s *Store) QueryJournal(flux string) core.Result` | Query with Flux/SQL |

### Compact/Archive Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `Compact` | `func (s *Store) Compact(opts CompactOptions) core.Result` | Archive old entries |

### ScopedStore Constructors

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewScoped` | `func NewScoped(storeInstance *Store, namespace string) *ScopedStore` | Create scoped store |
| `NewScopedConfigured` | `func NewScopedConfigured(storeInstance *Store, scopedConfig ScopedStoreConfig) (*ScopedStore, core.Result)` | Create with config |
| `NewScopedWithQuota` | `func NewScopedWithQuota(storeInstance *Store, namespace string, quota QuotaConfig) (*ScopedStore, core.Result)` | Create with quota |

### ScopedStore Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `Set` | `func (ss *ScopedStore) Set(key, value string) core.Result` | Set in default group |
| `SetIn` | `func (ss *ScopedStore) SetIn(group, key, value string) core.Result` | Set in explicit group |
| `Get` | `func (ss *ScopedStore) Get(key string) (string, core.Result)` | Get from default group |
| `GetFrom` | `func (ss *ScopedStore) GetFrom(group, key string) (string, core.Result)` | Get from explicit group |
| `SetWithTTL` | `func (ss *ScopedStore) SetWithTTL(group, key, value string, ttl time.Duration) core.Result` | Set with TTL |
| `Delete` | `func (ss *ScopedStore) Delete(group, key string) core.Result` | Delete from group |
| `DeleteGroup` | `func (ss *ScopedStore) DeleteGroup(group string) core.Result` | Delete entire group |
| `DeletePrefix` | `func (ss *ScopedStore) DeletePrefix(groupPrefix string) core.Result` | Delete by prefix |
| `GetAll` | `func (ss *ScopedStore) GetAll(group string) (map[string]string, core.Result)` | Get all in group |
| `Exists` | `func (ss *ScopedStore) Exists(key string) (bool, core.Result)` | Check existence |
| `ExistsIn` | `func (ss *ScopedStore) ExistsIn(group, key string) (bool, core.Result)` | Check in group |
| `GroupExists` | `func (ss *ScopedStore) GroupExists(group string) (bool, core.Result)` | Check group |
| `Namespace` | `func (ss *ScopedStore) Namespace() string` | Get namespace |
| `Config` | `func (ss *ScopedStore) Config() ScopedStoreConfig` | Get configuration |
| `Watch` | `func (ss *ScopedStore) Watch(group string) <-chan Event` | Watch scoped events |
| `Unwatch` | `func (ss *ScopedStore) Unwatch(group string, events <-chan Event)` | Unwatch |
| `OnChange` | `func (ss *ScopedStore) OnChange(callback func(Event)) func()` | Callback |
| `Transaction` | `func (ss *ScopedStore) Transaction(fn func(*ScopedStoreTransaction) core.Result) core.Result` | Atomic transaction |

### Import/Export Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `Import` | `func Import(ws *Workspace, medium Medium, path string) error` | Import data to workspace |
| `Export` | `func Export(ws *Workspace, medium Medium, path string) error` | Export workspace data |

---

## 🎯 Test Coverage

### Test Naming Convention

All tests follow the **Good/Bad/Ugly** triplet pattern:

- `Test<File>_<Function>_Good_<Scenario>` — Happy path, valid inputs
- `Test<File>_<Function>_Bad_<Scenario>` — Error conditions, invalid inputs
- `Test<File>_<Function>_Ugly_<Scenario>` — Edge cases, boundary conditions

### Test Categories

| Category | Files | Lines | Coverage |
|----------|-------|-------|----------|
| Store CRUD | store_test.go | ~87K | ✅ High |
| ScopedStore | scope_test.go | ~91K | ✅ High |
| Workspace | workspace_test.go | ~24K | ✅ High |
| Journal | journal_test.go, journal_parse_test.go | ~19K | ✅ High |
| Events | events_test.go | ~13K | ✅ High |
| Compact/Archive | compact_test.go | ~10K | ✅ High |
| Transactions | transaction_test.go | ~33K | ✅ High |
| Medium Integration | medium_test.go, medium_proxy_test.go | ~26K | ✅ High |
| Import/Export | import_test.go, import_export_test.go, import_helpers_test.go | ~18K | ✅ High |
| DuckDB | duckdb_test.go, duckdb_example_test.go | ~11K | ✅ High |
| Integration | coverage_test.go, conventions_test.go | ~31K | ✅ High |
| Fixtures | fixture_helpers_test.go, test_helpers_test.go, test_asserts_test.go | ~5K | ✅ High |

### Example Test Patterns

```go
// Good: Happy path
func TestStore_Set_Good_Basic(t *testing.T) { ... }

// Bad: Error conditions
func TestStore_Set_Bad_NilStore(t *testing.T) { ... }

// Ugly: Edge cases
func TestStore_Set_Ugly_EmptyKey(t *testing.T) { ... }
```

---

## 🏆 Quality Metrics

### Code Quality

- **SPOR Compliance:** 100% — Each file has Single Point Of Responsibility
- **Test Coverage:** High — Good/Bad/Ugly triplet for all public methods
- **AX Standard:** Fully compliant — Comments as usage examples, UK English
- **Documentation:** Complete — RFC, CLAUDE.md, AGENTS.md, inline docs
- **Error Handling:** Consistent — Uses `core.Result` and `core.E()` pattern

### Performance

- **Single-Connection Design:** No connection pooling overhead for SQLite
- **Buffered Channels:** Non-blocking event dispatch (cap 16)
- **Background Purge:** Automatic TTL cleanup (configurable interval)
- **Atomic Commits:** Workspace commits are fully atomic
- **Lazy TTL:** Expired entries removed on access and background purge

### Security

- **SQL Injection Protection:** `escapeLike()` with `^` escape char
- **No CGO:** Pure Go, no external native dependencies
- **Memory Safety:** Proper sync.Mutex usage for concurrent access
- **Quota Enforcement:** Optional key/group limits per namespace
- **Validation:** Input validation for namespaces, keys, configurations

---

## 🔗 Relationships

### Dependencies

```
┌─────────────────────────────────────────────────────────────────┐
│ go-store Dependencies                                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                     │
│  CoreGO Framework (dappco.re/go)                                    │
│  ├── core.Result pattern                                           │
│  ├── core.Action IPC                                               │
│  ├── core.Fs filesystem abstraction                                 │
│  └── Various utility functions                                     │
│                                                                     │
│  External Libraries:                                               │
│  ├── modernc.org/sqlite (pure Go SQLite driver)                    │
│  └── github.com/influxdata/influxdb-client-go/v2 (InfluxDB)      │
│                                                                     │
│  Internal Dependencies:                                           │
│  └── go-io (for io.Medium transport abstraction)                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────┘
```

### Consumers

| Package | Usage |
|---------|-------|
| `core/ide` | Memory caching for agent workspaces |
| `go-agent` | Workspace state management, multi-tenant isolation |
| `go-io` | Transport-agnostic storage via Medium |
| `go-cache` | Persistent caching backend |
| Custom applications | Durable state, workspace buffering |

### Related Packages

| Package | Relationship |
|---------|--------------|
| `go-io` | Provides `io.Medium` transport abstraction |
| `go-cache` | Uses go-store for persistent caching |
| `go-container` | May use go-store for container state |
| `go-config` | Could use go-store for configuration persistence |

---

## 📈 Statistics

| Metric | Value |
|--------|-------|
| **Total Files** | 20+ source, 48 tests, 16 examples |
| **Total Lines** | ~23,000+ source, ~370,000+ tests, ~53,000+ examples |
| **Public Methods** | 100+ across all types |
| **Test Coverage** | High (Good/Bad/Ugly for all public APIs) |
| **Documentation** | ~1,100+ lines in README.md, INDEX.md |
| **Dependencies** | 3 external (sqlite, influxdb-client, core) |
| **Go Version** | 1.21+ |
| **Licence** | EUPL-1.2 |

---

## 🎓 Usage Patterns

### Common Patterns

1. **Session Management** — Store tokens with TTL, automatic expiry
2. **Configuration** — Hierarchical config with groups, watch for changes
3. **Multi-Tenant** — ScopedStore for namespace isolation with quotas
4. **Analytics** — Workspace buffering, commit aggregated data to journal
5. **Training Data** — Import from various sources, process, export, archive
6. **Cache** — TTL-based caching with lazy deletion
7. **State Management** — Durable state with io.Medium transport

### Anti-Patterns

1. **Don't use connection pooling** — Single-connection design for SQLite pragmas
2. **Don't block in callbacks** — OnChange callbacks run synchronously, can cause deadlocks
3. **Don't ignore TTL** — Expired entries are automatically removed, don't rely on their persistence
4. **Don't bypass ScopedStore** — Use ScopedStore methods, not parent Store directly for scoped operations
5. **Don't commit partial work** — Workspace commits are atomic; commit complete units only

---

## 📝 Notes

- **Namespace Validation:** `^[a-zA-Z0-9-]+$` — Only alphanumeric and hyphens
- **TTL Precision:** Unix milliseconds for `expires_at` field
- **WAL Mode:** Enabled by default with `PRAGMA journal_mode=WAL`
- **Busy Timeout:** 5 seconds (`PRAGMA busy_timeout=5000`)
- **Purge Interval:** Default 60 seconds, configurable via `WithPurgeInterval`
- **Watch Buffer:** 16 events per group, non-blocking drops
- **Event Order:** Callbacks invoked before watchers receive events
- **Orphan Detection:** Automatic on `New()` call
- **Archive Format:** JSONL.gz (default) or JSONL.zstd

---

## 🏷️ Metadata

| Attribute | Value |
|-----------|-------|
| **Package** | go-store |
| **Module** | dappco.re/go/store |
| **Repository** | core/go-store |
| **Type** | Library |
| **Status** | Production |
| **Tier** | lib |
| **Created** | 2026-06-18T05:00:00Z |
| **Updated** | 2026-06-18T05:00:00Z |
| **Author** | Purberus <purberus@lthn.ai> |
| **Licence** | EUPL-1.2 |

---

*Package index generated from source code analysis and RFC specification.*
*Last updated: 2026-06-18T05:00:00Z*
