---
type: Knowledge Pack
title: CoreGo Framework
description: Complete knowledge pack for CoreGo — the zero-dependency Go framework for the Lethean ecosystem
author: Mistral Vibe
version: 1.0.0
created: 2026-06-17T14:00:00Z
tags: [framework, golang, zero-dependency, core, lethean]
---

# CoreGo Knowledge Pack

> **"The reference implementation — every shape decision propagates to ~30 downstream repos"**

This knowledge pack contains everything an agent needs to understand, use, and contribute to CoreGo — the foundational Go framework for the Lethean ecosystem.

---

## 🎯 Overview

**CoreGo** is a zero-dependency Go framework that provides:

- **Shared primitives** for all dAppCore repositories
- **Standard error handling** via `core.Result`
- **Panic recovery** middleware
- **Simplified downstream code** through consistent patterns
- **AX Standard** compliance (Agents-first development)

### Key Statistics

- **Total packages:** 50+ (see [INDEX.md](INDEX.md))
- **Stdlib packages wrapped:** 44 (from earlier exploration)
- **SPOR Rule:** Single Point Of Responsibility — each stdlib package has exactly ONE owner file
- **Downstream impact:** ~30+ repositories depend on CoreGo

---

## 📚 Documentation Structure

```
knowledge-packs/corego/
├── README.md              # This file — knowledge pack overview
├── INDEX.md               # Complete package catalog
├── SPEC.md                # CoreGo specification (if not in plans)
├── okf/
│   ├── index.md           # OKF bundle root
│   ├── log.md             # Change history
│   ├── framework/         # Framework-level concepts
│   │   ├── result.md      # core.Result pattern
│   │   ├── panic-recovery.md # Panic handling
│   │   └── ax-standard.md # AX Standard compliance
│   ├── packages/          # Individual package specs
│   │   ├── io.md          # I/O package
│   │   ├── json.md        # JSON utilities
│   │   ├── error.md       # Error handling
│   │   └── ... (50+ packages)
│   └── patterns/          # Common patterns
│       ├── spor.md       # Single Point Of Responsibility
│       ├── triplets.md    # Test triplet pattern
│       └── comments.md   # Comments-for-agents
└── examples/
    ├── basic-usage.md     # Getting started
    ├── error-handling.md  # Using core.Result
    └── testing.md        # Test triplet examples
```

---

## 🔗 Source of Truth

