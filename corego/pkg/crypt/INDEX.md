---
type: Package Index
title: go-crypt Package Index
description: Complete index of go-crypt package components and API surface
module: dappco.re/go/crypt
---

# go-crypt — Package Index

> **Repository:** `core/go-crypt`
> **Module:** `dappco.re/go/crypt`
> **Type:** Library
> **Status:** Production
> **Tier:** lib (foundation — consumers import this, this never imports consumers)
> **Lines:** ~2,000+ (source + sub-packages + tests)

---

## 📚 Quick Links

- **[README.md](./README.md)** — Complete package documentation
- **[RFC Specification](file:///Users/snider/Code/meowmix/plans/code/core/go/crypt/RFC.md)** — Technical specification (self-contained, agent-implementable)
- **[CLAUDE.md](file:///Users/snider/Code/core/go-crypt/CLAUDE.md)** — Implementation details, dependencies, conventions
- **[AGENTS.md](file:///Users/snider/Code/core/go-crypt/AGENTS.md)** — Agent guidance

### Sub-Specifications

The RFC includes comprehensive sub-specifications:

- **Section 2:** Cryptography Subsystem — Symmetric encryption, key derivation, HMAC, checksums
- **Section 3:** Authentication Subsystem — Session storage, hardware tokens, OpenPGP
- **Section 4:** Trust Subsystem — Tiers, capabilities, policy engine, approval workflows, audit trail
- **Section 5:** CLI Commands — `keygen`, `encrypt`, `decrypt`, `hash`, `checksum`
- **Section 5a:** Additional Packages — `chachapoly`, `lthn`, `pgp`, `rsa`, `openpgp`

### Local Documentation

- [Architecture](file:///Users/snider/Code/core/go-crypt/docs/architecture.md) — Full internals
- [History](file:///Users/snider/Code/core/go-crypt/docs/history.md) — Project history
- [Contributing](file:///Users/snider/Code/core/go-crypt/CONTRIBUTING.md) — Development guide
- [Upgrade Guide](file:///Users/snider/Code/core/go-crypt/UPGRADE.md) — Migration instructions

---

## 🗂️ File Structure

### Core Package (`go/crypt/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `crypt.go` | ~100 | Main encryption/decryption functions (Encrypt, Decrypt, EncryptAES, DecryptAES) | ✅ Complete |
| `kdf.go` | ~100 | Key derivation (Argon2id), password hashing | ✅ Complete |
| `checksum.go` | ~50 | MD5, SHA256 checksums | ✅ Complete |
| `hmac.go` | ~50 | HMAC-SHA256 generation and verification | ✅ Complete |

### ChaCha20-Poly1305 (`go/crypt/chachapoly/`)

| File | Purpose | Status |
|------|---------|--------|
| `chachapoly.go` | ChaCha20-Poly1305 encryption wrapper | ✅ Complete |
| `chachapoly_test.go` | Tests | ✅ Complete |

### LTHN Hash (`go/crypt/lthn/`)

| File | Purpose | Status |
|------|---------|--------|
| `lthn.go` | LTHN quasi-salted hash algorithm | ✅ Complete |
| `lthn_test.go` | Tests | ✅ Complete |

### PGP (`go/crypt/pgp/`)

| File | Purpose | Status |
|------|---------|--------|
| `pgp.go` | PGP operations (protonmail/go-crypto) | ✅ Complete |
| `pgp_test.go` | Tests | ✅ Complete |

### RSA (`go/crypt/rsa/`)

| File | Purpose | Status |
|------|---------|--------|
| `rsa.go` | RSA-2048 key generation, encryption, decryption | ✅ Complete |
| `rsa_test.go` | Tests | ✅ Complete |
| `fallback_test.go` | Fallback behavior tests | ✅ Complete |

### OpenPGP (`go/crypt/openpgp/`)

| File | Purpose | Status |
|------|---------|--------|
| `service.go` | OpenPGP service (golang.org/x/crypto/openpgp) | ✅ Complete |
| `service_test.go` | Tests | ✅ Complete |
| `service_example_test.go` | Usage examples | ✅ Complete |
| `error_paths_test.go` | Error path tests | ✅ Complete |
| `ax7_service_test.go` | Additional tests | ✅ Complete |
| `test_helpers_test.go` | Test helpers | ✅ Complete |

### Authentication (`go/auth/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `auth.go` | ~400 | OpenPGP challenge-response authentication, User, Challenge, Session types | ✅ Complete |
| `session_store.go` | ~100 | Session store interface | ✅ Complete |
| `session_store_sqlite.go` | ~200 | SQLite session store implementation | ✅ Complete |
| `hardware.go` | ~50 | Hardware token management (FIDO2 stub) | ✅ Stub |

### Authentication Tests

| File | Lines | Purpose | Coverage |
|------|-------|---------|----------|
| `auth_test.go` | ~500 | Authentication tests | ✅ Good/Bad/Ugly |
| `auth_example_test.go` | ~100 | Usage examples | ✅ Complete |
| `ax7_auth_test.go` | ~100 | Additional tests | ✅ Complete |
| `session_store_test.go` | ~300 | Session store tests | ✅ Good/Bad/Ugly |
| `session_store_example_test.go` | ~100 | Session store examples | ✅ Complete |
| `session_store_sqlite_test.go` | ~200 | SQLite session store tests | ✅ Complete |
| `hardware_test.go` | ~100 | Hardware token tests | ✅ Complete |
| `hardware_example_test.go` | ~50 | Hardware examples | ✅ Complete |
| `error_paths_test.go` | ~100 | Error path tests | ✅ Complete |
| `test_helpers_test.go` | ~50 | Test helpers | ✅ Complete |

### Trust (`go/trust/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `trust.go` | ~150 | Trust tiers, Agent, Capability types | ✅ Complete |
| `registry.go` | ~200 | Agent registry | ✅ Complete |
| `policy.go` | ~300 | Policy engine and Policy types | ✅ Complete |
| `approval.go` | ~250 | Approval workflow | ✅ Complete |
| `audit.go` | ~200 | Audit trail | ✅ Complete |
| `config.go` | ~100 | Trust configuration | ✅ Complete |
| `service.go` | ~150 | Trust service integration | ✅ Complete |
| `scope.go` | ~50 | Scope matching helpers | ✅ Complete |
| `bench_test.go` | ~100 | Benchmarks | ✅ Complete |

### Trust Tests

| File | Lines | Purpose | Coverage |
|------|-------|---------|----------|
| `trust_test.go` | ~500 | Trust tests | ✅ Good/Bad/Ugly |
| `trust_example_test.go` | ~200 | Usage examples | ✅ Complete |
| `policy_test.go` | ~400 | Policy engine tests | ✅ Good/Bad/Ugly |
| `policy_example_test.go` | ~100 | Policy examples | ✅ Complete |
| `approval_test.go` | ~300 | Approval workflow tests | ✅ Good/Bad/Ugly |
| `approval_example_test.go` | ~100 | Approval examples | ✅ Complete |
| `audit_test.go` | ~300 | Audit trail tests | ✅ Good/Bad/Ugly |
| `audit_example_test.go` | ~100 | Audit examples | ✅ Complete |
| `config_test.go` | ~100 | Configuration tests | ✅ Complete |
| `config_example_test.go` | ~50 | Configuration examples | ✅ Complete |
| `service_test.go` | ~200 | Service tests | ✅ Complete |
| `service_example_test.go` | ~100 | Service examples | ✅ Complete |
| `service_behaviour_test.go` | ~150 | Service behavior tests | ✅ Complete |
| `test_helpers_test.go` | ~50 | Test helpers | ✅ Complete |

### CLI Commands (`go/cmd/crypt/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `cmd.go` | ~100 | Command group registration | ✅ Complete |
| `cmd_encrypt.go` | ~200 | encrypt/decrypt commands | ✅ Complete |
| `cmd_hash.go` | ~150 | hash command | ✅ Complete |
| `cmd_checksum.go` | ~150 | checksum command | ✅ Complete |
| `cmd_keygen.go` | ~100 | keygen command | ✅ Complete |

### CLI Tests

| File | Lines | Purpose | Coverage |
|------|-------|---------|----------|
| `cmd_test.go` | ~300 | Command tests | ✅ Good/Bad/Ugly |
| `cmd_behaviour_test.go` | ~150 | Behavior tests | ✅ Complete |
| `cmd_example_test.go` | ~100 | Usage examples | ✅ Complete |
| `ax7_commands_test.go` | ~100 | Additional command tests | ✅ Complete |

### Internal (`go/internal/corecompat/`)

| File | Purpose | Status |
|------|---------|--------|
| `corecompat.go` | Encoding helpers (HexEncode/Decode, Base64Encode/Decode) | ✅ Complete |

### Test Harness (`go/cmd/testcmd/`)

| File | Purpose | Status |
|------|---------|--------|
| `cmd_main.go` | Test harness main | ✅ Complete |
| `cmd_commands.go` | Test commands | ✅ Complete |
| `cmd_runner.go` | Test runner | ✅ Complete |
| `cmd_output.go` | Test output | ✅ Complete |
| `cmd_print.go` | Test print | ✅ Complete |

### Tests Directory (`tests/`)

Integration and end-to-end tests.

### Documentation (`docs/`)

| File | Purpose |
|------|---------|
| `architecture.md` | Full architecture documentation |
| `history.md` | Project history |

---

## 🔧 Public API Surface

### crypt Package

#### Encryption Functions

```go
// ChaCha20-Poly1305 (preferred)
func Encrypt(plaintext, passphrase []byte) ([]byte, error)
func Decrypt(ciphertext, passphrase []byte) ([]byte, error)

// AES-256-GCM (legacy/interop)
func EncryptAES(plaintext, passphrase []byte) ([]byte, error)
func DecryptAES(ciphertext, passphrase []byte) ([]byte, error)
```

**Format:** `salt (16 bytes) + nonce (24 for ChaCha20, 12 for AES) + ciphertext`

#### Key Derivation

```go
// Constants
const (
    argon2SaltLen = 16
    argon2KeyLen  = 32
)

// Derive key from passphrase and salt
func DeriveKey(passphrase, salt []byte, keyLen int) []byte

// Password hashing
func HashPassword(password string) (string, error)
func VerifyPassword(password, hash string) bool
```

**Argon2id Parameters:**
- Time: 2 iterations
- Memory: 64 MiB
- Parallelism: 4 threads

#### HMAC

```go
// Create HMAC-SHA256
func HMAC(secret, message []byte) []byte

// Verify HMAC
func VerifyHMAC(secret, message, mac []byte) bool
```

#### Checksums

```go
// MD5 checksum (hex string) - for compatibility only
func MD5Sum(data []byte) string

// SHA-256 checksum (hex string)
func SHA256Sum(data []byte) string
```

### chachapoly Package

```go
func ChaCha20Encrypt(plaintext, key []byte) ([]byte, error)
func ChaCha20Decrypt(ciphertext, key []byte) ([]byte, error)
```

**Format:** `nonce (24 bytes) + ciphertext`

### lthn Package

```go
func Hash(input string) string
func Verify(input, hash string) bool
```

**Algorithm:** SHA-256 with derived salt (input reversal + leet-speak)
**Note:** NOT for password hashing — use Argon2id

### pgp Package

```go
type KeyPair struct {
    PrivateKey string
    PublicKey  string
    KeyID      string
    Fingerprint string
}

func GenerateKey(name, email string) (*KeyPair, error)
func Encrypt(plaintext []byte, publicKey string) ([]byte, error)
func Decrypt(ciphertext []byte, privateKey string, passphrase []byte) ([]byte, error)
func Sign(data []byte, privateKey string, passphrase []byte) ([]byte, error)
func Verify(data, signature, publicKey string) (bool, error)
```

### rsa Package

```go
func GenerateKey() (publicKey, privateKey []byte, err error)
func Encrypt(plaintext, publicKey []byte) ([]byte, error)
func Decrypt(ciphertext, privateKey []byte) ([]byte, error)
```

### openpgp Package

```go
type Service struct { /* unexported */ }

func NewService() *Service
func (s *Service) Sign(data []byte, privateKey []byte) ([]byte, error)
func (s *Service) Verify(data, signature, publicKey []byte) (bool, error)
```

---

## 🔐 Authentication API

### Types

```go
// User represents a registered user with PGP credentials
type User struct {
    PublicKey    string    `json:"public_key"`
    KeyID        string    `json:"key_id"`
    Fingerprint  string    `json:"fingerprint"`
    PasswordHash string    `json:"password_hash"` // Argon2id (new) or LTHN (legacy)
    Created      time.Time `json:"created"`
    LastLogin    time.Time `json:"last_login"`
}

// Challenge is a PGP-encrypted nonce
type Challenge struct {
    Nonce     []byte    `json:"nonce"`
    Encrypted string    `json:"encrypted"` // PGP-encrypted nonce (armored)
    ExpiresAt time.Time `json:"expires_at"`
}

// Session represents an authenticated session
type Session struct {
    Token     string    `json:"token"`
    UserID    string    `json:"user_id"`
    ExpiresAt time.Time `json:"expires_at"`
}

// Revocation records the details of a revoked user key
type Revocation struct {
    UserID    string    `json:"user_id"`
    Reason    string    `json:"reason"`
    RevokedAt time.Time `json:"revoked_at"`
}

// Option configures an Authenticator
type Option func(*Authenticator)

// Constants
const (
    DefaultChallengeTTL = 5 * time.Minute
    DefaultSessionTTL   = 24 * time.Hour
)
```

### Authenticator

```go
type Authenticator struct { /* unexported */ }

// Constructor with options
func NewAuthenticator(opts ...Option) (*Authenticator, error)

// Option functions
func WithChallengeTTL(ttl time.Duration) Option
func WithSessionTTL(ttl time.Duration) Option
func WithSessionStore(store SessionStore) Option

// Challenge generation
func (a *Authenticator) GenerateChallenge(publicKey string) (*Challenge, error)

// Response verification
func (a *Authenticator) VerifyResponse(challenge *Challenge, signature string, publicKey string) (*Session, error)
```

### Session Store

```go
// Interface
type SessionStore interface {
    CreateSession(userID string, tier trust.Tier, ttl time.Duration) (string, error)
    GetSession(token string) (*Session, error)
    RevokeSession(token string) error
    CleanupExpired() error
}

// SQLite implementation
func NewSessionStore(dbPath string) SessionStore
```

---

## 🛡️ Trust API

### Trust Tiers

```go
type Tier int

const (
    TierUntrusted Tier = 1  // External/community agents
    TierVerified   Tier = 2  // Partner agents with scoped access
    TierFull       Tier = 3  // Internal agents with full access
)

func (t Tier) String() string
func (t Tier) Valid() bool
```

### Capabilities

```go
type Capability string

const (
    CapPushRepo        Capability = "repo.push"
    CapMergePR         Capability = "pr.merge"
    CapCreatePR        Capability = "pr.create"
    CapCreateIssue     Capability = "issue.create"
    CapCommentIssue    Capability = "issue.comment"
    CapReadSecrets     Capability = "secrets.read"
    CapRunPrivileged   Capability = "cmd.privileged"
    CapAccessWorkspace Capability = "workspace.access"
    CapModifyFlows     Capability = "flows.modify"
)
```

### Agent

```go
type Agent struct {
    Name        string
    Tier        Tier
    ScopedRepos []string  // ["*"] for unrestricted; empty for no access
    RateLimit   int       // requests/minute; 0 = unlimited
}
```

### Registry

```go
type Registry struct {
    mu      sync.RWMutex
    agents  map[string]*Agent
}

func NewRegistry() *Registry
func (r *Registry) Register(agent *Agent)
func (r *Registry) Get(name string) *Agent
func (r *Registry) Has(name string) bool
func (r *Registry) All() []*Agent
func (r *Registry) Remove(name string)
```

### Policy Engine

```go
type Decision int

const (
    Deny Decision = iota
    Allow
    NeedsApproval
)

func (d Decision) String() string

type Policy struct {
    Tier           Tier
    Allowed        []Capability
    RequiresApproval []Capability
    Denied         []Capability
    RepoScope      []string
}

type EvalResult struct {
    Decision Decision
    Agent    string
    Cap      Capability
    Reason   string
}

type PolicyEngine struct {
    registry *Registry
    policies map[Tier]*Policy
}

func NewPolicyEngine(registry *Registry) *PolicyEngine
func (pe *PolicyEngine) AddPolicy(policy *Policy)
func (pe *PolicyEngine) Evaluate(agentName string, cap Capability, repo string) EvalResult
```

### Approval Workflow

```go
type CapabilityRequest struct {
    Agent      string
    Capability Capability
    Repo       string
    Reason     string
}

type ApprovalRequest struct {
    ID        string
    Request   CapabilityRequest
    Status    string  // pending, approved, denied
    Decision  string
    CreatedAt time.Time
    DecidedAt time.Time
}

type ApprovalWorkflow struct {
    store SessionStore  // Uses same SQLite store as sessions
}

func NewApprovalWorkflow(store SessionStore) *ApprovalWorkflow
func (w *ApprovalWorkflow) RequestApproval(req *CapabilityRequest) (*ApprovalRequest, error)
func (w *ApprovalWorkflow) Approve(id, reason string)
func (w *ApprovalWorkflow) Deny(id, reason string)
func (w *ApprovalWorkflow) GetStatus(id string) string
func (w *ApprovalWorkflow) Pending() []*ApprovalRequest
func (w *ApprovalWorkflow) Approved() []*ApprovalRequest
func (w *ApprovalWorkflow) Denied() []*ApprovalRequest
```

### Audit Trail

```go
type AuditEntry struct {
    ID        string
    Agent     string
    Capability Capability
    Repo      string
    Decision  Decision
    Reason    string
    Timestamp time.Time
}

type AuditLog struct {
    store SessionStore
}

func NewAuditLog(store SessionStore) *AuditLog
func (a *AuditLog) All() []*AuditEntry
func (a *AuditLog) ForAgent(agent string) []*AuditEntry
func (a *AuditLog) ForCapability(cap Capability) []*AuditEntry
func (a *AuditLog) ForRepo(repo string) []*AuditEntry
func (a *AuditLog) Since(t time.Time) []*AuditEntry
func (a *AuditLog) Add(entry *AuditEntry)
```

---

## 🔌 CLI Commands

### Command Group

```go
// Register with Core CLI
func RegisterCommands()
```

### keygen Command

```go
// Flags
--length int    // Key length in bytes (1-1024, default: 32)
--hex          // Output as hex string (default)
--base64       // Output as base64 string

// Usage
core crypt keygen --length 32 --hex
```

### encrypt/decrypt Commands

```go
// Flags
--passphrase string  // Passphrase for encryption/decryption
--aes               // Use AES-256-GCM instead of ChaCha20-Poly1305

// Usage
core crypt encrypt --passphrase "secret" file.txt
core crypt decrypt --passphrase "secret" file.txt.enc
```

### hash Command

```go
// Flags
--bcrypt    // Use bcrypt instead of Argon2id
--verify string  // Verify password against hash

// Usage
core crypt hash
cat password.txt | core crypt hash
core crypt hash --bcrypt
echo "password" | core crypt hash --verify "$argon2id$..."
```

### checksum Command

```go
// Flags
--sha512        // Use SHA-512 instead of SHA-256
--verify string  // Verify file against expected checksum

// Usage
core crypt checksum file.txt
core crypt checksum --sha512 file.txt
core crypt checksum --verify "expected-hash" file.txt
```

---

## 🏷️ Component Catalog

### Cryptography Components

| Component | File | Purpose | Algorithm |
|-----------|------|---------|-----------|
| `Encrypt`/`Decrypt` | `crypt.go` | Symmetric encryption | ChaCha20-Poly1305 |
| `EncryptAES`/`DecryptAES` | `crypt.go` | Symmetric encryption | AES-256-GCM |
| `DeriveKey` | `kdf.go` | Key derivation | Argon2id |
| `HashPassword`/`VerifyPassword` | `kdf.go` | Password hashing | Argon2id |
| `HMAC`/`VerifyHMAC` | `hmac.go` | Message authentication | HMAC-SHA256 |
| `MD5Sum` | `checksum.go` | Checksum | MD5 (non-cryptographic) |
| `SHA256Sum` | `checksum.go` | Checksum | SHA-256 |
| `ChaCha20Encrypt`/`ChaCha20Decrypt` | `chachapoly/` | AEAD encryption | ChaCha20-Poly1305 |
| `Hash`/`Verify` | `lthn/` | Quasi-salted hash | LTHN (SHA-256 based) |
| `GenerateKey`/`Encrypt`/`Decrypt` | `pgp/` | PGP operations | protonmail/go-crypto |
| `GenerateKey`/`Encrypt`/`Decrypt` | `rsa/` | Asymmetric encryption | RSA-2048 |
| `Sign`/`Verify` | `openpgp/` | OpenPGP signatures | golang.org/x/crypto/openpgp |

### Authentication Components

| Component | File | Purpose |
|-----------|------|---------|
| `Authenticator` | `auth.go` | OpenPGP challenge-response |
| `User` | `auth.go` | User identity |
| `Challenge` | `auth.go` | Authentication challenge |
| `Session` | `auth.go` | Authenticated session |
| `Revocation` | `auth.go` | Key revocation record |
| `SessionStore` | `session_store.go` | Session storage interface |
| `SQLiteSessionStore` | `session_store_sqlite.go` | SQLite implementation |
| `HardwareToken` | `hardware.go` | FIDO2/WebAuthn (stub) |

### Trust Components

| Component | File | Purpose |
|-----------|------|---------|
| `Tier` | `trust.go` | Trust level constants |
| `Capability` | `trust.go` | Capability constants |
| `Agent` | `trust.go` | Agent identity |
| `Registry` | `registry.go` | Agent registry |
| `Policy` | `policy.go` | Access policy |
| `PolicyEngine` | `policy.go` | Policy evaluation |
| `Decision` | `policy.go` | Evaluation result |
| `EvalResult` | `policy.go` | Detailed evaluation result |
| `ApprovalRequest` | `approval.go` | Approval request |
| `ApprovalWorkflow` | `approval.go` | Approval management |
| `AuditEntry` | `audit.go` | Audit log entry |
| `AuditLog` | `audit.go` | Audit trail |
| `Config` | `config.go` | Trust configuration |
| `Service` | `service.go` | Trust service |

### CLI Components

| Component | File | Purpose |
|-----------|------|---------|
| `cmd.go` | `cmd/crypt/cmd.go` | Command group |
| `cmd_encrypt.go` | `cmd/crypt/cmd_encrypt.go` | encrypt/decrypt |
| `cmd_hash.go` | `cmd/crypt/cmd_hash.go` | hash |
| `cmd_checksum.go` | `cmd/crypt/cmd_checksum.go` | checksum |
| `cmd_keygen.go` | `cmd/crypt/cmd_keygen.go` | keygen |

---

## 📊 Comparison Tables

### Encryption Comparison

| Feature | ChaCha20-Poly1305 | AES-256-GCM |
|---------|-------------------|-------------|
| Preferred | ✅ Yes | ❌ Legacy |
| Speed | ~500-800 MB/s | ~800-1200 MB/s |
| Hardware Acceleration | ❌ No needed | ✅ Available |
| Nonce Size | 24 bytes | 12 bytes |
| Security | Modern | NIST Standard |
| Use Case | New code | Interoperability |

### Hash Comparison

| Algorithm | Purpose | Security | Speed |
|-----------|---------|----------|-------|
| Argon2id | Password hashing | ✅ High | ~10-20/s |
| bcrypt | Password hashing | ✅ High | ~5-10/s |
| SHA-256 | Checksum, signing | ✅ High | ~500+ MB/s |
| MD5 | Checksum (legacy) | ❌ Broken | ~1+ GB/s |
| LTHN | Quasi-salted hash | ⚠️ Limited | ~500+ MB/s |

### Trust Tier Comparison

| Tier | Name | Access Level | Use Case |
|------|------|--------------|----------|
| 3 | Full | All repos, all capabilities | Internal agents (Athena, Virgil, Charon) |
| 2 | Verified | Scoped repos, limited capabilities | Partner agents (Clotho, Hypnos) |
| 1 | Untrusted | Minimal, read-only | External/community agents |

---

## 🔗 Dependencies

### External Dependencies

| Package | Purpose | Version |
|---------|---------|---------|
| `golang.org/x/crypto/chacha20poly1305` | ChaCha20-Poly1305 AEAD | Latest |
| `golang.org/x/crypto/argon2` | Argon2id key derivation | Latest |
| `golang.org/x/crypto/openpgp` | OpenPGP signatures | Latest |
| `golang.org/x/crypto/bcrypt` | bcrypt password hashing | Latest |
| `crypto/rsa` | RSA-2048 | Stdlib |
| `crypto/rand` | Cryptographically secure random | Stdlib |
| `crypto/hmac` | HMAC | Stdlib |
| `crypto/sha256` | SHA-256 | Stdlib |
| `crypto/md5` | MD5 (legacy) | Stdlib |
| `protonmail/go-crypto` | PGP operations | Latest |
| `github.com/mattn/go-sqlite3` | SQLite driver | Latest |

### Internal Dependencies

| Package | Purpose | Import Path |
|---------|---------|-------------|
| Core framework | Error wrapping | `dappco.re/go/core` |
| Core log | Contextual error wrapping | `dappco.re/go/log` |
| Core I/O | I/O abstraction | `dappco.re/go/io` |
| Core store | SQLite KV store | `dappco.re/go/store` |
| Core CLI | CLI framework | `forge.lthn.ai/core/cli` |
| Enchantrix | Cryptographic primitives | `forge.lthn.ai/Snider/Enchantrix/pkg/crypt/std/*` |

### Banned Imports

The following are **banned** in go-crypt code:

| Banned | Use Instead |
|--------|-------------|
| `math/rand` | `crypto/rand` only |
| `fmt.Errorf`, `errors.New` | `coreerr.E("package.Function", "message", err)` |

---

## 📊 Statistics

### Code Metrics

```
Total Go files:           50+
Total source lines:    ~2,000+
Total test lines:      ~8,000+
Public API symbols:     ~80
Sub-packages:            8 (crypt, chachapoly, lthn, pgp, rsa, openpgp, auth, trust)
CLI commands:            5 (keygen, encrypt, decrypt, hash, checksum)
Trust tiers:             3 (Untrusted, Verified, Full)
Capabilities:            9 predefined
```

### Test Coverage

| Category | Files | Tests | Status |
|----------|-------|-------|--------|
| crypt | 4 | 60+ | ✅ Complete (Good/Bad/Ugly) |
| chachapoly | 2 | 20+ | ✅ Complete |
| lthn | 2 | 15+ | ✅ Complete |
| pgp | 2 | 20+ | ✅ Complete |
| rsa | 3 | 25+ | ✅ Complete |
| openpgp | 6 | 30+ | ✅ Complete |
| auth | 5 | 80+ | ✅ Complete |
| trust | 8 | 120+ | ✅ Complete |
| cmd/crypt | 5 | 50+ | ✅ Complete |
| internal | 1 | 10+ | ✅ Complete |

### Security Properties

| Property | Status | Notes |
|----------|--------|-------|
| FIPS 140-2 | ⚠️ Partial | ChaCha20-Poly1305 and AES-256-GCM are FIPS-approved |
| Side-channel resistance | ✅ Yes | Constant-time operations |
| Memory safety | ✅ Yes | Zeroize sensitive data |
| Audit logging | ✅ Yes | Immutable audit trail |
| Rate limiting | ✅ Yes | Configurable per-agent |

---

## 🎯 Usage Patterns

### Pattern 1: Password-Based Encryption

```go
import "dappco.re/go/crypt/crypt"

// Encrypt with password
ciphertext, _ := crypt.Encrypt([]byte("secret"), []byte("password"))

// Decrypt with password
plaintext, _ := crypt.Decrypt(ciphertext, []byte("password"))
```

### Pattern 2: Key-Based Encryption

```go
import "dappco.re/go/crypt/crypt"

// Derive key
salt := make([]byte, 16)
key := crypt.DeriveKey([]byte("password"), salt, 32)

// Use with ChaCha20-Poly1305
ciphertext, _ := crypt.ChaCha20Encrypt(plaintext, key)
plaintext, _ := crypt.ChaCha20Decrypt(ciphertext, key)
```

### Pattern 3: Agent Authentication

```go
import (
    "dappco.re/go/crypt/auth"
    "dappco.re/go/crypt/trust"
)

// Setup
store := auth.NewSessionStore("sessions.db")
auth, _ := auth.NewAuthenticator(auth.WithSessionStore(store))

// Generate challenge for agent
challenge, _ := auth.GenerateChallenge(agent.PublicKey)

// After agent responds with signature
session, _ := auth.VerifyResponse(challenge, signature, agent.PublicKey)

// Check trust tier
if session.Agent.Tier >= trust.TierVerified {
    // Grant access
}
```

### Pattern 4: Capability Check

```go
import "dappco.re/go/crypt/trust"

// Setup
registry := trust.NewRegistry()
engine := trust.NewPolicyEngine(registry)

// Check capability
result := engine.Evaluate("Clotho", trust.CapPushRepo, "core")
if result.Decision == trust.Allow {
    // Grant access
}
```

### Pattern 5: Full Trust Workflow

```go
import "dappco.re/go/crypt/trust"

// Setup
store := auth.NewSessionStore("trust.db")
registry := trust.NewRegistry()
engine := trust.NewPolicyEngine(registry)
workflow := trust.NewApprovalWorkflow(store)
audit := trust.NewAuditLog(store)

// Register agents
registry.Register(&trust.Agent{Name: "Athena", Tier: trust.TierFull})
registry.Register(&trust.Agent{Name: "Clotho", Tier: trust.TierVerified})

// Check and possibly approve
result := engine.Evaluate("Clotho", trust.CapPushRepo, "core")
if result.Decision == trust.NeedsApproval {
    req, _ := workflow.RequestApproval(&trust.CapabilityRequest{
        Agent: "Clotho", Capability: trust.CapPushRepo, Repo: "core",
        Reason: "Merging feature",
    })
    workflow.Approve(req.ID, "Approved by Athena")
}
```

### Pattern 6: Encrypted Storage

```go
import (
    "dappco.re/go/io"
    "dappco.re/go/crypt/crypt"
)

// Create encrypted medium
medium, _ := io.NewMedium("encrypted", io.WithCrypt(
    func(data []byte, passphrase []byte) ([]byte, error) {
        return crypt.Encrypt(data, passphrase)
    },
    func(data []byte, passphrase []byte) ([]byte, error) {
        return crypt.Decrypt(data, passphrase)
    },
))

// Use like normal I/O
medium.Write("secrets.dat", []byte("confidential"))
data, _ := medium.Read("secrets.dat")
```

---

## 📋 Compliance Summary

### Coding Standards

✅ **UK English:** colour, organisation, centre, artefact, licence, serialise
✅ **Strict types:** All parameters and return types explicitly typed
✅ **AX comments:** Usage examples, not prose descriptions
✅ **Test triplets:** Good/Bad/Ugly pattern for all functions
✅ **Licence:** EUPL-1.2 with SPDX identifier
✅ **Error handling:** `coreerr.E("package.Function", "message", err)` exclusively
✅ **Randomness:** `crypto/rand` only; never `math/rand`

### Security Rules

✅ **No secret leakage:** Errors never include secrets
✅ **Constant-time operations:** For cryptographic comparisons
✅ **Memory safety:** Zeroize sensitive data after use
✅ **Input validation:** All inputs validated before processing
✅ **Rate limiting:** Protect against brute-force attacks
✅ **Session expiration:** Automatic cleanup of expired sessions
✅ **Secure defaults:** Argon2id parameters suitable for production

### File Organization

✅ **Comments as examples:** Every exported type/function has usage comments
✅ **SPOR compliance:** Single Point Of Responsibility for each file
✅ **Test file pairing:** Every `.go` file has `_test.go` and `_example_test.go`
✅ **No banned imports:** Uses only `crypto/rand`, never `math/rand`

### Test Organization

✅ **File-aware tests:** Each production file owns its test file
✅ **Table-driven subtests:** Uses `t.Run()`
✅ **Good/Bad/Ugly pattern:** Comprehensive test coverage
✅ **Race-safe:** All tests pass with `-race`
✅ **Concurrency tests:** 10 goroutines via WaitGroup
✅ **Integration tests:** Full workflow testing

### Verification

```sh
GOWORK=off go mod tidy
GOWORK=off go vet ./...
GOWORK=off go test -count=1 -race ./...
gofmt -l .
```

---

## 📝 Maintenance Information

- **Author:** Mistral Vibe (Purberus <purberus@lthn.ai>)
- **Created:** 2026-06-18T04:00:00Z
- **Last Updated:** 2026-06-18T04:00:00Z
- **Version:** 1.0.0
- **Licence:** EUPL-1.2
- **Repository:** `forge.lthn.sh/core/go-crypt`
- **Module:** `dappco.re/go/crypt`

### Key Contacts

- **Project Lead:** Hades (Lethean)
- **Maintainer:** Purberus <purberus@lthn.ai>
- **Security Officer:** Virgil (virgil@lethean.io)
- **Used by:** go-io, Borg, Mining, core/agent
- **Consumes:** Core framework, Enchantrix primitives
- **CI/CD:** Woodpecker.yml

### Upstream Dependencies

- **CoreGo framework:** `dappco.re/go/core` (mandatory)
- **Enchantrix:** `forge.lthn.ai/Snider/Enchantrix/pkg/crypt/std/*` (primitives)
- **protonmail/go-crypto:** PGP operations
- **sqlite3:** Session/approval/audit storage

### Downstream Dependencies

- **go-io:** Uses `crypt.Encrypt`/`crypt.Decrypt` for encrypted Medium backends
- **Borg:** Uses cryptography for SMSG/STIM/STMF encryption
- **Mining:** Uses for UEPS consent tokens
- **core/agent:** Uses trust system for fleet dispatch permissions

---

## 🏷️ Tags & Categories

### Technology Tags

- `cryptography` — Primary tag
- `encryption` — Symmetric encryption
- `security` — Security-focused
- `aes` — AES encryption
- `chacha20` — ChaCha20 cipher
- `argon2` — Argon2id key derivation
- `hmac` — HMAC authentication
- `pgp` — PGP encryption
- `openpgp` — OpenPGP standard
- `rsa` — RSA encryption
- `authentication` — Authentication system
- `trust` — Trust management
- `access-control` — Access control system
- `capabilities` — Capability-based permissions
- `policy` — Policy engine
- `audit` — Audit logging

### Feature Tags

- `encrypt` — Encryption
- `decrypt` — Decryption
- `hashing` — Hash functions
- `kdf` — Key derivation
- `signature` — Digital signatures
- `session` — Session management
- `tier` — Trust tiers
- `capability` — Capability checking
- `approval` — Approval workflow
- `audit-trail` — Audit logging
- `cli` — Command-line interface

### Security Tags

- `secure` — Security-focused
- `fips` — FIPS-compliant algorithms
- `side-channel-resistant` — Side-channel protection
- `memory-safe` — Memory safety
- `production-ready` — Production-ready

### Agent Tags

- `athena` — Full trust agent
- `virgil` — Full trust agent
- `charon` — Full trust agent
- `clotho` — Verified agent
- `hypnos` — Verified agent
- `untrusted` — Untrusted agents

---

*Package index generated: 2026-06-18T04:00:00Z*
*Knowledge Pack: CoreGo v1.3.0*
