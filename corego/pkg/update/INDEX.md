---
type: Package Index
title: go-update Package Index
description: Complete index of go-update package components and API surface
module: dappco.re/go/update
---

# go-update — Package Index

> **Repository:** `core/go-update`
> **Module:** `dappco.re/go/update`
> **Type:** Library
> **Status:** Production
> **Lines:** ~1,500 (source)

---

## 📚 Quick Links

- **[README.md](./README.md)** — Complete package documentation
- **[RFC Specification](file:///Users/snider/Code/meowmix/plans/code/core/go/update/RFC.md)** — Technical specification
- **[CLAUDE.md](file:///Users/snider/Code/core/go-update/CLAUDE.md)** — Implementation details
- **[AGENTS.md](file:///Users/snider/Code/core/go-update/AGENTS.md)** — Agent guidance

### Sub-Specifications

- [Models RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/update/RFC.models.md) — Data models
- [Commands RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/update/RFC.commands.md) — Command specifications
- [Catalog RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/update/RFC.catalog.md) — Catalog
- [Imports RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/update/RFC.imports.md) — Import structure

---

## 🗂️ File Structure

### Source Files

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `version.go` | ~50 | Version management, formatting, semver comparison | ✅ Complete |
| `github.go` | ~200 | GitHub API client, release fetching, channel filtering | ✅ Complete |
| `generic_http.go` | ~100 | Generic HTTP-based binary download | ✅ Complete |
| `updater.go` | ~150 | Update checking, downloading, applying | ✅ Complete |
| `service.go` | ~100 | Core framework integration, service lifecycle | ✅ Complete |
| `cmd.go` | ~100 | CLI command registration and handling | ✅ Complete |
| `cmd_unix.go` | ~50 | Unix-specific binary replacement (atomic rename) | ✅ Complete |
| `cmd_windows.go` | ~50 | Windows-specific binary replacement | ✅ Complete |
| `http_client.go` | ~50 | HTTP client with authentication support | ✅ Complete |

### Build Files

| File | Purpose |
|------|---------|
| `build/main.go` | Build helper for version injection |

---

## 🔧 Public API Surface

### Version Management

```go
// Variables
var Version = "v0.0.0"  // Set at build time via ldflags

// Functions
func formatVersionForComparison(v string) string
func formatVersionForDisplay(v string, forceSemVerPrefix bool) string
```

### GitHub Client

```go
type GithubClient interface {
    GetLatestRelease(ctx context.Context, org, repo, channel string) (*Release, error)
    GetReleaseByTag(ctx context.Context, org, repo, tag string) (*Release, error)
    ListReleases(ctx context.Context, org, repo string) ([]*Release, error)
}

func NewGithubClient() GithubClient

// With custom HTTP client
func NewGithubClientWithHTTPClient(client *http.Client) GithubClient
```

### Types

```go
type Release struct {
    TagName         string
    Name            string
    Body            string
    Assets          []Asset
    Prerelease      bool
    PublishedAt     time.Time
}

type Asset struct {
    Name               string
    DownloadURL        string
    BrowserDownloadURL string
    Size               int64
}
```

### Update Checking

```go
// Check if newer version available
func CheckForNewerVersion(org, repo, channel string, forceSemVerPrefix bool) (*Release, bool, error)

// Check and update automatically
func CheckForUpdates(org, repo, channel string, forceSemVerPrefix bool, binaryURLTemplate string) error

// Check only (no download)
func CheckOnly(org, repo, channel string, forceSemVerPrefix bool) (bool, error)
```

### Update Application

```go
// Download and apply update
func DoUpdate(url string) error

// Rollback on error
func RollbackError(err error) error
```

### HTTP Updates

```go
// Check for HTTP updates
func CheckForUpdatesHTTP(baseURL, currentVersion string) (*GenericUpdateInfo, error)

// Check only for HTTP updates
func CheckOnlyHTTP(baseURL, currentVersion string) (bool, error)

type GenericUpdateInfo struct {
    Version string
    URL     string
}
```

### Update Service

```go
type UpdateServiceConfig struct {
    RepoURL           string
    Channel           string
    CheckOnStartup    CheckOnStartupType
    BinaryURLTemplate string
    HTTPClient        *http.Client
    Logger            Logger
}

type CheckOnStartupType int

const (
    CheckAndNotifyOnStartup CheckOnStartupType = iota
    CheckAndUpdateOnStartup
    NoCheckOnStartup
)

type UpdateService struct {
    // Callbacks
    OnUpdateStarted   func()
    OnUpdateAvailable func(*Release)
    OnUpdateApplied   func(*Release)
    OnUpdateFailed    func(error)
}

func NewUpdateService(config UpdateServiceConfig) (*UpdateService, error)
func (s *UpdateService) Start() error
func (s *UpdateService) CheckForUpdates() (*Release, bool, error)
func (s *UpdateService) ApplyUpdate(release *Release) error
func (s *UpdateService) ForceCheck()
```

### CLI Commands

```go
func RegisterCommands()
```

Registers:
- `core update check` — Check for newer version
- `core update apply` — Download and apply update
- `core update status` — Show current version

---

## 📦 Component Catalog

### Version Management

| Component | File | Purpose |
|-----------|------|---------|
| `Version` | `version.go` | Current version (set at build time) |
| `formatVersionForComparison` | `version.go` | Ensure "v" prefix for semver |
| `formatVersionForDisplay` | `version.go` | Format for user display |

**Build Integration:**
```bash
go build -ldflags="-X dappco.re/go/update.Version=v1.2.3"
```

### GitHub Client

| Component | File | Purpose |
|-----------|------|---------|
| `GithubClient` | `github.go` | Interface for GitHub API |
| `NewGithubClient` | `github.go` | Factory function |
| `GetLatestRelease` | `github.go` | Get latest release by channel |
| `GetReleaseByTag` | `github.go` | Get specific release by tag |
| `ListReleases` | `github.go` | List all releases |
| `Channel filtering` | `github.go` | Filter by stable/beta/alpha |

### Update Logic

| Component | File | Purpose |
|-----------|------|---------|
| `CheckForNewerVersion` | `updater.go` | Check if update available |
| `CheckForUpdates` | `updater.go` | Check and update |
| `CheckOnly` | `updater.go` | Check only |
| `DoUpdate` | `updater.go` | Download and apply |
| `RollbackError` | `updater.go` | Rollback on failure |

### HTTP Updates

| Component | File | Purpose |
|-----------|------|---------|
| `CheckForUpdatesHTTP` | `generic_http.go` | Check HTTP endpoint |
| `CheckOnlyHTTP` | `generic_http.go` | Check only |
| `GenericUpdateInfo` | `generic_http.go` | HTTP update metadata |

### Platform-Specific

| Component | File | Purpose | Platform |
|-----------|------|---------|----------|
| Atomic rename | `cmd_unix.go` | Atomic binary replacement | Unix |
| Delete+rename | `cmd_windows.go` | Binary replacement | Windows |

### HTTP Client

| Component | File | Purpose |
|-----------|------|---------|
| `NewAuthenticatedClient` | `http_client.go` | HTTP client with auth |
| `GITHUB_TOKEN` | `http_client.go` | Environment variable support |

---

## 📊 Channels

### Supported Channels

| Channel | Tag Pattern | Use Case | Auto-Update |
|---------|-------------|----------|-------------|
| `""` (empty) | `v*` (all) | Latest stable | ✅ Yes |
| `stable` | `v*` (no suffix) | Production releases | ✅ Yes |
| `beta` | `v*-beta*` | Beta testers | ⚠️ Opt-in |
| `alpha` | `v*-alpha*` | Early adopters | ⚠️ Opt-in |

### Tag Examples

```
v1.2.3          → stable (matches "" and "stable")
v1.3.0-beta.1   → beta (matches "beta")
v1.4.0-alpha.1  → alpha (matches "alpha")
v2.0.0-rc.1     → prerelease (matches by prerelease flag)
```

---

## 🔗 Dependencies

### Internal Dependencies

| Package | Purpose | Import Path |
|---------|---------|-------------|
| go-log | Error handling | `forge.lthn.ai/core/go-log` |
| go-io | File operations | `dappco.re/go/io` |

### External Dependencies

| Package | Purpose | Version |
|---------|---------|---------|
| `golang.org/x/mod/semver` | Semantic version comparison | Latest |
| `github.com/minio/selfupdate` | Binary self-update primitives | Latest |
| `net/http` | HTTP client | Stdlib |
| `io` | I/O operations | Stdlib |
| `os` | File operations | Stdlib |

---

## 📊 Statistics

### Code Metrics

```
Total Go files:           10+
Total lines (source):    ~1,500
Total lines (tests):     ~1,000
Public API symbols:      ~15
Supported platforms:     2 (Unix, Windows)
Supported channels:      4 (stable, beta, alpha, empty)
Supported sources:       2 (GitHub, HTTP)
```

### Test Coverage

| Component | Files | Tests | Status |
|-----------|-------|-------|--------|
| Version | 1 | 10+ | ✅ Complete |
| GitHub | 1 | 20+ | ✅ Complete |
| Updater | 1 | 15+ | ✅ Complete |
| Generic HTTP | 1 | 10+ | ✅ Complete |
| Unix CMD | 1 | 5+ | ✅ Complete |
| Windows CMD | 1 | 5+ | ✅ Complete |
| Service | 1 | 10+ | ✅ Complete |
| CLI | 1 | 5+ | ✅ Complete |

---

## 🏷️ Tags & Categories

### Technology Tags

- `updates` — Primary tag
- `versioning` — Version management
- `releases` — Release management
- `distribution` — Binary distribution
- `self-update` — Self-updating binaries
- `binary` — Binary management
- `github` — GitHub integration
- `http` — HTTP downloads
- `rollback` — Rollback support

### Feature Tags

- `version` — Version management
- `update` — Update checking
- `download` — Binary download
- `apply` — Update application
- `channel` — Release channels
- `semver` — Semantic versioning
- `atomic` — Atomic operations

### Platform Tags

- `unix` — Unix/Linux/macOS support
- `windows` — Windows support
- `cross-platform` — Cross-platform support

---

## 🎯 Usage Patterns

### Pattern 1: Automatic Updates

```go
config := update.UpdateServiceConfig{
    RepoURL:        "https://github.com/core/myapp",
    Channel:        "stable",
    CheckOnStartup: update.CheckAndUpdateOnStartup,
}
service, _ := update.NewUpdateService(config)
service.Start()
```

### Pattern 2: Notify Only

```go
config := update.UpdateServiceConfig{
    RepoURL:        "https://github.com/core/myapp",
    Channel:        "beta",
    CheckOnStartup: update.CheckAndNotifyOnStartup,
}
service.OnUpdateAvailable = func(r *update.Release) {
    notifyUser(r)
}
```

### Pattern 3: Manual Check

```go
release, available, _ := update.CheckForNewerVersion(
    "core", "myapp", "stable", false)
if available {
    update.DoUpdate(release.Assets[0].DownloadURL)
}
```

### Pattern 4: Generic HTTP

```go
config := update.UpdateServiceConfig{
    RepoURL:        "https://releases.example.com",
    BinaryURLTemplate: "{version}/app-{os}-{arch}",
    CheckOnStartup: update.CheckAndUpdateOnStartup,
}
```

---

## 📋 Compliance Summary

### Coding Standards

✅ **UK English:** in comments and strings
✅ **Strict types:** All parameters and return types
✅ **Test naming:** `TestName_{Good,Bad,Ugly}` suffix pattern
✅ **License:** EUPL-1.2
✅ **Error handling:** `coreerr.E("scope", "message", err)` from go-log

### File I/O

✅ **Use:** `forge.lthn.ai/core/go-io` for file operations
❌ **Never use:** `os.ReadFile` or `os.WriteFile` directly

### Error Handling

✅ **Use:** `coreerr.E("FunctionName", "what failed", underlyingErr)`
❌ **Never use:** `fmt.Errorf`, `errors.New`, `errors.Wrap`

---

## 📝 Maintenance Information

- **Author:** Mistral Vibe (Purberus <purberus@lthn.ai>)
- **Created:** 2026-06-18T02:00:00Z
- **Last Updated:** 2026-06-18T02:00:00Z
- **Version:** 1.0.0
- **Licence:** EUPL-1.2
- **Repository:** `forge.lthn.sh/core/go-update`
- **Module:** `dappco.re/go/update`

### Key Contacts

- **Project Lead:** Hades (Lethean)
- **Maintainer:** Purberus <purberus@lthn.ai>
- **Used by:** Core CLI, Core Agent
- **CI/CD:** Woodpecker.yml

---

*Package index generated: 2026-06-18T02:00:00Z*
