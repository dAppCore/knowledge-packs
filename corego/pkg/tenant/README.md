# go-tenant — Multi-Tenancy Service

> **Package:** `dappco.re/go/tenant`  
> **Repository:** [`github.com/dappcore/go-tenant`](https://github.com/dappcore/go-tenant)  
> **Spec:** [`plans/code/core/go/tenant/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/tenant/RFC.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>  
> **Module:** `dappco.re/go/tenant`

---

## Overview

**go-tenant** provides CoreGO tenant service primitives for multi-tenancy support. It handles workspace resolution, user context management, feature entitlements, usage tracking, and PHP tenant API integration. The Go module acts as a consumer — PHP owns the database and business logic. Go calls PHP's REST API for all mutations and queries, then caches the results locally via go-store.

### Key capabilities

| Category | Features | Description |
|----------|----------|-------------|
| **Workspace Resolution** | Multi-tenant workspace management | Resolve workspaces from context |
| **User Context** | Authenticated user management | Inject and retrieve users from context |
| **Feature Entitlements** | Tier-based access control | Check if user has access to features |
| **Usage Tracking** | Resource consumption monitoring | Track and alert on usage |
| **PHP API Integration** | REST client for PHP backend | All mutations go through PHP API |
| **Local Caching** | go-store integration | Cache PHP responses locally |
| **CoreGo Service** | Full service integration | Works as a CoreGo service |

### Architecture

### Design Philosophy

```
┌─────────────────────────────────────────────────────────────────┐
│                        Go Application (go-tenant)                │
│  ┌─────────────────┐    ┌─────────────────┐    ┌────────────┐ │
│  │  Tenant Service │    │  User Context   │    │  Workspace  │ │
│  │  - Can()        │    │  - UserFromCtx  │    │  - Resolve  │ │
│  │  - CanN()       │    │  - WithUser     │    │  - FromCtx  │ │
│  │  - CanBoosted() │    │  - Tier access  │    │  - Validate │ │
│  │  - Usage alerts │    │  - Feature checks│   │             │ │
│  └────────┬────────┘    └────────┬────────┘    └──────┬──────┘ │
└───────────┼──────────────────────┼───────────────────────┼─────────┘
            │                      │                       │
            ▼                      ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                        PHP REST API                              │
│  (Owns the database and business logic)                          │
│  - User management                                                   │
│  - Workspace management                                              │
│  - Feature entitlements                                             │
│  - Usage tracking                                                   │
└─────────────────────────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Database                                     │
│  (PostgreSQL/MySQL - owned by PHP)                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Core Design Principles

1. **PHP Owns the Truth** — PHP maintains the database and business logic
2. **Go is a Consumer** — Go calls PHP API, doesn't modify database directly
3. **Local Caching** — go-store caches PHP responses for performance
4. **Context-Based** — User and workspace info flows through context
5. **Tier-Based Access** — Feature access controlled by user tier
6. **AX Standard** — Each `.go` file has `_test.go` and `_example_test.go`

### Module Structure

```
core/go-tenant/
├── go/                          # Go module root (dappco.re/go/tenant)
│   ├── tenant.go                # Main tenant service
│   ├── tenant_test.go           # Unit tests
│   ├── user.go                  # User types and context
│   ├── user_test.go             # User tests
│   ├── user_example_test.go    # User usage examples
│   ├── workspace.go            # Workspace types and resolution
│   ├── package.go              # Package types
│   ├── package_example_test.go # Package usage examples
│   ├── feature.go               # Feature entitlements
│   ├── feature_example_test.go # Feature usage examples
│   ├── entitlement.go           # Entitlement service
│   ├── boost.go                 # Usage boost tracking
│   ├── boost_test.go            # Boost tests
│   ├── boost_example_test.go  # Boost usage examples
│   ├── scope.go                 # Scope management
│   ├── scope_test.go            # Scope tests
│   ├── cache.go                 # Local caching with go-store
│   ├── cache_test.go            # Cache tests
│   ├── cache_example_test.go  # Cache usage examples
│   ├── service.go               # CoreGo service integration
│   ├── service_test.go          # Service tests
│   │
│   ├── cmd/                     # CLI commands (if any)
│   ├── tests/                   # Integration tests
│   ├── docs/                    # Documentation
│   ├── external/                # External dependencies
│   ├── go.mod                   # Module definition
│   └── go.sum                   # Dependency checksums
│
├── README.md
├── AGENTS.md
├── CLAUDE.md
└── LICENCE
```

---

## Packages

### Core Components

#### Tenant Service (`tenant.go`)

The main service providing:
- **`Can`** — Check if user can perform action on feature
- **`CanN`** — Check if user can perform N actions on feature
- **`CanBoosted`** — Check with boosted limits
- **Usage Alerts** — Track and alert on resource usage

#### User Management (`user.go`)

User types and context handling:
- **`User`** — Authenticated user with tier information
- **`UserTier`** — Free, Apollo, Hades tiers
- **`UserFromCtx`** — Retrieve user from context
- **`WithUser`** — Inject user into context

#### Workspace Resolution (`workspace.go`)

Workspace management:
- **`Workspace`** — Workspace type and metadata
- **`FromCtx`** — Get workspace from context
- **`Resolve`** — Resolve workspace from ID or path
- **`Validate`** — Validate workspace access

#### Feature Entitlements (`feature.go`)

Feature-based access control:
- **`Feature`** — Feature definition
- **`HasFeature`** — Check if user has feature access
- **`FeatureFromCtx`** — Get feature from context

#### Entitlement Service (`entitlement.go`)

Central entitlement checking:
- **`EntitlementService`** — Service interface
- **`LocalEntitlementService`** — Local implementation with caching
- **`Check`** — Check entitlement with caching

#### Usage Boost (`boost.go`)

Boosted limits for premium users:
- **`Boost`** — Boost configuration
- **`ApplyBoost`** — Apply boost to limits
- **`BoostedLimit`** — Calculate boosted limit

#### Scope Management (`scope.go`)

Scope-based access:
- **`Scope`** — Access scope (user, workspace, global)
- **`WithScope`** — Set scope in context
- **`ScopeFromCtx`** — Get scope from context

#### Local Caching (`cache.go`)

Caching layer using go-store:
- **`TenantCache`** — Cache for tenant data
- **`CacheKey`** — Generate cache keys
- **`GetCached`** — Get from cache
- **`SetCached`** — Set in cache
- **`Invalidate`** — Invalidate cache entries

### PHP API Integration

**TenantClient** — REST client for PHP backend:
- **`GetUser`** — Get user by ID
- **`GetWorkspace`** — Get workspace by ID
- **`ListWorkspaces`** — List user workspaces
- **`CheckFeature`** — Check feature access
- **`TrackUsage`** — Record resource usage

---

## Configuration

### Service Options

```go
type TenantOptions struct {
    APIURL   string        `json:"api_url"`    // PHP API base URL
    APIToken string        `json:"api_token"`  // Bearer token for authentication
    Timeout  time.Duration `json:"timeout"`    // HTTP timeout (default: 10s)
}

// Default options
opts := tenant.TenantOptions{
    APIURL:   "https://api.example.com",
    APIToken: "your-bearer-token",
    Timeout:  10 * time.Second,
}
```

### Service Registration

```go
// Register tenant service with Core
c := core.New(
    core.WithServiceLock(),
    core.WithService(tenant.Register),
)

// Or with custom options
c := core.New(
    core.WithServiceLock(),
    core.WithService(func(c *core.Core) core.Result {
        return core.Ok(tenant.New(c, tenant.TenantOptions{
            APIURL:   "https://api.example.com",
            APIToken: "token",
            Timeout:  10 * time.Second,
        }))
    }),
)
```

### Context Setup

```go
// Inject user into context
ctx := tenant.WithUser(context.Background(), &tenant.User{
    ID:    123,
    UUID:  "abc-123",
    Name:  "John Doe",
    Email: "john@example.com",
    Tier:  tenant.TierApollo,
})

// Retrieve user from context
userResult := tenant.UserFromCtx(ctx)
if !userResult.OK {
    log.Println("No user in context")
    return
}
user := userResult.Value.(*tenant.User)
```

---

## Commands

While primarily a library, go-tenant can be integrated into CLI tools:

```bash
# Pseudo-commands

# Check feature access
core tenant can --user 123 --feature api_access

# Check with count
core tenant can --user 123 --feature pages --count 5

# List user workspaces
core tenant workspaces --user 123

# Show user info
core tenant user --id 123
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

// Check if user can access a feature
result := service.Can(ctx, workspace, "pages", 1)
if result.IsDenied() {
    return core.E("pages", result.Reason, nil)
}

// User can access the feature
```

### 2. User Context Management

```go
// Inject user into context
ctx := tenant.WithUser(context.Background(), &tenant.User{
    ID:    123,
    Name:  "John Doe",
    Email: "john@example.com",
    Tier:  tenant.TierHades, // Premium tier
})

// Retrieve user anywhere in the call chain
userResult := tenant.UserFromCtx(ctx)
if !userResult.OK {
    return userResult
}
user := userResult.Value.(*tenant.User)

// Check user tier
if user.Tier == tenant.TierHades {
    // Hades users have unlimited access
}

// Check feature access directly on tier
if user.Tier.HasFeature("api_access") {
    // User has API access
}
```

### 3. Workspace Resolution

```go
// Resolve workspace from context
workspaceResult := tenant.WorkspaceFromCtx(ctx)
if !workspaceResult.OK {
    return workspaceResult
}
workspace := workspaceResult.Value.(*tenant.Workspace)

// Use workspace in feature checks
result := service.Can(ctx, workspace, "pages", 1)
```

### 4. Batch Feature Checks

```go
// Check multiple features at once
features := []string{"pages", "api_access", "export"}
for _, feature := range features {
    result := service.Can(ctx, workspace, feature, 1)
    if result.IsDenied() {
        log.Printf("Denied: %s - %s", feature, result.Reason)
    }
}
```

### 5. Usage Tracking

```go
// Track resource usage
service.RecordUsage(ctx, workspace, "pages", 1)

// Check usage with limits
result := service.CanN(ctx, workspace, "pages", 5)
if result.IsDenied() {
    log.Printf("Cannot create 5 pages: %s", result.Reason)
}
```

### 6. Tier-Based Feature Access

```go
// Check tier capabilities
user := tenant.UserFromCtx(ctx).Value.(*tenant.User)

// Max workspaces for tier
maxWorkspaces := user.Tier.MaxWorkspaces()
if maxWorkspaces != -1 && workspaceCount >= maxWorkspaces {
    return core.E("workspace", "max workspaces exceeded", nil)
}

// Feature access check
if !user.Tier.HasFeature("api_access") {
    return core.E("api", "feature not available for your tier", nil)
}
```

### 7. Custom Entitlement Service

```go
// Create custom entitlement service
customService := tenant.NewEntitlementService(
    cache,
    client,
    tenant.WithCustomLimits(map[string]int{"pages": 100}),
)

// Use in tenant service
service := tenant.New(c, tenant.TenantOptions{
    APIURL:   "https://api.example.com",
    APIToken: "token",
})
service.Entitlements = customService
```

---

## Testing

### Test Structure

Each file follows the AX standard:
- `_test.go` — Unit tests with mocked PHP API
- `_example_test.go` — Usage examples as tests

### Test Coverage

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

# Specific file
go test -v ./go/tenant/...

# With coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

---

## API reference

### Core Types

```go
// UserTier - User access tier
type UserTier string

const (
    TierFree   UserTier = "free"   // Free tier, limited features
    TierApollo UserTier = "apollo" // Standard paid tier
    TierHades  UserTier = "hades"  // Premium tier, unlimited
)

// Methods on UserTier
func (t UserTier) MaxWorkspaces() int           // Returns -1 for unlimited
func (t UserTier) HasFeature(code string) bool   // Check feature access

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

// Feature - Feature definition
type Feature struct {
    Code        string
    Name        string
    Description string
    Tier        UserTier // Minimum tier required
}

// TenantOptions - Service configuration
type TenantOptions struct {
    APIURL   string
    APIToken string
    Timeout  time.Duration
}

// Tenant - Main service
type Tenant struct {
    *core.ServiceRuntime[TenantOptions]
    client        *TenantClient
    cache         *TenantCache
    entitlements  EntitlementService
    alertHandlers []AlertHandler
    // ...
}
```

### Main Functions

```go
// Service registration
func Register(c *core.Core) core.Result
func New(c *core.Core, opts TenantOptions) *Tenant

// User context
func UserFromCtx(ctx context.Context) core.Result
func WithUser(ctx context.Context, user *User) context.Context
func UserIDFromCtx(ctx context.Context) core.Result
func WithUserID(ctx context.Context, userID int64) context.Context

// Workspace context
func WorkspaceFromCtx(ctx context.Context) core.Result
func WithWorkspace(ctx context.Context, workspace *Workspace) context.Context
func WorkspaceIDFromCtx(ctx context.Context) core.Result
func WithWorkspaceID(ctx context.Context, workspaceID int64) context.Context

// Feature context
func FeatureFromCtx(ctx context.Context, code string) core.Result
func WithFeature(ctx context.Context, feature *Feature) context.Context

// Scope context
func ScopeFromCtx(ctx context.Context) Scope
func WithScope(ctx context.Context, scope Scope) context.Context

// Tenant service methods
func (t *Tenant) Can(ctx context.Context, workspace *Workspace, featureCode string, count int) core.Result
func (t *Tenant) CanN(ctx context.Context, workspace *Workspace, featureCode string, count int) core.Result
func (t *Tenant) CanBoosted(ctx context.Context, workspace *Workspace, featureCode string, count int) core.Result
func (t *Tenant) RecordUsage(ctx context.Context, workspace *Workspace, featureCode string, count int)
func (t *Tenant) GetUser(ctx context.Context, userID int64) core.Result
func (t *Tenant) GetWorkspace(ctx context.Context, workspaceID int64) core.Result
func (t *Tenant) ListWorkspaces(ctx context.Context, userID int64) core.Result
```

### Entitlement Service

```go
// EntitlementService interface
type EntitlementService interface {
    Check(ctx context.Context, user *User, workspace *Workspace, featureCode string, count int) core.Result
}

// LocalEntitlementService implementation
func NewLocalEntitlementService(cache *TenantCache, client *TenantClient) EntitlementService

// TenantCache methods
func NewTenantCache(store any) *TenantCache
func (c *TenantCache) GetCached(ctx context.Context, key string, fn func() core.Result) core.Result
func (c *TenantCache) SetCached(ctx context.Context, key string, value any, ttl time.Duration) core.Result
func (c *TenantCache) Invalidate(ctx context.Context, key string) core.Result
```

### TenantClient (PHP API)

```go
// TenantClient for PHP API
func NewTenantClient(apiURL, apiToken string, opts ...TenantClientOption) *TenantClient
func (c *TenantClient) GetUser(ctx context.Context, userID int64) core.Result
func (c *TenantClient) GetWorkspace(ctx context.Context, workspaceID int64) core.Result
func (c *TenantClient) ListWorkspaces(ctx context.Context, userID int64) core.Result
func (c *TenantClient) CheckFeature(ctx context.Context, userID int64, featureCode string) core.Result
func (c *TenantClient) TrackUsage(ctx context.Context, userID, workspaceID int64, featureCode string, count int) core.Result
```

### Result Types

```go
// CanResult - Result of entitlement check
type CanResult struct {
    Allowed bool
    Reason string
    Limit  int
    Used   int
}

func (r *CanResult) IsAllowed() bool
func (r *CanResult) IsDenied() bool

// Sentinel errors
var (
    ErrNoUserContext    = errors.New("no user in context")
    ErrNoWorkspaceContext = errors.New("no workspace in context")
    ErrFeatureDenied     = errors.New("feature denied")
    ErrUsageExceeded     = errors.New("usage exceeded")
)
```

---

## Related documentation

### Internal Documentation

| Resource | Description | Location |
|----------|-------------|----------|
| RFC | Package specification | [plans/code/core/go/tenant/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/tenant/RFC.md) |

### External References

| Resource | URL |
|----------|-----|
| Repository | [github.com/dappcore/go-tenant](https://github.com/dappcore/go-tenant) |
| Module | [pkg.go.dev/dappco.re/go/tenant](https://pkg.go.dev/dappco.re/go/tenant) |
| PHP Backend | (Internal) PHP API for tenant management |

