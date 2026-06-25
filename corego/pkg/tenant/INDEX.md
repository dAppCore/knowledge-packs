# go-tenant Package Index

> **Package:** `dappco.re/go/tenant`  
> **Repository:** [`github.com/dappcore/go-tenant`](https://github.com/dappcore/go-tenant)  
> **Spec:** [`plans/code/core/go/tenant/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/tenant/RFC.md)  
> **Documentation:** [README.md](./README.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>

---

## Overview

**go-tenant** provides CoreGO multi-tenancy primitives for workspace resolution, user context management, feature entitlements, and usage tracking. It follows a consumer architecture where PHP owns the database and business logic, while Go acts as a client that calls PHP's REST API for all mutations and queries, caching results locally via go-store for performance.

### Key features

| Category | Count | Description |
|----------|-------|-------------|
| **User Tiers** | 3 | Free, Apollo (paid), Hades (premium) |
| **Context Types** | 4 | User, Workspace, Feature, Scope |
| **Entitlement Checks** | 3 | Can, CanN, CanBoosted |
| **Caching Layers** | 1 | go-store integration |
| **PHP API Methods** | 5 | GetUser, GetWorkspace, ListWorkspaces, CheckFeature, TrackUsage |
| **Usage Tracking** | 2 | RecordUsage, Usage Alerts |

---

## Architecture

### Consumer Architecture

**go-tenant** follows a clean separation of concerns:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              Go Layer (go-tenant)                            │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Tenant Service (tenant.go)                        │ │
│  │  Can(), CanN(), CanBoosted(), RecordUsage(), GetUser(), GetWorkspace()│ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                    │                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐          │
│  │  User Context    │  │  Workspace      │  │   Feature       │          │
│  │  (user.go)       │  │  (workspace.go)  │  │  (feature.go)   │          │
│  │  UserFromCtx     │  │  WorkspaceFromCtx│  │  HasFeature     │          │
│  │  WithUser        │  │  WithWorkspace  │  │  FeatureFromCtx │          │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘          │
│                                    │                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐          │
│  │   Entitlement    │  │    Boost        │  │    Scope        │          │
│  │  (entitlement.go)│  │  (boost.go)     │  │  (scope.go)     │          │
│  │  Check entitlements│  │  Boosted limits │  │  Access scopes  │          │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘          │
│                                    │                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐          │
│  │    Cache         │  │  TenantClient   │  │   Package       │          │
│  │  (cache.go)      │  │  (PHP REST)     │  │  (package.go)   │          │
│  │  Local caching   │  │  API client     │  │  Package mgmt   │          │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼ (REST API Calls)
┌─────────────────────────────────────────────────────────────────────────┐
│                              PHP Layer                                    │
│  (Owns database and business logic)                                        │
│  - User management                                                          │
│  - Workspace management                                                     │
│  - Feature entitlements                                                    │
│  - Usage tracking                                                          │
│  - Tier management                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                              Database                                     │
│  (PostgreSQL/MySQL - owned and managed by PHP)                              │
└─────────────────────────────────────────────────────────────────────────┘
```

### Core Design Principles

1. **PHP Owns the Truth** — PHP maintains all database state and business logic
2. **Go is a Thin Client** — Go only calls PHP API, never modifies database directly
3. **Context-Based** — User, workspace, and feature info flows through Go's context
4. **Tier-Based Access** — Feature access controlled by user tier (Free/Apollo/Hades)
5. **Local Caching** — go-store caches PHP responses to reduce API calls
6. **AX Standard** — Every `.go` file has `_test.go` and `_example_test.go`

### Module Structure

```
core/go-tenant/
├── go/
│   ├── tenant.go                    # Main tenant service (400+ lines)
│   ├── tenant_test.go               # Unit tests
│   ├── user.go                      # User types and context (200+ lines)
│   ├── user_test.go                 # User tests
│   ├── user_example_test.go        # User usage examples
│   ├── workspace.go                # Workspace types and resolution (200+ lines)
│   ├── package.go                  # Package types (150+ lines)
│   ├── package_test.go             # Package tests
│   ├── package_example_test.go    # Package usage examples
│   ├── feature.go                   # Feature entitlements (150+ lines)
│   ├── feature_test.go             # Feature tests
│   ├── feature_example_test.go    # Feature usage examples
│   ├── entitlement.go              # Entitlement service (200+ lines)
│   ├── boost.go                     # Usage boost tracking (150+ lines)
│   ├── boost_test.go                # Boost tests
│   ├── boost_example_test.go      # Boost usage examples
│   ├── scope.go                     # Scope management (100+ lines)
│   ├── scope_test.go                # Scope tests
│   ├── cache.go                     # Local caching (150+ lines)
│   ├── cache_test.go                # Cache tests
│   ├── cache_example_test.go      # Cache usage examples
│   ├── service.go                   # CoreGo service integration (200+ lines)
│   ├── service_test.go              # Service tests
│   │
│   ├── tests/                       # Integration tests
│   ├── docs/                        # Documentation
│   ├── external/                    # External dependencies
│   ├── go.mod                       # Module: dappco.re/go/tenant
│   └── go.sum                       # Dependencies
│
├── README.md
├── AGENTS.md
├── CLAUDE.md
└── LICENCE
```

### Context Flow

```
Request → Middleware → Context Setup
    ↓
┌─────────────────────┐
│   context.Context   │ ◄── User injected via WithUser()
│                     │ ◄── Workspace injected via WithWorkspace()
│                     │ ◄── Feature injected via WithFeature()
│                     │ ◄── Scope injected via WithScope()
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   Service Methods   │ ◄── Can(), CanN(), CanBoosted() use context
│   (Tenant)          │ ◄── RecordUsage() tracks resource consumption
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   PHP API Client    │ ◄── TenantClient calls PHP REST API
│   (TenantClient)    │ ◄── Caches responses via TenantCache
└─────────────────────┘
```

---

## Subpackages

### Core Components

| Component | File | Description | Lines | Status |
|-----------|------|-------------|-------|--------|
| Tenant Service | [tenant.go](file:///Users/snider/Code/core/go-tenant/go/tenant.go) | Main service with Can/CanN/CanBoosted methods | ~400 | Production |
| User Management | [user.go](file:///Users/snider/Code/core/go-tenant/go/user.go) | User types, tier, context management | ~200 | Production |
| Workspace | [workspace.go](file:///Users/snider/Code/core/go-tenant/go/workspace.go) | Workspace types and resolution | ~200 | Production |
| Package | [package.go](file:///Users/snider/Code/core/go-tenant/go/package.go) | Package types and management | ~150 | Production |
| Feature | [feature.go](file:///Users/snider/Code/core/go-tenant/go/feature.go) | Feature definitions and checks | ~150 | Production |
| Entitlement Service | [entitlement.go](file:///Users/snider/Code/core/go-tenant/go/entitlement.go) | Central entitlement checking with caching | ~200 | Production |
| Boost | [boost.go](file:///Users/snider/Code/core/go-tenant/go/boost.go) | Boosted limits for premium users | ~150 | Production |
| Scope | [scope.go](file:///Users/snider/Code/core/go-tenant/go/scope.go) | Access scope management (user, workspace, global) | ~100 | Production |
| Cache | [cache.go](file:///Users/snider/Code/core/go-tenant/go/cache.go) | Local caching using go-store | ~150 | Production |
| CoreGo Service | [service.go](file:///Users/snider/Code/core/go-tenant/go/service.go) | CoreGo service registration and lifecycle | ~200 | Production |

### PHP API Client

**TenantClient** provides REST API access to PHP backend:

| Method | Description | Endpoint |
|--------|-------------|----------|
| `GetUser` | Get user by ID | GET /users/{id} |
| `GetWorkspace` | Get workspace by ID | GET /workspaces/{id} |
| `ListWorkspaces` | List user's workspaces | GET /users/{id}/workspaces |
| `CheckFeature` | Check feature access | GET /users/{id}/features/{code} |
| `TrackUsage` | Record resource usage | POST /workspaces/{id}/usage |

---

## Configuration

### Tenant Options

```go
type TenantOptions struct {
    APIURL   string        `json:"api_url"`    // PHP API base URL (e.g., "https://api.example.com")
    APIToken string        `json:"api_token"`  // Bearer token for authentication
    Timeout  time.Duration `json:"timeout"`    // HTTP timeout (default: 10s)
}
```

### Service Registration

```go
// Simple registration with default options
c := core.New(
    core.WithServiceLock(),
    core.WithService(tenant.Register),
)

// Custom registration with specific options
c := core.New(
    core.WithServiceLock(),
    core.WithService(func(c *core.Core) core.Result {
        return core.Ok(tenant.New(c, tenant.TenantOptions{
            APIURL:   "https://api.example.com/v1",
            APIToken: os.Getenv("TENANT_API_TOKEN"),
            Timeout:  30 * time.Second,
        }))
    }),
)

// Or use NewService factory
tenantService := tenant.NewService(tenant.TenantOptions{
    APIURL:   "https://api.example.com",
    APIToken: "token",
})
c := core.New(
    core.WithServiceLock(),
    core.WithService(tenantService),
)
```

### TenantClient Options

```go
// Create client directly
client := tenant.NewTenantClient(
    "https://api.example.com",
    "bearer-token",
    tenant.WithTimeout(30 * time.Second),
    tenant.WithUserAgent("my-app/1.0"),
)

// Available client options:
func WithTimeout(d time.Duration) TenantClientOption
func WithUserAgent(ua string) TenantClientOption
func WithDebug(debug bool) TenantClientOption
```

### User Tiers

```go
type UserTier string

const (
    TierFree   UserTier = "free"   // Free tier: 1 workspace, limited features
    TierApollo UserTier = "apollo" // Paid tier: 5 workspaces, API access
    TierHades  UserTier = "hades"  // Premium tier: unlimited, all features
)

// Tier capabilities
func (t UserTier) MaxWorkspaces() int
func (t UserTier) HasFeature(code string) bool
```

---

## Commands

CLI integration examples (pseudo-commands):

```bash
# Check feature access for user
core tenant can --user 123 --feature pages --count 1

# Check multiple features
core tenant check --user 123 --features pages,api_access,export

# List user workspaces
core tenant workspaces list --user 123

# Show workspace info
core tenant workspace get --id 456

# Show user info
core tenant user get --id 123

# Check usage
core tenant usage --workspace 456 --feature pages

# Reset cache
core tenant cache reset
```

---

## Usage patterns

### 1. Basic Feature Check

```go
// Get tenant service from Core
service, err := core.ServiceFor[*tenant.Tenant](c, "tenant")
if err != nil {
    log.Fatal(err)
}

// Check if user can access "pages" feature in workspace
workspace := tenant.WorkspaceFromCtx(ctx).Value.(*tenant.Workspace)
result := service.Can(ctx, workspace, "pages", 1)

if result.IsDenied() {
    return core.E("pages", result.Reason, nil)
}

// User has access - proceed
```

### 2. Context-Based User and Workspace

```go
// Inject user into context at middleware layer
ctx := tenant.WithUser(context.Background(), &tenant.User{
    ID:    123,
    UUID:  "user-uuid",
    Name:  "John Doe",
    Email: "john@example.com",
    Tier:  tenant.TierApollo,
})

// Inject workspace
ctx = tenant.WithWorkspace(ctx, &tenant.Workspace{
    ID:     456,
    UUID:   "workspace-uuid",
    Name:   "My Workspace",
    OwnerID: 123,
    Tier:   tenant.TierApollo,
})

// Later in the call chain, retrieve them
user := tenant.UserFromCtx(ctx).Value.(*tenant.User)
workspace := tenant.WorkspaceFromCtx(ctx).Value.(*tenant.Workspace)

// Check tier-based feature access
if user.Tier.HasFeature("api_access") {
    // Enable API access
}

// Check max workspaces for tier
if user.Tier.MaxWorkspaces() != -1 && workspaceCount >= user.Tier.MaxWorkspaces() {
    return core.E("workspace", "max workspaces for tier", nil)
}
```

### 3. Entitlement Service with Caching

```go
// Get service with entitlement checking
service := c.GetService().(*tenant.Tenant)

// First call hits PHP API and caches result
result1 := service.Can(ctx, workspace, "pages", 1)

// Second call uses cached result (no PHP API call)
result2 := service.Can(ctx, workspace, "pages", 1)

// Cache invalidation
service.Cache.Invalidate(ctx, tenant.CacheKeyUser(user.ID))
```

### 4. Usage Tracking with Alerts

```go
// Record usage
service.RecordUsage(ctx, workspace, "pages", 1)

// Check with usage count
result := service.CanN(ctx, workspace, "pages", 5)
if result.IsDenied() {
    log.Printf("Cannot create 5 pages: %s (limit: %d, used: %d)", 
        result.Reason, result.Limit, result.Used)
}

// Boosted check for premium users
result = service.CanBoosted(ctx, workspace, "pages", 10)
if result.IsAllowed() {
    // User has boosted limit
}
```

### 5. Custom Entitlement Service

```go
// Create custom entitlement service with different limits
customService := tenant.NewEntitlementService(
    cache,
    client,
    tenant.WithCustomLimits(map[string]int{
        "pages": 100,
        "api_calls": 1000,
    }),
)

// Use in tenant service
service := tenant.New(c, tenant.TenantOptions{
    APIURL:   "https://api.example.com",
    APIToken: "token",
})
service.Entitlements = customService
```

### 6. Scope-Based Access

```go
// Set scope in context
ctx := tenant.WithScope(context.Background(), tenant.ScopeWorkspace)

// Get scope later
scope := tenant.ScopeFromCtx(ctx)

switch scope {
case tenant.ScopeUser:
    // User-level operation
case tenant.ScopeWorkspace:
    // Workspace-level operation
case tenant.ScopeGlobal:
    // Global operation
}
```

### 7. Direct PHP API Access

```go
// Create client
client := tenant.NewTenantClient("https://api.example.com", "token")

// Get user directly
userResult := client.GetUser(ctx, 123)
if !userResult.OK {
    log.Printf("Failed to get user: %v", userResult.Error)
    return
}
user := userResult.Value.(*tenant.User)

// Get workspace
workspaceResult := client.GetWorkspace(ctx, 456)
if !workspaceResult.OK {
    log.Printf("Failed to get workspace: %v", workspaceResult.Error)
    return
}

// Check feature
featureResult := client.CheckFeature(ctx, 123, "api_access")
```

---

## Testing

### Test Coverage by File

| File | Functions | Lines | Coverage |
|------|-----------|-------|----------|
| `tenant.go` | 15+ | ~400 | >80% |
| `user.go` | 10+ | ~200 | >80% |
| `workspace.go` | 10+ | ~200 | >80% |
| `feature.go` | 10+ | ~150 | >80% |
| `entitlement.go` | 10+ | ~200 | >80% |
| `boost.go` | 10+ | ~150 | >80% |
| `scope.go` | 5+ | ~100 | >80% |
| `cache.go` | 10+ | ~150 | >80% |
| `package.go` | 10+ | ~150 | >80% |
| `service.go` | 10+ | ~200 | >80% |

### Test Commands

```bash
# All tests
go test ./...

# With race detector
go test -race ./...

# Specific package
go test -v ./go/tenant/...

# With coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# Verbose output
go test -v ./go/tenant/...
```

---

## API reference

### User Types

```go
// UserTier - Access tier
type UserTier string

const (
    TierFree   UserTier = "free"   // Free: 1 workspace, basic features
    TierApollo UserTier = "apollo" // Paid: 5 workspaces, API access
    TierHades  UserTier = "hades"  // Premium: unlimited, all features
)

// Tier methods
func (t UserTier) MaxWorkspaces() int
func (t UserTier) HasFeature(code string) bool

// User - Authenticated user
type User struct {
    ID              int64
    UUID            string
    Name            string
    Email           string
    Tier            UserTier
    TierExpiresAt   *time.Time
    EmailVerifiedAt *time.Time
    CreatedAt       time.Time
}

// Context helpers
func UserFromCtx(ctx context.Context) core.Result
func WithUser(ctx context.Context, user *User) context.Context
func UserIDFromCtx(ctx context.Context) core.Result
func WithUserID(ctx context.Context, userID int64) context.Context
```

### Workspace Types

```go
// Workspace - User workspace
type Workspace struct {
    ID          int64
    UUID        string
    Name        string
    OwnerID     int64
    OwnerUUID   string
    Tier        UserTier
    CreatedAt   time.Time
    UpdatedAt   time.Time
}

// Context helpers
func WorkspaceFromCtx(ctx context.Context) core.Result
func WithWorkspace(ctx context.Context, workspace *Workspace) context.Context
func WorkspaceIDFromCtx(ctx context.Context) core.Result
func WithWorkspaceID(ctx context.Context, workspaceID int64) context.Context
```

### Feature Types

```go
// Feature - Feature definition
type Feature struct {
    Code        string
    Name        string
    Description string
    Tier        UserTier // Minimum tier required
}

// Context helpers
func FeatureFromCtx(ctx context.Context, code string) core.Result
func WithFeature(ctx context.Context, feature *Feature) context.Context
```

### Scope Types

```go
// Scope - Access scope
type Scope int

const (
    ScopeUser Scope = iota
    ScopeWorkspace
    ScopeGlobal
)

func ScopeFromCtx(ctx context.Context) Scope
func WithScope(ctx context.Context, scope Scope) context.Context
```

### Entitlement Types

```go
// CanResult - Entitlement check result
type CanResult struct {
    Allowed bool
    Reason  string
    Limit   int
    Used    int
}

func (r *CanResult) IsAllowed() bool
func (r *CanResult) IsDenied() bool

// EntitlementService interface
type EntitlementService interface {
    Check(ctx context.Context, user *User, workspace *Workspace, featureCode string, count int) core.Result
}

// LocalEntitlementService implementation
func NewLocalEntitlementService(cache *TenantCache, client *TenantClient) EntitlementService
```

### Tenant Service

```go
// Tenant - Main service
type Tenant struct {
    *core.ServiceRuntime[TenantOptions]
    client       *TenantClient
    cache        *TenantCache
    entitlements EntitlementService
}

// Service registration
func Register(c *core.Core) core.Result
func New(c *core.Core, opts TenantOptions) *Tenant

// Main methods
func (t *Tenant) Can(ctx context.Context, workspace *Workspace, featureCode string, count int) core.Result
func (t *Tenant) CanN(ctx context.Context, workspace *Workspace, featureCode string, count int) core.Result
func (t *Tenant) CanBoosted(ctx context.Context, workspace *Workspace, featureCode string, count int) core.Result
func (t *Tenant) RecordUsage(ctx context.Context, workspace *Workspace, featureCode string, count int)
func (t *Tenant) GetUser(ctx context.Context, userID int64) core.Result
func (t *Tenant) GetWorkspace(ctx context.Context, workspaceID int64) core.Result
func (t *Tenant) ListWorkspaces(ctx context.Context, userID int64) core.Result
```

### TenantClient (PHP API)

```go
// TenantClient - PHP REST API client
type TenantClient struct {
    // ...
}

func NewTenantClient(apiURL, apiToken string, opts ...TenantClientOption) *TenantClient
func WithTimeout(d time.Duration) TenantClientOption
func WithUserAgent(ua string) TenantClientOption
func WithDebug(debug bool) TenantClientOption

// API methods
func (c *TenantClient) GetUser(ctx context.Context, userID int64) core.Result
func (c *TenantClient) GetWorkspace(ctx context.Context, workspaceID int64) core.Result
func (c *TenantClient) ListWorkspaces(ctx context.Context, userID int64) core.Result
func (c *TenantClient) CheckFeature(ctx context.Context, userID int64, featureCode string) core.Result
func (c *TenantClient) TrackUsage(ctx context.Context, userID, workspaceID int64, featureCode string, count int) core.Result
```

### Cache

```go
// TenantCache - Local cache using go-store
type TenantCache struct {
    // ...
}

func NewTenantCache(store any) *TenantCache
func (c *TenantCache) GetCached(ctx context.Context, key string, fn func() core.Result) core.Result
func (c *TenantCache) SetCached(ctx context.Context, key string, value any, ttl time.Duration) core.Result
func (c *TenantCache) Invalidate(ctx context.Context, key string) core.Result
func (c *TenantCache) CacheKeyUser(userID int64) string
func (c *TenantCache) CacheKeyWorkspace(workspaceID int64) string
func (c *TenantCache) CacheKeyFeature(userID int64, featureCode string) string
```

### Sentinel Errors

```go
var (
    ErrNoUserContext     = errors.New("no user in context")
    ErrNoWorkspaceContext = errors.New("no workspace in context")
    ErrFeatureDenied      = errors.New("feature denied")
    ErrUsageExceeded      = errors.New("usage exceeded")
    ErrNotFound          = errors.New("not found")
    ErrUnauthorized      = errors.New("unauthorized")
)
```

---

## Related documentation

### Internal Documentation

| Resource | Description | Location |
|----------|-------------|----------|
| RFC | Package specification and design | [plans/code/core/go/tenant/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/tenant/RFC.md) |

### External References

| Resource | URL |
|----------|-----|
| Repository | [github.com/dappcore/go-tenant](https://github.com/dappcore/go-tenant) |
| Go Module | [pkg.go.dev/dappco.re/go/tenant](https://pkg.go.dev/dappco.re/go/tenant) |
| PHP Backend | (Internal) PHP API for tenant management |
| go-store | [dappco.re/go/store](dappco.re/go/store) — Caching backend |

