---
type: Package Deep Dive
title: go-miner — Mining Software Controller Library
description: Complete documentation for go-miner — unified interface for controlling mining software (XMRig, TT-Miner, simulated)
description: Wraps mining software behind a unified Miner interface with subprocess lifecycle management, real-time events via go-stream, profile persistence via go-store, circuit breakers, rate limiters, and panic-recovering supervisors
module: dappco.re/go/core/miner
repo: core/go-miner
tags: [mining, xmrig, ttminer, hashrate, profiles, subprocess, cryptocurrency, blockchain, controller, lifecycle, events, circuit-breaker, rate-limiter, supervisor]
lang: go
author: Mistral Vibe
version: 1.0.0
created: 2026-06-18T06:00:00Z
---

# go-miner — Mining Software Controller Library

> **"An agent should be able to implement this library from this document alone."**

`dappco.re/go/core/miner` is the **mining software controller library** that wraps **XMRig**, **TT-Miner**, and **simulated miners** behind a unified `Miner` interface. It extracted from `forge.lthn.ai/Snider/Mining/pkg/mining` and provides subprocess lifecycle management, real-time events through `go-stream`, profile persistence through `go-store`, and robust utilities like circuit breakers and rate limiters.

Used by the Lethean mining desktop and fleet applications to manage CPU and GPU mining across multiple algorithms and pools.

---

## 🎯 Overview

### What it is

- **Unified Miner Interface** — Single `Miner` interface for XMRig, TT-Miner, and simulated miners
- **Multi-Miner Manager** — Concurrent lifecycle coordinator with autostart on boot
- **Subprocess Lifecycle** — Process management via `c.Process()` from CoreGO
- **Real-Time Events** — Event bus broadcasting miner lifecycle and stats via `go-stream`
- **Profile Management** — CRUD operations backed by `go-store` for saved mining configs
- **Hashrate History** — Two-tier retention: 10s high-res (5 min) + 1 min low-res (24h)
- **Robust Utilities** — CircuitBreaker, RateLimiter, TaskSupervisor, LogBuffer
- **HTTP Stats Polling** — Generic `FetchJSONStats[T]` for miner API integration
- **CoreGO Integration** — Full framework integration with `core.Result` and `core.Action`

### Supported Miners

| Miner | Type | Algorithm | Platform | Status |
|-------|------|-----------|----------|--------|
| **XMRig** | CPU/GPU | RandomX, CryptoNight, KawPow | Linux/macOS/Windows | ✅ Production |
| **TT-Miner** | GPU | KawPow, Ethash | NVIDIA GPU, CUDA required | ✅ Production |
| **SimulatedMiner** | Test | Deterministic hashrate | Any | ✅ Production |

**Excluded from v1 (roadmap):**
- T-Rex, lolMiner, Rigel, SRBMiner, TeamRedMiner
- Auto-exchange / wallet tracking
- Pool auto-discovery

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      go-miner PACKAGE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     MINER INTERFACE                           │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │   │
│  │  │   XMRig      │  │  TT-Miner    │  │  Simulated   │   │   │
│  │  │   Miner      │  │   Miner      │  │   Miner      │   │   │
│  │  │ (CPU/GPU)    │  │ (NVIDIA GPU)  │  │ (Test Double)│   │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘   │   │
│  │         │                  │                  │            │   │
│  └─────────┼──────────────────┼──────────────────┼────────┘   │
│            │                  │                  │              │
│            └──────────────────┼──────────────────┘              │
│                               │                                      │
│                    ┌──────────────▼──────────────┐                 │
│                    │       Miner Interface        │                 │
│                    │  Install, Start, Stop,        │                 │
│                    │  GetStats, IsRunning,        │                 │
│                    │  GetHashrateHistory,        │                 │
│                    │  GetLogs, WriteStdin         │                 │
│                    └──────────────┬──────────────┘                 │
│                                     │                               │
│         ┌───────────────────────────┼───────────────────────┐      │
│         │                           │                           │      │
│  ┌─────▼──────┐            ┌─────▼──────┐            ┌─────▼──────┐  │
│  │  Manager    │            │  Factory    │            │  Profile   │  │
│  │             │            │             │            │  Manager  │  │
│  │ Multi-miner│            │ CreateMiner│            │  CRUD     │  │
│  │ lifecycle  │            │ registry   │            │  go-store │  │
│  └─────────────┘            └─────────────┘            └─────────────┘  │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    UTILITIES                                  │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │   │
│  │  │ Circuit      │  │  Rate        │  │  Task        │   │   │
│  │  │ Breaker      │  │  Limiter     │  │  Supervisor  │   │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │   │
│  │  ┌──────────────┐  ┌──────────────┐                       │   │
│  │  │ EventBus     │  │  LogBuffer    │                       │   │
│  │  │ (go-stream)  │  │  (io.Writer)  │                       │   │
│  │  └──────────────┘  └──────────────┘                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────┘

