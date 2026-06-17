---
type: Package Documentation
package: blockchain
module: dappco.re/go/blockchain
tier: lib
repo: core/go-blockchain
lang: go
tags:
  - blockchain
  - cryptonote
  - zanofork
  - cgo
  - crypto
  - p2p
  - wallet
---
# go-blockchain — Lethean Blockchain Implementation

> **Native Go implementation of the Lethean blockchain protocol (CryptoNote/Zano-fork)**

**RFC:** [plans/code/core/go/blockchain/RFC.md](../../../../../plans/code/core/go/blockchain/RFC.md)
**Source:** [~/Code/core/go-blockchain/](file:///Users/snider/Code/core/go-blockchain/)
**Module:** `dappco.re/go/blockchain`
**Lineage:** CryptoNote protocol, Zano fork

---

## 🎯 Overview

`go-blockchain` is the **canonical Go implementation** of the Lethean blockchain, a privacy-focused cryptocurrency based on the CryptoNote protocol (originally from Monero, forked from Zano).

### Key Features

- **CryptoNote Protocol** — Ring signatures, stealth addresses, confidential transactions
- **CGo Crypto Bridge** — High-performance C++ crypto library (libcryptonote) with Go bindings
- **Full Node** — Complete blockchain synchronization, validation, and serving
- **Wallet Support** — HD wallets, transaction signing, address generation
- **P2P Network** — Levin protocol-based peer-to-peer networking
- **RPC Interface** — JSON-RPC API for external integration
- **Mining** — Block production with mining pool support
- **TUI Dashboard** — Terminal-based explorer and monitoring

### Architecture Philosophy

1. **Pure Go Core** — All business logic in Go
2. **CGo for Crypto** — Performance-critical cryptography in C++ (ADR-001)
3. **SPOR Compliance** — Single Point Of Responsibility for stdlib wrappers
4. **Zero Dependencies** — No external blockchain dependencies
5. **Test Triplets** — Every package has `_test.go` + `_example_test.go`

### Protocol Details

- **Consensus:** Proof-of-Work (CryptoNight variant)
- **Block Time:** ~2 minutes
- **Difficulty Adjustment:** Every block (based on CryptoNote v2)
- **Transaction Types:** Standard, RingCT (confidential)
- **Address Format:** Base58-encoded public keys with checksum
- **Coin Unit:** LTHN (Lethean), divisible to 12 decimal places

---

## 📦 Package Structure

```
core/go-blockchain/
├── go/                          # Go source (module: dappco.re/go/blockchain)
│   ├── go.mod                    # Module definition + dependencies
│   ├── go.sum                    # Dependency checksums
│   │
│   ├── cmd/                      # Command-line tools
│   │   └── core-chain/           # Main CLI binary
│   │
│   ├── config/                   # Configuration management
│   │   ├── config.go             # Config types and parsing
│   │   ├── defaults.go           # Default configuration values
│   │   └── validation.go         # Config validation
│   │
│   ├── types/                    # Core blockchain types
│   │   ├── block.go              # Block and block header types
│   │   ├── transaction.go        # Transaction types
│   │   ├── hash.go               # Hash and cryptographic types
│   │   └── serialization.go      # Binary serialization
│   │
│   ├── wire/                     # Wire protocol (consensus-critical)
│   │   ├── encoding.go           # Binary encoding/decoding
│   │   ├── varint.go             # Variable-length integer encoding
│   │   └── attachment.go         # Transaction attachment handling
│   │
│   ├── crypto/                   # CGo crypto bridge
│   │   ├── bridge.h              # C API header
│   │   ├── bridge.cpp            # C++ bridge implementation
│   │   ├── crypto.go             # Go wrapper functions
│   │   └── upstream/             # Upstream C++ crypto (Zano commit fa1608cf)
│   │
│   ├── consensus/                # Consensus rules (pure Go)
│   │   ├── rules.go              # Consensus validation rules
│   │   ├── difficulty.go         # Difficulty adjustment algorithm
│   │   └── validation.go         # Block and transaction validation
│   │
│   ├── chain/                    # Blockchain storage and management
│   │   ├── chain.go              # Chain manager
│   │   ├── storage.go            # Block storage interface
│   │   ├── sync.go               # Chain synchronization
│   │   └── reorganization.go      # Chain reorganization logic
│   │
│   ├── p2p/                      # P2P networking (Levin protocol)
│   │   ├── node/                 # Node management
│   │   │   ├── node.go           # Node connection handling
│   │   │   ├── levin/            # Levin protocol implementation
│   │   │   └── ueps/             # UEPS (UDP Extended Protocol Support)
│   │   ├── peer.go               # Peer management
│   │   ├── connection.go         # Connection handling
│   │   └── handshake.go          # Network handshake protocol
│   │
│   ├── rpc/                      # JSON-RPC API
│   │   ├── server.go             # RPC server
│   │   ├── client.go             # RPC client
│   │   └── handlers.go           # RPC request handlers
│   │
│   ├── wallet/                   # Wallet functionality
│   │   ├── wallet.go             # Wallet manager
│   │   ├── keys.go               # Key management
│   │   ├── address.go            # Address generation
│   │   ├── transaction.go        # Transaction creation/signing
│   │   └── hd/                   # HD wallet support
│   │
│   ├── mining/                   # Mining operations
│   │   ├── miner.go              # Miner implementation
│   │   ├── pool.go               # Mining pool client
│   │   └── block_template.go     # Block template generation
│   │
│   ├── difficulty/               # Difficulty calculation
│   │   └── calculator.go         # Difficulty adjustment algorithm
│   │
│   ├── explorer/                 # Blockchain explorer
│   │   └── command.go            # Explorer CLI commands
│   │
│   ├── sync_command.go           # Sync command implementation
│   │
│   ├── sync_loop.go              # Main synchronization loop
│   │
│   ├── tools/                    # Utility tools
│   │   └── ...                   # Various blockchain tools
│   │
│   └── tui/                      # Terminal UI
│       ├── dashboard.go          # TUI dashboard
│       ├── blocks.go             # Block viewer
│       ├── transactions.go       # Transaction viewer
│       └── stats.go              # Statistics display
│
├── crypto/                       # C++ crypto library (cmake build)
│   ├── CMakeLists.txt            # CMake configuration
│   └── upstream/                 # Upstream Zano crypto code
│
├── external/                    # External dependencies (vendored)
├── docs/                        # Documentation
├── .core/                       # Core configuration
├── .forgejo/                    # Forgejo CI configuration
├── .woodpecker.yml               # Woodpecker CI configuration
├── AGENTS.md                    # Agent-specific guidelines
└── README.md                    # Quick start guide
```

---

## 🗂️ Subpackage Catalog

### config/
**Responsibility:** Configuration parsing, validation, and defaults

**Key Types:**
```go
type Config struct {
    Network     NetworkConfig
    Chain      ChainConfig
    P2P        P2PConfig
    Wallet     WalletConfig
    Mining     MiningConfig
    RPC        RPCConfig
    Storage    StorageConfig
}

type NetworkConfig struct {
    Name       string  // mainnet, testnet, devnet
    Magic      uint32  // Network magic number
    Port       int     // P2P port
    RPCPort    int     // RPC port
}

type ChainConfig struct {
    DataDir       string
    PruneMode     PruneMode
    SyncMode      SyncMode
    Checkpoint    *Checkpoint
}
```

**Features:**
- YAML/JSON configuration file support
- Environment variable overrides
- Validation on startup
- Network presets (mainnet, testnet, devnet)

### types/
**Responsibility:** Core blockchain data types

**Key Types:**
```go
// Block represents a blockchain block
type Block struct {
    Header       BlockHeader
    Transactions []*Transaction
    // Attachments for v2+
    Attachments  []*TransactionAttachment
    Proofs       []*TransactionProof
}

// BlockHeader contains block metadata
type BlockHeader struct {
    Version           uint8
    PreviousHash      [32]byte  // Hash of previous block
    MerkleRoot        [32]byte  // Merkle root of transactions
    Timestamp         uint64   // Unix timestamp
    Height            uint64   // Block height
    Nonce             [4]byte   // Mining nonce
    Difficulty        uint64   // Block difficulty
    Reward           uint64   // Block reward
    BaseReward        uint64   // Base reward (for calculation)
    AlreadyGeneratedCoins uint64
}

// Transaction represents a blockchain transaction
type Transaction struct {
    Version         uint8
    Inputs          []*TransactionInput
    Outputs         []*TransactionOutput
    Mixins          []*TransactionMixin  // For ring signatures
    Signatures      [][]byte              // Ring signatures
    Extra           []byte                // Additional data
    // v2+ fields
    Attachment      *TransactionAttachment
    Proof           *TransactionProof
}

// TransactionInput references previous outputs
type TransactionInput struct {
    KeyImage     [32]byte  // Key image (prevents double-spending)
    Amount       uint64   // Input amount
    OutputIndex  uint64   // Index in previous transaction
    OutputHash   [32]byte  // Hash of previous transaction
}

// TransactionOutput contains output data
type TransactionOutput struct {
    PublicKey   [33]byte  // Public key (compressed)
    Amount     uint64   // Output amount
    Index      uint64   // Output index
}

// Hash types
type BlockHash [32]byte
type TransactionHash [32]byte
type KeyImage [32]byte
```

**Features:**
- Binary serialization (wire format)
- JSON serialization
- Hash calculation
- Type validation
- Immutable by design

### wire/
**Responsibility:** Wire protocol encoding/decoding (consensus-critical)

**Key Features:**
- **Binary Encoding:** Efficient binary serialization for network transmission
- **Varint Support:** Two varint formats (CryptoNote LEB128, Portable Storage 2-bit)
- **Attachment Handling:** Opaque raw bytes for unknown attachment types
- **Version-Aware:** Handles different transaction versions (v0, v1, v2+)

**Key Functions:**
```go
// Encode block to binary
func EncodeBlock(b *Block) ([]byte, error)

// Decode block from binary
func DecodeBlock(data []byte) (*Block, error)

// Encode/Decode transactions
func EncodeTransaction(tx *Transaction) ([]byte, error)
func DecodeTransaction(data []byte) (*Transaction, error)

// Varint encoding/decoding
func EncodeVarint(v uint64) []byte
func DecodeVarint(data []byte) (uint64, int, error)
```

**Design Note:** `wire/` stores extra fields as opaque bytes to ensure bit-identical round-tripping without implementing all variant types.

### crypto/
**Responsibility:** CGo cryptographic operations (ADR-001)

**Architecture:**
- **Go Shell:** Thin Go wrapper (`crypto.go`)
- **C++ Library:** Performance-critical crypto (`libcryptonote.a`)
- **C API:** 29-function boundary (`bridge.h`)
- **Pointer Passing:** Only `uint8_t*` crosses the boundary

**CGo Bridge (bridge.h):**
```c
// Hash functions
extern void cn_hash_fast(const uint8_t *data, size_t length, uint8_t *hash);
extern void cn_keccak256(const uint8_t *data, size_t length, uint8_t *hash);

// Key operations
extern void cn_generate_keys(uint8_t *public_key, uint8_t *private_key);
extern void cn_generate_key_derivation(uint8_t *public_key, uint8_t *derivation);

// Ring signature operations
extern int cn_generate_ring_signature(...);
extern int cn_check_ring_signature(...);

// Bulletproof operations (for RingCT)
extern int cn_bulletproof_generate(...);
extern int cn_bulletproof_verify(...);
```

**Key Types:**
```go
type PublicKey [33]byte  // Compressed Ed25519 public key
type PrivateKey [32]byte  // Ed25519 private key
type KeyPair struct {
    Public  PublicKey
    Private PrivateKey
}
type KeyImage [32]byte  // Key image for double-spend prevention
type Signature [64]byte  // Ed25519 signature
```

**Build Process:**
```bash
# Build C++ crypto library
cmake -S crypto -B crypto/build -DCMAKE_BUILD_TYPE=Release
cmake --build crypto/build --parallel

# Then build Go code (requires libcryptonote.a)
go build ./...
```

**Upstream:** Zano commit `fa1608cf` in `crypto/upstream/`

### consensus/
**Responsibility:** Consensus rules and validation (PURE GO - NO STORAGE DEPENDENCY)

**Design Principle:** `consensus/` is **pure** — it takes types + config + height and returns errors. No dependency on `chain/` or storage.

**Key Functions:**
```go
// Validate a block against consensus rules
func ValidateBlock(block *Block, previous *Block, height uint64) error

// Validate a transaction
func ValidateTransaction(tx *Transaction, block *Block, height uint64) error

// Check if transaction inputs are valid (not double-spent)
func CheckInputs(inputs []*TransactionInput, outputs map[[32]byte][]*TransactionOutput) error

// Calculate block reward
func CalculateReward(height uint64, previousBlock *Block) uint64
```

**Consensus Rules:**
- Block version validation
- Previous hash validation
- Timestamp validation (±2 hours from network time)
- Difficulty validation
- Merkle root validation
- Transaction validation
- Input/output balance validation
- Ring signature validation
- Key image uniqueness (no double-spend)

### difficulty/
**Responsibility:** Difficulty adjustment algorithm

**Algorithm:** CryptoNote v2 (adjusts every block)

**Key Types:**
```go
type DifficultyCalculator struct {
    // Configuration
    TargetBlockTime uint64  // 120 seconds for Lethean
    MaxAdjustment   float64 // Maximum difficulty adjustment per block
}

func (dc *DifficultyCalculator) CalculateNext(
    currentDifficulty uint64,
    timestamps []uint64,
    height uint64,
) uint64
```

**Formula:**
```
new_difficulty = current_difficulty * (actual_time / target_time)
// Clamped to max adjustment factor
```

### chain/
**Responsibility:** Blockchain storage, synchronization, and reorganization

**Key Types:**
```go
type Chain struct {
    Config     *ChainConfig
    Store      ChainStore
    CurrentTip *Block
    Height     uint64
}

type ChainStore interface {
    GetBlock(hash [32]byte) (*Block, error)
    GetBlockByHeight(height uint64) (*Block, error)
    GetBlockHash(height uint64) ([32]byte, error)
    GetTransaction(hash [32]byte) (*Transaction, error)
    PutBlock(block *Block) error
    DeleteBlock(hash [32]byte) error
    GetHeight() uint64
}
```

**Features:**
- **Storage Backends:** SQLite (via go-store), LevelDB, Memory
- **Synchronization:** Full sync, fast sync, catch-up sync
- **Reorganization:** Automatic chain reorganization on fork detection
- **Pruning:** Block pruning modes (none, light, aggressive)
- **Checkpoints:** Periodic checkpoints for faster sync

**Sync Modes:**
1. **Full Sync:** Download and validate all blocks from genesis
2. **Fast Sync:** Download blocks, validate headers only, request transactions as needed
3. **Catch-up Sync:** Trust checkpoints, validate from there

### p2p/
**Responsibility:** Peer-to-peer networking (Levin protocol)

**Levin Protocol:** Binary protocol used by CryptoNote-based currencies

**Subpackages:**

#### p2p/node/
**Responsibility:** Node connection management

##### p2p/node/levin/
**Responsibility:** Levin protocol implementation

**Key Features:**
- Connection handshake
- Command dispatch
- Protocol version negotiation
- Keep-alive management
- Ban list management

##### p2p/node/ueps/
**Responsibility:** UDP Extended Protocol Support

**Features:**
- UDP-based peer discovery
- NAT traversal support
- Connection multiplexing

**Key Types:**
```go
type Node struct {
    Address     string
    Connection  net.Conn
    PeerID     [32]byte
    Version    uint32
    Height     uint64
    LastSeen   time.Time
}

type PeerManager struct {
    Nodes       map[[32]byte]*Node
    Banned      map[string]time.Time  // IP -> ban expiry
    Whitelisted map[string]bool       // IP -> whitelisted
}
```

**Commands:**
```
Handshake (0x0001) - Initial connection
Ping (0x0002) - Keep-alive
GetPeers (0x0003) - Request peer list
NewBlock (0x0004) - Announce new block
NewTransaction (0x0005) - Announce new transaction
GetBlock (0x0006) - Request block by hash
GetTransaction (0x0007) - Request transaction by hash
```

### rpc/
**Responsibility:** JSON-RPC API server and client

**Key Types:**
```go
type RPCServer struct {
    Chain   *Chain
    Wallet  *WalletManager
    Pool    *MiningPool
    Config  *RPCConfig
}

type RPCConfig struct {
    Enabled  bool
    BindAddr string
    Port     int
    Auth     *RPCAuth
}

type RPCAuth struct {
    Username string
    Password string
}
```

**RPC Methods:**

| Method | Description | Parameters |
|--------|-------------|------------|
| `get_block_count` | Get current block count | - |
| `get_block_hash` | Get block hash by height | height |
| `get_block` | Get full block by hash/height | hash/height |
| `get_last_block_header` | Get latest block header | - |
| `get_transaction` | Get transaction by hash | tx_hash |
| `send_raw_transaction` | Send raw transaction | tx_blob |
| `create_wallet` | Create new wallet | filename, password |
| `open_wallet` | Open existing wallet | filename, password |
| `get_balance` | Get wallet balance | address |
| `get_address` | Get wallet address | wallet_file, address_index |
| `transfer` | Send LTHN | wallet_file, destinations, fee, mixin |
| `start_mining` | Start mining | threads, address |
| `stop_mining` | Stop mining | - |

### wallet/
**Responsibility:** Wallet management, key generation, transaction signing

**Key Types:**
```go
type Wallet struct {
    FilePath   string
    Password   []byte  // Encrypted in memory
    Seed      [32]byte // Encrypted on disk
    Keys      []*KeyPair
    Addresses map[uint64]*Address  // index -> address
}

type Address struct {
    PublicKey PublicKey
    Index     uint64
    Label     string
}

type Account struct {
    AddressIndex uint64
    Addresses    []*Address
    Balance     uint64
}
```

**Features:**
- **HD Wallet:** Hierarchical Deterministic wallets (BIP-32 compatible)
- **Multiple Accounts:** Support for multiple spending keys
- **Address Book:** Named addresses
- **Transaction History:** Full transaction history
- **Offline Signing:** Sign transactions without network access
- **Encryption:** AES-256 encryption for wallet files

**Key Management:**
```go
// Generate new key pair
func GenerateKeys() (*KeyPair, error)

// Generate key from seed
func KeyFromSeed(seed [32]byte, index uint64) *KeyPair

// Generate key derivation for stealth addresses
func GenerateKeyDerivation(publicKey *PublicKey) *KeyDerivation

// Derive subaddress key
func DeriveSubaddress(publicKey *PublicKey, derivation *KeyDerivation, index uint64) *PublicKey
```

### wallet/hd/
**Responsibility:** HD (Hierarchical Deterministic) wallet support

**Features:**
- BIP-32 compatible derivation
- Hardened/non-hardened derivation
- Multiple derivation paths
- Seed phrase generation (13, 24, 25 words)

### mining/
**Responsibility:** Mining operations and pool management

**Key Types:**
```go
type Miner struct {
    Chain      *Chain
    Wallet     *Wallet
    Pool       *MiningPool
    Threads    int
    Running    bool
}

type MiningPool struct {
    URL        string
    Username   string
    Password   string
    Difficulty uint64
}

type BlockTemplate struct {
    PreviousHash [32]byte
    Height       uint64
    Difficulty   uint64
    Reward       uint64
    Timestamp    uint64
    Reserved     [4]byte
    // Plus coinbase transaction
}
```

**Features:**
- **Solo Mining:** Mine directly to your wallet
- **Pool Mining:** Mine to a mining pool
- **Block Template:** Generate and solve block templates
- **Mining Proxy:** Proxy mining connections
- **Stratum Protocol:** Support for Stratum mining protocol

### tui/
**Responsibility:** Terminal User Interface (TUI) dashboard

**Built with:** Charmbracelet BubbleTea

**Features:**
- **Dashboard:** Real-time blockchain statistics
- **Block Viewer:** Browse blocks and transactions
- **Transaction Viewer:** Inspect transaction details
- **Wallet Viewer:** View wallet balances and history
- **Peer Viewer:** Monitor network connections
- **Mining Viewer:** Monitor mining status

**Screens:**
```
┌─────────────────────────────────────────────────────────────┐
│  Lethean Blockchain Dashboard                            [q]uit │
├─────────────────────────────────────────────────────────────┤
│  Height: 1,234,567    │  Difficulty: 45,678,901    │  Hashrate: 1.2 GH/s │
│  Timestamp: 2026-06-17 15:30:45                          │  Peers: 42        │
├─────────────────────────────────────────────────────────────┤
│  Recent Blocks:                                               │
│  ┌──────────┬─────────────┬─────────────┬────────────┐    │
│  │ 1234567  │ 00:00:05 ago │ 12 txs      │ 1.234 LTHN  │    │
│  │ 1234566  │ 00:02:15 ago │ 8 txs       │ 0.567 LTHN  │    │
│  │ 1234565  │ 00:04:30 ago │ 15 txs      │ 2.345 LTHN  │    │
│  └──────────┴─────────────┴─────────────┴────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 Quick Start

### Basic Setup

```go
package main

import (
    "context"
    "log"
    
    blockchain "dappco.re/go/blockchain"
    "dappco.re/go/blockchain/config"
    "dappco.re/go/blockchain/chain"
)

func main() {
    // Load configuration
    cfg, err := config.Load("config.yml")
    if err != nil {
        log.Fatal(err)
    }
    
    // Create and start chain
    ch, err := chain.New(cfg)
    if err != nil {
        log.Fatal(err)
    }
    
    // Start synchronization
    ctx := context.Background()
    if err := ch.Sync(ctx); err != nil {
        log.Fatal(err)
    }
    
    // Wait for sync to complete
    <-ch.SyncDone()
    
    log.Println("Blockchain synced to height:", ch.Height())
}
```

### Using the CLI

```bash
# Build the CLI
cd ~/Code/core/go-blockchain
go build ./cmd/core-chain

# Sync the blockchain
./core-chain chain sync

# Run the TUI dashboard
./core-chain chain explorer

# Start the RPC server
./core-chain chain rpc --rpc-bind 127.0.0.1:46941

# Get blockchain info
curl http://127.0.0.1:46941/json_rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":"1","method":"get_block_count","params":{}}'
```

### Creating a Wallet

```go
import (
    "dappco.re/go/blockchain/wallet"
)

// Create a new wallet
w, err := wallet.Create("mywallet.keys", "secure-password")
if err != nil {
    panic(err)
}

// Get the primary address
addr := w.GetPrimaryAddress()
println("Wallet address:", addr.String())

// Save the wallet (auto-saves on changes)
if err := w.Save(); err != nil {
    panic(err)
}

// Open an existing wallet
w, err := wallet.Open("mywallet.keys", "secure-password")
```

### Sending a Transaction

```go
import (
    "dappco.re/go/blockchain/wallet"
    "dappco.re/go/blockchain/types"
)

// Open wallet
w, _ := wallet.Open("mywallet.keys", "secure-password")

// Create destination
Destination := wallet.Destination{
    Address: "LTHN_address_here",
    Amount:  1000000000, // 1 LTHN (in atomic units)
}

// Send transaction
txHash, err := w.Transfer([]wallet.Destination{Destination}, 10000, 3)
// ^^ 10000 = fee (atomic units), 3 = mixin count
if err != nil {
    panic(err)
}

fmt.Println("Transaction hash:", txHash)
```

### Mining

```go
import (
    "dappco.re/go/blockchain/mining"
    "dappco.re/go/blockchain/wallet"
    "dappco.re/go/blockchain/chain"
)

// Setup
w, _ := wallet.Open("mywallet.keys", "password")
ch, _ := chain.New(cfg)

// Create miner
miner := mining.NewMiner(ch, w)

// Start solo mining
if err := miner.Start(4); err != nil {  // 4 threads
    panic(err)
}

// Later, stop mining
miner.Stop()
```

---

## 🔐 Cryptographic Details

### Key Types

| Type | Size | Description |
|------|------|-------------|
| PublicKey | 33 bytes | Compressed Ed25519 public key |
| PrivateKey | 32 bytes | Ed25519 private key |
| KeyImage | 32 bytes | P = H_p(private_key) * G (prevents double-spend) |
| Signature | 64 bytes | Ed25519 signature |
| BlockHash | 32 bytes | Keccak256 hash |
| TransactionHash | 32 bytes | Keccak256 hash |

### Hash Functions

1. **Keccak256** — Primary hash function (SHA-3)
   - Used for: Block hashes, transaction hashes, key derivation
   - Note: On-chain curve points are stored **premultiplied by cofactor inverse (1/8)**

2. **Blake2b** — Alternative hash for some operations

### Signature Scheme

- **Ed25519** — Primary signature scheme
- **Ring Signatures** — For transaction signing (mixin count determines ring size)
  - Signer is anonymous among ring members
  - Provably unlinkable
  - Computationally efficient

### Confidential Transactions (RingCT)

- **Pedersen Commitments** — Hide transaction amounts
- **Range Proofs** — Prove amount is in valid range (0-2^64)
- **Bulletproofs** — Compact zero-knowledge proofs

---

## 🏗️ Architecture Deep Dive

### Data Flow

```
Network Layer (p2p/)
     │
     ▼
┌─────────────────────┐
│  Protocol Layer      │  ← Levin protocol
│  - Handshake         │
│  - Command dispatch  │
│  - Connection mgmt   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Message Layer       │  ← Encoded blocks/transactions
│  - Serialization      │
│  - Deserialization    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Consensus Layer     │  ← Validation (pure Go)
│  - Block validation  │
│  - Transaction check  │
│  - State transitions │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Storage Layer       │  ← Chain storage
│  - Block storage      │
│  - Transaction index  │
│  - UTXO set           │
└─────────────────────┘
```

### Package Dependency Graph

```
core/go (framework)
├── config/        # No external deps
├── types/         # No external deps
├── wire/          # Depends on: types
├── difficulty/    # Depends on: types
├── consensus/     # Depends on: types, wire, difficulty
├── crypto/        # CGo, depends on upstream C++
├── chain/         # Depends on: config, types, consensus, crypto, store
├── p2p/           # Depends on: config, types
│   └── node/      # Depends on: p2p
│       ├── levin/ # Depends on: node
│       └── ueps/  # Depends on: node
├── rpc/           # Depends on: chain, wallet, p2p
├── wallet/        # Depends on: config, types, crypto
│   └── hd/        # Depends on: wallet
└── mining/        # Depends on: chain, wallet, p2p
    └── pool/       # Depends on: mining
```

### External Dependencies

| Dependency | Purpose | Version |
|-------------|---------|---------|
| github.com/charmbracelet/bubbletea | TUI framework | v1.3.10 |
| github.com/spf13/cobra | CLI framework | v1.10.2 |
| golang.org/x/crypto | Crypto utilities | v0.50.0 |
| modernc.org/sqlite | SQLite library | v1.47.0 |
| C++ upstream (Zano) | Crypto operations | fa1608cf |

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| **Total Go Files** | 200+ |
| **Subpackages** | 12 (config, types, wire, crypto, consensus, difficulty, chain, p2p, rpc, wallet, mining, tui) |
| **C++ Files** | 50+ (upstream Zano crypto) |
| **CLOC (Go)** | 50,000+ |
| **CLOC (C++)** | 20,000+ |
| **Test Files** | 100+ (Good/Bad/Ugly triplets) |
| **Test Coverage** | High |
| **RPC Methods** | 50+ |
| **P2P Commands** | 20+ |

---

## 🔍 Testing

### Test Organization

All packages follow the **AX Standard triplet pattern**:

```
config/
├── config.go
├── config_test.go          # Good/Bad/Ugly tests
└── config_example_test.go # Example usage

chain/
├── chain.go
├── chain_test.go
└── chain_example_test.go
```

### Test Categories

1. **Unit Tests** — Test individual functions in isolation
2. **Integration Tests** — Test component interactions (build tag: `integration`)
3. **Network Tests** — Test P2P networking (requires testnet daemon)
4. **Performance Tests** — Benchmark critical operations

### Running Tests

```bash
# All tests
go test ./...

# With race detector (required before commit)
go test -race ./...

# Single package
go test ./config/...

# Single test
go test -run TestValidateBlock_Good ./consensus/...

# Integration tests (need C++ testnet daemon on localhost:46941)
go test -tags integration ./...

# Vertex checking (must pass before commit)
go vet ./...

# Module tidy (must produce no changes before commit)
go mod tidy
```

### Test Naming Convention

- `TestFoo_Good` — Happy path, valid inputs
- `TestFoo_Bad` — Expected errors, invalid inputs
- `TestFoo_Ugly` — Edge cases, panics, unusual conditions

---

## 🎯 Use Cases

### 1. Full Node

```go
// Run a full node with RPC and P2P
cfg := config.DefaultMainnet()
cfg.P2P.Enabled = true
cfg.RPC.Enabled = true
cfg.RPC.BindAddr = "0.0.0.0:46941"

chain, _ := chain.New(cfg)
chain.Start()

// Wait for shutdown signal
select {}
```

### 2. Light Client

```go
// Run a light client (no P2P, connects to remote node)
cfg := config.DefaultMainnet()
cfg.P2P.Enabled = false
cfg.RPC.Enabled = false

chain, _ := chain.New(cfg)
// Connect to remote node
chain.ConnectTo("remote-node.lthn.io:46941")

// Sync from remote
chain.Sync(context.Background())
```

### 3. Transaction Signer (Offline)

```go
// Create and sign transactions without network
w, _ := wallet.Open("offline-wallet.keys", "password")

// Create unsigned transaction
tx := types.NewTransaction(inputs, outputs)

// Sign with wallet
signedTx, _ := w.SignTransaction(tx)

// Export raw transaction
rawTx := signedTx.Serialize()

// Later, broadcast via online node
rpcClient := rpc.NewClient("http://node.lthn.io:46941")
rpcClient.SendRawTransaction(rawTx)
```

### 4. Block Explorer Backend

```go
// Query blockchain data for explorer
explorer := NewExplorer(chain)

// Get block by height
block, _ := explorer.GetBlockByHeight(1234567)

// Get transaction by hash
tx, _ := explorer.GetTransaction(txHash)

// Get address balance and history
balance, history, _ := explorer.GetAddressInfo(address)

// Search for transactions
results, _ := explorer.SearchTransactions(query)
```

### 5. Mining Pool

```go
// Create mining pool
pool := mining.NewPool(cfg)

// Start pool server
pool.Start(":5555")

// Accept connections from miners
pool.AcceptConnections()

// Generate and distribute block templates
template, _ := pool.GenerateTemplate()
pool.BroadcastTemplate(template)

// Validate and submit shares
pool.ValidateShare(miner, share)
```

---

## 🛠️ Development Notes

### Build Requirements

**C Toolchain:**
- cmake (3.10+)
- g++ or clang (C++17+)
- libssl-dev
- libboost-dev

**Go:**
- Go 1.26+
- CGO_ENABLED=1

### Building

```bash
# Clone repository
cd ~/Code/core/go-blockchain

# Build C++ crypto library
cmake -S crypto -B crypto/build -DCMAKE_BUILD_TYPE=Release
cmake --build crypto/build --parallel $(nproc)

# Build Go code
go build ./...

# Build CLI
cd cmd/core-chain
go build .
```

### Repository Layout Rules

From AGENTS.md:

1. **Pure Go packages first:** `config/`, `types/`, `wire/`, `difficulty/` build without C toolchain
2. **CGo packages:** `crypto/`, `consensus/`, `chain/`, `wallet/`, `mining/` require `libcryptonote.a`
3. **Keep consensus pure:** No storage dependency in `consensus/`
4. **Preserve SPDX headers** on all source files
5. **Match existing style** for doc comments
6. **Use dappco.re/go wrappers** instead of direct stdlib

### Coding Standards

- **UK English** in identifiers and comments (colour, behaviour, organisation)
- **Conventional Commits:** `type(scope): description`
- **Co-Author Trailer:** `Co-Authored-By: <name> <email>`
- **Error Format:** `package: description` (e.g., `types: invalid hex for hash`)
- **Error Wrapping:** `fmt.Errorf("package: description: %w", err)`
- **Import Order:** stdlib → golang.org/x → dappco.re → third-party → blank line between groups
- **No Emojis** in code or comments

### Block and Transaction Wire Format

**Block Wire Format:**
```
[4 bytes: Version]
[32 bytes: Previous Hash]
[32 bytes: Merkle Root]
[8 bytes: Timestamp]
[8 bytes: Height]
[4 bytes: Nonce]
[8 bytes: Difficulty]
[8 bytes: Reward]
[8 bytes: Base Reward]
[8 bytes: Already Generated Coins]
[Variable: Transaction Count (varint)]
[Variable: Transactions...]
```

**Transaction Wire Format (v1):**
```
[1 byte: Version]
[Variable: Input Count (varint)]
[Variable: Inputs...]
[Variable: Output Count (varint)]
[Variable: Outputs...]
[Variable: Mixin Count (varint)]
[Variable: Mixins...]
[Variable: Signature Count (varint)]
[Variable: Signatures...]
[Variable: Extra]
```

**Transaction Wire Format (v2+):**
```
[1 byte: Version]
[Variable: Input Count (varint)]
[Variable: Inputs...]
[Variable: Extra]
[Variable: Output Count (varint)]
[Variable: Outputs...]
[4 bytes: Hardfork ID]
[Variable: Attachment]
[Variable: Signatures]
[Variable: Proofs]
```

---

## 📖 Documentation References

### RFCs
1. **[Primary RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/blockchain/RFC.md)** — Canonical specification
2. **[CoreGO RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)** — Framework contract
3. **[Subpackage RFCs](file:///Users/snider/Code/meowmix/plans/code/core/go/blockchain/)** — Chain, wallet, p2p, etc.

### Agent Guidance
- **[CLAUDE.md](file:///Users/snider/Code/core/go-blockchain/CLAUDE.md)** — Build requirements, test conventions
- **[AGENTS.md](file:///Users/snider/Code/core/go-blockchain/AGENTS.md)** — Agent-specific rules

### Quick Reference
- **[README.md](file:///Users/snider/Code/core/go-blockchain/README.md)** — Quick start
- **[docs/](file:///Users/snider/Code/core/go-blockchain/docs/)** — Detailed documentation

---

## 🔗 Related Packages

| Package | Relationship | Purpose |
|---------|--------------|---------|
| [go-lns](../pkg/lns/README.md) | **Sibling** | Lethean Name System (uses blockchain) |
| [go-p2p](../pkg/p2p/README.md) | **Dependency** | P2P networking layer |
| [go-store](https://knowledge-packs/corego/pkg/store/README.md) | **Dependency** | Storage backend |
| [go-miner](../pkg/miner/README.md) | **Related** | Mining operations |
| [go-wallet](https://knowledge-packs/corego/pkg/wallet/README.md) | **Related** | Wallet management |

### Integration Flow

```
User → RPC Client → go-blockchain (RPC Server)
    ↓
Chain Manager → Consensus → Storage
    ↓
P2P Layer → Network → Other Nodes
    ↓
Wallet → Transaction Signing → Mining
```

---

## 🌍 Lethean Ecosystem

| Component | Description | Status |
|-----------|-------------|--------|
| **Blockchain** | Core consensus and transaction processing | ✅ Production |
| **LNS** | Lethean Name System (.lthn TLD) | ✅ Production |
| **VPN** | Decentralized VPN network | ✅ Production |
| **Wallet** | CLI and GUI wallets | ✅ Production |
| **Explorer** | Blockchain explorer | ✅ Production |
| **Mining Pool** | Community mining pools | ✅ Active |

**Networks:**
- `mainnet` — Production network (port 46941)
- `testnet` — Test network (port 56941)
- `devnet` — Development network (port 66941)

---

## 📞 Support

- **Primary RFC:** [plans/code/core/go/blockchain/RFC.md](../../../../../plans/code/core/go/blockchain/RFC.md)
- **Source Code:** [~/Code/core/go-blockchain/](file:///Users/snider/Code/core/go-blockchain/)
- **Module:** `dappco.re/go/blockchain`
- **Lethean Docs:** [https://core.help](https://core.help)
- **Lethean Website:** [https://lthn.io](https://lthn.io)

---

*Knowledge Pack: go-blockchain v1.0.0*
*Last Updated: 2026-06-17*
*Author: Purberus <purberus@lthn.ai>*
*Generated by Mistral Vibe. Co-Authored-By: Mistral Vibe <vibe@mistral.ai>*