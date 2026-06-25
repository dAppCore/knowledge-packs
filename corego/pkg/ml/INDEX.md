---
type: Package Index
package: ml
module: dappco.re/go/core/ml
repo: core/go-ml
title: go-ml Package Index
description: ML inference backends, scoring engine, and agent orchestrator with 23 capability probes, GGUF support, and dual-interface architecture
tags:
  - ai
  - ml
  - machine-learning
  - inference
  - scoring-engine
  - judge
  - benchmark
  - capability-probes
  - gguf
  - agent-orchestrator
  - dual-interface
---
# go-ml Package Index


**Repository:** `core/go-ml`  
**Module:** `dappco.re/go/core/ml`  
**Status:** ✅ Production-Ready  
**License:** EUPL-1.2  
**Language:** Go 1.25+  
**Dependencies:** Core framework, go-inference, testify  
**RFC:** [plans/code/core/go/ml/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/ml/RFC.md)  
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| RFC | Request for Comments specification | [plans/code/core/go/ml/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/ml/RFC.md) |
| CLAUDE.md | Development guidance | [~/Code/core/go-ml/CLAUDE.md](file:///Users/snider/Code/core/go-ml/CLAUDE.md) |
| AGENTS.md | Agent guidance | [~/Code/core/go-ml/AGENTS.md](file:///Users/snider/Code/core/go-ml/AGENTS.md) |
| GOAL.md | Quality findings | [~/Code/core/go-ml/GOAL.md](file:///Users/snider/Code/core/go-ml/GOAL.md) |
| Architecture | Design rationale | [~/Code/core/go-ml/docs/architecture.md](file:///Users/snider/Code/core/go-ml/docs/architecture.md) |
| Development Guide | Building, testing, standards | [~/Code/core/go-ml/docs/development.md](file:///Users/snider/Code/core/go-ml/docs/development.md) |
| Models Guide | Supported models and architectures | [~/Code/core/go-ml/docs/models.md](file:///Users/snider/Code/core/go-ml/docs/models.md) |

---

## Package overview

`go-ml` is the **ML inference backend and scoring engine** for the Lethean AI stack, providing a platform for AI model execution, evaluation, and management.

### Core capabilities

1. **Pluggable Inference Backends** — Multiple backend types (HTTP, Llama, Inference adapters)
2. **Multi-Suite Scoring Engine** — Concurrent evaluation across 5 scoring suites
3. **23 Capability Probes** — Binary pass/fail tests across AI capabilities
4. **GGUF Model Management** — Complete GGUF format parsing, conversion, and inventory
5. **Agent Orchestrator** — SSH-based remote checkpoint discovery and evaluation
6. **CLI & REST API** — Full command-line and HTTP interface

### Design philosophy

- **Dual-Interface Architecture** — Supports both `ml.Backend` (result-based) and `inference.TextModel` (streaming)
- **Adapter Pattern** — Bridging between interface families
- **Backend-Agnostic** — Swap backends without changing consumer code
- **Comprehensive Evaluation** — Multiple scoring methods for different use cases
- **Production-First** — Deployed in Lethean AI stack

---

## Architecture

### High-level overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        go-ml: ML Execution Layer                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    BACKEND LAYER                                        │ │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                  │ │
│  │  │ HTTPBackend │ │ LlamaBackend │ │ Inference   │                  │ │
│  │  │ (Ollama/LM  │ │ (subprocess) │ │ Adapter     │                  │ │
│  │  │  Studio)    │ │             │ │ (go-mlx/   │                  │ │
│  │  │             │ │             │ │  go-rocm)   │                  │ │
│  │  └─────────────┘ └─────────────┘ └─────────────┘                  │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    SCORING LAYER                                         │ │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │ │
│  │  │Heuristic│ │Semantic │ │ Content │ │Standard │ │ Exact   │      │ │
│  │  │(Regex)   │ │(LLM)    │ │(Probes) │ │(Bench)  │ │(Match)  │      │ │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘      │ │
│  │                    ScoringEngine.ScoreAll()                             │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    PROBES LAYER (23 Capability Tests)                     │ │
│  │  Math: Arithmetic, Algebra, Calculus, Logic, Puzzles                  │ │
│  │  Code: Syntax, Function, Algorithm, Debug, Optimize                    │ │
│  │  Reasoning: CoT, Common Sense, Analogy, Deduction, Induction          │ │
│  │  Language: Vocabulary, Grammar, Comprehension, Memory, Knowledge      │ │
│  │  Specialized: Creativity, Translation, Summarization                    │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    GGUF LAYER                                            │ │
│  │  Parse, Convert, Inventory, Metadata                                      │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    AGENT LAYER                                           │ │
│  │  Checkpoint Discovery → Evaluation → InfluxDB/DuckDB Streaming          │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    INTERFACE LAYER                                        │ │
│  │  CLI (lem)                REST API (/v1/ml/)                              │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Dual-interface system

The key architectural innovation is the **dual-interface pattern**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DUAL INTERFACE ARCHITECTURE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────┐      ┌─────────────────────────────┐   │
│  │          ml.Backend             │      │     inference.TextModel       │   │
│  │  (Result-based interface)      │      │    (Token-streaming interface) │   │
│  │                                   │      │                                 │   │
│  │  Generate() → *Result            │      │  Generate() → iter.Seq[Token] │   │
│  │  Chat() → *Result                │      │  Chat() → iter.Seq[Token]     │   │
│  │  Classify() → *ClassifyResult   │      │  Close() → error              │   │
│  │  Embed() → []float32           │      │  Info() → ModelInfo           │   │
│  │  HealthCheck() → error          │      │                                 │   │
│  └──────────────────────────────┘      └─────────────────────────────┘   │
│                                  ▲                     ▲                        │
│                                  │                     │                        │
│                    ┌─────────────────────────────┐                    │
│                    │           ADAPTERS             │                    │
│                    │  Bridge between interfaces    │                    │
│                    │                                 │                    │
│                    │  InferenceAdapter:            │                    │
│                    │    inference.TextModel →      │                    │
│                    │    ml.Backend                │                    │
│                    │                                 │                    │
│                    │  HTTPTextModel:               │                    │
│                    │    ml.HTTPBackend →           │                    │
│                    │    inference.TextModel        │                    │
│                    │                                 │                    │
│                    │  LlamaTextModel:              │                    │
│                    │    ml.LlamaBackend →          │                    │
│                    │    inference.TextModel        │                    │
│                    └─────────────────────────────┘                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Why Two Interfaces?**

| Aspect | `ml.Backend` | `inference.TextModel` |
|--------|--------------|----------------------|
| **Output Type** | `*Result` (complete) | `iter.Seq[Token]` (streaming) |
| **Error Handling** | `Result.Error` field | `token.Err()` after loop |
| **Use Case** | Simple, result-oriented | Token-level control, streaming |
| **Preferred For** | HTTP, Llama backends | GPU backends (Metal, ROCm) |
| **Maturity** | Production, stable | New, evolving |

### Repository structure

```
go/
├── Core Interface Files
│   ├── inference.go           # ml.Backend interface + Result/Message types
│   ├── contracts.go          # Interface compliance contracts
│   └── options.go            # Configuration options
│
├── Backend Implementations
│   ├── backend.go            # Backend base functionality
│   ├── backend_http.go       # HTTPBackend (Ollama, LM Studio, OpenAI-compat)
│   ├── backend_llama.go      # LlamaBackend (managed subprocess)
│   ├── adapter.go            # InferenceAdapter (bridges to inference.TextModel)
│   ├── backend_http_textmodel.go  # HTTP → inference.TextModel adapter
│   └── backend_llama_textmodel.go # Llama → inference.TextModel adapter
│
├── Scoring Engine
│   ├── score.go              # ScoringEngine + ScoreAll()
│   ├── judge.go              # LLM-based judge
│   ├── heuristic.go          # Heuristic (regex) scoring
│   └── contracts_test.go     # Interface compliance tests
│
├── Capability Probes
│   ├── probes.go             # 23 probe definitions
│   ├── probes_test.go        # Probe tests (Good/Bad/Ugly)
│   └── probes_*.go            # Individual probe implementations
│
├── GGUF Support
│   └── gguf/                 # GGUF format handling
│       ├── gguf.go           # Core GGUF types and parsing
│       ├── parser.go         # GGUF file parser
│       ├── convert.go        # Format conversion
│       ├── inventory.go      # Model inventory management
│       └── metadata.go       # Metadata extraction
│
├── Agent Orchestrator
│   └── agent/                # Remote evaluation system
│       ├── orchestrator.go   # Main orchestrator
│       ├── ssh.go            # SSH transport
│       ├── transport.go      # Transport interface
│       ├── checkpoint.go      # Checkpoint discovery
│       ├── eval.go           # Evaluation coordinator
│       └── stream.go         # InfluxDB/DuckDB streaming
│
├── Service Layer
│   ├── service.go            # Core service integration
│   └── service_test.go       # Service tests
│
├── CLI Commands
│   └── cmd/                  # Command-line interface
│       ├── cmd_bench/        # Benchmarking commands
│       │   ├── bench.go
│       │   └── ...
│       ├── cmd_gguf/         # GGUF management
│       │   └── gguf.go
│       ├── cmd_judge/        # Scoring/judging
│       │   └── judge.go
│       ├── cmd_probe/        # Capability probes
│       │   └── probe.go
│       └── cmd_agent/        # Agent orchestrator
│           └── agent.go
│
└── REST API
    └── api/                  # HTTP API layer
        ├── routes.go         # Route definitions
        ├── handlers.go       # Request handlers
        └── middleware.go     # Middleware
```

---

## Core interfaces

### ml.Backend

The primary interface for go-ml backends:

```go
type Backend interface {
    // Name identifies the backend
    Name() string
    
    // Generate text from a prompt
    Generate(ctx context.Context, prompt string, opts Options) (*Result, error)
    
    // Chat completion in conversational context
    Chat(ctx context.Context, messages []Message, opts Options) (*Result, error)
    
    // Classify text into categories
    Classify(ctx context.Context, text string, categories []string) (*ClassifyResult, error)
    
    // Generate embeddings
    Embed(ctx context.Context, text string) ([]float32, error)
    
    // Health verification
    HealthCheck(ctx context.Context) error
}

// Result contains the complete generation output
type Result struct {
    Text     string            // Generated text
    Tokens   []Token          // Individual tokens
    Metrics  map[string]float64 // Performance metrics
    Error    error            // Any error that occurred
    Duration time.Duration     // Time taken
}

// Token represents a single generated token
type Token struct {
    Text string
    ID   int
    LogProb float64
}

// Message for chat conversations
type Message struct {
    Role    string // "system", "user", "assistant"
    Content string
}

// Options for generation configuration
type Options struct {
    MaxTokens     int
    Temperature   float64
    TopP         float64
    TopK         int
    RepeatPenalty float64
    StopSequences []string
    Stream       bool
}
```

### Interface comparison

| Feature | ml.Backend | inference.TextModel |
|---------|------------|---------------------|
| Return Type | `*Result` | `iter.Seq[Token]` |
| Error Location | `Result.Error` | `token.Err()` |
| Streaming | Optional (Stream field) | Native (iterator) |
| Token Access | `Result.Tokens[]` | Iterator yields |
| Best For | Simple use cases | Token-level control |

---

## Backend implementations

### 1. HTTPBackend

OpenAI-compatible HTTP API backend for external LLM services:

```go
// Create a new HTTP backend
backend := ml.NewHTTPBackend("http://localhost:11434", "gemma4-1b")

// With configuration
backend := ml.NewHTTPBackend(
    "http://localhost:11434",
    "qwen3:8b",
    ml.WithHTTPTimeout(60*time.Second),
    ml.WithHTTPExtraHeaders(map[string]string{
        "Authorization": "Bearer ...",
        "Content-Type": "application/json",
    }),
    ml.WithHTTPInsecure(true),
)
```

**Supported Services:**
- **Ollama** — Local LLM inference (`http://localhost:11434`)
- **LM Studio** — Local LLM management
- **OpenAI-compatible** — Any OpenAI-compatible API
- **Custom HTTP** — Any REST-based inference endpoint

**Features:**
- Automatic JSON serialization/deserialization
- Configurable timeouts
- Custom headers support
- Streaming response support
- Health check endpoint

### 2. LlamaBackend

Managed llama-server subprocess backend:

```go
// Create with explicit paths
backend := ml.NewLlamaBackend(
    "/usr/local/bin/llama-server",
    "/path/to/model.gguf",
    ml.WithLlamaPort(8080),
    ml.WithLlamaHost("localhost"),
    ml.WithLlamaContextLength(4096),
    ml.WithLlamaThreads(4),
    ml.WithLlamaBatchSize(512),
)

// Start the server
if err := backend.Start(ctx); err != nil {
    panic(err)
}
defer backend.Stop(ctx)

// Health check
if err := backend.HealthCheck(ctx); err != nil {
    log.Fatal("Backend not healthy")
}

// Use the backend
resp, err := backend.Generate(ctx, "Hello, world!")
```

**Features:**
- Automatic subprocess lifecycle management
- Dynamic port allocation (if port 0 specified)
- Health monitoring
- GGUF model loading
- Context length configuration
- Multi-threading support
- Batch processing

### 3. InferenceAdapter

Bridges go-inference TextModel backends into ml.Backend:

```go
// Load a model via go-inference (auto-selects backend)
model, err := inference.LoadModel("/path/to/model/")
if err != nil {
    panic(err)
}
defer model.Close()

// Wrap as ml.Backend
adapter := ml.NewInferenceAdapter(model)

// Now use ml.Backend interface
resp, err := adapter.Generate(ctx, "Explain AI", ml.Options{
    MaxTokens: 256,
})
```

**How It Works:**
1. Takes an `inference.TextModel` (from go-mlx, go-rocm, etc.)
2. Wraps it to implement `ml.Backend` interface
3. Collects all tokens from the stream
4. Returns them as a `*Result`

**Backend Connection:**
- Automatically connects to all registered inference backends
- Respects platform-specific availability
- Falls back gracefully

### Reverse adapters

For using ml.Backend where inference.TextModel is expected:

```go
// Wrap HTTPBackend as inference.TextModel
httpBackend := ml.NewHTTPBackend("http://localhost:11434", "model")
textModel := ml.NewHTTPTextModel(httpBackend)

// Now can use in inference context
for token := range textModel.Generate(ctx, "Hello") {
    fmt.Print(token.Text)
}
```

---

## Scoring engine

### Architecture

The scoring engine evaluates responses **concurrently** across multiple scoring suites:

```go
// Create scoring engine
engine := ml.NewScoringEngine(
    backend,
    ml.WithSuites("heuristic,semantic,content,standard,exact"),
    ml.WithConcurrency(4),
    ml.WithWorkerPoolSize(10),
)

// Score multiple responses concurrently
scores := engine.ScoreAll(ctx, responses)

// Score a single response
score := engine.Score(ctx, response, "semantic")

// Score with specific suites
score := engine.ScoreWithSuites(ctx, response, []string{"heuristic", "semantic"})
```

### Scoring suites

#### 1. Heuristic Suite

**Type:** Regex-based pattern matching  
**LLM Required:** ❌ No  
**Speed:** Fast  

```go
// Define custom heuristic rules
rules := []ml.HeuristicRule{
    {
        Name:    "positive-sentiment",
        Pattern: `(?i)good|great|excellent|wonderful|fantastic`,
        Score:   1.0,
        Weight:  1.0,
    },
    {
        Name:    "negative-sentiment",
        Pattern: `(?i)bad|terrible|awful|poor|horrible`,
        Score:   -1.0,
        Weight:  1.0,
    },
    {
        Name:    "contains-answer",
        Pattern: `(?i)answer|response|result`,
        Score:   0.5,
        Weight:  0.5,
    },
}

// Run heuristic scoring
score := engine.ScoreHeuristic(ctx, response, rules)
```

**Use Cases:**
- Quick pattern-based evaluation
- Keyword detection
- Simple compliance checks
- High-speed filtering

#### 2. Semantic Suite

**Type:** LLM-based semantic evaluation  
**LLM Required:** ✅ Yes (judge model)  
**Speed:** Moderate  

```go
// Semantic evaluation with judge model
semanticScore := engine.ScoreSemantic(ctx, response, ml.SemanticCriteria{
    Relevance:   1.0,  // How relevant is the response?
    Helpfulness: 1.0,  // How helpful is the response?
    Accuracy:    1.0,  // How accurate is the response?
    Completeness: 0.5, // How complete is the response?
})
```

**Criteria:**
- **Relevance** — Does the response address the prompt?
- **Helpfulness** — Is the response useful?
- **Accuracy** — Is the response factually correct?
- **Completeness** — Does the response cover all aspects?

#### 3. Content Suite

**Type:** Sovereignty and alignment probes  
**LLM Required:** ✅ Yes (judge model)  
**Speed:** Moderate  

```go
// Evaluate against content policies
contentScore := engine.ScoreContent(ctx, response, ml.ContentPolicies{
    Truthfulness: 1.0,
    Safety:       1.0,
    Alignment:    1.0,
    Sovereignty:  1.0,
    Ethical:      1.0,
})
```

**Probe Categories:**
- **Truthfulness** — Factual accuracy, honesty
- **Safety** — Harmful content detection
- **Alignment** — Human value alignment
- **Sovereignty** — User autonomy and control
- **Ethical** — Ethical compliance

#### 4. Standard Suite

**Type:** Industry-standard benchmarks  
**LLM Required:** ✅ Yes (for some benchmarks)  
**Speed:** Slow  

| Benchmark | Purpose | Metric |
|-----------|---------|--------|
| **TruthfulQA** | Truthfulness | Accuracy (0-1) |
| **DoNotAnswer** | Refusal to answer | Compliance (0-1) |
| **Toxigen** | Toxicity detection | Safety score (0-1) |
| **GSM8K** | Grade-school math | Accuracy (0-1) |

```go
// Run specific benchmark
benchmarkScore, err := engine.ScoreBenchmark(ctx, response, "gsm8k")

// Run all standard benchmarks
allScores := engine.ScoreAllBenchmarks(ctx, response)
```

#### 5. Exact Suite

**Type:** Exact match comparison  
**LLM Required:** ❌ No  
**Speed:** Fast  

```go
// Compare against expected answer
exactScore := engine.ScoreExact(response, expected, ml.ExactOptions{
    CaseSensitive: false,
    TrimWhitespace: true,
})
```

**Use Cases:**
- Multiple choice questions
- Exact answer verification
- Simple Q&A evaluation
- Deterministic testing

### Score result

```go
type ScoreResult struct {
    Suite     string
    Score     float64       // Normalized score (0-1)
    MaxScore  float64       // Maximum possible score
    Details   map[string]any // Suite-specific details
    Duration  time.Duration // Time taken
}

type OverallScore struct {
    Total     float64
    BySuite   map[string]float64
    Duration  time.Duration
    Results   []ScoreResult
}
```

---

## Capability probes (23 total)

Binary pass/fail tests across AI capabilities. Each probe returns:

```go
type ProbeResult struct {
    ProbeName string
    Passed    bool
    Score     float64  // 0 or 1 (pass/fail)
    Details   string   // Additional information
    Duration  time.Duration
}
```

### Math & logic probes (5)

| # | Probe | Description |
|---|-------|-------------|
| 1 | `arithmetic` | Basic arithmetic operations |
| 2 | `algebra` | Algebraic equation solving |
| 3 | `calculus` | Calculus (derivatives, integrals) |
| 4 | `logic` | Logical reasoning and puzzles |
| 5 | `puzzles` | Problem-solving challenges |

**Example:**
```go
// Test: What is 2 + 2?
// Expected: 4
result := ml.RunProbe(ctx, backend, "arithmetic")
// Passes if answer is correct
```

### Code generation probes (5)

| # | Probe | Description |
|---|-------|-------------|
| 6 | `syntax` | Code syntax correctness |
| 7 | `function` | Function implementation |
| 8 | `algorithm` | Algorithm generation |
| 9 | `debug` | Bug fixing in code |
| 10 | `optimize` | Code optimization |

**Example:**
```go
// Test: Write a function that reverses a string
// Expected: Correct implementation
result := ml.RunProbe(ctx, backend, "function")
```

### Reasoning probes (5)

| # | Probe | Description |
|---|-------|-------------|
| 11 | `chain_of_thought` | Multi-step reasoning |
| 12 | `common_sense` | Everyday knowledge |
| 13 | `analogy` | Analytical reasoning |
| 14 | `deduction` | Logical deduction |
| 15 | `induction` | Generalization from examples |

**Example:**
```go
// Test: If all Bloops are Razzies and all Razzies are Lazzies, are all Bloops Lazzies?
// Expected: Yes
result := ml.RunProbe(ctx, backend, "deduction")
```

### Language & knowledge probes (5)

| # | Probe | Description |
|---|-------|-------------|
| 16 | `vocabulary` | Word knowledge and usage |
| 17 | `grammar` | Grammar and syntax |
| 18 | `comprehension` | Text understanding |
| 19 | `memory` | Context retention |
| 20 | `knowledge` | Factual accuracy |

**Example:**
```go
// Test: What is the capital of France?
// Expected: Paris
result := ml.RunProbe(ctx, backend, "knowledge")
```

### Specialized probes (3)

| # | Probe | Description |
|---|-------|-------------|
| 21 | `creativity` | Creative generation |
| 22 | `translation` | Language translation |
| 23 | `summarization` | Text summarization |

### Running probes

```go
// Run all 23 probes
results := ml.RunAllProbes(ctx, backend, "model-name")

// Run specific probes
results := ml.RunProbes(ctx, backend, []string{"arithmetic", "logic", "code"})

// Run single probe
result := ml.RunProbe(ctx, backend, "arithmetic")

// Custom probe with prompt
result := ml.RunCustomProbe(ctx, backend, ml.Probe{
    Name:    "custom-test",
    Prompt:  "Custom prompt here",
    Expected: []string{"expected1", "expected2"},
    Type:    ml.ProbeTypeMultipleChoice,
})
```

---

## GGUF support

### Overview

GGUF (Geraldo's Generic Unified Format) is a binary format for storing machine learning models. go-ml provides complete GGUF support including:

- **Parsing** — Read GGUF files and extract metadata
- **Conversion** — Convert between GGUF and other formats (Safetensors)
- **Inventory** — Manage collections of GGUF models
- **Validation** — Verify GGUF file integrity

### GGUF model

```go
// Parse a GGUF file
model, err := gguf.ParseFile("/path/to/model.gguf")
if err != nil {
    panic(err)
}

// Access metadata
fmt.Printf("Model: %s\n", model.Metadata["general.name"])
fmt.Printf("Architecture: %s\n", model.Metadata["general.architecture"])
fmt.Printf("Parameters: %d\n", model.ParameterCount())
fmt.Printf("Size: %d bytes\n", model.Size)

// Access tensors
tensors := model.Tensors()
for _, tensor := range tensors {
    fmt.Printf("Tensor: %s, Shape: %v, Type: %s\n", 
        tensor.Name, tensor.Shape, tensor.Type)
}
```

### GGUF metadata

Standard metadata fields:

```go
// Model identification
metadata["general.name"]           // Model name (e.g., "gemma-3-1b")
metadata["general.architecture"]    // Architecture (e.g., "llama", "gemma", "qwen")
metadata["general.author"]          // Author
metadata["general.version"]        // Version
metadata["general.license"]         // License (e.g., "apache-2.0", "mit")
metadata["general.description"]    // Description
metadata["general.url"]             // URL

// Model parameters
metadata["llama.block_count"]       // Number of transformer blocks
metadata["llama.embedding_length"]   // Embedding dimension
metadata["llama.feed_forward_length"] // Feed-forward dimension
metadata["llama.attention.head_count"] // Number of attention heads
metadata["llama.attention.head_count_kv"] // KV head count
metadata["llama.rope_frequency_base"] // RoPE frequency base
metadata["llama.rope_frequency_scale"] // RoPE frequency scale

// Training parameters
metadata["llama.pre_norm"]           // Use pre-normalization?
metadata["llama.post_norm"]          // Use post-normalization?
metadata["llama.scaling_type"]       // Scaling type

// Tokenizer
metadata["tokenizer.ggml.tokenizer"] // Tokenizer type
metadata["tokenizer.ggml.pre"]      // Pre-tokenization string
metadata["tokenizer.ggml.special_tokens"] // Special tokens
```

### GGUF conversion

```go
// Convert Safetensors to GGUF
err := gguf.ConvertSafetensorsToGGUF(
    "/path/to/input.safetensors",
    "/path/to/output.gguf",
    gguf.ConvertOptions{
        ModelName: "converted-model",
        Architecture: "llama",
        Quantization: gguf.QuantQ4_0,
    },
)

// Convert GGUF to Safetensors
err := gguf.ConvertGGUFToSafetensors(
    "/path/to/input.gguf",
    "/path/to/output.safetensors",
)
```

### Quantization types

| Type | Description | Bits | Size Reduction |
|------|-------------|------|-----------------|
| `F32` | Full precision | 32 | 1x |
| `F16` | Half precision | 16 | 2x |
| `Q4_0` | 4-bit, no scaling | 4 | 8x |
| `Q4_1` | 4-bit, with scaling | 4 | 8x |
| `Q5_0` | 5-bit | 5 | 6.4x |
| `Q5_1` | 5-bit, with scaling | 5 | 6.4x |
| `Q8_0` | 8-bit | 8 | 4x |

### GGUF inventory

```go
// Create inventory
inventory := gguf.NewInventory("/path/to/models/")

// List all models
models := inventory.List()
for _, model := range models {
    fmt.Printf("Name: %s\n", model.Name)
    fmt.Printf("  Path: %s\n", model.Path)
    fmt.Printf("  Size: %d bytes\n", model.Size)
    fmt.Printf("  Modified: %v\n", model.Modified)
    fmt.Printf("  Architecture: %s\n", model.Architecture)
}

// Find specific model
model, err := inventory.Find("gemma-3-1b")

// Filter by architecture
llamaModels := inventory.FilterByArchitecture("llama")

// Get model info
info, err := inventory.GetInfo("gemma-3-1b")
```

---

## Agent orchestrator

### Overview

The agent orchestrator provides **remote evaluation capabilities** for fine-tuned models and checkpoints:

- **SSH-based** — Connect to remote machines for evaluation
- **Checkpoint Discovery** — Find and catalog model checkpoints
- **Remote Evaluation** — Run probes and scoring on remote models
- **Metrics Streaming** — Stream results to InfluxDB and DuckDB
- **Batch Processing** — Evaluate multiple checkpoints in batch

### Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Agent Orchestrator Architecture                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐   │
│  │  Local          │     │   Remote         │     │   Database      │   │
│  │  Orchestrator   │────►│   M3 Mac         │────►│   Layer         │   │
│  │                 │     │                 │     │                 │   │
│  │  - Config      │     │  - Checkpoints   │     │  - InfluxDB    │   │
│  │  - Coordination │     │  - Evaluation    │     │  - DuckDB      │   │
│  │  - Streaming    │     │  - Probing       │     │  - Metrics     │   │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘   │
│                    ▲                         ▲                         ▲      │
│                    │                         │                         │      │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐   │
│  │  Transport       │     │   SSH           │     │   Metrics       │   │
│  │  Interface       │     │  Connection      │     │   Streaming     │   │
│  │                  │     │                  │     │   Interface     │   │
│  │  RemoteTransport  │────►│  SSHClient       │     │  MetricStreamer │   │
│  │  - SSH           │     │  - Connect      │     │  - Write       │   │
│  │  - Local         │     │  - Execute      │     │  - Batch       │   │
│  │  - Mock          │     │  - Disconnect   │     │                 │   │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Checkpoint discovery

```go
// Discover checkpoints on remote server
checkpoints, err := agent.DiscoverCheckpoints(
    ctx,
    agent.RemoteConfig{
        Host: "user@m3-mac",
        Port: 22,
        Path: "/path/to/checkpoints/",
        KeyPath: "~/.ssh/id_rsa",
    },
)

for _, cp := range checkpoints {
    fmt.Printf("Checkpoint: %s\n", cp.Name)
    fmt.Printf("  Path: %s\n", cp.Path)
    fmt.Printf("  Step: %d\n", cp.Step)
    fmt.Printf("  Time: %v\n", cp.Timestamp)
    fmt.Printf("  Size: %d bytes\n", cp.Size)
    fmt.Printf("  Metrics: %v\n", cp.Metrics)
}
```

### Remote evaluation

```go
// Evaluate a checkpoint on remote server
result, err := agent.EvaluateCheckpoint(
    ctx,
    agent.EvaluationRequest{
        Remote: agent.RemoteConfig{
            Host: "user@m3-mac",
            Path: "/path/to/checkpoint/",
        },
        Config: ml.EvaluationConfig{
            Probes: []string{"arithmetic", "logic", "code", "reasoning"},
            Suites: []string{"heuristic", "semantic"},
            MaxTokens: 512,
            Temperature: 0.7,
        },
    },
)

// Access results
for _, probeResult := range result.ProbeResults {
    fmt.Printf("%s: %v (%.2f)\n", 
        probeResult.ProbeName, probeResult.Passed, probeResult.Score)
}

for _, suiteResult := range result.SuiteResults {
    fmt.Printf("%s: %.2f\n", suiteResult.Suite, suiteResult.Score)
}
```

### Batch evaluation

```go
// Evaluate multiple checkpoints
results, err := agent.EvaluateCheckpoints(
    ctx,
    agent.BatchEvaluationRequest{
        Remote: agent.RemoteConfig{Host: "user@m3-mac"},
        Checkpoints: []string{
            "/path/to/cp1/",
            "/path/to/cp2/",
            "/path/to/cp3/",
        },
        Config: ml.EvaluationConfig{
            Probes: []string{"all"},
            Suites: []string{"all"},
        },
    },
)
```

### Metrics streaming

#### InfluxDB Streaming

```go
// Create InfluxDB streamer
streamer := agent.NewInfluxDBStreamer(
    "http://localhost:8086",
    "api-token",
    "org-name",
    "bucket-name",
)

// Stream evaluation results
streamer.Stream(ctx, result)

// Stream with tags
streamer.StreamWithTags(ctx, result, map[string]string{
    "model": "gemma-3-1b",
    "version": "v1.0",
    "type": "checkpoint",
})
```

#### DuckDB Streaming

```go
// Create DuckDB streamer
duckStreamer := agent.NewDuckDBStreamer("/path/to/db.duckdb")

// Stream results
duckStreamer.Stream(ctx, result)

// With table name
duckStreamer.StreamToTable(ctx, result, "evaluation_results")
```

---

## CLI commands

### Overview

The `lem` CLI provides a command-line interface for go-ml:

```bash
lem [command] [subcommand] [flags]
```

### Command hierarchy

```
lem/
├── bench/              # Benchmarking
│   ├── generate        # Text generation benchmarks
│   ├── chat           # Chat benchmarks
│   └── embed          # Embedding benchmarks
├── gguf/              # GGUF management
│   ├── info           # Show GGUF file info
│   ├── list           # List GGUF models
│   ├── convert        # Convert between formats
│   └── validate       # Validate GGUF files
├── judge/             # Scoring
│   ├── score          # Score responses
│   ├── compare        # Compare multiple responses
│   └── batch          # Batch scoring
├── probe/             # Capability probes
│   ├── run            # Run specific probes
│   ├── all            # Run all probes
│   └── list           # List available probes
├── agent/             # Agent orchestrator
│   ├── discover       # Discover checkpoints
│   ├── evaluate       # Evaluate checkpoint
│   ├── batch          # Batch evaluation
│   └── stream         # Stream to databases
└── serve/             # REST API
    └── start          # Start API server
```

### Command examples

#### Benchmarking

```bash
# Generate text benchmark
lem bench generate --backend http --url http://localhost:11434 --model qwen3:8b \
  --prompt "Explain machine learning" --tokens 512 --iterations 10

# Chat benchmark
lem bench chat --backend http --url http://localhost:11434 --model qwen3:8b \
  --iterations 100 --max-tokens 256

# Embedding benchmark
lem bench embed --backend http --url http://localhost:11434 --model qwen3:8b \
  --texts "text1,text2,text3" --iterations 50
```

#### GGUF Management

```bash
# Show GGUF file info
lem gguf info /path/to/model.gguf

# List all GGUF models in directory
lem gguf list /path/to/models/

# Convert Safetensors to GGUF
lem gguf convert /path/to/model.safetensors /path/to/model.gguf \
  --name "my-model" --arch llama --quant q4_0

# Validate GGUF file
lem gguf validate /path/to/model.gguf
```

#### Scoring

```bash
# Score responses
lem judge score --backend http --url http://localhost:11434 \
  --responses "response1,response2" --suites semantic,content

# Score from file
lem judge score --backend http --url http://localhost:11434 \
  --responses-file responses.json --suites all

# Compare multiple responses
lem judge compare --backend http --url http://localhost:11434 \
  --responses responses.json --output results.json
```

#### Probes

```bash
# Run specific probe
lem probe run --backend http --url http://localhost:11434 --model qwen3:8b \
  --probe arithmetic

# Run all probes
lem probe all --backend http --url http://localhost:11434 --model qwen3:8b \
  --output results.json

# Run multiple probes
lem probe run --backend http --url http://localhost:11434 \
  --probes arithmetic,logic,code

# List available probes
lem probe list
```

#### Agent

```bash
# Discover checkpoints
lem agent discover --remote user@m3-mac --path /path/to/checkpoints/

# Evaluate checkpoint
lem agent evaluate --remote user@m3-mac --checkpoint /path/to/cp/ \
  --probes arithmetic,logic,code

# Batch evaluation
lem agent batch --remote user@m3-mac \
  --checkpoints "/cp1/,/cp2/,/cp3/" --probes all

# Stream to InfluxDB
lem agent evaluate --remote user@m3-mac --checkpoint /path/to/cp/ \
  --stream-influx --influx-url http://localhost:8086 --influx-token token
```

#### Server

```bash
# Start REST API server
lem serve start --port 8080 --backend http --url http://localhost:11434

# With TLS
lem serve start --port 443 --tls-cert cert.pem --tls-key key.pem \
  --backend http --url http://localhost:11434

# With all backends
lem serve start --port 8080 --backends http,llama
```

---

## REST API

### Overview

The REST API provides HTTP access to all go-ml functionality:

```
Base URL: /v1/ml/
```

### Endpoints

#### Backend Management

| Method | Endpoint | Description | Request | Response |
|--------|----------|-------------|---------|----------|
| GET | `/backends` | List all backends | - | `BackendList` |
| GET | `/backends/{name}` | Get backend info | - | `BackendInfo` |
| POST | `/backends/{name}/register` | Register backend | `BackendConfig` | `BackendInfo` |
| DELETE | `/backends/{name}` | Unregister backend | - | - |

#### Inference

| Method | Endpoint | Description | Request | Response |
|--------|----------|-------------|---------|----------|
| POST | `/backends/{name}/generate` | Generate text | `GenerateRequest` | `GenerateResponse` |
| POST | `/backends/{name}/chat` | Chat completion | `ChatRequest` | `ChatResponse` |
| POST | `/backends/{name}/classify` | Classify text | `ClassifyRequest` | `ClassifyResponse` |
| POST | `/backends/{name}/embed` | Generate embeddings | `EmbedRequest` | `EmbedResponse` |

#### Scoring

| Method | Endpoint | Description | Request | Response |
|--------|----------|-------------|---------|----------|
| POST | `/score` | Score responses | `ScoreRequest` | `ScoreResponse` |
| POST | `/score/{suite}` | Score with specific suite | `ScoreRequest` | `ScoreResponse` |
| GET | `/suites` | List scoring suites | - | `SuiteList` |

#### Probes

| Method | Endpoint | Description | Request | Response |
|--------|----------|-------------|---------|----------|
| POST | `/probe` | Run probes | `ProbeRequest` | `ProbeResponse` |
| POST | `/probe/{name}` | Run specific probe | `ProbeRequest` | `ProbeResponse` |
| GET | `/probes` | List available probes | - | `ProbeList` |
| POST | `/probes/all` | Run all probes | `ProbeAllRequest` | `ProbeAllResponse` |

#### GGUF

| Method | Endpoint | Description | Request | Response |
|--------|----------|-------------|---------|----------|
| POST | `/gguf/info` | Get GGUF info | `GGUFRequest` | `GGUFInfo` |
| POST | `/gguf/convert` | Convert format | `ConvertRequest` | `ConvertResponse` |
| POST | `/gguf/validate` | Validate GGUF | `GGUFRequest` | `ValidateResponse` |
| GET | `/gguf/models` | List GGUF models | - | `GGUFModelList` |

#### Health & Status

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| GET | `/status` | System status |
| GET | `/version` | Version info |

### API examples

#### Generate Text

```bash
curl -X POST http://localhost:8080/v1/ml/backends/ollama/generate \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Explain the concept of machine learning",
    "max_tokens": 512,
    "temperature": 0.7
  }'

# Response
{
  "text": "Machine learning is...",
  "tokens": [{"text": "Machine", "id": 100, "logprob": -0.5}, ...],
  "metrics": {"generate_time": 0.123},
  "duration": "123ms"
}
```

#### Chat

```bash
curl -X POST http://localhost:8080/v1/ml/backends/ollama/chat \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "What is AI?"}
    ],
    "max_tokens": 256
  }'
```

#### Score

```bash
curl -X POST http://localhost:8080/v1/ml/score \
  -H "Content-Type: application/json" \
  -d '{
    "responses": ["The answer is 42.", "I don't know."],
    "suites": ["heuristic", "semantic", "exact"]
  }'
```

#### Run Probes

```bash
curl -X POST http://localhost:8080/v1/ml/probe \
  -H "Content-Type: application/json" \
  -d '{
    "backend": "ollama",
    "model": "qwen3:8b",
    "probes": ["arithmetic", "logic", "code"]
  }'
```

---

## Usage examples

### Basic inference

```go
package main

import (
    "context"
    "fmt"
    "dappco.re/go/core/ml"
)

func main() {
    ctx := context.Background()
    
    // Create HTTP backend for Ollama
    backend := ml.NewHTTPBackend("http://localhost:11434", "gemma4-1b")
    
    // Generate text
    resp, err := backend.Generate(ctx, "Explain machine learning", ml.Options{
        MaxTokens:   512,
        Temperature: 0.7,
        TopP:       0.9,
    })
    if err != nil {
        panic(err)
    }
    
    fmt.Println("Response:", resp.Text)
    fmt.Println("Tokens:", len(resp.Tokens))
    fmt.Println("Duration:", resp.Duration)
    fmt.Println("Metrics:", resp.Metrics)
}
```

### Chat conversation

```go
messages := []ml.Message{
    {Role: "system", Content: "You are a helpful coding assistant."},
    {Role: "user", Content: "Write a Python function to reverse a string."},
}

resp, err := backend.Chat(ctx, messages, ml.Options{
    MaxTokens: 256,
})
fmt.Println(resp.Text)
```

### Classification

```go
result, err := backend.Classify(ctx, "Machine learning is amazing!", 
    []string{"positive", "negative", "neutral"})
fmt.Printf("Class: %s, Score: %f\n", result.Class, result.Score)
```

### Embeddings

```go
embedding, err := backend.Embed(ctx, "Hello, world!")
fmt.Printf("Embedding dimension: %d\n", len(embedding))
```

### Scoring

```go
// Create scoring engine
engine := ml.NewScoringEngine(backend)

// Define responses to score
responses := []string{
    "The capital of France is Paris.",
    "I believe the capital of France is Paris.",
    "France's capital city is Paris.",
}

// Score all responses
scores := engine.ScoreAll(ctx, responses)
for i, score := range scores {
    fmt.Printf("Response %d: %.2f\n", i, score.Total)
    for suite, suiteScore := range score.BySuite {
        fmt.Printf("  %s: %.2f\n", suite, suiteScore)
    }
}
```

### Custom heuristic scoring

```go
// Define custom heuristic rules
rules := []ml.HeuristicRule{
    {
        Name:    "contains-answer",
        Pattern: `(?i)paris|capital`,
        Score:   1.0,
        Weight:  1.0,
    },
}

// Create custom heuristic scorer
scorer := ml.NewHeuristicScorer(rules)

// Score response
score := scorer.Score("The capital of France is Paris.")
fmt.Printf("Score: %.2f\n", score)
```

### Run capability probes

```go
// Run specific probes
results := ml.RunProbes(ctx, backend, []string{"arithmetic", "logic", "function"})

for _, result := range results {
    fmt.Printf("%s: %v (score: %.2f)\n", 
        result.ProbeName, result.Passed, result.Score)
}

// Get overall pass rate
passRate := ml.CalculatePassRate(results)
fmt.Printf("Pass rate: %.1f%%\n", passRate*100)
```

### GGUF operations

```go
// Parse and inspect GGUF
model, err := gguf.ParseFile("/path/to/model.gguf")
fmt.Printf("Model: %s\n", model.Metadata["general.name"])
fmt.Printf("Architecture: %s\n", model.Metadata["general.architecture"])

// Convert GGUF to Safetensors
err = gguf.ConvertGGUFToSafetensors(
    "/path/to/input.gguf",
    "/path/to/output.safetensors",
)

// Manage inventory
inventory := gguf.NewInventory("/path/to/models/")
models := inventory.List()
for _, m := range models {
    fmt.Printf("Found: %s (%s)\n", m.Name, m.Architecture)
}
```

---

## Testing

### Test structure

Following the AX Standard with Good/Bad/Ugly test triplets:

```
Backend Tests:
├── backend_http_test.go     # HTTPBackend tests
│   ├── TestHTTPBackend_Good_Generate
│   ├── TestHTTPBackend_Bad_InvalidURL
│   └── TestHTTPBackend_Ugly_Timeout
├── backend_llama_test.go    # LlamaBackend tests
│   ├── TestLlamaBackend_Good_Generate
│   ├── TestLlamaBackend_Bad_InvalidPath
│   └── TestLlamaBackend_Ugly_KilledProcess
└── adapter_test.go          # Adapter tests
    ├── TestInferenceAdapter_Good_Generate
    ├── TestInferenceAdapter_Bad_NilModel
    └── TestInferenceAdapter_Ugly_BrokenToken

Scoring Tests:
├── score_test.go            # ScoringEngine tests
├── judge_test.go            # Judge tests
└── heuristic_test.go        # Heuristic tests

Probe Tests:
├── probes_test.go           # All probe tests
│   ├── TestProbe_Arithmetic_Good
│   ├── TestProbe_Arithmetic_Bad
│   └── TestProbe_Arithmetic_Ugly
└── probes_*.go              # Individual probe implementations

GGUF Tests:
├── gguf/gguf_test.go        # GGUF parsing tests
├── gguf/parser_test.go      # Parser tests
└── gguf/convert_test.go     # Conversion tests

Agent Tests:
├── agent/orchestrator_test.go
├── agent/ssh_test.go
└── agent/stream_test.go
```

### Running tests

```bash
# All tests
go test ./...

# Specific test
go test -run TestHTTPBackend_Good_Generate

# With race detector
go test -race ./...

# Verbose output
go test -v ./...

# Benchmarks
go test -bench=. ./...

# Specific package
go test ./go

# With coverage
go test -cover ./... -coverprofile=coverage.out

# With short mode (skip long-running tests)
go test -short ./...
```

---

## Implementation notes

### Dual interface pattern

The package maintains both interface families for backward compatibility and flexibility:

- **`ml.Backend`** — Result-based, mature, stable
- **`inference.TextModel`** — Token-streaming, newer, evolving

**Migration Strategy:**
1. New GPU backends implement `inference.TextModel`
2. Use `InferenceAdapter` to bridge to `ml.Backend`
3. Gradually migrate consumers to `inference.TextModel`
4. Maintain both interfaces indefinitely for compatibility

### Adapter design

Adapters follow the **adapter pattern** to bridge between incompatible interfaces:

```go
// InferenceAdapter: inference.TextModel → ml.Backend
type InferenceAdapter struct {
    model inference.TextModel
}

func (a *InferenceAdapter) Generate(ctx context.Context, prompt string, opts Options) (*Result, error) {
    // Collect all tokens from the stream
    var tokens []ml.Token
    for tok := range a.model.Generate(ctx, prompt, opts.ToInference()...) {
        tokens = append(tokens, ml.Token{Text: tok.Text})
    }
    return &Result{Text: strings.Join(tokens, ""), Tokens: tokens}, nil
}

// HTTPTextModel: ml.HTTPBackend → inference.TextModel
type HTTPTextModel struct {
    backend *ml.HTTPBackend
}

func (m *HTTPTextModel) Generate(ctx context.Context, prompt string, opts ...inference.GenerateOption) iter.Seq[inference.Token] {
    // Call backend, stream individual tokens
    return func(yield func(inference.Token) bool) {
        resp, _ := m.backend.Generate(ctx, prompt, ml.OptionsFromInference(opts...))
        for _, tok := range resp.Tokens {
            if !yield(inference.Token{Text: tok.Text}) {
                return
            }
        }
    }
}
```

---

## Related packages

### Backend dependencies

| Package | Module | Relationship |
|---------|--------|--------------|
| [go-inference](../inference/) | `dappco.re/go/inference` | Shared interfaces, TextModel/Backend contracts |
| [go-mlx](../mlx/) | `dappco.re/go/mlx` | Apple Metal GPU backend |
| [go-rocm](../rocm/) | `dappco.re/go/rocm` | AMD ROCm GPU backend |

### Consumer packages

| Package | Module | Relationship |
|---------|--------|--------------|
| [go-ai](../ai/) | `dappco.re/go/ai` | Uses ml.Backend for AI integration |
| [go-i18n](../i18n/) | `dappco.re/go/i18n` | Uses ml.Backend for classification |

### Core framework

| Package | Relationship |
|---------|--------------|
| [CoreGO INDEX](../../INDEX.md) | Complete catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## References

1. **Repository** — [~/Code/core/go-ml/](file:///Users/snider/Code/core/go-ml/)
2. **CLAUDE.md** — [~/Code/core/go-ml/CLAUDE.md](file:///Users/snider/Code/core/go-ml/CLAUDE.md)
3. **AGENTS.md** — [~/Code/core/go-ml/AGENTS.md](file:///Users/snider/Code/core/go-ml/AGENTS.md)
4. **GOAL.md** — [~/Code/core/go-ml/GOAL.md](file:///Users/snider/Code/core/go-ml/GOAL.md)
5. **RFC** — [plans/code/core/go/ml/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/ml/RFC.md)
6. **Architecture** — [~/Code/core/go-ml/docs/architecture.md](file:///Users/snider/Code/core/go-ml/docs/architecture.md)
7. **Development Guide** — [~/Code/core/go-ml/docs/development.md](file:///Users/snider/Code/core/go-ml/docs/development.md)
8. **Models Guide** — [~/Code/core/go-ml/docs/models.md](file:///Users/snider/Code/core/go-ml/docs/models.md)
9. **Scoring Docs** — [~/Code/core/go-ml/docs/scoring.md](file:///Users/snider/Code/core/go-ml/docs/scoring.md)
10. **CoreGO INDEX** — [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md)

---

*Package index generated: 2026-06-18T00:00:00Z*
*Knowledge Pack: CoreGo v1.2.0*
*Module: dappco.re/go/core/ml*
*Maintainer: Purberus <purberus@lthn.ai>*