Data Flow:
  Manager → Miner.Install() → Miner.Start() → Miner.GetStats() → EventBus.Broadcast()
  ProfileManager ←→ go-store (persistence)
  Miner → LogBuffer → io.Writer (stdout/stderr capture)
```

### Architecture Layers

```
┌─────────────────────────────────────────────────────────────────┐
│ Layer 3: Application Integration                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Mining Desktop Application                             │   │
│  │ Fleet Management System                               │   │
│  │ Wails GUI Integration                                  │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│ Layer 2: Controller Library (go-miner)                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Miner Interface, Manager, Factory, ProfileManager     │   │
│  │ EventBus, CircuitBreaker, RateLimiter, TaskSupervisor │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│ Layer 1: CoreGO Framework                                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ c.Process(), go-stream, go-store, core.Result         │   │
│  └─────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│ Layer 0: External Dependencies                                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ XMRig binary, TT-Miner binary                           │   │
│  │ GitHub API (for version checks)                         │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📦 Quick Start

### Basic Miner Usage

```go
import miner "dappco.re/go/core/miner"

// Create an XMRig miner instance
minerInstance, err := miner.CreateMiner(miner.MinerTypeXMRig, "xmrig-cpu")
if err != nil {
    log.Fatal(err)
}

// Install the miner binary
err = minerInstance.Install()
if err != nil {
    log.Fatal(err)
}

// Check installation
details, err := minerInstance.CheckInstallation()
if err != nil || !details.IsInstalled {
    log.Fatal("Miner not installed")
}

// Configure mining
config := &miner.Config{
    Pool:   "pool.lthn.io:3333",
    Wallet: "LTHN_WALLET_ADDRESS",
    Algo:   "rx/0", // RandomX for CPU
    Threads: 4,
}

// Start mining
err = minerInstance.Start(config)
if err != nil {
    log.Fatal(err)
}

// Check if running
if minerInstance.IsRunning() {
    log.Println("Miner is running!")
}

// Get real-time stats
ctx := context.Background()
metrics, err := minerInstance.GetStats(ctx)
if err != nil {
    log.Fatal(err)
}

log.Printf("Hashrate: %d H/s", metrics.Hashrate)
log.Printf("Shares: %d", metrics.Shares)

// Stop mining
err = minerInstance.Stop()
if err != nil {
    log.Fatal(err)
}
```

### Using the Manager

```go
import miner "dappco.re/go/core/miner"

// Create a manager
manager := miner.NewManager()

// Add miners
xmrigMiner, _ := miner.CreateMiner(miner.MinerTypeXMRig, "xmrig-cpu")
ttMiner, _ := miner.CreateMiner(miner.MinerTypeTTMiner, "tt-miner-gpu")

manager.AddMiner(xmrigMiner)
manager.AddMiner(ttMiner)

// Install all
for _, m := range manager.GetMiners() {
    _ = m.Install()
}

// Start all with configuration
config := &miner.Config{
    Pool:   "pool.lthn.io:3333",
    Wallet: "LTHN_WALLET",
    Algo:   "kawpow", // GPU algorithm
    GPUEnabled: true,
    CUDA: true,
}

_ = manager.StartAll(config)

// Get aggregated stats from all miners
combinedMetrics := manager.GetCombinedStats()
log.Printf("Total Hashrate: %d H/s", combinedMetrics.Hashrate)

// Stop all
_ = manager.StopAll()
```

### With Profile Management

```go
import miner "dappco.re/go/core/miner"

// Create profile manager
profileManager, err := miner.NewProfileManager()
if err != nil {
    log.Fatal(err)
}

// Create a mining profile
profile := &miner.MiningProfile{
    Name: "LTHN-CPU-Mining",
    Config: &miner.Config{
        Pool:    "pool.lthn.io:3333",
        Wallet:  "LTHN_WALLET",
        Algo:    "rx/0",
        Threads: 4,
        Autostart: true,
    },
    MinerType: miner.MinerTypeXMRig,
}

// Save profile
err = profileManager.SaveProfile(profile)
if err != nil {
    log.Fatal(err)
}

// Load profile
savedProfile, err := profileManager.GetProfile("LTHN-CPU-Mining")
if err != nil {
    log.Fatal(err)
}

// Start from profile
manager := miner.NewManager()
minerInstance, _ := miner.CreateMiner(savedProfile.MinerType, "my-miner")
manager.AddMiner(minerInstance)

// Use profile config
err = minerInstance.Start(savedProfile.Config)
if err != nil {
    log.Fatal(err)
}
```

