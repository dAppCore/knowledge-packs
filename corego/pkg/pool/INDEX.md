---
type: Package Index
package: pool
module: dappco.re/go/pool
title: go-pool Package Index
description: ProgPoWZ mining pool backend for Lethean blockchain
---

# go-pool Package Index

**Repository:** `core/go-pool`  
**Module:** `dappco.re/go/pool`  
**Algorithm:** ProgPoWZ (Lethean / Zano)  
**Status:** ✅ Complete Documentation  
**Replaces:** Node.js cryptonote-nodejs-pool  
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| RFC | Official specification | [plans/code/core/go/pool/RFC.md](../../../../../plans/code/core/go/pool/RFC.md) |

---

## Package overview

**go-pool** is a complete Stratum mining pool backend for the Lethean blockchain. It implements a production-ready pool with Stratum TCP server, ProgPoWZ share validation via CGo, variable difficulty, IP banning, Redis state store, HTTP API, webhook notifications, and clustering support.

### Key Features

- **Stratum TCP server** — Multi-port with per-port difficulty (plain + TLS)
- **ProgPoWZ share validation** — CGo wrapper around `zano-node-util`
- **Variable difficulty (vardiff)** — Per-miner ring buffer retargeting
- **Share trust algorithm** — Probability decay to reduce CPU usage
- **IP banning** — Time-based, triggered by invalid share ratio
- **Block template management** — Daemon RPC polling + ZMQ fallback
- **Payout system** — RBPPS (proportional) and solo modes
- **Redis state store** — Miners, hashrates, shares, blocks, payments, charts
- **HTTP API** — 17 endpoints, wire-compatible with Node.js pool
- **Server-Sent Events** — Real-time `/live_stats` streaming
- **Webhook notifications** — Block events, payments
- **Clustering** — Horizontal scaling via Redis pub/sub
- **Wire-compatible** — HTTP API matches Node.js pool byte-for-byte

### Architecture Layers

1. **Pool Orchestrator** — Main pool type, owns all subsystems, lifecycle management
2. **Stratum Layer** — TCP server, miner state machine, vardiff, jobs, banning, trust
3. **Daemon Layer** — JSON-RPC client, block template polling with ZMQ fallback
4. **Wallet Layer** — JSON-RPC client, payout processing
5. **Hash Layer** — CGo wrapper for ProgPoWZ share validation
6. **Redis Layer** — State store, key patterns, client wrapper
7. **Share Processing** — Share validation + Redis recording
8. **Block Unlocker** — Block maturity checker, reward distribution
9. **Payments** — RBPPS payout processor
10. **Charts** — Periodic data collector for statistics
11. **API** — HTTP server, stats aggregator, request handlers, SSE
12. **Webhook** — Async HTTP POST dispatcher for pool events

---

## Components

### Orchestrator

| Type | File | Purpose |
|------|------|---------|
| `Pool` | `pool.go` | Main orchestrator |
| `Config` | `config.go` | Pool configuration |

### Stratum Layer

| Type | File | Purpose |
|------|------|---------|
| `Server` | `stratum/server.go` | TCP server |
| `Miner` | `stratum/miner.go` | Miner state machine |
| `VardiffConfig` | `stratum/vardiff.go` | Variable difficulty |
| `Job` | `stratum/job.go` | Mining job |
| `Template` | `stratum/template.go` | Block template |
| `BanConfig` | `stratum/ban.go` | IP banning |
| `TrustConfig` | `stratum/trust.go` | Share trust |

### Daemon Layer

| Type | File | Purpose |
|------|------|---------|
| `Client` | `daemon/client.go` | Daemon RPC client |
| `Poller` | `daemon/poller.go` | Template poller |

### Wallet Layer

| Type | File | Purpose |
|------|------|---------|
| `Client` | `wallet/client.go` | Wallet RPC client |
| `Processor` | `payments/processor.go` | Payout processor |

### Hash Layer

| Type | File | Purpose |
|------|------|---------|
| `ProgPoWZ` | `hash/progpowz.go` | CGo wrapper |
| `zano.h` | `hash/zano.h` | C header |

### Redis Layer

| Type | File | Purpose |
|------|------|---------|
| `Client` | `redis/client.go` | Redis client wrapper |
| `Keys` | `redis/keys.go` | Redis key patterns |

### Processing Layers

| Type | File | Purpose |
|------|------|---------|
| `Processor` | `share/processor.go` | Share validation |
| `Unlocker` | `unlocker/unlocker.go` | Block maturity checker |
| `Collector` | `charts/collector.go` | Charts data collector |

### API Layer

| Type | File | Purpose |
|------|------|---------|
| `Server` | `api/server.go` | HTTP API server |
| `Stats` | `api/stats.go` | Stats aggregator |
| `Handlers` | `api/handlers.go` | Request handlers |

### Webhook Layer

