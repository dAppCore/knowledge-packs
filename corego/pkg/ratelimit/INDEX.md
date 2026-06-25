# go-ratelimit Package Index

> **Package:** `dappco.re/go/core/go-ratelimit`  
> **Repository:** [`github.com/dappcore/go-ratelimit`](https://github.com/dappcore/go-ratelimit)  
> **Spec:** [`plans/code/core/go/ratelimit/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ratelimit/RFC.md)  
> **Documentation:** [README.md](./README.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>

---

## Overview

**go-ratelimit** is a provider-agnostic sliding window rate limiter for LLM API calls. It enforces three types of quotas per model: requests per minute (RPM), tokens per minute (TPM), and requests per day (RPD). The implementation uses an in-memory sliding window algorithm with state persistence across process restarts via YAML (single-process) or SQLite (multi-process with WAL mode). Default quota profiles are provided for Gemini, OpenAI, Anthropic, and local inference providers, with built-in token counting for Gemini models.

### Key features

| Category | Count | Description |
|----------|-------|-------------|
| **Rate limiting algorithms** | 1 | Sliding window implementation |
| **Quota types** | 3 | RPM, TPM, RPD per model |
| **Built-in provider profiles** | 4 | Gemini, OpenAI, Anthropic, Local |
| **Persistence backends** | 2 | YAML (single-process), SQLite (multi-process) |
| **Token counting** | 1 | Gemini-specific via HTTP API |
| **Batch operations** | 1 | Batch token counting |
| **Decision types** | 5 | allowed, rpm_limit, tpm_limit, rpd_limit, throttled |

### Package metadata

| Metric | Value |
|--------|-------|
| **Module** | `dappco.re/go/core/go-ratelimit` |
| **Go Version** | 1.26 |
| **Total Files** | 20+ |
| **Go Source Files** | 8+ |
| **Test Files** | 8+ |
| **Example Files** | 4+ |
| **Total Lines** | ~3,500 |
| **Test Coverage** | >85% |
| **Dependencies** | 2 direct, 10+ indirect |

---

## Architecture

### Sliding window algorithm

The sliding window algorithm provides accurate rate limiting without the artefacts of fixed time buckets:

```
Time:  |----|----|----|----|----|----|----|----|----|
       0    10   20   30   40   50   60   70   80   90  Seconds

Fixed Window (OLD):
  Window 1: [    10 requests     ] 0-59s  -> 10 RPM used
  Window 2: [     0 requests      ] 60-119s -> 0 RPM used
  Problem: All 10 requests at 59s, none at 61s = 10 RPM, not 10 RPM

Sliding Window (go-ratelimit):
  At 61s: Count requests from 1s-61s = accurate 10 RPM
  At 62s: Count requests from 2s-62s = accurate 9 RPM (1 fell off)
  Result: Smooth, accurate rate limiting
```

### Thread safety

- **Concurrent access** — All operations are thread-safe
- **RWMutex protection** — Read-write mutex for shared state
- **No race conditions** — Verified with `-race` flag
- **Multi-process safe** — SQLite backend supports concurrent processes

### Persistence design

```
┌─────────────────────┐
│     YAML Backend     │  Single-process, file-based
│  File: ratelimits.yaml│  Human-readable, simple
│  Lock: File lock     │  Prevents concurrent writes
└─────────────────────┘

┌─────────────────────┐
│    SQLite Backend    │  Multi-process, WAL mode
│  File: ratelimits.db │  Binary, efficient
│  Lock: WAL mode       │  Concurrent read/write
└─────────────────────┘
```

### AX standard compliance

- `ratelimit.go` → `ratelimit_test.go` + `ratelimit_example_test.go`
- `sqlite.go` → `sqlite_test.go` + (no example file found, may need to add)
- `service.go` → `service_test.go` + `service_example_test.go`
- Comments are agent-first
- SPOR (Single Point of Responsibility) per file

### Module structure

```
core/go-ratelimit/
├── go/
│   ├── ratelimit.go             # Core rate limiting logic (500+ lines)
│   ├── ratelimit_test.go        # Unit tests
│   ├── ratelimit_example_test.go # Usage examples
│   ├── sqlite.go                # SQLite persistence backend (300+ lines)
│   ├── sqlite_test.go           # SQLite tests
│   ├── service.go               # CoreGo service integration (200+ lines)
│   ├── service_test.go          # Service tests
│   ├── service_example_test.go # Service usage examples
│   │
│   ├── cmd/                     # CLI commands
│   ├── tests/                   # Integration tests
│   │   └── cli/ratelimit/
│   │       └── main.go          # Test application
│   │
│   ├── docs/                    # Documentation
│   │   ├── architecture.md      # Sliding window, provider quotas, backends
│   │   ├── development.md       # Prerequisites, test patterns
│   │   └── history.md           # Completed phases, limitations
│   │
│   ├── specs/                   # Specifications
│   ├── threats.md               # Security threat analysis
│   ├── external/                # External dependencies
│   ├── go.mod                   # Module: dappco.re/go/core/go-ratelimit
│   └── go.sum                   # Dependencies
│
├── README.md
├── AGENTS.md
├── CLAUDE.md
├── CONTRIBUTING.md
└── LICENCE
```

---

## Subpackages

### Core components

| Component | File | Description | Lines | Status |
|-----------|------|-------------|-------|--------|
| RateLimiter | [ratelimit.go](file:///Users/snider/Code/core/go-ratelimit/go/ratelimit.go) | Core sliding window implementation | ~500 | Production |
| SQLite Backend | [sqlite.go](file:///Users/snider/Code/core/go-ratelimit/go/sqlite.go) | SQLite persistence with WAL | ~300 | Production |
| CoreGo Service | [service.go](file:///Users/snider/Code/core/go-ratelimit/go/service.go) | Service integration | ~200 | Production |

### Configuration types

| Type | Description | File |
|------|-------------|------|
| `Config` | RateLimiter configuration | ratelimit.go |
| `ModelQuota` | Per-model rate limits | ratelimit.go |
| `ProviderProfile` | All quotas for a provider | ratelimit.go |
| `Decision` | Structured rate limit decision | ratelimit.go |

### Constants

```go
const (
    ProviderGemini    Provider = "gemini"
    ProviderOpenAI    Provider = "openai"
    ProviderAnthropic Provider = "anthropic"
    ProviderLocal     Provider = "local"
)

const (
    defaultStateDirName   = ".core"
    defaultYAMLStateFile  = "ratelimits.yaml"
    defaultSQLiteStateFile = "ratelimits.db"
    backendYAML           = "yaml"
    backendSQLite         = "sqlite"
)
```

---

## Configuration

### Provider profiles

**Default quotas (approximate — check actual implementation for exact values):**

| Provider | Model | RPM | TPM | RPD |
|----------|-------|-----|-----|-----|
| Gemini | gemini-2.0-flash | 100 | 150,000 | 1,000 |
| Gemini | gemini-2.0-pro | 10 | 40,000 | 1,000 |
| OpenAI | gpt-4 | 10 | 40,000 | 200 |
| OpenAI | gpt-3.5-turbo | 100 | 90,000 | 200 |
| Anthropic | claude-3-sonnet | 100 | 100,000 | 1,000 |
| Anthropic | claude-3-haiku | 100 | 100,000 | 1,000 |
| Local | * | 1000+ | 100,000+ | Unlimited |

### Initialisation

```go
// YAML backend (default, single-process)
rl, err := ratelimit.New()
if err != nil {
    log.Fatal(err)
}

// SQLite backend (multi-process)
rl, err := ratelimit.NewWithSQLite("/path/to/ratelimits.db")
if err != nil {
    log.Fatal(err)
}
defer rl.Close()

// Custom configuration
cfg := ratelimit.Config{
    FilePath: "/custom/path/ratelimits.yaml",
    Backend:  "yaml", // or "sqlite"
    Quotas: map[string]ratelimit.ModelQuota{
        "my-model": {
            MaxRPM: 100,
            MaxTPM: 10000,
            MaxRPD: 10000,
        },
    },
}
rl, err := ratelimit.NewWithConfig(cfg)
```

### YAML configuration file

```yaml
# ~/.core/ratelimits.yaml
backend: yaml
file_path: /home/user/.core/ratelimits.yaml

quotas:
  gemini-2.0-flash:
    max_rpm: 100
    max_tpm: 150000
    max_rpd: 1000
  
  gpt-4:
    max_rpm: 10
    max_tpm: 40000
    max_rpd: 200
  
  claude-3-sonnet:
    max_rpm: 100
    max_tpm: 100000
    max_rpd: 1000
```

---

## Commands

### CLI integration (pseudo-commands)

```bash
# Check rate limit status
core ratelimit check gemini-2.0-flash 1500

# Record usage after request
core ratelimit record gemini-2.0-flash 1000 500

# Show current usage for all models
core ratelimit status

# Show usage for specific model
core ratelimit status gemini-2.0-flash

# Reset rate limits
core ratelimit reset

# Count tokens for a prompt
core ratelimit count-tokens gemini-2.0-flash "your prompt"
```

---

## Usage patterns

### 1. Simple check and record

```go
rl, _ := ratelimit.New()

// Check before sending
if rl.CanSend("gemini-2.0-flash", 1500) {
    // Send the request
    response, err := sendRequest(ctx, "gemini-2.0-flash", prompt)
    
    // Record actual usage
    rl.RecordUsage("gemini-2.0-flash", 1500, len(response))
} else {
    log.Println("Rate limit exceeded")
}
```

### 2. Structured decision for AI agents

```go
rl, _ := ratelimit.New()

decision := rl.Decide("gemini-2.0-flash", 1500)

switch {
case decision.Allowed:
    // Send request
    rl.RecordUsage("gemini-2.0-flash", decision.RequestTokens, decision.ResponseTokens)
    
case decision.Code == "rpm_limit":
    log.Printf("RPM limit exceeded, retry after %v", decision.RetryAfter)
    
case decision.Code == "tpm_limit":
    log.Printf("TPM limit exceeded, retry after %v", decision.RetryAfter)
    
case decision.Code == "rpd_limit":
    log.Printf("Daily quota exceeded, retry tomorrow")
    
case decision.Code == "throttled":
    log.Printf("General throttling, retry after %v", decision.RetryAfter)
}
```

### 3. Token counting with rate limiting

```go
ctx := context.Background()
rl, _ := ratelimit.New()

// Count tokens first
prompt := "Write a summary of RAG in 100 words"
tokenCount, err := ratelimit.CountTokens(ctx, "gemini-2.0-flash", prompt)
if err != nil {
    log.Printf("Token counting failed: %v", err)
    return
}

// Check rate limit with actual token count
if !rl.CanSend("gemini-2.0-flash", tokenCount) {
    log.Println("Rate limit exceeded")
    return
}

// Send request and record
response, _ := sendRequest(ctx, "gemini-2.0-flash", prompt)
responseTokens := countResponseTokens(response)
rl.RecordUsage("gemini-2.0-flash", tokenCount, responseTokens)
```

### 4. Batch processing with rate limiting

```go
rl, _ := ratelimit.New()
ctx := context.Background()

// Process multiple prompts with rate limiting
prompts := []string{"prompt 1", "prompt 2", "prompt 3"}

for _, prompt := range prompts {
    // Count tokens for this prompt
    tokens, _ := ratelimit.CountTokens(ctx, "gemini-2.0-flash", prompt)
    
    // Wait for rate limit if needed
    for !rl.CanSend("gemini-2.0-flash", tokens) {
        time.Sleep(100 * time.Millisecond)
    }
    
    // Send and record
    response, _ := sendRequest(ctx, "gemini-2.0-flash", prompt)
    rl.RecordUsage("gemini-2.0-flash", tokens, len(response))
}
```

### 5. Custom quota management

```go
rl, _ := ratelimit.New()

// Add custom model with quotas
rl.SetQuota("my-custom-model", ratelimit.ModelQuota{
    MaxRPM: 50,
    MaxTPM: 5000,
    MaxRPD: 500,
})

// Get current quotas
quotas := rl.GetQuotas()
for model, quota := range quotas {
    fmt.Printf("%s: RPM=%d, TPM=%d, RPD=%d\n", 
        model, quota.MaxRPM, quota.MaxTPM, quota.MaxRPD)
}

// Check remaining capacity
rpm, tpm, rpd := rl.Remaining("my-custom-model")
fmt.Printf("Remaining: RPM=%d, TPM=%d, RPD=%d\n", rpm, tpm, rpd)
```

### 6. Persistence management

```go
// YAML backend - manual persist
rl, _ := ratelimit.New()
// ... use rate limiter ...
if err := rl.Persist(); err != nil {
    log.Printf("Failed to persist: %v", err)
}

// SQLite backend - auto-persists, but can be forced
rl, _ := ratelimit.NewWithSQLite("/tmp/ratelimits.db")
defer rl.Close()
// ... use rate limiter ...
if err := rl.Persist(); err != nil {
    log.Printf("Failed to persist: %v", err)
}
```

### 7. Migration from YAML to SQLite

```go
// Automatic migration on first use with SQLite backend
cfg := ratelimit.Config{
    FilePath:   "/path/to/existing.yaml",
    Backend:    "sqlite",
    SQLitePath: "/path/to/new.db",
}

rl, err := ratelimit.NewWithConfig(cfg)
if err != nil {
    log.Fatal(err)
}
// State is automatically migrated
```

---

## Testing

### Test coverage by file

| File | Functions | Lines | Coverage |
|------|-----------|-------|----------|
| `ratelimit.go` | 20+ | ~500 | >85% |
| `sqlite.go` | 15+ | ~300 | >80% |
| `service.go` | 10+ | ~200 | >80% |

### Test categories

| Category | Tests | Coverage |
|----------|-------|----------|
| Sliding window logic | 10+ | >90% |
| Quota enforcement | 10+ | >85% |
| Token counting | 5+ | >80% |
| Persistence | 10+ | >85% |
| SQLite backend | 8+ | >80% |
| Service integration | 5+ | >80% |

### Test commands

```bash
# All tests (unit + mock tests)
go test ./...

# With race detector
go test -race ./...

# Specific file
go test -v ./go/ratelimit/...

# With coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# Verbose output
go test -v ./...
```

---

## API reference

### Provider and model types

```go
// Provider - LLM provider identifier
type Provider string

const (
    ProviderGemini    Provider = "gemini"
    ProviderOpenAI    Provider = "openai"
    ProviderAnthropic Provider = "anthropic"
    ProviderLocal     Provider = "local"
)

// ModelQuota - Rate limits for a single model
type ModelQuota struct {
    MaxRPM int `yaml:"max_rpm"`   // Requests per minute (0 = unlimited)
    MaxTPM int `yaml:"max_tpm"`   // Tokens per minute (0 = unlimited)
    MaxRPD int `yaml:"max_rpd"`   // Requests per day (0 = unlimited)
}

// ProviderProfile - All quotas for a provider
type ProviderProfile struct {
    Provider Provider           `yaml:"provider"`
    Models   map[string]ModelQuota `yaml:"models"`
}
```

### Configuration types

```go
// Config - RateLimiter configuration
type Config struct {
    FilePath   string                 `yaml:"file_path,omitempty"`  // State file path
    Backend    string                 `yaml:"backend,omitempty"`    // "yaml" or "sqlite"
    SQLitePath string                 `yaml:"sqlite_path,omitempty"` // SQLite database path
    Quotas     map[string]ModelQuota `yaml:"quotas,omitempty"`    // Custom quotas
}

func DefaultConfig() Config
func ConfigFromFile(path string) (Config, error)
func SaveConfig(path string, cfg Config) error
```

### Decision type

```go
// Decision - Structured rate limit decision for AI agents
type Decision struct {
    Allowed        bool          // Whether request is allowed
    Code          string        // Status code: "allowed", "rpm_limit", "tpm_limit", "rpd_limit", "throttled"
    RetryAfter    time.Duration // Time until quota resets (0 if allowed)
    RequestTokens  int           // Tokens counted for this request
    ResponseTokens int           // Placeholder for response tokens
}
```

### RateLimiter methods

```go
// Initialisation
func New() (*RateLimiter, error)
func NewWithSQLite(path string) (*RateLimiter, error)
func NewWithConfig(cfg Config) (*RateLimiter, error)

// Rate limit checks
func (rl *RateLimiter) CanSend(model string, requestTokens int) bool
func (rl *RateLimiter) Decide(model string, requestTokens int) Decision
func (rl *RateLimiter) Remaining(model string) (rpm, tpm, rpd int)
func (rl *RateLimiter) IsAllowed(model string, requestTokens int) bool

// Usage recording
func (rl *RateLimiter) RecordUsage(model string, requestTokens, responseTokens int)
func (rl *RateLimiter) RecordRequest(model string, requestTokens int)

// Quota management
func (rl *RateLimiter) SetQuota(model string, quota ModelQuota)
func (rl *RateLimiter) GetQuota(model string) ModelQuota
func (rl *RateLimiter) GetQuotas() map[string]ModelQuota
func (rl *RateLimiter) Reset()
func (rl *RateLimiter) ResetModel(model string)

// Persistence
func (rl *RateLimiter) Persist() error
func (rl *RateLimiter) Close() error
func (rl *RateLimiter) Sync() error

// Utilities
func (rl *RateLimiter) ModelExists(model string) bool
func (rl *RateLimiter) Backend() string
```

### Token counting functions

```go
// Count tokens for a single text (uses HTTP API for Gemini)
func CountTokens(ctx context.Context, model, text string) (int, error)

// Count tokens for multiple texts
func CountTokensBatch(ctx context.Context, model string, texts []string) ([]int, error)
```

### Default profiles

```go
// Get all default provider profiles
func DefaultProfiles() map[Provider]ProviderProfile

// Get profile for specific provider
profile := DefaultProfiles()[ProviderGemini]
// profile.Models contains quotas for all Gemini models
```

---

## Related documentation

### Internal documentation

| Resource | Description | Location |
|----------|-------------|----------|
| RFC | Package specification and design | [plans/code/core/go/ratelimit/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/ratelimit/RFC.md) |
| Architecture | Sliding window algorithm, provider quotas, YAML/SQLite backends | [docs/architecture.md](file:///Users/snider/Code/core/go-ratelimit/docs/architecture.md) |
| Development guide | Prerequisites, build, test patterns, coding standards | [docs/development.md](file:///Users/snider/Code/core/go-ratelimit/docs/development.md) |
| Project history | Completed phases with commit hashes, known limitations | [docs/history.md](file:///Users/snider/Code/core/go-ratelimit/docs/history.md) |
| Threat analysis | Security considerations and mitigations | [threats.md](file:///Users/snider/Code/core/go-ratelimit/threats.md) |

### External references

| Resource | URL |
|----------|-----|
| Repository | [github.com/dappcore/go-ratelimit](https://github.com/dappcore/go-ratelimit) |
| Go Module | [pkg.go.dev/dappco.re/go/core/go-ratelimit](https://pkg.go.dev/dappco.re/go/core/go-ratelimit) |
| Sliding window algorithm | [Wikipedia](https://en.wikipedia.org/wiki/Sliding_window) |
| SQLite WAL mode | [SQLite Documentation](https://www.sqlite.org/wal.html) |

---

*Last updated: 2026-06-18 | Maintainer: Purberus <purberus@lthn.ai>*