---

## 🏗️ Architecture

### Core Components

#### Miner Interface

The unified interface that all mining adapters implement:

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

#### Miner Types

```go
const (
    MinerTypeXMRig      = "xmrig"
    MinerTypeTTMiner    = "ttminer"
    MinerTypeSimulated  = "simulated"
)
```

#### BaseMiner

Shared implementation for all miners:

```go
type BaseMiner struct {
    name      string
    minerType string
    path      string
    binaryPath string
    process   *core.Process
    config    *Config
    
    // Hashrate history (two-tier)
    highResHashrate []HashratePoint  // 10s intervals, 5 min retention
    lowResHashrate  []HashratePoint  // 1 min intervals, 24h retention
    
    // Log capture
    logBuffer *LogBuffer
    
    // Lifecycle management
    mu           sync.RWMutex
    isRunning   bool
    startTime   time.Time
}
```

### Adapter Implementations

#### XMRigMiner

CPU and GPU mining with XMRig:

```go
type XMRigMiner struct {
    *BaseMiner
    // XMRig-specific fields
}

// XMRig supports:
// - CPU mining: RandomX, CryptoNight, KawPow, Ethash, etc.
// - GPU mining: OpenCL (AMD/Intel), CUDA (NVIDIA)
// - Auto configuration generation
// - HTTP API for stats (default port: auto-selected)
// - Binary discovery and version checking
// - Config file generation and management

// Example: CPU mining with XMRig
minerInstance, _ := miner.CreateMiner(miner.MinerTypeXMRig, "xmrig-rx")
config := &miner.Config{
    Pool:    "pool.lthn.io:3333",
    Wallet:  "LTHN_WALLET",
    Algo:    "rx/0", // RandomX
    Threads: 4,
    HugePages: true,
}
_ = minerInstance.Start(config)

// Example: GPU mining with XMRig
config := &miner.Config{
    Pool:    "pool.lthn.io:3333",
    Wallet:  "LTHN_WALLET",
    Algo:    "kawpow",
    GPUEnabled: true,
    CUDA: true,
    Devices: "0", // Use first GPU
}
_ = minerInstance.Start(config)
```

#### TTMiner

NVIDIA GPU mining with TT-Miner:

```go
type TTMiner struct {
    *BaseMiner
    // TT-Miner-specific fields
}

// TT-Miner supports:
// - NVIDIA GPU mining (CUDA required)
// - KawPow, Ethash, and other GPU algorithms
// - HTTP API at port :4068 by default
// - Binary discovery and version checking

// Example: TT-Miner with KawPow
minerInstance, _ := miner.CreateMiner(miner.MinerTypeTTMiner, "tt-kawpow")
config := &miner.Config{
    Pool:    "pool.lthn.io:3333",
    Wallet:  "LTHN_WALLET",
    Algo:    "kawpow",
    GPUEnabled: true,
    CUDA: true,
}
_ = minerInstance.Start(config)
```

#### SimulatedMiner

Test double for development and testing:

```go
type SimulatedMiner struct {
    *BaseMiner
    baseHashrate int
    variance     float64
    random       *rand.Rand
}

// SimulatedMiner provides:
// - Deterministic hashrate with configurable variance
// - Realistic hashrate fluctuations
// - No hardware or external dependencies
// - Perfect for unit tests and CI/CD

// Example: Simulated miner for testing
minerInstance, _ := miner.CreateMiner(miner.MinerTypeSimulated, "simulated")
config := &miner.Config{
    Pool:   "pool.test:3333",
    Wallet: "TEST_WALLET",
    Algo:   "rx/0",
}
_ = minerInstance.Start(config)

// Get realistic stats without actual mining
metrics, _ := minerInstance.GetStats(context.Background())
// metrics.Hashrate will be realistic and fluctuate
```

### Manager

Multi-miner lifecycle coordinator:

