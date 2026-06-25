---
type: Package Documentation
package: ws
module: dappco.re/go/ws
repo: core/go-ws
lang: go
tags:
  - websocket
  - realtime
  - streaming
  - hub
  - pubsub
  - redis
  - authentication
  - reconnect
---
# go-ws — WebSocket Hub for Real-Time Event Streaming

**RFC:** [plans/code/core/go/ws/RFC.md](../../../../../plans/code/core/go/ws/RFC.md)
**Source:** [~/Code/core/go-ws/](file:///Users/snider/Code/core/go-ws/)
**Module:** `dappco.re/go/ws`
**Dependencies:** `dappco.re/go`, `dappco.re/go/log`, `github.com/gorilla/websocket`, `github.com/redis/go-redis/v9`
**Status:** ⚠️ Superseded by go/stream (see RFC note)

---

## Overview

`go-ws` provides a **complete WebSocket hub** for real-time event broadcasting from CoreGO services to frontend clients. It implements:

- **Hub pattern** — Central broker for managing WebSocket connections
- **Channel subscriptions** — Targeted message delivery to specific channels
- **Authentication** — Optional token-based auth on WebSocket upgrade
- **Redis pub/sub bridge** — Cross-instance coordination via Redis
- **Reconnecting client** — Automatic reconnection with exponential backoff
- **CoreGO integration** — Native support for Core ACTION messages

### Primary Use Cases

1. **Process output streaming** — Real-time stdout/stderr from backend processes
2. **Event broadcasting** — Push notifications to connected clients
3. **Multi-instance coordination** — Redis bridge for distributed deployments
4. **Frontend integration** — Real-time updates for web applications
5. **Agent communication** — Bidirectional messaging for AI agents

### Design Philosophy

- **Single-goroutine hub** — All state mutations serialized through channels
- **Channel-based routing** — Clients subscribe to named channels
- **Security-first** — Optional authentication, origin checking, input validation
- **Horizontal scaling** — Redis bridge enables multi-instance deployments
- **Resilient** — Reconnecting client with exponential backoff

---

## Architecture

### Component Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                           │
│  Real-time updates, process streaming, agent communication     │
├─────────────────────────────────────────────────────────────┤
│                    Hub Layer                                   │
│  Hub — Central connection manager, message broadcaster       │
│  Client — Individual WebSocket connection                     │
│  Channel — Named message routing                              │
├─────────────────────────────────────────────────────────────┤
│                    Authentication Layer                        │
│  Authenticator — Interface for auth during upgrade             │
│  APIKeyAuth — API key-based authentication                   │
│  BearerTokenAuth — Bearer token validation                   │
│  QueryTokenAuth — Query parameter token auth                  │
├─────────────────────────────────────────────────────────────┤
│                    Redis Bridge Layer                          │
│  RedisBridge — Pub/sub bridge for cross-instance messaging    │
│  RedisConfig — Redis connection configuration                │
├─────────────────────────────────────────────────────────────┤
│                    Client Layer                               │
│  ReconnectingClient — Client with auto-reconnect              │
│  ConnectionState — Disconnected/Connecting/Connected states   │
└─────────────────────────────────────────────────────────────┘
```

### Hub Architecture

The `Hub` is the central component that:
- Manages all active WebSocket connections
- Routes messages to appropriate channels
- Handles client registration/unregistration
- Maintains channel subscription state
- Implements heartbeat/keepalive

**Single-goroutine design:** All state mutations flow through a select loop on a single goroutine, ensuring thread-safe operations without locks for the critical path.

---

## Package structure

```
go-ws/
├── go/
│   ├── ws.go                   # Hub, Client, Message types + core methods
│   ├── auth.go                 # Authentication types and implementations
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
        └── (external dependencies)
```

---

## Getting started

### Basic Hub Setup

```go
package main

import (
    "context"
    "net/http"
    "time"

    "dappco.re/go/ws"
)

func main() {
    // Create hub with default configuration
    hub := ws.NewHub()

    // Start hub in background
    go hub.Run(context.Background())

    // Register WebSocket handler
    http.HandleFunc("/ws", hub.Handler())

    // Start HTTP server
    http.ListenAndServe(":8080", nil)
}
```

### With Custom Configuration

```go
hub := ws.NewHubWithConfig(ws.HubConfig{
    HeartbeatInterval:         30 * time.Second,
    PongTimeout:               60 * time.Second,
    WriteTimeout:              10 * time.Second,
    MaxSubscriptionsPerClient: 1024,
    AllowedOrigins:            []string{"https://app.example.com"},
})

go hub.Run(context.Background())
```

### CoreGO Service Integration

```go
import (
    core "dappco.re/go"
    "dappco.re/go/ws"
)

c, r := core.New(
    core.WithService(ws.NewService(ws.ServiceConfig{
        HubConfig: ws.HubConfig{
            AllowedOrigins: []string{"https://myapp.com"},
        },
    })),
)
if !r.OK {
    log.Fatal(r.Value)
}

// Access the hub from core
svc := core.MustServiceFor[*ws.Service](c, "ws")
hub := svc.Hub
```

---

## Core types

### Message

Standard WebSocket message format.

```go
type Message struct {
    Type      MessageType `json:"type"`               // Message type
    Channel   string      `json:"channel,omitempty"`   // Target channel
    ProcessID string      `json:"processId,omitempty"` // Process identifier
    Data      any         `json:"data,omitempty"`     // Payload
    Timestamp time.Time   `json:"timestamp"`          // Server timestamp
}
```

### Message Types

```go
const (
    TypeProcessOutput   MessageType = "process_output"   // Process stdout/stderr
    TypeProcessStatus   MessageType = "process_status"   // Process lifecycle events
    TypeEvent           MessageType = "event"            // Generic events
    TypeError           MessageType = "error"            // Error messages
    TypePing            MessageType = "ping"             // Keep-alive (client → server)
    TypePong            MessageType = "pong"             // Keep-alive (server → client)
    TypeSubscribe       MessageType = "subscribe"        // Subscribe to channel
    TypeUnsubscribe     MessageType = "unsubscribe"      // Unsubscribe from channel
)
```

### HubConfig

Hub configuration options.

```go
type HubConfig struct {
    HeartbeatInterval         time.Duration     // Ping interval (default: 30s)
    PongTimeout               time.Duration     // Pong wait timeout (default: 60s)
    WriteTimeout              time.Duration     // Write deadline (default: 10s)
    OnConnect                 func(*Client)     // Connection callback
    OnDisconnect              func(*Client)     // Disconnection callback
    Authenticator            Authenticator   // Auth during upgrade
    ChannelAuthoriser         ChannelAuthoriser // Channel access control
    MaxSubscriptionsPerClient int               // Per-client sub limit (default: 1024)
    AllowedOrigins            []string          // CORS origins
    CheckOrigin               func(*http.Request) bool // Custom origin check
    OnAuthFailure             func(*http.Request, AuthResult) // Auth failure callback
}
```

### Hub

Central WebSocket connection manager.

```go
type Hub struct {
    clients             map[*Client]bool           // Active connections
    broadcast           chan []byte               // Outbound messages
    register            chan *Client               // New connections
    unregister          chan *Client               // Disconnections
    subscribeRequests   chan subscriptionRequest   // Subscribe requests
    unsubscribeRequests chan subscriptionRequest   // Unsubscribe requests
    channels            map[string]map[*Client]bool // Channel subscriptions
    config              HubConfig                 // Configuration
    done                chan struct{}              // Shutdown signal
    running             bool                       // Running state
    mu                  sync.RWMutex               // State lock
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `NewHub()` | Create hub with defaults |
| `NewHubWithConfig(config)` | Create hub with custom config |
| `Run(ctx)` | Start hub event loop |
| `Handler()` | Return HTTP handler for WebSocket upgrade |
| `Broadcast(msg)` | Send message to all connected clients |
| `SendToChannel(channel, msg)` | Send to channel subscribers |
| `SendToClient(client, msg)` | Send to specific client |
| `SendProcessOutput(processID, line)` | Stream process output |
| `SendProcessStatus(processID, status, code)` | Send process status |
| `Close()` | Shutdown hub gracefully |

### Client

Individual WebSocket connection.

```go
type Client struct {
    hub           *Hub
    conn          *websocket.Conn
    send          chan []byte
    subscriptions map[string]bool
    mu            sync.RWMutex
    sendCloseOnce sync.Once
    
    UserID string         // Authenticated user ID
    Claims map[string]any // Auth metadata
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `Send(msg)` | Send message to client |
| `Subscribe(channel)` | Subscribe to channel |
| `Unsubscribe(channel)` | Unsubscribe from channel |
| `GetSubscriptions()` | Get active subscriptions |
| `Close()` | Close connection |

### ConnectionState

Reconnection states.

```go
type ConnectionState int

const (
    StateDisconnected ConnectionState = iota
    StateConnecting
    StateConnected
)
```

### ReconnectingClient

Client with automatic reconnection.

```go
type ReconnectingClient struct {
    // Configuration
    URL            string
    Dialer         *websocket.Dialer
    RequestHeader  http.Header
    
    // State
    State          ConnectionState
    conn           *websocket.Conn
    send           chan []byte
    done           chan struct{}
    
    // Reconnection
    MinBackoff     time.Duration
    MaxBackoff     time.Duration
    BackoffFactor  float64
}
```

**Key Methods:**

| Method | Description |
|--------|-------------|
| `Connect(ctx)` | Establish connection (blocks until connected) |
| `Send(msg)` | Send message with reconnection |
| `Receive()` | Receive messages |
| `Close()` | Close connection |
| `State()` | Get current connection state |

---

## Authentication

### Authenticator Interface

```go
type Authenticator interface {
    Authenticate(r *http.Request) AuthResult
}
```

### Built-in Authenticators

#### APIKeyAuthenticator

```go
// Create API key auth
auth := ws.NewAPIKeyAuth(map[string]string{
    "secret-key-1": "user-1",
    "secret-key-2": "user-2",
})

// With custom claims
auth := ws.NewAPIKeyAuthWithClaims(map[string]string{
    "secret-key": "user-1",
}, map[string]map[string]any{
    "user-1": {"role": "admin", "scopes": []string{"read", "write"}},
})
```

#### BearerTokenAuth

```go
// Validate bearer tokens
auth := ws.NewBearerTokenAuth(func(token string) (string, map[string]any, bool) {
    // Your token validation logic
    claims, err := jwt.Parse(token, keyFunc)
    if err != nil {
        return "", nil, false
    }
    return claims.Subject, claims.Map(), true
})
```

#### QueryTokenAuth

```go
// Validate tokens from query parameters
auth := ws.NewQueryTokenAuth(func(token string) (string, map[string]any, bool) {
    // Validate token from ?token=xxx
    return validateToken(token)
})
```

#### AuthenticatorFunc

```go
// Custom authentication function
auth := ws.AuthenticatorFunc(func(r *http.Request) ws.AuthResult {
    // Your custom auth logic
    token := r.Header.Get("X-API-Key")
    if token != "secret" {
        return ws.AuthResult{Valid: false, Error: errors.New("invalid token")}
    }
    return ws.AuthResult{
        Valid: true,
        UserID: "user-1",
        Claims: map[string]any{"role": "admin"},
    }
})
```

### AuthResult

```go
type AuthResult struct {
    Valid         bool           // Authentication succeeded
    Authenticated bool           // RFC-compatible alias for Valid
    UserID        string        // Authenticated user identifier
    Claims        map[string]any // Auth metadata
    Error         error         // Failure reason
}
```

---

## Redis pub/sub bridge

Cross-instance message coordination via Redis.

### RedisBridge

```go
type RedisBridge struct {
    hub     *Hub
    client  *redis.Client
    config  RedisConfig
    sourceID string // Unique identifier to prevent echo loops
}
```

### RedisConfig

```go
type RedisConfig struct {
    Addr     string      // Redis server address (e.g., "localhost:6379")
    Password string      // Redis password
    DB       int         // Database number (default: 0)
    Prefix   string      // Key prefix (default: "ws")
    TLSConfig *tls.Config // TLS configuration
}
```

### Usage

```go
// Create Redis bridge
bridge, r := ws.NewRedisBridge(hub, ws.RedisConfig{
    Addr:     "localhost:6379",
    Password: "secret",
    Prefix:   "myapp-ws",
})
if !r.OK {
    log.Fatal(r.Value)
}

// Start bridge
go bridge.Run(context.Background())

// Bridge will automatically:
// - Forward local messages to Redis
// - Receive Redis messages and forward to local hub
// - Prevent echo loops using sourceID
```

### Redis Channel Pattern

The bridge uses these Redis channels:
- `{prefix}:broadcast` — All messages
- `{prefix}:channel:{name}` — Channel-specific messages

---

## Channel subscriptions

### Subscribing from Client

Clients send JSON messages to subscribe:

```json
{"type": "subscribe", "channel": "process:proc-123"}
{"type": "subscribe", "channel": "events:user-1"}
```

### Server-Side Subscription Management

```go
// Subscribe a client to a channel
hub.Subscribe(client, "process:proc-123")

// Unsubscribe from a channel
hub.Unsubscribe(client, "process:proc-123")

// Send to all subscribers of a channel
hub.SendToChannel("process:proc-123", Message{
    Type:      TypeProcessOutput,
    ProcessID: "proc-123",
    Data:      "Process output line",
})
```

### Channel Authorization

```go
// Custom channel authorizer
hub := ws.NewHubWithConfig(ws.HubConfig{
    ChannelAuthoriser: func(client *ws.Client, channel string) bool {
        // Only allow users to subscribe to their own process channels
        if strings.HasPrefix(channel, "process:") {
            processID := strings.TrimPrefix(channel, "process:")
            return client.UserID == getProcessOwner(processID)
        }
        return true
    },
})
```

---

## Process integration

### Streaming Process Output

```go
// Send process output to channel subscribers
hub.SendProcessOutput("proc-123", "Starting process...\n")
hub.SendProcessOutput("proc-123", "Step 1 complete\n")

// Send process status
hub.SendProcessStatus("proc-123", "running", 0)
hub.SendProcessStatus("proc-123", "exited", 0)
```

### CoreGO ACTION Integration

```go
// Register action handler
core.RegisterAction(func(c *core.Core, msg core.Message) error {
    switch m := msg.(type) {
    case process.ActionProcessOutput:
        // Get hub from core
        svc := core.MustServiceFor[*ws.Service](c, "ws")
        svc.Hub.SendProcessOutput(m.ID, m.Line)
    case process.ActionProcessExited:
        svc.Hub.SendProcessStatus(m.ID, "exited", m.ExitCode)
    }
    return nil
})
```

---

## Reconnecting client

Client-side WebSocket connection with auto-reconnect.

### Usage

```go
// Create reconnecting client
client := ws.NewReconnectingClient(
    "ws://localhost:8080/ws",
    ws.ReconnectingClientConfig{
        Dialer: &websocket.Dialer{
            TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
        },
        MinBackoff:    100 * time.Millisecond,
        MaxBackoff:    30 * time.Second,
        BackoffFactor: 2.0,
        RequestHeader: http.Header{
            "Authorization": []string{"Bearer token"},
        },
    },
)

// Connect (blocks until connected)
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()

if err := client.Connect(ctx); err != nil {
    log.Fatal(err)
}

// Send messages
if err := client.Send(Message{
    Type:    TypeSubscribe,
    Channel: "process:proc-123",
}); err != nil {
    log.Fatal(err)
}

// Receive messages
for msg := range client.Receive() {
    fmt.Printf("Received: %+v\n", msg)
}
```

### Reconnection Behavior

- **Exponential backoff** — Wait time doubles after each failure
- **Jitter** — Random variation to prevent thundering herd
- **Max backoff** — Capped at MaxBackoff duration
- **State tracking** — ConnectionState indicates current state

---

## Security features

### Origin Checking

```go
// Allow specific origins only
hub := ws.NewHubWithConfig(ws.HubConfig{
    AllowedOrigins: []string{
        "https://app.example.com",
        "https://staging.example.com",
    },
})

// Custom origin check
hub := ws.NewHubWithConfig(ws.HubConfig{
    CheckOrigin: func(r *http.Request) bool {
        // Your custom logic
        return strings.HasSuffix(r.Header.Get("Origin"), ".example.com")
    },
})
```

### Message Validation

- **Channel name validation** — Max length 256 chars
- **Process ID validation** — Max length 128 chars
- **Message size limits** — Default 64 KB max
- **Input sanitization** — All inputs trimmed and validated

### Rate Limiting

```go
// Custom rate limiting via channel authorizer
hub := ws.NewHubWithConfig(ws.HubConfig{
    ChannelAuthoriser: func(client *ws.Client, channel string) bool {
        // Implement rate limiting logic
        if rateLimiter.IsOverLimit(client.UserID) {
            return false
        }
        return true
    },
})
```

---

## Testing

### Test Triplets

Each file has `_test.go` + `_example_test.go`:

```
go/
├── ws_test.go                  # Hub and Client tests
├── ws_example_test.go         # Example-based tests
├── auth_test.go                # Authentication tests
├── auth_example_test.go        # Auth examples
├── redis_test.go               # Redis bridge tests
├── redis_example_test.go      # Redis examples
├── service_test.go             # Service tests
├── service_example_test.go     # Service examples
├── errors_test.go              # Error tests
├── ws_bench_test.go            # Benchmark tests
└── ws_helpers_test.go          # Helper tests
```

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

## Performance considerations

### Throughput

- Single hub: ~10K messages/sec (benchmark)
- Per connection: ~1K messages/sec sustained
- Redis bridge: ~5K messages/sec cross-instance

### Memory Usage

- Base hub: ~1 MB + per-connection overhead
- Per connection: ~10 KB (buffers, channels)
- Per channel subscription: ~100 bytes

### Scalability

- **Vertical:** Single hub handles ~10K concurrent connections
- **Horizontal:** Redis bridge enables multi-instance scaling
- **Connection overhead:** ~2 goroutines per connection

---

## Best practices

### 1. Always Configure Allowed Origins

```go
// Production configuration
hub := ws.NewHubWithConfig(ws.HubConfig{
    AllowedOrigins: []string{"https://yourdomain.com"},
})
```

### 2. Use Authentication

```go
// Always require authentication in production
auth := ws.NewBearerTokenAuth(jwtValidator)
hub := ws.NewHubWithConfig(ws.HubConfig{
    Authenticator: auth,
    OnAuthFailure: func(r *http.Request, result ws.AuthResult) {
        log.Warn("auth failure", "error", result.Error)
    },
})
```

### 3. Implement Channel Authorization

```go
hub := ws.NewHubWithConfig(ws.HubConfig{
    ChannelAuthoriser: func(client *ws.Client, channel string) bool {
        // Prevent unauthorized channel access
        return isAuthorized(client.UserID, channel)
    },
})
```

### 4. Limit Subscriptions Per Client

```go
hub := ws.NewHubWithConfig(ws.HubConfig{
    MaxSubscriptionsPerClient: 100, // Prevent resource exhaustion
})
```

### 5. Use Redis Bridge for Multi-Instance

```go
// For distributed deployments
bridge, _ := ws.NewRedisBridge(hub, ws.RedisConfig{
    Addr: "redis-cluster:6379",
    Prefix: "myapp",
})
go bridge.Run(ctx)
```

### 6. Graceful Shutdown

```go
// Handle graceful shutdown
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

go hub.Run(ctx)

// On shutdown
cancel()
hub.Close()
```

### 7. Monitor Connection Health

```go
hub := ws.NewHubWithConfig(ws.HubConfig{
    OnConnect: func(client *ws.Client) {
        log.Info("client connected", "user", client.UserID)
        metrics.Inc("ws.connections")
    },
    OnDisconnect: func(client *ws.Client) {
        log.Info("client disconnected", "user", client.UserID)
        metrics.Dec("ws.connections")
    },
})
```

---

## Examples

### Example 1: Basic WebSocket Server

```go
func ExampleHub_basic() {
    hub := ws.NewHub()
    go hub.Run(context.Background())

    http.HandleFunc("/ws", hub.Handler())
    http.ListenAndServe(":8080", nil)
}
```

### Example 2: Authenticated Hub

```go
func ExampleHub_authenticated() {
    auth := ws.NewAPIKeyAuth(map[string]string{
        "secret": "user-1",
    })

    hub := ws.NewHubWithConfig(ws.HubConfig{
        Authenticator: auth,
        AllowedOrigins: []string{"https://app.example.com"},
    })

    go hub.Run(context.Background())
    http.HandleFunc("/ws", hub.Handler())
}
```

### Example 3: Process Output Streaming

```go
func ExampleHub_processOutput() {
    hub := ws.NewHub()
    go hub.Run(context.Background())

    // Simulate process output
    go func() {
        for i := 0; i < 10; i++ {
            hub.SendProcessOutput("proc-1", fmt.Sprintf("Line %d\n", i))
            time.Sleep(100 * time.Millisecond)
        }
        hub.SendProcessStatus("proc-1", "exited", 0)
    }()

    http.HandleFunc("/ws", hub.Handler())
}
```

### Example 4: Channel-Based Messaging

```go
func ExampleHub_channels() {
    hub := ws.NewHub()
    go hub.Run(context.Background())

    // Send to specific channel
    go func() {
        hub.SendToChannel("notifications", Message{
            Type: TypeEvent,
            Data: "Hello, subscribers!",
        })
    }()

    http.HandleFunc("/ws", hub.Handler())
}
```

### Example 5: Redis Bridge

```go
func ExampleRedisBridge() {
    hub := ws.NewHub()
    go hub.Run(context.Background())

    bridge, _ := ws.NewRedisBridge(hub, ws.RedisConfig{
        Addr: "localhost:6379",
    })
    go bridge.Run(context.Background())

    http.HandleFunc("/ws", hub.Handler())
}
```

### Example 6: Reconnecting Client

```go
func ExampleReconnectingClient() {
    client := ws.NewReconnectingClient(
        "ws://localhost:8080/ws",
        ws.ReconnectingClientConfig{
            MinBackoff:    100 * time.Millisecond,
            MaxBackoff:    5 * time.Second,
            BackoffFactor: 2.0,
        },
    )

    ctx := context.Background()
    if err := client.Connect(ctx); err != nil {
        log.Fatal(err)
    }

    // Send and receive
    client.Send(Message{Type: TypeSubscribe, Channel: "events"})
    for msg := range client.Receive() {
        fmt.Println(msg)
    }
}
```

---

## Debugging

### Enable Debug Logging

```go
import "dappco.re/go/log"

log.SetLevel(log.LevelDebug)

hub := ws.NewHub()
go hub.Run(context.Background())
```

### Common Issues

| Issue | Solution |
|-------|----------|
| Connection refused | Check WebSocket server is running |
| Origin not allowed | Configure AllowedOrigins |
| Auth failed | Verify Authenticator configuration |
| Message not received | Check channel subscriptions |
| Redis connection error | Verify Redis is accessible |
| High memory usage | Limit MaxSubscriptionsPerClient |

### Debugging Connections

```go
hub := ws.NewHubWithConfig(ws.HubConfig{
    OnConnect: func(client *ws.Client) {
        log.Info("client connected",
            "user", client.UserID,
            "remote", client.conn.RemoteAddr(),
        )
    },
    OnDisconnect: func(client *ws.Client) {
        log.Info("client disconnected", "user", client.UserID)
    },
})
```

---

## Metrics and monitoring

### Built-in Callbacks

```go
hub := ws.NewHubWithConfig(ws.HubConfig{
    OnConnect: func(client *ws.Client) {
        metrics.Counter("ws.connections.total").Inc()
        metrics.Gauge("ws.connections.active").Inc()
    },
    OnDisconnect: func(client *ws.Client) {
        metrics.Gauge("ws.connections.active").Dec()
    },
    OnAuthFailure: func(r *http.Request, result ws.AuthResult) {
        metrics.Counter("ws.auth.failures").Inc()
    },
})
```

### Connection Statistics

```go
// Get hub statistics
totalClients := len(hub.GetClients())
totalSubscriptions := hub.GetTotalSubscriptions()
```

---

## Notes

- **Repository:** `forge.lthn.sh/core/go-ws`
- **Primary Spec:** [RFC.md](../../../../../plans/code/core/go/ws/RFC.md)
- **Status:** ⚠️ Superseded by go/stream (see RFC)
- **Dependencies:** gorilla/websocket, go-redis/v9
- **Compatibility:** Go 1.26+
- **Security:** Authentication, origin checking, input validation

---

## Related packages

| Package | Relationship | Path |
|---------|--------------|------|
| [go-stream](../../stream/) | Successor package (recommended) | ../../stream/ |
| [go-p2p](../p2p/) | Peer-to-peer networking | ../p2p/ |
| [go-process](../process/) | Process management | ../process/ |
| [go-netops](../netops/) | Network operations | ../netops/ |
| [CoreGO INDEX](../../INDEX.md) | Package catalog | ../../INDEX.md |

