---
type: Index
title: CoreGo Package Catalog
description: Complete list of CoreGo packages with their purposes and relationships
---

# CoreGo Package Catalog

> **Repository:** `dappco.re/core/go`
> **Total Packages:** 50+ (CoreGo framework + related packages)
> **Last Updated:** 2026-06-17

---

## 🎯 Quick Navigation

- [Framework Packages](#-framework-packages) — Core framework primitives
- [I/O Packages](#-io-packages) — Storage and I/O operations
- [Blockchain Packages](#-blockchain-packages) — Blockchain implementation
- [AI/ML Packages](#-ai-ml-packages) — Machine learning backends
- [Network Packages](#-network-packages) — Network operations
- [Build Packages](#-build-packages) — Build and release tools
- [Other Packages](#-other-packages) — Miscellaneous utilities

---

## 🏗️ Framework Packages

Core framework primitives that form the foundation of CoreGo.

| Package | Description | Key Files | Status |
|---------|-------------|-----------|--------|
| [core](file:///Users/snider/Code/core/go/core.go) | **The Foundation** — 272 files, 100% SPOR, zero deps | core.go, options.go, result.go, service.go | ✅ **Complete** |
| [action](file:///Users/snider/Code/core/go/action.go) | Action execution framework | action.go, action_test.go, action_example_test.go | ✅ Complete |
| [api](file:///Users/snider/Code/core/go/api.go) | REST framework + OpenAPI client | api.go, api_bench_test.go, api_example_test.go | ✅ Complete |
| [context](file:///Users/snider/Code/core/go/context.go) | Context utilities | context.go | ✅ Complete |
| [error](file:///Users/snider/Code/core/go/error.go) | Error handling with SPOR | error.go | ✅ Complete |
| [format](file:///Users/snider/Code/core/go/format.go) | String formatting (fmt wrapper) | format.go | ✅ Complete |
| [io](file:///Users/snider/Code/core/go/io.go) | I/O operations wrapper | io.go | ✅ Complete |
| [json](file:///Users/snider/Code/core/go/json.go) | JSON encoding/decoding | json.go | ✅ Complete |
| [log](file:///Users/snider/Code/core/go/log.go) | Structured logging | log.go | ✅ Complete |
| [result](file:///Users/snider/Code/core/go/result.go) | **Result pattern + panic recovery** | result.go | ✅ **Critical** |
| [reflect](file:///Users/snider/Code/core/go/reflect.go) | Reflection utilities | reflect.go | ✅ Complete |
| [regexp](file:///Users/snider/Code/core/go/regexp.go) | Regex utilities | regexp.go | ✅ Complete |

---

## 📁 I/O Packages

Storage-agnostic I/O operations with multiple backends.

| Package | Description | Backends | Repository |
|---------|-------------|---------|------------|
| [go-io](file:///Users/snider/Code/core/go-io/) | **Mandatory I/O abstraction** | 8 backends | `dappco.re/go/io` |
| [go-cache](file:///Users/snider/Code/core/go-cache/) | Caching layer | go-io medium | `dappco.re/go/cache` |
| [go-store](file:///Users/snider/Code/core/go-store/) | SQLite key-value store | | `dappco.re/go/store` |
| [go-container](file:///Users/snider/Code/core/go-container/) | LinuxKit + TIM container runtime | | `dappco.re/go/container` |
| [go-stream](file:///Users/snider/Code/core/go-stream/) | **Transport-agnostic streaming** | 5 transports | `dappco.re/go/stream` |
| [go-crypt](file:///Users/snider/Code/core/go-crypt/) | **Cryptography & Trust** | 3 subsystems, 5 CLI commands | `dappco.re/go/crypt` |

**go-io backends:** S3, SQLite, local filesystem, HTTP, and more

---

## 🔗 Blockchain Packages

Lethean blockchain implementation and related services.

| Package | Description | Status | Repository |
|---------|-------------|--------|------------|
| [go-blockchain](file:///Users/snider/Code/core/go-blockchain/) | Core blockchain (CryptoNote+) | Production | `dappco.re/go/blockchain` |
| [go-lns](file:///Users/snider/Code/core/go-lns/) | **Lethean Name System** (native CoreGO) | Production | `dappco.re/go/lns` |
| [go-miner](file:///Users/snider/Code/core/go-miner/) | Mining operations | Production | `dappco.re/go/miner` |
| [go-p2p](file:///Users/snider/Code/core/go-p2p/) | P2P networking (node, levin, UEPS) | Production | `dappco.re/go/p2p` |
| [go-dns](file:///Users/snider/Code/core/go-dns/) | DNS resolver + server for .lthn TLD | Production | `dappco.re/go/dns` |
| [go-hsd](file:///Users/snider/Code/meowmix/plans/code/core/go/hsd/) | **Handshake Sidechain JSON-RPC client** | ✅ **NEW** | `dappco.re/go/hsd` |

---

## 🤖 AI/ML Packages

Machine learning inference backends and utilities.

| Package | Description | Backend | Status | Repository |
|---------|-------------|---------|--------|------------|
| [go-inference](file:///Users/snider/Code/core/go-inference/) | **Shared ML interfaces** (TextModel, Backend) | | Production | `dappco.re/go/inference` |
| [go-ml](file:///Users/snider/Code/core/go-ml/) | ML inference backends, scoring | CPU | Production | `dappco.re/go/ml` |
| [go-mlx](file:///Users/snider/Code/core/go-mlx/) | **Native Apple Metal GPU** | MLX | Production | `dappco.re/go/mlx` |
| [go-rocm](file:///Users/snider/Code/core/go-rocm/) | **AMD GPU (ROCm/HIP)** | ROCm | Production | `dappco.re/go/rocm` |
| [go-cuda](file:///Users/snider/Code/core/go-cuda/) | **NVIDIA GPU (CUDA)** | CUDA | ⚠️ Scaffold | `dappco.re/go/cuda` |
| [go-tpu](file:///Users/snider/Code/core/go-tpu/) | **Google TPU (PJRT/XLA)** | TPU | ⚠️ Scaffold | `dappco.re/go/tpu` |
| [go-rag](file:///Users/snider/Code/core/go-rag/) | RAG (Qdrant, Ollama embeddings) | | | `dappco.re/go/rag` |
| [go-ai](file:///Users/snider/Code/core/go-ai/) | AI metrics, RAG, security | | | `dappco.re/go/ai` |

**Backend Comparison:**
- MLX: Apple Silicon (Mac), optimized for gemma4
- ROCm: AMD GPU (Linux), production ready
- CUDA: NVIDIA GPU, hardware-light scaffold (gap filled)
- TPU: Google Cloud TPU, narrow surface scaffold (gap filled)

---

## 🌐 Network Packages

Network operations and protocols.

| Package | Description | Repository |
|---------|-------------|------------|
| [go-netops](file:///Users/snider/Code/core/go-netops/) | UniFi network controller client | `dappco.re/go/netops` |
| [go-proxy](file:///Users/snider/Code/core/go-proxy/) | Proxy server | `dappco.re/go/proxy` |
| [go-io](file:///Users/snider/Code/core/go-io/) | Storage-agnostic I/O (includes HTTP backend) | `dappco.re/go/io` |
| [go-stream](file:///Users/snider/Code/core/go-stream/) | **Transport-agnostic streaming** (WebSocket, SSE, Redis, ZMQ, TCP) | `dappco.re/go/stream` |
| [go-crypt](file:///Users/snider/Code/core/go-crypt/) | **Cryptography & Trust** (ChaCha20, AES, Argon2id, OpenPGP, 3-tier access control) | `dappco.re/go/crypt` |

---

## 🔨 Build Packages

Build, release, and development tools.

| Package | Description | Repository |
|---------|-------------|------------|
| [go-build](file:///Users/snider/Code/core/go-build/) | Build, release, signing, SDK generation | `dappco.re/go/build` |
| [go-config](file:///Users/snider/Code/core/go-config/) | Configuration (dual-Viper pattern) | `dappco.re/go/config` |
| [go-devops](file:///Users/snider/Code/core/go-devops/) | Deploy, multi-repo workflow | `dappco.re/go/devops` |
| [go-git](file:///Users/snider/Code/core/go-git/) | Multi-repo git operations | `dappco.re/go/git` |
| [go-forge](file:///Users/snider/Code/core/go-forge/) | Forgejo API client (~450 endpoints) | `dappco.re/go/forge` |
| [go-scm](file:///Users/snider/Code/core/go/scm/) | Multi-repo management, marketplace | `dappco.re/go/scm` |

---

## 📊 Standard Library Wrappers (44 packages)

Each wrapped by a single file following SPOR (Single Point Of Responsibility).

| # | Stdlib Package | Owner File | CoreGO Wrapper |
|---|----------------|------------|----------------|
| 1-5 | `bufio`, `bytes`, `cmp`, `compress/gzip`, `context` | scanner.go, io.go, math.go, embed.go, context.go | Various |
| 6-10 | `crypto/hkdf`, `crypto/hmac`, `crypto/rand`, `crypto/sha256`, `crypto/sha512` | hash.go, random.go | `core.HKDF`, `core.HMAC`, etc. |
| 11-15 | `crypto/sha3`, `database/sql`, `embed`, `encoding/base64`, `encoding/binary` | sha3.go, sql.go, embed.go, encode.go | Various |
| 16-20 | `encoding/hex`, `encoding/json`, `errors`, `fmt`, `go/ast` | encode.go, json.go, error.go, format.go, embed.go | Various |
| 21-25 | `go/parser`, `go/token`, `hash`, `html`, `html/template` | embed.go, string.go, template.go | Various |
| 26-30 | `text/template`, `io`, `io/fs`, `iter`, `math` | template.go, io.go, fs.go, iter.go, math.go | Various |
| 31-35 | `math/big`, `math/bits`, `math/rand/v2`, `mime/multipart`, `net` | math.go, sha3.go, random.go, api.go, net.go | Various |
| 36-40 | `net/http`, `net/http/httptest`, `net/url`, `os`, `os/exec` | api.go, os.go, process.go | Various |
| 41-44 | `os/user`, `path/filepath`, `reflect`, `regexp` | user.go, path.go, reflect.go, regexp.go | Various |

---

## 🎯 Additional Packages

| Package | Description | Repository |
|---------|-------------|------------|
| [go-ansible](file:///Users/snider/Code/core/go-ansible/) | Pure Go Ansible engine (41 modules) | `dappco.re/go/ansible` |
| [go-agent](file:///Users/snider/Code/core/go-agent/) | Agent dispatch, workspace, MCP, fleet | `dappco.re/go/agent` |
| [go-api](file:///Users/snider/Code/core/go-api/) | REST framework + OpenAPI client | `dappco.re/go/api` |
| [go-app](file:///Users/snider/Code/core/app/) | Application framework | `dappco.re/core/app` |
| [go-doc](file:///Users/snider/Code/core/docs/) | Documentation framework | `dappco.re/core/docs` |
| [go-ide](file:///Users/snider/Code/core/core-agent-ide/) | Agent IDE | `dappco.re/core/core-agent-ide` |

---

## 📈 Statistics

```
Total CoreGo packages:      15+ (framework)
Total go-* packages:        48 repos (from repo scan)
Total stdlib wrapped:       44
SPOR compliance:            100%
Test triplet coverage:      High
Downstream repositories:    ~30+
Package deep dives:         35 (core, go-lns, go-blockchain, go-dns, go-io, go-p2p, go-proxy, go-pool, go-process, go-i18n, go-log, go-cache, go-session, go-webview, go-ws, go-agent, go-gui, go-ide, go-inference, go-ml, go-config, go-build, go-ai, go-ansible, go-cgo, go-container, go-stream, go-crypt, go-store, go-miner, go-devops, go-forge, go-git, go-rag, go-ratelimit, go-tenant)
```

---

## 🔍 Gap Analysis

### ✅ Recently Filled Gaps

| Gap | Action | Date | Commit |
|-----|--------|------|--------|
| go-cuda RFC | Created [RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/cuda/RFC.md) | 2026-06-17 | 5557cf6 |
| go-tpu RFC | Created [RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/tpu/RFC.md) | 2026-06-17 | 5557cf6 |
| go/array RFC | Created [RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/array/RFC.md) | 2026-06-17 | 3912ef3 |
| go-hsd RFC | Created [RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/hsd/RFC.md) | 2026-06-17 | 5b55199 |

### ✅ All go-* Package RFCs Complete

All 42 go-* packages now have RFCs in `plans/code/core/go/<package>/`:
- All packages verified against code repos (excluding .wiki and codex-mantis variants)
- No missing RFCs identified

### 🔍 Quality Checks

- [x] All go-* packages have RFCs
- [x] INDEX.md updated with go-hsd
- [x] Codex-mantis variants identified (no separate RFCs needed)
- [ ] Verify all packages have test triplets (_test.go + _example_test.go)
- [ ] Check for packages with RFC but no code

---

## 📚 Package Deep Dives

Detailed documentation for specific packages:

| Package | Documentation | Description |
|---------|---------------|-------------|
| [core](./pkg/core/README.md) | ✅ **Complete** | **CoreGo Framework** — Zero-dep foundation, DI, service lifecycle, Result pattern, IPC, 272 files, 100% SPOR compliant |
| [go-lns](./pkg/lns/README.md) | ✅ **Complete** | Lethean Name System — full API, examples, architecture |
| [go-blockchain](./pkg/blockchain/README.md) | ✅ **Complete** | Lethean Blockchain — CryptoNote implementation, P2P, wallet, mining |
| [go-dns](./pkg/dns/README.md) | ✅ **Complete** | .lthn DNS Resolution — authoritative DNS server, HSD integration, tree-root cache |
| [go-io](./pkg/io/README.md) | ✅ **Complete** | **Mandatory I/O Abstraction** — universal transport layer, 10+ backends, sigil transformations |
| [go-p2p](./pkg/p2p/README.md) | ✅ **Complete** | **Peer-to-Peer Networking** — WebSocket transport, SMSG encryption, Poindexter KD-tree, Levin protocol |
| [go-proxy](./pkg/proxy/README.md) | ✅ **Complete** | **Stratum Mining Proxy** — NiceHash nonce-splitting, TCP/TLS, pool failover, monitoring API |
| [go-pool](./pkg/pool/README.md) | ✅ **Complete** | **Mining Pool Backend** — ProgPoWZ, Stratum, vardiff, Redis state, RBPPS payouts |
| [go-process](./pkg/process/README.md) | ✅ **Complete** | **Process Orchestration** — Daemon management, pipeline execution, health checks, cross-platform support |
| [go-i18n](./pkg/i18n/README.md) | ✅ **Complete** | **Grammar-Aware i18n** — Semantic intent, GrammarImprint, dual-class disambiguation, CLDR pluralization |
| [go-log](./pkg/log/README.md) | ✅ **Complete** | **Structured Logging & Error Handling** — Mandatory error creation, log levels, redaction, rotation |
| [go-cache](./pkg/cache/README.md) | ✅ **Complete** | **Storage-Agnostic Caching** — JSON cache with TTL, invalidation, HTTP caching, GitHub helpers |
| [go-session](./pkg/session/README.md) | ✅ **Complete** | **Claude Code Parser** — Session parsing, analytics, HTML/Video rendering, search |
| [go-webview](./pkg/webview/README.md) | ✅ **Complete** | **Browser Automation** — Chrome DevTools Protocol, Angular helpers, action pattern |
| [go-ws](./pkg/ws/README.md) | ✅ **Complete** | **WebSocket Hub** — Real-time streaming, authentication, Redis bridge, reconnecting client |
| [go-agent](./pkg/agent/README.md) | ✅ **Complete** | **AI Agent Orchestration** — Dispatch, MCP server, fleet management, multi-provider support |
| [go-gui](./pkg/gui/README.md) | ✅ **Complete** | **HUI Desktop GUI Framework** — Wails v3, Angular 20, 21 services, window management, MCP tools |
| [go-ide](./pkg/ide/README.md) | ✅ **Complete** | **Lethean Desktop IDE** — Thin Wails shell, 19 MCP tools, 4 modes, Vi Control Panel, multi-platform |
| [go-inference](./pkg/inference/README.md) | ✅ **Complete** | **Shared ML Inference Interfaces** — Zero-dep contract, backend registry, TextModel/Backend interfaces, streaming |
| [go-ml](./pkg/ml/README.md) | ✅ **Complete** | **ML Inference Backends & Scoring Engine** — Dual-interface, HTTP/Llama backends, 23 probes, GGUF support, agent orchestrator |
| [go-config](./pkg/config/README.md) | ✅ **Complete** | **Dual-Viper Configuration** — Hierarchical discovery, manifests, feature flags, watching, XDG compliance |
| [go-build](./pkg/build/README.md) | ✅ **Complete** | **Build Orchestration** — 12 builders, 8 publishers, SDK generation, Apple ecosystem, workflow automation |
| [go-ai](./pkg/ai/README.md) | ✅ **Complete** | **AI Integration Layer** — MCP server, metrics logging, RAG facade, provider routing, 50+ tools |
| [go-ansible](./pkg/ansible/README.md) | ✅ **Complete** | **Pure Go Ansible Engine** — 45 module handlers, SSH transport, YAML parsing, Jinja2 templating, role support |
| [go-cgo](./pkg/cgo/README.md) | ✅ **Complete** | **Standard CGo Harness** — Buffer, Scope, Call, type conversions, memory-safe C interop for go-blockchain, go-mlx, go-rocm |
| [go-devops](./pkg/devops/README.md) | ✅ **Complete** | **DevOps Orchestration** — Ansible executor, multi-target build, code signing, release orchestration, Hetzner/CloudNS APIs, SDK generation, devkit tooling |
| [go-forge](./pkg/forge/README.md) | ✅ **Complete** | **Forgejo API Client** — ~450 endpoints for repos, issues, PRs, projects, orgs, users, webhooks, releases, admin, CI/CD actions |
| [go-git](./pkg/git/README.md) | ✅ **Complete** | **Multi-Repository Git Operations** — Parallel status/push/pull, CoreGo service integration, path validation, iterator-based streaming |
| [go-container](./pkg/container/README.md) | ✅ **Complete** | **Docker-Free Container Runtime** — LinuxKit (default), TIM (experimental), VZ (Apple), dev environments, I/O Medium integration |
| [go-stream](./pkg/stream/README.md) | ✅ **Complete** | **Unified Stream Primitive** — Transport-agnostic hub (WebSocket, SSE, Redis, ZeroMQ, TCP), 5 adapters, reconnecting clients, CoreGO service integration |
| [go-crypt](./pkg/crypt/README.md) | ✅ **Complete** | **Cryptographic Primitives & Agent Trust** — ChaCha20-Poly1305, AES-256-GCM, Argon2id KDF, OpenPGP auth, 3-tier access control, capability policies, approval workflows, audit trails |
| [go-store](./pkg/store/README.md) | ✅ **Complete** | **SQLite Key-Value Store** — TTL expiry (triple-layered), namespace isolation via ScopedStore, reactive Watch/OnChange events, DuckDB workspace buffering, InfluxDB time-series journal, compressed JSONL cold archive, io.Medium transport abstraction |
| [go-miner](./pkg/miner/README.md) | ✅ **Complete** | **Mining Software Controller** — Unified Miner interface for XMRig/TT-Miner/Simulated, Manager for multi-miner lifecycle, ProfileManager with go-store, EventBus via go-stream, CircuitBreaker, RateLimiter, TaskSupervisor, LogBuffer |
| [go-rag](./pkg/rag/README.md) | ✅ **Complete** | **RAG (Retrieval-Augmented Generation)** — Qdrant vector database, Ollama embeddings, chunking, indexing, similarity search, query augmentation, multi-backend support |
| [go-ratelimit](./pkg/ratelimit/README.md) | ✅ **Complete** | **Rate Limiting Service** — Token bucket, sliding window, fixed window algorithms, Redis backend, middleware, multi-tenant support, configurable burst handling |
| [go-tenant](./pkg/tenant/README.md) | ✅ **Complete** | **Multi-Tenant Service** — Tenant isolation, scoped resource management, middleware, CoreGO service integration, Redis state backend, partition-aware routing |

More package deep dives coming soon...

---

## 📝 Notes

- **Repository:** `forge.lthn.sh/core/go`
- **Primary Spec:** [`plans/code/core/go/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)
- **Agent Guide:** [`core/go/AGENTS.md`](file:///Users/snider/Code/core/go/AGENTS.md)
- **Maintained by:** Purberus <purberus@lthn.ai> (knowledge pack)

---

*Package catalog last updated: 2026-06-17T21:45:00Z*
*Knowledge Pack: CoreGo v1.3.0*
*Purrr... Core framework documented!* 🐱
*Purrr... Documentation by Purberus <purberus@lthn.ai>*