```go
type Manager struct {
    miners []Miner
    eventBus *EventBus
    runningMiners sync.Map
    mu sync.RWMutex
}

// Manager provides:
// - Concurrent lifecycle management
// - Autostart on boot from saved profiles
// - Per-miner stats collection loop
// - Aggregated stats across all miners
// - Event broadcasting for all miners

// Example: Manager with autostart
manager := miner.NewManager()

// Load profiles and add miners
profiles, _ := profileManager.GetAllProfiles()
for _, profile := range profiles {
    minerInstance, _ := miner.CreateMiner(profile.MinerType, profile.Name)
    manager.AddMiner(minerInstance)
}

// Start all miners with their saved configs
_ = manager.StartAllFromProfiles()

// Collect stats every 10 seconds
go func() {
    ticker := time.NewTicker(10 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        combined := manager.GetCombinedStats()
        log.Printf("Total: %d H/s, Shares: %d", 
            combined.Hashrate, combined.Shares)
    }
}()
```

### ProfileManager

Profile persistence backed by go-store:

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

// ProfileManager provides:
// - CRUD operations for mining profiles
// - Profile validation
// - Start from profile functionality
// - Autostart configuration

// Example: Full profile workflow
profileManager, _ := miner.NewProfileManager()

// Create and save
profile := &miner.MiningProfile{
    Name: "My-LTHN-Config",
    Config: &miner.Config{
        Pool:   "pool.lthn.io:3333",
        Wallet: "LTHN_WALLET",
        Algo:   "rx/0",
        Threads: 8,
        Autostart: true,
    },
    MinerType: miner.MinerTypeXMRig,
}
_ = profileManager.SaveProfile(profile)

// List all profiles
profiles, _ := profileManager.GetAllProfiles()
for _, p := range profiles {
    log.Printf("Profile: %s (Type: %s)", p.Name, p.MinerType)
}

// Update profile
profile.Config.Threads = 12
_ = profileManager.SaveProfile(profile)

// Delete profile
_ = profileManager.DeleteProfile("My-LTHN-Config")
```

---

## 📡 Event System

### EventBus

Real-time event broadcasting via go-stream:

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
    hub *stream.Hub
    subscribers map[EventType][]chan MinerEventData
}

// Example: Subscribe to miner events
manager := miner.NewManager()
eventBus := manager.GetEventBus()

// Subscribe to stats events
statsCh := eventBus.Subscribe(miner.EventMinerStats)
go func() {
    for event := range statsCh {
        data := event.Data.(MinerStatsData)
        log.Printf("[%s] Hashrate: %d H/s", 
            data.MinerName, data.Metrics.Hashrate)
    }
}()

// Subscribe to all events
allCh := eventBus.SubscribeAll()
go func() {
    for event := range allCh {
        log.Printf("Event: %v - %v", event.Type, event.Data)
    }
}()
```

---

## 📊 Configuration

### Config Structure

The complete mining configuration:

```go
type Config struct {
    // Connection
    Pool    string `json:"pool"`         // Stratum endpoint, e.g. "pool.lthn.io:3333"
    Wallet  string `json:"wallet"`       // Mining address
    TLS     bool   `json:"tls,omitempty"` // Enable TLS
    
    // Algorithm
    Algo    string `json:"algo"`         // CPU algorithm: "rx/0", "cn/r", "kawpow"
    Coin    string `json:"coin,omitempty"` // Coin name (takes precedence over Algo)
    
    // CPU Settings
    Threads            int  `json:"threads,omitempty"`     // CPU thread count (0 = auto)
    HugePages          bool `json:"hugePages,omitempty"`   // Enable Linux huge pages
    CPUMaxThreadsHint int  `json:"cpuMaxThreadsHint,omitempty"` // xmrig max-threads-hint
    CPUPriority        int  `json:"cpuPriority,omitempty"`   // xmrig CPU priority (1-5)
    
    // GPU Settings
    GPUEnabled    bool   `json:"gpuEnabled,omitempty"`    // Enable GPU mining
    GPUPool       string `json:"gpuPool,omitempty"`       // Separate pool for GPU
    GPUWallet     string `json:"gpuWallet,omitempty"`     // Override wallet for GPU pool
    GPUPassword   string `json:"gpuPassword,omitempty"`   // GPU pool password
    GPUAlgo       string `json:"gpuAlgo,omitempty"`       // GPU algorithm
    OpenCL        bool   `json:"opencl,omitempty"`         // Enable AMD/Intel GPU (OpenCL)
    CUDA          bool   `json:"cuda,omitempty"`           // Enable NVIDIA GPU (CUDA)
    Devices       string `json:"devices,omitempty"`        // GPU device indices ("0", "0,1")
    GPUIntensity  int    `json:"gpuIntensity,omitempty"`   // Work intensity (0 = auto)
    GPUThreads    int    `json:"gpuThreads,omitempty"`     // GPU thread count (0 = auto)
    
    // HTTP API
    HTTPPort int `json:"httpPort,omitempty"` // Override API port (1024-65535 or 0 for auto)
    
    // Logging
    LogOutput bool `json:"logOutput,omitempty"` // Mirror miner output to stdout
    
    // Lifecycle
    Autostart       bool `json:"autostart,omitempty"`        // Start on manager init
    PauseOnActive   bool `json:"pauseOnActive,omitempty"`    // Pause when display active
    PauseOnBattery  bool `json:"pauseOnBattery,omitempty"`   // Pause when on battery
    
    // Metadata
    MinerType string `json:"minerType,omitempty"` // Adapter type when stored
}
```

