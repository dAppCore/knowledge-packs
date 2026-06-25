---
type: Package Documentation
package: config
module: dappco.re/go/config
repo: core/config
title: go-config
description: Core configuration module with dual-Viper pattern, manifest resolution, feature flags, and filesystem watching
lang: go
tags:
  - config
  - configuration
  - dual-viper
  - yaml
  - manifests
  - feature-flags
  - core-service
  - config-resolution
  - filesystem-watching
---
# go-config — Core configuration module

**Module:** `dappco.re/go/config`
**Repository:** [~/Code/core/config/](file:///Users/snider/Code/core/config/)
**Status:** Production-ready
**License:** EUPL-1.2
**Language:** Go 1.26+
**Dependencies:** Core framework, go-io (Medium abstraction), Viper, testify
**Maintainer:** Purberus <purberus@lthn.ai>
**Last Updated:** 2026-06-18

---

## Overview

`go-config` is the Core configuration module that provides a unified way for Core services and command-line tools to resolve configuration from multiple sources. It implements a **dual-Viper pattern** that prevents environment variables from leaking into saved configuration files.

### Key capabilities

1. **Dual-Viper pattern** — Two Viper instances: read view (file + env + defaults) and write view (file + explicit writes only)
2. **Hierarchical discovery** — Discovers `.core/config.yaml` files by walking up from project directory
3. **Manifest resolution** — Resolves typed Core manifests (`build.yaml`, `test.yaml`, `workspace.yaml`, `manifest.yaml`, `agent.yaml`, `zone.yaml`)
4. **Signature verification** — Signs and verifies view and package manifests with ed25519 signatures
5. **Feature flags** — Exposes feature flags through config, process-level defaults, and environment overrides
6. **Core service integration** — Runs as a Core service with lifecycle management, actions, commands, and optional filesystem watching
7. **Storage abstraction** — All file I/O through `coreio.Medium` abstraction from go-io

### Configuration resolution priority (ascending)

```
Defaults → File → Environment Variables (CORE_CONFIG_*) → Explicit Set()
```

---

## Architecture

### Dual-Viper pattern

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Dual-Viper Configuration Pattern                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Config Type                                          │ │
│  │  Holds two *viper.Viper instances:                                    │ │
│  │                                                                         │ │
│  │  ┌─────────────────────────┐  ┌─────────────────────────┐          │ │
│  │  │   v (read view)          │  │   f (write view)        │          │ │
│  │  │   Full configuration     │  │   File-only             │          │ │
│  │  │   - File data            │  │   - File data           │          │ │
│  │  │   - Environment vars     │  │   - Explicit Set()      │          │ │
│  │  │   - Defaults             │  │                         │          │ │
│  │  │                         │  │   Used for:             │          │ │
│  │  │   Used for:             │  │   - Commit()            │          │ │
│  │  │   - Get()                │  │   - Persistence         │          │ │
│  │  │   - All()                │  │                         │          │ │
│  │  │   - Reads                │  │                         │          │ │
│  │  └─────────────────────────┘  └─────────────────────────┘          │ │
│  │                                                                         │ │
│  │  INVARIANT: Write to both v and f; Read from v; Persist from f        │ │
│  │  This prevents environment variables from leaking into YAML files   │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Resolution Sources                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │  Priority (ascending):                                                  │ │
│  │                                                                         │ │
│  │  1. Defaults     — Hardcoded in code                                    │ │
│  │  2. File         — .core/config.yaml files                             │ │
│  │  3. Environment  — CORE_CONFIG_* variables                              │ │
│  │  4. Set()        — Explicit cfg.Set() calls                             │ │
│  │                                                                         │ │
│  │  Example: CORE_CONFIG_dev__editor=vim overrides config.yaml             │ │
│  │  Example: cfg.Set("dev.editor", "vim") overrides all                   │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Discovery Mechanism                                      │
│  Walk up from current directory looking for .core/config.yaml:           │
│  /project/.core/config.yaml                                               │
│  /project/.core/                                                           │
│  /user/.core/config.yaml                                                  │
│  ~/.core/config.yaml                                                      │
│  /etc/core/config.yaml                                                    │
│  /usr/local/etc/core/config.yaml                                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Component hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    go-config: Configuration Layer                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Core Types                                         │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │ │
│  │  │  Config     │  │   Service   │  │   Options   │              │ │
│  │  │             │  │             │  │             │              │ │
│  │  │ Dual-Viper  │  │ Core.Service│  │ WithMedium  │              │ │
│  │  │ Get/Set     │  │ Runtime     │  │ WithPath    │              │ │
│  │  │ Commit     │  │ Lifecycle   │  │ WithEnv     │              │ │
│  │  │ Watch      │  │ Start/Stop  │  │ Prefix     │              │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘              │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Manifest Support                                    │ │
│  │  build.yaml    test.yaml    workspace.yaml    manifest.yaml          │ │
│  │  agent.yaml    zone.yaml                                        │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Signature Support                                   │ │
│  │  Sign()        Verify()        ed25519 keys                            │ │
│  │  View manifests Package manifests                                     │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Feature Flags                                        │ │
│  │  Feature()    IsEnabled()    SetFeature()                              │ │
│  │  Process defaults + Environment + Explicit overrides                   │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Watch Support                                        │ │
│  │  Watch()      Unwatch()      Callback on file changes                  │ │
│  │  Filesystem monitoring for config.yaml changes                         │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Path Utilities                                       │ │
│  │  Resolve()   Join()    Clean()    Expand()    UserDir()                │ │
│  │  XDG compliance        Platform-specific paths                          │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Repository structure

```
go/
├── Core Configuration
│   ├── config.go            # Main Config type with dual-Viper
│   ├── config_test.go       # Config tests (Good/Bad/Ugly)
│   ├── config_example_test.go # Usage examples
│   ├── service.go           # Core service wrapper
│   └── service_test.go      # Service tests
│
├── Manifest Handling
│   ├── manifest.go          # Manifest resolution
│   ├── manifest_test.go     # Manifest tests
│   ├── schema/              # Manifest schemas
│   │   ├── build.yaml
│   │   ├── test.yaml
│   │   ├── workspace.yaml
│   │   ├── manifest.yaml
│   │   ├── agent.yaml
│   │   └── zone.yaml
│   └── schema.go            # Schema utilities
│
├── Resolution
│   ├── resolve.go           # Path/value resolution
│   ├── resolve_test.go      # Resolution tests
│   └── resolve_example_test.go
│
├── Discovery
│   ├── discover.go          # Config file discovery
│   └── discover_test.go     # Discovery tests
│
├── Watching
│   ├── watch.go             # Filesystem watching
│   └── watch_test.go        # Watch tests
│
├── Feature Flags
│   ├── feature.go           # Feature flag support
│   └── feature_test.go      # Feature tests
│
├── Environment
│   ├── env.go               # Environment variable handling
│   └── env_test.go          # Environment tests
│
├── Paths
│   ├── paths.go             # Path utilities
│   ├── paths_test.go        # Path tests
│   └── xdg.go               # XDG compliance
│
├── Workspace
│   ├── workspace.go         # Workspace utilities
│   └── workspace_test.go    # Workspace tests
│
├── Images Manifest
│   ├── images_manifest.go   # Docker image manifest support
│   └── images_manifest_test.go
│
├── Conclave
│   ├── conclave.go          # Conclave configuration
│   └── conclave_test.go     # Conclave tests
│
└── Module
    ├── go.mod
    └── go.sum
```

---

## Core types and interfaces

### Config type

```go
type Config struct {
    // Dual-Viper instances
    v *viper.Viper  // Read view: file + env + defaults
    f *viper.Viper  // Write view: file + explicit Set()
    
    // Options
    medium   coreio.Medium   // Storage abstraction
    path     string           // Config file path
    envPrefix string          // Environment variable prefix (default: CORE_CONFIG)
    
    // Watch support
    watchers map[string][]func()
}

// Interface compliance
var _ core.Config = (*Config)(nil)
```

### Interface contract

```go
// core.Config interface that Config implements
type Config interface {
    Get(key string, target any) error
    Set(key string, value any) error
    All() map[string]any
    Has(key string) bool
    Commit() error
    Watch(key string, callback func()) error
    Unwatch(key string, callback func()) error
}
```

---

## Core functionality

### 1. Basic configuration

#### Creating a config

```go
// Create with defaults
cfg, err := config.New()

// Create with options
cfg, err := config.New(
    config.WithMedium(coreio.Local),
    config.WithPath(".core/config.yaml"),
    config.WithEnvPrefix("MYAPP_CONFIG"),
)
```

#### Getting values

```go
// Get a value
var editor string
if err := cfg.Get("dev.editor", &editor); err != nil {
    panic(err)
}

// Get with default
var port int
if err := cfg.Get("server.port", &port); err == nil {
    // Use port
} else {
    port = 8080 // Default
}

// Get all configuration
all := cfg.All()

// Check if key exists
if cfg.Has("dev.editor") {
    // Key exists
}
```

#### Setting values

```go
// Set a value (writes to both v and f)
if err := cfg.Set("dev.editor", "vim"); err != nil {
    panic(err)
}

// Set multiple values
if err := cfg.Set("server.port", 8080); err != nil {
    panic(err)
}
if err := cfg.Set("server.host", "localhost"); err != nil {
    panic(err)
}
```

#### Committing changes

```go
// Persist changes to file (uses f, not v)
if err := cfg.Commit(); err != nil {
    panic(err)
}
```

### 2. Dual-Viper invariant

The critical invariant that must be maintained:

```go
// ✅ CORRECT: Write to both v and f
func (c *Config) Set(key string, value any) error {
    c.v.Set(key, value)  // Write to read view
    c.f.Set(key, value)  // Write to write view
    return nil
}

// ❌ WRONG: Only writing to one
func (c *Config) Set(key string, value any) error {
    c.v.Set(key, value)  // Missing: c.f.Set(key, value)
    return nil            // This would lose the value on Commit()
}

// ✅ CORRECT: Read from v (includes env vars)
func (c *Config) Get(key string, target any) error {
    return c.v.UnmarshalKey(key, target)  // Uses read view
}

// ✅ CORRECT: Persist from f (file-only, no env leakage)
func (c *Config) Commit() error {
    return c.f.WriteConfig()  // Uses write view
}
```

### 3. Configuration discovery

#### Hierarchical discovery

```go
// Discover config walking up from current directory
cfg, err := config.New(config.WithPath(""))
// Will check: ./core/config.yaml, ../core/config.yaml, ~/.core/config.yaml, etc.

// Discover from specific path
cfg, err := config.New(config.WithPath("/project/.core/config.yaml"))

// Discover with custom medium (e.g., S3, SQLite)
cfg, err := config.New(
    config.WithMedium(coreio.NewS3Medium(bucket, prefix)),
    config.WithPath("config.yaml"),
)
```

#### Discovery algorithm

```
1. Start from current working directory
2. Check for .core/config.yaml
3. If not found, move up one directory
4. Repeat until root directory or config found
5. Also check standard locations:
   - ~/.core/config.yaml
   - /etc/core/config.yaml
   - /usr/local/etc/core/config.yaml
6. Merge all found configs (later files override earlier)
```

### 4. Environment variables

#### Default prefix

```bash
# All environment variables with prefix CORE_CONFIG_ are loaded
CORE_CONFIG_dev__editor=vim
CORE_CONFIG_server__port=8080
CORE_CONFIG_database__host=localhost
```

#### Custom prefix

```go
// Use custom prefix
cfg, err := config.New(config.WithEnvPrefix("MYAPP"))
// Now loads MYAPP_dev__editor, MYAPP_server__port, etc.
```

#### Environment variable format

```bash
# Double underscore represents nested keys
CORE_CONFIG_server__database__host=localhost
# Translates to: server.database.host

CORE_CONFIG_dev__editor=vim
# Translates to: dev.editor
```

### 5. Feature flags

#### Basic usage

```go
// Enable a feature
if err := cfg.SetFeature("experimental.ai", true); err != nil {
    panic(err)
}

// Check if feature is enabled
if cfg.IsEnabled("experimental.ai") {
    // Use experimental AI feature
}

// Get feature value
var enabled bool
if err := cfg.Feature("experimental.ai", &enabled); err != nil {
    // Feature not found
}
```

#### Resolution priority

```
1. Explicit SetFeature() calls (highest priority)
2. Environment: CORE_FEATURE_experimental__ai=true
3. Process-level defaults
4. Config file: features.experimental.ai: true
```

---

## Manifest resolution

### Supported manifest types

| Manifest | Purpose | Module |
|----------|---------|--------|
| `build.yaml` | Build configuration | `dappco.re/go/build` |
| `test.yaml` | Test configuration | `dappco.re/go/config` |
| `workspace.yaml` | Workspace configuration | `dappco.re/go/config` |
| `manifest.yaml` | Package manifest | Various |
| `agent.yaml` | Agent configuration | `dappco.re/go/agent` |
| `zone.yaml` | Zone configuration | `dappco.re/go/config` |

### Manifest loading

```go
// Load a typed manifest
var buildConfig BuildConfig
if err := cfg.Resolve("build.yaml", &buildConfig); err != nil {
    panic(err)
}

// Load from specific path
var testConfig TestConfig
if err := cfg.Resolve("/path/to/test.yaml", &testConfig); err != nil {
    panic(err)
}
```

### Manifest structure example

```yaml
# build.yaml
name: my-app
version: 1.0.0
go:
  version: 1.26
  modules:
    - dappco.re/go/core
    - dappco.re/go/config
  tags:
    - production
    - linux
  ldflags: "-s -w"
```

---

## Signature verification

### Signing manifests

```go
// Sign a manifest
manifest := map[string]any{
    "name": "my-package",
    "version": "1.0.0",
}

privateKey := ed25519.New()
signature, err := config.Sign(manifest, privateKey)
if err != nil {
    panic(err)
}
```

### Verifying manifests

```go
// Verify a signed manifest
publicKey := privateKey.Public()
if err := config.Verify(manifest, signature, publicKey); err != nil {
    panic("Signature verification failed")
}
```

### View vs package manifests

- **View manifests** — User-level configuration views
- **Package manifests** — Package-level metadata and signatures

---

## Filesystem watching

### Watching configuration

```go
// Watch for changes to a specific key
if err := cfg.Watch("dev.editor", func() {
    var editor string
    if err := cfg.Get("dev.editor", &editor); err == nil {
        fmt.Printf("Editor changed to: %s\n", editor)
    }
}); err != nil {
    panic(err)
}

// Watch all configuration
if err := cfg.Watch("", func() {
    fmt.Println("Configuration changed")
    all := cfg.All()
    // Process changes
}); err != nil {
    panic(err)
}
```

### Unwatching

```go
// Stop watching a key
if err := cfg.Unwatch("dev.editor", callback); err != nil {
    panic(err)
}

// Stop watching all
if err := cfg.Unwatch("", callback); err != nil {
    panic(err)
}
```

### Watch implementation

- Uses filesystem notifications (inotify on Linux, FSEvents on macOS, ReadDirectoryChangesW on Windows)
- Callback triggered on file modification
- Debounced to prevent rapid-fire notifications

---

## Path utilities

### Path resolution

```go
// Resolve a path (expand ~, env vars, relative paths)
resolved, err := config.Resolve("~/projects/my-app")

// Join paths
joined := config.Join("/path/to", "dir", "file.txt")
// Result: /path/to/dir/file.txt

// Clean path (remove . and .., normalize)
cleaned := config.Clean("/path/to/../file.txt")
// Result: /path/file.txt

// Expand path (expand ~ and env vars)
expanded, err := config.Expand("$HOME/projects/${USER}")
```

### XDG compliance

```go
// Get XDG config home
configHome, err := config.XDGConfigHome()
// Result: ~/.config on Linux, ~/Library/Application Support on macOS

// Get XDG data home
dataHome, err := config.XDGDataHome()
// Result: ~/.local/share on Linux, ~/Library/Application Support on macOS

// Get XDG cache home
cacheHome, err := config.XDGCacheHome()
// Result: ~/.cache on Linux, ~/Library/Caches on macOS

// Get user home directory
home, err := config.UserDir()
// Result: /home/user or /Users/user
```

### Platform-specific paths

```go
// Get platform-specific config directory
configDir, err := config.ConfigDir()
// Result: /etc/core on Linux, /usr/local/etc/core on macOS

// Get platform-specific data directory
dataDir, err := config.DataDir()
```

---

## Service integration

### Running as a Core service

```go
// Create a service
svc, err := config.NewService(
    config.WithMedium(coreio.Local),
    config.WithPath(".core/config.yaml"),
)
if err != nil {
    panic(err)
}

// Start the service
if err := svc.Start(); err != nil {
    panic(err)
}
defer svc.Stop()

// Use the service
var editor string
if err := svc.Get("dev.editor", &editor); err != nil {
    panic(err)
}
```

### Service lifecycle

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Service Lifecycle                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  OnStartup()                                                                  │
│    └── Initialize Viper instances                                             │
│    └── Load configuration from medium                                        │
│    └── Start filesystem watcher (if enabled)                                │
│    └── Register actions/commands                                             │
│                                                                              │
│  OnShutdown()                                                                 │
│    └── Stop filesystem watcher                                                │
│    └── Clean up resources                                                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Testing

### Test structure

Following AX Standard with Good/Bad/Ugly test triplets:

```
Config Tests:
├── config_test.go
│   ├── TestConfig_New_Good
│   ├── TestConfig_New_Bad_InvalidMedium
│   ├── TestConfig_New_Ugly_NilMedium
│   ├── TestConfig_Get_Good
│   ├── TestConfig_Get_Bad_NotFound
│   ├── TestConfig_Get_Ugly_TypeMismatch
│   ├── TestConfig_Set_Good
│   ├── TestConfig_Set_Bad_ReadOnly
│   └── TestConfig_Set_Ugly_CircularReference
│
Manifest Tests:
├── manifest_test.go
│   ├── TestManifest_Resolve_Good
│   ├── TestManifest_Resolve_Bad_NotFound
│   └── TestManifest_Resolve_Ugly_InvalidYAML
│
Discovery Tests:
├── discover_test.go
│   ├── TestDiscover_Good_Found
│   ├── TestDiscover_Bad_NotFound
│   └── TestDiscover_Ugly_PermissionDenied
│
Feature Tests:
├── feature_test.go
│   ├── TestFeature_IsEnabled_Good
│   ├── TestFeature_IsEnabled_Bad_Disabled
│   └── TestFeature_IsEnabled_Ugly_InvalidName
│
Watch Tests:
├── watch_test.go
│   ├── TestWatch_Good_CallbackTriggered
│   ├── TestWatch_Bad_InvalidPath
│   └── TestWatch_Ugly_FilesystemError
│
Path Tests:
├── paths_test.go
│   ├── TestResolve_Good
│   ├── TestResolve_Bad_InvalidPath
│   └── TestResolve_Ugly_EmptyPath
│
XDG Tests:
├── xdg_test.go
│   ├── TestXDGConfigHome_Good
│   └── TestXDGConfigHome_Ugly_NoHome
│
Service Tests:
├── service_test.go
│   ├── TestService_Start_Good
│   ├── TestService_Start_Bad_AlreadyStarted
│   └── TestService_Start_Ugly_PanicInCallback
└── service_example_test.go
```

### Test helpers

```go
// Mock medium for testing
mockMedium := coreio.NewMockMedium(coreio.Files{
    ".core/config.yaml": `dev:
  editor: vim`,
})

cfg, err := config.New(config.WithMedium(mockMedium))

// Mock filesystem watcher
mockWatcher := &MockWatcher{
    OnChange: func() { callback() },
}
```

### Running tests

```bash
# From go/ directory
cd go && go test ./...

# Run specific test
cd go && GOWORK=off go test -run TestConfig_Get_Good

# With race detector
cd go && go test -race ./...

# With coverage
cd go && go test -cover ./...

# Full QA pipeline
cd go && core go qa

# Full QA with extras
cd go && core go qa full
```

---

## Quality metrics

| Metric | Status | Details |
|--------|--------|---------|
| **Dual-Viper Pattern** | Complete | Prevents env variable leakage |
| **Hierarchical Discovery** | Complete | Walks up directory tree |
| **Manifest Support** | Complete | 6 manifest types |
| **Signature Verification** | Complete | ed25519 support |
| **Feature Flags** | Complete | Multi-source resolution |
| **Filesystem Watching** | Complete | Cross-platform support |
| **Storage Abstraction** | Complete | go-io Medium |
| **Core Service Integration** | Complete | Full lifecycle support |
| **XDG Compliance** | Complete | Platform-specific paths |
| **Test Coverage** | Complete | Good/Bad/Ugly pattern |
| **Production Ready** | Complete | Deployed in Core ecosystem |

---

## Related packages

### Dependencies

| Package | Module | Relationship |
|---------|--------|--------------|
| [go-io](../io/) | `dappco.re/go/io` | Storage abstraction (Medium interface) |
| [go-log](../log/) | `dappco.re/go/log` | Error handling (`coreerr.E()`) |
| Core Framework | `dappco.re/go` | Framework primitives (`core.Core`, `core.Startable`) |
| Viper | `github.com/spf13/viper` | Configuration engine |
| Testify | `github.com/stretchr/testify` | Test assertions |

### Consumers

All Core services and tools that need configuration:
- [go-agent](../agent/) — Agent configuration
- [go-api](../api/) — API server configuration
- [go-build](../build/) — Build configuration
- [go-cli](../cli/) — CLI configuration
- [go-gui](../gui/) — GUI configuration
- [go-ide](../ide/) — IDE configuration
- [go-ml](../ml/) — ML configuration

### Core framework

| Package | Relationship |
|---------|--------------|
| [CoreGO INDEX](../../INDEX.md) | Complete catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## References

1. **Repository** — [~/Code/core/config/](file:///Users/snider/Code/core/config/)
2. **CLAUDE.md** — [~/Code/core/config/CLAUDE.md](file:///Users/snider/Code/core/config/CLAUDE.md)
3. **AGENTS.md** — [~/Code/core/config/AGENTS.md](file:///Users/snider/Code/core/config/AGENTS.md)
4. **README.md** — [~/Code/core/config/README.md](file:///Users/snider/Code/core/config/README.md)
5. **Documentation** — [~/Code/core/config/docs/](file:///Users/snider/Code/core/config/docs/)
6. **Architecture** — [~/Code/core/config/docs/architecture.md](file:///Users/snider/Code/core/config/docs/architecture.md)
7. **Development Guide** — [~/Code/core/config/docs/development.md](file:///Users/snider/Code/core/config/docs/development.md)
8. **CoreGO INDEX** — [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md)
