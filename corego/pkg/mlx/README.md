---
type: Package Deep Dive
title: go-mlx — Native Apple Metal GPU Inference & Training Engine
description: Complete documentation for go-mlx — Native Apple Silicon (M1-M4) Metal GPU inference and training engine with mlx-c CGO bindings
description: Native Apple Metal inference engine implementing inference.Backend and inference.TextModel contracts from go-inference, supporting Gemma 3/4, Qwen 2/3, Llama 3, MoE models, with session/state persistence, speculative decoding, split inference, SFT/LoRA/SSD training, LEK content scoring, AutoRound quantisation, hierarchical FFN-memory pretraining
module: dappco.re/go/mlx
repo: core/go-mlx
tags: [ml, machine-learning, inference, training, metal, gpu, apple-silicon, mlx, mlx-c, cgo, neural-networks, transformers, gemma, qwen, llama, moe, lora, sft, quantization, serving, speculative-decoding, kv-cache, session-state, agent-memory]
lang: go
author: Mistral Vibe
version: 1.0.0
created: 2026-06-18T07:00:00Z
---

# go-mlx — Native Apple Metal GPU Inference & Training Engine

> **"An agent should be able to load models, run stateful sessions, and train adapters from this document alone."**

`dappco.re/go/mlx` is the **native Apple Metal GPU inference and training engine** for Apple Silicon (M1-M4) macOS systems. Built on **mlx-c CGO bindings**, it implements the `inference.Backend` and `inference.TextModel` contracts from `dappco.re/go/inference` with full Metal compute shader acceleration.

This is the **production-grade Metal backend** for the Lethean AI platform, providing:
- **Inference:** Buffered/streaming generation, chat, embeddings with full sampling control
- **State Engine:** Persistent sessions with Wake/Sleep KV state capture and restore
- **Training:** Native SFT, LoRA, and SSD (self-distillation) on Metal
- **Advanced Features:** Speculative decoding, split inference (CPU/GPU/remote FFN placement)
- **Model Support:** ~19 registered architectures including Gemma 3/4, Qwen 2/3, Llama 3, MoE models
- **Scoring:** LEK content scorer (tier-1 non-LLM semantic analysis)
- **Quantisation:** AutoRound weight-only quantisation with per-component granularity

**Platform:** `darwin/arm64` only. On non-Mac or non-arm64 platforms, the package compiles without CGO and `Available()` returns `false`.

---

## 🎯 Overview

### What it is

- **Native Metal Engine** — Direct Metal compute shader execution via mlx-c CGO bindings
- **Inference Backend** — Full `inference.Backend` contract implementation
- **Text Model** — Full `inference.TextModel` contract with generation, chat, embeddings
- **Session/State Engine** — Persistent stateful sessions with Wake/Sleep KV capture
- **Speculative Decoding** — Target/draft accept-reject with native drafter detection
- **Split Inference** — Cross-runtime placement: Native Metal, CPU-side FFN, Remote FFN
- **Training Engine** — Native SFT, LoRA, SSD training on Metal GPU
- **LEK Scorer** — Tier-1 content scoring across 7 axes (compliance, sycophancy, hostility, etc.)
- **Quantisation** — AutoRound weight-only quantisation with model-specific overrides
- **FFN-Memory Pretraining** — Hierarchical memory pretraining for feed-forward layers
- **MoE Support** — Mixture-of-Experts with sparse expert routing
- **Multimodal** — Vision encoder support for Gemma 4 unified models

### Supported Models & Architectures

#### Model Families (19+ registered)

| Family | Types | Notes | Status |
|--------|-------|-------|--------|
| **Gemma** | `gemma3`, `gemma4`, `gemma4_text`, `gemma4_unified`, `diffusion_gemma` | Dense + MoE + per-layer-input gating + vision | ✅ Production |
| **Qwen** | `qwen3_6`, `qwen3_moe`, `qwen3_6_moe` | Dense + MoE variants | ✅ Production |
| **Mixture of Experts** | `mixtral`, `gpt_oss`, `deepseek`, `kimi`, `minimax_m2` | Sparse expert models | ✅ Production |
| **Linear/Recurrent** | `mamba2`, `rwkv7`, `retnet`, `gla`, `deltanet` | Alternative attention mechanisms | ✅ Production |
| **Sparse/Alternative Attention** | `nsa`, `moba`, `gsa`, `mla` | Native-sparse, mixture-of-block, gated-slot, multi-head-latent | ✅ Production |
| **Embedding/Rerank** | `bert`, `bert_rerank` | Encoders for RAG/scoring | ✅ Production |

#### Gemma 4 Variants (Lemma Family)

| Variant | Params | Architecture | Context | Status |
|---------|--------|-------------|---------|--------|
| **Lemer (E2B)** | 2.3B effective | Efficient dense + per-layer input | 128K | ✅ Production |
| **Lemma (E4B)** | 4.5B effective | Efficient dense + per-layer input | 128K | ✅ Production |
| **Lemmy (26B A4B)** | 26B total / 3.8B active | MoE (128 experts, 8+1 active) | 256K | ✅ Production |
| **Lemrd (31B)** | 30.7B | Dense | 256K | ✅ Production |

### Architecture Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                      go-mlx LAYERS                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           LAYER 4: FACADE (Public API)                      │   │
│  │  mlx.go, generate.go, native_model.go, primitives.go       │   │
│  │  Public Model API, generation, inference contracts        │   │
│  │  Tensor/optimiser re-exports, training entry points         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           LAYER 3: ENGINE (Metal Backend)                   │   │
│  │  pkg/metal/ (~100 files)                                    │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │   │
│  │  │ Tensors       │  │ Autodiff     │  │ Optimisers   │   │   │
│  │  │ (Array)       │  │ (VJP)        │  │ (AdamW)      │   │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │   │
│  │  │ KV Caches    │  │ Chat         │  │ Model Loaders│   │   │
│  │  │ (Standard,   │  │ Formatters   │  │ (per-arch)   │   │   │
│  │  │  Rotating,   │  │              │  │              │   │   │
│  │  │  Paged)      │  │              │  │              │   │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │   │
│  │  ┌──────────────┐  ┌──────────────┐                       │   │
│  │  │ Losses       │  │ Mixers       │                       │   │
│  │  │              │  │ (Attention    │                       │   │
│  │  └──────────────┘  │  variants)    │                       │   │
│  │                     └──────────────┘                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           LAYER 2: MODEL ARCHITECTURES                      │   │
│  │  pkg/metal/model/<arch>/                                  │   │
│  │  gemma3/, gemma4/, qwen3/, mixtral/, mamba2/, etc.         │   │
│  │  Per-architecture forward passes implementing             │   │
│  │  InternalModel interface                                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           LAYER 1: STATE & TRAINING                        │   │
│  │  session/, kv/, spine/, train/, memorypretrain/          │   │
│  │  Session/state engine, KV snapshots, training loops        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           LAYER 0: CGO & EXTERNAL                           │   │
│  │  mlx-c (C bindings) → MLX Framework → Metal GPU            │   │
│  │  lib/mlx/ (submodule v0.31.1)                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────┘
```

### Build Architecture

```
mlx (facade) → pkg/metal (engine) → mlx-c (CGO) → MLX Framework → Metal GPU
```

---

## 📦 Quick Start

### Basic Model Loading & Generation

```go
import (
    "context"
    "log"
    
    "dappco.re/go/mlx"
)

