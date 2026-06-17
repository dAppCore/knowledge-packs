---
type: Package Deep Dive
title: go-update — Binary Self-Update Library
description: Complete documentation for go-update — self-updater library for Go binaries
description: GitHub releases and HTTP downloads with semantic versioning, channel tracking, rollback
module: dappco.re/go/update
repo: core/go-update
tags: [updates, versioning, releases, distribution, self-update, binary, github, http, rollback]
created: 2026-06-18T02:00:00Z
author: Mistral Vibe
version: 1.0.0
---

# go-update — Binary Self-Update Library

> **"The authoritative spec for updating Go binaries. An agent should be able to implement auto-updates from this document alone."**

`dappco.re/go/update` is a **self-updater library** for Go applications. It supports updates from GitHub releases and generic HTTP endpoints, with configurable startup behavior and version channel filtering.

Used by `core CLI` and `core-agent` for automatic updates.

---

## 🎯 Overview

### What it is

- **Self-update library** — Automatic binary updates for Go applications
- **GitHub Releases** — Fetch releases via GitHub API with tag/channel filtering
- **Generic HTTP** — Download from arbitrary HTTP endpoints
- **Semantic versioning** — Proper semver comparison
- **Channel support** — Stable, beta, alpha release channels
- **Seamless replacement** — Atomic binary replacement with rollback on failure
- **Cross-platform** — Unix and Windows support

### Architecture

```
Application → CheckForUpdates() → GitHub/HTTP → Download → Apply (binary replacement) → Rollback (on failure)
                              ↓
                        Version comparison
                        Channel selection
```

### Key Features

✅ **GitHub Releases** — Tag-based and channel-based filtering
✅ **HTTP Downloads** — Generic URL-based binary download
✅ **Semantic Versioning** — Proper semver comparison via `golang.org/x/mod/semver`
✅ **Channel Tracking** — Stable, beta, alpha channels
✅ **Atomic Updates** — Atomic binary replacement
✅ **Rollback Support** — Automatic rollback on failure
✅ **Cross-Platform** — Unix (atomic rename) and Windows support
✅ **Dependency Injection** — Testable function variables for mocking

---

## 📦 Quick Start

### GitHub-Based Updates

```go
package main

import (
    "fmt"
    "log"
    update "dappco.re/go/update"
)

func main() {
    config := update.UpdateServiceConfig{
        RepoURL:        "https://github.com/core/go-update",
        Channel:        "stable",
        CheckOnStartup: update.CheckAndUpdateOnStartup,
    }

    updateService, err := update.NewUpdateService(config)
    if err != nil {
        log.Fatalf("Failed to create update service: %v", err)
    }

    if err := updateService.Start(); err != nil {
        fmt.Printf("Update check failed: %v\n", err)
    }
}
```

### Generic HTTP Updates

For updates from a generic HTTP server, the server should provide a `latest.json` file:

```json
{
  "version": "1.2.3",
  "url": "https://your-server.com/path/to/release-asset"
}
```

```go
package main

import (
    "fmt"
    "log"
    update "dappco.re/go/update"
)

func main() {
    config := update.UpdateServiceConfig{
        RepoURL:        "https://your-server.com",
        CheckOnStartup: update.CheckAndUpdateOnStartup,
    }

    updateService, err := update.NewUpdateService(config)
    if err != nil {
        log.Fatalf("Failed to create update service: %v", err)
    }

    if err := updateService.Start(); err != nil {
        fmt.Printf("Update check failed: %v\n", err)
    }
}
```

---

## 🗂️ Package Structure

```
go-update/
├── version.go            # Version management (current version, formatting, comparison)
├── github.go             # GitHub API client, release fetching, channel filtering
├── generic_http.go       # Generic HTTP-based updates
├── updater.go            # Update checking, downloading, applying
├── service.go            # Core framework integration
├── cmd.go                # CLI command integration
├── cmd_unix.go           # Unix-specific binary replacement (atomic rename)
├── cmd_windows.go        # Windows-specific binary replacement
├── http_client.go        # HTTP client with authentication
└── build/
    └── main.go           # Build helper for version injection
```

