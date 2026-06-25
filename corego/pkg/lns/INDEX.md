---
type: Package Index
package: lns
title: go-lns Package Documentation Index
---
# go-lns Package Documentation

**Module:** `dappco.re/go/lns`
**Repository:** `core/go-lns`
**RFC:** [../../../../../plans/code/core/go/lns/RFC.md](../../../../../plans/code/core/go/lns/RFC.md)

---

## Documentation

| Document | Description |
|----------|-------------|
| [README.md](./README.md) | Complete package overview, API reference, examples |
| [RFC.md](../../../../../plans/code/core/go/lns/RFC.md) | Canonical specification |
| [RFC.lns.md](../../../../../plans/code/core/go-lns/docs/RFC.lns.md) | LNS-specific protocol details |

---

## Subpackages

### Core Subpackages

| Package | Description | Key Files |
|---------|-------------|-----------|
| [pkg/covenant/](./) | Name rules, catalogs, verification | covenant.go, catalog.go, verification.go |
| [pkg/dns/](./) | DNS resolution, encoding | resolver.go, encoding.go, lookup.go |
| [pkg/primitives/](./) | Shared types, serialization | primitives.go, types.go |
| [pkg/wallet/](./) | HD wallet, key management | wallet.go, keys.go |
| [pkg/chain/](./) | Chain sync, validation | chain.go, validator.go |
| [pkg/blockstore/](./) | Block storage (SQLite) | store.go, query.go |
| [pkg/net/](./) | Peer networking | net.go, peer.go |
| [pkg/mempool/](./) | Transaction mempool | mempool.go, fees.go |
| [pkg/mining/](./) | Block production | mining.go, miner.go |
| [pkg/hd/](./) | HD key derivation | hd.go, derivation.go |
| [pkg/script/](./) | Transaction scripting | script.go, vm.go |

### Internal Packages

| Package | Description | Key Files |
|---------|-------------|-----------|
| `internal/nameutil/` | Name canonicalization, validation | canonicalize.go, validate.go |

---

## Quick links

### RFC Actions
- **Name Management:** `lns.name.*` (register, renew, transfer, revoke, lookup)
- **DNS Resolution:** `lns.dns.*` (resolve, reverse, serve, health, discover)
- **Wallet:** `lns.wallet.*` (create, balance, send, receive, history)
- **Chain:** `lns.chain.*` (height, block, sync, status, peers)
- **Peer:** `lns.peer.*` (connect, disconnect, broadcast, sync)
- **Mempool:** `lns.mempool.*` (add, remove, get, fees)

### Key Types

```go
// From pkg/primitives/
type NameHash [32]byte
type NameID [32]byte
type Covenant int  // OPEN, BID, REVEAL, REGISTER, RENEW, REVOKE, TRANSFER, UPDATE

// From pkg/covenant/
type NameState struct {
    Name   string
    Hash   NameHash
    State  Covenant
    Height uint64
    Owner  string
    Value  uint64
}

// From pkg/dns/
type NameRecord struct {
    Name   string
    A      []string
    AAAA   []string
    TXT    []string
    NS     []string
    DS     []string
}
```

---

## Related knowledge packs

| Package | Knowledge Pack | Relationship |
|---------|----------------|--------------|
| go-dns | [../../pkg/dns/](../../pkg/dns/) | Depends on LNS for .lthn resolution |
| go-hsd | [../../pkg/hsd/](../../pkg/hsd/) | HSD sidechain client for DNS records |
| go-blockchain | [../../pkg/blockchain/](../../pkg/blockchain/) | Sibling blockchain implementation |
| go-p2p | [../../pkg/p2p/](../../pkg/p2p/) | P2P networking dependency |
| go-store | [../../pkg/store/](../../pkg/store/) | Storage backend dependency |

---

## Statistics

| Metric | Value |
|--------|-------|
| Total subpackages | 11 |
| Core actions | 30+ |
| Covenant states | 8 |
| DNS record types | 6 |
| Test coverage | Good/Bad/Ugly triplets |

---

*Knowledge pack: go-lns v1.0.0*
*Last updated: 2026-06-17*
