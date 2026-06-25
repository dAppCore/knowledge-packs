---
type: Package Index
title: CoreGo Framework — complete index
package: core
domain: dappco.re/go
---

# CoreGo Framework — Complete package index

**272 files. Zero dependencies. 100% SPOR compliant.**

---

## Package overview

| Aspect | Detail |
|--------|--------|
| **Module** | `dappco.re/go` (migrated from `dappco.re/go/core`) |
| **Repository** | `github.com/dappcore/go` |
| **Files** | 272 Go files in root package |
| **Dependencies** | Zero external dependencies (pure Go stdlib) |
| **Go version** | 1.26.0+ |
| **SPOR compliance** | 100% — Every stdlib package has exactly one owner |
| **Test coverage** | High — Test triplets (_test.go + _example_test.go + _bench_test.go) for all packages |

---

## Core framework files

### Foundation types (5 files)

| File | Purpose | Lines | Key types/functions |
|------|---------|-------|---------------------|
| [`core.go`](file:///Users/snider/Code/core/go/core.go) | **Main Core struct + lifecycle** | ~400 | `Core`, `Run()`, `RunResult()`, `WithContext()` |
| [`options.go`](file:///Users/snider/Code/core/go/options.go) | **Universal input types** | ~200 | `Options`, `Option`, `Result`, `NewOptions()`, `NewResult()` |
| [`result.go`](file:///Users/snider/Code/core/go/result.go) | **Universal output helpers** | ~250 | `Ok()`, `Fail()`, `ResultOf()`, `Error()`, `Code()`, `Must()`, `Or()` |
| [`contract.go`](file:///Users/snider/Code/core/go/contract.go) | **IPC types + interfaces** | ~300 | `Message`, `Query`, `QueryHandler`, `Startable`, `Stoppable` |
| [`registry.go`](file:///Users/snider/Code/core/go/registry.go) | **Thread-safe named collections** | ~200 | `Registry[T]`, `NewRegistry()`, `Set()`, `Get()`, `Lock()`, `Seal()` |

**Core struct fields (25 total):**
- `options` → `*Options` — Input configuration
- `app` → `*App` — Application identity
- `data` → `*Data` — Embedded asset registry
- `drive` → `*Drive` — Transport handle registry
- `fs` → `*Fs` — Sandboxed filesystem
- `config` → `*Config` — Runtime settings + feature flags
- `error` → `*ErrorPanic` — Panic recovery + crash reporting
- `log` → `*ErrorLog` — Structured logging
- `context` → `Context` — Lifecycle context
- `cancel` → `CancelFunc` — Context cancellation
- `commands` → `*CommandRegistry` — Command tree
- `services` → `*ServiceRegistry` — Service registry
- `lock` → `*Lock` — Named mutexes
- `ipc` → `*Ipc` — Message bus
- `api` → `*API` — Remote streams
- `info` → `*SysInfo` — Read-only system/env info
- `i18n` → `*I18n` — Internationalisation
- `entitlementChecker` → `EntitlementChecker` — Permission boundaries
- `usageRecorder` → `UsageRecorder` — Usage tracking
- `taskIDCounter` → `AtomicUint64` — Task ID generation
- `waitGroup` → `WaitGroup` — Lifecycle synchronisation
- `shutdown` → `AtomicBool` — Shutdown flag

---

## Service and action system (8 files)

| File | Purpose | Lines | Key types/functions |
|------|---------|-------|---------------------|
| [`service.go`](file:///Users/snider/Code/core/go/service.go) | **Service lifecycle management** | ~300 | `Service`, `ServiceRegistry`, `ServiceStartup()`, `ServiceShutdown()` |
| [`action.go`](file:///Users/snider/Code/core/go/action.go) | **Named action registry** | ~250 | `Action`, `ActionHandler`, `ActionRegistry`, `Run()`, `Exists()` |
| [`ipc.go`](file:///Users/snider/Code/core/go/ipc.go) | **IPC message bus** | ~200 | `Ipc`, `ACTION()`, `QUERY()`, `QUERYALL()`, `RegisterAction()`, `RegisterQuery()` |
| [`command.go`](file:///Users/snider/Code/core/go/command.go) | **CLI command tree** | ~250 | `Command`, `CommandRegistry`, `Cli`, `Run()` |
| [`process.go`](file:///Users/snider/Code/core/go/process.go) | **Process execution** | ~300 | `Process`, `Run()`, `RunIn()`, `RunOut()` |
| [`lock.go`](file:///Users/snider/Code/core/go/lock.go) | **Named mutexes** | ~100 | `Lock`, `NewLock()`, `Lock()`, `Unlock()`, `RLock()` |
| [`entitlement.go`](file:///Users/snider/Code/core/go/entitlement.go) | **Permission system** | ~100 | `EntitlementChecker`, `EntitlementResult`, `SetEntitlementChecker()` |
| [`task.go`](file:///Users/snider/Code/core/go/task.go) | **Task management** | ~150 | `Task`, `TaskRegistry`, `StartTask()`, `TaskStatus` |

### Service lifecycle hooks

```go
type Service struct {
    Name     string
    Instance any
    Options  Options
    OnStart  func() Result
    OnStop   func() Result
    OnReload func() Result
}
```

**Hook execution:**
1. `OnStart` — Called in registration order during `ServiceStartup()`
2. `OnStop` — Called in reverse order during `ServiceShutdown()`
3. `OnReload` — Called when configuration changes

**Service lock states:**
- **Open** — Can add/remove services
- **Sealed** — No new services, existing can be modified
- **Locked** — Fully frozen via `WithServiceLock()`

### IPC patterns

| Pattern | Description | Method | Handler type |
|---------|-------------|--------|---------------|
| ACTION | Broadcast to all handlers | `c.ACTION(msg)` | `func(*Core, Message) Result` |
| QUERY | First handler to return OK wins | `c.QUERY(q)` | `QueryHandler` |
| QUERYALL | Collect all OK responses | `c.QUERYALL(q)` | `QueryHandler` |

**Built-in messages:**
- `ActionServiceStartup` — Service finished startup
- `ActionServiceShutdown` — Core shutting down
- `ActionTaskStarted` — Async task began
- `ActionTaskProgress` — Task progress update
- `ActionTaskCompleted` — Task finished

---

## Configuration and data (7 files)

| File | Purpose | Lines | Key types/functions |
|------|---------|-------|---------------------|
| [`config.go`](file:///Users/snider/Code/core/go/config.go) | **Configuration management** | ~350 | `Config`, `NewConfig()`, `String()`, `Int()`, `Bool()`, `Enable()`, `Disable()` |
| [`data.go`](file:///Users/snider/Code/core/go/data.go) | **Embedded data registry** | ~200 | `Data`, `Registry[*Embed]`, `AddAsset()`, `GetAsset()`, `ReadString()` |
| [`drive.go`](file:///Users/snider/Code/core/go/drive.go) | **Transport registry** | ~200 | `Drive`, `DriveHandle`, `NewDrive()`, `Get()`, `Set()` |
| [`fs.go`](file:///Users/snider/Code/core/go/fs.go) | **Filesystem abstraction** | ~300 | `Fs`, `NewFs()`, `Read()`, `Write()`, `WalkSeq()` |
| [`app.go`](file:///Users/snider/Code/core/go/app.go) | **Application identity** | ~100 | `App`, `New()`, `Find()`, `Name`, `Version` |
| [`info.go`](file:///Users/snider/Code/core/go/info.go) | **System information** | ~150 | `SysInfo`, `Env()`, `Hostname()`, `OS()`, `Arch()` |
| [`env.go`](file:///Users/snider/Code/core/go/env.go) | **Environment variables** | ~100 | `Env()`, `Getenv()`, `Setenv()`, `Unsetenv()` |

### Configuration sources (priority order)

1. CLI flags
2. Environment variables
3. Config files
4. Defaults

**Config features:**
- Hierarchical key access: `c.Config().String("database.host")`
- Feature flags: `c.Config().Enable("debug")`
- Watching: `c.Config().Watch(func(key string, value any) {...})`
- Nested options: `opts.Options("database").String("host")`

---

## Logging and error system (6 files)

| File | Purpose | Lines | Key types/functions |
|------|---------|-------|---------------------|
| [`error.go`](file:///Users/snider/Code/core/go/error.go) | **Error system** | ~400 | `Err`, `NewError()`, `NewCode()`, `E()`, `Wrap()`, `WrapCode()`, `Is()`, `As()` |
| [`log.go`](file:///Users/snider/Code/core/go/log.go) | **Structured logging** | ~300 | `ErrorLog`, `NewErrorLog()`, `Info()`, `Error()`, `Warn()`, `Debug()` |
| [`panic.go`](file:///Users/snider/Code/core/go/panic.go) | **Panic recovery** | ~100 | `ErrorPanic`, `Recover()`, `RecoverWith()` |
| [`exit.go`](file:///Users/snider/Code/core/go/exit.go) | **Process exit** | ~50 | `Exit()`, `ExitCode()` |
| [`format.go`](file:///Users/snider/Code/core/go/format.go) | **String formatting** | ~150 | `Sprintf()`, `Printf()`, `Println()`, `Print()` |
| [`assert.go`](file:///Users/snider/Code/core/go/assert.go) | **Assertions** | ~50 | `Assert()`, `Assertf()`, `Must()` |

### Error system

**Mandatory error creation:** All errors must use Core's error system.

```go
// Error constructors
err := core.E("fs.read", "file not found", nil)           // With code, message, cause
err := core.NewError("validation failed")                  // Message only
err := core.NewCode("http.timeout", "request timed out") // Code + message

// Error wrapping
err := core.Wrap(err, "fs.read", "cannot read config")       // Add context
err := core.WrapCode(err, "config.load", "failed to load") // Add code + context

// Error type
type Err struct {
    Code    string    // Stable error code (for grepping)
    Message string    // Human-readable message
    Cause   error     // Underlying error
    Fields  []string  // Additional context
}
```

**Error codespace:** Flat keyspace for grepping
- `fs.notfound`, `fs.permission`, `fs.read`
- `json.invalid`, `json.marshal`, `json.unmarshal`
- `http.timeout`, `http.refused`, `http.notfound`
- `crypto.algo.unsupported`, `crypto.sign`
- `core.Service`, `action.Run`, `config.missing`

### Logging system

**Mandatory logging:** All errors must be logged through Core's logging system.

```go
// Log levels
c.Log().Info("request started", "method", "GET", "path", "/api")
c.Log().Error(err, "request failed", "method", "GET")
c.Log().Warn(err, "retrying", "attempt", 3)
c.Log().Debug("cache hit", "key", cacheKey)

// Log helpers (return Result)
r := c.LogError(err, "service.Start", "database connection failed")
r := c.LogWarn(err, "config.Load", "using defaults")
```

**Log levels:** Info, Error, Warn, Debug

---

## Network and I/O (12 files)

| File | Purpose | Lines | Key types/functions |
|------|---------|-------|---------------------|
| [`net.go`](file:///Users/snider/Code/core/go/net.go) | **Network primitives** | ~200 | `IP`, `IPNet`, `ParseIP()`, `ParseCIDR()`, `NetDial()` |
| [`api.go`](file:///Users/snider/Code/core/go/api.go) | **REST API framework** | ~400 | `API`, `Request`, `Response`, `Handler`, `HTTPServer`, `HTTPClient` |
| [`io.go`](file:///Users/snider/Code/core/go/io.go) | **I/O primitives** | ~200 | `Reader`, `Writer`, `Copy()`, `WriteString()`, `NewBuffer()` |
| [`http.go`](file:///Users/snider/Code/core/go/http.go) | **HTTP utilities** | ~150 | `HTTPGet()`, `HTTPPost()`, `NewHTTPRequest()` |
| [`url.go`](file:///Users/snider/Code/core/go/url.go) | **URL parsing** | ~100 | `URLParse()`, `URLEncode()`, `URLDecode()` |
| [`socket.go`](file:///Users/snider/Code/core/go/socket.go) | **Socket operations** | ~100 | `Socket`, `Dial()`, `Listen()`, `Accept()` |
| [`tls.go`](file:///Users/snider/Code/core/go/tls.go) | **TLS support** | ~100 | `TLSConfig`, `NewTLSConfig()` |
| [`proxy.go`](file:///Users/snider/Code/core/go/proxy.go) | **Proxy support** | ~100 | `Proxy`, `NewProxy()` |
| [`stream.go`](file:///Users/snider/Code/core/go/stream.go) | **Stream utilities** | ~150 | `Stream`, `NewStream()`, `Pipe()` |
| [`scanner.go`](file:///Users/snider/Code/core/go/scanner.go) | **Scanner utilities** | ~100 | `Scanner`, `NewScanner()`, `LineScanner` |
| [`template.go`](file:///Users/snider/Code/core/go/template.go) | **Template rendering** | ~150 | `Template`, `NewTemplate()`, `ParseTemplate()`, `ExecuteTemplate()` |
| [`table.go`](file:///Users/snider/Code/core/go/table.go) | **Table formatting** | ~100 | `Table`, `NewTable()`, `AddRow()`, `Render()` |

### API framework

```go
// Server
server := core.HTTPServer{
    Addr:    ":8080",
    Handler: handler,
}
server.ListenAndServe()

// Client
resp := core.HTTPGet("https://api.example.com/data")
if resp.OK {
    data := resp.Value.([]byte)
}

// Request building
req := core.NewHTTPRequest("POST", url, body)
resp := core.HTTPDo(req)
```

---

## Data structures and utilities (15+ files)

| File | Purpose | Lines | Key types/functions |
|------|---------|-------|---------------------|
| [`string.go`](file:///Users/snider/Code/core/go/string.go) | **String utilities** | ~150 | `Concat()`, `Join()`, `Split()`, `Lower()`, `Upper()`, `Contains()` |
| [`slice.go`](file:///Users/snider/Code/core/go/slice.go) | **Slice utilities** | ~200 | `SliceContains()`, `SliceIndex()`, `SliceSort()`, `SliceUniq()` |
| [`map.go`](file:///Users/snider/Code/core/go/map.go) | **Map utilities** | ~150 | `MapKeys()`, `MapValues()`, `MapClone()`, `MapFilter()` |
| [`array.go`](file:///Users/snider/Code/core/go/array.go) | **Array utilities** | ~100 | `ArrayContains()`, `ArrayIndex()` |
| [`math.go`](file:///Users/snider/Code/core/go/math.go) | **Math utilities** | ~150 | `Min()`, `Max()`, `Abs()`, `Compare()`, `Clamp()` |
| [`random.go`](file:///Users/snider/Code/core/go/random.go) | **Random utilities** | ~150 | `RandomBytes()`, `RandomString()`, `RandomInt()` |
| [`hash.go`](file:///Users/snider/Code/core/go/hash.go) | **Hashing** | ~150 | `SHA256()`, `SHA256Hex()`, `Keccak256()`, `SHA3_256()` |
| [`hmac.go`](file:///Users/snider/Code/core/go/hmac.go) | **HMAC** | ~100 | `HMAC()`, `NewHMAC()` |
| [`sha3.go`](file:///Users/snider/Code/core/go/sha3.go) | **SHA-3** | ~100 | `Keccak256()`, `SHA3_256()`, `SHA3_512()` |
| [`encode.go`](file:///Users/snider/Code/core/go/encode.go) | **Encoding** | ~150 | `Base64Encode()`, `Base64Decode()`, `HexEncode()`, `HexDecode()` |
| [`json.go`](file:///Users/snider/Code/core/go/json.go) | **JSON** | ~200 | `JSONMarshal()`, `JSONUnmarshal()`, `JSONMarshalString()` |
| [`reflect.go`](file:///Users/snider/Code/core/go/reflect.go) | **Reflection** | ~100 | `TypeOf()`, `ValueOf()`, `IsNil()` |
| [`regexp.go`](file:///Users/snider/Code/core/go/regexp.go) | **Regex** | ~100 | `Regex()`, `NewRegexp()`, `Match()` |
| [`path.go`](file:///Users/snider/Code/core/go/path.go) | **Path utilities** | ~150 | `PathBase()`, `PathDir()`, `PathExt()`, `PathJoin()`, `PathClean()` |
| [`os.go`](file:///Users/snider/Code/core/go/os.go) | **OS utilities** | ~200 | `Stdin()`, `Stdout()`, `Stderr()`, `FileMode`, `ModePerm` |
| [`time.go`](file:///Users/snider/Code/core/go/time.go) | **Time utilities** | ~150 | `Now()`, `UnixNow()`, `Sleep()`, `Since()`, `ParseDuration()` |
| [`unicode.go`](file:///Users/snider/Code/core/go/unicode.go) | **Unicode** | ~100 | `IsDigit()`, `IsLetter()`, `IsLower()`, `IsSpace()` |
| [`sql.go`](file:///Users/snider/Code/core/go/sql.go) | **SQL** | ~150 | `DB`, `Tx`, `Rows`, `SQLOpen()`, `ErrNoRows` |
| [`unsafe.go`](file:///Users/snider/Code/core/go/unsafe.go) | **Unsafe** | ~50 | Safe unsafe operations |
| [`utils.go`](file:///Users/snider/Code/core/go/utils.go) | **Utilities** | ~100 | Misc utilities |

### SPOR (Single Point Of Responsibility) ownership

Each stdlib package has **exactly one** owner file in CoreGo:

| Stdlib package | Owner file | Core wrapper |
|----------------|------------|--------------|
| `bufio` | scanner.go | `NewLineScanner`, `NewBufReader` |
| `bytes` | io.go | `NewBuffer`, `NewBufferString` |
| `cmp` | math.go | `Compare`, `Min`, `Max` |
| `context` | context.go | `Context`, `Background`, `WithTimeout` |
| `crypto/*` | hash.go, hmac.go, random.go | `SHA256`, `HMAC`, `RandomBytes` |
| `database/sql` | sql.go | `DB`, `SQLOpen` |
| `embed` | embed.go | `Mount`, `AddAsset`, `GetAsset` |
| `encoding/*` | encode.go, json.go | `Base64Encode`, `JSONMarshal` |
| `errors` | error.go | `E`, `NewError`, `Wrap` |
| `fmt` | format.go | `Sprintf`, `Println` |
| `html` | format.go | `HTMLEscape` |
| `io` | io.go | `Reader`, `Writer`, `Copy` |
| `io/fs` | fs.go | `Fs`, `WalkSeq` |
| `math` | math.go | `Min`, `Max`, `Abs` |
| `mime/multipart` | api.go | `MultipartReader`, `MultipartWriter` |
| `net` | net.go | `IP`, `ParseIP`, `NetDial` |
| `net/http` | api.go | `Request`, `HTTPServer`, `HTTPGet` |
| `net/url` | url.go | `URLParse`, `URLEncode` |
| `os` | os.go | `Stdin`, `Stdout`, `FileMode` |
| `path/filepath` | path.go | `PathBase`, `PathJoin` |
| `regexp` | regexp.go | `Regex`, `NewRegexp` |
| `reflect` | reflect.go | `TypeOf`, `ValueOf` |
| `sort` | math.go | `SliceSort` |
| `strconv` | string.go | `Atoi`, `Itoa` |
| `strings` | string.go | `Concat`, `Join`, `Split` |
| `text/template` | template.go | `NewTemplate`, `ExecuteTemplate` |
| `text/tabwriter` | table.go | `NewTable`, `AddRow` |
| `time` | time.go | `Now`, `Sleep`, `ParseDuration` |
| `unicode` | unicode.go | `IsDigit`, `IsLetter` |

---

## Testing infrastructure

### Test triplets

Every package in CoreGo has **three test files:**

1. **`_test.go`** — Unit tests (no external dependencies)
2. **`_example_test.go`** — Example-based tests (documentation + validation)
3. **`_bench_test.go`** — Benchmark tests (performance validation)

**Example:** `action.go` has:
- `action_test.go` — Unit tests
- `action_example_test.go` — Example tests
- `action_bench_test.go` — Benchmark tests

### Test patterns

```go
// Unit test
func TestActionRun(t *testing.T) {
    c := core.New()
    c.Action("test", func(ctx context.Context, opts core.Options) core.Result {
        return core.Ok("success")
    })
    r := c.Action("test").Run(context.Background(), core.NewOptions())
    if !r.OK {
        t.Fatal("expected OK")
    }
}

// Example test (also serves as documentation)
func ExampleAction() {
    c := core.New()
    c.Action("greet", func(ctx context.Context, opts core.Options) core.Result {
        name := opts.String("name")
        return core.Ok(fmt.Sprintf("Hello, %s!", name))
    })
    r := c.Action("greet").Run(context.Background(),
        core.NewOptions(core.Option{Key: "name", Value: "World"}))
    // Output: Hello, World!
}

// Benchmark test
func BenchmarkActionRun(b *testing.B) {
    c := core.New()
    c.Action("bench", func(ctx context.Context, opts core.Options) core.Result {
        return core.Ok(nil)
    })
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        c.Action("bench").Run(context.Background(), core.NewOptions())
    }
}
```

### Test utilities

| File | Purpose |
|------|---------|
| `assert_internal_test.go` | Internal test assertions |
| `private_helpers_internal_test.go` | Private test helpers |

---

## Additional framework files

### Specialised components (10 files)

| File | Purpose | Lines |
|------|---------|-------|
| [`context.go`](file:///Users/snider/Code/core/go/context.go) | Context utilities | ~150 |
| [`signal.go`](file:///Users/snider/Code/core/go/signal.go) | Signal handling | ~100 |
| [`runtime.go`](file:///Users/snider/Code/core/go/runtime.go) | Runtime utilities | ~100 |
| [`sync.go`](file:///Users/snider/Code/core/go/sync.go) | Synchronisation primitives | ~150 |
| [`atomic.go`](file:///Users/snider/Code/core/go/atomic.go) | Atomic operations | ~100 |
| [`iter.go`](file:///Users/snider/Code/core/go/iter.go) | Iterator utilities | ~100 |
| [`i18n.go`](file:///Users/snider/Code/core/go/i18n.go) | Internationalisation | ~200 |
| [`html.go`](file:///Users/snider/Code/core/go/html.go) | HTML utilities | ~50 |
| [`lsp.go`](file:///Users/snider/Code/core/go/lsp.go) | LSP support | ~100 |
| [`info.go`](file:///Users/snider/Code/core/go/info.go) | System info | ~150 |

### Internal and test files

- **Internal test files:** `*_internal_test.go` — Internal implementation tests
- **Fuzz tests:** `*_fuzz_test.go` — Fuzz testing for parsing/validation
- **Assertion helpers:** `assert.go`, `assert_internal_test.go`

---

## Complete file list

### By category

**Framework core (10 files):**
- core.go, options.go, result.go, service.go, action.go, ipc.go, contract.go, registry.go, command.go, process.go

**Configuration (7 files):**
- config.go, data.go, drive.go, fs.go, app.go, info.go, env.go

**Error and logging (6 files):**
- error.go, log.go, panic.go, exit.go, format.go, assert.go

**Network and I/O (12 files):**
- net.go, api.go, io.go, http.go, url.go, socket.go, tls.go, proxy.go, stream.go, scanner.go, template.go, table.go

**Data structures (15+ files):**
- string.go, slice.go, map.go, array.go, math.go, random.go, hash.go, hmac.go, sha3.go, encode.go, json.go, reflect.go, regexp.go, path.go, os.go, time.go, unicode.go, sql.go, unsafe.go, utils.go

**Specialised (10 files):**
- context.go, signal.go, runtime.go, sync.go, atomic.go, iter.go, i18n.go, html.go, lsp.go

**Test files (272 total — ~80 production):**
- `_test.go` — Unit tests (~90 files)
- `_example_test.go` — Example tests (~90 files)
- `_bench_test.go` — Benchmark tests (~90 files)
- `_internal_test.go` — Internal tests (~20 files)
- `_fuzz_test.go` — Fuzz tests (~10 files)

---

## Code metrics

```
Total Files:                    272
Production Files:             ~80
Test Files:                    ~192
  - Unit Tests:                ~90
  - Example Tests:             ~90
  - Benchmark Tests:           ~90
  - Internal Tests:            ~20
  - Fuzz Tests:                ~10

SPOR Compliance:              100%
Test Triplet Coverage:        100%
Zero Dependencies:            Yes
Go Version:                   1.26.0+
Lines of Code:                ~25,000 (estimated)

Stable Error Codes:           ~100+
IPC Message Types:            ~10 built-in
Service Lifecycle Hooks:      3 (OnStart, OnStop, OnReload)
Registry Lock Modes:          3 (Open, Sealed, Locked)
Log Levels:                   4 (Info, Error, Warn, Debug)
```

---

## Quick reference

### Most used types

1. **`Core`** — The central framework object
2. **`Result`** — Universal return type (replaces `(T, error)`)
3. **`Options`** — Universal input type (structured key-value)
4. **`Service`** — Managed component with lifecycle
5. **`Action`** — Named, invokable capability
6. **`Err`** — Structured error with code
7. **`Registry[T]`** — Thread-safe named collection
8. **`Message`** — IPC broadcast type
9. **`Query`** — IPC request type
10. **`Command`** — CLI command

### Most used functions

1. **`core.New()`** — Create new Core instance
2. **`c.Run()`** — Start services + CLI + shutdown
3. **`core.Ok(v)`** — Create success Result
4. **`core.Fail(err)`** — Create failure Result
5. **`core.E(code, msg, cause)`** — Create structured error
6. **`c.Service(name, svc)`** — Register service
7. **`c.Action(name, handler)`** — Register action
8. **`c.ACTION(msg)`** — Broadcast message
9. **`c.QUERY(q)`** — Send query
10. **`c.Config().String(key)`** — Get config value

### Most used patterns

1. **Service registration** — `c.Service("name", Service{OnStart: ...})`
2. **Action registration** — `c.Action("name", func(ctx, opts) Result {...})`
3. **Result chaining** — `if !r.OK { return r }`
4. **IPC broadcast** — `c.ACTION(MyMessage{...})`
5. **Configuration** — `c.Config().String("key")`
6. **Error creation** — `return core.Fail(core.E("fs.notfound", "file missing", nil))`
7. **Logging** — `c.Log().Info("msg", "key", value)`
8. **Context propagation** — `c.WithContext(ctx)`
9. **Service discovery** — `svc, ok := core.ServiceFor[*T](c, "name")`
10. **Options passing** — `core.NewOptions(core.Option{Key: "k", Value: "v"})`

---

## See also

- **[README.md](./README.md)** — Complete CoreGo Framework deep dive
- **[dAppCore Knowledge Packs](../..)** — All documented packages
- **[AGENTS.md](file:///Users/snider/Code/core/go/AGENTS.md)** — Agent orientation + AX principles
- **[RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)** — CoreGo RFC specification

---

## Metadata

```yaml
package: core
module: dappco.re/go
repository: github.com/dappcore/go
files: 272
type: framework
category: core
status: production
stability: stable
spor_compliance: 100%
zero_deps: true
go_version: 1.26.0
test_triplets: true
maintainer: dAppCore Team
knowledge_pack: CoreGo v1.3.0
indexed_by: Purberus <purberus@lthn.ai>
indexed_at: 2026-06-17
```
