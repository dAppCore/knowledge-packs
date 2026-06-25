---
type: Package Index
title: go-stream Package Index
description: Complete index of go-stream package components and API surface
module: dappco.re/go/core/stream
---

# go-stream — Package Index

> **Repository:** `core/go-stream`
> **Module:** `dappco.re/go/core/stream`
> **Type:** Library
> **Status:** Production
> **Supersedes:** `dappco.re/go/ws` (go-ws)
> **Lines:** ~3,000+ (source + adapters + tests)

---

## Quick links

- **[README.md](./README.md)** — Complete package documentation
- **[RFC Specification](file:///Users/snider/Code/meowmix/plans/code/core/go/stream/RFC.md)** — Technical specification (self-contained, agent-implementable)
- **[CLAUDE.md](file:///Users/snider/Code/core/go-stream/CLAUDE.md)** — Implementation details, banned imports, conventions
- **[AGENTS.md](file:///Users/snider/Code/core/go-stream/AGENTS.md)** — Agent guidance

### Sub-Specifications

The RFC includes comprehensive sub-specifications:

- **Section 4:** Stream Interface — Core abstraction
- **Section 5:** Hub — Central broker architecture
- **Section 6:** Peer — Connection endpoint representation
- **Section 7:** HubConfig — Configuration options
- **Section 8:** Auth — Authentication interfaces and implementations
- **Section 9:** Message Envelope — WebSocket JSON format
- **Section 10:** WebSocket Adapter — Full feature specification
- **Section 11:** SSE Adapter — Server-Sent Events
- **Section 12:** Redis Bridge — Cross-instance coordination
- **Section 13:** ZeroMQ Adapter — High-throughput IPC
- **Section 14:** TCP Adapter — Raw TCP transport
- **Section 15:** Pipe — Stream composition
- **Section 16:** Stats — Monitoring and metrics
- **Section 17:** Consumer Usage Patterns — Real-world examples
- **Section 18:** Test Naming — Good/Bad/Ugly pattern

---

## File structure

### Core Package Files (`go/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `stream.go` | ~331 | Stream interface, Frame, Channel, Peer, Pipe, Envelope, ConnectionState | ✅ Complete |
| `hub.go` | ~906 | Hub type, Run(), SendToChannel, Subscribe, Publish, Broadcast, lifecycle | ✅ Complete |
| `hub_config.go` | ~100 | HubConfig, DefaultHubConfig, ChannelAuthoriser | ✅ Complete |
| `auth.go` | ~196 | Authenticator, ConnAuthenticator, AuthResult, built-in implementations | ✅ Complete |
| `message.go` | ~59 | Message, MessageType constants (go-ws compatibility) | ✅ Complete |
| `stats.go` | ~50 | HubStats type | ✅ Complete |
| `service.go` | ~202 | CoreGO service integration, action handlers | ✅ Complete |
| `errors.go` | ~20 | Sentinel errors via core.E() | ✅ Complete |

### WebSocket Adapter (`go/adapter/ws/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `ws.go` | ~300 | Adapter, Config, New(), Mount(), Handler(), ServeHTTP() | ✅ Complete |
| `reconnect.go` | ~200 | ReconnectingClient, ReconnectConfig, Connect(), Send(), State() | ✅ Complete |
| `ws_test.go` | ~500 | Comprehensive tests (Good/Bad/Ugly) | ✅ Complete |
| `ws_example_test.go` | ~100 | Usage examples as tests | ✅ Complete |
| `ws_behaviour_test.go` | ~200 | Behavioural tests, edge cases | ✅ Complete |

### SSE Adapter (`go/adapter/sse/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `sse.go` | ~250 | Adapter, Config, New(), Mount(), Handler(), HandlerForChannel() | ✅ Complete |
| `sse_test.go` | ~400 | Comprehensive tests | ✅ Complete |
| `sse_example_test.go` | ~100 | Usage examples | ✅ Complete |
| `sse_behaviour_test.go` | ~150 | Behavioural tests | ✅ Complete |

### Redis Bridge (`go/adapter/redis/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `redis.go` | ~200 | Bridge, Config, NewBridge(), Start(), Stop(), PublishToChannel(), PublishBroadcast(), SourceID() | ✅ Complete |
| `redis_test.go` | ~300 | Comprehensive tests | ✅ Complete |
| `redis_example_test.go` | ~100 | Usage examples | ✅ Complete |
| `redis_behaviour_test.go` | ~150 | Echo prevention tests | ✅ Complete |

### ZeroMQ Adapter (`go/adapter/zmq/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `zmq.go` | ~300 | Adapter, Config, Mode, Role, New(), Mount(), Start(), Publish() | ✅ Complete |
| `zmq_test.go` | ~300 | Comprehensive tests | ✅ Complete |
| `zmq_example_test.go` | ~100 | Usage examples | ✅ Complete |
| `zmq_behaviour_test.go` | ~150 | PubSub/PushPull mode tests | ✅ Complete |

### TCP Adapter (`go/adapter/tcp/`)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `tcp.go` | ~400 | Adapter, Config, New(), Mount(), Listen(), Dial(), framing | ✅ Complete |
| `reconnect.go` | ~250 | ReconnectingTCP, ReconnectConfig, Connect(), Send(), Close() | ✅ Complete |
| `tcp_test.go` | ~400 | Comprehensive tests | ✅ Complete |
| `tcp_example_test.go` | ~100 | Usage examples | ✅ Complete |
| `reconnect_test.go` | ~200 | ReconnectingTCP tests | ✅ Complete |
| `tcp_behaviour_test.go` | ~150 | Framing edge cases | ✅ Complete |
| `reconnect_example_test.go` | ~50 | Reconnect examples | ✅ Complete |

### External Package (`external/go/`)

| File | Purpose |
|------|---------|
| `external.go` | External dependencies wrapper |

### Tests Directory (`tests/`)

Integration and end-to-end tests.

### Documentation (`docs/`)

| File | Purpose |
|------|---------|
| `architecture.md` | Full architecture documentation |
| Additional docs as needed |

---

## Public API surface

### Stream Interface

The central transport-agnostic abstraction:

```go
type Stream interface {
    // Publish sends frame to all subscribers of channel
    Publish(channel string, frame []byte) core.Result

    // Subscribe registers handler for channel frames
    // Returns unsubscribe function. Safe for concurrent calls.
    Subscribe(channel string, handler func([]byte)) func()

    // Broadcast sends frame to every connected peer regardless of subscriptions
    Broadcast(frame []byte) core.Result

    // Pipe connects this stream to dst: every published frame is forwarded
    // Returns a stop function
    Pipe(destination Stream) func()

    // Stats returns a snapshot of current hub state
    Stats() HubStats
}

// Helper types
type Frame = []byte
type Channel = string
```

### Hub Type

```go
type Hub struct {
    // Internal state (unexported)
    peers             map[*Peer]bool
    broadcastQueue    chan broadcastDelivery
    publishQueue      chan publishDelivery
    register          chan *Peer
    unregister        chan *Peer
    channels          map[string]map[*Peer]bool
    channelHandlers   map[string]map[uint64]func([]byte)
    broadcastHandlers map[uint64]func([]byte)
    publishHandlers   map[uint64]func(string, []byte)
    config            HubConfig
    done              chan struct{}
    doneOnce          sync.Once
    running           bool
    mutex             sync.RWMutex
}

// Constructors
func NewHub() *Hub
func NewHubWithConfig(config HubConfig) *Hub

// Lifecycle
func (h *Hub) Run(ctx context.Context)
func (h *Hub) Running() bool
func (h *Hub) Config() HubConfig

// Channel operations
func (h *Hub) SendToChannel(channel string, frame []byte) core.Result
func (h *Hub) Publish(channel string, frame []byte) core.Result
func (h *Hub) PublishFromPeer(source *Peer, channel string, frame []byte) core.Result
func (h *Hub) PublishFromBridge(channel string, frame []byte) core.Result

// Subscription operations
func (h *Hub) Subscribe(channel string, handler func([]byte)) func()
func (h *Hub) SubscribeWithError(channel string, handler func([]byte)) core.Result
func (h *Hub) SubscribeE(channel string, handler func([]byte)) core.Result
func (h *Hub) SubscribePeer(peer *Peer, channel string) core.Result
func (h *Hub) CanSubscribePeer(peer *Peer, channel string) core.Result
func (h *Hub) UnsubscribePeer(peer *Peer, channel string)

// Broadcast operations
func (h *Hub) Broadcast(frame []byte) core.Result
func (h *Hub) BroadcastFromPeer(source *Peer, frame []byte) core.Result
func (h *Hub) BroadcastFromBridge(frame []byte) core.Result

// Peer management
func (h *Hub) AddPeer(peer *Peer) core.Result
func (h *Hub) RemovePeer(peer *Peer)
func (h *Hub) PeerCount() int
func (h *Hub) AllPeers() iter.Seq[*Peer]

// Channel management
func (h *Hub) ChannelCount() int
func (h *Hub) ChannelSubscriberCount(channel string) int
func (h *Hub) AllChannels() iter.Seq[string]

// Statistics and introspection
func (h *Hub) Stats() HubStats
func (h *Hub) SubscribePublished(handler func(string, []byte)) func()
func (h *Hub) SubscribeBroadcast(handler func([]byte)) func()

// Pipe support
func (h *Hub) Pipe(dst Stream) func()
```

### HubConfig

```go
type HubConfig struct {
    // HeartbeatInterval is the server-side ping interval for WebSocket peers
    // Defaults to 30 seconds. Ignored by SSE and TCP adapters.
    HeartbeatInterval time.Duration

    // PongTimeout is the deadline after a ping before the WS connection is closed
    // Must be greater than HeartbeatInterval. Defaults to 60 seconds.
    PongTimeout time.Duration

    // WriteTimeout is the per-write deadline for WS and TCP adapters
    // Defaults to 10 seconds.
    WriteTimeout time.Duration

    // OnConnect is called when a peer registers. Optional.
    OnConnect func(peer *Peer)

    // OnDisconnect is called when a peer unregisters. Optional.
    OnDisconnect func(peer *Peer)

    // ChannelAuthoriser optionally decides whether a peer may subscribe to a channel
    // Return true to allow. When nil, all subscriptions are allowed.
    ChannelAuthoriser func(peer *Peer, channel string) bool
}

func DefaultHubConfig() HubConfig
func normalizeHubConfig(config HubConfig) HubConfig
```

### Peer Type

```go
type Peer struct {
    // ID is a random UUID assigned on creation
    ID string

    // UserID is the authenticated user identifier. Empty when no auth is configured.
    UserID string

    // Claims holds arbitrary auth metadata (roles, tenant ID, scopes)
    Claims map[string]any

    // Transport identifies the adapter type for logging and metrics
    // Values: "ws", "sse", "tcp", "zmq"
    Transport string

    // Internal (unexported)
    send          chan []byte
    subscriptions map[string]bool
    closeHook     func()
    mutex         sync.RWMutex
    closeOnce     sync.Once
}

// Constructor
func NewPeer(transport string) *Peer

// Methods
func (peer *Peer) Subscriptions() []string
func (peer *Peer) Send(frame []byte) bool
func (peer *Peer) Close()
func (peer *Peer) SetCloseHook(closeFunc func())
func (peer *Peer) SendQueue() <-chan []byte
```

### Connection State

```go
type ConnectionState int

const (
    StateDisconnected ConnectionState = iota
    StateConnecting
    StateConnected
)

func (state ConnectionState) String() string
```

### Message Types

```go
type MessageType string

const (
    TypeProcessOutput MessageType = "process_output"
    TypeProcessStatus MessageType = "process_status"
    TypeEvent         MessageType = "event"
    TypeError         MessageType = "error"
    TypePing          MessageType = "ping"
    TypePong          MessageType = "pong"
    TypeSubscribe     MessageType = "subscribe"
    TypeUnsubscribe   MessageType = "unsubscribe"
)

func (messageType MessageType) String() string

type Message struct {
    Type      MessageType `json:"type"`
    Channel   string      `json:"channel,omitempty"`
    ProcessID string      `json:"processId,omitempty"`
    Data      any         `json:"data,omitempty"`
    Timestamp time.Time   `json:"timestamp"`
}
```

### Envelope Type

```go
type Envelope struct {
    SourceID string
    Channel  string
    Frame    []byte
}
```

### HubStats Type

```go
type HubStats struct {
    Peers           int            `json:"peers"`
    Channels        int            `json:"channels"`
    SubscriberCount map[string]int `json:"subscriber_count"`
}
```

---

## Authentication API

### HTTP Authenticator (WebSocket, SSE)

```go
type Authenticator interface {
    Authenticate(r *http.Request) AuthResult
}

type AuthResult struct {
    Valid  bool
    UserID string
    Claims map[string]any
    Error  error
}

type AuthenticatorFunc func(r *http.Request) AuthResult

func (f AuthenticatorFunc) Authenticate(r *http.Request) AuthResult

// Built-in implementations
func NewAPIKeyAuth(keys map[string]string) *APIKeyAuthenticator
func (a *APIKeyAuthenticator) Authenticate(r *http.Request) AuthResult

type BearerTokenAuth struct {
    Validate func(token string) AuthResult
}
func (b *BearerTokenAuth) Authenticate(r *http.Request) AuthResult

type QueryTokenAuth struct {
    Validate func(token string) AuthResult
}
func (q *QueryTokenAuth) Authenticate(r *http.Request) AuthResult
```

### Connection Authenticator (TCP, ZeroMQ)

```go
type ConnAuthenticator interface {
    AuthenticateConn(handshake []byte) AuthResult
}

type ConnAuthenticatorFunc func(handshake []byte) AuthResult

func (f ConnAuthenticatorFunc) AuthenticateConn(handshake []byte) AuthResult
```

---

## Adapter APIs

### WebSocket Adapter

```go
// Config
type Config struct {
    Authenticator    stream.Authenticator
    OnAuthFailure    func(r *http.Request, result stream.AuthResult)
    ReadBufferSize   int
    WriteBufferSize  int
    CheckOrigin      func(r *http.Request) bool
}

// Adapter
type Adapter struct { /* unexported */ }

func New(config Config) *Adapter
func (a *Adapter) Mount(hub *stream.Hub)
func (a *Adapter) Handler() http.HandlerFunc
func (a *Adapter) ServeHTTP(w http.ResponseWriter, r *http.Request)
func (a *Adapter) HandlerForChannel(channel string) http.HandlerFunc

// Reconnecting Client
type ReconnectConfig struct {
    URL               string
    InitialBackoff    time.Duration
    MaxBackoff        time.Duration
    BackoffMultiplier float64
    MaxRetries        int
    OnConnect         func()
    OnDisconnect      func()
    OnReconnect       func(attempt int)
    OnMessage         func(msg stream.Message)
    Dialer            *websocket.Dialer
    Headers           http.Header
}

type ReconnectingClient struct { /* unexported */ }

func NewReconnectingClient(config ReconnectConfig) *ReconnectingClient
func (rc *ReconnectingClient) Connect(ctx context.Context) error
func (rc *ReconnectingClient) Send(msg stream.Message) error
func (rc *ReconnectingClient) State() stream.ConnectionState
func (rc *ReconnectingClient) Close() error
```

### SSE Adapter

```go
type Config struct {
    Authenticator    stream.Authenticator
    HeartbeatInterval time.Duration
    RetryMs          int
}

type Adapter struct { /* unexported */ }

func New(config Config) *Adapter
func (a *Adapter) Mount(hub *stream.Hub)
func (a *Adapter) Handler() http.HandlerFunc
func (a *Adapter) HandlerForChannel(channel string) http.HandlerFunc
```

### Redis Bridge

```go
type Config struct {
    Addr      string
    Password  string
    DB        int
    Prefix    string
    TLSConfig *tls.Config
}

type Bridge struct { /* unexported */ }

func NewBridge(hub *stream.Hub, cfg Config) (*Bridge, error)
func (b *Bridge) Start(ctx context.Context) error
func (b *Bridge) Stop() error
func (b *Bridge) PublishToChannel(channel string, frame []byte) error
func (b *Bridge) PublishBroadcast(frame []byte) error
func (b *Bridge) SourceID() string
```

### ZeroMQ Adapter

```go
type Mode int

const (
    ModePubSub Mode = iota
    ModePushPull
)

type Role int

const (
    RolePublisher  Role = iota
    RoleSubscriber
    RolePusher
    RolePuller
)

type Config struct {
    Mode     Mode
    Endpoint string
    Role     Role
    Topics   []string
}

type Adapter struct { /* unexported */ }

func New(config Config) *Adapter
func (a *Adapter) Mount(hub *stream.Hub)
func (a *Adapter) Start(ctx context.Context) error
func (a *Adapter) Publish(channel string, frame []byte) error
```

### TCP Adapter

```go
const MaxFrameSize = 65535

type Config struct {
    Addr              string
    ConnAuthenticator stream.ConnAuthenticator
    HandshakeTimeout  time.Duration
    TLS               *tls.Config
}

type Adapter struct { /* unexported */ }

func New(config Config) *Adapter
func (a *Adapter) Mount(hub *stream.Hub)
func (a *Adapter) Listen(ctx context.Context) error
func (a *Adapter) Dial(ctx context.Context, hub *stream.Hub) (*stream.Peer, error)

// Reconnecting TCP
type ReconnectConfig struct {
    Addr              string
    InitialBackoff    time.Duration
    MaxBackoff        time.Duration
    BackoffMultiplier float64
    MaxRetries        int
    TLS               *tls.Config
    OnConnect         func()
    OnDisconnect      func()
    OnMessage         func(channel string, frame []byte)
}

type ReconnectingTCP struct { /* unexported */ }

func NewReconnectingTCP(config ReconnectConfig) *ReconnectingTCP
func (rc *ReconnectingTCP) Connect(ctx context.Context) error
func (rc *ReconnectingTCP) Send(channel string, frame []byte) error
func (rc *ReconnectingTCP) Close() error
```

---

## Pipe and composition

```go
// Stream interface Pipe method
func Pipe(src Stream, dst Stream) func()

// Hub implements Pipe
func (h *Hub) Pipe(dst Stream) func()

// Hub supports publish/broadcast subscription for advanced piping
func (h *Hub) SubscribePublished(handler func(string, []byte)) func()
func (h *Hub) SubscribeBroadcast(handler func([]byte)) func()
```

---

## Utility functions

```go
// Internal helpers (exported for adapter use)
func NewPeer(transport string) *Peer
func (state ConnectionState) String() string
func randomUUID() string
func encodeTCPFrame(channel string, frame []byte) []byte
func cloneFrame(frame []byte) []byte
func onceFunction(handler func()) func()
```

---

## Component catalogue

### Core Components

| Component | File | Purpose |
|-----------|------|---------|
| `Stream` interface | `stream.go` | Transport-agnostic event pipe |
| `Hub` | `hub.go` | Central channel-based broker |
| `Peer` | `stream.go` | Connected endpoint representation |
| `HubConfig` | `hub_config.go` | Hub configuration |
| `AuthResult` | `auth.go` | Authentication result |
| `Message` | `message.go` | WebSocket message envelope |
| `HubStats` | `stats.go` | Hub statistics snapshot |

### Authentication Components

| Component | File | Purpose |
|-----------|------|---------|
| `Authenticator` | `auth.go` | HTTP request authenticator interface |
| `AuthenticatorFunc` | `auth.go` | Function adapter for Authenticator |
| `APIKeyAuthenticator` | `auth.go` | API key-based authentication |
| `BearerTokenAuth` | `auth.go` | Bearer token authentication |
| `QueryTokenAuth` | `auth.go` | Query parameter authentication |
| `ConnAuthenticator` | `auth.go` | Raw connection authenticator |
| `ConnAuthenticatorFunc` | `auth.go` | Function adapter for ConnAuthenticator |

### Adapter Components

| Adapter | Package | Purpose |
|---------|---------|---------|
| WebSocket | `adapter/ws` | Bidirectional browser streaming |
| SSE | `adapter/sse` | Server-push over HTTP |
| Redis | `adapter/redis` | Cross-instance pub/sub |
| ZeroMQ | `adapter/zmq` | High-throughput IPC |
| TCP | `adapter/tcp` | Raw TCP framing |

### Service Integration

| Component | File | Purpose |
|-----------|------|---------|
| `Service` | `service.go` | CoreGO service wrapper |
| `NewService` | `service.go` | Service factory function |
| Action handlers | `service.go` | stream.publish, stream.broadcast, etc. |

---

## Transport comparison

| Feature | WebSocket | SSE | Redis | ZeroMQ | TCP |
|---------|-----------|-----|--------|--------|-----|
| Direction | Bidirectional | Server-push | Cross-instance | IPC | Bidirectional |
| Protocol | WS (HTTP upgrade) | HTTP/1.1 | Redis pub/sub | ZMQ | Raw TCP |
| Frame Format | JSON Message | text/event-stream | JSON envelope | Binary | Length-prefixed |
| Auth | HTTP header | HTTP header | None (hub-level) | Handshake | Handshake |
| TLS Support | ✅ | ✅ | ✅ (Redis) | ❌ (pure Go) | ✅ |
| Reconnect | ✅ (client) | ✅ (browser) | ✅ | ❌ | ✅ (client) |
| Max Frame Size | Configurable | Configurable | Configurable | Configurable | 65,535 |
| Use Case | Browser, interactive | Live stats, events | Cluster coordination | Daemon IPC | VPN, stratum |

---

## Dependencies

### Internal Dependencies

| Package | Purpose | Import Path |
|---------|---------|-------------|
| Core framework | Result pattern, error handling, logging | `dappco.re/go` |
| gorilla/websocket | WebSocket implementation | `github.com/gorilla/websocket` |
| go-zeromq/zmq4 | ZeroMQ implementation | `github.com/go-zeromq/zmq4` |

### External Dependencies (via Core)

All standard library usage is wrapped via CoreGO utilities:
- `fmt` → `core.Sprintf`, `core.Print`
- `log` → `core.Print`, `core.Error`
- `errors` → `core.E()`
- `strings` → `core.Contains`, `core.TrimPrefix`, etc.
- `path/filepath` → `core.JoinPath`, `core.PathBase`
- `encoding/json` → `core.JSONMarshal`, `core.JSONUnmarshal`

### Banned Imports

The following are **banned** in stream package code:

| Banned | Use Instead |
|--------|-------------|
| `fmt` | `core.Sprintf`, `core.Print` |
| `log` | `core.Print`, `core.Error` |
| `errors` | `core.E(scope, message, cause)` |
| `os` | `c.Fs()` (via Core) |
| `os/exec` | `c.Process()` (via Core) |
| `strings` | `core.Contains`, `core.TrimPrefix`, etc. |
| `path/filepath` | `core.JoinPath`, `core.PathBase` |
| `encoding/json` | `core.JSONMarshalString`, `core.JSONUnmarshalString` |

---

## Usage patterns

### Pattern 1: Basic Pub/Sub

```go
hub := stream.NewHub()
go hub.Run(ctx)

// Publisher
_ = hub.Publish("hashrate", []byte(`{"h":123456}`))

// Subscriber
unsub := hub.Subscribe("hashrate", func(frame []byte) {
    handleHashrateUpdate(frame)
})
defer unsub()
```

### Pattern 2: Multi-Transport Hub

```go
hub := stream.NewHub()
go hub.Run(ctx)

// Mount all adapters
ws.New(ws.Config{...}).Mount(hub)
sse.New(sse.Config{...}).Mount(hub)
tcp.New(tcp.Config{...}).Mount(hub)

// Publish once, deliver to all transports
hub.Publish("block", blockBytes)
```

### Pattern 3: Cross-Instance with Redis

```go
// Instance 1
hub1 := stream.NewHub()
bridge1, _ := redis.NewBridge(hub1, redis.Config{Addr: "redis:6379", Prefix: "pool"})
go bridge1.Start(ctx)

// Instance 2
hub2 := stream.NewHub()
bridge2, _ := redis.NewBridge(hub2, redis.Config{Addr: "redis:6379", Prefix: "pool"})
go bridge2.Start(ctx)

// Publish on hub1, receive on hub2 via Redis
hub1.Publish("block", data)
// → Redis pub/sub → bridge2 receives → hub2 delivers to local subscribers
```

### Pattern 4: Daemon to Browser Pipeline

```go
// Daemon → ZMQ → go-pool → WS → Browser
zmqHub := stream.NewHub()
wsHub := stream.NewHub()

zmq.New(zmq.Config{Role: zmq.RoleSubscriber}).Mount(zmqHub)
ws.New(ws.Config{}).Mount(wsHub)

// Forward daemon events to browsers
stop := stream.Pipe(zmqHub, wsHub)
defer stop()

// Daemon sends block → zmqHub → wsHub → all browser clients
```

### Pattern 5: Channel Authorization

```go
hub := stream.NewHubWithConfig(stream.HubConfig{
    ChannelAuthoriser: func(p *stream.Peer, ch string) bool {
        // Admin can subscribe to any channel
        if p.Claims["role"] == "admin" {
            return true
        }
        // Others can only subscribe to public channels
        return !strings.HasPrefix(ch, "private:")
    },
})
```

---

## Compliance summary

### Coding Standards

✅ **UK English:** colour, organisation, centre, behaviour, licence, serialise
✅ **Strict types:** All parameters and return types explicitly typed
✅ **AX comments:** Usage examples, not prose descriptions
✅ **Test triplets:** Good/Bad/Ugly pattern for all functions
✅ **Licence:** EUPL-1.2 with SPDX identifier
✅ **Error handling:** `core.E(scope, message, cause)` exclusively

### Safety Rules

✅ **No fmt imports:** Uses `core.Sprintf`, `core.Print`
✅ **No log imports:** Uses `core.Print`, `core.Error`
✅ **No errors package:** Uses `core.E()`
✅ **Panic recovery:** All handler invocations wrapped in defer/recover
✅ **Non-blocking:** Send operations are non-blocking with drop semantics
✅ **Echo prevention:** Redis bridge prevents infinite loops

### File Organization

✅ **Comments as examples:** Every exported type/function has usage comments
✅ **SPOR compliance:** Single Point Of Responsibility for each file
✅ **Test file pairing:** Every `.go` file has `_test.go` and `_example_test.go`
✅ **Build tags:** Server-only and WASM-only files properly tagged

### Test Organization

✅ **File-aware tests:** Each production file owns its test file
✅ **Table-driven subtests:** Uses `t.Run()`
✅ **Good/Bad/Ugly pattern:** Comprehensive test coverage
✅ **Race-safe:** All tests pass with `-race`
✅ **Integration tests:** Adapters tested with real connections

### Verification

```sh
GOWORK=off go mod tidy
GOWORK=off go vet ./...
GOWORK=off go test -count=1 -race ./...
gofmt -l .
```

---

## Maintenance information

- **Author:** Mistral Vibe (Purberus <purberus@lthn.ai>)
- **Created:** 2026-06-18T03:00:00Z
- **Last Updated:** 2026-06-18T03:00:00Z
- **Version:** 1.0.0
- **Licence:** EUPL-1.2
- **Repository:** `forge.lthn.sh/core/go-stream`
- **Module:** `dappco.re/go/core/stream`

### Key Contacts

- **Project Lead:** Hades (Lethean)
- **Maintainer:** Purberus <purberus@lthn.ai>
- **Supersedes:** `dappco.re/go/ws` (go-ws)
- **Used by:** Core API, go-pool, go-miner, go-p2p, Core MCP, Core Agent
- **Consumes:** Core framework
- **CI/CD:** Woodpecker.yml

### Upstream Dependencies

- **CoreGo framework:** `dappco.re/go` (mandatory)
- **gorilla/websocket:** WebSocket implementation
- **go-zeromq/zmq4:** ZeroMQ implementation (pure Go)