### Example Configurations

#### XMRig CPU Mining (RandomX)

```go
config := &miner.Config{
    Pool:    "pool.lthn.io:3333",
    Wallet:  "LTHN_WALLET_ADDRESS",
    Algo:    "rx/0",
    Threads: 8,
    HugePages: true,
    TLS:      true,
    LogOutput: true,
    Autostart: true,
}
```

#### XMRig GPU Mining (KawPow with CUDA)

```go
config := &miner.Config{
    Pool:    "pool.lthn.io:3333",
    Wallet:  "LTHN_WALLET_ADDRESS",
    Algo:    "kawpow",
    GPUEnabled: true,
    CUDA:      true,
    Devices:   "0",
    GPUIntensity: 2,
    GPUThreads:    2,
}
```

#### TT-Miner GPU Mining

```go
config := &miner.Config{
    Pool:    "pool.lthn.io:3333",
    Wallet:  "LTHN_WALLET_ADDRESS",
    Algo:    "kawpow",
    GPUEnabled: true,
    CUDA:      true,
    MinerType: miner.MinerTypeTTMiner,
}
```

#### Simulated Miner for Testing

```go
config := &miner.Config{
    Pool:    "test.pool:3333",
    Wallet:  "TEST_ADDRESS",
    Algo:    "rx/0",
    MinerType: miner.MinerTypeSimulated,
}
```

---

## 📈 Metrics & Monitoring

### PerformanceMetrics

```go
type PerformanceMetrics struct {
    Hashrate      int                    `json:"hashrate"`        // Current hashrate in H/s
    Shares        int                    `json:"shares"`          // Total shares submitted
    Rejected      int                    `json:"rejected"`        // Rejected shares
    Uptime        int                    `json:"uptime"`          // Uptime in seconds
    LastShare     int64                  `json:"lastShare,omitempty"` // Unix timestamp of last share
    Algorithm     string                 `json:"algorithm,omitempty"` // Current algorithm
    AvgDifficulty int                    `json:"avgDifficulty,omitempty"` // Average difficulty
    DiffCurrent   int                    `json:"diffCurrent,omitempty"` // Current difficulty
    ExtraData     map[string]interface{} `json:"extraData,omitempty"` // Miner-specific data
}
```

### Hashrate History

Two-tier hashrate tracking:

```go
type HashratePoint struct {
    Timestamp time.Time `json:"timestamp"`
    Hashrate  int       `json:"hashrate"`
}

// High-resolution: 10s intervals, 5 min retention
// Low-resolution: 1 min intervals, 24h retention

// Get current hashrate history
points := minerInstance.GetHashrateHistory()

// Add a new point (called automatically during GetStats)
minerInstance.AddHashratePoint(HashratePoint{
    Timestamp: time.Now(),
    Hashrate:  metrics.Hashrate,
})

// Reduce high-res to low-res (called automatically)
minerInstance.ReduceHashrateHistory(time.Now())
```

### Combined Stats

```go
// Manager provides aggregated stats
type CombinedMetrics struct {
    TotalHashrate  int
    TotalShares    int
    TotalRejected  int
    MinerCount     int
    RunningCount   int
}

combined := manager.GetCombinedStats()
log.Printf("Total: %d H/s from %d miners", 
    combined.TotalHashrate, combined.RunningCount)
```

---

## 🛡️ Robust Utilities

### CircuitBreaker

Protects against cascading failures:

```go
type CircuitBreaker struct {
    state        CircuitState  // Closed, Open, HalfOpen
    failureCount int
    successCount int
    lastFailure  time.Time
    lastSuccess  time.Time
    timeout      time.Duration
    maxFailures  int
    mu           sync.RWMutex
}

type CircuitState int
const (
    CircuitClosed CircuitState = iota
    CircuitOpen
    CircuitHalfOpen
)

// Example: Protect HTTP version checks
breaker := miner.NewCircuitBreaker(3, 30*time.Second)

func checkVersion() error {
    result := breaker.Execute(func() error {
        return fetchLatestVersionFromGitHub()
    })
    return result
}
```

