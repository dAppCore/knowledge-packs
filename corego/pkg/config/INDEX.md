---
type: Package Index
package: config
module: dappco.re/go/config
repo: core/config
title: go-config Package Index
description: Core configuration module with dual-Viper pattern, hierarchical discovery, manifest resolution, and filesystem watching
tags:
  - config
  - configuration
  - yaml
  - viper
  - dual-viper
  - manifests
  - feature-flags
  - filesystem-watching
  - xdg
  - core-service
---
# go-config Package Index

> **Core Configuration Module — Unified configuration resolution for the Core ecosystem**

**Repository:** `core/config`  
**Module:** `dappco.re/go/config`  
**Status:** ✅ Production-Ready  
**License:** EUPL-1.2  
**Language:** Go 1.26+  
**Dependencies:** Core framework, go-io, Viper, testify  
**Last Updated:** 2026-06-18  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| CLAUDE.md | Development guidance | [~/Code/core/config/CLAUDE.md](file:///Users/snider/Code/core/config/CLAUDE.md) |
| AGENTS.md | Agent guidance | [~/Code/core/config/AGENTS.md](file:///Users/snider/Code/core/config/AGENTS.md) |
| Architecture | Design rationale | [~/Code/core/config/docs/architecture.md](file:///Users/snider/Code/core/config/docs/architecture.md) |
| Development Guide | Building, testing, standards | [~/Code/core/config/docs/development.md](file:///Users/snider/Code/core/config/docs/development.md) |

---

## 🎯 Package Overview

`go-config` is the **Core configuration module** that provides a unified, consistent way for Core services and command-line tools to resolve configuration from multiple sources. It is the foundational configuration layer used throughout the Core ecosystem.

### Design Philosophy

- **Dual-Viper Pattern** — Maintains two Viper instances to prevent environment variable leakage into saved files
- **Hierarchical Discovery** — Walks up directory tree to find `.core/config.yaml` files
- **Storage Abstraction** — All file I/O through `coreio.Medium` from go-io
- **Service Integration** — Runs as a Core service with full lifecycle management
- **Security-First** — Environment variables never persist to disk

### Configuration Resolution Priority

```
Lowest Priority ← Defaults → File → Environment Variables → Explicit Set() ← Highest Priority
```

---

## 🏗️ Architecture

### Dual-Viper Pattern

The core architectural innovation that sets go-config apart:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Dual-Viper Configuration Architecture                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Config Struct                                         │ │
│  │                                                                         │ │
│  │  ┌─────────────────────────┐      ┌─────────────────────────┐        │ │
│  │  │   v: Read View            │      │   f: Write View          │        │ │
│  │  │   (*viper.Viper)          │      │   (*viper.Viper)          │        │ │
│  │  │                              │      │                             │        │ │
│  │  │  Contains:                  │      │  Contains:               │        │ │
│  │  │  - File data               │      │  - File data            │        │ │
│  │  │  - Environment variables   │      │  - Explicit Set()       │        │ │
│  │  │  - Defaults                │      │  calls                   │        │ │
│  │  │                              │      │                             │        │ │
│  │  │  Used for:                  │      │  Used for:               │        │ │
│  │  │  - Get() calls              │      │  - Commit() calls        │        │ │
│  │  │  - All() calls              │      │  - Persistence           │        │ │
│  │  │  - Has() calls              │      │                             │        │ │
│  │  │  - Reads                   │      │                             │        │ │
│  │  └─────────────────────────┘      └─────────────────────────┘        │ │
│  │                                                                         │ │
│  │  ┌─────────────────────────────────────────────────────────────────┐ │ │
│  │  │                    CRITICAL INVARIANT                                │ │ │
│  │  │                                                                     │ │ │
│  │  │  Write operations: Set() → writes to BOTH v and f                   │ │ │
│  │  │  Read operations:  Get() → reads from v ONLY                         │ │ │
│  │  │  Persist operations: Commit() → writes from f ONLY                  │ │ │
│  │  │                                                                     │ │ │
│  │  │  This prevents CORE_CONFIG_* environment variables from             │ │ │
│  │  │  leaking into saved YAML configuration files.                        │ │ │
│  │  └─────────────────────────────────────────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Resolution Chain                                         │
│                                                                              │
│  Request: cfg.Get("server.port", &port)                                    │
│            ↓                                                                  │
│  1. Check v (read view) for "server.port"                                 │
│     ├─ Check explicit Set() calls                                           │
│     ├─ Check CORE_CONFIG_server__port environment variable                 │
│     ├─ Check .core/config.yaml file                                        │
│     └─ Check built-in defaults                                              │
│            ↓                                                                  │
│  2. If found in v, return value                                             │
│  3. If not found, return error                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Component Layers

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    go-config: Configuration Stack                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Core Configuration Layer                           │ │
│  │  Config type + Dual-Viper + Basic CRUD + Commit + Watch                │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Discovery Layer                                       │ │
│  │  Hierarchical file discovery + Standard path checking                    │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Manifest Layer                                        │ │
│  │  Typed manifest resolution + Schema validation + Signing               │ │
│  │  build.yaml | test.yaml | workspace.yaml | manifest.yaml |          │ │
│  │  agent.yaml | zone.yaml                                                │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Feature Flags Layer                                   │ │
│  │  Multi-source feature flag resolution                                    │ │
│  │  SetFeature() → Environment → Process Defaults → Config File           │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Watch Layer                                           │ │
│  │  Filesystem monitoring + Callback registration + Debouncing            │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Path Utilities Layer                                   │ │
│  │  Resolve + Join + Clean + Expand + XDG compliance                       │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                    Service Layer                                         │ │
│  │  Core service integration + Lifecycle management                        │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Repository Structure

```
go/
├── Core Configuration
│   ├── config.go                # Main Config type with dual-Viper implementation
│   ├── config_test.go           # Good/Bad/Ugly tests for Config
│   ├── config_example_test.go  # Usage examples
│   ├── service.go               # Core.ServiceRuntime wrapper
│   └── service_test.go          # Service tests
│
├── Discovery
│   ├── discover.go              # Config file discovery (hierarchical walk-up)
│   ├── discover_test.go         # Discovery tests
│   └── discover_example_test.go # Discovery examples
│
├── Resolution
│   ├── resolve.go               # Path and value resolution
│   ├── resolve_test.go          # Resolution tests
│   └── resolve_example_test.go  # Resolution examples
│
├── Manifests
│   ├── manifest.go              # Manifest resolution engine
│   ├── manifest_test.go         # Manifest tests
│   ├── schema/                  # Manifest schema definitions
│   │   ├── build.yaml           # Build manifest schema
│   │   ├── test.yaml            # Test manifest schema
│   │   ├── workspace.yaml       # Workspace manifest schema
│   │   ├── manifest.yaml        # Package manifest schema
│   │   ├── agent.yaml           # Agent manifest schema
│   │   └── zone.yaml            # Zone manifest schema
│   └── schema.go                # Schema utilities
│
├── Features
│   ├── feature.go               # Feature flag implementation
│   ├── feature_test.go          # Feature tests
│   └── feature_example_test.go  # Feature examples
│
├── Environment
│   ├── env.go                   # Environment variable handling
│   └── env_test.go              # Environment tests
│
├── Paths
│   ├── paths.go                 # Path manipulation utilities
│   ├── paths_test.go            # Path tests
│   ├── xdg.go                   # XDG Base Directory compliance
│   └── xdg_test.go              # XDG tests
│
├── Watching
│   ├── watch.go                 # Filesystem watch implementation
│   └── watch_test.go            # Watch tests
│
├── Workspace
│   ├── workspace.go             # Workspace utilities
│   └── workspace_test.go        # Workspace tests
│
├── Images
│   ├── images_manifest.go       # Docker image manifest support
│   └── images_manifest_test.go  # Images manifest tests
│
├── Conclave
│   ├── conclave.go              # Conclave configuration
│   └── conclave_test.go         # Conclave tests
│
└── Module
    ├── go.mod                   # Go module definition
    └── go.sum                   # Dependency checksums
```

---

## 🔌 Core Interfaces & Types

### Config Interface

```go
type Config interface {
    // Get retrieves a value by key
    Get(key string, target any) error
    
    // Set stores a value by key
    Set(key string, value any) error
    
    // All returns the entire configuration as a map
    All() map[string]any
    
    // Has checks if a key exists
    Has(key string) bool
    
    // Commit persists changes to storage
    Commit() error
    
    // Watch registers a callback for key changes
    Watch(key string, callback func()) error
    
    // Unwatch removes a previously registered callback
    Unwatch(key string, callback func()) error
}
```

### Config Type

```go
type Config struct {
    // Dual-Viper instances
    v *viper.Viper  // Read view: file + env + defaults + explicit Set()
    f *viper.Viper  // Write view: file + explicit Set() only
    
    // Configuration
    medium   coreio.Medium   // Storage abstraction (go-io)
    path     string           // Config file path
    envPrefix string          // Environment variable prefix (default: "CORE_CONFIG")
    
    // Watch state
    watchers map[string][]func()  // Registered callbacks
}

// Interface compliance assertion
var _ core.Config = (*Config)(nil)
```

### Service Type

```go
type Service struct {
    config *Config
    core.ServiceRuntime[Options]
}

// Lifecycle hooks
func (s *Service) OnStartup() error { ... }
func (s *Service) OnShutdown() error { ... }
```

---

## 🎯 Core Functionality

### 1. Dual-Viper Configuration

#### The Problem

Traditional configuration systems have a flaw: when you save configuration to a file, environment variables can accidentally leak into the saved file, causing unexpected behavior across different environments.

#### The Solution

Dual-Viper maintains two separate Viper instances:

```go
// v (read view): Contains everything
v := viper.New()
v.SetDefault("server.port", 8080)
v.SetConfigFile(".core/config.yaml")
v.ReadInConfig()
v.AutomaticEnv()  // Loads CORE_CONFIG_* variables

// f (write view): Contains only file + explicit Set()
f := viper.New()
f.SetConfigFile(".core/config.yaml")
f.ReadInConfig()

// When setting a value: write to BOTH
func (c *Config) Set(key string, value any) error {
    c.v.Set(key, value)  // Write to read view
    c.f.Set(key, value)  // Write to write view
    return nil
}

// When getting a value: read from v (includes env vars)
func (c *Config) Get(key string, target any) error {
    return c.v.UnmarshalKey(key, target)
}

// When committing: write from f (no env leakage)
func (c *Config) Commit() error {
    return c.f.WriteConfig()
}
```

#### Invariant Enforcement

```
✅ Write: Set() → BOTH v AND f
✅ Read:  Get() → v ONLY
✅ Save:  Commit() → f ONLY

❌ Write to only one Viper = BUG
❌ Read from f = Missing environment variables
❌ Save from v = Environment variable leakage
```

### 2. Hierarchical Discovery

#### Discovery Algorithm

```
Starting from: /project/subdir

Step 1: Check /project/subdir/.core/config.yaml
Step 2: Check /project/subdir/.core/
Step 3: Check /project/.core/config.yaml
Step 4: Check /project/.core/
Step 5: Check /project/.core/config.yaml
Step 6: Check ~/.core/config.yaml
Step 7: Check /etc/core/config.yaml
Step 8: Check /usr/local/etc/core/config.yaml

Found: /project/.core/config.yaml
Loading...
Continue to check for overrides in parent directories...
```

#### Code Usage

```go
// Auto-discover from current directory
cfg, err := config.New()

// Discover from specific project root
cfg, err := config.New(config.WithPath("/project/.core/config.yaml"))

// Discover with custom storage
cfg, err := config.New(
    config.WithMedium(coreio.NewS3Medium("my-bucket", "config/")),
    config.WithPath("config.yaml"),
)
```

### 3. Environment Variables

#### Default Prefix: CORE_CONFIG_

```bash
# Set configuration via environment
CORE_CONFIG_dev__editor=vim
CORE_CONFIG_server__port=8080
CORE_CONFIG_database__host=localhost
CORE_CONFIG_database__port=5432
```

**Mapping:** Double underscore (`__`) represents YAML nesting
- `CORE_CONFIG_dev__editor` → `dev.editor`
- `CORE_CONFIG_server__database__host` → `server.database.host`

#### Custom Prefix

```go
// Use custom environment prefix
cfg, err := config.New(config.WithEnvPrefix("MYAPP"))

# Now these variables are loaded:
MYAPP_dev__editor=vim
MYAPP_server__port=8080
```

#### Priority Override

```bash
# Environment variable overrides file value
CORE_CONFIG_server__port=9090

# File contains: server.port: 8080
# Result: cfg.Get("server.port") returns 9090
```

### 4. Manifest Resolution

#### Supported Manifest Types

| Manifest File | Purpose | Used By |
|---------------|---------|---------|
| `build.yaml` | Build configuration | go-build |
| `test.yaml` | Test configuration | go-config |
| `workspace.yaml` | Workspace layout | go-config |
| `manifest.yaml` | Package metadata | Various |
| `agent.yaml` | Agent configuration | go-agent |
| `zone.yaml` | Zone configuration | go-config |

#### Manifest Loading

```go
// Load a typed manifest
var buildConfig struct {
    Name    string
    Version string
    Go      struct {
        Version string
        Modules []string
    }
}

if err := cfg.Resolve("build.yaml", &buildConfig); err != nil {
    panic(err)
}
```

#### Manifest Signature Verification

```go
// Sign a manifest
privateKey := ed25519.New()
manifest := map[string]any{"name": "my-pkg", "version": "1.0.0"}
signature, err := config.Sign(manifest, privateKey)

// Verify a manifest
publicKey := privateKey.Public()
if err := config.Verify(manifest, signature, publicKey); err != nil {
    panic("Signature verification failed")
}
```

### 5. Feature Flags

#### Usage

```go
// Enable a feature
if err := cfg.SetFeature("experimental.ai", true); err != nil {
    panic(err)
}

// Check if feature is enabled
if cfg.IsEnabled("experimental.ai") {
    // Use experimental AI feature
}

// Get feature value with default
var enabled bool
if err := cfg.Feature("experimental.ai", &enabled); err != nil {
    enabled = false  // Default if not set
}
```

#### Resolution Priority

```
1. Explicit SetFeature() calls (highest)
2. CORE_FEATURE_* environment variables
3. Process-level defaults
4. Config file values (lowest)
```

Example:
```bash
# Enable via environment
CORE_FEATURE_experimental__ai=true

# Or set explicitly
cfg.SetFeature("experimental.ai", true)
```

### 6. Filesystem Watching

#### Watch API

```go
// Watch a specific key
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
}); err != nil {
    panic(err)
}

// Stop watching
if err := cfg.Unwatch("dev.editor", callback); err != nil {
    panic(err)
}
```

#### Implementation Details

- Uses platform-native filesystem notifications:
  - Linux: inotify
  - macOS: FSEvents
  - Windows: ReadDirectoryChangesW
- Callback is debounced to prevent rapid-fire notifications
- Automatically re-establishes watch after filesystem events

### 7. Path Utilities

#### Resolution

```go
// Resolve path (expand ~, env vars, relative)
resolved, err := config.Resolve("~/projects/my-app")
// Result: /home/user/projects/my-app

// Join paths
joined := config.Join("/path/to", "dir", "file.txt")
// Result: /path/to/dir/file.txt

// Clean path
cleaned := config.Clean("/path/to/../file.txt")
// Result: /path/file.txt

// Expand path
expanded, err := config.Expand("$HOME/projects/${USER}")
// Result: /home/user/projects/user
```

#### XDG Compliance

```go
// Get XDG config home
configHome, err := config.XDGConfigHome()
// Linux: ~/.config
// macOS: ~/Library/Application Support
// Windows: %APPDATA%

// Get XDG data home
dataHome, err := config.XDGDataHome()
// Linux: ~/.local/share
// macOS: ~/Library/Application Support
// Windows: %LOCALAPPDATA%

// Get XDG cache home
cacheHome, err := config.XDGCacheHome()
// Linux: ~/.cache
// macOS: ~/Library/Caches
// Windows: %LOCALAPPDATA%/Cache

// Get user home
home, err := config.UserDir()
// Result: /home/user or /Users/user
```

### 8. Service Integration

#### As a Core Service

```go
// Create service
svc, err := config.NewService(
    config.WithMedium(coreio.Local),
    config.WithPath(".core/config.yaml"),
)

// Start service (lifecycle hooks run automatically)
if err := svc.Start(); err != nil {
    panic(err)
}
defer svc.Stop()

// Use service like Config
var editor string
if err := svc.Get("dev.editor", &editor); err != nil {
    panic(err)
}
```

#### Service Lifecycle

```
OnStartup():
  1. Initialize Viper instances (v and f)
  2. Load configuration from medium
  3. Start filesystem watcher (if enabled)
  4. Register actions and commands

OnShutdown():
  1. Stop filesystem watcher
  2. Clean up resources
```

---

## 🚀 Usage Examples

### Basic Configuration

```go
package main

import (
    core "dappco.re/go"
    config "dappco.re/go/config"
    coreio "dappco.re/go/io"
)

func main() {
    // Create configuration
    cfg, err := config.New(
        config.WithMedium(coreio.Local),
        config.WithPath(".core/config.yaml"),
    )
    if err != nil {
        panic(err)
    }
    
    // Get a value
    var editor string
    if err := cfg.Get("dev.editor", &editor); err != nil {
        editor = "vim"  // Default
    }
    core.Println("Editor:", editor)
    
    // Set a value
    if err := cfg.Set("dev.editor", "nvim"); err != nil {
        panic(err)
    }
    
    // Persist changes
    if err := cfg.Commit(); err != nil {
        panic(err)
    }
}
```

### With Environment Variables

```bash
# Set via environment
export CORE_CONFIG_dev__editor=vim
export CORE_CONFIG_server__port=8080

# Go code
go run main.go
# Output: Editor: vim (from environment)
```

### Hierarchical Configuration

```
/project/
├── .core/
│   └── config.yaml
│       # contents: dev.editor: vim
│
└── subdir/
    └── app.go

# In app.go:
cfg, err := config.New()
// Discovers /project/.core/config.yaml
// cfg.Get("dev.editor") returns "vim"
```

### Manifest Usage

```go
// Load build manifest
var buildConfig struct {
    Name    string
    Version string
    Go      struct {
        Version string
        Tags    []string
    }
}

if err := cfg.Resolve("build.yaml", &buildConfig); err != nil {
    panic(err)
}

fmt.Printf("Building %s v%s with Go %s\n",
    buildConfig.Name,
    buildConfig.Version,
    buildConfig.Go.Version)
```

### Feature Flags

```go
// Check for experimental feature
if cfg.IsEnabled("experimental.ai") {
    // Use experimental AI
    ai.EnableExperimental()
}

// Enable via environment
// CORE_FEATURE_experimental__ai=true

// Or enable programmatically
cfg.SetFeature("experimental.ai", true)
```

### Filesystem Watch

```go
// Watch for config changes
if err := cfg.Watch("", func() {
    fmt.Println("Configuration changed!")
    all := cfg.All()
    // React to changes...
}); err != nil {
    panic(err)
}

// Keep program running
select {}
```

### Service Mode

```go
package main

import (
    core "dappco.re/go"
    config "dappco.re/go/config"
)

func main() {
    // Create and start service
    svc, err := config.NewService()
    if err != nil {
        panic(err)
    }
    
    // Start as a Core service
    if err := svc.Start(); err != nil {
        panic(err)
    }
    defer svc.Stop()
    
    // Use the service
    var port int
    if err := svc.Get("server.port", &port); err != nil {
        port = 8080
    }
    
    // Wait for shutdown
    core.Wait()
}
```

---

## 🧪 Testing

### Test Structure

All tests follow the AX Standard with Good/Bad/Ugly triplets:

```
├── config_test.go           # Config type tests
│   ├── TestConfig_New_Good
│   ├── TestConfig_New_Bad_InvalidOptions
│   ├── TestConfig_New_Ugly_NilMedium
│   ├── TestConfig_Get_Good_Found
│   ├── TestConfig_Get_Bad_NotFound
│   └── TestConfig_Get_Ugly_TypeMismatch
│
├── config_example_test.go  # Usage examples
├── service_test.go          # Service tests
├── discover_test.go         # Discovery tests
├── resolve_test.go          # Resolution tests
├── feature_test.go          # Feature flag tests
├── env_test.go              # Environment tests
├── paths_test.go            # Path tests
├── xdg_test.go              # XDG tests
├── watch_test.go            # Watch tests
└── manifest_test.go         # Manifest tests
```

### Mock Testing

```go
// Test with mock medium
mockFiles := coreio.Files{
    ".core/config.yaml": `dev:
  editor: vim
server:
  port: 8080`,
}
mockMedium := coreio.NewMockMedium(mockFiles)

cfg, err := config.New(config.WithMedium(mockMedium))

// Now cfg will return values from the mock
var editor string
if err := cfg.Get("dev.editor", &editor); err != nil {
    panic(err)
}
// editor == "vim"
```

### Running Tests

```bash
# From go/ directory
cd /Users/snider/Code/core/config/go && go test ./...

# Run specific test
cd go && GOWORK=off go test -run TestConfig_Get_Good

# With race detector
cd go && go test -race ./...

# With coverage
cd go && go test -cover ./... -coverprofile=coverage.out

# Full QA
cd go && core go qa

# Full QA with extras
cd go && core go qa full
```

---

## 📈 Quality Metrics

| Metric | Status | Details |
|--------|--------|---------|
| **Dual-Viper Pattern** | ✅ | Prevents env variable leakage to disk |
| **Hierarchical Discovery** | ✅ | Walks directory tree for config files |
| **Manifest Support** | ✅ | 6 manifest types with schema validation |
| **Signature Verification** | ✅ | ed25519 signing and verification |
| **Feature Flags** | ✅ | Multi-source resolution |
| **Filesystem Watching** | ✅ | Cross-platform (inotify, FSEvents, etc.) |
| **Storage Abstraction** | ✅ | go-io Medium interface |
| **Core Service Integration** | ✅ | Full lifecycle with Start/Stop |
| **XDG Compliance** | ✅ | Linux, macOS, Windows support |
| **Test Coverage** | ✅ | Good/Bad/Ugly pattern throughout |
| **Error Handling** | ✅ | Core error wrapping (coreerr.E) |
| **Production Ready** | ✅ | Deployed throughout Core ecosystem |

---

## 📝 Best Practices

### Always Use Dual-Viper Correctly

```go
// ✅ CORRECT
cfg.Set("key", value)   // Writes to BOTH v and f
cfg.Get("key", &value)  // Reads from v
cfg.Commit()           // Writes from f

// ❌ WRONG - Will cause env leakage
cfg.v.Set("key", value) // Only writes to read view
cfg.f.Set("key", value) // Only writes to write view

// ❌ WRONG - Will lose environment variables
cfg.Get("key", &value)  // Reads from f (missing env vars)
```

### Environment Variable Naming

```bash
# ✅ CORRECT - Double underscore for nesting
CORE_CONFIG_server__database__host=localhost

# ❌ WRONG - Single underscore (won't nest properly)
CORE_CONFIG_server_database_host=localhost

# ❌ WRONG - Missing prefix
server__database__host=localhost
```

### Configuration File Location

```
# ✅ CORRECT locations
./.core/config.yaml
~/.core/config.yaml
/etc/core/config.yaml

# ❌ WRONG - Not in standard locations
./config.yaml
./.config/config.yaml
/var/config.yaml
```

### Feature Flag Naming

```go
// ✅ CORRECT - Dot notation
"experimental.ai"
"beta.features.ml"
"debug.tracing"

// ❌ WRONG - Underscores (confusing with env vars)
"experimental_ai"
"beta_features_ml"
```

---

## 🔗 Related Packages

### Dependencies

| Package | Module | Relationship |
|---------|--------|--------------|
| [go-io](../io/) | `dappco.re/go/io` | Storage abstraction (Medium interface) |
| [go-log](../log/) | `dappco.re/go/log` | Error handling (`coreerr.E()` helper) |
| Core Framework | `dappco.re/go` | Framework primitives |
| Viper | `github.com/spf13/viper` | Configuration engine |
| Testify | `github.com/stretchr/testify` | Test assertions |

### Consumer Packages

All Core services that use configuration:

| Package | Module | Configuration Used For |
|---------|--------|---------------------|
| [go-agent](../agent/) | `dappco.re/go/agent` | Agent settings, dispatch config |
| [go-api](../api/) | `dappco.re/go/api` | Server ports, TLS, CORS |
| [go-build](../build/) | `dappco.re/go/build` | Build targets, flags, signing |
| [go-cli](../cli/) | `dappco.re/go/cli` | CLI defaults, themes |
| [go-config](../config/) | `dappco.re/go/config` | This package |
| [go-gui](../gui/) | `dappco.re/go/gui` | Window settings, themes |
| [go-ide](../ide/) | `dappco.re/go/ide` | IDE preferences, display modes |
| [go-ml](../ml/) | `dappco.re/go/core/ml` | ML backend settings |
| [go-inference](../inference/) | `dappco.re/go/inference` | Inference defaults |

### Core Framework

| Package | Relationship |
|---------|--------------|
| [CoreGO INDEX](../../INDEX.md) | Complete catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## 📚 References

1. **Repository** — [~/Code/core/config/](file:///Users/snider/Code/core/config/)
2. **CLAUDE.md** — [~/Code/core/config/CLAUDE.md](file:///Users/snider/Code/core/config/CLAUDE.md)
3. **AGENTS.md** — [~/Code/core/config/AGENTS.md](file:///Users/snider/Code/core/config/AGENTS.md)
4. **README.md** — [~/Code/core/config/README.md](file:///Users/snider/Code/core/config/README.md)
5. **Documentation** — [~/Code/core/config/docs/](file:///Users/snider/Code/core/config/docs/)
6. **Architecture** — [~/Code/core/config/docs/architecture.md](file:///Users/snider/Code/core/config/docs/architecture.md)
7. **Development Guide** — [~/Code/core/config/docs/development.md](file:///Users/snider/Code/core/config/docs/development.md)
8. **Index** — [~/Code/core/config/docs/index.md](file:///Users/snider/Code/core/config/docs/index.md)
9. **CoreGO INDEX** — [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md)

---

*Package index generated: 2026-06-18T00:30:00Z*
*Knowledge Pack: CoreGo v1.2.0*
*Module: dappco.re/go/config*
*Maintainer: Purberus <purberus@lthn.ai>*