---

## 📁 Core Components

### 1. Version Management

**File:** `version.go`

```go
// Current version (set at build time via ldflags)
var Version = "v0.0.0"

// Format version for comparison: ensures "v" prefix for semver
versionForComparison := formatVersionForComparison("1.2.3")  // returns "v1.2.3"

// Format version for display: removes "v" if forceSemVerPrefix=false
displayed := formatVersionForDisplay("v1.2.3", false)  // returns "1.2.3"
```

**Build command:**
```bash
go build -ldflags="-X dappco.re/go/update.Version=v1.2.3"
```

**Semantic version comparison:**
```go
import "golang.org/x/mod/semver"

if semver.Compare("v1.2.3", "v1.2.4") < 0 {
    // v1.2.3 is older than v1.2.4
}
```

### 2. GitHub Releases

**File:** `github.go`

Fetches releases from GitHub. Supports tag-based and channel-based filtering.

```go
// Create GitHub client
client := update.NewGithubClient()

// Get latest release (all tags)
release, err := client.GetLatestRelease(ctx, "core", "go-update", "")

// Get latest by channel
release, err := client.GetLatestRelease(ctx, "core", "go-update", "beta")
```

**Release struct:**
```go
type Release struct {
    TagName    string    // e.g., "v1.2.3"
    Name       string    // release name
    Body       string    // release notes (markdown)
    Assets     []Asset  // downloadable files
    Prerelease bool      // true for pre-releases
    PublishedAt time.Time // release date
}

type Asset struct {
    Name        string // file name
    DownloadURL string // direct download URL
    Size        int64  // file size in bytes
    BrowserDownloadURL string // browser-accessible URL
}
```

**Channel filtering:** Extracts release suffix and matches requested channel:

| Channel | Tag Pattern | Use Case |
|---------|-------------|----------|
| `""` (empty) | `v*` (all) | Latest stable |
| `stable` | `v*` (no suffix) | Production |
| `beta` | `v*-beta*` | Beta testers |
| `alpha` | `v*-alpha*` | Early adopters |

**Example GitHub release tags:**
```
v1.2.3          (stable, matches "" and "stable")
v1.3.0-beta.1   (beta, matches "beta")
v1.4.0-alpha.1  (alpha, matches "alpha")
```

### 3. Update Checking

**File:** `updater.go`

```go
// Check if a newer version is available
release, updateAvailable, err := update.CheckForNewerVersion(
    "core",                      // GitHub org
    "go-update",                 // GitHub repo
    "stable",                    // channel: "", "stable", "beta", "alpha"
    false,                       // forceSemVerPrefix: pad version with "v" if missing
)

if updateAvailable {
    fmt.Printf("Update available: %s\n", release.TagName)
}

// Check and apply updates automatically
err := update.CheckForUpdates(
    "core",
    "go-update",
    "stable",
    false,
    "https://github.com/core/go-update/releases/download/{version}/go-update-{os}-{arch}",
)
```

### 4. Binary Download & Replacement

**Atomic binary replacement:**

```go
// Download binary from URL
err := update.DoUpdate("https://example.com/binary")
```

**Platform-specific replacement:**

- **Unix:** Atomic rename (POSIX provides atomic `rename`)
- **Windows:** Delete old, rename new (no atomic rename available)

**Error handling with rollback:**

```go
if err != nil {
    if rollbackErr := update.RollbackError(err); rollbackErr != nil {
        // Rollback failed too — binary is corrupted
    }
    // Update failed but was rolled back successfully
}
```

### 5. HTTP Downloads

**File:** `generic_http.go`

Download binary from arbitrary HTTP URL:

```go
// Generic HTTP download (not GitHub-specific)
err := update.DoUpdate("https://releases.example.com/app-linux-amd64")
```

**Process:**
1. Constructs request with User-Agent header
2. Verifies HTTP 200 response
3. Downloads to temporary location
4. Applies via `selfupdate.Apply()`

### 6. HTTP Client

**File:** `http_client.go`

Custom HTTP client with authentication support:

