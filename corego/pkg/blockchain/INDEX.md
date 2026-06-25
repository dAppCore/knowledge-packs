---
type: Package Index
package: blockchain
title: go-blockchain Package Documentation Index
---
# go-blockchain Package Documentation

> **Lethean Blockchain** — CryptoNote/Zano-fork implementation in Go

**Module:** `dappco.re/go/blockchain`
**Repository:** `core/go-blockchain`
**RFC:** [../../../../../plans/code/core/go/blockchain/RFC.md](../../../../../plans/code/core/go/blockchain/RFC.md)

---

## Documentation

| Document | Description |
|----------|-------------|
| [README.md](./README.md) | Complete package overview, architecture, use cases |
| [RFC.md](../../../../../plans/code/core/go/blockchain/RFC.md) | Canonical specification |
| [CLAUDE.md](file:///Users/snider/Code/core/go-blockchain/CLAUDE.md) | Build requirements, test conventions |
| [AGENTS.md](file:///Users/snider/Code/core/go-blockchain/AGENTS.md) | Agent-specific guidelines |

---

## Subpackage catalog

### Core Packages

| Package | Description | Key Files | Dependencies |
|---------|-------------|-----------|--------------|
| [config/](./) | Configuration parsing and validation | config.go, defaults.go, validation.go | None |
| [types/](./) | Core blockchain data types | block.go, transaction.go, hash.go | None |
| [wire/](./) | Wire protocol encoding/decoding | encoding.go, varint.go | types |
| [difficulty/](./) | Difficulty adjustment algorithm | calculator.go | types |

### Consensus & Crypto

| Package | Description | Key Files | Dependencies |
|---------|-------------|-----------|--------------|
| [crypto/](./) | CGo crypto bridge (ADR-001) | bridge.h, bridge.cpp, crypto.go | C++ upstream |
| [consensus/](./) | Consensus rules (PURE GO) | rules.go, validation.go | types, wire, difficulty |

### Chain Management

| Package | Description | Key Files | Dependencies |
|---------|-------------|-----------|--------------|
| [chain/](./) | Blockchain storage and sync | chain.go, storage.go, sync.go | config, types, consensus, crypto |

### Networking

| Package | Description | Key Files | Dependencies |
|---------|-------------|-----------|--------------|
| [p2p/](./) | P2P networking | peer.go, connection.go, handshake.go | config, types |
| [p2p/node/](./) | Node management | node.go | p2p |
| [p2p/node/levin/](./) | Levin protocol | protocol.go, commands.go | node |
| [p2p/node/ueps/](./) | UDP Extended Protocol | ueps.go | node |

### RPC & API

| Package | Description | Key Files | Dependencies |
|---------|-------------|-----------|--------------|
| [rpc/](./) | JSON-RPC API | server.go, client.go, handlers.go | chain, wallet, p2p |

### Wallet

| Package | Description | Key Files | Dependencies |
|---------|-------------|-----------|--------------|
| [wallet/](./) | Wallet management | wallet.go, keys.go, address.go, transaction.go | config, types, crypto |
| [wallet/hd/](./) | HD wallet support | hd.go, derivation.go | wallet |

### Mining

| Package | Description | Key Files | Dependencies |
|---------|-------------|-----------|--------------|
| [mining/](./) | Mining operations | miner.go, pool.go, block_template.go | chain, wallet, p2p |

### User Interface

| Package | Description | Key Files | Dependencies |
|---------|-------------|-----------|--------------|
| [tui/](./) | Terminal UI dashboard | dashboard.go, blocks.go, transactions.go | chain, wallet |
| [cmd/core-chain/](./) | CLI commands | main.go, sync_command.go, explorer_command.go | All packages |

---

## Quick links

### Key Types

```go
// Core types
type Block struct { Header BlockHeader; Transactions []*Transaction }
type BlockHeader struct { Version uint8; PreviousHash [32]byte; MerkleRoot [32]byte; ... }
type Transaction struct { Version uint8; Inputs []*Input; Outputs []*Output; ... }

// Crypto types
type PublicKey [33]byte
type PrivateKey [32]byte
type KeyImage [32]byte
type Signature [64]byte

// Hash types
type BlockHash [32]byte
type TransactionHash [32]byte
```

### Network Ports

| Network | P2P Port | RPC Port |
|---------|----------|----------|
| mainnet | 46940 | 46941 |
| testnet | 56940 | 56941 |
| devnet | 66940 | 66941 |

### RPC Methods

**Blockchain:**
- `get_block_count` — Get current block height
- `get_block_hash` — Get block hash by height
- `get_block` — Get full block
- `get_last_block_header` — Get latest block header

**Transaction:**
- `get_transaction` — Get transaction by hash
- `send_raw_transaction` — Broadcast transaction

**Wallet:**
- `create_wallet` — Create new wallet
- `open_wallet` — Open existing wallet
- `get_balance` — Get wallet balance
- `get_address` — Get wallet address
- `transfer` — Send LTHN

**Mining:**
- `start_mining` — Start mining
- `stop_mining` — Stop mining

---

## Related knowledge packs

| Package | Knowledge Pack | Relationship |
|---------|----------------|--------------|
| go-lns | [../pkg/lns/](../pkg/lns/) | **Sibling** — Uses blockchain for LNS operations |
| go-p2p | [../pkg/p2p/](../pkg/p2p/) | **Dependency** — P2P networking layer |
| go-store | [../pkg/store/](../pkg/store/) | **Dependency** — Storage backend |

---