| Type | File | Purpose |
|------|------|---------|
| `Notifier` | `webhook/notifier.go` | Webhook notifications |

---

## File structure

```
go-pool/
├── go/
│   ├── pool.go                    # Main Pool orchestrator
│   ├── config.go                 # Configuration
│   ├── cmd/
│   │   └── go-pool/
│   │       └── main.go           # Binary entrypoint
│   ├── stratum/
│   │   ├── server.go             # TCP server
│   │   ├── miner.go              # Miner state machine
│   │   ├── vardiff.go            # Variable difficulty
│   │   ├── job.go                # Job management
│   │   ├── template.go           # Block template
│   │   ├── ban.go                # IP banning
│   │   └── trust.go              # Share trust
│   ├── daemon/
│   │   ├── client.go             # Daemon RPC client
│   │   └── poller.go             # Template poller
│   ├── wallet/
│   │   └── client.go             # Wallet RPC client
│   ├── hash/
│   │   ├── progpowz.go           # CGo wrapper
│   │   └── zano.h                # C header
│   ├── redis/
│   │   ├── client.go             # Redis client wrapper
│   │   └── keys.go               # Redis key patterns
│   ├── share/
│   │   └── processor.go          # Share validation
│   ├── unlocker/
│   │   └── unlocker.go           # Block maturity checker
│   ├── payments/
│   │   └── processor.go          # Payout processor
│   ├── charts/
│   │   └── collector.go          # Charts data collector
│   ├── api/
│   │   ├── server.go             # HTTP API server
│   │   ├── stats.go              # Stats aggregator
│   │   └── handlers.go           # Request handlers
│   ├── webhook/
│   │   └── notifier.go           # Webhook notifications
│   └── tests/
│       └── ...                   # Test files
├── go.mod
├── go.sum
└── go.work
```

---

## Quick start

### Basic Setup

```go
config, _ := pool.LoadConfig("config.json")
p, _ := pool.New(config)
p.Start()
```

### Configuration File

```json
{
  "stratum": {
    "ports": [{"port": 5555, "difficulty": 100000}],
    "vardiff": {"enabled": true, "target_time": 60},
    "ban": {"enabled": true, "invalid_share_threshold": 0.5},
    "trust": {"enabled": true, "penalty": 0.1, "decay": 0.999}
  },
  "daemon": {"host": "localhost", "port": 18081, "zmq_block": "tcp://localhost:18083"},
  "wallet": {"host": "localhost", "port": 18082, "address": "LYourAddress"},
  "redis": {"host": "localhost", "port": 6379},
  "api": {"host": "0.0.0.0", "port": 8080, "admin_password": "secret"},
  "webhook": {"block_found_url": "https://webhook.site/..."}
}
```

### Using the CLI

```bash
go build -o go-pool ./cmd/go-pool
./go-pool -config config.json
```

---

## Use cases

### 1. Run a Lethean Mining Pool

```go
config := &pool.Config{
    Stratum: pool.StratumConfig{
        Ports: []pool.PortConfig{{Port: 5555, Difficulty: 100000}},
    },
    Daemon: pool.DaemonConfig{Host: "localhost", Port: 18081},
    Redis:  pool.RedisConfig{Host: "localhost", Port: 6379},
}
p, _ := pool.New(config)
p.Start()
```

### 2. Multiple Ports with Different Difficulties

```go
Ports: [
    {Port: 5555, Difficulty: 100000, TLS: false},
    {Port: 5556, Difficulty: 1000000, TLS: false},
    {Port: 5557, Difficulty: 10000000, TLS: true},
]
```

### 3. Get Pool Statistics

```go
stats := p.Stats()
fmt.Printf("HashRate: %v H/s\n", stats.HashRate)
fmt.Printf("Miners: %d\n", stats.Miners)
```

### 4. Clustering with Redis

```go
// Multiple instances share Redis
config1 := &pool.Config{Redis: pool.RedisConfig{Host: "redis-cluster"}}
config2 := &pool.Config{Redis: pool.RedisConfig{Host: "redis-cluster"}}
p1, _ := pool.New(config1)
p2, _ := pool.New(config2)
p1.Start()
p2.Start()
```

### 5. Webhook Notifications

```go
Webhook: pool.WebhookConfig{
    BlockFoundURL:    "https://discord.com/api/webhooks/...",
    BlockUnlockedURL: "https://discord.com/api/webhooks/...",
    PaymentURL:        "https://discord.com/api/webhooks/...",
}
```

---

## Configuration

### StratumConfig

```go
type StratumConfig struct {
    Ports       []PortConfig
    Vardiff     VardiffConfig
    Ban         BanConfig
    Trust       TrustConfig
    MaxConns    int
    ReadTimeout time.Duration
}

type VardiffConfig struct {
    Enabled        bool
    MinDifficulty  int64
    MaxDifficulty  int64
    TargetTime    time.Duration
    RetargetRate  int
    Variance      float64
}

type BanConfig struct {
    Enabled               bool
    InvalidShareThreshold float64
    BanDuration           int
}

type TrustConfig struct {
    Enabled bool
    Penalty float64
    Decay   float64
}
```