func main() {
    // Check if Metal is available
    if !mlx.Available() {
        log.Fatal("Metal not available on this platform")
    }
    
    // Load a model from safetensors or GGUF
    model, err := mlx.LoadModel("path/to/gemma-3-4b",
        mlx.WithContextLength(8192),
        mlx.WithQuantization(4), // 4-bit quantization
    )
    if err != nil {
        log.Fatal(err)
    }
    defer model.Close() // CRITICAL: Release GPU memory
    
    // Get model info
    info := model.Info()
    log.Printf("Architecture: %s", info.Architecture)
    log.Printf("Layers: %d", info.NumLayers)
    log.Printf("Context Length: %d", info.ContextLength)
    log.Printf("Quantization: %d-bit", info.Quantization)
    
    // Buffered generation
    prompt := "Explain the Lethean project in 3 sentences."
    response, err := model.Generate(prompt,
        mlx.WithTemperature(0.8),
        mlx.WithMaxTokens(1024),
    )
    if err != nil {
        log.Fatal(err)
    }
    
    log.Printf("Response: %s", response)
}
```

### Streaming Generation

```go
// Streaming tokens
for token := range model.GenerateStream(prompt) {
    // token.ID - token integer ID
    // token.Value - decoded string value
    log.Printf("Token: %s", token.Value)
}
```

### Chat API

```go
// Create a chat session
session := model.NewChatSession()

// Chat with history
messages := []string{
    "You are a helpful assistant.",
    "What is Lethean?",
}

response, err := session.Chat(messages,
    mlx.WithTemperature(0.7),
    mlx.WithMaxTokens(512),
)
if err != nil {
    log.Fatal(err)
}

log.Printf("Assistant: %s", response)
```

### With io.Medium Model Loading

```go
import (
    "dappco.re/go/mlx"
    "dappco.re/go/core/io"
)

// Load from S3
s3Medium := io.S3("garage.lthn.io/models/")
model, err := mlx.LoadModelFromMedium(s3Medium, "gemma-3-4b",
    mlx.WithContextLength(4096),
)
if err != nil {
    log.Fatal(err)
}
defer model.Close()

// Load from HuggingFace Hub
hfMedium := io.HuggingFace("google/gemma-3-4b")
model, err = mlx.LoadModelFromMedium(hfMedium, "",
    mlx.WithQuantization(4),
)
```

---

## 🏗️ Architecture Deep Dive

### Facade Layer (Root Package)

The public API surface exposing the Metal backend:

**Key Files:**
- `mlx.go` — Main package exports, Available(), Backend()
- `generate.go` — Generate, GenerateStream, Chat, GenerateChunks
- `native_model.go` — NativeModel implementation
- `primitives.go` — Tensor/optimiser re-exports
- `register_metal.go` — Auto-registration with go-inference

**`init()` Auto-Registration:**
```go
//go:build darwin && arm64 && !nomlx

func init() {
    inference.Register("metal", &MetalBackend{})
}
```

On macOS arm64, the Metal backend auto-registers itself with `go-inference`. On other platforms or with `-tags nomlx`, it compiles without CGO and `Available()` returns `false`.

### Engine Layer (`pkg/metal/`)

The core Metal engine with ~100 files:

#### Core Types

```go
// Array wraps mlx_array C handles with finalizer
type Array struct {
    handle *C.mlx_array
    shape  []int
    dtype  DType
}

// Model is the loaded model with inference capabilities
type Model struct {
    metalModel *metal.Model
    config     *ModelConfig
    info       *ModelInfo
    // ...
}

