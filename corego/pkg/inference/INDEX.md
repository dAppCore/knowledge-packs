---
type: Package Index
package: inference
module: dappco.re/go/inference
repo: core/go-inference
title: go-inference Package Index
description: Shared ML inference interfaces for text generation backends - zero-dep contract with backend registry, streaming support, and capability detection
tags:
  - ai
  - inference
  - machine-learning
  - llm
  - text-generation
  - backend-interface
  - shared-contract
  - zero-deps
  - stdlib-only
  - streaming
  - go1.23-plus
---
# go-inference Package Index

> **Shared ML Inference Interfaces — The unified contract for text generation backends**

**Repository:** `core/go-inference`  
**Module:** `dappco.re/go/inference`  
**Status:** ✅ Production-Ready  
**License:** EUPL-1.2  
**Language:** Go 1.25+  
**Dependencies:** Zero external (stdlib only)  
**RFC:** [plans/code/core/go/inference/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/inference/RFC.md)  
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| RFC | Request for Comments specification | [plans/code/core/go/inference/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/inference/RFC.md) |
| CLAUDE.md | Development guidance | [~/Code/core/go-inference/CLAUDE.md](file:///Users/snider/Code/core/go-inference/CLAUDE.md) |
| AGENTS.md | Agent guidance | [~/Code/core/go-inference/AGENTS.md](file:///Users/snider/Code/core/go-inference/AGENTS.md) |
| GOAL.md | Quality findings and remediation | [~/Code/core/go-inference/GOAL.md](file:///Users/snider/Code/core/go-inference/GOAL.md) |
| Architecture | Design rationale and structure | [~/Code/core/go-inference/docs/architecture.md](file:///Users/snider/Code/core/go-inference/docs/architecture.md) |
| Development Guide | Building, testing, standards | [~/Code/core/go-inference/docs/development.md](file:///Users/snider/Code/core/go-inference/docs/development.md) |
| History | Project phases and limitations | [~/Code/core/go-inference/docs/history.md](file:///Users/snider/Code/core/go-inference/docs/history.md) |

---

## 🎯 Package Overview

`go-inference` is the **shared interface contract** for text generation backends in the Core Go ecosystem. It defines the fundamental `TextModel`, `Backend`, `Token`, `Message`, and associated configuration types that all GPU-specific backends implement and consumers depend on.

### Design Philosophy

- **Pure Interface Package** — No implementations, only contracts
- **Zero Dependencies** — Stdlib only, compiles on all platforms
- **Backend-Agnostic** — Multiple backends can be swapped transparently
- **Platform-Aware** — Automatic backend selection based on platform
- **Stable Contract** — Strict rules govern changes to shared interfaces

### Backend Ecosystem

| Backend | Repository | Platform | Status | Priority |
|---------|------------|----------|--------|----------|
| **Metal (MLX)** | [core/go-mlx](file:///Users/snider/Code/core/go-mlx/) | macOS darwin/arm64 | ✅ Production | 1 (Highest) |
| **ROCm** | [core/go-rocm](file:///Users/snider/Code/core/go-rocm/) | Linux amd64 | ✅ Production | 2 |
| **CPU (llama.cpp)** | [core/go-ml](file:///Users/snider/Code/core/go-ml/) | Cross-platform | ✅ Production | 3 |
| **CUDA** | [core/go-cuda](file:///Users/snider/Code/core/go-cuda/) | Linux/Windows | ⚠️ Scaffold | 4 |
| **TPU** | [core/go-tpu](file:///Users/snider/Code/core/go-tpu/) | Google Cloud | ⚠️ Scaffold | 5 |

---

## 🏗️ Architecture

### Component Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    go-inference: Contract Layer                               │
│  Zero external dependencies, stdlib only, universal compatibility             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    CORE INTERFACES                                     │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │ │
│  │  │ TextModel    │  │   Backend    │  │   Token      │                  │ │
│  │  │             │  │              │  │              │                  │ │
│  │  │ Generate()   │  │ LoadModel()  │  │ Text        │                  │ │
│  │  │ Chat()       │  │ Supports()   │  │ Role        │                  │ │
│  │  │ Close()      │  │ HealthCheck()│  │ Err         │                  │ │
│  │  │ Info()       │  │              │  │              │                  │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                  │ │
│  │                                                                    │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │ │
│  │  │  Message     │  │ ClassifyResult│  │ BatchResult  │                  │ │
│  │  │             │  │              │  │              │                  │ │
│  │  │ Role        │  │ Score        │  │ Results     │                  │ │
│  │  │ Content     │  │ Label        │  │ Errors      │                  │ │
│  │  │ Name        │  │ Reason       │  │              │                  │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                  │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    CONFIGURATION TYPES                                       │
│  ┌─────────────────────────┐  ┌─────────────────────────┐                │
│  │    GenerateConfig         │  │       LoadConfig           │                │
│  │  MaxTokens                │  │    Backend                 │                │
│  │  Temperature              │  │    Quantization            │                │
│  │  TopP                    │  │    Device                  │                │
│  │  TopK                    │  │    NumGPU                  │                │
│  │  RepeatPenalty           │  │    NumThread               │                │
│  │  StopSequences           │  │    ...                     │                │
│  │  Stream                 │  │                             │                │
│  └─────────────────────────┘  └─────────────────────────┘                │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    REGISTRY SYSTEM                                           │
│  BackendRegistry map[string]Backend                                         │
│  Register(name, backend)     List() []string                                │
│  Get(name) Backend           Default() string                                 │
│  LoadModel(path, opts...) (TextModel, error)                                 │
│  Priority: metal > rocm > llama_cpp > any available                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                    OPTIONAL INTERFACES                                       │
│  ┌──────────────────────┐  ┌──────────────────────┐                      │
│  │  AttentionInspector   │  │   TrainableModel      │                      │
│  │  - GetAttentionPattern() │  │   - Train()           │                      │
│  │                        │  │   - SaveCheckpoint()  │                      │
│  └──────────────────────┘  └──────────────────────┘                      │
│  ┌──────────────────────┐  ┌──────────────────────┐                      │
│  │  Adapter              │  │   StateTracker        │                      │
│  │  - Adapt()            │  │   - Track()            │                      │
│  └──────────────────────┘  └──────────────────────┘                      │
│  Discovered via type assertion, not via extending core interfaces           │
├─────────────────────────────────────────────────────────────────────────────┤
│                    BACKEND IMPLEMENTATIONS (External)                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │  go-mlx     │  │  go-rocm     │  │   go-ml      │  │  go-cuda     │       │
│  │  (Metal)    │  │  (AMD ROCm)  │  │  (CPU/Llama) │  │  (NVIDIA)   │       │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘       │
└─────────────────────────────────────────────────────────────────────────────┘
```

### File Structure

```
go/
├── inference.go           # Core interfaces: TextModel, Backend, Token, Message, ClassifyResult, BatchResult
├── options.go            # Configuration: GenerateConfig, LoadConfig, With*() functional options
├── discover.go           # Model discovery: Discover() scans for model directories
├── training.go           # Training support: TrainableModel, Adapter interfaces
├── capability.go         # Capability detection helpers
├── contracts_test.go     # Interface compliance tests
├── inference_test.go     # Core tests + stub implementations
├── capability_test.go   # Capability detection tests
├── options_test.go       # Configuration tests
├── training_test.go      # Training tests
├── decode/               # Model decoding: GGUF, Safetensors
│   ├── decode.go
│   ├── gguf.go
│   └── safetensors.go
├── parser/               # Model parsing
│   └── parser.go
├── model/                # Model utilities
│   └── pack/
├── state/                # State management
│   ├── state.go
│   └── ...
├── quant/                # Quantization handling
│   └── quant.go
├── scheduler/            # Inference request scheduling
│   └── scheduler.go
├── service.go            # Service layer integration
├── probe.go              # Health probing
├── split.go              # Token splitting
├── eval/                 # Evaluation and benchmarking
│   └── ...
└── bench/                # Performance benchmarking
    └── ...
```

---

## 🔌 Core Interfaces

### TextModel

The primary interface for all text generation:

```go
type TextModel interface {
    // Generate produces a stream of tokens from a prompt
    Generate(ctx context.Context, prompt string, opts ...GenerateOption) iter.Seq[Token]
    
    // Chat generates a response in a conversational context
    Chat(ctx context.Context, messages []Message, opts ...GenerateOption) iter.Seq[Token]
    
    // Close releases resources held by the model
    Close() error
    
    // Info returns model metadata
    Info() ModelInfo
}
```

**Key Methods:**
- `Generate()` — Streaming text generation from a prompt
- `Chat()` — Streaming conversational AI
- `Close()` — Resource cleanup
- `Info()` — Model metadata

### Backend

The backend factory interface:

```go
type Backend interface {
    Name() string                    // Backend identifier ("metal", "rocm", etc.)
    LoadModel(ctx context.Context, path string, opts LoadConfig) (TextModel, error)
    Supports(capability string) bool // Capability detection
    HealthCheck(ctx context.Context) error
}
```

**Backend Registration:**
```go
// In backend package init()
func init() {
    inference.Register("metal", &MetalBackend{})
}
```

### Token

Streaming token representation:

```go
type Token struct {
    Text string    // The generated text token
    Err  error     // Error if generation failed (checked after iterator ends)
}
```

**Error Pattern:** Follows `database/sql` pattern — errors retrieved after iterator completes.

### Message

Chat message structure:

```go
type Message struct {
    Role    string // "system", "user", "assistant"
    Content string // Message text content
    Name    string // Optional speaker name
}
```

---

## 📦 Configuration

### GenerateConfig

Text generation configuration:

```go
type GenerateConfig struct {
    MaxTokens     int       // Maximum tokens to generate
    Temperature   float64   // Sampling temperature (0-2)
    TopP         float64   // Nucleus sampling threshold (0-1)
    TopK         int       // Top-K sampling
    RepeatPenalty float64   // Repetition penalty
    StopSequences []string // Stop generation at these sequences
    Stream       bool      // Enable/disable streaming
}

// Functional options
func WithMaxTokens(n int) GenerateOption
func WithTemperature(t float64) GenerateOption
func WithTopP(p float64) GenerateOption
func WithTopK(k int) GenerateOption
func WithRepeatPenalty(p float64) GenerateOption
func WithStopSequences(seq ...string) GenerateOption
func WithStream(b bool) GenerateOption
```

### LoadConfig

Model loading configuration:

```go
type LoadConfig struct {
    Backend        string      // Explicit backend name
    Quantization   Quantization // Quantization level
    Device         string      // Device to use
    NumGPU         int         // Number of GPUs
    NumThread      int         // Number of CPU threads
}

// Functional options
func WithBackend(name string) LoadOption
func WithQuantization(q Quantization) LoadOption
func WithDevice(device string) LoadOption
func WithNumGPU(n int) LoadOption
func WithNumThread(n int) LoadOption
```

---

## 🚀 Usage Patterns

### Basic Generation

```go
import (
    "context"
    "fmt"
    "dappco.re/go/inference"
    _ "forge.lthn.ai/core/go-mlx" // Side-effect: registers Metal backend
)

func main() {
    ctx := context.Background()
    
    // Auto-select backend based on platform
    model, err := inference.LoadModel("/path/to/model/")
    if err != nil {
        panic(err)
    }
    defer model.Close()
    
    // Streaming generation
    for token := range model.Generate(ctx, "Explain AI alignment") {
        fmt.Print(token.Text)
    }
}
```

### Chat with Configuration

```go
messages := []inference.Message{
    {Role: "system", Content: "You are a helpful assistant."},
    {Role: "user", Content: "What is machine learning?"},
}

for token := range model.Chat(
    ctx, 
    messages,
    inference.WithMaxTokens(256),
    inference.WithTemperature(0.7),
) {
    fmt.Print(token.Text)
}
```

### Explicit Backend Selection

```go
// Force Metal on macOS
model, _ := inference.LoadModel("/path/to/model/", 
    inference.WithBackend("metal"))

// Force ROCm on Linux
model, _ := inference.LoadModel("/path/to/model/", 
    inference.WithBackend("rocm"))

// Force CPU
model, _ := inference.LoadModel("/path/to/model/", 
    inference.WithBackend("llama_cpp"))
```

### Model Discovery

```go
// Discover all available models
models, err := inference.Discover("/path/to/models/")
if err != nil {
    panic(err)
}

for _, info := range models {
    fmt.Printf("Model: %s\n", info.Name)
    fmt.Printf("  Backend: %s\n", info.Backend)
    fmt.Printf("  Size: %d bytes\n", info.Size)
    fmt.Printf("  Capabilities: %v\n", info.Capabilities)
}
```

---

## 🔥 Streaming with iter.Seq

Go 1.23+ introduced range-over-function iterators for efficient streaming:

```go
// Generate with streaming
for token := range model.Generate(ctx, "Hello", 
    inference.WithMaxTokens(100),
) {
    // Process each token as it's generated
    if err := token.Err; err != nil {
        log.Fatal(err)
    }
    fmt.Print(token.Text)
}

// Error handling: token.Err is checked after iterator completes
for token := range model.Generate(ctx, prompt) {
    // token.Text contains the generated text
}
// After loop, check for errors
// (The pattern follows database/sql.Row.Err())
```

**Iterator Lifecycle:**
1. Iterator yields tokens as they're generated
2. On error, iterator yields final token with `Err` set
3. Context cancellation terminates the iterator
4. Errors are retrieved via `token.Err` after the loop

---

## 🎯 Model Discovery

### Directory Scanning

```go
// Find all models in a directory tree
models, err := inference.Discover("/path/to/models/")

// With options
models, err := inference.Discover(
    "/path/to/models/",
    inference.WithRecursive(true),
    inference.WithPattern("*.gguf"),
)
```

**Model Directory Structure:**
```
my-model/
├── config.json          # Model metadata and configuration
├── model.safetensors    # Model weights
├── tokenizer.json       # Tokenizer configuration
└── tokenizer_config.json # Tokenizer details
```

### ModelInfo

```go
type ModelInfo struct {
    Name         string       // Model name
    Path         string       // Full path to model directory
    Backend      string       // Recommended backend
    Size         int64        // Total size in bytes
    Modified     time.Time    // Last modification time
    Capabilities []string     // Supported capabilities (training, attention, etc.)
    Config       map[string]any // Model configuration from config.json
}
```

---

## 🔍 Capability Detection

Optional interfaces are discovered at runtime via type assertion:

```go
// Check if model supports training
if trainable, ok := model.(inference.TrainableModel); ok {
    err := trainable.Train(ctx, config)
}

// Check if model supports attention inspection
if inspector, ok := model.(inference.AttentionInspector); ok {
    pattern, err := inspector.GetAttentionPattern(layer, head)
}

// Or use helper function
if inference.Supports(model, "training") {
    // Model can be fine-tuned
}
```

### Optional Interfaces

| Interface | Purpose | Key Methods |
|-----------|---------|-------------|
| `AttentionInspector` | Inspect attention patterns | `GetAttentionPattern(layer, head int) ([]float32, error)` |
| `TrainableModel` | Fine-tuning support | `Train(ctx, config) error`, `SaveCheckpoint(path) error` |
| `Adapter` | Task-specific adaptation | `Adapt(ctx, task, config) (TextModel, error)` |
| `StateTracker` | State management | `Track(state) error`, `GetState() State` |

---

## 📊 Backend Registry

### Registration Pattern

Backends register themselves via `init()` with appropriate build tags:

```go
// In go-mlx package
//go:build darwin && arm64
// +build darwin,arm64

package mlx

import "dappco.re/go/inference"

func init() {
    inference.Register("metal", &MetalBackend{})
}
```

### Registry Functions

```go
// Register a backend
inference.Register(name string, backend Backend)

// Get a specific backend
backend := inference.Get(name)

// List all registered backends
backends := inference.List()

// Get default backend for current platform
defaultBackend := inference.Default()

// Load model (auto-selects backend)
model, err := inference.LoadModel(path)

// Load model with explicit backend
model, err := inference.LoadModel(path, inference.WithBackend("metal"))
```

### Default Backend Priority

The default backend is automatically selected based on:

1. **Platform Detection**
   - macOS (darwin/arm64) → `metal` (MLX)
   - Linux (amd64) → `rocm` (AMD ROCm)
   - Any platform → `llama_cpp` (CPU)

2. **Availability Check**
   - Verifies backend is compiled in (build tags)
   - Checks for required hardware/software

3. **Fallback Chain**
   - `metal` → `rocm` → `llama_cpp` → first available

---

## 🧪 Testing

### Test Structure

Following the AX Standard with test triplets for all public symbols:

| Test Type | Suffix | Purpose |
|-----------|--------|---------|
| `Test*_Good*` | Happy path tests |
| `Test*_Bad*` | Expected error conditions |
| `Test*_Ugly*` | Panics and edge cases |

### Test Helpers

```go
// resetBackends(t) - clears global registry for isolated tests
func resetBackends(t *testing.T)

// stubBackend - minimal test backend implementation
var stubBackend = &StubBackend{...}

// stubTextModel - minimal test TextModel implementation
var stubTextModel = &StubTextModel{...}
```

### Running Tests

```bash
# All tests
go test ./...

# Single test
go test -run TestTextModel_Good_Generate

# With race detector
go test -race ./...

# Verbose output
go test -v ./...

# Benchmarks only
go test -bench=. -run=^$ ./...

# Specific package
go test ./go

# With coverage
go test -cover ./...
```

---

## 📈 Quality Metrics

| Metric | Status | Details |
|--------|--------|---------|
| **Zero Dependencies** | ✅ | Stdlib only, no external packages |
| **Cross-Platform** | ✅ | Compiles on all platforms |
| **Interface Stability** | ✅ | Strict stability rules |
| **Test Coverage** | ✅ | Good/Bad/Ugly pattern |
| **Streaming Support** | ✅ | Go 1.23+ iter.Seq |
| **Documentation** | ✅ | Complete README + INDEX |
| **Backend Support** | ✅ | Multiple implementations |
| **Auto-Selection** | ✅ | Smart backend priority |

### Stability Contract Rules

1. **Never** change existing method signatures on `TextModel` or `Backend`
2. **Only** add methods when two or more consumers need them
3. **Prefer** new interfaces that embed `TextModel` over extending it
4. **New fields** on config structs are safe (zero-value defaults)
5. **All new** interface methods require Virgil approval before merging

---

## 🔗 Related Packages

### Backend Implementations

| Package | Module | Platform | Status | Documentation |
|---------|--------|----------|--------|---------------|
| [go-mlx](../mlx/) | `dappco.re/go/mlx` | macOS | ✅ Production | [README](../mlx/README.md) |
| [go-rocm](../rocm/) | `dappco.re/go/rocm` | Linux | ✅ Production | [README](../rocm/README.md) |
| [go-ml](../ml/) | `dappco.re/go/ml` | Cross-platform | ✅ Production | [README](../ml/README.md) |
| [go-cuda](../cuda/) | `dappco.re/go/cuda` | NVIDIA | ⚠️ Scaffold | [RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/cuda/RFC.md) |
| [go-tpu](../tpu/) | `dappco.re/go/tpu` | Google Cloud | ⚠️ Scaffold | [RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/tpu/RFC.md) |

### Consumer Packages

| Package | Module | Purpose |
|---------|--------|---------|
| [go-ai](../ai/) | `dappco.re/go/ai` | AI integration layer, MCP tools |
| [go-rag](../rag/) | `dappco.re/go/rag` | RAG (Qdrant + Ollama embeddings) |
| [go-i18n](../i18n/) | `dappco.re/go/i18n` | Uses TextModel for classification |

### Core Framework

| Package | Relationship |
|---------|--------------|
| [CoreGO INDEX](../../INDEX.md) | Complete catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## 📝 Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-06-17 | Complete knowledge pack documentation | Purberus |
| 2026-05-XX | Go 1.23+ iter.Seq migration | Maintainer |
| 2026-04-XX | Backend registry refactor | Maintainer |
| 2026-03-XX | Optional interfaces pattern | Maintainer |
| 2026-02-XX | Initial contract definition | Maintainer |

---

## 🎯 Tags

```yaml
# Core Identity
- inference
- ml
- ai
- machine-learning
- llm
- text-generation
- backend-interface
- contract
- shared

# Interfaces
- textmodel
- backend
- token
- message
- streaming

# Capabilities
- generation
- chat
- classification
- batch-processing
- training
- attention-inspection
- adaptation

# Architecture
- interface
- registry
- backend
- streaming
- iter-seq
- type-assertion
- capability-detection

# Platforms
- cross-platform
- macos
- linux
- windows
- darwin
- arm64
- amd64

# Quality
- zero-deps
- stdlib-only
- production-ready
- well-tested
- stable-contract
- high-coverage
- documented

# Backends
- metal
- mlx
- rocm
- cuda
- tpu
- llama-cpp

# Ecosystem
- corego
- dappcore
- lethean
- forgejo

# Compliance
- ax-standard
- spor
- test-triplets
```

---

## 📚 References

1. **Repository** — [~/Code/core/go-inference/](file:///Users/snider/Code/core/go-inference/)
2. **CLAUDE.md** — [~/Code/core/go-inference/CLAUDE.md](file:///Users/snider/Code/core/go-inference/CLAUDE.md)
3. **AGENTS.md** — [~/Code/core/go-inference/AGENTS.md](file:///Users/snider/Code/core/go-inference/AGENTS.md)
4. **GOAL.md** — [~/Code/core/go-inference/GOAL.md](file:///Users/snider/Code/core/go-inference/GOAL.md)
5. **RFC** — [plans/code/core/go/inference/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/inference/RFC.md)
6. **Architecture Docs** — [~/Code/core/go-inference/docs/architecture.md](file:///Users/snider/Code/core/go-inference/docs/architecture.md)
7. **Development Guide** — [~/Code/core/go-inference/docs/development.md](file:///Users/snider/Code/core/go-inference/docs/development.md)
8. **History** — [~/Code/core/go-inference/docs/history.md](file:///Users/snider/Code/core/go-inference/docs/history.md)
9. **Index** — [~/Code/core/go-inference/docs/index.md](file:///Users/snider/Code/core/go-inference/docs/index.md)
10. **CoreGO INDEX** — [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md)

---

*Package index generated: 2026-06-17T23:45:00Z*
*Knowledge Pack: CoreGo v1.2.0*
*Module: dappco.re/go/inference*
*Maintainer: Purberus <purberus@lthn.ai>*
