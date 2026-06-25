---
type: Package Documentation
package: lns
module: dappco.re/go/lns
tier: lib
repo: core/go-lns
lang: go
tags:
  - lns
  - naming
  - dns
  - resolution
  - handshake
  - blockchain
  - covenants
---
# go-lns — Lethean Name System Knowledge Pack

**RFC:** [plans/code/core/go/lns/RFC.md](../../../../../plans/code/core/go/lns/RFC.md)
**Source:** [~/Code/core/go-lns/](file:///Users/snider/Code/core/go-lns/)
**Module:** `dappco.re/go/lns`

---

## Overview

`go-lns` is the native Go implementation of the Lethean Name System (LNS), providing:

- **.lthn name registration** — Auction-based name acquisition (OPEN → BID → REVEAL → REGISTER)
- **DNS resolution** — A, AAAA, TXT, NS, DS record resolution for .lthn names
- **VPN gateway discovery** — Gateway lookup for Lethean VPN network
- **Covenant enforcement** — Name rules, reserved names, locked catalogs
- **CoreGO integration** — Full CoreApp service with automatic MCP/API/CLI exposure

### Lineage
- **Forked from:** Handshake (HSD 8.99.0)
- **Upstream:** `forge.lthn.ai/lthn/itns-sidechain`
- **Implementation:** Native Go, no JS runtime dependency

### Design Philosophy
- **Self-contained:** No runtime dependence on JS reference implementation
- **Package-first:** Each subsystem (`pkg/*`) owns its domain
- **API mirroring:** Top-level `lns.go` mirrors all public APIs from subpackages
- **CoreGO native:** Every subsystem is a CoreApp service with actions

---

## Package structure

```
core/go-lns/
├── go/                          # Go source (module: dappco.re/go/lns)
│   ├── lns.go                    # Top-level API mirror + service entrypoint
│   ├── service.go                # Core service registration
│   ├── go.mod                    # Module definition
│   ├── go.sum                    # Dependency checksums
│   ├── lns_test.go               # Package tests
│   ├── lns_example_test.go       # Example tests
│   ├── action_handlers.go        # Core action handlers
│   │
│   ├── internal/                 # Internal helpers
│   │   └── nameutil/             # Name canonicalization + validation
│   │
│   └── pkg/                      # Subpackages (see below)
│       ├── covenant/             # Name rules + catalogs
│       ├── dns/                  # DNS resolution + encoding
│       ├── primitives/           # Shared types + serialization
│       ├── chain/                # Chain sync + validation
│       ├── blockstore/           # Block storage (go-store)
│       ├── wallet/               # HD wallet + key management
│       ├── net/                  # Peer networking (go-p2p)
│       ├── mempool/              # Transaction mempool
│       ├── mining/               # Block production
│       ├── hd/                   # HD key derivation
│       └── script/               # Transaction scripting
│
├── docs/                        # Documentation
│   ├── RFC.md                   # Primary spec (canonical)
│   ├── RFC.lns.md               # LNS-specific protocol details
│   ├── CLAUDE.md                # Agent guidance
│   └── js-*/                    # JS reference (read-only, porting source)
│
├── tests/                       # Integration tests
└── README.md                    # Quick start guide
```

---

## Subpackage catalogue

### pkg/covenant/
**Responsibility:** Name rules, reserved/locked catalogs, verification logic

| File | Purpose | Actions Prefix |
|------|---------|----------------|
| `covenant.go` | Covenant enums, rule helpers | `lns.name.*` |
| `catalog.go` | Reserved/locked name catalogs | `lns.name.*` |
| `verification.go` | Name state verification | `lns.name.*` |

**Key Types:**
```go
type Covenant int  // OPEN, BID, REVEAL, REGISTER, RENEW, REVOKE, TRANSFER, UPDATE
type NameState struct { Name string; State Covenant; Height uint64; Owner string }
```

**Actions:**
- `lns.name.register` — OPEN + BID + REVEAL + REGISTER sequence
- `lns.name.renew` — Renew before expiry
- `lns.name.transfer` — Transfer to new owner
- `lns.name.update` — Update DNS records
- `lns.name.revoke` — Revoke name
- `lns.name.lookup` — Check name state
- `lns.name.auction` — View auction state
- `lns.name.available` — Check availability

### pkg/dns/
**Responsibility:** DNS-oriented resolution, record encoding, lookup helpers

| File | Purpose | Actions Prefix |
|------|---------|----------------|
| `resolver.go` | DNS resolution logic | `lns.dns.*` |
| `encoding.go` | Record encoding/decoding | `lns.dns.*` |
| `lookup.go` | Name lookup helpers | `lns.dns.*` |

**Key Types:**
```go
type DNSRecord struct {
    Name  string
    Type  string  // A, AAAA, TXT, NS, DS
    Data  string
    TTL   uint32
}
type NameRecord struct {
    Name   string
    A      []string
    AAAA   []string
    TXT    []string
    NS     []string
    DS     []string
}
```

**Actions:**
- `lns.dns.resolve` — A record lookup
- `lns.dns.resolve.txt` — TXT record lookup
- `lns.dns.resolve.all` — All record types
- `lns.dns.reverse` — PTR (reverse DNS)
- `lns.dns.serve` — Start DNS server
- `lns.dns.health` — Health check
- `lns.dns.discover` — Rescan chain

### pkg/primitives/
**Responsibility:** Shared LNS data structures, binary/JSON forms, state types

**Key Types:**
```go
type NameHash [32]byte      // SHA3-256 hash of normalized name
type NameID [32]byte        // Unique name identifier
type RecordData struct {    // Generic record data
    Type  string
    Data  []byte
}
type NameState struct {    // Current state of a name
    Name     string
    Hash     NameHash
    State    Covenant
    Height  uint64
    Owner   string
    Value   uint64
}
```

### pkg/wallet/
**Responsibility:** HD wallet, key management, transaction signing

**Key Types:**
```go
type Wallet struct {
    Seed     [32]byte
    Keys     []*KeyPair
    Addresses []string
}
type KeyPair struct {
    Private [32]byte
    Public  [33]byte  // Compressed public key
}
```

**Actions:**
- `lns.wallet.create` — Create HD wallet
- `lns.wallet.balance` — Get balance
- `lns.wallet.send` — Send HNS
- `lns.wallet.receive` — Generate address
- `lns.wallet.history` — Transaction history
- `lns.wallet.import` — Import seed/key
- `lns.wallet.export` — Export seed

### pkg/chain/
**Responsibility:** Chain synchronization, validation, block storage

**Key Types:**
```go
type Block struct {
    Header   BlockHeader
    Transactions []Transaction
}
type BlockHeader struct {
    Version       uint8
    PreviousHash [32]byte
    MerkleRoot   [32]byte
    Timestamp    uint64
    Height       uint64
    Nonce        uint64
}
```

**Actions:**
- `lns.chain.height` — Current height
- `lns.chain.block` — Get block by height/hash
- `lns.chain.sync` — Start sync
- `lns.chain.status` — Sync status
- `lns.chain.peers` — Connected peers

### pkg/blockstore/
**Responsibility:** Block storage via go-store (SQLite backend)

**Dependencies:**
- `dappco.re/go/store` — SQLite key-value store

### pkg/net/
**Responsibility:** Peer networking via go-p2p

**Dependencies:**
- `dappco.re/go/p2p` — P2P networking layer

**Actions:**
- `lns.peer.connect` — Connect to peer
- `lns.peer.disconnect` — Disconnect from peer
- `lns.peer.broadcast` — Broadcast transaction
- `lns.peer.sync` — Sync with peer

### pkg/mempool/
**Responsibility:** Transaction mempool

**Key Types:**
```go
type Mempool struct {
    Transactions map[[32]byte]*Transaction
    FeePool     *FeePool
}
type FeePool struct {
    MinFee  uint64
    MaxFee  uint64
    Median uint64
}
```

**Actions:**
- `lns.mempool.add` — Add transaction to mempool
- `lns.mempool.remove` — Remove transaction
- `lns.mempool.get` — Get transaction by hash
- `lns.mempool.fees` — Get fee estimates

### pkg/mining/
**Responsibility:** Block production, mining operations

### pkg/hd/
**Responsibility:** HD (Hierarchical Deterministic) key derivation

### pkg/script/
**Responsibility:** Transaction scripting

---

## Core actions reference

All actions are registered with Core and automatically expose via:
- **MCP (Model Context Protocol)** — For AI agent integration
- **REST API** — For HTTP clients
- **CLI** — For command-line usage

### Action Registration

```go
func Register(c *core.Core) error {
    // Register all action handlers
    c.Action("lns.name.register").Handle(nameRegisterHandler)
    c.Action("lns.dns.resolve").Handle(dnsResolveHandler)
    // ... all other actions
    return nil
}
```

### Action Invocation

```go
// From another service
result := c.Action("lns.dns.resolve").Run(ctx, core.Options{
    core.Option{Key: "name", Value: "charon.lthn"},
})

if !result.OK {
    return core.E("myapp", "DNS resolution failed", result.Value)
}

records := result.Value.([]DNSRecord)
```

---

## Quick start

### Basic Usage

```go
package main

import (
    core "dappco.re/go/core"
    "dappco.re/go/lns"
)

func main() {
    // Create Core with LNS service
    c := core.New(
        core.WithService(lns.Register),
        core.WithServiceLock(),
    )
    
    // Resolve a .lthn name
    hash, err := lns.ResolveString("example.lthn")
    if err != nil {
        panic(err)
    }
    
    core.Println("Name hash:", hash)
    
    // Run the application
    if err := c.Run(); err != nil {
        panic(err)
    }
}
```

### DNS Resolution

```go
import "dappco.re/go/lns"

// Resolve A record
ip, err := lns.ResolveA("gateway.lthn")

// Resolve all records
records, err := lns.ResolveAll("charon.lthn")

// Reverse DNS (PTR)
name, err := lns.ReverseDNS("10.69.69.165")
```

### Name Registration

```go
import "dappco.re/go/lns"

// Check if name is available
available, err := lns.NameAvailable("myapp.lthn")

// Start registration process (OPEN phase)
err = lns.NameOpen("myapp.lthn", bidAmount, blind)

// Place bid (BID phase)
err = lns.NameBid("myapp.lthn", bidAmount, blind)

// Reveal bid (REVEAL phase)
err = lns.NameReveal("myapp.lthn", bidAmount, blind, reveal)

// Complete registration (REGISTER phase)
err = lns.NameRegister("myapp.lthn", records)
```

---

## Security and validation

### Name Validation Rules

From `pkg/covenant/`:

1. **Label Format:**
   - Lowercase a-z, 0-9, and hyphens only
   - Must start and end with alphanumeric
   - Max length: 63 characters per label
   - Max 255 bytes total

2. **Reserved Names:**
   - `lethean`, `snider`, `darbs`, `charon`, etc. (see catalog)
   - Cannot be registered

3. **Locked Names:**
   - Premium names reserved for special use
   - Require special permissions

### Covenant State Machine

```
                    ┌─────────────────────────────────────────────┐
                    │                                     │
                    ▼                                     ▼
┌──────────┐            ┌──────────┐            ┌──────────┐         ┌──────────┐
│  OPEN    │            │   BID    │            │ REVEAL   │         │ REGISTER │
└──────────┘            └──────────┘            └──────────┘         └──────────┘
     │                       │                       │                     │
     ▼                       ▼                       ▼                     ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                        Auction Period (10 blocks)                       │
│                                                                         │
└────────────────────────────────────────────────────────────────────────────┘
     │                       │                       │                     │
     ▼                       ▼                       ▼                     ▼
                    ┌────────────────────────────────────────┐
                    │              Owned State                │
                    │  (name belongs to bidder/owner)         │
                    └────────────────────────────────────────┘
```

### Validation Example

```go
import "dappco.re/go/lns"

// Validate a name
valid, reason := lns.ValidateName("myapp.lthn")
if !valid {
    return core.E("validation", "invalid name", reason)
}

// Check if name is reserved
if lns.IsReserved("lethean") {
    return core.E("validation", "name is reserved", nil)
}

// Check name state
state := lns.GetNameState("charon.lthn")
if state.State != lns.CovenantRegistered {
    return core.E("validation", "name not registered", nil)
}
```

---

## Architecture

### Package Dependencies

```
lns (top-level)
├── pkg/primitives    # No dependencies (pure types)
├── pkg/covenant     # Depends on: primitives
├── pkg/dns          # Depends on: primitives, covenant
├── pkg/wallet       # Depends on: primitives, hd
├── pkg/chain        # Depends on: primitives, blockstore
├── pkg/blockstore   # Depends on: go-store
├── pkg/net          # Depends on: go-p2p
├── pkg/mempool      # Depends on: primitives
├── pkg/mining       # Depends on: chain, wallet
└── pkg/hd           # No external deps
```

### External Dependencies

| Package | External Dep | Purpose |
|---------|--------------|---------|
| `pkg/blockstore` | `dappco.re/go/store` | SQLite storage |
| `pkg/net` | `dappco.re/go/p2p` | P2P networking |
| `internal/nameutil` | `dappco.re/go/core` | Name utilities |

---

## Testing

### Test Organization

All packages follow the **AX Standard triplet pattern**:

```
pkg/covenant/
├── covenant.go           # Implementation
├── covenant_test.go      # Unit tests
└── covenant_example_test.go  # Example tests
```

### Test Naming Convention

- `TestFoo_Good` — Happy path tests
- `TestFoo_Bad` — Expected error tests
- `TestFoo_Ugly` — Edge cases, panics, unusual inputs

### Running Tests

```bash
# Full test suite
cd ~/Code/core/go-lns
gowork=off go test ./...

# Single package
cd ~/Code/core/go-lns/go/pkg/covenant
go test -v ./...

# With race detector
go test -race ./...

# Coverage
go test -cover ./...
```

### Mocking

- **No real blockchain needed:** Tests use mock data
- **No SSH needed:** Ansible tests use mock SSH client
- **Deterministic:** All tests are deterministic (no randomness)

---

## Documentation references

### RFCs

1. **[Primary RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/lns/RFC.md)** — Canonical specification
2. **[LNS Protocol RFC](file:///Users/snider/Code/core/go-lns/docs/RFC.lns.md)** — Protocol details
3. **[CoreGO RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)** — Framework contract

### Agent Guidance

- **[CLAUDE.md](file:///Users/snider/Code/core/go-lns/CLAUDE.md)** — Claude Code conventions
- **[AGENTS.md](file:///Users/snider/Code/core/go-lns/AGENTS.md)** — Agent-specific rules

### Quick Reference

- **[README.md](file:///Users/snider/Code/core/go-lns/README.md)** — Quick start
- **[docs/](file:///Users/snider/Code/core/go-lns/docs/)** — Detailed documentation

---

## Related packages

| Package | Relationship | Purpose |
|---------|--------------|---------|
| [go-dns](../pkg/dns/README.md) | **Depends on** | DNS resolution for .lthn names |
| [go-hsd](../pkg/hsd/README.md) | **Depends on** | Handshake Sidechain client |
| [go-blockchain](../pkg/blockchain/README.md) | **Sibling** | Main Lethean blockchain |
| [go-p2p](https://knowledge-packs/corego/pkg/p2p/README.md) | **Dependency** | P2P networking layer |
| [go-store](https://knowledge-packs/corego/pkg/store/README.md) | **Dependency** | Storage backend |

### Integration Flow

```
User Query (charon.lthn)
    ↓
Core Action: lns.dns.resolve
    ↓
LNS pkg/dns: Parse name, check cache
    ↓
LNS pkg/covenant: Verify name state
    ↓
LNS pkg/chain: Check blockchain (via go-blockchain)
    ↓
LNS pkg/net: Query peers (via go-p2p)
    ↓
go-hsd: Query HSD sidechain for DNS records
    ↓
Return DNS records (A, AAAA, TXT, etc.)
```

---

## Use cases

### 1. DNS Server for .lthn TLD

```go
c := core.New(
    core.WithService(lns.Register),
    core.WithService(dns.Register),
)
c.Run()

// Now serves DNS queries for .lthn names
// dig charon.lthn @localhost
```

### 2. Name Registration Service

```go
// Register a new .lthn name
bidAmount := 1000000000  // 1 LTHN
blind, reveal := lns.PrepareBid("myapp.lthn", bidAmount)

// OPEN phase
lns.NameOpen("myapp.lthn", bidAmount, blind)

// BID phase (within auction window)
lns.NameBid("myapp.lthn", bidAmount, blind)

// REVEAL phase
lns.NameReveal("myapp.lthn", bidAmount, blind, reveal)

// REGISTER phase
lns.NameRegister("myapp.lthn", lns.NameRecords{
    A: []string{"10.69.69.165"},
    TXT: []string{"v=lthn1 type=gateway"},
})
```

### 3. VPN Gateway Discovery

```go
// Find VPN gateway for a name
gateway, err := lns.LookupGateway("charon.lthn")
if err != nil {
    return err
}

// Connect to gateway
conn, err := vpn.Connect(gateway.Address, gateway.Port)
```

### 4. Name Portfolio Management

```go
// List all names owned by an address
names, err := lns.ListNamesByOwner("LTHN_address_here")

// Get expiration dates
for _, name := range names {
    expiry := lns.GetExpiration(name)
    if time.Until(expiry) < 30*24*time.Hour {
        // Renew before expiry
        lns.NameRenew(name)
    }
}
```

---

## Development notes

### Repository Layout Rules

From AGENTS.md:

1. **Keep changes surgical and package-local**
2. **Preserve SPDX headers** on Go files
3. **Match existing doc comment style**
4. **Reuse existing helpers** (don't reimplement)
5. **Use internal/nameutil** for name canonicalization
6. **Maintain Get* alias pattern** in lns.go

### Build Constraints

```bash
# This repo is NOT in the parent go.work
GOWORK=off go test ./...
GOWORK=off go build ./...
```

### Coding Standards

- **UK English** in comments and identifiers (colour, behaviour, organisation)
- **Conventional commits:** `type(scope): description`
- **Error handling:** Use `core.E(scope, message, err)`
- **Test naming:** Good/Bad/Ugly suffix pattern
- **No direct stdlib:** Use `dappco.re/go` wrappers

### Validation Commands

```bash
# Full validation
gowork=off go test ./...
gowork=off go vet ./...
gowork=off go mod tidy

# Check for changes
git diff --stat
```

---

## References

- **Primary RFC:** [plans/code/core/go/lns/RFC.md](../../../../../plans/code/core/go/lns/RFC.md)
- **Source Code:** [~/Code/core/go-lns/](file:///Users/snider/Code/core/go-lns/)
- **Module:** `dappco.re/go/lns`
- **Lethean Docs:** [https://core.help](https://core.help)

---

*Knowledge pack: go-lns v1.0.0*
*Last updated: 2026-06-17*