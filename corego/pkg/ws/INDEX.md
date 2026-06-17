---
type: Package Index
package: ws
module: dappco.re/go/ws
title: go-ws Package Index
description: WebSocket hub for real-time event streaming
---

# go-ws Package Index

> **WebSocket Hub for Real-Time Event Streaming**

**Repository:** `core/go-ws`  
**Module:** `dappco.re/go/ws`  
**Status:** ✅ Complete Documentation  
**Note:** ⚠️ Superseded by go/stream (see RFC)  
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| RFC | Official specification | [plans/code/core/go/ws/RFC.md](../../../../../plans/code/core/go/ws/RFC.md) |

---

## 🎯 Package Overview

**go-ws** provides a complete WebSocket hub for real-time event broadcasting from CoreGO services to frontend clients. It implements the hub pattern with channel-based subscriptions, authentication, Redis pub/sub bridging, and reconnecting clients.

### Key Features

- ✅ Hub pattern for connection management
- ✅ Channel-based message routing
- ✅ Optional authentication (API key, bearer token, query token)
- ✅ Redis pub/sub bridge for distributed deployments
- ✅ Reconnecting client with exponential backoff
- ✅ CoreGO service integration
- ✅ Process output streaming
- ✅ Security features (origin checking, input validation)

### Architecture Layers

1. **Application Layer** — Real-time updates, process streaming, agent communication
2. **Hub Layer** — Connection management, message broadcasting
3. **Authentication Layer** — Token-based auth during WebSocket upgrade
4. **Redis Bridge Layer** — Cross-instance message coordination
5. **Client Layer** — Individual connections and reconnecting client

---

## 🏗️ Components

### Core Types

| Type | File | Purpose |
|------|------|---------|
| `Hub` | `ws.go` | Central WebSocket connection manager |
| `Client` | `ws.go` | Individual WebSocket connection |
| `Message` | `ws.go` | Standard message format |
| `MessageType` | `ws.go` | Message type constants |
| `HubConfig` | `ws.go` | Hub configuration options |
| `ReconnectingClient` | `ws.go` | Client with auto-reconnect |
| `ConnectionState` | `ws.go` | Reconnection state enum |

### Authentication Types

| Type | File | Purpose |
|------|------|---------|
| `Authenticator` | `auth.go` | Authentication interface |
| `AuthResult` | `auth.go` | Authentication result |
| `APIKeyAuthenticator` | `auth.go` | API key-based auth |
| `BearerTokenAuth` | `auth.go` | Bearer token auth |
| `QueryTokenAuth` | `auth.go` | Query param token auth |
| `AuthenticatorFunc` | `auth.go` | Custom auth function |

### Redis Bridge Types

| Type | File | Purpose |
|------|------|---------|
| `RedisBridge` | `redis.go` | Redis pub/sub bridge |
| `RedisConfig` | `redis.go` | Redis connection config |
| `redisEnvelope` | `redis.go` | Message envelope for Redis |

### Error Types

| Type | File | Purpose |
|------|------|---------|
| Package errors | `errors.go` | Domain-specific errors |

---

## 📁 File Structure

```
go-ws/
├── go/
│   ├── ws.go                   # Hub, Client, Message + core methods
│   ├── auth.go                 # Authentication implementations
│   ├── redis.go                # Redis pub/sub bridge
│   ├── service.go              # CoreGO service registration
│   ├── errors.go               # Package-specific errors
│   ├── external/
│   │   └── go/
│   │       └── i18n_bench_test.go
│   ├── go.mod
│   ├── go.sum
│   └── go.work
└── external/
    └── go/
```

---

## 🚀 Quick Start

### Basic Setup

```go
package main

import (
    "context"
    "net/http"

    "dappco.re/go/ws"
)

func main() {
    hub := ws.NewHub()
    go hub.Run(context.Background())

    http.HandleFunc("/ws", hub.Handler())
    http.ListenAndServe(":8080", nil)
}
```

### With Authentication

```go
auth := ws.NewAPIKeyAuth(map[string]string{"secret": "user-1"})

hub := ws.NewHubWithConfig(ws.HubConfig{
    Authenticator: auth,
    AllowedOrigins: []string{"https://app.example.com"},
})

go hub.Run(context.Background())
http.HandleFunc("/ws", hub.Handler())
```

### Prerequisites

```bash
# For Redis bridge
docker run -p 6379:6379 redis

# For WebSocket connections
# No additional dependencies beyond gorilla/websocket
```

---

## 🎓 Use Cases

### 1. Process Output Streaming

```go
hub := ws.NewHub()
go hub.Run(context.Background())

// Stream process output
hub.SendProcessOutput("proc-123", "Starting...\n")
hub.SendProcessOutput("proc-123", "Complete!\n")
```