### RateLimiter

Token bucket rate limiting per IP:

```go
type RateLimiter struct {
    tokens     float64
    maxTokens  float64
    refillRate float64
    lastRefill time.Time
    mu         sync.Mutex
}

// Example: Rate limit API calls
limiter := miner.NewRateLimiter(10, 1*time.Second) // 10 requests per second

func makeAPICall() {
    if !limiter.Allow() {
        log.Println("Rate limited, try again later")
        return
    }
    // Make API call
}
```

### TaskSupervisor

Panic-recovering goroutine restarter:

```go
type TaskSupervisor struct {
    task     func()
    onPanic  func(recovered any)
    maxRestarts int
    restartCount int
    stop     chan struct{}
    done     chan struct{}
    mu       sync.Mutex
}

// Example: Supervise a background task
supervisor := miner.NewTaskSupervisor(
    func() {
        // Task that might panic
        collectMinerStats()
    },
    func(recovered any) {
        log.Printf("Panic recovered: %v", recovered)
    },
    5, // Max restarts before giving up
)

// Start the supervised task
supervisor.Start()

// Stop the supervisor
supervisor.Stop()
```

### LogBuffer

Timestamped ring buffer for miner output:

```go
type LogBuffer struct {
    lines     []LogLine
    capacity  int
    head      int
    count     int
    mu        sync.Mutex
}

type LogLine struct {
    Timestamp time.Time
    Line      string
}

// LogBuffer satisfies io.Writer
var _ io.Writer = (*LogBuffer)(nil)

// Example: Capture miner output
logBuffer := miner.NewLogBuffer(500) // 500 line capacity

// Write to buffer (automatically timestamped)
logBuffer.Write([]byte("Miner started\n"))

// Get all lines
lines := logBuffer.GetLines()
for _, line := range lines {
    log.Printf("[%s] %s", line.Timestamp, line.Line)
}

// Get as strings
strings := logBuffer.GetStrings()
```

---

## 🔌 CoreGO Integration

### Service Integration

```go
// go-miner integrates with CoreGO service framework

// Register actions
err := miner.RegisterActions()
if err != nil {
    log.Fatal(err)
}

// Actions registered:
// - miner.Install
// - miner.Uninstall
// - miner.Start
// - miner.Stop
// - miner.GetStats
// - miner.GetConfig
// - miner.SetConfig
// - miner.ListProfiles
// - miner.GetProfile
// - miner.SaveProfile
// - miner.DeleteProfile
```

### With CoreGO Framework

```go
import (
    "dappco.re/go"
    miner "dappco.re/go/core/miner"
)

// Create a CoreGO service with mining support
service := core.New(
    core.WithServiceLock(),
    core.WithActionHandlers(miner.GetActionHandlers()),
)

// Start the service
result := service.Start()
if !result.OK {
    log.Fatal(result.Error())
}

// Use CoreGO primitives
fs := core.Fs{}
processManager := core.Process{}

// All I/O goes through CoreGO
files, result := fs.List("/path/to/miners")
if result.OK {
    // Process files
}
```

---

## 📦 Installation & Distribution

### Binary Management

```go
// Install miner binary
minerInstance, _ := miner.CreateMiner(miner.MinerTypeXMRig, "xmrig")
err := minerInstance.Install()
if err != nil {
    log.Fatal(err)
}

// Check installation
details, err := minerInstance.CheckInstallation()
if err != nil {
    log.Fatal(err)
}

if details.IsInstalled {
    log.Printf("Version: %s", details.Version)
    log.Printf("Path: %s", details.Path)
    log.Printf("Binary: %s", details.MinerBinary)
}

// Get latest version from GitHub
version, err := minerInstance.GetLatestVersion()
if err != nil {
    log.Printf("Could not fetch version: %v", err)
} else {
    log.Printf("Latest version: %s", version)
}

// Uninstall
minerInstance, _ = miner.CreateMiner(miner.MinerTypeXMRig, "xmrig")
err = minerInstance.Uninstall()
if err != nil {
    log.Fatal(err)
}
```

### Binary Discovery

XMRig and TT-Miner binaries are discovered in XDG data directories:

```
Linux:   ~/.local/share/lethean-desktop/miners/
macOS:   ~/Library/Application Support/lethean-desktop/miners/
Windows: %APPDATA%\lethean-desktop\miners\
```

---

## 🏆 Features Summary

### ✅ What go-miner Provides

