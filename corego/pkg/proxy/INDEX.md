---
type: Package Index
package: proxy
title: go-proxy Package Documentation Index
---
# go-proxy Package Documentation

> **Stratum Mining Proxy** — CryptoNote protocol proxy library

**Module:** `dappco.re/go/core/proxy`
**Repository:** `core/go-proxy`
**RFC:** [../../../../../plans/code/core/go/proxy/RFC.md](../../../../../plans/code/core/go/proxy/RFC.md)

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [README.md](./README.md) | Complete package overview, architecture, API reference, examples |
| [RFC.md](../../../../../plans/code/core/go/proxy/RFC.md) | Canonical specification |

---

## 🗂️ Subpackages

### Core Subpackages

| Package | Description | Key Files |
|---------|-------------|-----------|
| [go/pool/](file:///Users/snider/Code/core/go-proxy/go/pool/) | Stratum client, failover strategy | client.go, strategy.go |
| [go/splitter/nicehash/](file:///Users/snider/Code/core/go-proxy/go/splitter/nicehash/) | Nonce-splitting mode (256 miners/pool) | splitter.go, mapper.go, storage.go |
| [go/splitter/simple/](file:///Users/snider/Code/core/go-proxy/go/splitter/simple/) | Passthrough mode (1 miner/pool) | splitter.go, mapper.go |
| [go/log/](file:///Users/snider/Code/core/go-proxy/go/log/) | Access log, share log | access.go, share.go |
| [go/api/](file:///Users/snider/Code/core/go-proxy/go/api/) | HTTP monitoring API | router.go |

---

## 🎯 Quick Links

### Operating Modes

**NiceHash Mode (Recommended for high-volume):**
- Splits 32-bit nonce space by fixing byte 39 of blob
- 256 miners share one pool connection
- Maximizes pool connection reuse
- Minimal reconnection overhead

**Simple Mode:**
- One pool connection per miner
- Optional connection reuse (ReuseTimeout)
- Simpler architecture, higher connection count

### Miner State Machine

Linear state transitions:
```
WaitLogin (10s timeout) → WaitReady (600s timeout) → Ready (600s inactivity timeout) → Closing
```

### Hashrate Windows

Rolling hashrate calculated over 5 time windows:
- **60 seconds** — Real-time
- **600 seconds** — 10-minute average
- **3600 seconds** — 1-hour average
- **12 hours** — Daily trend
- **24 hours** — Long-term trend

### Protocol

- **Miner-side:** Stratum JSON-RPC over TCP/TLS
- **Pool-side:** Stratum JSON-RPC over TCP/TLS
- **Extensions:** Algorithm negotiation, RigID, custom difficulty

---

## 🏗️ Architecture Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    Monitoring Layer                           │
│  HTTP API: /1/summary, /1/workers, /1/miners                 │
│  Access log (JSON lines), Share log (JSON lines)             │
├─────────────────────────────────────────────────────────────┤
│                    Proxy Orchestrator                         │
│  Proxy — tick loop, listeners, splitter, stats, workers, bus  │
├─────────────────────────────────────────────────────────────┤
│                    Splitter Layer                              │
│  NonceSplitter: 256-slot table, NonceMapper pool               │
│  SimpleSplitter: Passthrough, upstream reuse pool             │
├─────────────────────────────────────────────────────────────┤
│                    Pool Layer                                 │
│  StratumClient: Outbound pool TCP/TLS connection              │
│  FailoverStrategy: Primary + ordered fallbacks               │
├─────────────────────────────────────────────────────────────┤
│                    Miner Layer                                │
│  Miner: State machine, TCP connection handler                │
│  Server: TCP listener, rate limiting, connection acceptance   │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 Related Knowledge Packs

| Package | Knowledge Pack | Relationship |
|---------|----------------|--------------|
| go-p2p | [../p2p/](../p2p/) | Peer-to-peer networking (alternative to Stratum) |
| go-blockchain | [../blockchain/](../blockchain/) | Blockchain implementation (uses go-proxy for mining) |
| go-io | [../io/](../io/) | I/O abstraction (used for log file storage) |
| go-api | [../api/](../api/) | HTTP API framework (used for monitoring API) |
| core/go | [../../../../../corego/README.md](../../../../../README.md) | Foundation framework (required dependency) |

---

## 📊 Statistics

- **Total Files:** 80+ Go files
- **Test Coverage:** High (Good/Bad/Ugly triplets per file)
- **Subpackages:** 5 (pool, nicehash, simple, log, api)
- **Lines of Code:** ~10,000 (estimated)
- **Dependencies:** 2 (core/go, core/api)

### NiceHash Mode Stats
- **Miners per pool connection:** 256
- **NonceStorage slots:** 256 per NonceMapper
- **Connection reuse:** Automatic
- **Slot allocation:** Round-robin with cursor

### Simple Mode Stats
- **Miners per pool connection:** 1
- **Connection reuse:** Configurable via ReuseTimeout
- **Idle pool size:** Configurable
- **Memory per mapper:** ~1 KB

### Protocol Stats
- **Miner states:** 4 (WaitLogin, WaitReady, Ready, Closing)
- **Worker identification modes:** 6 (rig-id, user, password, agent, ip, disabled)
- **Hashrate windows:** 5
- **Log types:** 2 (Access, Share)
- **HTTP endpoints:** 3 (/1/summary, /1/workers, /1/miners)

---

## 🔗 Source Code Structure

```
go-proxy/
├── go/
│   ├── proxy.go          # Proxy orchestrator, Job, Config types
│   ├── server.go         # TCP server with rate limiting
│   ├── miner.go          # Miner state machine
│   ├── worker.go         # Worker aggregate stats with rolling windows
│   ├── stats.go          # Global stats and hashrate calculation
│   ├── events.go         # Event bus (Login, Submit, Accept, Reject, Close)
│   ├── config.go         # JSON config loading, validation, hot-reload
│   ├── job.go            # Job value type (blob, job_id, target, algo)
│   ├── error.go          # Error types and codes
│   ├── state_impl.go     # Miner state implementation
│   ├── io_helpers.go     # I/O utilities
│   ├── sharelog_impl.go  # Share logging implementation
│   ├── ax_helpers.go     # AX framework helpers
│   ├── core_impl.go      # Core integration
│   └── ... (50+ additional files)
├── go/pool/
│   ├── client.go         # StratumClient - pool TCP/TLS connection
│   └── strategy.go       # FailoverStrategy - primary + fallbacks
├── go/splitter/
│   ├── nicehash/
│   │   ├── splitter.go   # NonceSplitter - manages NonceMapper pool
│   │   ├── mapper.go     # NonceMapper - one pool connection + 256 slots
│   │   └── storage.go    # NonceStorage - 256-slot fixed-byte allocation
│   └── simple/
│       ├── splitter.go   # SimpleSplitter - passthrough mode
│       └── mapper.go     # SimpleMapper - one pool connection per miner
├── go/log/
│   ├── access.go         # AccessLog - connection open/close logging
│   └── share.go          # ShareLog - accept/reject logging
├── go/api/
│   └── router.go         # HTTP API routes and handlers
├── docs/
│   └── ...
└── go.work
```

---

## 🚀 Usage Examples

### Basic Usage

```go
// Load config
cfg, result := proxy.LoadConfig("config.json")

// Create proxy
p, result := proxy.New(cfg)

// Start (blocks until Stop)
p.Start()

// Stop cleanly
p.Stop()
```

### Configuration

```json
{
  "mode": "nicehash",
  "bind": [{"host": "0.0.0.0", "port": 3333, "tls": false}],
  "pools": [
    {
      "url": "pool.lthn.io:3333",
      "user": "WALLET",
      "pass": "x",
      "tls": false,
      "enabled": true
    }
  ],
  "workers": "rig-id",
  "custom-diff": 100000,
  "algo-ext": true,
  "http": {"enabled": true, "host": "127.0.0.1", "port": 8080}
}
```

### Event Subscription

```go
events := proxy.NewEventBus()
sub := events.Subscribe(func(event proxy.Event) {
    switch e := event.(type) {
    case *proxy.LoginEvent:
        fmt.Printf("Login: %s\n", e.Miner.User)
    case *proxy.AcceptEvent:
        fmt.Printf("Accept: %d H/s\n", e.Difficulty)
    }
})
```

### Custom Difficulty

```go
// Global setting
cfg.CustomDiff = 50000

// Per-miner via login suffix
// Miner sends: {"login": "WALLET+50000", "pass": "x", ...}
```

---

## 🛡️ Security Features

| Feature | Description |
|---------|-------------|
| **TLS (Inbound)** | Encrypt miner connections |
| **TLS (Outbound)** | Encrypt pool connections with certificate pinning |
| **Rate Limiting** | Per-IP token bucket (configurable) |
| **Access Password** | Login authentication |
| **HTTP Bearer Auth** | Monitoring API authentication |
| **Restricted Mode** | Read-only HTTP API |
| **Certificate Pinning** | Pool certificate fingerprint validation |

---

## 📈 Protocol Extensions

| Extension | Description | RFC Field |
|-----------|-------------|-----------|
| Algorithm Negotiation | Forward algo from miners to pools | `algo` in login/job |
| RigID | Track miners by rig identifier | `rigid` in login |
| Custom Difficulty | Per-miner difficulty override | Login suffix: `WALLET+DIFF` |
| TLS Fingerprint | Pool certificate pinning | `tls-fingerprint` in pool config |
| NiceHash Mode | Nonce-space splitting | Mode configuration |

---

*Knowledge Pack: go-proxy v1.0.0*
*Last Updated: 2026-06-17*
*Maintained by: Purberus <purberus@lthn.ai>*
