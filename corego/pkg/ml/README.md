---
type: Package Documentation
package: ml
module: dappco.re/go/core/ml
repo: core/go-ml
lang: go
tags:
  - ai
  - ml
  - machine-learning
  - inference-backend
  - scoring-engine
  - agent-orchestrator
  - gguf
  - llm
  - judge
  - benchmark
---
# go-ml — ML Inference Backends & Scoring Engine

> **ML inference backends, multi-suite scoring engine, and agent orchestrator for the Lethean AI stack**

**Module:** `dappco.re/go/core/ml`
**Repository:** [~/Code/core/go-ml/](file:///Users/snider/Code/core/go-ml/)
**Status:** ✅ Production-Ready
**License:** EUPL-1.2
**Language:** Go 1.25+
**Dependencies:** Core framework, go-inference, testify
**RFC:** [plans/code/core/go/ml/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/ml/RFC.md)
**CLAUDE.md:** [~/Code/core/go-ml/CLAUDE.md](file:///Users/snider/Code/core/go-ml/CLAUDE.md)
**AGENTS.md:** [~/Code/core/go-ml/AGENTS.md](file:///Users/snider/Code/core/go-ml/AGENTS.md)
**GOAL.md:** [~/Code/core/go-ml/GOAL.md](file:///Users/snider/Code/core/go-ml/GOAL.md)
**Maintainer:** Purberus <purberus@lthn.ai>
**Last Updated:** 2026-06-17

---

## 🎯 Overview

`go-ml` is the **ML inference backend and scoring engine** for the Lethean AI stack. It provides:

1. **Pluggable Inference Backends** — Multiple backend types for different use cases
2. **Multi-Suite Scoring Engine** — Concurrent evaluation across multiple scoring suites
3. **23 Capability Probes** — Binary pass/fail tests across various AI capabilities
4. **GGUF Model Management** — Format parsing, conversion, and inventory
5. **Agent Orchestrator** — SSH-based checkpoint discovery and evaluation
6. **CLI & REST API** — Full command-line and HTTP interface

### Backend Types

| Backend | Type | Platform | Use Case |
|---------|------|----------|----------|
| `HTTPBackend` | HTTP | Cross-platform | Ollama, LM Studio, OpenAI-compatible APIs |
| `LlamaBackend` | Subprocess | Cross-platform | Managed llama-server processes |
| `InferenceAdapter` | Bridge | Cross-platform | Wraps go-inference TextModel backends |

### Scoring Engine

The scoring engine evaluates model responses across **5 different suites**:

| Suite | Purpose | LLM Required |
|-------|---------|---------------|
| **Heuristic** | Regex-based pattern matching | ❌ No |
| **Semantic** | LLM-based semantic evaluation | ✅ Yes |
| **Content** | Sovereignty and alignment probes | ✅ Yes |
| **Standard** | Benchmark tests (TruthfulQA, DoNotAnswer, Toxigen, GSM8K) | ✅ Yes |
| **Exact** | Exact match comparison | ❌ No |

---

## 🏗️ Architecture

### Dual-Interface Backend System

`go-ml` maintains **two interface families** that coexist and are connected by adapters:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Dual Interface System                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────┐      ┌─────────────────────────┐              │
│  │   ml.Backend             │      │   inference.TextModel    │              │
│  │  (Primary interface)     │◄─────►│   (go-inference)          │              │
│  │                         │      │   (Token-level control)   │              │
│  │  - Generate() Result      │      │   - Generate() iter.Seq   │              │
│  │  - Chat() Result         │      │   - Chat() iter.Seq      │              │
│  │  - Classify()             │      │   - Close()              │              │
│  │  - Embed()               │      │   - Info()               │              │
│  └─────────────────────────┘      └─────────────────────────┘              │
│                              ▲     ▲                                   │
│                              │     │                                   │
│                    ┌─────────┴─────┴─────────┐                         │
│                    │         ADAPTERS          │                         │
│                    │  InferenceAdapter        │                         │
│                    │  HTTPTextModel           │                         │
│                    │  LlamaTextModel          │                         │
│                    └─────────────────────────┘                         │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Backend Implementations                                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         │
│  │  HTTPBackend    │  │  LlamaBackend    │  │ InferenceAdapter │         │
│  │                │  │                 │  │                 │         │
│  │ OpenAI-compat  │  │ Subprocess      │  │ go-inference    │         │
│  │ Ollama         │  │ llama-server    │  │ bridge          │         │
│  │ LM Studio      │  │ Managed         │  │ Metal/ROCm/...   │         │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘         │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Scoring Engine                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │  Engine.ScoreAll()                                                │ │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐     │ │
│  │  │ Heur.   │ │ Semantic│ │ Content │ │ Standard│ │ Exact   │     │ │
│  │  │ Regex   │ │ LLM Judge││Probes  │ │Benchmarks│ │ Match   │     │ │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘     │ │
│  │  Worker Pool (Concurrency: configurable)                             │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Agent Orchestrator                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │  SSH Checkpoint Discovery      InfluxDB Streaming    DuckDB          │ │
│  │  RemoteTransport interface ────►  Metrics ────►  Analytics          │ │
│  │  M3 Mac remote execution                                           │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Service Layer                                             │
│  Core.ServiceRuntime[Options]                                               │
│  Lifecycle: OnStartup → Register backends, Initialize judge/engine          │
│  Dynamic backend registration via RegisterBackend()                         │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Adapter Map

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ADAPTER CONNECTIONS                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  inference.TextModel ──► InferenceAdapter ──► ml.Backend                       │
│         (token streaming)        (bridging)         (result-based)             │
│                                                                              │
│  ml.HTTPBackend ──► HTTPTextModel ──► inference.TextModel                     │
│        (Result)           (adapting)           (streaming)                    │
│                                                                              │
│  ml.LlamaBackend ──► LlamaTextModel ──► inference.TextModel                    │
│       (Result)            (adapting)           (streaming)                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

`InferenceAdapter` bridges GPU backends (`inference.TextModel`) into `ml.Backend`.
Reverse adapters (`HTTPTextModel`, `LlamaTextModel`) allow HTTP/Llama backends to be used where `inference.TextModel` is expected.

---

## 📦 Repository Structure

```
go/
├── inference.go           # ml.Backend interface + Result types
├── backend.go            # Backend base + common functionality
├── backend_http.go       # HTTPBackend implementation
├── backend_llama.go       # LlamaBackend (subprocess llama-server)
├── adapter.go            # InferenceAdapter - bridges inference.TextModel to ml.Backend
├── backend_http_textmodel.go  # HTTPTextModel adapter
├── backend_llama_textmodel.go # LlamaTextModel adapter
│
├── score.go              # ScoringEngine + ScoreAll() + suites
├── judge.go              # LLM-based judge for semantic/content scoring
├── heuristic.go          # Heuristic scoring (regex-based)
├── probes.go             # 23 capability probes
├── probes_test.go        # Probe tests
│
├── gguf/                 # GGUF format handling
│   ├── gguf.go           # GGUF parsing and metadata
│   ├── parser.go         # GGUF file parser
│   ├── convert.go        # GGUF conversion utilities
│   └── inventory.go      # GGUF model inventory
│
├── agent/                # Agent orchestrator
│   ├── orchestrator.go   # Main orchestrator
│   ├── ssh.go            # SSH transport
│   ├── transport.go      # Transport interface
│   ├── checkpoint.go      # Checkpoint discovery
│   ├── eval.go           # Evaluation coordinator
│   └── stream.go         # InfluxDB/DuckDB streaming
│
├── service.go            # Core service integration
├── options.go            # Configuration options
├── contracts.go          # Interface compliance contracts
├── contracts_test.go     # Contract tests
│
├── cmd/                  # CLI commands
│   ├── cmd_bench/        # Benchmarking commands
│   ├── cmd_gguf/         # GGUF management commands
│   ├── cmd_judge/        # Judge/scoring commands
│   ├── cmd_probe/        # Capability probe commands
│   └── cmd_agent/        # Agent orchestrator commands
│
└── api/                  # REST API
    ├── routes.go         # API route definitions
    ├── handlers.go       # HTTP handlers
    └── middleware.go     # API middleware
```

---

## 🔌 Core Interfaces

### ml.Backend

The primary backend interface for go-ml:

```go
type Backend interface {
    // Name returns the backend identifier
    Name() string
    
    // Generate produces text from a prompt
    Generate(ctx context.Context, prompt string, opts Options) (*Result, error)
    
    // Chat generates a response in conversational context
    Chat(ctx context.Context, messages []Message, opts Options) (*Result, error)
    
    // Classify categorizes input text
    Classify(ctx context.Context, text string, categories []string) (*ClassifyResult, error)
    
    // Embed generates embeddings
    Embed(ctx context.Context, text string) ([]float32, error)
    
    // HealthCheck verifies backend is operational
    HealthCheck(ctx context.Context) error
}

type Result struct {
    Text     string
    Tokens   []Token
    Metrics  map[string]float64
    Error    error
    Duration time.Duration
}

type Message struct {
    Role    string // "system", "user", "assistant"
    Content string
}
```

### TextModel vs Backend

| Aspect | inference.TextModel | ml.Backend |
|--------|---------------------|------------|
| **Output** | `iter.Seq[Token]` | `*Result` |
| **Error Handling** | `token.Err()` after iterator | `Result.Error` |
| **Use Case** | Token-level control | Result-based simplicity |
| **Preferred** | New GPU backends | Existing HTTP/Llama |

### Adapter Pattern

```go
// Wrap inference.TextModel as ml.Backend
type InferenceAdapter struct {
    model inference.TextModel
}

func (a *InferenceAdapter) Generate(ctx context.Context, prompt string, opts Options) (*Result, error) {
    tokens := collectTokens(ctx, a.model.Generate(ctx, prompt, opts.ToInference()...))
    return &Result{
        Text:   string(tokens),
        Tokens: tokens,
    }, nil
}

// Wrap ml.Backend as inference.TextModel
type BackendTextModel struct {
    backend ml.Backend
}

func (m *BackendTextModel) Generate(ctx context.Context, prompt string, opts ...inference.GenerateOption) iter.Seq[inference.Token] {
    // Convert options, call backend, stream tokens
}
```

---

## 🎯 Backend Implementations

### HTTPBackend

OpenAI-compatible HTTP API backend:

```go
backend := ml.NewHTTPBackend("http://localhost:11434", "qwen3:8b")

// With options
backend := ml.NewHTTPBackend(
    "http://localhost:11434",
    "gemma4-1b",
    ml.WithHTTPTimeout(30*time.Second),
    ml.WithHTTPExtraHeaders(map[string]string{"Authorization": "Bearer ..."}),
)

// Usage
resp, err := backend.Generate(ctx, "Hello", ml.Options{
    MaxTokens: 256,
    Temperature: 0.7,
})
```

**Supported APIs:**
- Ollama (`http://localhost:11434`)
- LM Studio
- Any OpenAI-compatible endpoint
- Custom HTTP inference servers

### LlamaBackend

Managed llama-server subprocess backend:

```go
backend := ml.NewLlamaBackend(
    "/path/to/llama-server",
    "/path/to/model.gguf",
    ml.WithLlamaPort(8080),
    ml.WithLlamaContextLength(4096),
)

// Start the server
if err := backend.Start(ctx); err != nil {
    panic(err)
}
defer backend.Stop(ctx)

// Use the backend
resp, err := backend.Generate(ctx, "Hello")
```

**Features:**
- Automatic subprocess lifecycle management
- Health monitoring
- Dynamic port allocation
- GGUF model support

### InferenceAdapter

Bridges go-inference TextModel backends:

```go
// Assuming go-mlx is imported (registers "metal" backend)
model, _ := inference.LoadModel("/path/to/model/")

// Wrap as ml.Backend
adapter := ml.NewInferenceAdapter(model)

// Now can use ml.Backend interface
resp, err := adapter.Generate(ctx, "Hello")
```

**Backend Connection:**
- Automatically connects to all registered inference backends
- Respects platform-specific backend availability
- Falls back gracefully

---

## 📊 Scoring Engine

### Engine Architecture

The scoring engine runs **concurrently** across all configured suites:

```go
engine := ml.NewScoringEngine(
    ml.WithBackend(backend),
    ml.WithSuites("heuristic,semantic,content,standard,exact"),
    ml.WithConcurrency(4),
)

// Score multiple responses
scores := engine.ScoreAll(ctx, responses)

// Score a single response
score := engine.Score(ctx, response, "semantic")
```

### Scoring Suites

#### 1. Heuristic Suite (No LLM Required)

Regex-based pattern matching:

```go
// Define heuristic rules
rules := []ml.HeuristicRule{
    {
        Name:    "positive-sentiment",
        Pattern: `(?i)good|great|excellent|wonderful`,
        Score:   1.0,
    },
    {
        Name:    "negative-sentiment",
        Pattern: `(?i)bad|terrible|awful|poor`,
        Score:   -1.0,
    },
}

// Run heuristic scoring
score := engine.ScoreHeuristic(response, rules)
```

#### 2. Semantic Suite (LLM Judge)

LLM-based semantic evaluation:

```go
// Uses a judge model to evaluate responses
semanticScore := engine.ScoreSemantic(ctx, response, criteria)
```

**Criteria:**
- Relevance to prompt
- Helpfulness
- Accuracy
- Completeness

#### 3. Content Suite (Sovereignty Probes)

Alignment and safety probes:

```go
// Evaluate against content policies
contentScore := engine.ScoreContent(ctx, response, policies)
```

**Probe Categories:**
- Truthfulness
- Safety
- Alignment
- Sovereignty
- Ethical compliance

#### 4. Standard Suite (Benchmarks)

Industry-standard benchmarks:

| Benchmark | Purpose | Metric |
|-----------|---------|--------|
| TruthfulQA | Truthfulness | Accuracy |
| DoNotAnswer | Refusal | Compliance |
| Toxigen | Toxicity | Safety score |
| GSM8K | Math reasoning | Accuracy |

```go
// Run standard benchmarks
benchmarkScore := engine.ScoreBenchmark(ctx, response, "gsm8k")
```

#### 5. Exact Suite

Exact match comparison:

```go
exactScore := engine.ScoreExact(response, expected)
```

---

## 🎯 Capability Probes (23 Total)

Binary pass/fail tests across AI capabilities:

### Math & Logic
1. **Arithmetic** — Basic math operations
2. **Algebra** — Equation solving
3. **Calculus** — Derivatives and integrals
4. **Logic** — Logical reasoning
5. **Puzzles** — Problem-solving

### Code Generation
6. **Syntax** — Code syntax correctness
7. **Function** — Function implementation
8. **Algorithm** — Algorithm generation
9. **Debug** — Bug fixing
10. **Optimize** — Code optimization

### Reasoning
11. **Chain-of-Thought** — Multi-step reasoning
12. **Common Sense** — Everyday knowledge
13. **Analogy** — Analytical reasoning
14. **Deduction** — Logical deduction
15. **Induction** — Generalization

### Language & Knowledge
16. **Vocabulary** — Word knowledge
17. **Grammar** — Language rules
18. **Comprehension** — Text understanding
19. **Memory** — Context retention
20. **Knowledge** — Factual accuracy

### Specialized
21. **Creativity** — Creative generation
22. **Translation** — Language translation
23. **Summarization** — Text summarization

### Running Probes

```go
// Run all probes
results := ml.RunAllProbes(ctx, backend, "model-name")

// Run specific probe
result := ml.RunProbe(ctx, backend, "arithmetic")

// Custom probe
result := ml.RunCustomProbe(ctx, backend, customProbe)
```

---

## 📦 GGUF Support

### GGUF Model Management

```go
// Parse GGUF file
model, err := gguf.ParseFile("/path/to/model.gguf")

// Get metadata
fmt.Printf("Model: %s\n", model.Metadata["general.name"])
fmt.Printf("Architecture: %s\n", model.Metadata["general.architecture"])

// Convert between formats
err := gguf.ConvertSafetensorsToGGUF(input, output)

// Manage inventory
inventory := gguf.NewInventory("/path/to/models/")
models := inventory.List()
```

### GGUF Metadata

Standard GGUF metadata fields:

```go
// Model identification
metadata["general.name"]          // Model name
metadata["general.architecture"]   // e.g., "llama", "gemma", "qwen"
metadata["general.author"]         // Author
metadata["general.version"]       // Version
metadata["general.license"]        // License
metadata["general.description"]   // Description

// Model parameters
metadata["llama.block_count"]      // Number of blocks
metadata["llama.embedding_length"] // Embedding dimension
metadata["llama.feed_forward_length"] // Feed-forward dimension
metadata["llama.attention.head_count"] // Number of attention heads

// Training
metadata["llama.pre_norm"]        // Pre-normalization
metadata["llama.post_norm"]       // Post-normalization
```

---

## 🚀 Agent Orchestrator

SSH-based remote evaluation system:

### Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Agent Orchestrator                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐   │
│  │  Checkpoint      │     │   Evaluation     │     │   Streaming     │   │
│  │  Discovery       │────►│   Coordinator    │────►│   to DBs        │   │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘   │
│                    ▲                         │                         ▲      │
│                    │                         │                         │      │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐   │
│  │  Remote          │     │   Probe         │     │   InfluxDB      │   │
│  │  M3 Mac         │     │   Execution      │     │   Metrics       │   │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘   │
│                                  ▲                                    │
│                                  │                                    │
│                         ┌─────────────────┐                           │
│                         │   DuckDB        │                           │
│                         │   Analytics      │                           │
│                         └─────────────────┘                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Checkpoint Discovery

```go
// Discover checkpoints on remote server
checkpoints, err := agent.DiscoverCheckpoints(ctx, "user@m3-mac", "/path/to/checkpoints/")

for _, cp := range checkpoints {
    fmt.Printf("Checkpoint: %s\n", cp.Name)
    fmt.Printf("  Path: %s\n", cp.Path)
    fmt.Printf("  Step: %d\n", cp.Step)
    fmt.Printf("  Metrics: %v\n", cp.Metrics)
}
```

### Remote Evaluation

```go
// Evaluate checkpoint on remote
result, err := agent.EvaluateCheckpoint(ctx, 
    "user@m3-mac",
    "/path/to/checkpoint/",
    ml.EvaluationConfig{
        Probes: []string{"arithmetic", "logic", "code"},
        Suites: []string{"heuristic", "semantic"},
    },
)
```

### Streaming to Databases

```go
// Stream results to InfluxDB
streamer := agent.NewInfluxDBStreamer("http://localhost:8086", "token", "org", "bucket")

// Stream during evaluation
streamer.Stream(ctx, result)

// Stream to DuckDB
duckStreamer := agent.NewDuckDBStreamer("/path/to/db.duckdb")
duckStreamer.Stream(ctx, result)
```

---

## 🔌 CLI Commands

### Command Structure

```
lem [command] [subcommand] [options]
```

### Available Commands

| Command | Description |
|---------|-------------|
| `lem bench` | Run benchmarking tests |
| `lem gguf` | GGUF model management |
| `lem judge` | Run scoring engine |
| `lem probe` | Run capability probes |
| `lem agent` | Agent orchestrator |
| `lem serve` | Start REST API server |

### Command Examples

```bash
# Benchmark a model
lem bench generate --model gemma4-1b --tokens 512 --iterations 10

# Run probes
lem probe run --backend http --url http://localhost:11434 --model qwen3:8b

# Run all probes
lem probe all --backend http --url http://localhost:11434

# Judge responses
lem judge score --responses responses.json --suites semantic,content

# GGUF info
lem gguf info /path/to/model.gguf

# GGUF convert
lem gguf convert /path/to/model.safetensors /path/to/model.gguf

# Agent evaluation
lem agent evaluate --remote user@m3-mac --checkpoint /path/to/cp

# Start API server
lem serve --port 8080 --backend http --url http://localhost:11434
```

---

## 🌐 REST API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/v1/ml/backends` | List available backends |
| GET | `/v1/ml/backends/{name}` | Get backend info |
| POST | `/v1/ml/backends/{name}/generate` | Generate text |
| POST | `/v1/ml/backends/{name}/chat` | Chat completion |
| POST | `/v1/ml/score` | Score responses |
| POST | `/v1/ml/probe` | Run probes |
| GET | `/v1/ml/probes` | List available probes |
| GET | `/v1/ml/health` | Health check |

### API Example

```bash
# Generate text
curl -X POST http://localhost:8080/v1/ml/backends/ollama/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Hello", "max_tokens": 256}'

# Chat
curl -X POST http://localhost:8080/v1/ml/backends/ollama/chat \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Hello"}], "max_tokens": 256}'

# Score
curl -X POST http://localhost:8080/v1/ml/score \
  -H "Content-Type: application/json" \
  -d '{"responses": ["response 1"], "suites": ["heuristic", "semantic"]}'
```

---

## 🚀 Usage Examples

### Basic Inference

```go
package main

import (
    "context"
    "fmt"
    "dappco.re/go/core/ml"
)

func main() {
    ctx := context.Background()
    
    // Create HTTP backend
    backend := ml.NewHTTPBackend("http://localhost:11434", "gemma4-1b")
    
    // Generate text
    resp, err := backend.Generate(ctx, "Explain machine learning", ml.Options{
        MaxTokens:   512,
        Temperature: 0.7,
    })
    if err != nil {
        panic(err)
    }
    
    fmt.Println(resp.Text)
    fmt.Printf("Duration: %v\n", resp.Duration)
    fmt.Printf("Tokens: %d\n", len(resp.Tokens))
}
```

### Chat Conversation

```go
messages := []ml.Message{
    {Role: "system", Content: "You are a helpful assistant."},
    {Role: "user", Content: "What is the capital of France?"},
}

resp, err := backend.Chat(ctx, messages, ml.Options{
    MaxTokens: 256,
})
fmt.Println(resp.Text)
```

### Scoring

```go
// Create scoring engine
engine := ml.NewScoringEngine(backend)

// Score multiple responses
responses := []string{
    "The answer is 42.",
    "I don't know.",
    "Let me think about that...",
}

scores := engine.ScoreAll(ctx, responses)
for i, score := range scores {
    fmt.Printf("Response %d: score=%.2f\n", i, score.Total)
}
```

### Probes

```go
// Run capability probes
results := ml.RunAllProbes(ctx, backend, "gemma4-1b")

for _, result := range results {
    fmt.Printf("%s: %v\n", result.ProbeName, result.Passed)
}
```

### GGUF Management

```go
// Parse GGUF
model, err := gguf.ParseFile("model.gguf")
fmt.Printf("Architecture: %s\n", model.Architecture())

// List models in directory
inventory := gguf.NewInventory("/path/to/models/")
models := inventory.List()
for _, m := range models {
    fmt.Printf("Model: %s\n", m.Name)
}
```

---

## 🧪 Testing

### Test Structure

Following AX Standard with Good/Bad/Ugly test triplets:

```bash
# All tests
go test ./...

# Specific test
go test -run TestBackend_Good_Generate

# With race detector
go test -race ./...

# Benchmarks
go test -bench=. ./...
```

### Test Organization

```
go/
├── backend_http_test.go      # HTTPBackend tests
├── backend_llama_test.go     # LlamaBackend tests
├── adapter_test.go           # Adapter tests
├── score_test.go             # Scoring engine tests
├── judge_test.go             # Judge tests
├── probes_test.go            # Probe tests
├── gguf/                     # GGUF tests
│   ├── gguf_test.go
│   ├── parser_test.go
│   └── convert_test.go
└── agent/                    # Agent tests
    ├── orchestrator_test.go
    └── ssh_test.go
```

---

## 📈 Quality Metrics

- ✅ **Multiple Backend Types** — HTTP, Llama, Inference adapters
- ✅ **Comprehensive Scoring** — 5 scoring suites with 23 probes
- ✅ **GGUF Support** — Full GGUF format parsing and management
- ✅ **Agent Orchestration** — Remote evaluation with streaming
- ✅ **CLI & API** — Full command-line and REST interfaces
- ✅ **Well-Tested** — Good/Bad/Ugly pattern throughout
- ✅ **Core Integration** — Full Core framework support
- ✅ **Production Ready** — Deployed in Lethean AI stack

---

## 📝 Stability Notes

### Dual Interface Pattern

The package maintains both `ml.Backend` and `inference.TextModel` interfaces:

- **`ml.Backend`** — Result-based, preferred for existing HTTP/Llama code
- **`inference.TextModel`** — Token-streaming, preferred for new GPU backends

Adapters bridge between the two worlds seamlessly.

### Backward Compatibility

- Existing code using `ml.Backend` continues to work unchanged
- New code can use either interface
- Adapters ensure interoperability

---

## 🔗 Related Packages

### Backend Dependencies

| Package | Module | Relationship |
|---------|--------|--------------|
| [go-inference](../inference/) | `dappco.re/go/inference` | Shared interfaces, TextModel/Backend contracts |
| [go-mlx](../mlx/) | `dappco.re/go/mlx` | Apple Metal GPU backend (inference.TextModel) |
| [go-rocm](../rocm/) | `dappco.re/go/rocm` | AMD ROCm GPU backend (inference.TextModel) |

### Consumer Packages

| Package | Module | Relationship |
|---------|--------|--------------|
| [go-ai](../ai/) | `dappco.re/go/ai` | Uses ml.Backend for AI integration |
| [go-i18n](../i18n/) | `dappco.re/go/i18n` | Uses ml.Backend for classification |

### Core Framework

| Package | Relationship |
|---------|--------------|
| [CoreGO INDEX](../../INDEX.md) | Complete catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## 📚 References

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

*Package documentation generated: 2026-06-17T23:55:00Z*
*Knowledge Pack: CoreGo v1.2.0*
*Module: dappco.re/go/core/ml*
*Maintainer: Purberus <purberus@lthn.ai>*
