---
type: Package Documentation
package: proxy
module: dappco.re/go/core/proxy
repo: core/go-proxy
lang: go
tags:
  - stratum
  - mining
  - proxy
  - nicehash
  - tcp
  - tls
  - websocket
---
# go-proxy — Stratum mining proxy

**RFC:** [plans/code/core/go/proxy/RFC.md](../../../../../plans/code/core/go/proxy/RFC.md)
**Source:** [~/Code/core/go-proxy/](file:///Users/snider/Code/core/go-proxy/)
**Module:** `dappco.re/go/core/proxy`
**Dependencies:** `dappco.re/core/go`, `dappco.re/core/go/api`

---

## Overview

`go-proxy` is a CryptoNote Stratum protocol proxy library that accepts miner connections over TCP (optionally TLS), and routes mining work to upstream pool connections. It supports two primary operating modes:

1. **NiceHash mode** — Splits the 32-bit nonce space across up to 256 simultaneous miners per upstream pool connection, maximising pool connection reuse
2. **Simple (passthrough) mode** — One upstream connection per miner group, with optional connection reuse on disconnect

### Primary use cases

- **Mining pool operator** — Deploy as a public-facing proxy that connects to backend pools
- **Mining farm** — Centralise management of multiple miners with load balancing
- **NiceHash seller** — Maximise hashrate rental profitability with nonce-splitting
- **Private mining** — Route miners through a secure, monitored gateway

### Design philosophy

- **Zero-downtime hot-reload** — Configuration changes without restarting
- **Connection reuse** — Minimise pool reconnection latency (NiceHash mode: 256 miners/pool)
- **Protocol extensions** — Support for algorithm negotiation, RigID, custom difficulty
- **Security-first** — TLS for both inbound (miners) and outbound (pools), rate limiting, access control
- **Observability** — Monitoring API, access logs, share logs
- **Resilience** — Primary/failover pool strategy with automatic reconnection

---

## Architecture

### Component stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Monitoring Layer                           │
│  HTTP API (/1/summary, /1/workers, /1/miners)               │
│  Access log, Share log (append-only line files)                │
├─────────────────────────────────────────────────────────────┤
│                    Proxy Orchestrator                         │
│  Proxy — tick loop, listeners, splitter, stats, workers, bus  │
├─────────────────────────────────────────────────────────────┤
│                    Splitter Layer                              │
│  NonceSplitter (NiceHash) — 256-slot table, NonceMapper pool   │
│  SimpleSplitter — Passthrough, upstream reuse pool             │
├─────────────────────────────────────────────────────────────┤
│                    Pool Layer                                 │
│  StratumClient — Outbound pool TCP/TLS connection             │
│  FailoverStrategy — Primary + ordered fallbacks               │
├─────────────────────────────────────────────────────────────┤
│                    Miner Layer                                │
│  Miner — State machine (WaitLogin → WaitReady → Ready)        │
│  Server — TCP listener, rate limiting, connection acceptance  │
└─────────────────────────────────────────────────────────────┘
```

### Import graph (no circular dependencies)

```
proxy (root)        — Shared types: Job, PoolConfig, Config, Miner, UpstreamStats, events
  ├── pool/         — StratumClient, FailoverStrategy (imports proxy)
  ├── nicehash/     — NonceSplitter, NonceMapper, NonceStorage (imports proxy, pool)
  ├── simple/       — SimpleSplitter, SimpleMapper (imports proxy, pool)
  ├── log/          — AccessLog, ShareLog (imports proxy)
  └── api/          — HTTP handlers (imports proxy, core/api)
```

**Key insight:** `proxy.go` uses interface injection (`Splitter`, `ShareSink`) to avoid direct imports of sub-packages, maintaining clean separation.

---

## Data flow

### Inbound path (Miner → Proxy)

```
Miner (TCP) → Server.accept()
                 → Rate limit check (per-IP token bucket)
                 → Miner.handleLogin()
                      → Events.Dispatch(LoginEvent)
                           → CustomDiff.Apply(miner)     (sets miner.customDiff)
                           → Workers.OnLogin(event)      (upsert worker record)
                           → Splitter.OnLogin(event)     (assigns mapper slot)
                                → NonceMapper.Add(miner)
                                     → NonceStorage.Add(miner) → miner.FixedByte = slot
                                → if mapper active: miner.ForwardJob(currentJob)
```

### Outbound path (Pool → Proxy → Miners)

```
Pool (TCP) → pool.StratumClient read loop → OnJob(job)
               → NonceMapper.onJob(job)
                    → NonceStorage.SetJob(job)
                         → for each active slot: miner.ForwardJob(job)
                              → Miner sends JSON over TCP
```

### Submit path (Miner → Proxy → Pool)

```
Miner submit → Miner.handleSubmit()
                 → Events.Dispatch(SubmitEvent)
                      → Splitter.OnSubmit(event)
                           → NonceMapper.Submit(event)
                                → SubmitContext stored by sequence
                                → pool.StratumClient.Submit(jobID, nonce, result, algo)
                                     → Pool reply → OnResultAccepted(seq, ok, err)
                                          → Events.Dispatch(AcceptEvent | RejectEvent)
                                               → Workers.OnAccept / OnReject
                                               → Stats.OnAccept / OnReject
                                               → ShareLog.OnAccept / OnReject
                                               → Miner.Success or Miner.ReplyWithError
```

---

## Core components

### 1. Config

Central configuration loaded from JSON with hot-reload support:

```go
// Load configuration
cfg, result := proxy.LoadConfig("config.json")
if !result.OK { log.Fatal(result.Error) }

// Validate required fields
if result := cfg.Validate(); !result.OK {
    log.Fatal(result.Error)
}
```

**Configuration structure:**

```go
type Config struct {
    Mode             string       `json:"mode"`              // "nicehash" or "simple"
    Bind             []BindAddr   `json:"bind"`              // Listen addresses
    Pools            []PoolConfig `json:"pools"`             // Primary + fallbacks
    TLS              TLSConfig    `json:"tls"`               // Inbound TLS
    HTTP             HTTPConfig   `json:"http"`              // Monitoring API
    AccessPassword   string       `json:"access-password"`  // Login password
    CustomDiff       uint64       `json:"custom-diff"`      // Global diff override
    CustomDiffStats  bool         `json:"custom-diff-stats"`// Per-bucket stats
    AlgoExtension    bool         `json:"algo-ext"`          // Algo field in jobs
    Workers          WorkersMode  `json:"workers"`           // "rig-id", "user", "ip", etc.
    AccessLogFile    string       `json:"access-log-file"`  // Connection log
    ReuseTimeout     int          `json:"reuse-timeout"`    // Simple mode: seconds
    Retries          int          `json:"retries"`          // Pool reconnect attempts
    RetryPause       int          `json:"retry-pause"`      // Seconds between retries
    Watch            bool         `json:"watch"`            // Hot-reload on change
    RateLimit        RateLimit    `json:"rate-limit"`       // Per-IP limits
}
```

### 2. Proxy orchestrator

Top-level component that wires all subsystems:

```go
// Create proxy from config
p, result := proxy.New(cfg)
if !result.OK {
    log.Fatal(result.Error)
}

// Start all components (TCP listeners, pool connections, tick loop, HTTP API)
p.Start()

// Stop cleanly (waits up to 5 seconds for in-flight submits)
p.Stop()

// Hot-reload configuration
p.Reload(newCfg)
```

**Proxy structure:**
```go
type Proxy struct {
    config   *Config
    splitter Splitter
    stats    *Stats
    workers  *Workers
    events   *EventBus
    servers  []*Server
    ticker   *time.Ticker
    watcher  *ConfigWatcher
    done     chan struct{}
}
```

### 3. Server (TCP listener)

Accepts miner connections with rate limiting:

```go
// Create server
srv, result := proxy.NewServer(
    proxy.BindAddr{Host: "0.0.0.0", Port: 3333, TLS: false},
    nil,  // TLS config (nil for plain TCP)
    proxy.NewRateLimiter(proxy.RateLimit{MaxConnectionsPerMinute: 30}),
    func(conn net.Conn, port uint16) {
        // Called on each accepted connection
        m := proxy.NewMiner(conn, port, nil)
        m.Start()
    },
)
if result.OK {
    srv.Start()
}
```

### 4. Miner state machine

Each TCP connection = one `Miner` with linear state transitions:

```
WaitLogin → WaitReady → Ready → Closing
```

**States:**
- `MinerStateWaitLogin` — Connection open, awaiting login request (10s timeout)
- `MinerStateWaitReady` — Login validated, awaiting upstream job (600s timeout)
- `MinerStateReady` — Receiving jobs, accepting submits (600s inactivity timeout)
- `MinerStateClosing` — TCP close in progress

**Miner structure:**
```go
type Miner struct {
    id         int64      // Internal ID
    rpcID      string     // UUID v4 (session ID)
    state      MinerState
    extAlgo    bool       // Algorithm extension active
    extNH      bool       // NiceHash mode active
    ip         string     // Remote IP
    localPort  uint16
    user       string     // Wallet address (from login)
    password   string     // Password (from login)
    agent      string     // Agent string (from login)
    rigID      string     // Rig ID (from login)
    fixedByte  uint8      // NiceHash slot index (0-255)
    mapperID   int64      // Which NonceMapper owns this miner
    routeID    int64      // SimpleMapper ID
    customDiff uint64     // Custom difficulty (0 = use pool diff)
    diff       uint64     // Last difficulty from pool
    rx, tx     uint64     // Bytes received/sent
    connectedAt time.Time
    lastActivityAt time.Time
    conn       net.Conn
    tlsConn    *tls.Conn  // nil if plain TCP
}
```

### 5. Workers (aggregate stats)

Per-worker statistics with rolling hashrate windows:

```go
// Create workers tracker
workers := proxy.NewWorkers(proxy.WorkersByRigID, events)

// On login, worker record is created/updated
workers.OnLogin(proxy.Event{Miner: miner})

// Get all worker records
records := workers.List()
```

**WorkerRecord:**
```go
type WorkerRecord struct {
    Name        string
    LastIP      string
    Connections uint64
    Accepted    uint64
    Rejected    uint64
    Invalid     uint64
    Hashes      uint64  // Sum of accepted share difficulties
    LastHashAt  time.Time
    windows     [5]tickWindow  // 60s, 600s, 3600s, 12h, 24h rolling hashrate
}
```

**Hashrate calculation:**
```go
record.Hashrate(60)   // H/s over last 60 seconds
record.Hashrate(3600) // H/s over last hour
```

### 6. Stats (global aggregates)

Global counters and hashrate windows:

```go
stats := proxy.NewStats(events)

// Access counters
stats.Accepted()    // Total accepted shares
stats.Rejected()    // Total rejected shares
stats.Invalid()     // Total invalid shares
stats.Hashes()      // Total hashes (sum of difficulties)
stats.Hashrate(60)  // Global H/s over last 60 seconds
```

### 7. Event bus

Publish-subscribe for mining events:

```go
// Create event bus
events := proxy.NewEventBus()

// Subscribe to events
sub := events.Subscribe(func(event proxy.Event) {
    switch e := event.(type) {
    case *proxy.LoginEvent:
        fmt.Printf("Miner logged in: %s\n", e.Miner.User)
    case *proxy.AcceptEvent:
        fmt.Printf("Share accepted: %d H/s\n", e.Difficulty)
    case *proxy.RejectEvent:
        fmt.Printf("Share rejected: %s\n", e.Reason)
    }
})

// Unsubscribe
sub.Unsubscribe()
```

**Event types:**
- `LoginEvent` — Miner successfully logged in
- `AcceptEvent` — Share accepted by pool
- `RejectEvent` — Share rejected by pool
- `SubmitEvent` — Miner submitted a share
- `CloseEvent` — Miner disconnected

---

## Splitter implementations

### NiceHash splitter (nonce-splitting mode)

Partitions 32-bit nonce space by fixing one byte (byte 39 of blob). Each `NonceMapper` manages one pool connection with 256 miner slots.

```go
// Create NiceHash splitter
splitter := nicehash.NewNonceSplitter(cfg, events, strategyFactory)
splitter.Connect()

// How it works:
// - OnLogin: Assign miner to first NonceMapper with free slot
// - If all mappers full: Create new NonceMapper (new pool connection)
// - OnSubmit: Route to miner's NonceMapper
// - Each NonceMapper owns 256 slots (0-255)
// - Each slot fixes byte 39 of blob to slot index
```

**NonceStorage (256-slot table):**
```go
type NonceStorage struct {
    slots   [256]int64  // 0 = free, +minerID = active, -minerID = dead
    miners  map[int64]*proxy.Miner
    job     proxy.Job
    prevJob proxy.Job
    cursor  int  // Round-robin allocation starting point
}
```

**NonceMapper:**
- Manages one pool connection (`pool.Strategy`)
- Holds `NonceStorage` for 256 miners
- Implements `pool.StratumListener` for job/result events
- Tracks in-flight submissions via `SubmitContext`

### Simple splitter (passthrough mode)

One pool connection per miner, with optional reuse:

```go
// Create simple splitter
splitter := simple.NewSimpleSplitter(cfg, events, strategyFactory)
splitter.Connect()

// With reuse (default ReuseTimeout = 0 = disabled)
// When miner disconnects, connection held for ReuseTimeout seconds
// Next miner can inherit the idle connection
```

**SimpleMapper:**
- Manages one pool connection
- Serves one active miner at a time
- Becomes idle on miner disconnect
- Reclaimed or stopped based on `ReuseTimeout`

---

## Pool layer

### StratumClient

Outbound pool connection with automatic reconnection:

```go
// Create client
client := pool.NewStratumClient(poolCfg, listener)

// Connect to pool (with optional TLS)
result := client.Connect()
if result.OK {
    client.Login()  // Send login request
}

// Submit a share
seq := client.Submit(jobID, nonce, result, algo)

// Disconnect
client.Disconnect()
```

**Features:**
- Automatic TLS with certificate pinning
- JSON-RPC protocol implementation
- Sequence number tracking for result correlation
- Keepalive support

### FailoverStrategy

Primary + ordered fallback pools with automatic reconnection:

```go
// Create strategy factory
factory := pool.NewStrategyFactory(cfg.Pools)

// Strategy handles connection lifecycle
strategy := factory.NewStrategy()

// Connect to primary (or first available) pool
strategy.Connect()
```

**Reconnection logic:**
1. Try primary pool
2. On failure: Try next pool in order
3. Wait `RetryPause` seconds between attempts
4. Up to `Retries` attempts per pool
5. Rotates through pool list on persistent failures

---

## Logging layer

### Access log

Connection-level logging (open/close):

```go
type AccessLog struct {
    file *os.File
}

// Log connection open
log.LogOpen(miner)

// Log connection close
log.LogClose(miner, reason)
```

**Format:** JSON lines with timestamp, event type, IP, user, duration, bytes

### Share log

Share-level logging (accept/reject):

```go
type ShareLog struct {
    file *os.File
}

// Log accepted share
log.LogAccept(event)

// Log rejected share
log.LogReject(event)
```

**Format:** JSON lines with timestamp, miner, job, difficulty, result

---

## HTTP monitoring API

Lightweight HTTP API for monitoring and management:

```go
// Enable HTTP API in config
cfg.HTTP.Enabled = true
cfg.HTTP.Host = "127.0.0.1"
cfg.HTTP.Port = 8080
cfg.HTTP.AccessToken = "secret-token"  // Optional Bearer auth
cfg.HTTP.Restricted = true  // Read-only GET only

// Access endpoints (with auth if configured):
// GET /1/summary    — Global stats
// GET /1/workers    — Per-worker stats
// GET /1/miners     — Active miner connections
```

**Response format:** JSON

**Example /1/summary:**
```json
{
  "uptime": 3600,
  "connected": 42,
  "accepted": 1523,
  "rejected": 12,
  "hashrate_60s": 1523000,
  "hashrate_600s": 1489000,
  "hashrate_3600s": 1512000
}
```

---

## Security features

### TLS support

**Inbound (miner-facing):**
```go
cfg.TLS.Enabled = true
cfg.TLS.CertFile = "/etc/proxy/cert.pem"
cfg.TLS.KeyFile = "/etc/proxy/key.pem"
```

**Outbound (pool-facing):**
```go
poolCfg.TLS = true
poolCfg.TLSFingerprint = "abc123..."  // SHA-256 of pool cert
```

### Rate limiting

Per-IP connection rate limiting with token bucket:

```go
cfg.RateLimit.MaxConnectionsPerMinute = 30
cfg.RateLimit.BanDurationSeconds = 300  // 5 minute ban on exceed
```

### Access control

**Login password:**
```go
cfg.AccessPassword = "secret123"
// Miners must include in login: {"login": "WALLET", "pass": "secret123", ...}
```

**HTTP API authentication:**
```go
cfg.HTTP.AccessToken = "bearer-token"
// Requests must include: Authorization: Bearer bearer-token
```

**Restricted mode:**
```go
cfg.HTTP.Restricted = true  // Only GET requests allowed
```

---

## Statistics and monitoring

### Hashrate windows

Rolling hashrate calculated over multiple time windows:
- **60 seconds** — Real-time hashrate
- **600 seconds (10 min)** — Smoothed hashrate
- **3600 seconds (1 hour)** — Stable hashrate
- **12 hours** — Daily trend
- **24 hours** — Long-term trend

**Calculation:** `hashrate = (sum of share difficulties) / (window duration in seconds)`

### Worker identification

Workers are identified by configurable fields from login:

```go
const (
    WorkersByRigID  WorkersMode = "rig-id"   // rigid field
    WorkersByUser   WorkersMode = "user"     // user field
    WorkersByPass   WorkersMode = "password" // pass field
    WorkersByAgent  WorkersMode = "agent"    // agent field
    WorkersByIP     WorkersMode = "ip"       // connection IP
    WorkersDisabled WorkersMode = "false"    // disabled
)
```

### Custom difficulty

Per-miner difficulty override:

```go
// Global custom difficulty
cfg.CustomDiff = 50000

// Per-miner via login suffix: WALLET+50000
// Miner login: {"login": "WALLET+50000", "pass": "x", ...}
```

---

## Protocol extensions

### Algorithm negotiation

Forward algorithm field from miners to pools:

```go
cfg.AlgoExtension = true

// Miner login includes: {"algo": ["cn/r", "rx/0", "cn/half"]}
// Proxy forwards algo to pool in submit requests
```

### RigID extension

Track miners by rig identifier:

```go
cfg.Workers = proxy.WorkersByRigID

// Miner login includes: {"rigid": "rig-alpha"}
// Worker stats grouped by rigid
```

---

## Configuration example

**config.json:**
```json
{
  "mode": "nicehash",
  "bind": [
    {"host": "0.0.0.0", "port": 3333, "tls": false},
    {"host": "0.0.0.0", "port": 3443, "tls": true}
  ],
  "pools": [
    {
      "url": "pool.lthn.io:3333",
      "user": "WALLET_ADDRESS",
      "pass": "x",
      "rig-id": "proxy-01",
      "algo": "cn/r",
      "tls": false,
      "enabled": true
    },
    {
      "url": "backup.pool.io:3333",
      "user": "WALLET_ADDRESS",
      "pass": "x",
      "tls": true,
      "tls-fingerprint": "abc123...",
      "enabled": true
    }
  ],
  "tls": {
    "enabled": true,
    "cert": "/etc/proxy/cert.pem",
    "cert_key": "/etc/proxy/key.pem"
  },
  "http": {
    "enabled": true,
    "host": "127.0.0.1",
    "port": 8080,
    "access-token": "secret-token",
    "restricted": false
  },
  "access-password": "miner-password",
  "custom-diff": 100000,
  "algo-ext": true,
  "workers": "rig-id",
  "access-log-file": "/var/log/proxy/access.log",
  "share-log-file": "/var/log/proxy/shares.log",
  "reuse-timeout": 60,
  "retries": 5,
  "retry-pause": 10,
  "watch": true,
  "rate-limit": {
    "max-connections-per-minute": 30,
    "ban-duration": 300
  }
}
```

---

## File structure

```
go-proxy/
├── go/
│   ├── proxy.go          # Proxy orchestrator, Job type, Config
│   ├── server.go         # TCP server with rate limiting
│   ├── miner.go          # Miner state machine
│   ├── worker.go         # Worker aggregate stats
│   ├── stats.go          # Global stats
│   ├── events.go         # Event bus
│   ├── config.go         # Configuration loading/reloading
│   ├── error.go          # Error types
│   ├── job.go            # Job value type
│   ├── state_impl.go     # Miner state implementation
│   ├── io_helpers.go     # I/O utilities
│   ├── sharelog_impl.go  # Share logging
│   ├── ax_helpers.go     # AX framework helpers
│   ├── core_impl.go      # Core integration
│   ├── reload_test.go    # Config reload tests
│   └── ... (50+ files)
├── go/pool/
│   ├── client.go         # StratumClient
│   └── strategy.go       # FailoverStrategy
├── go/splitter/
│   ├── nicehash/
│   │   ├── splitter.go   # NonceSplitter
│   │   ├── mapper.go     # NonceMapper
│   │   └── storage.go    # NonceStorage
│   └── simple/
│       ├── splitter.go   # SimpleSplitter
│       └── mapper.go     # SimpleMapper
├── go/log/
│   ├── access.go         # AccessLog
│   └── share.go          # ShareLog
├── go/api/
│   └── router.go         # HTTP API routes
├── docs/
│   └── ...
└── go.work
```

---

## Usage examples

### Basic proxy

```go
package main

import (
    "log"
    "dappco.re/go/core/proxy"
)

func main() {
    // Load config
    cfg, result := proxy.LoadConfig("config.json")
    if !result.OK {
        log.Fatal(result.Error)
    }

    // Create and start proxy
    p, result := proxy.New(cfg)
    if !result.OK {
        log.Fatal(result.Error)
    }

    // Start (blocks until Stop())
    p.Start()
}
```

### Custom splitter

```go
// Create custom splitter
splitter := nicehash.NewNonceSplitter(cfg, events, factory)

// Create proxy with custom splitter
p := proxy.NewWithSplitter(cfg, splitter)
p.Start()
```

### Event handling

```go
// Subscribe to events
events := proxy.NewEventBus()
sub := events.Subscribe(func(event proxy.Event) {
    switch e := event.(type) {
    case *proxy.AcceptEvent:
        log.Printf("Accepted: %d H/s from %s", e.Difficulty, e.Miner.User)
    case *proxy.RejectEvent:
        log.Printf("Rejected: %s from %s", e.Reason, e.Miner.User)
    }
})
defer sub.Unsubscribe()
```

### Monitoring API

```go
// Enable HTTP API
cfg.HTTP.Enabled = true
cfg.HTTP.Port = 8080

// Start proxy
p, _ := proxy.New(cfg)

// In another goroutine or process:
// curl http://localhost:8080/1/summary
// curl -H "Authorization: Bearer TOKEN" http://localhost:8080/1/workers
```

---

## Performance characteristics

| Metric | NiceHash Mode | Simple Mode |
|--------|---------------|-------------|
| Miners per pool connection | 256 | 1 |
| Connection reuse | Automatic | Configurable (ReuseTimeout) |
| Memory per miner | ~500 bytes | ~1 KB |
| CPU overhead | Low | Low |
| Latency | < 1ms | < 1ms |

---

## Related knowledge packs

| Package | Knowledge Pack | Relationship |
|---------|----------------|--------------|
| go-p2p | [../p2p/](../p2p/) | Peer-to-peer networking (alternative transport) |
| go-blockchain | [../blockchain/](../blockchain/) | Blockchain implementation (uses go-proxy for mining) |
| go-io | [../io/](../io/) | I/O abstraction (used for log file storage) |
| core/go | [../../../../../corego/README.md](../../../../../README.md) | Foundation framework (required dependency) |

---

*Knowledge Pack: go-proxy v1.0.0*
*Last Updated: 2026-06-17*
*Maintained by: Purberus <purberus@lthn.ai>*
