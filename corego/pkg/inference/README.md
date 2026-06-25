---
type: Package Documentation
package: inference
module: dappco.re/go/inference
repo: core/go-inference
lang: go
tags:
  - ai
  - inference
  - machine-learning
  - text-generation
  - backend-interface
  - shared-contract
  - textmodel
  - llm
  - stdlib-only
  - zero-deps
---
# go-inference — Shared ML Inference Interfaces

**Module:** `dappco.re/go/inference`
**Module Path:** `dappco.re/go/inference`
**Repository:** [~/Code/core/go-inference/](file:///Users/snider/Code/core/go-inference/)
**Status:** Production
**License:** EUPL-1.2
**Language:** Go 1.25+
**Dependencies:** Zero external (stdlib only)
**RFC:** [plans/code/core/go/inference/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/inference/RFC.md)
**CLAUDE.md:** [~/Code/core/go-inference/CLAUDE.md](file:///Users/snider/Code/core/go-inference/CLAUDE.md)
**AGENTS.md:** [~/Code/core/go-inference/AGENTS.md](file:///Users/snider/Code/core/go-inference/AGENTS.md)
**GOAL.md:** [~/Code/core/go-inference/GOAL.md](file:///Users/snider/Code/core/go-inference/GOAL.md)
**Maintainer:** Purberus <purberus@lthn.ai>
**Last Updated:** 2026-06-17

---

## Overview

`go-inference` is the shared interface contract for text generation backends in the Core Go ecosystem. It defines the `TextModel`, `Backend`, `Token`, `Message`, and associated configuration types that GPU-specific backends implement and consumers depend on.

### Key Characteristics

- **Zero External Dependencies** — Stdlib only, compiles on all platforms regardless of GPU availability
- **Backend-Agnostic** — Pure interface package, no implementations
- **Platform-Aware Registry** — Automatic backend selection (Metal on macOS, ROCm on Linux) with explicit pinning support
- **Stability Contract** — Changes affect go-mlx, go-rocm, and go-ml simultaneously; strict stability rules
- **Streaming Support** — Uses Go 1.23+ `iter.Seq[Token]` for streaming token generation

### Backend Consumers

| Backend | Repository | Platform | Status |
|---------|------------|----------|--------|
| **go-mlx** | [core/go-mlx](file:///Users/snider/Code/core/go-mlx/) | macOS (darwin/arm64) | ✅ Production |
| **go-rocm** | [core/go-rocm](file:///Users/snider/Code/core/go-rocm/) | Linux (amd64) | ✅ Production |
| **go-ml** | [core/go-ml](file:///Users/snider/Code/core/go-ml/) | CPU + Wrapper | ✅ Production |
| **go-cuda** | [core/go-cuda](file:///Users/snider/Code/core/go-cuda/) | NVIDIA CUDA | ⚠️ Scaffold |
| **go-tpu** | [core/go-tpu](file:///Users/snider/Code/core/go-tpu/) | Google TPU | ⚠️ Scaffold |

---

## Architecture

### Component Layers

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    go-inference Contract Layer                                │
│  Zero dependencies, stdlib only, compiles everywhere                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Core Interfaces (go/)                                     │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐              │
│  │  inference.go    │ │   options.go     │ │   discover.go    │              │
│  │  TextModel       │ │   GenerateConfig │ │   Discover()     │              │
│  │  Backend         │ │   LoadConfig    │ │   Model Dir Scan │              │
│  │  Token          │ │   With* Options │ │                 │              │
│  │  Message        │ │   Apply* Helpers │ │                 │              │
│  │  ClassifyResult│ │                 │ │                 │              │
│  │  BatchResult   │ │                 │ │                 │              │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Registry System                                          │
│  Backend Registry: Register() | Get() | List() | Default()                │
│  LoadModel(): Routes to backend via WithBackend() or Default()              │
│  Priority: metal > rocm > llama_cpp > any available                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Optional Interfaces                                       │
│  AttentionInspector | TrainableModel | Adapter | StateTracker               │
│  Discovered via type assertion, not via extending core interfaces           │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Backend Implementations (Separate Repos)                 │
│  go-mlx (Metal) | go-rocm (AMD) | go-ml (CPU/Wrapper) | go-cuda (NVIDIA)     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Core Files

| File | Purpose | Key Types/Functions |
|------|---------|---------------------|
| `inference.go` | **Core interfaces** | `TextModel`, `Backend`, `Token`, `Message`, `ClassifyResult`, `BatchResult`, `Register`, `Get`, `List`, `Default`, `LoadModel` |
| `options.go` | **Configuration** | `GenerateConfig`, `LoadConfig`, `With*()` functional options, `Apply*Opts` helpers |
| `discover.go` | **Model discovery** | `Discover()` scans directories for model dirs (config.json + *.safetensors) |
| `training.go` | **Training support** | `TrainableModel` interface, `Adapter` interface, `LoadTrainable()` |
| `capability.go` | **Capability detection** | `Supports*()` helpers for runtime capability checking |
| `decode/` | **Model decoding** | `Decoder`, `NewGGUFDecoder`, `NewSafetensorsDecoder` |
| `parser/` | **Model parsing** | `Parser`, `NewGGUFParser` |
| `model/` | **Model utilities** | Model file handling, pack operations |
| `state/` | **State management** | Session state, conversation history |
| `quant/` | **Quantization** | Quantization level detection and handling |
| `scheduler/` | **Inference scheduling** | Request queueing, prioritization |
| `service.go` | **Service layer** | Service registration, lifecycle management |
| `probe.go` | **Health probing** | Backend health checks, availability |
| `split.go` | **Token splitting** | Text splitting utilities |
| `eval/` | **Evaluation** | Model evaluation, benchmarking |
| `bench/` | **Benchmarking** | Performance testing tools |

### Backend Registry Pattern

```go
// Backends register via init() with build tags
//go:build darwin && arm64
// +build darwin,arm64

package mlx

import "dappco.re/go/inference"

func init() {
    inference.Register("metal", &MetalBackend{})
}
```

**Default Backend Selection Priority:**
1. `metal` (Apple Silicon via MLX)
2. `rocm` (AMD GPU via ROCm)
3. `llama_cpp` (CPU via llama.cpp)
4. Any other available backend

**Explicit Backend Pinning:**
```go
model, err := inference.LoadModel(path, inference.WithBackend("metal"))
```

---

## Core interfaces

### TextModel Interface

The primary interface for text generation:

```go
type TextModel interface {
    // Generate produces a stream of tokens from a prompt
    Generate(ctx context.Context, prompt string, opts ...inference.GenerateOption) iter.Seq[inference.Token]
    
    // Chat generates a response in a conversational context
    Chat(ctx context.Context, messages []inference.Message, opts ...inference.GenerateOption) iter.Seq[inference.Token]
    
    // Close releases resources held by the model
    Close() error
    
    // Info returns model metadata
    Info() inference.ModelInfo
}
```

### Backend Interface

The backend factory interface:

```go
type Backend interface {
    // Name returns the backend identifier (e.g., "metal", "rocm", "llama_cpp")
    Name() string
    
    // LoadModel loads a model from the given path
    LoadModel(ctx context.Context, path string, opts inference.LoadConfig) (TextModel, error)
    
    // Supports returns true if this backend supports the given capability
    Supports(capability string) bool
    
    // HealthCheck verifies the backend is operational
    HealthCheck(ctx context.Context) error
}
```

### Optional Interfaces

Additional capabilities discovered via type assertion:

```go
// AttentionInspector - inspect attention patterns
type AttentionInspector interface {
    TextModel
    GetAttentionPattern(layer, head int) ([]float32, error)
}

// TrainableModel - supports fine-tuning
type TrainableModel interface {
    TextModel
    Train(ctx context.Context, config inference.TrainConfig) error
    SaveCheckpoint(path string) error
}

// Adapter - model adapter for specific tasks
type Adapter interface {
    Adapt(ctx context.Context, task string, config map[string]any) (TextModel, error)
}
```

---

## Configuration types

### GenerateConfig

```go
type GenerateConfig struct {
    MaxTokens     int
    Temperature   float64
    TopP         float64
    TopK         int
    RepeatPenalty float64
    StopSequences []string
    Stream       bool
    // ... additional fields
}

// Functional options
func WithMaxTokens(n int) inference.GenerateOption
func WithTemperature(t float64) inference.GenerateOption
func WithStopSequences(seq ...string) inference.GenerateOption
// ... additional With* functions
```

### LoadConfig

```go
type LoadConfig struct {
    Backend    string
    Quantization inference.Quantization
    Device     string
    NumGPU     int
    NumThread  int
    // ... additional fields
}

func WithBackend(name string) inference.LoadOption
func WithQuantization(q inference.Quantization) inference.LoadOption
func WithDevice(device string) inference.LoadOption
```

---

## Streaming with iter.Seq

Go 1.23+ range-over-function iterators:

```go
ctx := context.Background()
model, _ := inference.LoadModel("/path/to/model/")
defer model.Close()

// Streaming generation
for token := range model.Generate(ctx, "Hello, world!", 
    inference.WithMaxTokens(256),
    inference.WithTemperature(0.7),
) {
    if token.Err != nil {
        log.Fatal(token.Err)
    }
    fmt.Print(token.Text)
}

// Chat streaming
messages := []inference.Message{
    {Role: "user", Content: "Hello!"},
}
for token := range model.Chat(ctx, messages) {
    fmt.Print(token.Text)
}
```

**Error Handling Pattern:**
- Errors are retrieved via `token.Err()` after iterator finishes
- Follows `database/sql` `Row.Err()` pattern
- Iterator continues until context cancellation or error

---

## Model discovery

### Discover Function

```go
// Scan directories for model dirs
models, err := inference.Discover("/path/to/models/")
for _, modelInfo := range models {
    fmt.Printf("Model: %s (Backend: %s)\n", 
        modelInfo.Name, modelInfo.Backend)
}
```

**Model Directory Structure:**
```
model-name/
├── config.json          # Model configuration
├── model.safetensors    # Model weights
├── tokenizer.json       # Tokenizer
└── ...
```

### ModelInfo

```go
type ModelInfo struct {
    Name        string
    Path        string
    Backend     string
    Size        int64
    Modified    time.Time
    Capabilities []string
    Config      map[string]any
}
```

---

## Capability detection

```go
// Check if backend supports specific capability
if inference.Supports(model, "training") {
    // Model can be fine-tuned
    trainable := model.(inference.TrainableModel)
    trainable.Train(ctx, config)
}

// Check for attention inspection
if attention, ok := model.(inference.AttentionInspector); ok {
    pattern := attention.GetAttentionPattern(0, 0)
}
```

---

## Backend implementations

### go-mlx (Apple Metal)

- **Repository:** [core/go-mlx](file:///Users/snider/Code/core/go-mlx/)
- **Module:** `dappco.re/go/mlx`
- **Platform:** macOS (darwin/arm64)
- **Backend Name:** `"metal"`
- **Features:** Full GPU acceleration, optimized for Apple Silicon

### go-rocm (AMD GPU)

- **Repository:** [core/go-rocm](file:///Users/snider/Code/core/go-rocm/)
- **Module:** `dappco.re/go/rocm`
- **Platform:** Linux (amd64)
- **Backend Name:** `"rocm"`
- **Features:** AMD ROCm/HIP support

### go-ml (CPU + Wrapper)

- **Repository:** [core/go-ml](file:///Users/snider/Code/core/go-ml/)
- **Module:** `dappco.re/go/ml`
- **Platform:** Cross-platform
- **Backend Name:** `"llama_cpp"` (HTTP backend)
- **Features:** CPU inference, scoring engine, llama.cpp integration

### go-cuda (NVIDIA CUDA) — Scaffold

- **Repository:** [core/go-cuda](file:///Users/snider/Code/core/go-cuda/)
- **Module:** `dappco.re/go/cuda`
- **Platform:** Linux/Windows (NVIDIA GPU)
- **Backend Name:** `"cuda"`
- **Status:** ⚠️ Scaffold (hardware-light, gap-filled)

### go-tpu (Google Cloud TPU) — Scaffold

- **Repository:** [core/go-tpu](file:///Users/snider/Code/core/go-tpu/)
- **Module:** `dappco.re/go/tpu`
- **Platform:** Google Cloud
- **Backend Name:** `"tpu"`
- **Status:** ⚠️ Scaffold (narrow surface, gap-filled)

---

## Usage examples

### Basic Generation

```go
package main

import (
    "context"
    "fmt"
    "dappco.re/go/inference"
    _ "forge.lthn.ai/core/go-mlx" // Register Metal backend
)

func main() {
    ctx := context.Background()
    
    // Load model with automatic backend selection
    model, err := inference.LoadModel("/path/to/model/")
    if err != nil {
        panic(err)
    }
    defer model.Close()
    
    // Generate text
    for token := range model.Generate(ctx, "Explain AI alignment", 
        inference.WithMaxTokens(512),
        inference.WithTemperature(0.9),
    ) {
        fmt.Print(token.Text)
    }
}
```

### Chat Conversation

```go
package main

import (
    "context"
    "fmt"
    "dappco.re/go/inference"
)

func main() {
    ctx := context.Background()
    model, _ := inference.LoadModel("/path/to/model/")
    defer model.Close()
    
    messages := []inference.Message{
        {Role: "system", Content: "You are a helpful assistant."},
        {Role: "user", Content: "What is machine learning?"},
    }
    
    for token := range model.Chat(ctx, messages, 
        inference.WithMaxTokens(256),
    ) {
        fmt.Print(token.Text)
    }
}
```

### Explicit Backend Selection

```go
// Force Metal backend on macOS
model, err := inference.LoadModel("/path/to/model/",
    inference.WithBackend("metal"))

// Force ROCm backend on Linux
model, err := inference.LoadModel("/path/to/model/",
    inference.WithBackend("rocm"))
```

### Model Discovery

```go
// Discover all models in a directory
models, err := inference.Discover("/path/to/models/")
if err != nil {
    panic(err)
}

for _, info := range models {
    fmt.Printf("Found: %s (Type: %s, Backend: %s)\n",
        info.Name, info.Config["type"], info.Backend)
}
```

---

## Testing

### Test Structure

Following the AX Standard with test triplets:
- `*_test.go` — Unit and integration tests
- `*_example_test.go` — Usage examples

**Naming Convention:**
- `Test<Symbol>_Good*` — Happy path tests
- `Test<Symbol>_Bad*` — Expected error conditions
- `Test<Symbol>_Ugly*` — Panics and edge cases

### Test Helpers

```go
// resetBackends(t) - clears global registry for isolated tests
// stubBackend - test backend implementation
// stubTextModel - test TextModel implementation
```

### Running Tests

```bash
# All tests
go test ./...

# Single test
go test -run TestTextModel_Good ./inference

# With race detector
go test -race ./...

# Verbose
go test -v ./...

# Benchmarks
go test -bench=. ./...
```

---

## Quality metrics

- Zero dependencies — Stdlib only, no external packages
- Cross-platform — Compiles on all platforms
- Stable contract — Strict stability rules for interface changes
- Well-tested — Good/Bad/Ugly test pattern
- Streaming support — Go 1.23+ iter.Seq
- Backend-agnostic — Multiple backend implementations
- Auto-selection — Smart backend priority
- Type-safe — Strong Go typing throughout

---

## Stability rules

This package is the shared contract. Changes affect multiple backends simultaneously:

1. **Never** change existing method signatures on `TextModel` or `Backend`
2. **Only** add methods when two or more consumers need them
3. **Prefer** new interfaces that embed `TextModel` over extending `TextModel`
4. **New fields** on `GenerateConfig` or `LoadConfig` are safe (zero-value defaults)
5. **All new** interface methods require Virgil approval before merging

---

## Related packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-mlx](../mlx/) | Apple Metal backend implementation | [../mlx/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/mlx/) |
| [go-rocm](../rocm/) | AMD ROCm backend implementation | [../rocm/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/rocm/) |
| [go-ml](../ml/) | CPU backend + scoring engine | [../ml/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/ml/) |
| [go-ai](../ai/) | AI integration layer | [../ai/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/ai/) |
| [go-rag](../rag/) | RAG (Qdrant + Ollama) | [../rag/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/rag/) |
| [CoreGO INDEX](../../INDEX.md) | Complete catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## References

1. **Repository** — [~/Code/core/go-inference/](file:///Users/snider/Code/core/go-inference/)
2. **CLAUDE.md** — [~/Code/core/go-inference/CLAUDE.md](file:///Users/snider/Code/core/go-inference/CLAUDE.md)
3. **AGENTS.md** — [~/Code/core/go-inference/AGENTS.md](file:///Users/snider/Code/core/go-inference/AGENTS.md)
4. **GOAL.md** — [~/Code/core/go-inference/GOAL.md](file:///Users/snider/Code/core/go-inference/GOAL.md)
5. **RFC** — [plans/code/core/go/inference/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/inference/RFC.md)
6. **Architecture Docs** — [~/Code/core/go-inference/docs/architecture.md](file:///Users/snider/Code/core/go-inference/docs/architecture.md)
7. **Development Guide** — [~/Code/core/go-inference/docs/development.md](file:///Users/snider/Code/core/go-inference/docs/development.md)
8. **History** — [~/Code/core/go-inference/docs/history.md](file:///Users/snider/Code/core/go-inference/docs/history.md)

---

*Package documentation: 2026-06-17T23:30:00Z*
*Knowledge pack: CoreGo v1.2.0*
*Module: dappco.re/go/inference*