1. **Unified Interface** — Single `Miner` interface for all supported miners
2. **Multi-Miner Support** — XMRig (CPU/GPU), TT-Miner (NVIDIA GPU), Simulated
3. **Lifecycle Management** — Install, Start, Stop, Uninstall with proper error handling
4. **Real-Time Events** — Event bus with go-stream integration
5. **Profile Management** — CRUD with go-store persistence
6. **Hashrate Tracking** — Two-tier history with automatic reduction
7. **Robust Utilities** — CircuitBreaker, RateLimiter, TaskSupervisor, LogBuffer
8. **HTTP Stats** — Generic stats fetching with type safety
9. **CoreGO Integration** — Full framework integration
10. **Cross-Platform** — Linux, macOS, Windows support
11. **Algorithm Support** — RandomX, CryptoNight, KawPow, Ethash, and more
12. **GPU Support** — NVIDIA CUDA, AMD/Intel OpenCL

### ❌ What go-miner Does NOT Do

1. **No Wails Integration** — Desktop GUI is separate concern (`core/gui`)
2. **No Wallet Management** — Only mining control, no wallet tracking
3. **No Auto-Exchange** — Mining rewards are not automatically exchanged
4. **No Pool Discovery** — Pools must be configured manually
5. **No Additional Miners** — T-Rex, lolMiner, Rigel, SRBMiner, TeamRedMiner are excluded from v1

---

## 📚 Testing

### Test Patterns

All tests follow the **Good/Bad/Ugly** triplet pattern:
- `TestFilename_Function_Good_*` — Happy path, valid inputs
- `TestFilename_Function_Bad_*` — Error conditions, invalid inputs
- `TestFilename_Function_Ugly_*` — Edge cases, boundary conditions

### Running Tests

```bash
# Run all tests
go test ./... -count=1

# Run with race detector (must pass before commit)
go test -race ./...

# Run with coverage
go test -cover ./...

# Run specific test
go test -v -run TestMiner_Start_Good ./...

# Run all Good tests
go test -v -run Test.*_Good ./...
```

### Test File Structure

```
miner_test.go              # Miner interface tests
miners_impl_test.go        # XMRig, TT-Miner, Simulated implementation tests
manager_test.go            # Manager lifecycle tests
factory_test.go            # MinerFactory tests
profile_test.go            # ProfileManager tests
events_test.go             # EventBus tests
metrics_test.go            # Metrics and hashrate history tests
circuit_breaker_test.go    # CircuitBreaker tests
rate_limiter_test.go       # RateLimiter tests
supervisor_test.go         # TaskSupervisor tests
log_buffer_test.go         # LogBuffer tests
service_test.go            # Service integration tests
types_test.go              # Type validation tests
util_impl_test.go          # Utility function tests
```

### Example Tests

```
miner_example_test.go      # Basic miner usage examples
miners_impl_example_test.go # Adapter-specific examples
manager_service_impl_example_test.go # Manager service examples
service_impl_example_test.go # Service integration examples
```

---

## 🎨 Code Examples

### Example: Complete Mining Application

```go
package main

import (
    "context"
    "log"
    "time"
    
    miner "dappco.re/go/core/miner"
)

func main() {
    // Create profile manager
    profileManager, err := miner.NewProfileManager()
    if err != nil {
        log.Fatal(err)
    }
    
    // Create and save a profile
    profile := &miner.MiningProfile{
        Name: "LTHN-Mining",
        Config: &miner.Config{
            Pool:    "pool.lthn.io:3333",
            Wallet:  "LTHN_WALLET",
            Algo:    "rx/0",
            Threads: 4,
            Autostart: true,
        },
        MinerType: miner.MinerTypeXMRig,
    }
    
    if err := profileManager.SaveProfile(profile); err != nil {
        log.Fatal(err)
    }
    
    // Create manager and miner
    manager := miner.NewManager()
    minerInstance, err := miner.CreateMiner(miner.MinerTypeXMRig, "main-miner")
    if err != nil {
        log.Fatal(err)
    }
    
    // Install miner
    if err := minerInstance.Install(); err != nil {
        log.Printf("Install failed: %v", err)
    }
    
    // Add to manager and start
    manager.AddMiner(minerInstance)
    if err := minerInstance.Start(profile.Config); err != nil {
        log.Fatal(err)
    }
    
    // Setup event bus
    eventBus := manager.GetEventBus()
    statsCh := eventBus.Subscribe(miner.EventMinerStats)
    
    go func() {
        for event := range statsCh {
            data := event.Data.(miner.MinerStatsData)
            log.Printf("[%s] Hashrate: %d H/s, Shares: %d", 
                data.MinerName, data.Metrics.Hashrate, data.Metrics.Shares)
        }
    }()
    
    // Monitor stats
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        combined := manager.GetCombinedStats()
        log.Printf("Total Hashrate: %d H/s", combined.TotalHashrate)
        
        // Get individual miner stats
        if metrics, err := minerInstance.GetStats(context.Background()); err == nil {
            log.Printf("Miner Uptime: %ds", metrics.Uptime)
        }
    }
}
```

