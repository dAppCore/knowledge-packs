# go-rag — Retrieval-Augmented Generation

> **Package:** `dappco.re/go/rag`  
> **Repository:** [`github.com/dappcore/go-rag`](https://github.com/dappcore/go-rag)  
> **Spec:** [`plans/code/core/go/rag/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/rag/RFC.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>  
> **Status:** ✅ Production Ready  
> **Module:** `forge.lthn.ai/core/go-rag`

---

## 📋 Overview

**go-rag** is a comprehensive Retrieval-Augmented Generation (RAG) library for Go. It provides document chunking with multiple strategies, embedding generation via Ollama, vector storage and similarity search via Qdrant, TF-IDF keyword fallback, and result formatting in multiple output formats. Designed around decoupled `Embedder` and `VectorStore` interfaces for maximum flexibility and testability.

### 🎯 Key Capabilities

| Category | Features | Description |
|----------|----------|-------------|
| **Chunking** | 3-level Markdown splitting + sentence/paragraph chunkers | Configurable overlap, multiple strategies |
| **Embeddings** | Ollama integration | Local LLM embedding generation |
| **Vector Storage** | Qdrant gRPC | Vector database with cosine similarity search |
| **Fallback** | TF-IDF keyword search | Boosts relevance when vector search is unavailable |
| **Post-Filtering** | Keyword boosting | Enhances results based on keyword matches |
| **Output Formats** | Plain text, XML, JSON | Flexible result formatting for LLM prompt injection |
| **Interfaces** | Embedder, VectorStore | Decoupled design for easy mocking and testing |

### 🏗️ Architecture

### Module Structure

```
core/go-rag/
├── go/                          # Go module root (dappco.re/go/rag)
│   ├── qdrant.go                # Qdrant vector store implementation
│   ├── qdrant_test.go           # Unit tests
│   ├── query.go                 # Query operations and types
│   ├── query_test.go           # Query tests
│   ├── query_example_test.go  # Query usage examples
│   ├── ingest.go                # Document ingestion
│   ├── ingest_test.go           # Ingestion tests
│   ├── ingest_example_test.go # Ingestion usage examples
│   ├── chunk.go                 # Chunking strategies
│   ├── vectorstore.go           # VectorStore interface
│   ├── service.go               # CoreGo service integration
│   ├── service_test.go          # Service tests
│   │
│   ├── cmd/                     # CLI commands
│   │   └── rag/                  # RAG-specific commands
│   │       ├── cmd_rag.go       # Main rag command
│   │       ├── cmd_collections.go # Collection management
│   │       ├── cmd_commands.go   # Common command utilities
│   │       ├── cmd_ingest.go    # Ingestion commands
│   │       ├── cmd_query.go     # Query commands
│   │       ├── cmd_commands_test.go
│   │       └── cmd_commands_example_test.go
│   │
│   ├── specs/                   # Specifications
│   ├── docs/                    # Documentation
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

1. **Interface-First** — `Embedder` and `VectorStore` interfaces decouple implementation from business logic
2. **Mock-Based Testing** — All operations can be tested without external services
3. **Multiple Chunking Strategies** — 3-level Markdown, sentence-based, paragraph-based
4. **Fallback Support** — TF-IDF keyword search when vector search is unavailable
5. **AX Standard** — Each `.go` file has `_test.go` and `_example_test.go`

---

## 📦 Packages

### Chunking Strategies

#### Markdown Chunking
- **3-Level Splitting** — Document → Sections → Chunks
- **Configurable Overlap** — Adjustable chunk overlap for better context
- **Semantic Boundaries** — Respects Markdown structure (headers, code blocks, etc.)

#### Sentence-Based Chunking
- **NLTK-inspired** — Splits on sentence boundaries
- **Configurable Length** — Adjustable sentence count per chunk
- **Context Preservation** — Maintains document flow

#### Paragraph-Based Chunking
- **Simple & Effective** — Splits on paragraph breaks
- **Configurable Size** — Adjustable paragraph count per chunk
- **Fast Processing** — Minimal computational overhead

### Embedding Generation

**Ollama Integration:**
- Local LLM embedding models
- Batch embedding generation
- Configurable model selection
- Error handling and retries

### Vector Storage (Qdrant)

**Features:**
- gRPC-based communication
- Cosine similarity search
- Collection management
- Vector indexing and querying
- Result filtering and pagination

### Fallback Mechanisms

#### TF-IDF Keyword Search
- Traditional keyword-based search
- Automatic fallback when vector search fails
- Configurable keyword extraction

#### Keyword Boosting
- Post-filtering enhancement
- Boosts results matching specific keywords
- Configurable boost factors

---

## 🔧 Configuration

### Client Initialization

```go
import "dappco.re/go/rag"

// Create Qdrant store
store, err := rag.NewQdrantStore("http://localhost:6333")
if err != nil {
    log.Fatal(err)
}

// Or with custom configuration
store, err = rag.NewQdrantClient(rag.QdrantConfig{
    Host:   "localhost",
    Port:   6334, // gRPC port
    UseTLS: false,
})
```

### Query Configuration

```go
cfg := rag.QueryConfig{
    Collection: "my-docs",
    Limit:      5,
    Threshold:  0.6, // Minimum similarity score
    Category:   "api", // Optional category filter
    Keywords:   true, // Enable keyword boosting
}
```

---

## 🚀 Commands

### CLI Integration

```bash
# Ingest documents
core rag ingest /path/to/docs my-collection

# Query collection
core rag query "how does RAG work?" my-collection --limit 5

# List collections
core rag collections list

# Get collection stats
core rag collections stats my-collection
```

---

## 📝 Usage Patterns

### 1. Basic Ingestion and Query

```go
import "dappco.re/go/rag"

ctx := context.Background()

// Ingest a directory of Markdown files
err := rag.IngestDirectory(ctx, "/path/to/docs", "my-collection", false)
if err != nil {
    log.Fatal(err)
}

// Query for relevant context
context, err := rag.QueryDocsContext(ctx, "how does rate limiting work?", "my-collection", 5)
if err != nil {
    log.Fatal(err)
}

fmt.Println(context)
```

### 2. Interface-Based Usage

```go
// Use interfaces for dependency injection
var (
    store     rag.VectorStore = rag.NewQdrantStore("http://localhost:6333")
    embedder  rag.Embedder   = rag.NewOllamaEmbedder("all-minilm")
)

// Query with custom implementations
results, err := rag.QueryWith(ctx, store, embedder, "question", "collection", 5)
if err != nil {
    log.Fatal(err)
}

for _, result := range results {
    fmt.Printf("Score: %.4f, Source: %s\n", result.Score, result.Source)
    fmt.Printf("Text: %s\n\n", result.Text)
}
```

### 3. Advanced Query with Options

```go
ctx := context.Background()

// Query with custom configuration
results, err := rag.QueryDocs(ctx, "search query", "my-collection", rag.QueryConfig{
    Limit:     10,
    Threshold: 0.7,
    Keywords:  true,
})
if err != nil {
    log.Fatal(err)
}

// Get collection statistics
stats, err := rag.CollectionStats(ctx, "my-collection")
if err != nil {
    log.Fatal(err)
}

fmt.Printf("Collection has %d documents\n", stats.DocumentCount)
```

### 4. Ingestion with Custom Chunker

```go
ctx := context.Background()

// Use sentence-based chunking
chunker := rag.NewSentenceChunker(rag.SentenceChunkerConfig{
    SentencesPerChunk: 5,
    Overlap:           2,
})

// Ingest with custom chunker
result := rag.IngestFileWith(ctx, "/path/to/file.md", "my-collection", rag.IngestOptions{
    Chunker: chunker,
})

if !result.OK {
    log.Fatal(result.Error)
}
```

### 5. Streaming Query Results

```go
ctx := context.Background()

// Stream results as they become available
for result := range rag.QueryContextIter(ctx, "query", "collection", 5) {
    fmt.Printf("Score: %.4f\n", result.Score)
    fmt.Printf("Source: %s\n", result.Source)
    fmt.Printf("Text: %s\n\n", result.Text)
}
```

---

## 🧪 Testing

### Test Structure

Each file follows the AX standard:
- `_test.go` — Unit tests with mocked services
- `_example_test.go` — Usage examples as tests

### Test Coverage

| File | Functions | Lines | Coverage |
|------|-----------|-------|----------|
| `qdrant.go` | 15+ | ~400 | >80% |
| `query.go` | 20+ | ~600 | >85% |
| `ingest.go` | 15+ | ~500 | >80% |
| `chunk.go` | 10+ | ~300 | >80% |
| `vectorstore.go` | 10+ | ~200 | >75% |
| `service.go` | 10+ | ~200 | >75% |

### Test Tags

```bash
# Unit tests only (no external services)
go test ./...

# Full suite with live Qdrant + Ollama
go test -tags rag ./...

# With race detector
go test -race ./...

# With coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

---

## 📖 API Reference

### Core Types

```go
// QueryConfig - Configuration for query operations
type QueryConfig struct {
    Collection string
    Limit      uint64
    Threshold  float32
    Category   string
    Keywords   bool
}

// QueryResult - A single query result
type QueryResult struct {
    Text       string
    Source     string
    Section    string
    Category   string
    ChunkIndex int
    Score      float32
}

// QueryResult methods
func (r QueryResult) GetText() string
func (r QueryResult) GetScore() float32
func (r QueryResult) GetSource() string
func (r QueryResult) HasChunkIndex() bool

// QdrantConfig - Qdrant connection configuration
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

### Main Functions

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

// Collections
func CollectionStats(ctx context.Context, collection string) (*CollectionStats, error)
func ListCollectionsSeq(ctx context.Context) iter.Seq[string]

// Chunking
func ChunkBySentences(text string, cfg SentenceChunkerConfig) []string
func ChunkByParagraphs(text string, cfg ParagraphChunkerConfig) []string
```

### Interfaces

```go
// VectorStore interface
type VectorStore interface {
    Upsert(ctx context.Context, collection string, id string, vector []float32, metadata map[string]string) error
    Query(ctx context.Context, collection string, queryVector []float32, limit uint64, threshold float32) ([]QueryResult, error)
    // ... more methods
}

// Embedder interface
type Embedder interface {
    Embed(ctx context.Context, text string) ([]float32, error)
    EmbedBatch(ctx context.Context, texts []string) ([][]float32, error)
    Dimensionality() int
}
```

---

## 🔗 Related Documentation

### Internal Documentation

| Resource | Description | Location |
|----------|-------------|----------|
| RFC | Package specification | [plans/code/core/go/rag/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/rag/RFC.md) |
| Architecture | Interfaces, chunking strategy, pipelines | [docs/architecture.md](file:///Users/snider/Code/core/go-rag/docs/architecture.md) |
| Development Guide | Prerequisites, build, test patterns | [docs/development.md](file:///Users/snider/Code/core/go-rag/docs/development.md) |
| Project History | Completed phases, known limitations | [docs/history.md](file:///Users/snider/Code/core/go-rag/docs/history.md) |

### External References

| Resource | URL |
|----------|-----|
| Repository | [github.com/dappcore/go-rag](https://github.com/dappcore/go-rag) |
| Module | [pkg.go.dev/dappco.re/go/rag](https://pkg.go.dev/dappco.re/go/rag) |
| Qdrant | [qdrant.tech](https://qdrant.tech) |
| Ollama | [ollama.ai](https://ollama.ai) |

---

## 📊 Statistics

### Code Metrics

```
Total Repository Size:    ~20 MB
Go Source Files:         15+
Test Files:             15+
Example Files:          10+
Total Go Lines:          ~5,000
Test Lines:             ~3,000
Documentation:          ~1,000 lines
```

### Feature Coverage

| Feature | Status | Details |
|---------|--------|---------|
| Chunking Strategies | ✅ Complete | 3-level Markdown, sentence, paragraph |
| Embedding Generation | ✅ Complete | Ollama integration |
| Vector Storage | ✅ Complete | Qdrant gRPC |
| Similarity Search | ✅ Complete | Cosine similarity |
| Keyword Fallback | ✅ Complete | TF-IDF search |
| Keyword Boosting | ✅ Complete | Post-filter enhancement |
| Output Formats | ✅ Complete | Plain text, XML, JSON |
| Service Integration | ✅ Complete | CoreGo service |
| CLI Commands | ✅ Complete | Ingest, query, collections |

### Test Statistics

| Type | Count | Coverage |
|------|-------|----------|
| Unit Tests | 50+ | >80% |
| Example Tests | 15+ | N/A |
| Integration Tests | 10+ | >75% |

### Dependency Statistics

```
Direct Dependencies:    4
  ├── dappco.re/go v0.10.4
  ├── github.com/ledongthuc/pdf v0.0.0-20250511090121-5959a4027728
  ├── github.com/ollama/ollama v0.18.1
  └── github.com/qdrant/go-client v1.17.1

Indirect Dependencies: 15+
```

---

## 🏷️ Tags

#rag #retrieval-augmented-generation #llm #embeddings #vector-database #qdrant #ollama #chunking #similarity-search #cosine-similarity #tf-idf #keyword-boosting #document-processing #markdown #pdf #ai #machine-learning #agent-first #ax-standard

---

*Last updated: 2026-06-18 | Maintainer: Purberus <purberus@lthn.ai>*
