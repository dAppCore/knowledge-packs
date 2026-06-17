---
type: Package Documentation
package: api
module: dappco.re/go/api
repo: core/api
lang: go
tags:
  - rest
  - gin
  - openapi
  - swagger
  - websocket
  - sse
  - authentication
  - authorization
  - middleware
  - rate-limiting
  - webhook
  - graphql
  - chat
---
# go-api — REST Framework with OpenAPI Generation

> **The REST framework for the Lethean ecosystem — Gin-based HTTP engine with OpenAPI, WebSocket, SSE, and ToolBridge**

**Source:** [~/Code/core/api/](file:///Users/snider/Code/core/api/)
**Module:** `dappco.re/go/api`
**PHP Package:** `forge.lthn.ai/core/api` (Laravel package in php/)
**Dependencies:** `dappco.re/go`, `dappco.re/go/cli`, `github.com/gin-gonic/gin`, `github.com/casbin/casbin/v2`, `github.com/coreos/go-oidc/v3`
**Status:** ✅ Production-Ready
**License:** EUPL-1.2

---

## 🎯 Overview

`go-api` is the **REST framework** for CoreGO applications, providing a Gin-based HTTP engine with comprehensive features including OpenAPI/Swagger generation, WebSocket support, Server-Sent Events (SSE), ToolBridge for MCP integration, authentication (Authentik OIDC + Bearer tokens), rate limiting, webhooks, GraphQL, chat completions, and more.

### Primary Use Cases

1. **REST API Server** — Build HTTP APIs with Gin router
2. **OpenAPI Generation** — Auto-generate OpenAPI 3.1 specifications
3. **SDK Generation** — Generate client SDKs in 11+ languages
4. **WebSocket Support** — Real-time bidirectional communication
5. **Server-Sent Events** — Real-time streaming to clients
6. **Authentication** — Authentik OIDC + static Bearer token support
7. **Authorization** — Casbin-based policy enforcement
8. **Rate Limiting** — Token bucket and sliding window algorithms
9. **Webhooks** — Incoming and outgoing webhook support
10. **GraphQL** — GraphQL endpoint with schema support
11. **Chat Completions** — AI chat endpoint with local and remote backends
12. **ToolBridge** — MCP tool integration for AI agents
13. **Upstream Proxy** — Reverse proxy with load balancing

### Design Philosophy

- **Modular** — Route groups for organizing endpoints by domain
- **Extensible** — Middleware chain and functional options pattern
- **Observable** — OpenTelemetry tracing, metrics, pprof
- **Secure** — SSRF protection, input validation, rate limiting
- **Developer-Friendly** — Auto-generated docs, SDKs, and examples
- **Production-Ready** — Graceful shutdown, health checks, caching

### Repository Layout

```
core/api/
├── go/                                ← Go module root (dappco.re/go/api)
├── php/                               ← PHP Laravel package
├── docs/                              ← Shared documentation
├── sdk-config/                        ← SDK generation configs
└── scripts/                           ← Build helpers
```

**Cross-language symmetry:** `dappco.re/<lang>/api/<feature>` ↔ `core/api/<lang>/<feature>`

---

## 🏗️ Architecture

### Component Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Engine Layer                             │
│  Engine — Central router with middleware chain              │
│  RouteGroup — Domain-specific endpoint groups                │
│  StreamGroup — WebSocket/SSE streaming groups                │
│  DescribableGroup — Route groups with OpenAPI metadata      │
├─────────────────────────────────────────────────────────────┤
│                    Feature Layer                            │
│  OpenAPI — Spec generation + Swagger UI                      │
│  ToolBridge — MCP tool registration as REST endpoints       │
│  Authentication — Authentik OIDC + Bearer tokens            │
│  Authorization — Casbin policy enforcement                  │
│  Rate Limiting — Token bucket + sliding window              │
│  Webhooks — Incoming/outgoing HTTP callbacks                 │
│  GraphQL — GraphQL endpoint with schema support              │
│  Chat — AI chat completions (local + remote backends)       │
│  Upstream — Reverse proxy with load balancing               │
├─────────────────────────────────────────────────────────────┤
│                    Transport Layer                           │
│  HTTP/1.1 — Standard HTTP                                    │
│  HTTP/2 — Full support                                       │
│  HTTP/3 — Optional support                                   │
│  WebSocket — Bidirectional real-time                         │
│  SSE — Server-Sent Events                                    │
├─────────────────────────────────────────────────────────────┤
│                    Middleware Layer                          │
│  Auth — Bearer token validation                              │
│  RateLimit — Request throttling                              │
│  CORS — Cross-origin handling                                │
│  Logging — Request/response logging                          │
│  Tracing — OpenTelemetry instrumentation                     │
│  SSRF Guard — SSRF attack prevention                        │
│  Sunset — Deprecation headers                                │
│  Brotli — Response compression                               │
│  Gzip — Response compression                                 │
├─────────────────────────────────────────────────────────────┤
│                    Utility Layer                             │
│  Response — Standardized response helpers                    │
│  Cache — HTTP caching with TTL                               │
│  Transformers — Request/response transformation             │
│  Helpers — JSON, validation, error handling                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 Package Structure

```
core/api/go/
├── api.go                          # Engine type + New() + Serve()
├── api_test.go                      # Engine tests
├── api_example_test.go              # Engine usage examples
├── api_describable_test.go           # Describable tests
├── api_renderable_test.go           # Renderable tests
│
├── group.go                         # RouteGroup interface + base implementations
├── group_test.go                    # RouteGroup tests
├── group_example_test.go            # RouteGroup examples
│
├── options.go                       # Functional options for Engine
├── options_test.go                  # Options tests
├── options_example_test.go          # Options usage examples
│
├── server.go                        # HTTP server lifecycle management
├── server_test.go                   # Server tests
│
├── service.go                       # Core service integration
├── service_test.go                  # Service tests
├── service_example_test.go          # Service usage examples
│
├── middleware.go                    # Middleware helpers
├── middleware_test.go               # Middleware tests
├── middleware_example_test.go       # Middleware usage examples
│
├── authentik.go                    # Authentik OIDC integration
├── authentik_test.go                # Authentik tests
├── authentik_example_test.go        # Authentik usage examples
├── authentik_integration_test.go    # Authentik integration tests
│
├── authz.go                         # Authorization (Casbin)
├── authz_test.go                    # Authorization tests
│
├── ratelimit.go                     # Rate limiting
├── ratelimit_test.go                # Rate limit tests
├── ratelimit_internal_test.go       # Internal rate limit tests
│
├── cache.go                         # HTTP caching
├── cache_test.go                    # Cache tests
├── cache_config.go                  # Cache configuration
├── cache_config_test.go             # Cache config tests
├── cache_config_example_test.go     # Cache config examples
├── cache_control.go                 # Cache control headers
├── cache_control_test.go            # Cache control tests
├── cache_control_example_test.go    # Cache control examples
│
├── webhook.go                       # Webhook handling
├── webhook_test.go                  # Webhook tests
├── webhook_example_test.go          # Webhook usage examples
├── webhook_internal_test.go         # Internal webhook tests
│
├── swagger.go                       # Swagger UI
├── swagger_test.go                  # Swagger tests
├── swagger_internal_test.go         # Internal Swagger tests
│
├── openapi.go                       # OpenAPI spec generation
├── openapi_test.go                  # OpenAPI tests
├── openapi_example_test.go          # OpenAPI usage examples
├── openapitools.json                # OpenAPI tools config
│
├── codegen.go                       # SDK code generation
├── codegen_test.go                  # Codegen tests
├── codegen_example_test.go          # Codegen usage examples
│
├── export.go                        # OpenAPI export helpers
├── export_test.go                   # Export tests
├── export_example_test.go           # Export usage examples
│
├── spec_registry.go                 # OpenAPI spec registry
├── spec_registry_test.go            # Spec registry tests
├── spec_registry_example_test.go    # Spec registry usage examples
│
├── spec_builder_helper.go           # OpenAPI spec builder helpers
├── spec_builder_helper_test.go      # Spec builder helper tests
│
├── response.go                      # Response helpers
├── response_test.go                 # Response tests
├── response_example_test.go         # Response usage examples
├── response_meta.go                 # Response metadata
├── response_meta_test.go            # Response metadata tests
├── response_meta_example_test.go    # Response metadata usage examples
│
├── graphql.go                       # GraphQL support
├── graphql_test.go                  # GraphQL tests
├── graphql_example_test.go          # GraphQL usage examples
│
├── chat_completions.go              # Chat completions endpoint
├── chat_completions_test.go         # Chat completions tests
├── chat_completions_example_test.go # Chat completions usage examples
├── chat_completions_internal_test.go # Internal chat tests
│
├── chat_remote.go                    # Remote chat backend
├── chat_remote_test.go               # Remote chat tests
├── chat_remote_example_test.go       # Remote chat usage examples
├── chat_remote_internal_test.go      # Internal remote chat tests
│
├── chat_adapter.go                  # Chat adapter interface
├── chat_adapter_anthropic.go        # Anthropic adapter
├── chat_adapter_anthropic_test.go   # Anthropic adapter tests
├── chat_adapter_ollama.go           # Ollama adapter
├── chat_adapter_ollama_test.go      # Ollama adapter tests
│
├── bridge.go                        # ToolBridge (MCP tools as REST)
├── bridge_test.go                   # Bridge tests
├── bridge_example_test.go           # Bridge usage examples
├── bridge_internal_test.go          # Internal bridge tests
│
├── sse.go                           # Server-Sent Events
├── sse_test.go                      # SSE tests
├── sse_example_test.go               # SSE usage examples
├── sse_internal_test.go             # Internal SSE tests
│
├── websocket.go                     # WebSocket support
├── websocket_test.go                # WebSocket tests
├── websocket_example_test.go        # WebSocket usage examples
├── websocket_internal_test.go      # Internal WebSocket tests
│
├── ssrf_guard.go                    # SSRF attack prevention
├── ssrf_guard_test.go               # SSRF guard tests
├── ssrf_guard_example_test.go       # SSRF guard usage examples
├── ssrf_guard_internal_test.go      # Internal SSRF tests
│
├── timeout.go                       # Request timeout handling
├── timeout_test.go                  # Timeout tests
│
├── tracing.go                       # OpenTelemetry tracing
├── tracing_test.go                  # Tracing tests
├── tracing_example_test.go         # Tracing usage examples
│
├── pprof.go                         # pprof endpoint
│
├── expvar.go                        # expvar endpoint
│
├── sunset.go                        # Deprecation headers
├── sunset_test.go                   # Sunset tests
├── sunset_example_test.go           # Sunset usage examples
│
├── brotli.go                        # Brotli compression
├── brotli_test.go                   # Brotli tests
├── brotli_example_test.go           # Brotli usage examples
│
├── gzip.go                          # Gzip compression
├── gzip_test.go                     # Gzip tests
│
├── json_helpers.go                  # JSON utilities
├── json_helpers_test.go             # JSON helpers tests
├── json_helpers_example_test.go     # JSON helpers usage examples
│
├── i18n.go                          # i18n support
├── i18n_test.go                     # i18n tests
├── i18n_example_test.go             # i18n usage examples
│
├── location.go                      # Location headers
├── location_test.go                 # Location tests
│
├── string_constants.go              # String constants
├── text_helpers.go                  # Text utilities
│
├── transformer.go                   # Request/response transformers
├── transformer_test.go              # Transformer tests
├── transformer_example_test.go     # Transformer usage examples
├── transformer_in.go                # Input transformers
├── transformer_in_test.go           # Input transformer tests
├── transformer_in_example_test.go   # Input transformer usage examples
├── transformer_out.go               # Output transformers
├── transformer_out_test.go          # Output transformer tests
├── transformer_out_example_test.go # Output transformer usage examples
│
├── transport.go                     # HTTP transport utilities
├── transport_test.go                # Transport tests
├── transport_example_test.go        # Transport usage examples
│
├── transport_client.go              # HTTP client transport
├── transport_client_test.go         # Client transport tests
├── transport_client_example_test.go # Client transport usage examples
├── transport_client_internal_test.go # Internal client tests
│
├── upstream_balancer.go              # Load balancer
├── upstream_balancer_test.go         # Balancer tests
├── upstream_balancer_internal_test.go # Internal balancer tests
│
├── upstream_registry.go              # Upstream registry
├── upstream_registry_test.go         # Registry tests
├── upstream_registry_internal_test.go # Internal registry tests
│
├── upstream_router.go                # Upstream router
├── upstream_router_test.go            # Router tests
├── upstream_router_example_test.go    # Router usage examples
├── upstream_router_internal_test.go  # Internal router tests
│
├── upstream_transport.go             # Upstream transport
├── upstream_transport_test.go        # Upstream transport tests
├── upstream_transport_internal_test.go
│
├── strict_bind_test.go              # Strict bind tests
│
├── go.mod
├── go.sum
├── go.work
│
├── cmd/
│   └── api/
│       ├── cmd.go                    # CLI command
│       ├── cmd_test.go               # CLI tests
│       ├── cmd_example_test.go       # CLI usage examples
│       ├── cmd_args.go               # CLI argument parsing
│       ├── cmd_args_test.go          # CLI args tests
│       ├── cmd_spec.go               # Spec command
│       ├── cmd_spec_test.go          # Spec command tests
│       ├── cmd_sdk.go                # SDK command
│       ├── cmd_sdk_test.go           # SDK command tests
│       └── core_helpers.go            # Core CLI helpers
│
└── pkg/
    └── stream/                       # Streaming support
        └── (see separate documentation)
```

---

## 🚀 Getting Started

### Basic HTTP Server

```go
package main

import (
    "context"
    "os"
    "os/signal"
    "syscall"

    "dappco.re/go/api"
)

func main() {
    // Create engine with default options
    engine, err := api.New()
    if err != nil {
        panic(err)
    }

    // Register a simple route group
    engine.Register(myRouteGroup)

    // Handle graceful shutdown
    ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
    defer stop()

    // Start server
    if err := engine.Serve(ctx); err != nil {
        os.Exit(1)
    }
}
```

### With Custom Configuration

```go
package main

import (
    "dappco.re/go/api"
)

func main() {
    engine, err := api.New(
        api.WithAddr(":8081"),
        api.WithCORS("*"),
        api.WithResponseMeta(),
        api.WithSwagger("My API", "1.0.0"),
        api.WithPProf(),
        api.WithExpvar(),
    )
    if err != nil {
        panic(err)
    }

    engine.Register(myRouteGroup)
    engine.Serve(context.Background())
}
```

### With Authentication

```go
package main

import (
    "dappco.re/go/api"
)

func main() {
    engine, err := api.New(
        api.WithAddr(":8081"),
        api.WithBearerAuth("my-secret-token"),
        api.WithAuthentik("https://auth.example.com", "client-id", "client-secret"),
    )
    if err != nil {
        panic(err)
    }

    // Require authentication for all routes
    engine.Register(authRouteGroup)
    
    engine.Serve(context.Background())
}
```

### With OpenAPI

```go
package main

import (
    "dappco.re/go/api"
)

func main() {
    engine, err := api.New(
        api.WithAddr(":8081"),
        api.WithSwagger("My API", "1.0.0"),
        api.WithOpenAPISpec("/openapi.json"),
    )
    if err != nil {
        panic(err)
    }

    // Register describable route groups for auto-documentation
    engine.Register(&MyDescribableGroup{})
    
    engine.Serve(context.Background())
}
```

---

## 🔧 Core Types

### Engine

The central API server type that manages route groups, middleware, and configuration.

```go
type Engine struct {
    // Server configuration
    addr             string
    http3Enabled     bool
    http3Addr        string
    
    // Route management
    groups           []RouteGroup
    streamGroups     []apistream.StreamGroup
    
    // Middleware
    middlewares      []gin.HandlerFunc
    
    // Chat completions
    chatCompletionsResolver *ModelResolver
    chatCompletionsPath    string
    
    // SDK generation
    sdkGenEnabled    bool
    
    // Caching
    cacheTTL         time.Duration
    cacheMaxEntries  int
    cacheMaxBytes    int
    
    // WebSocket
    wsHandler        http.Handler
    wsGinHandler     gin.HandlerFunc
    wsPath           string
    
    // SSE
    sseBroker        *SSEBroker
    ssePath          string
    
    // Swagger/OpenAPI
    swaggerEnabled   bool
    swaggerTitle     string
    swaggerSummary   string
    swaggerDesc      string
    swaggerVersion   string
    swaggerPath      string
    swaggerTermsOfService string
    swaggerServers   []string
    swaggerContactName string
    swaggerContactURL string
    swaggerContactEmail string
    swaggerLicenseName string
    swaggerLicenseURL string
    swaggerSecuritySchemes map[string]any
    swaggerExternalDocsDescription string
    swaggerExternalDocsURL string
    
    // Authentication
    authentikConfig  AuthentikConfig
    bearerToken      string
    bearerConfigured bool
    strictBind       bool
    publicBind       bool
    
    // Features
    pprofEnabled    bool
    expvarEnabled   bool
    
    // Upstream
    upstreamRouter   *upstreamRouterConfig
    
    // Chat remote
    chatRemote       *chatRemoteConfig
    chatAllowRemote  bool
}
```

**Key Methods:**
- `New(opts ...Option) (*Engine, error)` — Create new engine
- `Serve(ctx context.Context) error` — Start HTTP server
- `Handler() http.Handler` — Get HTTP handler (for custom server)
- `Register(groups ...RouteGroup)` — Register route groups
- `Use(middleware ...gin.HandlerFunc)` — Add middleware

### RouteGroup Interface

```go
type RouteGroup interface {
    Name() string
    BasePath() string
    RegisterRoutes(*gin.RouterGroup)
}
```

### StreamGroup Interface

```go
type StreamGroup interface {
    RouteGroup
    Channels() []string
}
```

### DescribableGroup Interface

```go
type DescribableGroup interface {
    RouteGroup
    Describe() []RouteDescription
}
```

---

## 📡 Features

### OpenAPI / Swagger

Auto-generate OpenAPI 3.1 specifications from route descriptions.

```go
engine, _ := api.New(
    api.WithSwagger("My API", "1.0.0"),
    api.WithOpenAPISpec("/openapi.json"),
    api.WithSwaggerUI("/docs"),
)
```

**Swagger Options:**
- `WithSwagger(title, version string)` — API metadata
- `WithSwaggerDescription(desc string)` — API description
- `WithSwaggerSummary(summary string)` — API summary
- `WithSwaggerTermsOfService(tos string)` — Terms of service URL
- `WithSwaggerContact(name, url, email string)` — Contact info
- `WithSwaggerLicense(name, url string)` — License info
- `WithSwaggerServers(servers []string)` — Server URLs
- `WithOpenAPISpec(path string)` — OpenAPI JSON endpoint
- `WithSwaggerUI(path string)` — Swagger UI endpoint

### SDK Generation

Generate client SDKs in 11+ languages from OpenAPI specs.

```go
engine, _ := api.New(
    api.WithSDKGen("my-api"),
    api.WithSDKGenPath("/sdk"),
)
```

**SDK Generation:**
```bash
# Generate SDK
core api sdk --output ./client

# Generate specific language
core api sdk --language python --output ./client/python
```

### WebSocket Support

Bidirectional WebSocket communication with automatic route registration.

```go
engine, _ := api.New(
    api.WithWebSocket("/ws"),
)

// Register WebSocket route group
engine.Register(&MyWebSocketGroup{})
```

### Server-Sent Events (SSE)

Real-time streaming to clients with channel-based routing.

```go
engine, _ := api.New(
    api.WithSSE("/events"),
)

// Broadcast to channel
go engine.SSEBroker().Broadcast("updates", data)
```

### Authentication

Multiple authentication methods with middleware support.

**Bearer Token:**
```go
engine, _ := api.New(
    api.WithBearerAuth("my-secret-token"),
)

// Protect routes
engine.RequireAuth()
```

**Authentik OIDC:**
```go
engine, _ := api.New(
    api.WithAuthentik(
        "https://auth.example.com",
        "client-id",
        "client-secret",
    ),
)
```

**Authorization with Casbin:**
```go
// Require specific group
api.RequireGroup("admin")

// Custom policy enforcement
api.WithAuthzPolicy("my-policy.conf")
```

### Rate Limiting

Flexible rate limiting with multiple algorithms.

```go
engine, _ := api.New(
    api.WithRateLimit("100/s"),  // 100 requests per second
    api.WithRateLimitBurst(50),    // Burst capacity
    api.WithRateLimitAlgorithm(api.RateLimitTokenBucket),
)
```

**Algorithms:**
- `RateLimitTokenBucket` — Token bucket algorithm
- `RateLimitSlidingWindow` — Sliding window algorithm

### Webhooks

Incoming and outgoing webhook support with signature verification.

```go
engine, _ := api.New(
    api.WithWebhook("/webhook", "secret"),
)

// Register webhook handler
engine.RegisterWebhook("github", myWebhookHandler)
```

### GraphQL

GraphQL endpoint with schema support.

```go
engine, _ := api.New(
    api.WithGraphQL("/graphql", schema),
)
```

### Chat Completions

AI chat endpoint with local and remote backends.

```go
engine, _ := api.New(
    api.WithChatCompletions("/chat/completions"),
    api.WithChatModel("gpt-4"),
)

// With remote backend
engine, _ := api.New(
    api.WithChatCompletionsRemote("https://api.openai.com"),
    api.WithChatAllowRemote(true),
)
```

### ToolBridge

Automatically expose MCP tools as REST endpoints.

```go
engine, _ := api.New(
    api.WithToolBridge("/tools"),
)

// MCP tools are automatically registered as POST endpoints
```

### Upstream Proxy

Reverse proxy with load balancing and failover.

```go
engine, _ := api.New(
    api.WithUpstreamRouter("/proxy"),
    api.WithUpstreamBalancer(api.UpstreamRoundRobin),
)

// Add upstream servers
engine.AddUpstream("backend1", "http://localhost:8080")
engine.AddUpstream("backend2", "http://localhost:8081")
```

---

## 🛡️ Security Features

### SSRF Guard

Prevent Server-Side Request Forgery attacks.

```go
engine, _ := api.New(
    api.WithSSRFGuard(),
    api.WithSSRFAllowedHosts("api.example.com", "trusted.service"),
)
```

### Strict Bind Mode

Enforce security constraints on server binding.

```go
engine, _ := api.New(
    api.WithStrictBind(),      // Reject non-loopback binds
    api.WithPublicBind(),      // Allow public binds
    api.WithBearerAuth("token"), // Required for public binds
)
```

### Sunset Headers

Deprecation warnings via HTTP headers.

```go
engine, _ := api.New(
    api.WithSunset("2026-12-31", "/v2"),
)
```

---

## 📦 Middleware

### Built-in Middleware

| Middleware | Purpose |
|------------|---------|
| `RateLimit()` | Rate limiting |
| `Auth()` | Bearer token authentication |
| `Authentik()` | Authentik OIDC authentication |
| `CORS()` | Cross-origin resource sharing |
| `Logging()` | Request/response logging |
| `Tracing()` | OpenTelemetry tracing |
| `SSRFGuard()` | SSRF attack prevention |
| `Brotli()` | Brotli compression |
| `Gzip()` | Gzip compression |
| `Cache()` | HTTP caching |

### Custom Middleware

```go
func MyMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Before request
        c.Next()
        // After request
    }
}

engine.Use(MyMiddleware())
```

### Route-Specific Middleware

```go
// Apply to specific routes
group := engine.Group("/admin")
group.Use(api.RequireAuth())
group.Use(api.RequireGroup("admin"))
```

---

## 🌐 Response Helpers

Standardized response formatting with metadata support.

```go
// Success response
c.JSON(api.Response{
    OK:      true,
    Data:    data,
    Message: "Success",
})

// Error response
c.JSON(api.Response{
    OK:      false,
    Error:   "Not found",
    Message: "Resource not found",
})

// With metadata
c.JSON(api.ResponseWithMeta{
    Response: api.Response{OK: true, Data: items},
    Meta: api.Meta{
        Total: totalCount,
        Page:  page,
        Limit: limit,
    },
})
```

---

## 🧪 Testing

Comprehensive test coverage with Good/Bad/Ugly pattern.

### Running Tests

```bash
cd ~/Code/core/api/go

# All tests
go test ./...

# With coverage
go test -cover ./...

# With race detector
go test -race ./...

# Specific features
go test -run TestBridge ./...
go test -run TestOpenAPI ./...
go test -run TestAuthentik ./...
```

### Test Patterns

- `_Good` — Happy path tests
- `_Bad` — Expected error cases
- `_Ugly` — Panics and edge cases

---

## 🚀 Command-Line Interface

### Build Commands

```bash
# Build binary
core build

# Build library
go build ./...
```

### API Commands

```bash
# Generate OpenAPI spec
core api spec --output openapi.json

# Generate SDK
core api sdk --output ./client

# Generate SDK for specific language
core api sdk --language python --output ./client/python
```

### Development Commands

```bash
# Run tests
core go test

# Run QA (fmt + vet + lint + test)
core go qa

# Full QA (includes race, vuln, security)
core go qa full
```

---

## 📊 Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/api` |
| **Repository** | `core/api` |
| **PHP Package** | `forge.lthn.ai/core/api` |
| **Language** | Go 1.26+ |
| **Dependencies** | `dappco.re/go`, `dappco.re/go/cli`, `github.com/gin-gonic/gin`, `github.com/casbin/casbin/v2`, `github.com/coreos/go-oidc/v3` |
| **Test Pattern** | Good/Bad/Ugly |
| **Features** | OpenAPI, Swagger, WebSocket, SSE, Auth, Rate Limit, Webhooks, GraphQL, Chat, ToolBridge, Upstream Proxy |
| **Transports** | HTTP/1.1, HTTP/2, HTTP/3 (optional), WebSocket, SSE |
| **Security** | SSRF Guard, Strict Bind, Bearer Auth, Authentik OIDC, Casbin Authz |
| **Documentation** | ✅ Complete |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-cli](../cli/) | CLI framework | ../cli/ |
| [go-mcp](../mcp/) | MCP server with ToolBridge | ../mcp/ |
| [go-process](../process/) | Process management | ../process/ |
| [go-ws](../ws/) | WebSocket hub | ../ws/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |

---

## 📝 Changelog

| Date | Change | Commit |
|------|--------|--------|
| 2026-06-17 | Complete knowledge pack documentation | N/A |
| 2026-05-XX | OpenAPI generation finalized | N/A |
| 2026-04-30 | CLAUDE.md guidance created | N/A |

---

## 🎯 Tags

```yaml
- rest
- gin
- openapi
- swagger
- websocket
- sse
- server-sent-events
- authentication
- authorization
- oidc
- bearer-token
- casbin
- rate-limiting
- webhook
- graphql
- chat
- ai
- completions
- toolbridge
- mcp
- upstream-proxy
- load-balancing
- ssrf-guard
- security
- middleware
- caching
- compression
- brotli
- gzip
- tracing
- opentelemetry
- pprof
- expvar
- production-ready
- high-coverage
```

---

## 📚 References

1. **Repository** — [~/Code/core/api/](file:///Users/snider/Code/core/api/)
2. **CLAUDE.md** — [~/Code/core/api/CLAUDE.md](file:///Users/snider/Code/core/api/CLAUDE.md)
3. **CoreGO Framework** — [CoreGO INDEX](../../INDEX.md)
4. **Gin Framework** — [github.com/gin-gonic/gin](https://github.com/gin-gonic/gin)
5. **OpenAPI Specification** — [spec.openapis.org](https://spec.openapis.org)

---

*Package documentation generated: 2026-06-17T20:00:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