### DaemonConfig

```go
type DaemonConfig struct {
    Host     string
    Port     int
    ZMQBlock string // ZMQ address for block notifications
}
```

### WalletConfig

```go
type WalletConfig struct {
    Host           string
    Port           int
    Address        string
    PayoutThreshold float64
    Fee            float64
}
```

### RedisConfig

```go
type RedisConfig struct {
    Host     string
    Port     int
    Password string
    DB       int
    PoolSize int
}
```

### APIConfig

```go
type APIConfig struct {
    Host       string
    Port       int
    AdminUser string
    AdminPass string
    TLS        bool
    TLSCert    string
    TLSKey     string
}
```

---

## Stratum protocol

### Connection Flow

```
Miner → TCP Connect → Server → Login → Job Distribution → Share Submission
```

### Supported Methods

- `login` — Authenticate and get initial job
- `submit` — Submit share for validation
- `job` — New mining job (server → miner)
- `set_difficulty` — Update difficulty (server → miner)

### Difficulty Adjustment

Variable difficulty uses ring buffer retargeting with configurable:
- Target time per share
- Retarget rate (shares between adjustments)
- Variance tolerance
- Min/max difficulty bounds

---

## Payout system

### Modes

- **RBPPS** (Round-Based Proportional) — Default, recommended
- **Solo** — Solo mining mode
- **PPLNS** — Pay Per Last N Shares (planned)

### Process

1. Block Found → Record shares
2. Block Mature (60 confirmations) → Unlock
3. Reward Distribution → Calculate shares
4. Wallet Transfer → Send payments

---

## Redis state store

### Key Patterns

- Miner state and statistics
- Pool statistics
- Block information
- Shares per block
- Chart data
- Payout records

---

## HTTP API

### 17 Wire-Compatible Endpoints

**Public:**
- `/get_stat` — Pool statistics
- `/get_network_stat` — Network statistics
- `/get_mining_stat` — Mining statistics
- `/get_block_candidate` — Current block
- `/get_last_block_header` — Last block header
- `/get_pool_block` — Pool block info
- `/get_pool_blocks` — List pool blocks
- `/get_market` — Market price
- `/get_chart` — Chart data

**Miner:**
- `/get_miner_stat/:address` — Miner statistics
- `/get_miner_chart/:address` — Miner chart data

**Admin (Password Protected):**
- `/admin/stats` — Detailed stats
- `/admin/miners` — List miners
- `/admin/blocks` — List blocks
- `/admin/payments` — List payments
- `/admin/log` — Stream logs

**Real-Time:**
- `/live_stats` — Server-Sent Events

---

## Clustering

Multiple pool instances share state via Redis pub/sub:
- Event broadcasting
- Shared miner state
- Distributed locking
- Load balancing

---

## Webhook notifications

Async HTTP POST notifications for:
- Block found
- Block unlocked
- Block orphaned
- Payment sent

---

## Testing

All subsystems have comprehensive tests:
- Stratum layer tests
- Daemon client tests
- Wallet client tests
- Hash validation tests
- Redis client tests
- Share processor tests
- Payments tests
- API tests
- Webhook tests

### Running Tests

```bash
cd ~/Code/core/go-pool/go
go test -v ./...
go test -cover ./...
go test -bench . -benchmem
```

---

## Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/pool` |
| **Repository** | `core/go-pool` |
| **Algorithm** | ProgPoWZ |
| **Language** | Go 1.26+ (with CGo) |
| **Dependencies** | `dappco.re/go`, `dappco.re/go/api`, `dappco.re/go/stream`, `dappco.re/go/cache`, `zano-node-util` (C) |
| **Test Triplets** | ✅ Complete |
| **RFC Compliance** | ✅ Verified |
| **Documentation** | ✅ Complete |
| **Wire-Compatible** | ✅ Node.js pool |

---

## Related packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-proxy](../proxy/) | Stratum mining proxy | ../proxy/ |
| [go-p2p](../p2p/) | P2P networking | ../p2p/ |
| [go-api](../api/) | HTTP API framework | ../api/ |
| [go-stream](../stream/) | Redis pub/sub bridge | ../stream/ |
| [go-cache](../cache/) | Caching layer | ../cache/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |

---

## Tags

```yaml
- mining-pool
- stratum
- progpowz
- lethean
- zano
- cryptonote
- redis
- tcp
- tls
- payouts
- block-template
- vardiff
- share-validation
- clustering
- webhook
- monitoring
- production-ready
- wire-compatible
```

---

*Package index generated: 2026-06-17T18:00:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
