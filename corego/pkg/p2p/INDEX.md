---
type: Package Index
package: p2p
title: go-p2p Package Documentation Index
---
# go-p2p Package Documentation

> **Peer-to-Peer Networking for Distributed Mining** — Complete P2P networking layer

**Module:** `dappco.re/go/core/p2p`
**Repository:** `core/go-p2p`
**RFC:** [../../../../../plans/code/core/go/p2p/RFC.md](../../../../../plans/code/core/go/p2p/RFC.md)

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [README.md](./README.md) | Complete package overview, API reference, architecture, examples |
| [RFC.md](../../../../../plans/code/core/go/p2p/RFC.md) | Canonical specification |
| [RFC.models.md](../../../../../plans/code/core/go/p2p/RFC.models.md) | Data models and structures |

---

## 🗂️ Subpackages

### Core Subpackages

| Package | Description | Key Files |
|---------|-------------|-----------|
| [go/node/](file:///Users/snider/Code/core/go-p2p/go/node/) | Node identity, roles, controller, worker | identity.go, controller.go, worker.go, service.go, protocol.go |
| [go/ueps/](file:///Users/snider/Code/core/go-p2p/go/ueps/) | Urgent Event Protocol Server (consciousness-routing overlay) | packet.go, reader.go, bench_test.go |
| [go/levin/](file:///Users/snider/Code/core/go-p2p/go/levin/) | CryptoNote Levin protocol implementation | packet.go, header.go, command.go |
| [pkg/contentbus/](file:///Users/snider/Code/core/go-p2p/pkg/contentbus/) | Pub/sub controller for P2P consumers | controller.go, options.go |
| [pkg/api/](file:///Users/snider/Code/core/go-p2p/pkg/api/) | HTTP API provider for P2P operations | provider.go, routes.go |

---

## 🎯 Quick Links

### Node Architecture
- **Node Roles:** `RoleController`, `RoleWorker`, `RoleDual`
- **Identity:** Ed25519 public/private keys, X25519 for ECDH
- **Storage:** XDG-compliant (`~/.local/share/lethean-desktop/node/private.key`, `~/.config/lethean-desktop/node.json`)

### Message Types (16 total)

**Connection Lifecycle:**
- `MsgHandshake`, `MsgHandshakeAck` — Connection establishment with challenge-response auth
- `MsgPing`, `MsgPong` — Keepalive and latency measurement
- `MsgDisconnect` — Graceful disconnection

**Miner Operations:**
- `MsgStartMiner`, `MsgStopMiner`, `MsgMinerAck` — Miner lifecycle control
- `MsgGetStats`, `MsgStats` — Statistics collection and reporting

**Deployment:**
- `MsgDeploy`, `MsgDeployAck` — Bundle deployment (miners, profiles)

**Logging:**
- `MsgGetLogs`, `MsgLogs` — Log retrieval

**Error Handling:**
- `MsgError` — Error responses with codes

### Transport Layer
- **Protocol:** WebSocket (gorilla/websocket) with TLS support
- **Encryption:** SMSG (Secure Messaging) via X25519 ECDH + HMAC-SHA256
- **Deduplication:** 5-minute message ID window prevents replay attacks
- **Rate Limiting:** Token bucket per peer (configurable)

### Peer Management
- **PeerRegistry:** Persistent peer storage with allowlist support
- **Poindexter KD-tree:** Spatial indexing for intelligent peer selection
- **Metrics:** Ping latency, hops, geographic distance, reliability score (0-100)

---

## 🏗️ Architecture Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                           │
│  Mining coordination, command dispatch, stats collection      │
├─────────────────────────────────────────────────────────────┤
│                    Message Layer                               │
│  16 message types with JSON payloads, routing, replies        │
├─────────────────────────────────────────────────────────────┤
│                    Transport Layer                             │
│  WebSocket with TLS, SMSG encryption, deduplication, rate limiting │
├─────────────────────────────────────────────────────────────┤
│                    Peer Management Layer                       │
│  PeerRegistry, Poindexter KD-tree, PeerConnection state       │
├─────────────────────────────────────────────────────────────┤
│                    Node Identity Layer                          │
│  NodeIdentity, NodeManager, Ed25519/X25519 key pairs          │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 Related Knowledge Packs

| Package | Knowledge Pack | Relationship |
|---------|----------------|--------------|
| go-blockchain | [../../pkg/blockchain/](../../pkg/blockchain/) | Uses go-p2p for peer-to-peer blockchain sync |
| go-lns | [../../pkg/lns/](../../pkg/lns/) | Uses go-p2p for distributed name system |
| go-dns | [../../pkg/dns/](../../pkg/dns/) | Uses go-p2p for DNS propagation |
| core/go | [../../../../../corego/README.md](../../../../../README.md) | Foundation framework (required dependency) |

---

## 📊 Statistics

- **Total Message Types:** 16
- **Node Roles:** 3 (Controller, Worker, Dual)
- **Protocol Versions:** 1.0 (backwards-compatible)
- **Authentication Modes:** 2 (Open, Allowlist)
- **Encryption:** X25519 ECDH + HMAC-SHA256 (SMSG)
- **Deduplication Window:** 5 minutes
- **Test Coverage:** High (Good/Bad/Ugly triplets per file)
- **Subpackages:** 5 (node, ueps, levin, contentbus, api)

---

## 🔗 Source Code Structure

```
go-p2p/
├── go/
│   ├── node/              # Node identity, controller, worker, service
│   │   ├── identity.go    # NodeIdentity, NodeManager
│   │   ├── controller.go  # Controller for dispatching commands
│   │   ├── worker.go      # Worker for processing messages
│   │   ├── service.go     # Service lifecycle management
│   │   ├── protocol.go    # Message types and payloads
│   │   ├── message.go     # Message structure and routing
│   │   ├── transport.go   # WebSocket transport
│   │   ├── bundle.go      # Deployment bundles
│   │   └── ... (20+ files)
│   ├── ueps/              # Urgent Event Protocol Server
│   │   ├── packet.go      # UEPS packet structure
│   │   ├── reader.go      # Packet reader/parser
│   │   └── bench_test.go  # Benchmark tests
│   └── levin/             # CryptoNote Levin protocol
│       ├── packet.go      # Levin packet encoding
│       ├── header.go      # Levin header structure
│       └── command.go     # Levin command constants
├── pkg/
│   ├── contentbus/        # Pub/sub for P2P consumers
│   └── api/               # HTTP API provider
└── go.work
```

---

## 🚀 Usage Examples

### Basic Node Setup

```go
// Initialize node identity
manager, err := p2p.NewNodeManager()
if err != nil { panic(err) }

// Create peer registry
registry, err := p2p.NewPeerRegistry("/path/to/peers.json")

// Configure transport
config := p2p.DefaultTransportConfig()
config.ListenAddr = ":9091"

// Create transport
transport := p2p.NewTransport(manager, registry, config)

// Start listening
transport.Start()
```

### Sending Messages

```go
// Create a message
msg, err := p2p.NewMessage(p2p.MsgGetStats, "node-1", "node-2", nil)

// Send to specific peer
transport.Send("node-2", msg)

// Broadcast to all peers
transport.Broadcast(msg)
```

### Controller Operations

```go
controller := p2p.NewController(manager, registry, transport)

// Get stats from remote peer
stats, err := controller.GetRemoteStats("peer-id")

// Start miner on remote peer
controller.StartRemoteMiner("peer-id", "xmrig", "profile-123", nil)
```

### UEPS Protocol

```go
// Create signed UEPS packet
builder := ueps.NewBuilder(ueps.IntentCompute, payloadBytes)
builder.Header.ThreatScore = 1000
serialised, err := builder.MarshalAndSign(sharedSecret)
```

---

*Knowledge Pack: go-p2p v1.0.0*
*Last Updated: 2026-06-17*
*Maintained by: Purberus <purberus@lthn.ai>*