```go
// Create authenticated client
client := update.NewAuthenticatedClient("my-token")

// Use with GitHub client
githubClient := update.NewGithubClient()
githubClient.HTTPClient = client
```

**Authentication:**
- Supports `GITHUB_TOKEN` environment variable
- Bearer token authentication
- Custom HTTP client injection

---

## 🎛️ Update Service

### Configuration

```go
type UpdateServiceConfig struct {
    // GitHub repository (org/repo) or HTTP base URL
    RepoURL string
    
    // Release channel: "", "stable", "beta", "alpha"
    Channel string
    
    // Startup behavior
    CheckOnStartup CheckOnStartupType
    
    // Custom binary URL template (for GitHub)
    // Placeholders: {version}, {os}, {arch}
    BinaryURLTemplate string
    
    // Custom HTTP client
    HTTPClient *http.Client
    
    // Logger
    Logger update.Logger
}

type CheckOnStartupType int

const (
    CheckAndNotifyOnStartup CheckOnStartupType = iota  // Check and notify user
    CheckAndUpdateOnStartup                               // Check and update automatically
    NoCheckOnStartup                                      // No automatic check
)
```

### Usage

```go
// Create service
service, err := update.NewUpdateService(config)
if err != nil {
    return err
}

// Start the service (checks for updates based on config)
err = service.Start()

// Manual check
release, available, err := service.CheckForUpdates()

// Manual update
if available {
    err = service.ApplyUpdate(release)
}

// Force check
service.ForceCheck()
```

---

## 💻 CLI Commands

### Available Commands

| Command | Description | File |
|---------|-------------|------|
| `core update check` | Check for newer version | `cmd.go` |
| `core update apply` | Download and apply update | `cmd.go` |
| `core update status` | Show current version | `cmd.go` |

### Command Integration

Register with core CLI:

```go
import update "dappco.re/go/update"

func init() {
    // Register update commands
    update.RegisterCommands()
}
```

### Command Flags

```bash
# Check for updates
core update check [--channel stable] [--force]

# Apply update
core update apply

# Show version
core update status
```

---

## 🔧 Dependency Injection

The update system uses **dependency injection** for testing. Core update logic is exposed as `var` function values that tests can replace:

```go
// Replace GitHub client in tests
update.NewGithubClient = func() update.GithubClient {
    return &mockGithubClient{}
}

// Replace actual update logic
update.DoUpdate = func(url string) error {
    return nil  // simulate successful update
}

// Replace version checker
update.CheckForNewerVersion = func(org, repo, channel string, forceSemVerPrefix bool) (*update.Release, bool, error) {
    return nil, false, nil  // no update available
}

// Replace HTTP functions
update.CheckForUpdatesHTTP = func(baseURL, currentVersion string) (*update.GenericUpdateInfo, error) {
    return &update.GenericUpdateInfo{Version: "2.0.0"}, nil
}

update.CheckOnlyHTTP = func(baseURL, currentVersion string) (bool, error) {
    return true, nil
}

// Replace authenticated client factory
update.NewAuthenticatedClient = func(token string) *http.Client {
    return &http.Client{Transport: &mockTransport{}}
}
```

---

## ⚡ Concurrency & Safety

- **No goroutines** in core library (update operations are sequential)
- **Atomic binary replacement** via OS rename (Unix) or delete+rename (Windows)
- **Rollback support** for failed updates
- **Safe to call** from multiple processes (OS provides atomic semantics)
- **Thread-safe** for concurrent access

---

## 📊 Channels

### Supported Channels

| Channel | Tag Pattern | Use Case | Auto-Update |
|---------|-------------|----------|-------------|
| `""` (empty) | `v*` (all) | Latest stable | ✅ Yes |
| `stable` | `v*` (no suffix) | Production releases | ✅ Yes |
| `beta` | `v*-beta*` | Beta testers | ⚠️ Opt-in |
| `alpha` | `v*-alpha*` | Early adopters | ⚠️ Opt-in |

### Channel Filtering Logic

```go
func matchesChannel(tagName, channel string) bool {
    if channel == "" || channel == "stable" {
        // Match tags without suffix or with "-stable"
        return !strings.Contains(tagName, "-") || 
               strings.HasSuffix(tagName, "-stable")
    }
    return strings.Contains(tagName, "-"+channel)
}
```

