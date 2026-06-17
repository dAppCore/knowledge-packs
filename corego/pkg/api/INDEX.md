---
type: Package Index
package: api
module: dappco.re/go/api
repo: core/api
title: go-api Package Index
description: REST framework with OpenAPI generation, WebSocket, SSE, and ToolBridge for CoreGO
lang: go
php-package: forge.lthn.ai/core/api
tags:
  - rest
  - gin
  - openapi
  - swagger
  - websocket
---

# go-api Package Index

> **REST Framework for the Lethean Ecosystem**

**Repository:** `core/api`  
**Module:** `dappco.re/go/api`  
**PHP Package:** `forge.lthn.ai/core/api`  
**Status:** ✅ Production-Ready  
**License:** EUPL-1.2  
**Test Pattern:** Good/Bad/Ugly  
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| CLAUDE.md | Development guidance | [~/Code/core/api/CLAUDE.md](file:///Users/snider/Code/core/api/CLAUDE.md) |
| Repository README | Original package README | [~/Code/core/api/README.md](file:///Users/snider/Code/core/api/README.md) |

---

## 🎯 Package Overview

`go-api` is the **REST framework** for CoreGO and Lethean ecosystem applications, providing a comprehensive Gin-based HTTP engine with auto-generated OpenAPI/Swagger documentation, WebSocket support, Server-Sent Events (SSE), ToolBridge for MCP integration, authentication providers, rate limiting, webhooks, GraphQL, chat completions, and upstream proxy capabilities.

### Core Capabilities

1. **REST API Server** — Full-featured HTTP server with Gin router
2. **OpenAPI 3.1 Generation** — Auto-generate specifications from code
3. **SDK Generation** — Generate client SDKs in 11+ languages
4. **Multi-Transport** — HTTP/1.1, HTTP/2, HTTP/3 (optional), WebSocket, SSE
5. **Authentication** — Authentik OIDC + static Bearer tokens
6. **Authorization** — Casbin-based policy enforcement
7. **Rate Limiting** — Token bucket and sliding window algorithms
8. **Webhooks** — Incoming/outgoing with signature verification
9. **GraphQL** — Schema support with query execution
10. **Chat Completions** — AI endpoints with local/remote backends
11. **ToolBridge** — MCP tool integration as REST endpoints
12. **Upstream Proxy** — Reverse proxy with load balancing
13. **Security** — SSRF guard, strict bind mode, deprecation headers
14. **Observability** — OpenTelemetry tracing, pprof, expvar
15. **Compression** — Brotli and Gzip response compression

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

### Component Layers

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
│  ToolBridge — MCP tools as REST endpoints                    │
│  Authentication — Authentik OIDC + Bearer tokens            │
│  Authorization — Casbin policy enforcement                  │
│  Rate Limiting — Token bucket + sliding window              │
│  Webhooks — Incoming/outgoing HTTP callbacks                 │
│  GraphQL — GraphQL endpoint with schema                     │
│  Chat — AI completions (local + remote)                     │
│  Upstream — Reverse proxy with load balancing               │
├─────────────────────────────────────────────────────────────┤
│                    Transport Layer                           │
│  HTTP/1.1, HTTP/2, HTTP/3 (optional)                          │
│  WebSocket — Bidirectional real-time                         │
│  SSE — Server-Sent Events                                    │
├─────────────────────────────────────────────────────────────┤
│                    Middleware Layer                          │
│  Auth, RateLimit, CORS, Logging, Tracing, SSRF, Sunset      │
│  Brotli, Gzip, Cache                                          │
├─────────────────────────────────────────────────────────────┤
│                    Utility Layer                             │
│  Response helpers, Cache, Transformers, JSON/i18n helpers   │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 Package Structure

```
core/api/go/
├── Core Files
│   ├── api.go                          # Engine type + New() + Serve()
│   ├── api_test.go                      # Engine tests
│   ├── api_example_test.go              # Engine usage examples
│   ├── api_describable_test.go           # Describable tests
│   ├── api_renderable_test.go           # Renderable tests
│   │
│   ├── group.go                         # RouteGroup interface
│   ├── group_test.go                    # RouteGroup tests
│   ├── group_example_test.go            # RouteGroup examples
│   │
│   ├── options.go                       # Functional options
│   ├── options_test.go                  # Options tests
│   ├── options_example_test.go          # Options usage examples
│   │
│   ├── server.go                        # HTTP server lifecycle
│   ├── server_test.go                   # Server tests
│   │
│   ├── service.go                       # Core service integration
│   ├── service_test.go                  # Service tests
│   ├── service_example_test.go          # Service usage examples
│   │
│   └── middleware.go                    # Middleware helpers
│   └── middleware_test.go               # Middleware tests
│   └── middleware_example_test.go       # Middleware usage examples
│
├── Authentication & Authorization
│   ├── authentik.go                    # Authentik OIDC
│   ├── authentik_test.go                # Authentik tests
│   ├── authentik_example_test.go        # Authentik usage examples
│   ├── authentik_integration_test.go    # Authentik integration tests
│   │
│   └── authz.go                         # Casbin authorization
│   └── authz_test.go                    # Authorization tests
│
├── Rate Limiting
│   └── ratelimit.go                     # Rate limiting
│   └── ratelimit_test.go                # Rate limit tests
│   └── ratelimit_internal_test.go       # Internal tests
│
├── Caching
│   ├── cache.go                         # HTTP caching
│   ├── cache_test.go                    # Cache tests
│   ├── cache_config.go                  # Cache configuration
│   ├── cache_config_test.go             # Config tests
│   ├── cache_config_example_test.go     # Config examples
│   ├── cache_control.go                 # Cache control headers
│   └── cache_control_test.go            # Cache control tests
│
├── OpenAPI & Swagger
│   ├── openapi.go                       # OpenAPI spec generation
│   ├── openapi_test.go                  # OpenAPI tests
│   ├── openapi_example_test.go          # OpenAPI usage examples
│   │
│   ├── swagger.go                       # Swagger UI
│   ├── swagger_test.go                  # Swagger tests
│   └── swagger_internal_test.go         # Internal Swagger tests
│   │
│   ├── export.go                        # OpenAPI export helpers
│   ├── export_test.go                   # Export tests
│   └── export_example_test.go           # Export usage examples
│   │
│   ├── spec_registry.go                 # OpenAPI spec registry
│   ├── spec_registry_test.go            # Spec registry tests
│   └── spec_registry_example_test.go    # Spec registry usage examples
│   │
│   ├── spec_builder_helper.go           # Spec builder helpers
│   └── spec_builder_helper_test.go      # Spec builder tests
│   │
│   └── openapitools.json                # OpenAPI tools config
│
├── SDK Generation
│   └── codegen.go                       # SDK code generation
│   └── codegen_test.go                  # Codegen tests
│   └── codegen_example_test.go          # Codegen usage examples
│
├── WebSocket
│   └── websocket.go                     # WebSocket support
│   └── websocket_test.go                # WebSocket tests
│   └── websocket_example_test.go        # WebSocket usage examples
│   └── websocket_internal_test.go      # Internal WebSocket tests
│
├── Server-Sent Events (SSE)
│   └── sse.go                           # SSE broker
│   └── sse_test.go                      # SSE tests
│   └── sse_example_test.go               # SSE usage examples
│   └── sse_internal_test.go             # Internal SSE tests
│
├── Webhooks
│   └── webhook.go                       # Webhook handling
│   └── webhook_test.go                  # Webhook tests
│   └── webhook_example_test.go          # Webhook usage examples
│   └── webhook_internal_test.go         # Internal webhook tests
│
├── GraphQL
│   └── graphql.go                       # GraphQL endpoint
│   └── graphql_test.go                  # GraphQL tests
│   └── graphql_example_test.go          # GraphQL usage examples
│
├── Chat Completions
│   ├── chat_completions.go              # Chat endpoint
│   ├── chat_completions_test.go         # Chat tests
│   ├── chat_completions_example_test.go # Chat usage examples
│   └── chat_completions_internal_test.go # Internal chat tests
│   │
│   ├── chat_remote.go                    # Remote chat backend
│   ├── chat_remote_test.go               # Remote chat tests
│   └── chat_remote_example_test.go       # Remote chat usage examples
│   └── chat_remote_internal_test.go      # Internal remote tests
│   │
│   ├── chat_adapter.go                  # Adapter interface
│   ├── chat_adapter_anthropic.go        # Anthropic adapter
│   └── chat_adapter_anthropic_test.go   # Anthropic tests
│   ├── chat_adapter_ollama.go           # Ollama adapter
│   └── chat_adapter_ollama_test.go      # Ollama tests
│
├── ToolBridge (MCP Integration)
│   └── bridge.go                        # ToolBridge
│   └── bridge_test.go                   # Bridge tests
│   └── bridge_example_test.go           # Bridge usage examples
│   └── bridge_internal_test.go          # Internal bridge tests
│
├── Upstream Proxy
│   ├── upstream_balancer.go              # Load balancer
│   ├── upstream_balancer_test.go         # Balancer tests
│   └── upstream_balancer_internal_test.go # Internal balancer tests
│   │
│   ├── upstream_registry.go              # Upstream registry
│   ├── upstream_registry_test.go         # Registry tests
│   └── upstream_registry_internal_test.go # Internal registry tests
│   │
│   ├── upstream_router.go                # Upstream router
│   ├── upstream_router_test.go            # Router tests
│   ├── upstream_router_example_test.go    # Router usage examples
│   └── upstream_router_internal_test.go  # Internal router tests
│   │
│   └── upstream_transport.go             # Upstream transport
│   └── upstream_transport_test.go        # Transport tests
│   └── upstream_transport_internal_test.go # Internal transport tests
│
├── Security
│   ├── ssrf_guard.go                    # SSRF prevention
│   ├── ssrf_guard_test.go               # SSRF tests
│   ├── ssrf_guard_example_test.go       # SSRF usage examples
│   └── ssrf_guard_internal_test.go      # Internal SSRF tests
│   │
│   ├── timeout.go                       # Request timeout
│   └── timeout_test.go                  # Timeout tests
│   │
│   ├── sunset.go                        # Deprecation headers
│   ├── sunset_test.go                   # Sunset tests
│   └── sunset_example_test.go           # Sunset usage examples
│
├── Compression
│   ├── brotli.go                        # Brotli compression
│   ├── brotli_test.go                   # Brotli tests
│   └── brotli_example_test.go           # Brotli usage examples
│   │
│   ├── gzip.go                          # Gzip compression
│   └── gzip_test.go                     # Gzip tests
│
├── Observability
│   ├── tracing.go                       # OpenTelemetry tracing
│   ├── tracing_test.go                  # Tracing tests
│   └── tracing_example_test.go          # Tracing usage examples
│   │
│   ├── pprof.go                         # pprof endpoint
│   └── expvar.go                        # expvar endpoint
│
├── Utilities
│   ├── response.go                      # Response helpers
│   ├── response_test.go                 # Response tests
│   ├── response_example_test.go         # Response usage examples
│   │
│   ├── response_meta.go                 # Response metadata
│   ├── response_meta_test.go            # Metadata tests
│   └── response_meta_example_test.go    # Metadata usage examples
│   │
│   ├── json_helpers.go                  # JSON utilities
│   ├── json_helpers_test.go             # JSON tests
│   └── json_helpers_example_test.go     # JSON usage examples
│   │
│   ├── i18n.go                          # i18n support
│   ├── i18n_test.go                     # i18n tests
│   └── i18n_example_test.go             # i18n usage examples
│   │
│   ├── location.go                      # Location headers
│   └── location_test.go                 # Location tests
│   │
│   ├── string_constants.go              # String constants
│   └── text_helpers.go                  # Text utilities
│
├── Transformers
│   ├── transformer.go                   # Request/response transformers
│   ├── transformer_test.go              # Transformer tests
│   └── transformer_example_test.go      # Transformer usage examples
│   │
│   ├── transformer_in.go                # Input transformers
│   ├── transformer_in_test.go           # Input transformer tests
│   └── transformer_in_example_test.go   # Input usage examples
│   │
│   ├── transformer_out.go               # Output transformers
│   └── transformer_out_test.go          # Output transformer tests
│   └── transformer_out_example_test.go # Output usage examples
│
├── Transport
│   ├── transport.go                     # HTTP transport utilities
│   ├── transport_test.go                # Transport tests
│   └── transport_example_test.go        # Transport usage examples
│   │
│   ├── transport_client.go              # HTTP client transport
│   ├── transport_client_test.go         # Client transport tests
│   └── transport_client_example_test.go # Client usage examples
│   └── transport_client_internal_test.go # Internal client tests
│
├── CLI Commands
│   └── cmd/
│       └── api/
│           ├── cmd.go                    # Main CLI command
│           ├── cmd_test.go               # CLI tests
│           └── cmd_example_test.go       # CLI usage examples
│           │
│           ├── cmd_args.go               # Argument parsing
│           ├── cmd_args_test.go          # Args tests
│           └── cmd_args_example_test.go   # Args usage examples
│           │
│           ├── cmd_spec.go               # Spec command
│           ├── cmd_spec_test.go          # Spec tests
│           └── cmd_spec_example_test.go   # Spec usage examples
│           │
│           ├── cmd_sdk.go                # SDK command
│           ├── cmd_sdk_test.go           # SDK tests
│           └── cmd_sdk_example_test.go   # SDK usage examples
│           │
│           └── core_helpers.go            # Core CLI helpers
│
├── Configuration
│   └── strict_bind_test.go              # Strict bind tests
│
├── Module Files
│   ├── go.mod
│   ├── go.sum
│   └── go.work
│
└── Symlinks
    ├── AGENTS.md -> ../AGENTS.md
    ├── CLAUDE.md -> ../CLAUDE.md
    ├── README.md -> ../README.md
    └── docs -> ../docs
```

---

## 🚀 Quick Start

### Basic HTTP Server

```go
engine, _ := api.New()
engine.Register(myRouteGroup)
engine.Serve(context.Background())
```

### With Configuration

```go
engine, _ := api.New(
    api.WithAddr(":8081"),
    api.WithCORS("*"),
    api.WithResponseMeta(),
    api.WithSwagger("My API", "1.0.0"),
)
engine.Register(myRouteGroup)
engine.Serve(context.Background())
```

### With Authentication

```go
engine, _ := api.New(
    api.WithAddr(":8081"),
    api.WithBearerAuth("secret-token"),
    api.WithAuthentik("https://auth.example.com", "client-id", "client-secret"),
)
engine.Register(authRouteGroup)
engine.Serve(context.Background())
```

### With All Features

```go
engine, _ := api.New(
    api.WithAddr(":8081"),
    api.WithCORS("*"),
    api.WithResponseMeta(),
    api.WithSwagger("My API", "1.0.0"),
    api.WithOpenAPISpec("/openapi.json"),
    api.WithSwaggerUI("/docs"),
    api.WithBearerAuth("secret"),
    api.WithRateLimit("100/s"),
    api.WithWebSocket("/ws"),
    api.WithSSE("/events"),
    api.WithPProf(),
    api.WithExpvar(),
    api.WithSSRFGuard(),
    api.WithBrotli(),
    api.WithGzip(),
)
engine.Register(
    myRouteGroup,
    myWebSocketGroup,
    myDescribableGroup,
)
engine.Serve(context.Background())
```

---

## 🔧 Core Components

### Engine

Central API server with configuration, route management, and lifecycle.

**File:** `api.go`

```go
type Engine struct {
    // Server config
    addr, http3Addr string
    http3Enabled bool
    
    // Routes
    groups []RouteGroup
    streamGroups []apistream.StreamGroup
    
    // Middleware
    middlewares []gin.HandlerFunc
    
    // Features
    chatCompletionsPath string
    sdkGenEnabled bool
    cacheTTL time.Duration
    cacheMaxEntries, cacheMaxBytes int
    wsHandler http.Handler
    wsGinHandler gin.HandlerFunc
    wsPath string
    sseBroker *SSEBroker
    ssePath string
    
    // Swagger/OpenAPI
    swaggerEnabled bool
    swaggerTitle, swaggerSummary, swaggerDesc, swaggerVersion string
    swaggerPath string
    swaggerTermsOfService string
    swaggerServers []string
    swaggerContactName, swaggerContactURL, swaggerContactEmail string
    swaggerLicenseName, swaggerLicenseURL string
    swaggerSecuritySchemes map[string]any
    swaggerExternalDocsDescription, swaggerExternalDocsURL string
    
    // Auth
    authentikConfig AuthentikConfig
    bearerToken string
    bearerConfigured, strictBind, publicBind, chatAllowRemote bool
    
    // Observability
    pprofEnabled, expvarEnabled bool
    
    // Upstream
    upstreamRouter *upstreamRouterConfig
    chatRemote *chatRemoteConfig
}
```

**Key Methods:**
- `New(opts ...Option) (*Engine, error)`
- `Serve(ctx context.Context) error`
- `Handler() http.Handler`
- `Register(groups ...RouteGroup)`
- `Use(middleware ...gin.HandlerFunc)`

### Functional Options

All configuration is done via functional options:

```go
// Server
api.WithAddr(":8080")
api.WithHTTP3(":8443")

// Features
api.WithCORS("*")
api.WithResponseMeta()
api.WithSwagger(title, version)
api.WithOpenAPISpec(path)
api.WithSwaggerUI(path)
api.WithSDKGen(name)
api.WithWebSocket(path)
api.WithSSE(path)

// Security
api.WithBearerAuth(token)
api.WithAuthentik(url, clientID, clientSecret)
api.WithRateLimit(rate)
api.WithSSRFGuard()
api.WithStrictBind()
api.WithPublicBind()

// Observability
api.WithPProf()
api.WithExpvar()
api.WithTracing()

// Compression
api.WithBrotli()
api.WithGzip()

// Cache
api.WithCacheTTL(duration)
api.WithCacheMaxEntries(n)
api.WithCacheMaxBytes(bytes)
```

---

## 📡 Features

### OpenAPI & Swagger

Auto-generate OpenAPI 3.1 specifications and serve Swagger UI.

**Options:**
- `WithSwagger(title, version)` — API metadata
- `WithSwaggerDescription(desc)` — API description
- `WithSwaggerSummary(summary)` — API summary
- `WithSwaggerContact(name, url, email)` — Contact info
- `WithSwaggerLicense(name, url)` — License info
- `WithSwaggerServers(servers)` — Server URLs
- `WithOpenAPISpec(path)` — OpenAPI JSON endpoint
- `WithSwaggerUI(path)` — Swagger UI endpoint

**Usage:**
```go
// Register describable route groups for auto-documentation
engine.Register(&MyDescribableGroup{})

// Generate spec manually
spec := engine.BuildOpenAPISpec()
```

### SDK Generation

Generate client SDKs in multiple languages from OpenAPI specs.

**CLI:**
```bash
# Generate SDK
core api sdk --output ./client

# Specific language
core api sdk --language python --output ./client/python

# With package name
core api sdk --language go --package myapi --output ./client/go
```

### WebSocket

Bidirectional WebSocket communication.

**Setup:**
```go
engine, _ := api.New(api.WithWebSocket("/ws"))

// Register WebSocket route group
engine.Register(&MyWebSocketGroup{})
```

**Broadcast:**
```go
// Access hub
wsHub := engine.WSHandler().(*ws.Hub)

// Broadcast to all
wsHub.Broadcast("message", data)

// Broadcast to room
wsHub.BroadcastToRoom("room1", "message", data)
```

### Server-Sent Events (SSE)

Real-time streaming to clients.

**Setup:**
```go
engine, _ := api.New(api.WithSSE("/events"))
```

**Broadcast:**
```go
// Access broker
broker := engine.SSEBroker()

// Broadcast to channel
broker.Broadcast("updates", data)

// Broadcast to specific client
broker.BroadcastToClient(clientID, "updates", data)
```

### Authentication

**Bearer Token:**
```go
engine, _ := api.New(api.WithBearerAuth("secret-token"))

// Protect routes with middleware
routeGroup := &MyRouteGroup{}
routeGroup.Use(api.RequireAuth())
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

// Protect routes
routeGroup.Use(api.RequireAuthentik())
```

**Authorization:**
```go
// Require specific group
routeGroup.Use(api.RequireGroup("admin"))

// Custom policy
api.WithAuthzPolicy("policy.conf")
```

### Rate Limiting

**Token Bucket (default):**
```go
engine, _ := api.New(
    api.WithRateLimit("100/s"),
    api.WithRateLimitBurst(50),
)
```

**Sliding Window:**
```go
engine, _ := api.New(
    api.WithRateLimit("100/s"),
    api.WithRateLimitAlgorithm(api.RateLimitSlidingWindow),
)
```

### Webhooks

**Incoming Webhooks:**
```go
engine, _ := api.New(api.WithWebhook("/webhook", "secret"))

// Register handler
engine.RegisterWebhook("github", func(c *gin.Context, payload []byte) {
    // Handle webhook
})
```

**Outgoing Webhooks:**
```go
// Send webhook
api.SendWebhook(ctx, url, payload, headers)
```

### GraphQL

```go
schema := graphql.MustNewSchema(&MyQuery{}, &MyMutation{})

engine, _ := api.New(
    api.WithGraphQL("/graphql", schema),
)
```

### Chat Completions

**Local Backend:**
```go
engine, _ := api.New(
    api.WithChatCompletions("/chat/completions"),
    api.WithChatModel("gpt-4"),
)
```

**Remote Backend:**
```go
engine, _ := api.New(
    api.WithChatCompletions("/chat/completions"),
    api.WithChatCompletionsRemote("https://api.openai.com"),
    api.WithChatAllowRemote(true),
)
```

**Adapters:**
- Anthropic
- Ollama
- OpenAI
- Custom adapters via `chat_adapter.go`

### ToolBridge

Automatically expose MCP tools as REST endpoints.

```go
engine, _ := api.New(
    api.WithToolBridge("/tools"),
)

// MCP tools registered via go-mcp are automatically available
// POST /tools/{tool_name}
```

### Upstream Proxy

Reverse proxy with load balancing.

**Setup:**
```go
engine, _ := api.New(
    api.WithUpstreamRouter("/proxy"),
    api.WithUpstreamBalancer(api.UpstreamRoundRobin),
)

// Add upstream servers
engine.AddUpstream("backend1", "http://localhost:8080")
engine.AddUpstream("backend2", "http://localhost:8081")
```

**Balancing Algorithms:**
- `UpstreamRoundRobin` — Round-robin
- `UpstreamRandom` — Random selection
- `UpstreamLeastConnections` — Least connections
- `UpstreamIPHash` — IP-based hashing

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

**Behavior:**
- Without `WithPublicBind()`: Only loopback addresses (127.0.0.1, ::1) allowed
- With `WithPublicBind()`: Any address allowed but requires Bearer auth
- Without Bearer auth: Public binds rejected even with `WithPublicBind()`

### Sunset Headers

Deprecation warnings via HTTP headers.

```go
engine, _ := api.New(
    api.WithSunset("2026-12-31", "/v2"),
)
```

This adds `Sunset: Sun, 31 Dec 2026 23:59:59 GMT` and `Link: </v2>; rel="successor-version"` headers.

---

## 📦 Middleware

### Built-in Middleware

| Middleware | Function | Purpose |
|------------|----------|---------|
| `RateLimit()` | `api.RateLimit()` | Rate limiting |
| `Auth()` | `api.RequireAuth()` | Bearer token auth |
| `Authentik()` | `api.RequireAuthentik()` | Authentik OIDC auth |
| `CORS()` | Built-in | Cross-origin handling |
| `Logging()` | Built-in | Request logging |
| `Tracing()` | Built-in | OpenTelemetry tracing |
| `SSRFGuard()` | Built-in | SSRF prevention |
| `Brotli()` | `api.WithBrotli()` | Brotli compression |
| `Gzip()` | `api.WithGzip()` | Gzip compression |
| `Cache()` | `api.WithCache()` | HTTP caching |

### Custom Middleware

```go
func MyMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Before
        start := time.Now()
        c.Next()
        // After
        latency := time.Since(start)
        c.Set("latency", latency)
    }
}

engine.Use(MyMiddleware())
```

### Route-Specific Middleware

```go
// Apply to route group
myGroup := &MyRouteGroup{}
myGroup.Use(
    api.RequireAuth(),
    api.RequireGroup("admin"),
)

// Apply to specific routes
engine.GET("/admin", api.RequireAuth(), api.RequireGroup("admin"), handler)
```

---

## 🌐 Response Helpers

Standardized response formatting.

### Basic Responses

```go
// Success with data
c.JSON(api.Response{
    OK:      true,
    Data:    data,
    Message: "Success",
})

// Error
c.JSON(api.Response{
    OK:      false,
    Error:   "not_found",
    Message: "Resource not found",
})
```

### With Metadata

```go
c.JSON(api.ResponseWithMeta{
    Response: api.Response{
        OK:      true,
        Data:    items,
        Message: "Items retrieved",
    },
    Meta: api.Meta{
        Total:     totalCount,
        Page:      page,
        Limit:     limit,
        TotalPages: totalPages,
    },
})
```

### Paginated Responses

```go
c.JSON(api.PaginatedResponse{
    Data: items,
    Meta: api.PaginationMeta{
        Page:         page,
        Limit:        limit,
        Total:        totalCount,
        TotalPages:   totalPages,
        HasNextPage:  hasNext,
        HasPrevPage:  hasPrev,
    },
})
```

---

## 🧪 Testing

Comprehensive test coverage following the Good/Bad/Ugly pattern.

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
go test -run TestRateLimit ./...
```

### Test Naming Convention

- `_Good` — Happy path tests
- `_Bad` — Expected error cases
- `_Ugly` — Panics and edge cases

Example:
```go
func TestEngine_Serve_Good(t *testing.T) { ... }
func TestEngine_Serve_Bad(t *testing.T) { ... }
func TestAuthentik_Login_Ugly(t *testing.T) { ... }
```

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
core api spec --format yaml --output openapi.yaml

# Generate SDK
core api sdk --output ./client
core api sdk --language python --output ./client/python
core api sdk --language go --package myapi --output ./client/go

# Full help
core api --help
core api spec --help
core api sdk --help
```

### Development Commands

```bash
# Run tests
core go test

# Run QA (fmt + vet + lint + test)
core go qa

# Full QA (includes race, vuln, security)
core go qa full

# Format code
gofmt -l .

# Lint
core go lint
```

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| **Total Go files** | ~200+ |
| **Test files** | ~150+ |
| **Example test files** | ~75+ |
| **Features** | 13+ |
| **Middleware** | 10+ |
| **Transports** | 5 (HTTP/1.1, HTTP/2, HTTP/3, WebSocket, SSE) |
| **Auth methods** | 2 (Bearer, Authentik OIDC) |
| **Rate limit algorithms** | 2 (Token Bucket, Sliding Window) |
| **Upstream balancers** | 4 (Round Robin, Random, Least Connections, IP Hash) |
| **Chat adapters** | 2+ (Anthropic, Ollama, extensible) |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-cli](../cli/) | CLI framework (used by cmd/api/) | ../cli/ |
| [go-mcp](../mcp/) | MCP server (ToolBridge integration) | ../mcp/ |
| [go-process](../process/) | Process management | ../process/ |
| [go-ws](../ws/) | WebSocket hub (used by WebSocket feature) | ../ws/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |

---

## 📈 Quality Metrics

- ✅ **OpenAPI 3.1 Compliance** — Full specification support
- ✅ **Test Coverage** — Good/Bad/Ugly pattern for all scenarios
- ✅ **Transport Flexibility** — Multiple HTTP versions and real-time protocols
- ✅ **Security** — SSRF guard, strict bind, authentication, authorization
- ✅ **Observability** — OpenTelemetry tracing, pprof, expvar
- ✅ **Documentation** — Complete README + INDEX
- ✅ **Cross-Language** — Go + PHP Laravel packages
- ✅ **Extensibility** — Middleware, route groups, custom adapters

---

## 📝 Changelog

| Date | Change | Commit |
|------|--------|--------|
| 2026-06-17 | Complete knowledge pack documentation | N/A |
| 2026-05-XX | Chat completions finalized | N/A |
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
- round-robin
- random
- least-connections
- ip-hash
- ssrf-guard
- security
- middleware
- caching
- http-cache
- compression
- brotli
- gzip
- tracing
- opentelemetry
- pprof
- expvar
- production-ready
- high-coverage
- cross-language
- go
- php
- laravel
```

---

## 📚 References

1. **Repository** — [~/Code/core/api/](file:///Users/snider/Code/core/api/)
2. **CLAUDE.md** — [~/Code/core/api/CLAUDE.md](file:///Users/snider/Code/core/api/CLAUDE.md)
3. **CoreGO Framework** — [CoreGO INDEX](../../INDEX.md)
4. **Gin Framework** — [github.com/gin-gonic/gin](https://github.com/gin-gonic/gin)
5. **OpenAPI Specification** — [spec.openapis.org/oas/v3.1](https://spec.openapis.org/oas/v3.1.html)
6. **Authentik** — [goauthentik.io](https://goauthentik.io)
7. **Casbin** — [casbin.org](https://casbin.org)

---

*Package index generated: 2026-06-17T20:00:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