// ModelInfo contains metadata about a loaded model
type ModelInfo struct {
    Architecture    string
    NumLayers      int
    HiddenSize     int
    NumAttentionHeads int
    ContextLength  int
    Quantization   int
    // ...
}
```

#### Key Components

| Component | File | Purpose |
|-----------|------|---------|
| **Tensors** | `tensor.go`, `array.go` | Array creation, manipulation, operations |
| **Autodiff** | `grad.go`, `vjp.go` | Automatic differentiation, VJP computation |
| **Optimisers** | `optim.go`, `adamw.go` | Optimisation algorithms (AdamW, etc.) |
| **Losses** | `loss.go` | Loss functions for training |
| **KV Caches** | `cache.go`, `rotating_cache.go`, `paged_cache.go` | Standard, rotating, paged KV caches |
| **Attention** | `attention.go`, `rope.go`, `sdpa.go` | Attention mechanisms, RoPE, SDPA |
| **Model Loaders** | `model/*.go` | Per-architecture forward passes |
| **Chat Formatters** | `chat_format.go` | Chat template processing |
| **Tokenizers** | `tokenizer.go` | BPE tokenisation (SentencePiece + GPT-2) |

### Model Architecture Layer (`pkg/metal/model/<arch>/`)

Per-architecture implementations:

| Architecture | Directory | Features |
|--------------|-----------|----------|
| **Gemma 3** | `gemma3/` | Standard transformer |
| **Gemma 4** | `gemma4/` | Dual attention, MoE, per-layer input gating |
| **Qwen 3** | `qwen3/` | Dense + MoE variants |
| **Mixtral** | `mixtral/` | Sparse MoE |
| **Mamba2** | `mamba2/` | Mamba/State-Space Model |
| **RWKV7** | `rwkv7/` | RWKV v7 |
| **RetNet** | `retnet/` | Retentive Network |
| **GLA** | `gla/` | Gated Linear Attention |
| **DeltaNet** | `deltanet/` | Delta Network |

Each architecture implements the `InternalModel` interface with its own forward pass.

### State & Training Layer

#### Session/State Engine

**Files:** `session.go`, `session_agent.go`, `session_defaults.go`, `conversation_continuity.go`, `session/`

Persists stateful sessions with KV cache capture:

```go
// Session wraps a model with persistent state
session := model.NewSession()

// Wake from previous state (restore KV cache)
err := session.WakeFromBundle(bundlePath)

// Generate with state
response, err := session.Generate(prompt)

// Sleep to durable storage (capture KV cache)
bundle, err := session.Sleep()

// Save bundle for later
err = bundle.SaveToFile("session.mlxbundle")
```

**Key Features:**
- **KV State Store (`kv/`)** — Durable KV state blocks with snapshots, labels, and analysis
- **Agent-Memory Fold (`session_agent.go`)** — Checkpoint and fold live context into fresh summary+tail state
- **Conversation Continuity (`conversation_continuity.go`)** — No-prompt-replay chat: stateless chat requests matched to conversation with retained state (0% replay)
- **Prompt Cache (`prompt_cache.go`)** — Warms token-prefix KV cache from prompt, chunks, snapshot, or state

#### KV State Store (`kv/`)

```go
// Snapshot carries layer-by-layer KV state
type Snapshot struct {
    HeadData  map[string]*KVState  // Per-layer key/value heads
    Logits    []float32
    Tokens    []int
    // ...
}

// Analysis computes quality metrics
type Analysis struct {
    KVCoherence    float64
    CrossAlignment float64
    PhaseLock      float64
    KVCoupling     float64
    JointCollapse  int
    Composite      float64  // 0-10000 score
}

// Analyze snapshot without forward pass
analysis := kv.Analyze(snapshot)
```

#### Training (`train/`, `sft.go`, `ssd.go`, `model_lora.go`)

Native Metal training:

```go
// Supervised Fine-Tuning (SFT)
trainResult, err := model.TrainSFT(
    dataset,  // Training dataset
    &mlx.SFTConfig{
        LoRAConfig: &mlx.LoRAConfig{
            Rank: 8,
            Alpha: 16,
            TargetModules: []string{"q_proj", "k_proj", "v_proj", "o_proj"},
        },
        Epochs: 3,
        BatchSize: 4,
        LearningRate: 1e-5,
    },
)

// Self-Distillation (SSD)
ssdResult, err := model.TrainSSD(
    samples,  // Sampled outputs from frozen model
    &mlx.SSDConfig{
        LoRAConfig: &mlx.LoRAConfig{...},
        Steps: 1000,
    },
)

// LoRA Management
// Apply LoRA adapter
model.ApplyLoRA(adapterPath)

// Load LoRA adapter
model.LoadLoRA(adapterPath, "lora-name")

// Unload LoRA
model.UnloadLoRA("lora-name")

// Merge LoRA into base model
model.MergeLoRA("lora-name")

// Swap between multiple LoRAs
model.SwapLoRA("current-lora", "new-lora")
```

### Advanced Features

#### Speculative Decoding (`speculative.go`)

Target/draft accept-reject decoding:

```go
// Enable speculative decoding
generateOptions := []mlx.GenerateOption{
    mlx.WithSpeculativeDecoding(true),
    mlx.WithDraftModel(draftModel),  // Separate smaller model for drafting
}

response, err := model.Generate(prompt, generateOptions...)
```

**Gemma 4 Native Drafter:**
- Gemma 4 ships with a native **MTP assistant drafter** (block-draft + batched-verify)
- Drafter detection is reactive: model declares its drafter by files beside it
- Engine reacts automatically via `draft_detect.go`
- Drafters are the `-it-assistant` model variants

#### Split Inference (`split_executor.go`, `split_*` files)

Cross-runtime placement for models that don't fit in GPU memory:

```go
// Native Metal runtime (GPU-resident)
// CPU-side FFN (offload feed-forward layers)
// Remote FFN (HTTP-backed FFN placement)

// Configure split placement
placement := &mlx.SplitPlacement{
    NativeRuntime: &mlx.NativeRuntimeConfig{
        // GPU-resident slice
    },
    CPUFFN: &mlx.CPUFFNConfig{
        // Offload FFN to CPU
        Enabled: true,
    },
    RemoteFFN: &mlx.RemoteFFNConfig{
        // HTTP-backed FFN
        Endpoint: "http://remote-ffn-server:8080",
    },
}

model, err := mlx.LoadModel(path, mlx.WithSplitPlacement(placement))
```

**Residency Reporting:**
- `memory_plan.go` — Turns `DeviceInfo` + `ModelInfo` into residency plan
- `metal_capabilities.go` — Reports backend capabilities to inference layer
- Automatic layer placement based on memory constraints

### Serving & Throughput Engine

High-throughput serving with:
- **Shared-prefix KV (RadixAttention)** — Efficient KV sharing for shared prefixes
- **Continuous batching + chunked prefill** — Optimal batch processing
- **Hierarchical/quantised KV cache** — Memory-efficient caching
- **Quantised-weight loading** — Including w4a16-ct, AWQ, GPTQ
- **Speculative verification** — Faster decoding with verification
- **Jump-forward constrained decoding** — Optimized decoding paths
- **Multi-LoRA apply** — Apply multiple LoRAs simultaneously
- **Online weight swap** — Hot-swap model weights

Full detail: **[RFC.serving.md](file:///Users/snider/Code/meowmix/plans/code/core/go/mlx/RFC.serving.md)**

---

## 🎯 LEK Content Scorer

The tier-1 non-LLM content scorer (`pkg/score/`):

```go
import "dappco.re/go/mlx/pkg/score"

// Score text across 7 axes
result := score.Score(text)

// ScoreResult contains:
// - LEK heuristic (tier-1 axis-set)
// - Sycophancy (4-tier pattern matching)
// - Hostility (directed-anger / AngerScore)
// - Authority (authority-figure noun distribution)
// - Dialect (contraction / colloquial-form allowlist)
// - Differential (reversal-tokeniser signal)
// - Phonetic (CMUdict / metaphone phonetic dimensions)

// Composite score (0-10000)
composite := result.Composite

// Individual axis scores
lekScore := result.LEKScore
sycophancyScore := result.SycophancyScore
hostilityScore := result.HostilityScore
```

**Use Cases:**
- Training score cascade (§10) — LEK scorer rides the SFT eval loop
- Serving content filtering
- Quality probes and eval reports
- Checkpoint selection driven by semantic analysis

---

## 📊 Quantisation

### AutoRound Quantisation (`quant/autoround/`)

Weight-only rounding primitive with per-component granularity:

```go
// Load with quantization
model, err := mlx.LoadModel(path,
    mlx.WithQuantization(4),  // 4-bit
    mlx.WithQuantizationGroupSize(64),
)

// Per-component quantization (Gemma 4 always quantizes router at 8-bit)
model, err = mlx.LoadModel(path,
    mlx.WithQuantization(4),
    mlx.WithPerComponentQuantization(map[string]int{
        "router_proj": 8,  // Router always 8-bit
    }),
)
```

**Supported Quantisations:**
- `affine` — Standard affine quantisation
- `q4_0`, `q4_1`, `q5_0`, `q5_1`, `q8_0` — Various bits
- `mxfp4` — Mixed floating-point formats
- AutoRound — Automatic rounding optimization

### Model-Specific Quantisation Rules

**Gemma 4:** Router projection always quantised 8-bit, group-size 64, regardless of model's overall level (per-path predicate applied during load).

---

## 🔄 FFN-Memory Pretraining

Hierarchical memory pretraining for feed-forward layers:

```go
// Configure FFN memory
ffnMemoryConfig := &mlx.FFNMemoryConfig{
    MemorySize: 1024,
    NumHeads:   16,
    // ...
}

// Create memory pretraining task
pretrainConfig := &mlx.MemoryPretrainConfig{
    Model:           model,
    FFNMemoryConfig: ffnMemoryConfig,
    Dataset:        dataset,
    Steps:          10000,
    // ...
}

// Run pretraining
result, err := mlx.MemoryPretrain(pretrainConfig)
```

**Features:**
- Small local models retrieve context-dependent memory
- Extra parameters attached to each feed-forward layer
- Backed by Metal kernels (`ffn_memory_metal.go`)
- Durable bank files for persistence

---

## 🎯 Tokenisation

Pure Go SentencePiece tokeniser (no external dependencies):

```go
import "dappco.re/go/mlx"

// Tokenizer from model
tokenizer := model.Tokenizer()

// Encode text to tokens
tokens, err := tokenizer.Encode(text)

// Decode tokens to text
text, err := tokenizer.Decode(tokens)

// Token ID lookup
tokenID := tokenizer.GetTokenID("hello")
tokenStr := tokenizer.GetTokenString(12345)

// Special tokens
bosToken := tokenizer.BOSToken()
eosToken := tokenizer.EOSToken()
padToken := tokenizer.PadToken()

// Chat templates
// Gemma and Qwen chat templates applied by registered formatters
```

---

## 🔧 Memory Management

**CRITICAL:** Metal memory is manual at the boundary. Failure to follow these causes OOM crashes and GPU hangs.

### Memory Management Rules

1. **Before loading:** Set limits
   ```go
   mlx.SetCacheLimit(n)        // Set KV cache limit
   mlx.SetMemoryLimit(n)       // Set total memory limit
   ```

2. **Between model loads:** Clean up
   ```go
   model.Close()               // Release all GPU memory
   runtime.GC()                // Run garbage collector
   // Then load next model
   ```

3. **During probe/eval loops:** Clean up after each evaluation
   ```go
   for _, sample := range dataset {
       result := model.Evaluate(sample)
       runtime.GC()  // Critical!
   }
   ```

4. **`Close()` releases all GPU memory** back to Metal — without it, memory persists

### Memory Planning

```go
// Device info
deviceInfo := mlx.GetDeviceInfo()

// Model info
modelInfo := model.Info()

// Create residency plan
plan, err := mlx.CreateMemoryPlan(deviceInfo, modelInfo)

// Check if model fits
if plan.Fits() {
    // Load model
} else {
    // Use split inference or smaller model
}
```

### Metal Capabilities

```go
// Get backend capabilities
caps := mlx.GetMetalCapabilities()

// Check features
if caps.SupportsFP16 {
    // Use FP16
}
if caps.SupportsAttention {
    // Use optimized attention
}
```

---

## 📈 Sampling & Generation Options

### Sampling Strategies

| Strategy | Parameter | Behavior |
|----------|-----------|----------|
| **Greedy** | none | Highest logit, deterministic |
| **Temperature** | `WithTemperature(t)` | Softmax scaling (<1 sharp, >1 random) |
| **Top-P** | `WithTopP(p)` | Nucleus sampling, cumulative probability |
| **Top-K** | `WithTopK(k)` | Sample from top k tokens |
| **Min-P** | `WithMinP(p)` | Minimum probability threshold |

**Applied in sequence:** Temperature → Top-P → Top-K → Min-P

### Generate Options

```go
generateOptions := []mlx.GenerateOption{
    mlx.WithTemperature(0.8),
    mlx.WithTopP(0.9),
    mlx.WithTopK(50),
    mlx.WithMinP(0.1),
    mlx.WithMaxTokens(1024),
    mlx.WithStopSequences([]string{"\n\n", "<|eot|>"}),
    mlx.WithRepetitionPenalty(1.1),
    mlx.WithPresencePenalty(0.1),
    mlx.WithFrequencyPenalty(0.1),
    mlx.WithSpeculativeDecoding(true),
    mlx.WithThinkingMode(true),
    mlx.WithCacheMode(mlx.CacheModeFull),
}

response, err := model.Generate(prompt, generateOptions...)
```

### Cache Modes

```go
const (
    CacheModeNone      = "none"      // No KV cache
    CacheModeFull      = "full"      // Full KV cache
    CacheModeSliding   = "sliding"   // Sliding window cache
    CacheModeRotating  = "rotating"  // Rotating cache (256-token chunks)
    CacheModePaged     = "paged"     // Paged cache (block-oriented, for Gemma 4)
)
```

---

## 🔌 CoreGO & Inference Integration

### Inference Contract

go-mlx implements the full `inference.Backend` and `inference.TextModel` contracts:

```go
// Backend registration (auto-registered via init())
backend := mlx.Backend()

// Check availability
if backend.Available() {
    // Use backend
}

// TextModel contract
var textModel inference.TextModel = model

// Generate via contract
response, err := textModel.Generate(prompt, options...)
```

### With go-inference

```go
import (
    "dappco.re/go/inference"
    _ "dappco.re/go/mlx"  // Auto-registers "metal" backend
)

// Get Metal backend
backend := inference.GetBackend("metal")

// Check availability
if backend == nil || !backend.Available() {
    log.Fatal("Metal backend not available")
}

// Create model via inference contract
model, err := backend.LoadModel(modelPath, config)
```

### With go-ml

go-ml drives go-mlx without importing Metal directly:

```go
import "dappco.re/go/core/ml"

// ModelWorkflow drives training
err := ml.ModelWorkflow{
    Backend: "metal",
    Model:   modelPath,
    // ...
}.Run()
```

---

## 📦 CLI Tools

### `cmd/mlx` — Primary Entry Point (`lthn-mlx`)

The primary user-facing binary with comprehensive sub-commands:

| Command | Description |
|---------|-------------|
| `serve` | Mount OpenAI/Anthropic/Ollama-compatible HTTP listener on `:36911` |
| `generate` | Direct generation |
| `chat` | Chat with model |
| `embed` | Create embeddings |
| `diffuse` | Diffusion generation |
| `vision` | Vision tasks |
| `wav` | Audio generation |
| `multimodal` | Multimodal tasks |
| `sft` | Supervised Fine-Tuning |
| `ssd` | Self-Distillation |
| `tune` | General tuning |
| `split-ffn-tune` | Split FFN tuning |
| `pack` | Model packing |
| `fuse` | LoRA fusion |
| `score` | Content scoring |
| `ebook` | E-book generation |
| `memory-pretrain-build` | Memory pretraining |

**Admin HTTP Surface (`/v1/admin/*`):**
- Machine identity
- Model download (HF allowlist-gated)
- Hot-swap reload
- SFT kick-off
- Gated by 256-bit bearer token at `~/Lethean/data/admin.token`

**Features:**
- Embeds compiled Metal library (`embed_metallib.go`)
- Wails-based web frontend (`frontend/`)
- Logs metallib provenance for reproducibility (`metallib_provenance.go`)
- Menubar mode when launched from Finder (`.app` bundle)

### `cmd/violet` — Local Sidecar Daemon

Minimal local-native inference sidecar:

```bash
# Start violet daemon
violet -config violet.toml

# Or via environment
VIOLET_SOCKET_PATH=/tmp/violet.sock violet
```

**Features:**
- Unix domain socket server
- JSON-line framed protocol
- Loads models on first use
- Keeps models resident until exit
- Lightweight companion for native macOS apps

**Protocol:**
- One `Request` per frame (JSON)
- One `Response` returned per frame
- Streaming output is out of scope for v0 contract

### Daemon Infrastructure (`pkg/daemon/`)

Pure-Go server layer behind `cmd/violet`:

```go
// Create server
server := daemon.NewServer("/tmp/violet.sock")

// Register generate backend
server.Registry().RegisterGenerateBackend(mlxBackend)

// Start serving
server.ListenAndServe()
```

**Key Types:**
- `Request` / `Response` / `Message`
- `GenerateRequest` / `GenerateResult`
- `NativeGenerateRunner` with model cache (copy-on-write)

---

## 🎯 Pluggable Component Contracts (`pkg/scheme/`)

Three-registry contract layer for extensible components:

```go
// QuantScheme - weight quantisation format
type QuantScheme struct {
    ID          string
    StateKind  StateKind  // StateNone, StateKVCache, StateRecurrent
    // ...
}

// Mixer - sequence-mixing scheme
type Mixer struct {
    ID          string
    StateKind  StateKind
    // ...
}

// CacheScheme - KV storage strategy
type CacheScheme struct {
    ID          string
    StateKind  StateKind
    // ...
}

// Register new schemes
scheme.RegisterQuantScheme(&QuantScheme{ID: "affine", StateKind: StateNone})
scheme.RegisterMixer(&Mixer{ID: "softmax", StateKind: StateKVCache})
scheme.RegisterCacheScheme(&CacheScheme{ID: "q8", StateKind: StateKVCache})
```

**Pure Go with no driver tensor type** — Relocates to go-inference unchanged, so every backend (Metal, ROCm, CPU) inherits one catalogue.

---

## 🔄 State & Bundle Management

### Session State (`session/`)

High-level `Session` type for agent-memory lifecycle:

```go
// Wake agent memory (restore durable KV prefix)
err := session.WakeAgentMemory(wakeOptions)

// Sleep agent memory (stream to durable blocks)
bundle, err := session.SleepAgentMemory(sleepOptions)

// Export artifacts
artifacts, err := session.ExportArtifacts(exportOptions)
```

### KV Snapshot (`kv/`)

Durable KV state with analysis:

```go
// Create snapshot
snapshot := kv.NewSnapshot()

// Add layer data
snapshot.AddLayerKV(layerIndex, key, value)

// Analyze quality
analysis := kv.Analyze(snapshot)

// Check scores
if analysis.Composite > 8000 {
    // High quality
}
```

### Portable Bundles (`bundle/`)

Portable session-state artifacts:

```go
// Create bundle
bundle := bundle.New(kvSnapshot, tokenizer, runtimeConfig)

// Validate compatibility
if err := bundle.Validate(); err != nil {
    // Incompatible
}

// Save to file
bundle.SaveToFile("session.mlxbundle")

// Load from file
loadedBundle, err := bundle.LoadFromFile("session.mlxbundle")

// Restore from bundle
session.RestoreFromBundle(loadedBundle)
```

### Artifact Export (`artifact/`)

Session-state export records:

```go
// Export to inference/state store
record := artifact.Export(snapshot, options)

// Write to store
artifact.WriteToStore(record, storeURI)
```

### Block Cache (`blockcache/`)

Block-prefix cache metadata:

```go
// Create block cache
cache := blockcache.NewService(
    blockcache.WithBlockSize(128),
    blockcache.WithMaxBlocks(1000),
)

// Hash and cache prompt prefix
blockRef := cache.HashPromptPrefix(prompt)

// Warm KV from cache
cache.WarmKVFromBlockRef(blockRef)
```

---

## 📊 Observability & Probing

### Probe System (`probe/`)

Observability vocabulary for go-mlx:

```go
// Sink interface for events
type Sink interface {
    Receive(event Event)
}

// Bus fans events to multiple sinks
bus := probe.NewBus()
bus.AddSink(sink1)
bus.AddSink(sink2)

// Recorder stores events for tests
recorder := probe.NewRecorder()
bus.AddSink(recorder)
```

**Event Kinds:**
- `KindToken` — Token generated
- `KindLogits` — Logits computed
- `KindEntropy` — Entropy measured
- `KindSelectedHeads` — Attention heads selected
- `KindLayerCoherence` — Layer coherence
- `KindRouterDecision` — MoE router decision
- `KindExpertResidency` — Expert residency
- `KindResidual` — Residual computation
- `KindCachePressure` — Cache pressure
- `KindMemoryPressure` — Memory pressure
- `KindTraining` — Training step
- `KindScore` — Scoring event

### InfluxDB Sink

```go
// Create InfluxDB line protocol sink
sink := probe.NewLineProtocolSink(
    probe.WithFilePath("/tmp/metrics.lp"),
    probe.WithHTTPEndpoint("http://influxdb:8086"),
)

// Add to bus
bus.AddSink(sink)

// Events are emitted as v0-schema InfluxDB lines
// measurement: training_loss, content_score
```

---

## 🏷️ Model Operations

### Model Packing (`pack/`)

```go
// Pack model with metadata
packResult, err := mlx.PackModel(modelPath, packConfig)
```

### LoRA Fusion (`lora/`)

```go
// Fuse LoRA adapters
fusedModel, err := mlx.FuseLoRA(baseModelPath, loraPaths...)
```

### Model Merging (`merge/`)

```go
// Merge models (Linear, SLERP, TIES, DARE)
mergedModel, err := mlx.MergeModels(
    modelPaths,
    mlx.WithMergeMethod(mlx.MergeMethodLinear),
    mlx.WithMergeWeights(weights),
)
```

### GGUF Quantisation

```go
// Quantize to GGUF
quantizedPath, err := mlx.QuantizeToGGUF(
    modelPath,
    mlx.WithGGUFQuantization(mlx.GGUFQuantTypeQ4_0),
)
```

### KV Snapshots

```go
// Create KV snapshot
snapshot := mlx.CreateKVSnapshot(model)

// Save snapshot
snapshot.SaveToFile("kv_snapshot.mlxkv")

// Load snapshot
loadedSnapshot, err := mlx.LoadKVSnapshot("kv_snapshot.mlxkv")

// Restore to model
model.RestoreKVSnapshot(loadedSnapshot)
```

---

## 🎨 Code Examples

### Example: Complete Inference Application

```go
package main

import (
    "context"
    "log"
    
    "dappco.re/go/mlx"
)

func main() {
    // Check availability
    if !mlx.Available() {
        log.Fatal("Metal not available")
    }
    
    // Set memory limits
    mlx.SetCacheLimit(4 * 1024 * 1024 * 1024)  // 4GB cache
    mlx.SetMemoryLimit(8 * 1024 * 1024 * 1024)  // 8GB total
    
    // Load model
    model, err := mlx.LoadModel("/Models/gemma-3-4b",
        mlx.WithContextLength(4096),
        mlx.WithQuantization(4),
    )
    if err != nil {
        log.Fatal(err)
    }
    defer model.Close()
    
    // Print model info
    info := model.Info()
    log.Printf("Loaded %s with %d layers", info.Architecture, info.NumLayers)
    
    // Create chat session
    session := model.NewChatSession()
    
    // Multi-turn conversation
    messages := []string{
        "You are a helpful AI assistant.",
        "What is Lethean?",
    }
    
    for i := 0; i < 5; i++ {
        response, err := session.Chat(messages,
            mlx.WithTemperature(0.7),
            mlx.WithMaxTokens(256),
        )
        if err != nil {
            log.Fatal(err)
        }
        
        log.Printf("Assistant: %s\n", response)
        messages = append(messages, response)
        
        // Runtime GC to prevent memory buildup
        runtime.GC()
    }
    
    // Score the last response
    result := score.Score(response)
    log.Printf("LEK Score: %.2f", result.Composite)
}
```

### Example: Fine-Tuning with LoRA

```go
// Load base model
model, err := mlx.LoadModel("gemma-3-4b",
    mlx.WithContextLength(2048),
)
if err != nil {
    log.Fatal(err)
}
defer model.Close()

// Prepare dataset
dataset, err := mlx.LoadDataset("path/to/dataset.jsonl")
if err != nil {
    log.Fatal(err)
}

// Train with LoRA
trainResult, err := model.TrainSFT(
    dataset,
    &mlx.SFTConfig{
        LoRAConfig: &mlx.LoRAConfig{
            Rank:           8,
            Alpha:          16,
            TargetModules: []string{"q_proj", "k_proj", "v_proj", "o_proj"},
            Dropout:        0.1,
        },
        Epochs:        3,
        BatchSize:     4,
        LearningRate:  1e-5,
        WarmupSteps:   100,
        MaxSeqLength:  2048,
    },
)
if err != nil {
    log.Fatal(err)
}

log.Printf("Training complete: %+v", trainResult)

// Save LoRA adapter
adapterPath := "lora_adapter.mlx"
err = model.SaveLoRA(adapterPath, "my-lora")
if err != nil {
    log.Fatal(err)
}

// Apply LoRA for inference
model.ApplyLoRA(adapterPath)

// Generate with fine-tuned model
response, err := model.Generate("Test prompt",
    mlx.WithTemperature(0.7),
    mlx.WithMaxTokens(256),
)
```

### Example: Session with State Persistence

```go
// Create model
model, _ := mlx.LoadModel("gemma-3-4b")
defer model.Close()

// Create session
session := model.NewSession()

// Initial prompt
prompt := "Tell me about the Lethean project."

// Generate and capture state
response1, _ := session.Generate(prompt, mlx.WithMaxTokens(256))

// Sleep to durable storage
bundle, _ := session.Sleep()
bundle.SaveToFile("conversation.mlxbundle")

// Later: Wake from bundle
newSession := model.NewSession()
newSession.WakeFromBundle("conversation.mlxbundle")

// Continue conversation (0% replay - no prompt repetition)
prompt2 := "What are its key features?"
response2, _ := newSession.Generate(prompt2, mlx.WithMaxTokens(256))

// The KV state from the first interaction is restored
// No need to repeat the initial prompt
```

### Example: Speculative Decoding

```go
// Load main model
mainModel, _ := mlx.LoadModel("gemma-4-9b")
defer mainModel.Close()

// Load draft model (smaller, faster)
draftModel, _ := mlx.LoadModel("gemma-2b")
defer draftModel.Close()

// Enable speculative decoding
response, _ := mainModel.Generate(
    prompt,
    mlx.WithSpeculativeDecoding(true),
    mlx.WithDraftModel(draftModel),
    mlx.WithMaxTokens(1024),
)

// Or use Gemma 4's native drafter
gemma4Model, _ := mlx.LoadModel("gemma-4-9b-it-assistant")
// Native drafter is automatically detected
response, _ = gemma4Model.Generate(
    prompt,
    mlx.WithSpeculativeDecoding(true),  // Uses native drafter
    mlx.WithMaxTokens(1024),
)
```

### Example: Split Inference for Large Models

```go
// Configure split placement
placement := &mlx.SplitPlacement{
    NativeRuntime: &mlx.NativeRuntimeConfig{
        // Keep attention layers on GPU
        LayerRange: &mlx.LayerRange{Start: 0, End: 20},
    },
    CPUFFN: &mlx.CPUFFNConfig{
        // Offload FFN layers 20-40 to CPU
        Enabled:      true,
        LayerRange:   &mlx.LayerRange{Start: 20, End: 40},
    },
    RemoteFFN: &mlx.RemoteFFNConfig{
        // Send layers 40-60 to remote server
        Enabled:   true,
        Endpoint:  "http://remote-gpu:8080",
        LayerRange: &mlx.LayerRange{Start: 40, End: 60},
    },
}

// Load model with split placement
model, err := mlx.LoadModel("gemma-4-32b",
    mlx.WithSplitPlacement(placement),
)
if err != nil {
    log.Fatal(err)
}
defer model.Close()

// Generate normally - layers are automatically distributed
response, _ := model.Generate(prompt, mlx.WithMaxTokens(512))
```

### Example: Content Scoring with LEK

```go
import "dappco.re/go/mlx/pkg/score"

text := "This is a sample text to evaluate."

// Score across all axes
result := score.Score(text)

// Check overall quality
if result.Composite > 7000 {
    log.Println("High quality content")
} else if result.Composite > 5000 {
    log.Println("Medium quality content")
} else {
    log.Println("Low quality content")
}

// Check specific concerns
if result.HostilityScore > 0.8 {
    log.Println("WARNING: Hostile content detected")
}

if result.SycophancyScore > 0.7 {
    log.Println("WARNING: Sycophantic content detected")
}

// Axis breakdown
log.Printf("LEK: %.2f", result.LEKScore)
log.Printf("Sycophancy: %.2f", result.SycophancyScore)
log.Printf("Hostility: %.2f", result.HostilityScore)
log.Printf("Authority: %.2f", result.AuthorityScore)
log.Printf("Dialect: %.2f", result.DialectScore)
log.Printf("Differential: %.2f", result.DifferentialScore)
log.Printf("Phonetic: %.2f", result.PhoneticScore)
```

---

## 🔧 Build & Development

### Prerequisites

```bash
# macOS with Apple Silicon (M1-M4)
# Xcode with Metal support
# Go 1.21+
# CMake 3.25+
# Git for submodules
```

### Building mlx-c

```bash
# Clone with submodules
git clone --recursive https://github.com/dAppCore/go-mlx.git
cd go-mlx

# Update submodules
git submodule update --init --recursive

# Build mlx-c library (~2 min on M3 Ultra)
go generate ./...

# The compiled libraries are in dist/lib/
# Headers are in dist/include/ (committed for module consumers)
```

### Build Tags

```go
//go:build darwin && arm64 && !nomlx

// On non-darwin/arm64 or with -tags nomlx:
// Package compiles without CGO and Available() returns false
```

### Clean Rebuild

```bash
# Clean and rebuild
rm -rf build dist && go generate ./...
```

### Testing

```bash
# Run all tests
go test ./...

# Run specific test
go test -run TestRMSNorm_Good ./go/internal/metal/

# Run benchmarks
go test -bench=. -benchtime=2s ./go/internal/metal/

# On sandboxed systems
GOCACHE=/tmp/codex-go-mlx-cache go test ./...
```

### CGO Flags

```bash
# Common CGO flags
CGO_CFLAGS="-I./dist/include" \
CGO_LDFLAGS="-L./dist/lib -lmlx" \
go build
```

---

## 📊 Performance Considerations

### Model Loading

- **Lazy Evaluation:** Operations build computation graph, dispatched to Metal only on `Eval()`
- **`Detach()`:** Breaks graph connections to free GPU memory between generation steps
- **Finalizers:** `Array` uses `runtime.SetFinalizer` to call `mlx_array_free`
- **Explicit Free:** Call `Free()` for immediate release

### Memory Optimization

- **Layer-wise Quantisation:** Different quantization per layer (e.g., router always 8-bit for Gemma 4)
- **KV Cache Modes:** Standard, rotating, paged - choose based on use case
- **Shared Prefix:** RadixAttention for shared prefix KV
- **Continuous Batching:** Optimal batch processing

### Benchmarks

```bash
# Inference benchmarks
go test -bench=. -benchtime=2s ./...

# Memory plan benchmarks
go test -bench=BenchmarkMemoryPlan ./...

# KV cache benchmarks
go test -bench=BenchmarkKVCache ./...
```

---

## 🏆 Key Features Summary

### ✅ What go-mlx Provides

1. **Native Metal Acceleration** — Direct Metal compute shader execution
2. **Inference Backend** — Full `inference.Backend` contract implementation
3. **Text Model** — Full `inference.TextModel` contract
4. **Multi-Model Support** — 19+ registered architectures
5. **Session/State Engine** — Persistent sessions with Wake/Sleep
6. **Speculative Decoding** — Faster generation with draft models
7. **Split Inference** — Cross-runtime placement for large models
8. **Native Training** — SFT, LoRA, SSD on Metal GPU
9. **LEK Scorer** — Tier-1 content scoring
10. **Quantisation** — AutoRound with per-component control
11. **FFN-Memory Pretraining** — Hierarchical memory pretraining
12. **MoE Support** — Mixture-of-Experts with sparse routing
13. **Multimodal** — Vision encoder support
14. **CLI Tools** — `lthn-mlx` and `violet` binaries
15. **Daemon Infrastructure** — Unix socket server for local inference
16. **Pluggable Components** — Quant, mixer, cache scheme registries

### ❌ What go-mlx Does NOT Do

1. **No Cross-Platform** — macOS arm64 only (stub returns false on other platforms)
2. **No Non-Apple GPU** — Use go-cuda for NVIDIA, go-rocm for AMD
3. **No CPU-Only** — Requires Metal GPU (Apple Silicon)
4. **No External Dependencies** — mlx-c is vendored and built from source
5. **No Windows/Linux** — Metal is Apple-only

---

## 📚 Testing

### Test Patterns

All tests follow the **Good/Bad/Ugly** triplet pattern:
- `_Good` — happy path
- `_Bad` — expected error conditions
- `_Ugly` — panic / edge cases

**Model Test Skipping:**
```go
// Tests requiring model files on disk use t.Skip()
func TestGeneration_Good(t *testing.T) {
    modelPath := "/Volumes/Data/lem/safetensors/gemma-3-4b"
    if !filepath.Exists(modelPath) {
        t.Skip("Model not available")
    }
    // Test...
}
```

### Test Categories

```
Root Package Tests:
- mlx_test.go (20K lines) - Main package tests
- backend_test.go (71K lines) - Backend implementation tests
- generate_test.go - Generation tests
- inference_contract_test.go (23K lines) - Inference contract tests
- speculative_test.go (21K lines) - Speculative decoding tests
- tokeniser_test.go - Tokenisation tests
- training tests (sft_test.go, ssd_test.go, etc.)

Internal/Metal Tests:
- All Metal engine tests with Good/Bad/Ugly triplets
- Model-specific tests for each architecture
- KV cache tests
- Memory management tests
```

---

## 📖 References

### RFC Specifications

- **[RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/mlx/RFC.md)** — Complete technical specification
- **[RFC.serving.md](file:///Users/snider/Code/meowmix/plans/code/core/go/mlx/RFC.serving.md)** — Serving & throughput engine specification

### Related Documentation

- **[CLAUDE.md](file:///Users/snider/Code/core/go-mlx/CLAUDE.md)** — Implementation details, file map, build instructions
- **[AGENTS.md](file:///Users/snider/Code/core/go-mlx/AGENTS.md)** — Agent-specific guidance
- **[CoreGO RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md)** — Framework specification
- **[go-inference RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/inference/RFC.md)** — Inference contracts
- **[go-ml RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/ml/RFC.md)** — ML backends & scoring

### Local Documentation

- `docs/index.md` — Top-level guide, feature matrix, supported models
- `docs/architecture.md` — CGO binding, model architectures, weight loading
- `docs/compute.md` — Frame-oriented Metal compute, pixel buffers, kernels
- `docs/models.md` — Model loading, architectures, tokenisation, chat templates
- `docs/training.md` — LoRA fine-tuning, AdamW, gradient computation
- `docs/distillation.md` — Knowledge distillation (KL, soft cross-entropy)
- `docs/grpo.md` — Group-relative policy optimisation for RL
- `docs/eval.md` — Dataset-native perplexity, quality probes, eval reports
- `docs/model-operations.md` — Model merge, GGUF quantise, KV snapshot
- `docs/development.md` — Prerequisites, CGO flags, test patterns
- `docs/build.md` — Build pipeline, CMake, build tags
- `docs/history.md` — Completed phases, commit hashes, limitations
- `examples/` — Per-feature usage examples

### Source Code

- **Repository:** `github.com/dAppCore/go-mlx` / `dappco.re/go/mlx`
- **Module:** `dappco.re/go/mlx`
- **Package:** `dappco.re/go/mlx`
- **Go Version:** 1.21+
- **Platform:** darwin/arm64

### Dependencies

```
dappco.re/go                    # CoreGO framework
dappco.re/go/core/inference   # Inference contracts
dappco.re/go/core/io           # io.Medium for model staging
mlx-c (via CGO)                  # MLX framework C bindings (vendored)
lib/mlx/ (submodule)            # Upstream MLX v0.31.1 (pinned)
```

### External Resources

- **MLX Framework:** https://ml-explore.github.io/mlx/
- **Apple Metal:** https://developer.apple.com/metal/
- **Gemma Models:** https://ai.google.dev/gemma
- **Qwen Models:** https://qwenlm.github.io/

---

## 🏷️ Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/mlx` |
| **Repository** | `core/go-mlx` |
| **Type** | Library |
| **Status** | Production |
| **Tier** | lib (foundation) |
| **Platform** | darwin/arm64 only |
| **Lines** | ~500,000+ (estimated total) |
| **Source Files** | 200+ (root + subpackages) |
| **Test Files** | 100+ |
| **Submodules** | mlx (v0.31.1 pinned) |
| **CGO** | Required (mlx-c bindings) |
| **Test Coverage** | High |
| **Licence** | EUPL-1.2 |
| **Language** | Go 1.21+ |
| **Author** | Purberus <purberus@lthn.ai> |
| **Created** | 2026-06-18T07:00:00Z |
| **Version** | 1.0.0 |

---

## 🎯 Subpackage Overview

| Subpackage | Purpose | Files |
|------------|---------|-------|
| `pkg/metal/` | Metal engine core | ~100 |
| `pkg/metal/model/<arch>/` | Per-architecture implementations | ~20 |
| `session/` | Session/state engine | ~10 |
| `kv/` | KV snapshot and block storage | ~10 |
| `spine/` | Shared types and helpers | ~10 |
| `train/` | Training infrastructure | ~20 |
| `memorypretrain/` | FFN-memory pretraining | ~15 |
| `probe/` | Observability and metrics | ~10 |
| `profile/` | Architecture/algorithm registry | ~10 |
| `agent/` | Wake/sleep orchestration | ~10 |
| `bundle/` | Portable state bundles | ~10 |
| `artifact/` | Session-state export | ~10 |
| `blockcache/` | Block-prefix cache | ~10 |
| `chaptersmoke/` | KV smoke benchmarks | ~5 |
| `adapter/` | Inference model adapter | ~5 |
| `pkg/daemon/` | Violet daemon | ~10 |
| `pkg/scheme/` | Pluggable component contracts | ~5 |
| `pkg/score/` | LEK content scorer | ~10 |
| `pkg/memvid/` | State store compatibility shim | ~5 |
| `cmd/mlx/` | CLI tool (`lthn-mlx`) | ~30 |
| `cmd/violet/` | Sidecar daemon | ~5 |

---

*Documentation generated from source code analysis and RFC specification.*
*Last updated: 2026-06-18T07:00:00Z*