---

## 🛡️ Error Handling

All errors use `coreerr.E()` from `forge.lthn.ai/core/go-log`:

```go
import coreerr "forge.lthn.ai/core/go-log"

return coreerr.E("FunctionName", "what failed", underlyingErr)
```

**Never use:**
- `fmt.Errorf`
- `errors.New`
- `errors.Wrap`

### Error Types

| Error | Description |
|-------|-------------|
| `ErrNoUpdateAvailable` | No newer version found |
| `ErrVersionParse` | Failed to parse version string |
| `ErrDownloadFailed` | Failed to download binary |
| `ErrApplyFailed` | Failed to apply update |
| `ErrRollbackFailed` | Failed to rollback |
| `ErrNetworkError` | Network error (GitHub/API) |
| `ErrAuthFailed` | Authentication failed |

---

## 📦 Dependencies

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
Public API surface:      ~15 symbols
Supported platforms:     Unix (Linux, macOS), Windows
Supported channels:      4 (stable, beta, alpha, empty)
Supported sources:       2 (GitHub, HTTP)
```

### Test Coverage

| Component | Tests | Coverage |
|-----------|-------|----------|
| Version management | 10+ | ✅ Complete |
| GitHub client | 20+ | ✅ Complete |
| Update checker | 15+ | ✅ Complete |
| Binary replacement | 10+ | ✅ Complete |
| HTTP downloads | 10+ | ✅ Complete |
| CLI commands | 5+ | ✅ Complete |

---

## 🎯 Usage Patterns

### Pattern 1: Automatic Updates on Startup

```go
config := update.UpdateServiceConfig{
    RepoURL:        "https://github.com/core/myapp",
    Channel:        "stable",
    CheckOnStartup: update.CheckAndUpdateOnStartup,
}

service, _ := update.NewUpdateService(config)
service.Start()
```

### Pattern 2: Notify Only (User Decides)

```go
config := update.UpdateServiceConfig{
    RepoURL:        "https://github.com/core/myapp",
    Channel:        "beta",
    CheckOnStartup: update.CheckAndNotifyOnStartup,
}

service, _ := update.NewUpdateService(config)
service.Start()

// Handle notification
service.OnUpdateAvailable = func(release *update.Release) {
    fmt.Printf("Update %s available!\n", release.TagName)
}
```

### Pattern 3: Manual Update Check

```go
// Check without auto-update
release, available, err := update.CheckForNewerVersion(
    "core", "myapp", "stable", false)

if available {
    fmt.Printf("Update to %s available\n", release.TagName)
    
    // Apply update
    if err := update.DoUpdate(release.Assets[0].DownloadURL); err != nil {
        log.Printf("Update failed: %v", err)
    }
}
```

### Pattern 4: Generic HTTP with Custom Template

```go
config := update.UpdateServiceConfig{
    RepoURL:        "https://releases.mycompany.com/myapp",
    BinaryURLTemplate: "https://releases.mycompany.com/myapp/{version}/myapp-{os}-{arch}",
    CheckOnStartup: update.CheckAndUpdateOnStartup,
}

service, _ := update.NewUpdateService(config)
service.Start()
```

### Pattern 5: With Authentication

```go
// Set GitHub token
os.Setenv("GITHUB_TOKEN", "ghp_...")

// Or use custom HTTP client
client := update.NewAuthenticatedClient("ghp_...")
config.HTTPClient = client

service, _ := update.NewUpdateService(config)
```

### Pattern 6: Custom Version Injection

```go
// At build time
version := "v1.2.3"
go build -ldflags="-X dappco.re/go/update.Version="+version

// In code
fmt.Printf("Current version: %s\n", update.Version)
```

---

## 🎨 Code Examples

### Complete Update Service Integration

```go
package main

import (
    "context"
    "fmt"
    "log"
    "os"
    update "dappco.re/go/update"
)