### Example: GPU Mining with Multiple Algorithms

```go
// Setup GPU miners for different algorithms
xmrigGPU, _ := miner.CreateMiner(miner.MinerTypeXMRig, "xmrig-gpu")
ttMiner, _ := miner.CreateMiner(miner.MinerTypeTTMiner, "tt-miner")

// Install both
_ = xmrigGPU.Install()
_ = ttMiner.Install()

// Configure for different pools/algorithms
xmrigConfig := &miner.Config{
    Pool:    "kawpow.pool.io:3333",
    Wallet:  "WALLET",
    Algo:    "kawpow",
    GPUEnabled: true,
    CUDA: true,
    Devices: "0,1",
    MinerType: miner.MinerTypeXMRig,
}

ttConfig := &miner.Config{
    Pool:    "ethash.pool.io:3333",
    Wallet:  "WALLET",
    Algo:    "ethash",
    GPUEnabled: true,
    CUDA: true,
    Devices: "0,1",
    MinerType: miner.MinerTypeTTMiner,
}

// Start both
_ = xmrigGPU.Start(xmrigConfig)
_ = ttMiner.Start(ttConfig)

// Monitor combined performance
manager := miner.NewManager()
manager.AddMiner(xmrigGPU)
manager.AddMiner(ttMiner)

combined := manager.GetCombinedStats()
log.Printf("Total GPU Hashrate: %d H/s", combined.TotalHashrate)
```

### Example: Testing with Simulated Miner

```go
// Create simulated miner for testing
simMiner, _ := miner.CreateMiner(miner.MinerTypeSimulated, "test-miner")

// Configure
config := &miner.Config{
    Pool:    "test.pool:3333",
    Wallet:  "TEST_WALLET",
    Algo:    "rx/0",
    MinerType: miner.MinerTypeSimulated,
}

// Start
_ = simMiner.Start(config)

// Get stats - will have realistic simulated hashrate
metrics, _ := simMiner.GetStats(context.Background())
if metrics.Hashrate > 0 {
    log.Printf("Simulated hashrate: %d H/s", metrics.Hashrate)
}

// Stop
_ = simMiner.Stop()

// Clean up
_ = simMiner.Uninstall()
```

---

## 📖 References

### RFC Specification

- **[RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/miner/RFC.md)** — Complete technical specification

### Related Documentation

- **[CLAUDE.md](file:///Users/snider/Code/core/go-miner/CLAUDE.md)** — Implementation details, file map, conventions
- **[AGENTS.md](file:///Users/snider/Code/core/go-miner/AGENTS.md)** — Agent guidance
- **[CoreGO RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)** — Framework specification
- **[go-process RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/process/RFC.md)** — Process management specification
- **[go-stream RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/stream/RFC.md)** — Event streaming specification
- **[go-store RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/store/RFC.md)** — Storage specification

### Source Code

- **Repository:** `github.com/dAppCore/go-miner` / `dappco.re/go/core/miner`
- **Module:** `dappco.re/go/core/miner`
- **Package:** `dappco.re/go/core/miner`
- **Go Version:** 1.21+

### Dependencies

```
dappco.re/go                    # CoreGO framework (core primitives)
dappco.re/go/core/process      # Process management
dappco.re/go/core/stream       # Event streaming
dappco.re/go/store              # Profile persistence
```

### External Dependencies

None! The package only interfaces with external miner binaries (XMRig, TT-Miner) via subprocess and HTTP API. The binaries themselves are downloaded at runtime.

---

## 🏷️ Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/core/miner` |
| **Repository** | `core/go-miner` |
| **Type** | Library |
| **Status** | Production |
| **Tier** | lib (foundation) |
| **Lines** | ~50,000+ (source + tests + examples) |
| **Source Files** | 18+ |
| **Test Files** | 20+ |
| **Example Files** | 10+ |
| **Test Coverage** | High |
| **Licence** | EUPL-1.2 |
| **Language** | Go 1.21+ |
| **Author** | Purberus <purberus@lthn.ai> |
| **Created** | 2026-06-18T06:00:00Z |
| **Version** | 1.0.0 |

---

*Documentation generated from source code analysis and RFC specification.*
*Last updated: 2026-06-18T06:00:00Z*
