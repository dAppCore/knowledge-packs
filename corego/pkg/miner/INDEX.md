---
type: Package Index
title: go-miner Package Index
description: Complete index of go-miner package components and API surface
description: Mining software controller library with unified Miner interface for XMRig, TT-Miner, and simulated miners, including Manager, Factory, ProfileManager, EventBus, and resilience utilities
module: dappco.re/go/core/miner
repo: core/go-miner
---

# go-miner — Package Index

---

## Quick links

- **[README.md](./README.md)** — Complete package documentation
- **[RFC Specification](file:///Users/snider/Code/meowmix/plans/code/core/go/miner/RFC.md)** — Technical specification (self-contained, agent-implementable)
- **[CLAUDE.md](file:///Users/snider/Code/core/go-miner/CLAUDE.md)** — Implementation details, file map, conventions, banned imports
- **[AGENTS.md](file:///Users/snider/Code/core/go-miner/AGENTS.md)** — Agent guidance

### Sub-specifications

The RFC includes comprehensive sub-specifications:

- **Section 1:** Overview — Package purpose, scope, supported miners
- **Section 2:** File Map — All source files and their purposes
- **Section 3:** Miner Interface — Complete interface definition with usage examples
- **Section 4:** Types — Config, PerformanceMetrics, HashratePoint, InstallationDetails, etc.
- **Section 5:** Miner Types — XMRig, TT-Miner, Simulated implementations
- **Section 6:** Manager — Multi-miner lifecycle coordinator
- **Section 7:** Factory — MinerFactory registry and CreateMiner
- **Section 8:** Profiles — ProfileManager CRUD operations
- **Section 9:** Events — EventBus, event types, MinerEventData
- **Section 10:** Errors — MinerError typed errors
- **Sections 11-20:** Additional types and utilities

### Local documentation

- [CLAUDE.md](file:///Users/snider/Code/core/go-miner/CLAUDE.md) — Development conventions, test patterns
- [AGENTS.md](file:///Users/snider/Code/core/go-miner/AGENTS.md) — Agent-specific guidance
- Additional docs in package repository

---

## File structure

### Core package files (`go/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `miner.go` | 508 | `Miner` interface + `BaseMiner` shared implementation | ✅ Complete |
| `manager.go` | 133 | `Manager` — multi-miner lifecycle coordinator | ✅ Complete |
| `factory.go` | 275 | `MinerFactory` registry + `CreateMiner` | ✅ Complete |
| `types.go` | 18,276 | All type definitions: Config, PerformanceMetrics, HashratePoint, InstallationDetails, AvailableMiner, SystemInfo, API | ✅ Complete |
| `service.go` | 363 | Core lifecycle integration, Action registration | ✅ Complete |
| `errors.go` | 567 | `MinerError` typed errors + constructor functions | ✅ Complete |
| `xmrig.go` | 1,584 | `XMRigMiner` adapter — Install, CheckInstallation, config generation | ✅ Complete |
| `xmrig_start.go` | 853 | `XMRigMiner.Start` — process launch, config file write | ✅ Complete |
| `ttminer.go` | 1,730 | `TTMiner` adapter — Install, Start, CheckInstallation | ✅ Complete |
| `simulated.go` | 2,220 | `SimulatedMiner` — test double with deterministic hashrate | ✅ Complete |
| `profile.go` | 3,476 | `ProfileManager` CRUD backed by go-store | ✅ Complete |
| `events.go` | 4,387 | `EventBus`, event types, MinerEventData, MinerStatsData | ✅ Complete |
| `metrics.go` | 1,057 | `PerformanceMetrics`, atomic counters, `LatencyHistogram` | ✅ Complete |
| `circuit_breaker.go` | 1,659 | `CircuitBreaker` — closed/open/half-open with result cache | ✅ Complete |
| `rate_limiter.go` | 603 | `RateLimiter` — token bucket, per-IP, idle eviction | ✅ Complete |
| `supervisor.go` | 2,145 | `TaskSupervisor` — panic-recovering restart loop | ✅ Complete |
| `log_buffer.go` | 1,171 | `LogBuffer` — timestamped ring buffer satisfying `io.Writer` | ✅ Complete |
| `http_stats.go` | 310 | `FetchJSONStats[T]` generic HTTP stats fetcher | ✅ Complete |

### API subpackage (`go/pkg/api/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `provider.go` | ~230 | REST endpoint provider (mount on parent router) | ✅ Complete |

**Total Core Source Lines:** ~50,000+

### Test files (`go/`)

#### Unit tests

| File | Lines | Purpose | Pattern |
|------|-------|---------|---------|
| `miner_test.go` | 70,753 | Miner interface tests | Good/Bad/Ugly |
| `miners_impl_test.go` | 58,838 | XMRig, TT-Miner, Simulated implementation tests | Good/Bad/Ugly |
| `manager_test.go` | 24,652 | Manager lifecycle tests | Good/Bad/Ugly |
| `factory_test.go` | 5,215 | MinerFactory tests | Good/Bad/Ugly |
| `profile_test.go` | 6,027 | ProfileManager tests | Good/Bad/Ugly |
| `events_test.go` | 2,310 | EventBus tests | Good/Bad/Ugly |
| `metrics_test.go` | 2,146 | Metrics and hashrate history tests | Good/Bad/Ugly |
| `circuit_breaker_test.go` | 684 | CircuitBreaker tests | Good/Bad/Ugly |
| `rate_limiter_test.go` | - | RateLimiter tests | Good/Bad/Ugly |
| `supervisor_test.go` | 6,723 | TaskSupervisor tests | Good/Bad/Ugly |
| `log_buffer_test.go` | 5,005 | LogBuffer tests | Good/Bad/Ugly |
| `http_stats_test.go` | - | HTTP stats tests | Good/Bad/Ugly |
| `service_test.go` | 56,402 | Service integration tests | Good/Bad/Ugly |
| `types_test.go` | 46,855 | Type validation tests | Good/Bad/Ugly |
| `util_impl_test.go` | 41,285 | Utility function tests | Good/Bad/Ugly |
| `service_impl_test.go` | 48,773 | Service implementation tests | Good/Bad/Ugly |
| `manager_service_impl_test.go` | 37,232 | Manager service implementation tests | Good/Bad/Ugly |
| `miner_install_test.go` | 8,811 | Miner installation tests | Good/Bad/Ugly |
| `simulated_lifecycle_test.go` | 7,913 | Simulated miner lifecycle tests | Good/Bad/Ugly |
| `archive_extract_test.go` | 7,218 | Archive extraction tests | Good/Bad/Ugly |
| `service_doctor_metrics_test.go` | 5,043 | Service doctor metrics tests | Good/Bad/Ugly |
| `service_list_helpers_test.go` | 5,329 | Service list helpers tests | Help |
| `service_nil_guard_test.go` | 4,782 | Service nil guard tests | Good/Bad |
| `service_profile_handlers_test.go` | 5,091 | Service profile handlers tests | Good/Bad |
| `ax7_triplets_test.go` | 265,465 | Comprehensive triplet tests | Good/Bad/Ugly |
| `base_miner_paths_test.go` | 8,830 | Base miner paths tests | Good/Bad/Ugly |

**Total Test Lines:** ~800,000+

#### Example tests

| File | Lines | Purpose |
|------|-------|---------|
| `miner_example_test.go` | - | Basic miner usage examples |
| `miners_impl_example_test.go` | 7,291 | Adapter-specific examples |
| `manager_service_impl_example_test.go` | 2,220 | Manager service examples |
| `service_impl_example_test.go` | 1,130 | Service integration examples |
| `circuit_breaker_example_test.go` | 94 | CircuitBreaker usage examples |
| `errors_example_test.go` | 1,167 | Error handling examples |
| `events_example_test.go` | 271 | Event system examples |
| `factory_example_test.go` | 734 | Factory usage examples |
| `log_buffer_example_test.go` | 674 | LogBuffer examples |
| `profile_example_test.go` | 754 | Profile management examples |
| `types_example_test.go` | 1,167 | Type usage examples |
| `util_impl_example_test.go` | 1,018 | Utility examples |

**Total Example Lines:** ~20,000+

---

## Public API surface

### Miner interface

The unified interface for all mining adapters:

```go
type Miner interface {
    // Lifecycle
    Install() error
    Uninstall() error
    Start(config *Config) error
    Stop() error
    IsRunning() bool
    
    // Information
    GetName() string
    GetType() string
    GetPath() string
    GetBinaryPath() string
    CheckInstallation() (*InstallationDetails, error)
    GetLatestVersion() (string, error)
    
    // Monitoring
    GetStats(ctx context.Context) (*PerformanceMetrics, error)
    GetHashrateHistory() []HashratePoint
    AddHashratePoint(point HashratePoint)
    ReduceHashrateHistory(now time.Time)
    GetLogs() []string
    WriteStdin(input string) error
}
```

### Miner types

```go
const (
    MinerTypeXMRig      = "xmrig"
    MinerTypeTTMiner    = "ttminer"
    MinerTypeSimulated  = "simulated"
)
```

### BaseMiner

Shared implementation providing common functionality:

```go
type BaseMiner struct {
    name        string
    minerType   string
    path        string
    binaryPath  string
    process     *core.Process
    config      *Config
    
    // Hashrate history (two-tier)
    highResHashrate []HashratePoint  // 10s intervals, 5 min retention
    lowResHashrate  []HashratePoint  // 1 min intervals, 24h retention
    
    // Log capture
    logBuffer *LogBuffer
    
    // Lifecycle
    mu         sync.RWMutex
    isRunning bool
    startTime time.Time
}
```

### Configuration types

```go
// Config holds all parameters for starting a mining instance
type Config struct {
    // Connection
    Pool    string `json:"pool"`
    Wallet  string `json:"wallet"`
    TLS     bool   `json:"tls,omitempty"`
    
    // Algorithm
    Algo    string `json:"algo"`
    Coin    string `json:"coin,omitempty"`
    
    // CPU Settings
    Threads            int  `json:"threads,omitempty"`
    HugePages          bool `json:"hugePages,omitempty"`
    CPUMaxThreadsHint int  `json:"cpuMaxThreadsHint,omitempty"`
    CPUPriority        int  `json:"cpuPriority,omitempty"`
    
    // GPU Settings
    GPUEnabled    bool   `json:"gpuEnabled,omitempty"`
    GPUPool       string `json:"gpuPool,omitempty"`
    GPUWallet     string `json:"gpuWallet,omitempty"`
    GPUPassword   string `json:"gpuPassword,omitempty"`
    GPUAlgo       string `json:"gpuAlgo,omitempty"`
    OpenCL        bool   `json:"opencl,omitempty"`
    CUDA          bool   `json:"cuda,omitempty"`
    Devices       string `json:"devices,omitempty"`
    GPUIntensity  int    `json:"gpuIntensity,omitempty"`
    GPUThreads    int    `json:"gpuThreads,omitempty"`
    
    // HTTP API
    HTTPPort int `json:"httpPort,omitempty"`
    
    // Logging
    LogOutput bool `json:"logOutput,omitempty"`
    
    // Lifecycle
    Autostart       bool `json:"autostart,omitempty"`
    PauseOnActive   bool `json:"pauseOnActive,omitempty"`
    PauseOnBattery  bool `json:"pauseOnBattery,omitempty"`
    
    // Metadata
    MinerType string `json:"minerType,omitempty"`
}

// PerformanceMetrics is the normalised stats payload
type PerformanceMetrics struct {
    Hashrate      int                    `json:"hashrate"`
    Shares        int                    `json:"shares"`
    Rejected      int                    `json:"rejected"`
    Uptime        int                    `json:"uptime"`
    LastShare     int64                  `json:"lastShare,omitempty"`
    Algorithm     string                 `json:"algorithm,omitempty"`
    AvgDifficulty int                    `json:"avgDifficulty,omitempty"`
    DiffCurrent   int                    `json:"diffCurrent,omitempty"`
    ExtraData     map[string]interface{} `json:"extraData,omitempty"`
}

// HashratePoint is a single hashrate measurement
type HashratePoint struct {
    Timestamp time.Time `json:"timestamp"`
    Hashrate  int       `json:"hashrate"`
}

// InstallationDetails is the result of CheckInstallation
type InstallationDetails struct {
    IsInstalled bool   `json:"isInstalled"`
    MinerBinary string `json:"minerBinary,omitempty"`
    Path        string `json:"path,omitempty"`
    Version     string `json:"version,omitempty"`
    ConfigPath  string `json:"configPath,omitempty"`
}

// AvailableMiner describes a supported miner type
type AvailableMiner struct {
    MinerType    string   `json:"minerType"`
    DisplayName string   `json:"displayName"`
    Description string   `json:"description"`
    SupportedAlgos []string `json:"supportedAlgos"`
    Platforms    []string `json:"platforms"`
    IsGPU        bool     `json:"isGPU"`
}

// SystemInfo contains system hardware information
type SystemInfo struct {
    CPU         CPUInfo    `json:"cpu"`
    GPU         []GPUInfo `json:"gpu"`
    Memory      MemoryInfo `json:"memory"`
    OS          OSInfo    `json:"os"`
}
```

### Manager types

```go
type Manager struct {
    miners        []Miner
    eventBus     *EventBus
    runningMiners sync.Map
    mu            sync.RWMutex
}

// CombinedMetrics aggregates stats from all miners
type CombinedMetrics struct {
    TotalHashrate  int
    TotalShares    int
    TotalRejected  int
    MinerCount     int
    RunningCount   int
}
```

### ProfileManager types

```go
type ProfileManager struct {
    store *store.Store
    mu    sync.RWMutex
}

type MiningProfile struct {
    Name       string
    Config     *Config
    MinerType string
    CreatedAt time.Time
    UpdatedAt time.Time
}

type RawConfig struct {
    Raw string `json:"raw"`
}
```

### Event types

```go
type EventType int

const (
    EventMinerInstalled EventType = iota
    EventMinerUninstalled
    EventMinerStarted
    EventMinerStopped
    EventMinerStats
    EventMinerError
)

type MinerEventData struct {
    MinerName string
    MinerType string
    Timestamp time.Time
    Data      map[string]any
}

type MinerStatsData struct {
    MinerName string
    Metrics   *PerformanceMetrics
    Timestamp time.Time
}

type EventBus struct {
    hub          *stream.Hub
    subscribers map[EventType][]chan MinerEventData
    allSubscribers []chan MinerEventData
    mu           sync.RWMutex
}
```

### Utility types

```go
// CircuitBreaker states
type CircuitState int
const (
    CircuitClosed CircuitState = iota
    CircuitOpen
    CircuitHalfOpen
)

type CircuitBreaker struct {
    state        CircuitState
    failureCount int
    successCount int
    lastFailure  time.Time
    lastSuccess  time.Time
    timeout      time.Duration
    maxFailures  int
    mu           sync.RWMutex
}

type RateLimiter struct {
    tokens     float64
    maxTokens  float64
    refillRate float64
    lastRefill time.Time
    mu         sync.Mutex
}

type TaskSupervisor struct {
    task        func()
    onPanic     func(recovered any)
    maxRestarts int
    restartCount int
    stop        chan struct{}
    done        chan struct{}
    mu          sync.Mutex
}

type LogLine struct {
    Timestamp time.Time
    Line      string
}

type LogBuffer struct {
    lines    []LogLine
    capacity int
    head     int
    count    int
    mu       sync.Mutex
}
```

---

## Public API methods

### Miner factory

| Method | Signature | Description |
|--------|-----------|-------------|
| `CreateMiner` | `func CreateMiner(minerType, instanceName string) (Miner, error)` | Create a miner instance by type |

### Miner interface methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `Install` | `func (m Miner) Install() error` | Download and extract miner binary |
| `Uninstall` | `func (m Miner) Uninstall() error` | Remove miner binary and configs |
| `Start` | `func (m Miner) Start(config *Config) error` | Start mining with config |
| `Stop` | `func (m Miner) Stop() error` | Stop the miner process |
| `IsRunning` | `func (m Miner) IsRunning() bool` | Check if miner is running |
| `GetName` | `func (m Miner) GetName() string` | Get miner instance name |
| `GetType` | `func (m Miner) GetType() string` | Get miner type (xmrig, ttminer, simulated) |
| `GetPath` | `func (m Miner) GetPath() string` | Get installation directory |
| `GetBinaryPath` | `func (m Miner) GetBinaryPath() string` | Get binary path |
| `CheckInstallation` | `func (m Miner) CheckInstallation() (*InstallationDetails, error)` | Verify binary exists |
| `GetLatestVersion` | `func (m Miner) GetLatestVersion() (string, error)` | Fetch latest version from GitHub |
| `GetStats` | `func (m Miner) GetStats(ctx context.Context) (*PerformanceMetrics, error)` | Get current performance metrics |
| `GetHashrateHistory` | `func (m Miner) GetHashrateHistory() []HashratePoint` | Get hashrate history |
| `AddHashratePoint` | `func (m Miner) AddHashratePoint(point HashratePoint)` | Add a hashrate datapoint |
| `ReduceHashrateHistory` | `func (m Miner) ReduceHashrateHistory(now time.Time)` | Reduce high-res to low-res |
| `GetLogs` | `func (m Miner) GetLogs() []string` | Get captured log lines |
| `WriteStdin` | `func (m Miner) WriteStdin(input string) error` | Send command to stdin |

### Manager methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewManager` | `func NewManager() *Manager` | Create a new manager |
| `AddMiner` | `func (m *Manager) AddMiner(miner Miner)` | Add a miner to the manager |
| `RemoveMiner` | `func (m *Manager) RemoveMiner(miner Miner)` | Remove a miner |
| `GetMiners` | `func (m *Manager) GetMiners() []Miner` | Get all managed miners |
| `StartAll` | `func (m *Manager) StartAll(config *Config) error` | Start all miners with config |
| `StartAllFromProfiles` | `func (m *Manager) StartAllFromProfiles() error` | Start all from saved profiles |
| `StopAll` | `func (m *Manager) StopAll() error` | Stop all miners |
| `GetCombinedStats` | `func (m *Manager) GetCombinedStats() *CombinedMetrics` | Get aggregated stats |
| `GetEventBus` | `func (m *Manager) GetEventBus() *EventBus` | Get the event bus |

### ProfileManager methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewProfileManager` | `func NewProfileManager() (*ProfileManager, error)` | Create profile manager |
| `SaveProfile` | `func (pm *ProfileManager) SaveProfile(profile *MiningProfile) error` | Save a profile |
| `GetProfile` | `func (pm *ProfileManager) GetProfile(name string) (*MiningProfile, error)` | Get a profile by name |
| `GetAllProfiles` | `func (pm *ProfileManager) GetAllProfiles() ([]*MiningProfile, error)` | Get all profiles |
| `DeleteProfile` | `func (pm *ProfileManager) DeleteProfile(name string) error` | Delete a profile |
| `UpdateProfile` | `func (pm *ProfileManager) UpdateProfile(profile *MiningProfile) error` | Update a profile |
| `StartFromProfile` | `func (pm *ProfileManager) StartFromProfile(name string) (Miner, error)` | Start miner from profile |

### EventBus methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `Subscribe` | `func (eb *EventBus) Subscribe(eventType EventType) chan MinerEventData` | Subscribe to specific event type |
| `SubscribeAll` | `func (eb *EventBus) SubscribeAll() chan MinerEventData` | Subscribe to all events |
| `Unsubscribe` | `func (eb *EventBus) Unsubscribe(eventType EventType, ch chan MinerEventData)` | Unsubscribe from event type |
| `UnsubscribeAll` | `func (eb *EventBus) UnsubscribeAll(ch chan MinerEventData)` | Unsubscribe from all events |
| `Emit` | `func (eb *EventBus) Emit(eventType EventType, data MinerEventData)` | Emit an event |

### Utility methods

#### CircuitBreaker

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewCircuitBreaker` | `func NewCircuitBreaker(maxFailures int, timeout time.Duration) *CircuitBreaker` | Create circuit breaker |
| `Execute` | `func (cb *CircuitBreaker) Execute(fn func() error) error` | Execute with circuit protection |
| `State` | `func (cb *CircuitBreaker) State() CircuitState` | Get current state |
| `Reset` | `func (cb *CircuitBreaker) Reset()` | Reset to closed state |

#### RateLimiter

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewRateLimiter` | `func NewRateLimiter(rate float64, per time.Duration) *RateLimiter` | Create rate limiter |
| `Allow` | `func (rl *RateLimiter) Allow() bool` | Check if request is allowed |
| `Wait` | `func (rl *RateLimiter) Wait(ctx context.Context) error` | Wait for token availability |

#### TaskSupervisor

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewTaskSupervisor` | `func NewTaskSupervisor(task func(), onPanic func(any), maxRestarts int) *TaskSupervisor` | Create supervisor |
| `Start` | `func (ts *TaskSupervisor) Start()` | Start the supervised task |
| `Stop` | `func (ts *TaskSupervisor) Stop()` | Stop the supervisor |
| `IsRunning` | `func (ts *TaskSupervisor) IsRunning() bool` | Check if running |

#### LogBuffer

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewLogBuffer` | `func NewLogBuffer(capacity int) *LogBuffer` | Create log buffer |
| `Write` | `func (lb *LogBuffer) Write(p []byte) (n int, err error)` | Write to buffer (io.Writer) |
| `GetLines` | `func (lb *LogBuffer) GetLines() []LogLine` | Get all lines with timestamps |
| `GetStrings` | `func (lb *LogBuffer) GetStrings() []string` | Get all lines as strings |
| `Clear` | `func (lb *LogBuffer) Clear()` | Clear the buffer |

#### HTTP stats

| Method | Signature | Description |
|--------|-----------|-------------|
| `FetchJSONStats` | `func FetchJSONStats[T any](ctx context.Context, url string) (*T, error)` | Fetch JSON stats from URL |

### Adapter-specific methods

#### XMRigMiner

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewXMRigMiner` | `func NewXMRigMiner(name string) *XMRigMiner` | Create XMRig miner |
| All `Miner` interface methods | | Inherited from BaseMiner |

#### TTMiner

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewTTMiner` | `func NewTTMiner(name string) *TTMiner` | Create TT-Miner instance |
| All `Miner` interface methods | | Inherited from BaseMiner |

#### SimulatedMiner

| Method | Signature | Description |
|--------|-----------|-------------|
| `NewSimulatedMiner` | `func NewSimulatedMiner(name string) *SimulatedMiner` | Create simulated miner |
| `SetBaseHashrate` | `func (s *SimulatedMiner) SetBaseHashrate(h int)` | Set base hashrate |
| `SetVariance` | `func (s *SimulatedMiner) SetVariance(v float64)` | Set hashrate variance |
| All `Miner` interface methods | | Inherited from BaseMiner |

---

## Test coverage

### Test naming convention

All tests follow the **Good/Bad/Ugly** triplet pattern:
- `TestFilename_Function_Good_Scenario` — Happy path, valid inputs
- `TestFilename_Function_Bad_Scenario` — Error conditions, invalid inputs
- `TestFilename_Function_Ugly_Scenario` — Edge cases, boundary conditions

### Test categories

| Category | Files | Lines | Coverage |
|----------|-------|-------|----------|
| Miner Interface | miner_test.go | ~70K | ✅ High |
| Miner Implementations | miners_impl_test.go | ~58K | ✅ High |
| Manager | manager_test.go | ~24K | ✅ High |
| Factory | factory_test.go | ~5K | ✅ High |
| Profile Management | profile_test.go | ~6K | ✅ High |
| Events | events_test.go | ~2K | ✅ High |
| Metrics | metrics_test.go | ~2K | ✅ High |
| Circuit Breaker | circuit_breaker_test.go | ~680 | ✅ High |
| Supervisor | supervisor_test.go | ~6K | ✅ High |
| Log Buffer | log_buffer_test.go | ~5K | ✅ High |
| Service | service_test.go, service_impl_test.go | ~105K | ✅ High |
| Types | types_test.go | ~46K | ✅ High |
| Utilities | util_impl_test.go | ~41K | ✅ High |
| Installation | miner_install_test.go | ~8K | ✅ High |
| Simulated Lifecycle | simulated_lifecycle_test.go | ~7K | ✅ High |
| Archive | archive_extract_test.go | ~7K | ✅ High |
| Service Helpers | service_*_test.go files | ~15K | ✅ High |
| Triplets | ax7_triplets_test.go | ~265K | ✅ High |
| Paths | base_miner_paths_test.go | ~8K | ✅ High |

**Total Test Coverage: High** — All public APIs have Good/Bad/Ugly tests

### Example test patterns

```go
// Good: Happy path
func TestMiner_Start_Good_Basic(t *testing.T) { ... }

// Bad: Error conditions
func TestMiner_Start_Bad_AlreadyRunning(t *testing.T) { ... }

// Ugly: Edge cases
func TestMiner_Start_Ugly_NilConfig(t *testing.T) { ... }
```

---

## Relationships

### Dependencies

```
┌─────────────────────────────────────────────────────────────────┐
│ go-miner Dependencies                                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                     │
│  CoreGO Framework (dappco.re/go):                                 │
│  ├── core.Result pattern                                          │
│  ├── core.Action IPC                                              │
│  ├── c.Process() — subprocess management                         │
│  ├── c.Fs() — filesystem abstraction                               │
│  └── Various utility functions                                    │
│                                                                     │
│  Internal Dependencies:                                           │
│  ├── go-stream (dappco.re/go/core/stream) — event broadcasting    │
│  ├── go-store (dappco.re/go/store) — profile persistence           │
│  └── go-process (dappco.re/go/core/process) — lifecycle mgmt    │
│                                                                     │
│  External Binaries (downloaded at runtime):                         │
│  ├── XMRig — https://github.com/xmrig/xmrig                       │
│  └── TT-Miner — https://github.com/SChernykh/tt-miner              │
│                                                                     │
│  External APIs:                                                    │
│  └── GitHub API — for version checks                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────┘
```

### Consumers

| Package | Usage |
|---------|-------|
| Lethean Desktop | GUI mining application |
| Fleet Management | Multi-node mining orchestration |
| Custom Applications | Mining integration |

### Related packages

| Package | Relationship |
|---------|--------------|
| `go-process` | Provides subprocess management (`c.Process()`) |
| `go-stream` | Provides event broadcasting infrastructure |
| `go-store` | Provides profile persistence |
| `go-p2p` | Peer-to-peer networking for mining pools |
| `go-pool` | Mining pool backend (complementary) |

---

## Usage patterns

### Common patterns

1. **Single Miner** — Create and start one miner instance
2. **Multi-Miner** — Use Manager for multiple miners
3. **Profile-Based** — Save configs and start from profiles
4. **Event-Driven** — Subscribe to events for real-time monitoring
5. **Stats Collection** — Poll GetStats() for performance metrics
6. **Testing** — Use SimulatedMiner for development
7. **Autostart** — Configure profiles to start on boot

### Anti-patterns

1. **Don't ignore errors** — All methods return errors, handle them
2. **Don't block in event handlers** — Keep event callbacks fast
3. **Don't share Miner instances** — Each miner should have its own instance
4. **Don't call Start() twice** — Returns MinerExistsError
5. **Don't call Stop() on stopped miner** — Returns MinerNotRunningError
6. **Don't bypass Manager** — Use Manager for multi-miner scenarios

---

## Notes

### Supported platforms

| Platform | XMRig | TT-Miner | Notes |
|----------|-------|----------|-------|
| Linux | ✅ Yes | ✅ Yes | Full support |
| macOS | ✅ Yes | ❌ No | TT-Miner requires CUDA (NVIDIA) |
| Windows | ✅ Yes | ✅ Yes | Full support |

### Supported algorithms

**CPU (XMRig):**
- RandomX (rx/0, rx/wow, rx/loki)
- CryptoNight (cn/r, cn/fast, cn/half, cn/2, cn/xao)
- Argon2 (argon2d, argon2id)
- KawPow
- Ethash
- And more...

**GPU (XMRig):**
- All CPU algorithms via OpenCL/CUDA
- GPU-specific optimizations

**GPU (TT-Miner):**
- KawPow
- Ethash
- ProgPoW
- And more...

### XDG directories

```
Linux:   ~/.local/share/lethean-desktop/miners/
macOS:   ~/Library/Application Support/lethean-desktop/miners/
Windows: %APPDATA%\lethean-desktop\miners\
```

### Binary discovery

Miners are discovered in the XDG data directory. If not found, they are downloaded from GitHub releases.

### Version checking

Latest versions are fetched from GitHub API:
- XMRig: https://github.com/xmrig/xmrig/releases
- TT-Miner: https://github.com/SChernykh/tt-miner/releases

### HTTP API ports

- XMRig: Auto-selected (default range)
- TT-Miner: 4068 by default

### Hashrate history retention

- **High-resolution:** 10s intervals, 5 min retention
- **Low-resolution:** 1 min intervals, 24h retention
- **Automatic reduction:** High-res points > 5 min are aggregated to low-res

### Circuit breaker defaults

- Max failures: 3
- Timeout: 30 seconds
- States: Closed → Open → HalfOpen

### Rate limiter defaults

- Token bucket algorithm
- Configurable rate and period
- Thread-safe

### Task supervisor defaults

- Max restarts: 5
- Panic recovery: Yes
- Restart delay: Configurable

### Log buffer defaults

- Capacity: 500 lines
- Timestamp precision: time.Time
- Satisfies: io.Writer interface

---

## Metadata

| Attribute | Value |
|-----------|-------|
| **Package** | go-miner |
| **Module** | dappco.re/go/core/miner |
| **Repository** | core/go-miner |
| **Type** | Library |
| **Status** | Production |
| **Tier** | lib |
| **Created** | 2026-06-18T06:00:00Z |
| **Updated** | 2026-06-18T06:00:00Z |
| **Author** | Purberus <purberus@lthn.ai> |
| **Licence** | EUPL-1.2 |

---

*Package index generated from source code analysis and RFC specification.*
*Last updated: 2026-06-18T06:00:00Z*
