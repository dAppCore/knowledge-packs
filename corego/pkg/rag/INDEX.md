# go-rag Package Index

> **Package:** `dappco.re/go/rag`  
> **Repository:** [`github.com/dappcore/go-rag`](https://github.com/dappcore/go-rag)  
> **Spec:** [`plans/code/core/go/rag/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/rag/RFC.md)  
> **Documentation:** [README.md](./README.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>

---

## Overview

**go-rag** provides Retrieval-Augmented Generation capabilities for Go applications. It implements document chunking (3-level Markdown, sentence-based, paragraph-based), embedding generation via Ollama, vector storage and similarity search via Qdrant gRPC, TF-IDF keyword fallback, keyword boosting post-filtering, and flexible output formatting (plain text, XML, JSON) — all designed around decoupled `Embedder` and `VectorStore` interfaces for testability and flexibility.

### Key features

| Category | Count | Description |
|----------|-------|-------------|
| **Chunking strategies** | 3 | 3-level Markdown, sentence-based, paragraph-based |
| **Embedding backends** | 1 | Ollama (extensible via interface) |
| **Vector databases** | 1 | Qdrant gRPC (extensible via interface) |
| **Fallback mechanisms** | 2 | TF-IDF keyword search, keyword boosting |
| **Output formats** | 3 | Plain text, XML, JSON |
| **CLI commands** | 4 | ingest, query, collections list, collections stats |
| **Interfaces** | 2 | Embedder, VectorStore |

### Package metadata

| Metric | Value |
|--------|-------|
| **Module** | `dappco.re/go/rag` |
| **Go Version** | 1.26.0 |
| **Total Files** | 30+ |
| **Go Source Files** | 15+ |
| **Test Files** | 15+ |
| **Example Files** | 10+ |
| **Total Lines** | ~8,000 |
| **Test Coverage** | >80% |
| **Dependencies** | 4 direct, 15+ indirect |

---

## Architecture

### Interface-first design

**go-rag** follows a clean interface-based architecture:

```
┌─────────────────────────────────────────────────────────────────┐
│                        Application Code                            │
└─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Embedder Interface                            │
│  Embed(ctx, text) ([]float32, error)                              │
│  EmbedBatch(ctx, texts) ([][]float32, error)                       │
│  Dimensionality() int                                               │
└─────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
            ┌───────────┐   ┌───────────┐   ┌───────────┐
            │ Ollama    │   │ Custom 1  │   │ Custom 2  │
            │ Embedder  │   │ Embedder  │   │ Embedder  │
            └───────────┘   └───────────┘   └───────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     VectorStore Interface                          │
│  Upsert(ctx, collection, id, vector, metadata) error              │
│  Query(ctx, collection, queryVector, limit, threshold) ([]Result, error) │
│  Delete(ctx, collection, id) error                                  │
│  ListCollections(ctx) ([]string, error)                           │
└─────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
            ┌───────────┐   ┌───────────┐   ┌───────────┐
            │ Qdrant    │   │ Custom 1  │   │ Custom 2  │
            │ VectorStore│   │ VectorStore│   │ VectorStore│
            └───────────┘   └───────────┘   └───────────┘
```

### AX standard compliance

Every `.go` file has corresponding `_test.go` and `_example_test.go`. Comments are agent-first. SPOR (Single Point of Responsibility) per file. Decoupled interfaces enable mock-based testing.

### Module structure

```
core/go-rag/
├── go/
│   ├── qdrant.go                    # Qdrant implementation
│   ├── qdrant_test.go               # Unit tests
│   ├── query.go                     # Query operations & types
│   ├── query_test.go                # Query tests
│   ├── query_example_test.go       # Query examples
│   ├── ingest.go                    # Document ingestion
│   ├── ingest_test.go               # Ingestion tests
│   ├── ingest_example_test.go      # Ingestion examples
│   ├── chunk.go                     # Chunking strategies
│   ├── vectorstore.go               # VectorStore interface
│   ├── service.go                   # CoreGo service
│   ├── service_test.go              # Service tests
│   │
│   ├── cmd/rag/
│   │   ├── cmd_rag.go               # Main rag command
│   │   ├── cmd_collections.go       # Collection management
│   │   ├── cmd_commands.go           # Common utilities
│   │   ├── cmd_ingest.go            # Ingestion commands
│   │   ├── cmd_query.go             # Query commands
│   │   ├── cmd_commands_test.go
│   │   └── cmd_commands_example_test.go
│   │
│   ├── specs/                       # Specifications
│   ├── docs/                        # Documentation
│   │   ├── architecture.md          # Architecture overview
│   │   ├── development.md           # Development guide
│   │   └── history.md               # Project history
│   │
│   ├── external/                    # External dependencies
│   ├── go.mod                       # Module definition
│   └── go.sum                       # Dependencies
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

| Package | Description | Files | Status |
|---------|-------------|-------|--------|
| [qdrant.go](file:///Users/snider/Code/core/go-rag/go/qdrant.go) | Qdrant vector store | ~400 lines | Production |
| [query.go](file:///Users/snider/Code/core/go-rag/go/query.go) | Query operations | ~600 lines | Production |
| [ingest.go](file:///Users/snider/Code/core/go-rag/go/ingest.go) | Document ingestion | ~500 lines | Production |
| [chunk.go](file:///Users/snider/Code/core/go-rag/go/chunk.go) | Chunking strategies | ~300 lines | Production |
| [vectorstore.go](file:///Users/snider/Code/core/go-rag/go/vectorstore.go) | VectorStore interface | ~200 lines | Production |
| [service.go](file:///Users/snider/Code/core/go-rag/go/service.go) | CoreGo service | ~200 lines | Production |

### CLI commands

| Command | Description | File |
|---------|-------------|------|
| `core rag ingest` | Ingest documents into collection | [cmd_ingest.go](file:///Users/snider/Code/core/go-rag/go/cmd/rag/cmd_ingest.go) |
| `core rag query` | Query a collection | [cmd_query.go](file:///Users/snider/Code/core/go-rag/go/cmd/rag/cmd_query.go) |
| `core rag collections list` | List all collections | [cmd_collections.go](file:///Users/snider/Code/core/go-rag/go/cmd/rag/cmd_collections.go) |
| `core rag collections stats` | Show collection statistics | [cmd_collections.go](file:///Users/snider/Code/core/go-rag/go/cmd/rag/cmd_collections.go) |

---

## Configuration

### Qdrant configuration

```go
type QdrantConfig struct {
    Host   string // Qdrant server hostname
    Port   int    // Qdrant gRPC port (default: 6334)
    APIKey string // Optional API key
    UseTLS bool   // Enable TLS
}

// Default configuration
func DefaultQdrantConfig() QdrantConfig {
    return QdrantConfig{
        Host:   "localhost",
        Port:   6334,
        UseTLS: false,
    }
}

// Create store from endpoint URL
store, err := rag.NewQdrantStore("http://localhost:6333")

// Or with explicit configuration
store, err := rag.NewQdrantClient(rag.QdrantConfig{
    Host:   "qdrant.example.com",
    Port:   6334,
    APIKey: "your-api-key",
    UseTLS: true,
})
```

### Query configuration

```go
type QueryConfig struct {
    Collection string  // Vector collection to search
    Limit      uint64 // Maximum results (default: 5)
    Threshold  float32 // Minimum similarity score (0-1, default: 0.5)
    Category   string  // Optional category filter
    Keywords   bool    // Enable keyword boosting (default: false)
}

func DefaultQueryConfig() QueryConfig {
    return QueryConfig{
        Collection: "hostuk-docs",
        Limit:      5,
        Threshold:  0.5,
    }
}
```

### Ingestion configuration

```go
type IngestOptions struct {
    Chunker   Chunker       // Custom chunking strategy
    Recursive bool          // Recursively ingest directories
    Metadata  map[string]string // Additional metadata
}

func DefaultIngestOptions() IngestOptions {
    return IngestOptions{
        Chunker:   DefaultChunker(),
        Recursive: true,
    }
}
```

---

## Commands

### CLI commands

```bash
# Ingest a directory
core rag ingest /path/to/docs my-collection

# Ingest a single file
core rag ingest /path/to/file.md my-collection

# Query a collection
core rag query "how does RAG work?" my-collection --limit 5

# Query with threshold
core rag query "search term" my-collection --limit 10 --threshold 0.7

# List all collections
core rag collections list

# Show collection statistics
core rag collections stats my-collection

# Query with keyword boosting
core rag query "search term" my-collection --keywords
```

---

## Usage patterns

### 1. Quick start

```go
import "dappco.re/go/rag"

ctx := context.Background()

// Ingest documents
err := rag.IngestDirectory(ctx, "/path/to/docs", "my-collection", false)

// Query for context
context, err := rag.QueryDocsContext(ctx, "your query", "my-collection", 5)

// Use in LLM prompt
prompt := "Answer the question using this context:\n" + context + "\n\nQuestion: " + query
```

### 2. Interface-based testing

```go
// Mock implementations for testing
type mockEmbedder struct{}
func (m *mockEmbedder) Embed(ctx context.Context, text string) ([]float32, error) {
    return make([]float32, 1536), nil // Return mock embedding
}
func (m *mockEmbedder) EmbedBatch(ctx context.Context, texts []string) ([][]float32, error) {
    results := make([][]float32, len(texts))
    for i := range texts {
        results[i] = make([]float32, 1536)
    }
    return results, nil
}
func (m *mockEmbedder) Dimensionality() int { return 1536 }

type mockVectorStore struct{}
func (m *mockVectorStore) Upsert(ctx context.Context, collection, id string, vector []float32, metadata map[string]string) error {
    return nil
}
func (m *mockVectorStore) Query(ctx context.Context, collection string, queryVector []float32, limit uint64, threshold float32) ([]rag.QueryResult, error) {
    return []rag.QueryResult{
        {Text: "mock result", Source: "mock.md", Score: 0.95},
    }, nil
}

// Test with mocks
results, err := rag.QueryWith(ctx, &mockVectorStore{}, &mockEmbedder{}, "test", "test-collection", 5)
```

### 3. Custom chunking

```go
// Sentence-based chunking
chunker := rag.NewSentenceChunker(rag.SentenceChunkerConfig{
    SentencesPerChunk: 5,
    Overlap:           2,
})

// Ingest with custom chunker
err := rag.IngestFileWith(ctx, "file.md", "collection", rag.IngestOptions{
    Chunker: chunker,
})

// Paragraph-based chunking
chunker := rag.NewParagraphChunker(rag.ParagraphChunkerConfig{
    ParagraphsPerChunk: 3,
    Overlap:            1,
})
```

### 4. Batch operations

```go
// Ingest multiple files
files := []string{"file1.md", "file2.md", "file3.md"}
for _, file := range files {
    err := rag.IngestFile(ctx, file, "my-collection")
    if err != nil {
        log.Printf("Failed to ingest %s: %v", file, err)
    }
}

// Query and process results
results, err := rag.QueryDocs(ctx, "query", "collection", rag.QueryConfig{
    Limit:     10,
    Threshold: 0.7,
})

for _, result := range results {
    fmt.Printf("Score: %.4f, Source: %s\n", result.Score, result.Source)
    fmt.Printf("Text: %s\n\n", result.Text)
}
```

### 5. Collection management

```go
// List all collections
collections := rag.ListCollectionsSeq(ctx)
for collection := range collections {
    fmt.Println(collection)
}

// Get collection statistics
stats, err := rag.CollectionStats(ctx, "my-collection")
if err != nil {
    log.Fatal(err)
}

fmt.Printf("Documents: %d\n", stats.DocumentCount)
fmt.Printf("Vectors: %d\n", stats.VectorCount)
fmt.Printf("Average Vector Size: %d\n", stats.AvgVectorSize)
```

---

## Testing

### Test coverage by file

| File | Functions | Lines | Coverage |
|------|-----------|-------|----------|
| `qdrant.go` | 15+ | ~400 | >80% |
| `query.go` | 20+ | ~600 | >85% |
| `ingest.go` | 15+ | ~500 | >80% |
| `chunk.go` | 10+ | ~300 | >80% |
| `vectorstore.go` | 10+ | ~200 | >75% |
| `service.go` | 10+ | ~200 | >75% |
| `cmd/rag/*.go` | 20+ | ~500 | >75% |

### Test tags

```bash
# Unit tests (no external services required)
go test ./...

# Full integration tests (requires Qdrant + Ollama)
go test -tags rag ./...

# With race detector
go test -race ./...

# Specific package
go test -v ./go/...

# With coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html
```

All core functionality can be tested without external services using mock implementations of `Embedder` and `VectorStore` interfaces.

---

## API reference

### Core types

```go
// QueryConfig - Query configuration
type QueryConfig struct {
    Collection string
    Limit      uint64
    Threshold  float32
    Category   string
    Keywords   bool
}

// QueryResult - Single query result
type QueryResult struct {
    Text       string
    Source     string
    Section    string
    Category   string
    ChunkIndex int
    Index      int
    Score      float32
}

// Methods on QueryResult
func (r QueryResult) GetText() string
func (r QueryResult) GetScore() float32
func (r QueryResult) GetSource() string
func (r QueryResult) HasChunkIndex() bool

// CollectionStats - Collection statistics
type CollectionStats struct {
    CollectionName string
    DocumentCount  int
    VectorCount    int
    AvgVectorSize  int
    TotalSize      int64
}
```

### Qdrant configuration

```go
type QdrantConfig struct {
    Host   string
    Port   int
    APIKey string
    UseTLS bool
}

func DefaultQdrantConfig() QdrantConfig
func NewQdrantStore(endpoint string) core.Result
func NewQdrantClient(cfg QdrantConfig) core.Result
```

### Main functions

```go
// Ingestion
func IngestDirectory(ctx context.Context, dir, collection string, recursive bool) error
func IngestFile(ctx context.Context, path, collection string) error
func IngestFileWith(ctx context.Context, path, collection string, opts IngestOptions) core.Result
func IngestDirWith(ctx context.Context, dir, collection string, opts IngestOptions) core.Result

// Query
func QueryDocs(ctx context.Context, query, collection string, cfg QueryConfig) ([]QueryResult, error)
func QueryDocsContext(ctx context.Context, query, collection string, limit int) (string, error)
func QueryWith(ctx context.Context, store VectorStore, embedder Embedder, query, collection string, limit uint64) ([]QueryResult, error)
func QueryContextWith(ctx context.Context, store VectorStore, embedder Embedder, query, collection string, limit uint64) (string, error)
func QueryContextIter(ctx context.Context, query, collection string, limit uint64) iter.Seq[QueryResult]

// Collections
func CollectionStats(ctx context.Context, collection string) (*CollectionStats, error)
func ListCollectionsSeq(ctx context.Context) iter.Seq[string]

// Chunking
func ChunkBySentences(text string, cfg SentenceChunkerConfig) []string
func ChunkByParagraphs(text string, cfg ParagraphChunkerConfig) []string
func DefaultChunker() Chunker

// Convenience helpers
func FileExtensions() []string
func ShouldProcess(path string) bool
func JoinResults(results []QueryResult) string
func KeywordFilterSeq(results []QueryResult, keywords []string) iter.Seq[QueryResult]
```

### Interfaces

```go
// VectorStore - Vector database abstraction
type VectorStore interface {
    Upsert(ctx context.Context, collection, id string, vector []float32, metadata map[string]string) error
    Query(ctx context.Context, collection string, queryVector []float32, limit uint64, threshold float32) ([]QueryResult, error)
    Delete(ctx context.Context, collection, id string) error
    DeleteCollection(ctx context.Context, collection string) error
    ListCollections(ctx context.Context) ([]string, error)
    CollectionStats(ctx context.Context, collection string) (*CollectionStats, error)
}

// Embedder - Embedding generation abstraction
type Embedder interface {
    Embed(ctx context.Context, text string) ([]float32, error)
    EmbedBatch(ctx context.Context, texts []string) ([][]float32, error)
    Dimensionality() int
}

// Chunker - Chunking strategy abstraction
type Chunker interface {
    Chunk(text string) []string
}
```

---

## Related documentation

### Internal documentation

| Resource | Description | Location |
|----------|-------------|----------|
| RFC | Package specification and design | [plans/code/core/go/rag/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/rag/RFC.md) |
| Architecture | Interfaces, chunking strategy, ingestion and query pipelines | [docs/architecture.md](file:///Users/snider/Code/core/go-rag/docs/architecture.md) |
| Development guide | Prerequisites, build, test patterns, coding standards | [docs/development.md](file:///Users/snider/Code/core/go-rag/docs/development.md) |
| Project history | Completed phases with commit hashes, known limitations | [docs/history.md](file:///Users/snider/Code/core/go-rag/docs/history.md) |

### External references

| Resource | URL |
|----------|-----|
| Repository | [github.com/dappcore/go-rag](https://github.com/dappcore/go-rag) |
| Go Module | [pkg.go.dev/dappco.re/go/rag](https://pkg.go.dev/dappco.re/go/rag) |
| Qdrant | [qdrant.tech](https://qdrant.tech) — Vector similarity search engine |
| Ollama | [ollama.ai](https://ollama.ai) — Local LLM inference |
| RAG Pattern | [Wikipedia](https://en.wikipedia.org/wiki/Retrieval-augmented_generation) |

---

## Dependency analysis

```
Direct Dependencies:    4
  ├── dappco.re/go v0.10.4              # Core utilities
  ├── github.com/ledongthuc/pdf v0.0.0-20250511090121-5959a4027728  # PDF extraction
  ├── github.com/ollama/ollama v0.18.1    # Ollama client
  └── github.com/qdrant/go-client v1.17.1 # Qdrant client

Indirect Dependencies: 15+
  ├── go.opentelemetry.io/otel v1.42.0
  ├── google.golang.org/grpc v1.79.2
  ├── gonum.org/v1/gonum v0.17.0
  └── gopkg.in/yaml.v3 v3.0.1
```

---

*Last updated: 2026-06-18 | Maintainer: Purberus <purberus@lthn.ai>*