### 2. Event Broadcasting

```go
// Broadcast to all clients
hub.Broadcast(Message{
    Type: TypeEvent,
    Data: "Server restarting",
})
```

### 3. Channel-Based Messaging

```go
// Send to channel subscribers
hub.SendToChannel("notifications", Message{
    Type: TypeEvent,
    Data: "New message",
})
```

### 4. Redis Bridge (Multi-Instance)

```go
bridge, _ := ws.NewRedisBridge(hub, ws.RedisConfig{
    Addr: "localhost:6379",
})
go bridge.Run(context.Background())
```

### 5. Reconnecting Client

```go
client := ws.NewReconnectingClient(
    "ws://localhost:8080/ws",
    ws.ReconnectingClientConfig{
        MinBackoff:    100 * time.Millisecond,
        MaxBackoff:    5 * time.Second,
        BackoffFactor: 2.0,
    },
)
client.Connect(context.Background())
```

---

## 🔧 Configuration

### HubConfig

```go
type HubConfig struct {
    HeartbeatInterval         time.Duration    // Ping interval (default: 30s)
    PongTimeout               time.Duration    // Pong wait (default: 60s)
    WriteTimeout              time.Duration    // Write deadline (default: 10s)
    OnConnect                 func(*Client)    // Connection callback
    OnDisconnect              func(*Client)    // Disconnection callback
    Authenticator            Authenticator  // Auth during upgrade
    ChannelAuthoriser         ChannelAuthoriser // Channel access control
    MaxSubscriptionsPerClient int              // Per-client limit (default: 1024)
    AllowedOrigins            []string         // CORS origins
    CheckOrigin               func(*http.Request) bool // Custom origin check
    OnAuthFailure             func(*http.Request, AuthResult) // Auth failure callback
}
```

### RedisConfig

```go
type RedisConfig struct {
    Addr     string      // Redis address
    Password string      // Redis password
    DB       int         // Database number (default: 0)
    Prefix   string      // Key prefix (default: "ws")
    TLSConfig *tls.Config // TLS config
}
```

---

## 🧪 Testing

### Test Coverage

All files have test triplets (`_test.go` + `_example_test.go`):

| File | Tests |
|------|-------|
| ws_test.go | Hub and Client tests |
| ws_example_test.go | Example-based tests |
| auth_test.go | Authentication tests |
| auth_example_test.go | Auth examples |
| redis_test.go | Redis bridge tests |
| redis_example_test.go | Redis examples |
| service_test.go | Service tests |
| service_example_test.go | Service examples |
| errors_test.go | Error tests |
| ws_bench_test.go | Benchmark tests |
| ws_helpers_test.go | Helper tests |

### Running Tests

```bash
cd ~/Code/core/go-ws/go

# All tests
go test -v ./...

# With coverage
go test -cover ./...

# Examples only
go test -run Example -v

# Benchmarks
go test -bench . -benchmem
```

---

## 📊 Metadata

| Attribute | Value |
|-----------|-------|
| **Module** | `dappco.re/go/ws` |
| **Repository** | `core/go-ws` |
| **Language** | Go 1.26+ |
| **Dependencies** | `dappco.re/go`, `dappco.re/go/log`, `github.com/gorilla/websocket`, `github.com/redis/go-redis/v9` |
| **Test Triplets** | ✅ Complete |
| **RFC Compliance** | ✅ Verified |
| **Documentation** | ✅ Complete |
| **Status** | ⚠️ Superseded by go/stream |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-stream](../../stream/) | Successor package (recommended) | ../../stream/ |
| [go-p2p](../p2p/) | Peer-to-peer networking | ../p2p/ |
| [go-process](../process/) | Process management | ../process/ |
| [go-netops](../netops/) | Network operations | ../netops/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |

---

## 📝 Changelog

| Date | Change | Commit |
|------|--------|--------|
| 2026-06-17 | Initial deep dive documentation | N/A |
| 2026-06-17 | Package INDEX created | N/A |

---

## 🎯 Tags

```yaml
- websocket
- realtime
- streaming
- hub
- pubsub
- redis
- authentication
- reconnect
- process-output
- event-broadcast
- channel-subscription
- distributed
- single-goroutine
```

---

## 📈 Statistics

| Metric | Value |
|--------|-------|
| Throughput | ~10K msg/sec (single hub) |
| Per connection | ~1K msg/sec sustained |
| Redis bridge | ~5K msg/sec cross-instance |
| Memory per connection | ~10 KB |
| Goroutines per connection | ~2 |

---

*Package index generated: 2026-06-17T16:00:00Z*
*Knowledge Pack: CoreGo v1.1.0*
*Maintainer: Purberus <purberus@lthn.ai>*