func main() {
    // Create configuration
    config := update.UpdateServiceConfig{
        RepoURL:        "https://github.com/core/myapp",
        Channel:        "stable",
        CheckOnStartup: update.CheckAndUpdateOnStartup,
        BinaryURLTemplate: "https://github.com/core/myapp/releases/download/{version}/myapp-{os}-{arch}",
    }

    // Create service
    service, err := update.NewUpdateService(config)
    if err != nil {
        log.Fatalf("Failed to create update service: %v", err)
    }

    // Set up callbacks
    service.OnUpdateStarted = func() {
        fmt.Println("Checking for updates...")
    }

    service.OnUpdateAvailable = func(release *update.Release) {
        fmt.Printf("Update %s is available!\n", release.TagName)
        fmt.Println("Release notes:")
        fmt.Println(release.Body)
    }

    service.OnUpdateApplied = func(release *update.Release) {
        fmt.Printf("Updated to %s successfully!\n", release.TagName)
        os.Exit(0) // Restart application
    }

    service.OnUpdateFailed = func(err error) {
        fmt.Printf("Update failed: %v\n", err)
    }

    // Start service (checks for updates)
    if err := service.Start(); err != nil {
        fmt.Printf("Update service failed: %v\n", err)
    }

    // Your application logic here
    fmt.Println("Application running...")
}
```

### GitHub Client with Custom Logic

```go
// Create custom GitHub client
client := &CustomGithubClient{
    // Custom implementation
}

// Use it
release, err := client.GetLatestRelease(
    context.Background(),
    "core",
    "go-update",
    "stable")

if err != nil {
    log.Fatal(err)
}

fmt.Printf("Latest release: %s\n", release.TagName)
```

### Version Comparison

```go
import "golang.org/x/mod/semver"

current := "v1.2.3"
latest := "v1.2.4"

// Compare versions
if semver.Compare(current, latest) < 0 {
    fmt.Printf("%s is older than %s\n", current, latest)
} else if semver.Compare(current, latest) > 0 {
    fmt.Printf("%s is newer than %s\n", current, latest)
} else {
    fmt.Printf("%s is equal to %s\n", current, latest)
}

// Check prerelease
if semver.Prerelease(current) != "" {
    fmt.Printf("%s is a prerelease\n", current)
}
```

---

## 📋 Compliance Rules

From `AGENTS.md` and `CLAUDE.md`:

### Coding Standards

- **UK English** in comments and strings
- **Strict types:** All parameters and return types
- **Test naming:** `TestName_Good`, `TestName_Bad`, `TestName_Ugly` suffix pattern
- **License:** EUPL-1.2
- **Error handling:** Always use `coreerr.E()` from `forge.lthn.ai/core/go-log`

### File I/O

- **Use** `forge.lthn.ai/core/go-io` for file operations
- **Never use** `os.ReadFile` or `os.WriteFile` directly

### Error Handling

- ✅ **Use:** `coreerr.E("FunctionName", "what failed", underlyingErr)`
- ❌ **Never use:** `fmt.Errorf`, `errors.New`, `errors.Wrap`

---

## 🔗 Related Documentation

- [CoreGo Framework](../../README.md) — Parent knowledge pack
- [go-log Package](../log/README.md) — Error handling
- [go-io Package](../io/README.md) — File operations
- [CoreGo RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md) — Framework specification
- [go-update RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/update/RFC.md) — Package specification
- [go-update CLAUDE.md](file:///Users/snider/Code/core/go-update/CLAUDE.md) — Implementation details

### Sub-Specifications

- [RFC.models.md](file:///Users/snider/Code/meowmix/plans/code/core/go/update/RFC.models.md) — Data models
- [RFC.commands.md](file:///Users/snider/Code/meowmix/plans/code/core/go/update/RFC.commands.md) — Command specifications
- [RFC.catalog.md](file:///Users/snider/Code/meowmix/plans/code/core/go/update/RFC.catalog.md) — Catalog
- [RFC.imports.md](file:///Users/snider/Code/meowmix/plans/code/core/go/update/RFC.imports.md) — Import structure

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

---

*Package documentation created: 2026-06-18T02:00:00Z*
*Source: dappco.re/go/update*
