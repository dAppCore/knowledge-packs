# go-ratelimit — Provider-Agnostic Rate Limiting

> **Package:** `dappco.re/go/core/go-ratelimit`  
> **Repository:** [`github.com/dappcore/go-ratelimit`](https://github.com/dappcore/go-ratelimit)  
> **Spec:** [`plans/code/core/go/ratelimit/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/ratelimit/RFC.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>  
> **Status:** ✅ Production Ready  
> **Module:** `dappco.re/go/core/go-ratelimit`

---

## 📋 Overview

**go-ratelimit** is a provider-agnostic sliding window rate limiter for LLM API calls. It enforces requests per minute (RPM), tokens per minute (TPM), and requests per day (RPD) quotas per model using an in-memory sliding window algorithm. State persists across process restarts via YAML (single-process) or SQLite (multi-process, WAL mode). Includes default quota profiles for Gemini, OpenAI, Anthropic, and local inference providers, plus a Gemini-specific token counting helper.

### 🎯 Key Capabilities

| Category | Features | Description |
|----------|----------|-------------|
| **Rate Limiting** | Sliding window algorithm | Accurate quota enforcement |
| **Quota Types** | 3 types | RPM, TPM, RPD per model |
| **Provider Profiles** | 4 built-in | Gemini, OpenAI, Anthropic, Local |
| **Persistence** | 2 backends | YAML (single-process), SQLite (multi-process) |
| **Token Counting** | Built-in | Gemini-specific token counter via HTTP API |
| **Migration** | YAML → SQLite | Seamless backend migration |
| **Agent Workflows** | Structured decisions | Retry guidance with structured verdicts |

### 🏗️ Architecture

### Module Structure

```
core/go-ratelimit/
├── go/                          # Go module root (dappco.re/go/core/go-ratelimit)
│   ├── ratelimit.go             # Core rate limiting logic
│   ├── ratelimit_test.go        # Unit tests
│   ├── ratelimit_example_test.go # Usage examples
│   ├── sqlite.go                # SQLite persistence backend
│   ├── sqlite_test.go           # SQLite tests
│   ├── service.go               # CoreGo service integration
│   ├── service_test.go          # Service tests
│   ├── service_example_test.go # Service usage examples
│   │
│   ├── cmd/                     # CLI commands (if any)
│   ├── tests/                   # Integration tests
│   │   └── cli/                 # CLI test utilities
│   │       └── ratelimit/       # Rate limit test app
│   │           └── main.go
│   │
│   ├── docs/                    # Documentation
│   │   ├── architecture.md      # Sliding window algorithm, provider quotas
│   │   ├── development.md       # Prerequisites, test patterns
│   │   └── history.md           # Completed phases, known limitations
│   │
│   ├── specs/                   # Specifications
│   ├── threats.md               # Security threat analysis
│   ├── external/                # External dependencies
│   ├── go.mod                   # Module definition
│   └── go.sum                   # Dependency checksums
│
├── README.md
├── AGENTS.md
├── CLAUDE.md
├── CONTRIBUTING.md
└── LICENCE
```

### Core Design Principles

1. **Provider-Agnostic** — Works with any LLM provider, not just built-in profiles
2. **Sliding Window** — Accurate quota tracking without fixed time buckets
3. **Persistent State** — Survives process restarts via YAML or SQLite
4. **Multi-Process Safe** — SQLite backend with WAL mode for concurrent access
5. **AX Standard** — Each `.go` file has `_test.go` and `_example_test.go`
6. **Agent-First** — Structured decisions with retry guidance for AI workflows

---

## 📦 Packages

### Core Rate Limiting (`ratelimit.go`)

#### Quota Types

- **RPM (Requests Per Minute)** — Rate limit based on request count
- **TPM (Tokens Per Minute)** — Rate limit based on token count
- **RPD (Requests Per Day)** — Daily quota limit

#### Provider Profiles

Built-in profiles for popular LLM providers:

| Provider | Models | RPM | TPM | RPD |
|----------|--------|-----|-----|-----|
| **Gemini** | gemini-2.0-flash, gemini-2.0-pro, gemini-1.5-flash, etc. | Varies by model | Varies by model | Varies by model |
| **OpenAI** | gpt-4, gpt-3.5-turbo, etc. | Varies by model | Varies by model | Varies by model |
| **Anthropic** | claude-3-sonnet, claude-3-haiku, etc. | Varies by model | Varies by model | Varies by model |
| **Local** | ollama, mlx, llama.cpp | High/Unlimited | High/Unlimited | Unlimited |

#### Sliding Window Algorithm

- **Accurate Tracking** — No fixed time bucket artifacts
- **Efficient** — O(1) operations for most checks
- **Thread-Safe** — Concurrent access protected
- **Memory Efficient** — Only stores recent requests

### Persistence Backends

#### YAML Backend (Default, Single-Process)

- **File-based** — Simple YAML file storage
- **Fast** — Low latency for single-process use
- **Human-Readable** — Easy to inspect and debug
- **Default Location** — `~/.core/ratelimits.yaml`

#### SQLite Backend (Multi-Process)

- **WAL Mode** — Concurrent read/write support
- **Persistent** — Survives process restarts
- **Scalable** — Handles high request volumes
- **Migration Support** — Can migrate from YAML

### Token Counting

**Gemini-Specific Token Counter:**
- Counts tokens via HTTP API
- Accurate for Gemini models
- Handles batch counting
- Error resilient

### CoreGo Service Integration (`service.go`)

Full CoreGo service with:
- Query handlers for rate limit checks
- Action handlers for usage recording
- Configuration via Core config system
- Lifecycle management

---

## 🔧 Configuration

### Basic Initialization

```go
import "dappco.re/go/core/go-ratelimit"

// YAML backend (default, single-process)
rl, err := ratelimit.New()
if err != nil {
    log.Fatal(err)
}

// SQLite backend (multi-process)
rl, err := ratelimit.NewWithSQLite("/tmp/ratelimits.db")
if err != nil {
    log.Fatal(err)
}
defer rl.Close()
```

### Custom Configuration

```go
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

### Default Provider Profiles

```go
// Get default profiles for all providers
profiles := ratelimit.DefaultProfiles()

// Get profile for specific provider
profile := profiles[ratelimit.ProviderGemini]

// Use in custom configuration
cfg := ratelimit.Config{
    Quotas: profile.Models,
}
```

---

## 🚀 Commands

While primarily a library, go-ratelimit can be integrated into CLI tools:

```bash
# Check if request is allowed
core ratelimit check gemini-2.0-flash 1500

# Record usage
core ratelimit record gemini-2.0-flash 1000 500

# Show current usage
core ratelimit status

# Reset rate limits
core ratelimit reset
```

---

## 📝 Usage Patterns

### 1. Simple Rate Limit Check

```go
import "dappco.re/go/core/go-ratelimit"

rl, _ := ratelimit.New()

// Check if request is allowed
if rl.CanSend("gemini-2.0-flash", 1500) {
    // Send request
    rl.RecordUsage("gemini-2.0-flash", 1000, 500)
} else {
    // Rate limited
    log.Println("Rate limit exceeded")
}
```

### 2. Structured Decision for Agents

```go
decision := rl.Decide("gemini-2.0-flash", 1500)
if !decision.Allowed {
    log.Printf("Throttled (%s); retry after %s", decision.Code, decision.RetryAfter)
    // decision.Code: "rpm_limit", "tpm_limit", "rpd_limit", or "throttled"
    // decision.RetryAfter: time until quota resets
} else {
    // Send request and record usage
    rl.RecordUsage("gemini-2.0-flash", decision.RequestTokens, decision.ResponseTokens)
}
```

### 3. Token Counting

```go
// Count tokens for a prompt (uses HTTP API for Gemini)
tokenCount, err := ratelimit.CountTokens(ctx, "gemini-2.0-flash", "your prompt here")
if err != nil {
    log.Printf("Failed to count tokens: %v", err)
}

// Check and send
if rl.CanSend("gemini-2.0-flash", tokenCount) {
    // Send request
}
```

### 4. Batch Token Counting

```go
// Count tokens for multiple prompts
texts := []string{"prompt 1", "prompt 2", "prompt 3"}
tokenCounts, err := ratelimit.CountTokensBatch(ctx, "gemini-2.0-flash", texts)
if err != nil {
    log.Fatal(err)
}

for i, count := range tokenCounts {
    fmt.Printf("Text %d: %d tokens\n", i, count)
}
```

### 5. Custom Quotas

```go
// Set custom quota for a model
rl.SetQuota("my-custom-model", ratelimit.ModelQuota{
    MaxRPM: 100,
    MaxTPM: 10000,
    MaxRPD: 10000,
})

// Check with custom quota
if rl.CanSend("my-custom-model", 500) {
    rl.RecordUsage("my-custom-model", 250, 250)
}
```

### 6. Persistence

```go
// Manually persist state (YAML backend)
if err := rl.Persist(); err != nil {
    log.Printf("Failed to persist: %v", err)
}

// SQLite backend auto-persists, but can be forced
if err := rl.Persist(); err != nil {
    log.Printf("Failed to persist: %v", err)
}

// Close SQLite connection on shutdown
defer rl.Close()
```

### 7. Migration from YAML to SQLite

```go
// Migrate existing YAML state to SQLite
cfg := ratelimit.Config{
    FilePath:   "/path/to/ratelimits.yaml",
    Backend:    "sqlite",
    SQLitePath: "/path/to/ratelimits.db",
}

rl, err := ratelimit.NewWithConfig(cfg)
if err != nil {
    log.Fatal(err)
}

// State is automatically migrated on first use
```

---

## 🧪 Testing

### Test Structure

Each file follows the AX standard:
- `_test.go` — Unit tests
- `_example_test.go` — Usage examples as tests

### Test Coverage

| File | Functions | Lines | Coverage |
|------|-----------|-------|----------|
| `ratelimit.go` | 20+ | ~500 | >85% |
| `sqlite.go` | 15+ | ~300 | >80% |
| `service.go` | 10+ | ~200 | >80% |

### Test Commands

```bash
# All tests
go test ./...

# With race detector
go test -race ./...

# Specific package
go test -v ./go/...

# With coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

---

## 📖 API Reference

### Core Types

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

// Config - RateLimiter configuration
type Config struct {
    FilePath   string                 `yaml:"file_path,omitempty"`
    Backend    string                 `yaml:"backend,omitempty"`
    Quotas     map[string]ModelQuota `yaml:"quotas,omitempty"`
}

// Decision - Structured rate limit decision
type Decision struct {
    Allowed      bool
    Code        string        // "allowed", "rpm_limit", "tpm_limit", "rpd_limit", "throttled"
    RetryAfter  time.Duration // Time until quota resets
    RequestTokens int
    ResponseTokens int
}
```

### Main Functions

```go
// Initialization
func New() (*RateLimiter, error)
func NewWithSQLite(path string) (*RateLimiter, error)
func NewWithConfig(cfg Config) (*RateLimiter, error)

// Rate limit checks
func (rl *RateLimiter) CanSend(model string, requestTokens int) bool
func (rl *RateLimiter) Decide(model string, requestTokens int) Decision
func (rl *RateLimiter) Remaining(model string) (rpm, tpm, rpd int)

// Usage recording
func (rl *RateLimiter) RecordUsage(model string, requestTokens, responseTokens int)
func (rl *RateLimiter) RecordRequest(model string, requestTokens int)

// Quota management
func (rl *RateLimiter) SetQuota(model string, quota ModelQuota)
func (rl *RateLimiter) GetQuota(model string) ModelQuota
func (rl *RateLimiter) GetQuotas() map[string]ModelQuota

// Token counting
func CountTokens(ctx context.Context, model, text string) (int, error)
func CountTokensBatch(ctx context.Context, model string, texts []string) ([]int, error)

// Persistence
func (rl *RateLimiter) Persist() error
func (rl *RateLimiter) Close() error

// Default profiles
func DefaultProfiles() map[Provider]ProviderProfile
func DefaultConfig() Config
```

### Configuration Functions

```go
// Config helpers
func ConfigFromFile(path string) (Config, error)
func SaveConfig(path string, cfg Config) error
```

---

## 🔗 Related Documentation

### Internal Documentation

| Resource | Description | Location |
|----------|-------------|----------|
| RFC | Package specification | [plans/code/core/go/ratelimit/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/ratelimit/RFC.md) |
| Architecture | Sliding window algorithm, provider quotas, backends | [docs/architecture.md](file:///Users/snider/Code/core/go-ratelimit/docs/architecture.md) |
| Development Guide | Prerequisites, test patterns, coding standards | [docs/development.md](file:///Users/snider/Code/core/go-ratelimit/docs/development.md) |
| Project History | Completed phases, known limitations | [docs/history.md](file:///Users/snider/Code/core/go-ratelimit/docs/history.md) |
| Threat Analysis | Security considerations | [threats.md](file:///Users/snider/Code/core/go-ratelimit/threats.md) |

### External References

| Resource | URL |
|----------|-----|
| Repository | [github.com/dappcore/go-ratelimit](https://github.com/dappcore/go-ratelimit) |
| Module | [pkg.go.dev/dappco.re/go/core/go-ratelimit](https://pkg.go.dev/dappco.re/go/core/go-ratelimit) |
| Sliding Window Algorithm | [Wikipedia](https://en.wikipedia.org/wiki/Sliding_window) |

---

## 📊 Statistics

### Code Metrics

```
Total Repository Size:    ~15 MB
Go Source Files:         8+
Test Files:             8+
Example Files:          4+
Documentation:          1000+ lines

Lines of Code:
  └── Go:              ~1,500
  └── Tests:           ~1,000
  └── Documentation:    ~1,000
  └── Total:           ~3,500
```

### Feature Coverage

| Feature | Status | Details |
|---------|--------|---------|
| Sliding Window Rate Limiting | ✅ Complete | Accurate, efficient, thread-safe |
| RPM Limiting | ✅ Complete | Requests per minute |
| TPM Limiting | ✅ Complete | Tokens per minute |
| RPD Limiting | ✅ Complete | Requests per day |
| YAML Backend | ✅ Complete | Single-process persistence |
| SQLite Backend | ✅ Complete | Multi-process persistence with WAL |
| Token Counting | ✅ Complete | Gemini-specific via HTTP API |
| Batch Token Counting | ✅ Complete | Multiple texts at once |
| Structured Decisions | ✅ Complete | Agent-friendly verdicts |
| Provider Profiles | ✅ Complete | Gemini, OpenAI, Anthropic, Local |
| Custom Quotas | ✅ Complete | Per-model quota configuration |
| State Migration | ✅ Complete | YAML to SQLite migration |
| CoreGo Service | ✅ Complete | Full service integration |

### Test Statistics

| Type | Count | Coverage |
|------|-------|----------|
| Unit Tests | 50+ | >85% |
| Example Tests | 10+ | N/A |
| Total | 60+ | >85% |

### Dependency Statistics

```
Direct Dependencies:    2
  ├── dappco.re/go v0.10.4
  └── gopkg.in/yaml.v3 v3.0.1

Indirect Dependencies: 10+
  ├── golang.org/x/net v0.52.0
  ├── golang.org/x/sys v0.43.0
  ├── golang.org/x/text v0.36.0
  └── gopkg.in/yaml.v3 v3.0.1
```

---

## 🏷️ Tags

#ratelimit #rate-limiting #llm #ai #throttling #quota #sliding-window #rpm #tpm #rpd #yaml #sqlite #persistence #multi-process #token-counting #gemini #openai #anthropic #local-inference #agent-first #ax-standard #provider-agnostic

---

*Last updated: 2026-06-18 | Maintainer: Purberus <purberus@lthn.ai>*