The **canonical specification** lives in:
- [`~/Code/meowmix/plans/code/core/go/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)
- [`~/Code/core/go/AGENTS.md`](file:///Users/snider/Code/core/go/AGENTS.md)

This knowledge pack **aggregates and organizes** that information for agent consumption.

---

## 📦 Core Concepts

### 1. The Result Pattern

**Purpose:** Standard error handling with logging and panic recovery

```go
// core.Result wraps values with error context
result := core.OK(value)           // Success
result := core.E(err)              // Error
result := core.Panic(err, msg)    // Panic with context

// Automatic logging and recovery
result.Log()                      // Logs error if present
result.Panic()                    // Panics with context
result.Unwrap()                   // Extract value or panic
```

**AX Standard:** Every function returns `core.Result[T]` instead of `(T, error)`

### 2. Panic Recovery

**Purpose:** Graceful degradation when panics occur

```go
// Automatic recovery in handlers
func handler() core.Result {
    defer core.Recover(&result)  // Catches panics, converts to error
    // ... code that may panic
}
```

**Key insight:** Downstream code is simpler because panics are automatically recovered and logged.

### 3. SPOR (Single Point Of Responsibility)

Each stdlib package is wrapped by **exactly one file** in CoreGo:

| Stdlib Package | Owner File | CoreGO Wrapper |
|----------------|------------|----------------|
| `fmt` | `format.go` | `core.Sprintf` |
| `io` | `io.go` | `core.Reader` |
| `json` | `json.go` | `core.JSONMarshal` |
| `http` | `api.go` | `core.Request` |

**Why it matters:** Prevents duplicate wrappers and ensures consistency across the ecosystem.

### 4. Test Triplets

Every package has three files:

```
pkg/
├── file.go              # Implementation
├── file_test.go         # Unit tests (Good/Bad/Ugly pattern)
└── file_example_test.go # Example tests
```

**Pattern:** Good tests (happy path), Bad tests (error cases), Ugly tests (edge cases)

---

## 🗂️ Package Catalog

See [INDEX.md](INDEX.md) for the complete list of 50+ packages.

### Core Packages

| Package | Purpose | Files |
|---------|---------|-------|
| `action` | Action execution framework | action.go, action_test.go |
| `api` | HTTP client/server | api.go, api_test.go |
| `context` | Context utilities | context.go |
| `error` | Error handling | error.go |
| `format` | String formatting | format.go |
| `io` | I/O operations | io.go |
| `json` | JSON encoding/decoding | json.go |
| `log` | Structured logging | log.go |
| `result` | Result type and panic recovery | result.go |

### I/O Packages

| Package | Purpose | Backend |
|---------|---------|---------|
| `go-io` | Storage-agnostic I/O | 8 backends |
| `go-cache` | Caching layer | go-io medium |
| `go-store` | SQLite key-value store | |

### Blockchain Packages

| Package | Purpose | Status |
|---------|---------|--------|
| `go-blockchain` | Blockchain implementation | Production |
| `go-lns` | Lethean Name System | Production |
| `go-miner` | Mining operations | Production |

### AI/ML Packages

| Package | Purpose | Backend |
|---------|---------|---------|
| `go-ai` | AI operations | |
| `go-ml` | ML inference | CPU |
| `go-mlx` | Apple Metal GPU | Production |
| `go-rocm` | AMD GPU | Production |
| `go-cuda` | NVIDIA GPU | Scaffold |
| `go-tpu` | Google TPU | Scaffold |
| `go-inference` | Shared ML interfaces | |

---

## 🚀 Getting Started

### For Agents

1. **Read the RFC:** Start with [`plans/code/core/go/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)
2. **Explore AGENTS.md:** [`core/go/AGENTS.md`](file:///Users/snider/Code/core/go/AGENTS.md)
3. **Browse packages:** See [INDEX.md](INDEX.md)
4. **Use Result:** Always return `core.Result[T]`
5. **Follow SPOR:** Each stdlib package has one owner

### For Developers

1. **Clone the repo:** `git clone forge.lthn.sh/core/go`
2. **Read the RFC:** Understand the contract
3. **Use core.Result:** For all error handling
4. **Write triplets:** Implementation + tests + examples
5. **Follow AX Standard:** Comments for agents, not humans

---

## 🔍 Exploring the Codebase

### Key Files

- [`core/go/AGENTS.md`](file:///Users/snider/Code/core/go/AGENTS.md) — Agent-specific guidance
- [`core/go/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md) — Primary specification
- [`core/go/result.go`](file:///Users/snider/Code/core/go/result.go) — Result type implementation
- [`core/go/api.go`](file:///Users/snider/Code/core/go/api.go) — HTTP utilities

### Discovery Commands

```bash
# List all packages
ls -1 /Users/snider/Code/core/go/*.go | sed 's/.go$//' | sort

# Count files
ls /Users/snider/Code/core/go/*.go | wc -l

# Find test triplets
for file in /Users/snider/Code/core/go/*.go; do
    base=$(basename $file .go)
    if [ -f "${file}_test.go" ] && [ -f "${file}_example_test.go" ]; then
        echo "✓ $base has complete triplet"
    fi
done
```

---

## 📊 Quick Stats

```
Total Go files:        160+ (from earlier exploration)
Total stdlib wrapped:   44
Total downstream repos: ~30+
SPOR compliance:       100% (each stdlib package has one owner)
Test triplet coverage:  High (most packages have triplets)
```

---

## 🎯 Use Cases

### When to Use CoreGo

✅ **New dAppCore repository** — Always start with CoreGo
✅ **Error handling** — Use `core.Result[T]` instead of `(T, error)`
✅ **Panic recovery** — Use `core.Recover` in handlers
✅ **I/O operations** — Use `go-io` backends instead of raw file operations
✅ **HTTP clients** — Use `core.Request` instead of raw `http.Client`

### When NOT to Use CoreGo

❌ **Performance-critical code** — CoreGo adds minimal overhead but may not be suitable for extreme performance requirements
❌ **External libraries** — Don't wrap external library calls in CoreGo (use the library directly)
❌ **Stdlib packages already wrapped** — Don't re-wrap; use the existing CoreGo wrapper

---

## 🔗 Related Knowledge Packs

- [CoreGUI](../coregui/README.md) — GUI framework (Wails v2)
- [CoreTS](../corets/README.md) — TypeScript framework
- [CoreCLI](../corecli/README.md) — CLI framework
- [CorePlay](../coreplay/README.md) — Play framework (Go backend + TS frontend)
- [CorePHP](../corephp/README.md) — PHP framework

---

## 💡 Agent Tips

1. **Always check the RFC first** — The spec drives the code
2. **Follow SPOR** — Don't duplicate stdlib wrappers
3. **Use core.Result** — Never return raw `(T, error)`
4. **Write triplets** — Every new package needs `_test.go` and `_example_test.go`
5. **Comments for agents** — Write documentation that agents can parse

---

## 📝 Maintenance

This knowledge pack is maintained by Mistral Vibe. Updates are triggered by:

- Changes to `plans/code/core/go/`
- Changes to `core/go/` repository
- New downstream repository requirements
- AX Standard updates

---

*Knowledge Pack v1.0.0*
*Created: 2026-06-17T14:00:00Z*
*Author: Mistral Vibe*
*Source: Lethean CoreGo Framework*
