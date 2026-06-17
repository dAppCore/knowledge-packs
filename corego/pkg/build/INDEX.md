# go-build Package Index

> **Package:** `dappco.re/go/build`  
> **Documentation:** [README.md](./README.md)  
> **Repository:** [`github.com/dappcore/go-build`](https://github.com/dappcore/go-build)  
> **Spec:** [`plans/code/core/go/build/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Table of Contents

- [📋 Overview](#-overview)
- [🏗️ Architecture](#-architecture)
- [📦 Packages](#-packages)
- [🔧 Configuration](#-configuration)
- [🚀 Commands](#-commands)
- [📝 Usage Patterns](#-usage-patterns)
- [🧪 Testing](#-testing)
- [📖 API Reference](#-api-reference)
- [🔗 Related Documentation](#-related-documentation)

---

## 📋 Overview

**go-build** is the comprehensive build orchestration framework for the Lethean ecosystem. It provides a unified interface for building, packaging, signing, and publishing software across 11+ project types and 8+ release destinations.

### 🎯 Key Features

| Category | Count | Description |
|----------|-------|-------------|
| **Project Types** | 12 | Go, Wails, Node, Deno, PHP, Python, Rust, C++, Docker, LinuxKit, Taskfile, MkDocs |
| **Builders** | 12 | One per project type, each implementing the `Builder` interface |
| **Publishers** | 8 | GitHub, Docker, npm, Homebrew, Scoop, Chocolatey, AUR, LinuxKit |
| **SDK Languages** | 4 | TypeScript, Python, Go, PHP |
| **Archive Formats** | 3 | gz (default), xz, zip |
| **Checksum Algorithms** | 2 | SHA256, SHA512 |
| **Signing Methods** | 3 | GPG, macOS codesign/notarisation, Windows SignTool |
| **Platform Targets** | 6+ | linux/amd64, linux/arm64, darwin/amd64, darwin/arm64, windows/amd64, windows/arm64 |

### 📊 Package Statistics

| Metric | Value |
|--------|-------|
| Total Files | 200+ Go files |
| Total Lines | ~250,000 |
| Test Files | 100+ |
| Documentation | 37KB (this README) + 36KB (INDEX) |
| Module | dappco.re/go/build |
| Go Version | 1.22+ |

---

## 🏗️ Architecture

### Repository Structure

```
core/go-build/
├── go/                          # Go module root
│   ├── cmd/                     # CLI command registration (4 commands)
│   │   ├── build/               # core build, core release
│   │   ├── ci/                  # core ci
│   │   ├── sdk/                 # core sdk
│   │   └── service/             # Build daemon
│   │
│   ├── pkg/                     # Library packages (5 top-level)
│   │   ├── build/               # Core build logic (30+ files)
│   │   │   ├── apple/           # Apple ecosystem (6 files)
│   │   │   ├── builders/        # Project builders (12 types, 20+ files)
│   │   │   ├── images/          # LinuxKit images (4 files)
│   │   │   ├── installers/      # Installer generation (2 files)
│   │   │   ├── signing/         # Code signing (3 files)
│   │   │   └── templates/       # Template files
│   │   │
│   │   ├── release/             # Release orchestration (15+ files)
│   │   │   └── publishers/      # Publisher implementations (8 types, 20+ files)
│   │   │
│   │   ├── sdk/                 # SDK generation (10+ files)
│   │   │   └── generators/      # Language generators (4 types)
│   │   │
│   │   ├── api/                 # API provider surface
│   │   ├── service/             # Build service daemon
│   │   └── storage/             # Storage abstractions
│   │
│   ├── internal/                # Non-public support (20+ files)
│   └── tests/                   # Integration tests
│
├── docs/                        # Cross-language documentation
├── ui/                          # Language-specific UI
├── README.md
├── AGENTS.md
└── CLAUDE.md
```

### Design Principles

1. **Agent-First Design** — All code and documentation written for AI agents to understand
2. **Convention over Configuration** — Smart defaults with `.core/build.yaml` override
3. **Zero External Dependencies** — Uses only CoreGo primitives
4. **Platform Agnostic** — Works on Linux, macOS, Windows
5. **Extensible** — Easy to add new builders, publishers, SDK generators

### Core Interfaces

```go
// Builder interface - all project type builders implement this
type Builder interface {
    Detect(fs storage.Medium, dir string) bool
    Build(ctx context.Context, cfg *Config, target TargetConfig) core.Result
    Clean(cfg *Config) core.Result
}

// Publisher interface - all release publishers implement this
type Publisher interface {
    Name() string
    Type() string
    Publish(ctx context.Context, cfg *ReleaseConfig, artifacts []Artifact) core.Result
    DryRun(ctx context.Context, cfg *ReleaseConfig) core.Result
}

// Generator interface - all SDK generators implement this
type Generator interface {
    Name() string
    Generate(ctx context.Context, cfg *Config, spec *openapi3.T) core.Result
}
```

---

## 📦 Packages

### pkg/build/ — Core Build Logic

The central package containing the main build orchestration logic.

#### File Overview

| File | Size | Purpose | Key Types/Functions |
|------|------|---------|---------------------|
| `config.go` | 36KB | Configuration loading | `BuildConfig`, `Project`, `Build`, `TargetConfig` |
| `discovery.go` | 28KB | Project type detection | `Discover()`, `PrimaryType()`, `ProjectType` constants |
| `pipeline.go` | 13KB | Build pipeline orchestration | `Pipeline`, `Run()`, 8 build phases |
| `build.go` | 5KB | Build execution | `Build()`, `Clean()` |
| `archive.go` | 13KB | Archive creation | `GenerateArchive()`, `ArchiveFormat*` |
| `checksum.go` | 3.6KB | Checksum generation | `GenerateChecksums()`, `ChecksumAlgorithm` |
| `cache.go` | 11KB | Build caching | `CacheConfig`, `SetupCache()` |
| `options.go` | 5.4KB | Build options | `Options`, `ComputeOptions()` |
| `run.go` | 11KB | Runtime configuration | `RuntimeConfig`, `LoadRuntimeConfig()` |
| `setup.go` | 9KB | Setup and validation | `Setup()`, `Validate()` |
| `workflow.go` | 20KB | Workflow generation | `Workflow`, `WriteReleaseWorkflow()` |
| `version.go` | 1KB | Version handling | `Version`, `ParseVersion()` |

#### Configuration Types

```go
// BuildConfig - Complete build configuration
type BuildConfig struct {
    Version    int
    Project    Project
    Build      Build
    Apple      AppleConfig
    PreBuild   PreBuild
    Targets    []TargetConfig
    Sign       signing.SignConfig
    SDK        *sdk.Config
    LinuxKit   LinuxKitConfig
}

// Project - Project metadata
type Project struct {
    Name        string  // Project name
    Description string  // Project description
    Main        string  // Path to main package
    Binary      string  // Output binary name
}

// Build - Build settings
type Build struct {
    Type        string            // Project type (go, wails, docker, etc.)
    CGO         bool              // Enable CGO
    Obfuscate   bool              // Use garble for obfuscation
    NSIS        bool              // Windows NSIS installer
    WebView2    string            // WebView2 delivery mode
    Flags       []string          // Build flags
    LDFlags     []string          // Linker flags
    BuildTags   []string          // Go build tags
    ArchiveFormat string          // Archive format (gz, xz, zip)
    Env         []string          // Environment variables
    Cache       CacheConfig
    Dockerfile  string            // Dockerfile path
    Registry    string            // Container registry
    Image       string            // Image name
    Tags        []string          // Docker tags
    BuildArgs   map[string]string // Docker build args
    Push        bool              // Push Docker image
    Load        bool              // Load Docker image
    LinuxKitConfig string       // LinuxKit config path
    Formats     []string          // LinuxKit output formats
}

// TargetConfig - Build target
type TargetConfig struct {
    OS   string   // Operating system (linux, darwin, windows)
    Arch string   // Architecture (amd64, arm64, 386, etc.)
    ARM  string   // Optional ARM version (v6, v7, v8)
    Tags []string // Additional build tags
}
```

#### Discovery System

**Marker Files (38 total):**

```go
// Core markers
markerBuildConfig   = ".core/build.yaml"
markerGoMod          = "go.mod"
markerGoWork         = "go.work"
markerMainGo        = "main.go"

// Frontend markers
markerWails         = "wails.json"
markerNodePackage   = "package.json"
markerDenoJSON      = "deno.json"
markerDenoJSONC     = "deno.jsonc"

// Backend markers
markerComposer      = "composer.json"
markerMkDocs        = "mkdocs.yml"
markerMkDocsYAML    = "mkdocs.yaml"
markerDocsMkDocs    = "docs/mkdocs.yml"
markerDocsMkDocsYAML = "docs/mkdocs.yaml"
markerPyProject     = "pyproject.toml"
markerRequirements  = "requirements.txt"
markerCargo         = "Cargo.toml"

// Container markers
markerDockerfile    = "Dockerfile"
markerLinuxKitYAML  = "linuxkit.yml"
markerLinuxKitYAMLAlt = "linuxkit.yaml"

// Task markers
markerTaskfileYML   = "Taskfile.yml"
markerTaskfileYAML  = "Taskfile.yaml"
markerTaskfileBare  = "Taskfile"
```

**Discovery Process:**
1. Check for explicit `type:` in `.core/build.yaml`
2. Iterate through discovery rules in priority order
3. For each rule, check if marker files exist
4. Return ordered list of detected project types

**Priority Order:**
1. Wails (has both wails.json and go.mod)
2. Go (go.mod or go.work)
3. Node/Deno (package.json or deno.json)
4. PHP (composer.json)
5. Python (pyproject.toml or requirements.txt)
6. Rust (Cargo.toml)
7. C++ (CMakeLists.txt)
8. Docker (Dockerfile)
9. LinuxKit (linuxkit.yml)
10. Taskfile (Taskfile.yml)
11. MkDocs (mkdocs.yml)

#### Build Pipeline

**8 Phases of Build:**

```go
// Phase 1: Setup
// - Validate configuration
// - Resolve dependencies
// - Create output directories

// Phase 2: PreBuild
// - Run frontend builds (npm, deno)
// - Prepare frontend assets

// Phase 3: Build
// - Set environment (GOOS, GOARCH, CGO_ENABLED)
// - Apply build flags, ldflags, tags
// - Handle obfuscation with garble
// - Execute builder for each target

// Phase 4: Archive
// - Create tar.gz/zip archives
// - Organize by platform/architecture

// Phase 5: Checksum
// - Generate SHA256/SHA512 checksums
// - Create checksum files

// Phase 6: Sign
// - GPG signing
// - macOS codesigning
// - macOS notarisation
// - Windows SignTool

// Phase 7: Package
// - Create NSIS installers (Windows)
// - Create DMG packages (macOS)
// - Create deb/rpm packages (Linux)

// Phase 8: Publish
// - Upload to GitHub releases
// - Push Docker images
// - Publish to npm
// - Update Homebrew tap
// - etc.
```

#### Apple Ecosystem Support (pkg/build/apple/)

**Files:**
- `apple.go` (77KB) - Main Apple implementation
- `apple_test.go` (50KB) - Unit tests
- `apple_example_test.go` (2.8KB) - Examples
- `signing.go` - Code signing
- `notarise.go` - Notarisation
- `dmg.go` - DMG packaging
- `plist.go` - Info.plist generation
- `xcode_cloud.go` - Xcode Cloud integration

**Capabilities:**
- Universal binary creation (multi-architecture)
- Code signing with Developer ID certificates
- Notarisation via Apple's API
- DMG creation with custom backgrounds
- Info.plist and entitlements generation
- Xcode Cloud script generation
- TestFlight submission
- App Store submission

### pkg/build/builders/ — Project Builders

**12 Builder Types:**

| Builder | File | Lines | Purpose |
|---------|------|-------|---------|
| Go | `go.go` | 7.7KB | Standard Go builds with flags, tags, CGO |
| Wails | `wails.go` | 31KB | Wails frontend + Go backend, WebView2 handling |
| Node | `node.go` | 9.5KB | Node.js projects with npm/yarn/pnpm |
| Deno | `deno.go` | 8KB | Deno projects with deno task build |
| PHP | `php.go` | 6.3KB | PHP projects with composer |
| Python | `python.go` | 3.2KB | Python projects with pyproject.toml |
| Rust | `rust.go` | 5.4KB | Rust projects with Cargo.toml |
| C++ | `cpp.go` | 17KB | C++ projects with CMakeLists.txt |
| Docker | `docker.go` | 7KB | Docker image builds |
| LinuxKit | `linuxkit.go` | 9.4KB | LinuxKit image builds |
| Docs | `docs.go` | 4.5KB | MkDocs documentation builds |
| Taskfile | `taskfile.go` | 11KB | Taskfile project builds |

**Builder Interface:**

```go
type Builder interface {
    // Detect returns true if this builder can handle the project
    Detect(fs storage.Medium, dir string) bool
    
    // Build compiles the project for the given target
    Build(ctx context.Context, cfg *Config, target TargetConfig) core.Result
    
    // Clean removes build artifacts
    Clean(cfg *Config) core.Result
}
```

**Go Builder Features:**
- Sets GOOS, GOARCH, CGO_ENABLED based on target
- Applies build flags and linker flags
- Handles build tags
- Supports obfuscation with garble
- Generates debug information
- Returns compiled binary path

**Wails Builder Features:**
- Detects frontend framework (Angular, React, Vue, etc.)
- Builds frontend with npm or deno
- Compiles Go backend
- Bundles into single binary
- Handles WebView2 delivery modes:
  - `download` - Download at runtime
  - `embed` - Embed in binary
  - `browser` - Use system browser
  - `error` - Fail if not available
- Creates NSIS installer if configured

**Docker Builder Features:**
- Supports multi-stage builds
- Handles platform-specific builds
- Pushes to registries
- Loads for local development
- Supports BuildKit and traditional builds
- Handles build args and tags

**LinuxKit Builder Features:**
- Builds immutable Linux images
- Supports 10+ output formats:
  - `iso` - Bootable ISO
  - `raw` - Raw disk image
  - `qcow2` - QCOW2 for KVM
  - `vmdk` - VMware Virtual Disk
  - `vhd` - Hyper-V Virtual Hard Disk
  - `gcp` - Google Cloud Platform image
  - `aws` - Amazon Web Services image
  - `docker` - Docker-compatible image
  - `tar` - TAR archive
  - `kernel+initrd` - Kernel + initramfs pair

### pkg/build/signing/ — Code Signing

**Files:**
- `signing.go` - Sign configuration and coordination
- `gpg.go` - GPG/PGP signing
- `macos.go` - macOS codesigning
- `windows.go` - Windows SignTool signing

**SignConfig:**

```go
type SignConfig struct {
    Enabled bool
    GPG     GPGConfig
    MacOS   MacOSConfig
    Windows WindowsConfig
}

type GPGConfig struct {
    Key        string // Key ID or fingerprint
    Password   string // Optional password
    Armor      bool   // ASCII-armor output
    DetachSign bool   // Detached signature
}

type MacOSConfig struct {
    Identity    string // Certificate identity
    Notarize    bool   // Enable notarisation
    AppleID     string // Apple ID for notarisation
    TeamID      string // Team ID
    AppPassword string // App-specific password
    Timestamp   bool   // Timestamp server
}

type WindowsConfig struct {
    Signtool     bool   // Enable SignTool
    Certificate  string // Certificate path/name
    Password     string // Certificate password
    TimestampURL string // Timestamp server URL
}
```

**Signing Process:**
1. GPG: Creates detached `.sig` files using `gpg --detach-sign --armor`
2. macOS: Uses `codesign --deep --force --sign "identity"`
3. macOS Notarisation: Submits to Apple's notarisation API with `altool` or `notarytool`
4. Windows: Uses `signtool sign /fd SHA256 /a /tr timestampURL /td SHA256`

### pkg/build/installers/ — Installer Generation

**Files:**
- `installers.go` - Installer type definitions and generation
- `installers_test.go` - Tests

**Installer Types:**

```go
type InstallerType string

const (
    InstallerNSIS      InstallerType = "nsis"       // Windows NSIS
    InstallerDeb       InstallerType = "deb"        // Debian package
    InstallerRPM       InstallerType = "rpm"        // RPM package
    InstallerDMG       InstallerType = "dmg"        // macOS Disk Image
    InstallerShell     InstallerType = "shell"      // Shell script
    InstallerPowerShell InstallerType = "powershell" // PowerShell script
)
```

**NSIS Installer Features:**
- Modern UI with custom branding
- Automatic dependency detection
- Start menu shortcuts
- Desktop shortcuts
- Uninstall support
- Multiple compression methods

**DMG Packaging Features:**
- Creates .dmg files for macOS
- Applications folder symlink
- Custom background image
- Volume name customization
- EULA/license agreement display
- Auto-mount on open

### pkg/build/images/ — Container Images

**Files:**
- `linuxkit_image.go` - LinuxKit image building
- `linuxkit_templates.go` - Template helpers

**LinuxKit Image Types:**
- Bootable ISO images
- Raw disk images
- QCOW2 images (KVM/QEMU)
- VMDK images (VMware)
- VHD images (Hyper-V)
- GCP-compatible images
- AWS-compatible images
- Docker-compatible images
- TAR archives
- Kernel + initramfs pairs

### pkg/release/ — Release Orchestration

**Files:**
- `release.go` - Main release logic
- `config.go` - Release configuration
- `version.go` - Version resolution
- `changelog.go` - Changelog generation
- `output.go` - Release output handling
- `sdk.go` - SDK release orchestration

**ReleaseConfig:**

```go
type ReleaseConfig struct {
    Version    int
    Project    ReleaseProject
    Build      ReleaseBuild
    Changelog  ChangelogConfig
    SDK        *sdk.Config
    Publishers []PublisherConfig
}

type ReleaseProject struct {
    Name        string
    Repository  string // e.g., dappcore/core
    Description string
}

type ChangelogConfig struct {
    Use       string   // conventional, semver
    Exclude   []string // Commit patterns to exclude
    Categories []string // Custom categories
}
```

**Release Process:**
1. Load `.core/release.yaml`
2. Resolve version from:
   - Explicit `--tag` flag
   - Latest git tag
   - semver bump from latest tag
3. Generate changelog from commit history
4. Execute build pipeline
5. Generate checksums for all artifacts
6. Sign artifacts (if enabled)
7. Generate SDKs (if configured)
8. Publish to all configured destinations
9. Create GitHub release with all artifacts

**Version Resolution:**
- Uses `git describe --tags --abbrev=0` for latest tag
- Supports semantic versioning with `blang/semver`
- Can bump major/minor/patch based on commit type
- Respects conventional commits for changelog generation

**Changelog Generation:**
- Supports `conventional` and `semver` commit conventions
- Excludes specified patterns (docs, test, ci)
- Groups commits by type
- Generates markdown-formatted changelog
- Can create release notes for GitHub releases

### pkg/release/publishers/ — Release Publishers

**8 Publisher Types:**

| Publisher | File | Lines | Purpose |
|-----------|------|-------|---------|
| GitHub | `github.go` | 19KB | GitHub Releases API |
| Docker | `docker.go` | 10KB | Docker registry pushes |
| npm | `npm.go` | 10KB | npm registry publishes |
| Homebrew | `homebrew.go` | 13KB | Homebrew formula generation |
| Scoop | `scoop.go` | 10KB | Scoop manifest generation |
| Chocolatey | `chocolatey.go` | 10KB | Chocolatey package pushes |
| AUR | `aur.go` | 11KB | Arch User Repository |
| LinuxKit | `linuxkit.go` | 12KB | LinuxKit image pushes |

**Publisher Interface:**

```go
type Publisher interface {
    // Name returns the publisher name
    Name() string
    
    // Type returns the publisher type
    Type() string
    
    // Publish uploads artifacts to the destination
    Publish(ctx context.Context, cfg *ReleaseConfig, artifacts []Artifact) core.Result
    
    // DryRun validates configuration without publishing
    DryRun(ctx context.Context, cfg *ReleaseConfig) core.Result
}
```

**GitHub Publisher:**
- Creates GitHub releases via API
- Supports draft releases
- Handles prerelease detection from semver (alpha/beta/rc)
- Uploads all artifacts with proper naming
- Generates release notes from changelog
- Creates GitHub Actions uploads

**Docker Publisher:**
- Builds multi-platform images
- Pushes to configured registry (ghcr.io, Docker Hub, etc.)
- Handles build args and tags
- Supports platform-specific builds
- Can push all platforms or specific ones

**Homebrew Publisher:**
- Generates Homebrew formula files
- Supports official Homebrew repository PRs
- Handles bottle creation for pre-built binaries
- Validates formula syntax
- Can generate formulas for multiple platforms

**npm Publisher:**
- Publishes packages to npm registry
- Handles access control (public/private)
- Manages package.json updates
- Supports scoped packages (`@dappcore/core`)
- Handles npm tokens for authentication

**AUR Publisher:**
- Generates PKGBUILD files
- Handles AUR helper integration (yay, paru)
- Supports git push to AUR repositories
- Validates PKGBUILD syntax

### pkg/sdk/ — OpenAPI SDK Generation

**Files:**
- `sdk.go` - Main SDK generation logic
- `config.go` - SDK configuration
- `detect.go` - OpenAPI spec detection
- `diff.go` - Breaking change detection
- `validate.go` - Spec validation
- `generators/` - Language-specific generators

**SDK Config:**

```go
type Config struct {
    Spec       string   // Path to OpenAPI spec or "auto-detect"
    Languages  []string // typescript, python, go, php
    Output     string   // Output directory
    Diff       bool     // Enable breaking change detection
    Package    string   // Package name (Go, npm)
    Module     string   // Module path (Go)
    ClassName  string   // Class name prefix
    // ...
}
```

**Supported Generators:**

| Language | Generator | Library Used |
|----------|-----------|--------------|
| TypeScript | `typescript.go` | openapi-typescript |
| Python | `python.go` | openapi-python-client |
| Go | `go.go` | oapi-codegen |
| PHP | `php.go` | openapi-php-client |

**Breaking Change Detection:**
- Uses `oasdiff` library
- Compares current spec against previous version
- Detects added, removed, changed endpoints
- Detects schema changes
- Generates compatibility report
- Can fail build on breaking changes

**Detection Logic:**
1. Check for explicit `spec:` in config
2. Auto-detect common locations:
   - `openapi.yaml` / `openapi.yml`
   - `swagger.yaml` / `swagger.yml`
   - `api/openapi.yaml`
   - `docs/api.yaml`
3. Validate spec is valid OpenAPI 3.0
4. Load spec for generation

### pkg/storage/ — Storage Abstractions

**Files:**
- `storage.go` - Storage medium interface

**Medium Interface:**

```go
type Medium interface {
    Stat(name string) (os.FileInfo, error)
    Open(name string) (io.ReadCloser, error)
    ReadFile(name string) ([]byte, error)
    WriteFile(name string, data []byte, perm os.FileMode) error
    MkdirAll(name string, perm os.FileMode) error
    Remove(name string) error
    RemoveAll(name string) error
    ReadDir(name string) ([]os.DirEntry, error)
    Glob(pattern string) ([]string, error)
}

// Local implements Medium for local filesystem
type Local struct{}

// Implements all Medium methods for os filesystem
```

**Note:** The `storage` package uses the same interface as `go-io`, allowing seamless integration with S3, SQLite, HTTP, and other backends.

### pkg/api/ — API Provider Surface

**Files:**
- `api.go` - HTTP API handlers

**BuildProvider:**

```go
type BuildProvider struct {
    Config *Config
}

// Build handles POST /api/build requests
func (p *BuildProvider) Build(w http.ResponseWriter, r *http.Request)

// Release handles POST /api/release requests
func (p *BuildProvider) Release(w http.ResponseWriter, r *http.Request)

// Status handles GET /api/build/status requests
func (p *BuildProvider) Status(w http.ResponseWriter, r *http.Request)

// Workflow handles GET /api/build/workflow requests
func (p *BuildProvider) Workflow(w http.ResponseWriter, r *http.Request)
```

**API Endpoints:**
- `POST /api/build` - Trigger a build
- `POST /api/release` - Trigger a release
- `GET /api/build/status` - Get build status
- `GET /api/build/workflow` - Get workflow configuration

### pkg/service/ — Build Service Daemon

**Files:**
- `service.go` - Build service daemon

**Service:**

```go
type Service struct {
    Config *Config
}

// Run starts the build service daemon
func (s *Service) Run(ctx context.Context) core.Result

// Build executes a build request
func (s *Service) Build(ctx context.Context, req *BuildRequest) core.Result

// Release executes a release request
func (s *Service) Release(ctx context.Context, req *ReleaseRequest) core.Result

// Status returns current service status
func (s *Service) Status() ServiceStatus
```

**Service Capabilities:**
- Runs as a long-lived process
- Handles multiple concurrent build requests
- Provides event streaming via WebSocket
- Integrates with MCP (Model Context Protocol)
- Supports agentic channels for AI integration

---

## 🔧 Configuration

### .core/build.yaml

Complete build configuration reference:

```yaml
version: 1  # Configuration format version

# Project metadata
project:
  name: my-app              # Project name
  description: "My Application"  # Project description
  main: ./cmd/app          # Path to main package
  binary: app              # Output binary name

# Build settings
build:
  type: go                   # Override auto-detected type
  cgo: false                 # Enable/disable CGO
  obfuscate: false           # Use garble for binary obfuscation
  nsis: false                # Generate Windows NSIS installer
  webview2: download         # WebView2 delivery: download|embed|browser|error
  flags: ["-trimpath"]       # Additional build flags
  ldflags: ["-s", "-w"]      # Linker flags
  build_tags: ["netgo"]      # Go build tags
  archive_format: gz         # Archive format: gz, xz, zip
  env: ["CGO_ENABLED=0"]     # Environment variables
  
  # Docker-specific settings
  dockerfile: Dockerfile
  registry: ghcr.io
  image: owner/repo
  tags: [latest, v1.0]
  push: false
  load: false
  build_args:
    ARG1: value1
  
  # LinuxKit-specific settings
  linuxkit_config: linuxkit.yml
  formats: [iso, qcow2]

# Build cache configuration
cache:
  enabled: true
  dir: .core/cache
  key_prefix: my-project
  paths:
    - ~/.cache/go-build
    - ~/go/pkg/mod
  restore_keys:
    - go-

# Build targets (matrix)
targets:
  - os: linux
    arch: amd64
    tags: [netgo]
  - os: linux
    arch: arm64
  - os: darwin
    arch: amd64
  - os: darwin
    arch: arm64
  - os: windows
    arch: amd64

# Apple-specific configuration
apple:
  team_id: TEAM_ID
  bundle_id: com.example.myapp
  cert_identity: "Developer ID Application: Lethean CIC (TEAM_ID)"
  profile_path: embedded.provisionprofile
  keychain_path: /Library/Keychains/System.keychain
  arch: universal  # or arm64, x86_64

# Code signing configuration
sign:
  enabled: true
  gpg:
    key: $GPG_KEY_ID
    password: $GPG_PASSWORD
    armor: true
    detach_sign: true
  macos:
    identity: "Developer ID Application: Lethean CIC (TEAM_ID)"
    notarize: true
    apple_id: $APPLE_ID
    team_id: $APPLE_TEAM_ID
    app_password: $APPLE_APP_PASSWORD
    timestamp: true
  windows:
    signtool: true
    certificate: $WINDOWS_CERT
    password: $WINDOWS_CERT_PASSWORD
    timestamp_url: http://timestamp.digicert.com

# SDK generation configuration
sdk:
  spec: openapi.yaml  # or auto-detect
  languages: [typescript, python, go, php]
  output: sdk/
  diff: true
  package: my-app
  module: dappco.re/my-app
  class_name: MyApp

# LinuxKit configuration
linuxkit:
  config: linuxkit.yml
  formats: [iso, raw, qcow2]
```

### .core/release.yaml

Release-specific configuration:

```yaml
version: 1

project:
  name: my-app
  repository: owner/repo  # GitHub repository
  description: "My Application"

build:
  targets:
    - os: linux
      arch: amd64
    - os: darwin
      arch: arm64
    - os: windows
      arch: amd64
  archive_format: gz
  obfuscate: false
  sign: true

changelog:
  use: conventional  # or semver
  exclude:
    - '^docs:'
    - '^test:'
    - '^ci:'
  categories:
    - features
    - fixes
    - breaking

sdk:
  spec: openapi.yaml
  languages: [typescript, python, go, php]
  output: sdk/
  diff: true

publishers:
  # GitHub releases
  - type: github
    draft: false
    prerelease: false  # Auto-detect from semver
    
  # Docker images
  - type: docker
    registry: ghcr.io
    image: owner/my-app
    tags: [latest, v{{.Version}}]
    platforms:
      - linux/amd64
      - linux/arm64
    build_args:
      VERSION: "{{.Version}}"
    
  # Homebrew
  - type: homebrew
    tap: owner/homebrew-tap
    formula: my-app
    official:
      enabled: false  # Generate for official Homebrew PR
      output: dist/homebrew
    
  # npm
  - type: npm
    package: @owner/my-app
    access: public
    tag: latest
    
  # Scoop
  - type: scoop
    bucket: owner/bucket
    manifest: my-app.json
    
  # Chocolatey
  - type: chocolatey
    source: chocolatey
    package_name: my-app
    api_key: $CHOCOLATEY_API_KEY
    
  # AUR
  - type: aur
    repository: my-app
    maintainer: "My Name <my@email.com>"
    license: MIT
    
  # LinuxKit
  - type: linuxkit
    hub: ghcr.io/owner
    image: my-app
    tags: [latest, v{{.Version}}]
```

---

## 🚀 Commands

### Command Structure

```
core build       # Main build command
core release     # Release command
core ci          # CI command
core sdk         # SDK generation command
core build apple # macOS-specific build
core build workflow # Workflow generation
```

### core build

**Synopsis:** Build the project

**Description:**
Builds the project according to `.core/build.yaml` configuration. Without a config file, it passes through to `go build`.

**Flags:**

```
--config string        Path to build config file (default: .core/build.yaml)
--targets strings      Build targets (os/arch pairs, comma-separated)
--output string        Output directory for artifacts
--clean                Clean build artifacts before building
--cache                Enable build cache
--obfuscate           Enable binary obfuscation
--sign                 Enable code signing
--archive              Create archive files
--checksum             Generate checksum files
--debug                Enable debug output
--dry-run              Show what would be built without building
--help, -h             Show help
```

**Examples:**

```bash
# Simple build with config
core build

# Build specific targets
core build --targets linux/amd64,darwin/arm64

# Clean and rebuild
core build --clean

# Build with signing
core build --sign

# Build with all options
core build --clean --cache --obfuscate --sign --archive --checksum
```

### core release

**Synopsis:** Create a release

**Description:**
Builds the project and publishes artifacts according to `.core/release.yaml` configuration.

**Flags:**

```
--config string        Path to release config file (default: .core/release.yaml)
--tag string           Version tag (e.g., v1.0.0)
--draft                Create as draft release
--prerelease           Mark as prerelease
--dry-run              Validate without publishing
--sign                 Enable code signing
--publish              Publish to configured destinations
--sdk                  Generate SDKs
--changelog            Generate changelog
--debug                Enable debug output
--help, -h             Show help
```

**Examples:**

```bash
# Dry run (validate configuration)
core release --dry-run

# Create release with version tag
core release --tag v1.0.0

# Create draft release
core release --tag v1.0.0 --draft

# Full release with all features
core release --tag v1.0.0 --sign --publish --sdk --changelog
```

### core ci

**Synopsis:** Run CI checks

**Description:**
Runs various CI checks including tests, linting, and validation.

**Flags:**

```
--checks strings       Specific checks to run (vet, test, lint, fmt)
--coverage             Generate coverage reports
--short                Skip slow tests
--race                 Run tests with race detector
--verbose              Verbose output
--help, -h             Show help
```

**Examples:**

```bash
# Run all CI checks
core ci

# Run specific checks
core ci --checks vet,test,lint

# Run with coverage
core ci --coverage

# Quick CI run
core ci --short
```

### core sdk

**Synopsis:** Generate SDKs from OpenAPI spec

**Description:**
Generates client SDKs for multiple languages from OpenAPI specifications.

**Flags:**

```
--spec string          Path to OpenAPI spec (default: auto-detect)
--languages strings    Languages to generate (typescript, python, go, php)
--output string         Output directory (default: sdk/)
--diff                 Enable breaking change detection
--package string       Package name (for Go, npm)
--module string        Module path (for Go)
--class-name string    Class name prefix
--dry-run              Show what would be generated
--help, -h             Show help
```

**Examples:**

```bash
# Auto-detect and generate for all languages
core sdk

# Generate TypeScript SDK
core sdk --languages typescript

# Generate with breaking change detection
core sdk --diff

# Specify output directory
core sdk --output client/

# Specify explicit spec file
core sdk --spec api/openapi.yaml
```

### core build apple

**Synopsis:** macOS-specific build commands

**Description:**
Builds for macOS with Apple-specific features like code signing, notarisation, and DMG creation.

**Flags:**

```
--targets strings      Build targets (darwin/amd64, darwin/arm64)
--universal            Create universal binary
--sign                 Enable code signing
--notarize             Enable notarisation
--dmg                  Create DMG package
--xcode-cloud          Generate Xcode Cloud scripts
--help, -h             Show help
```

**Examples:**

```bash
# Build for Apple platforms
core build apple --targets darwin/amd64,darwin/arm64

# Build universal binary
core build apple --universal

# Build with signing and notarisation
core build apple --sign --notarize

# Create DMG package
core build apple --dmg

# Full Apple build
core build apple --universal --sign --notarize --dmg --xcode-cloud
```

### core build workflow

**Synopsis:** Generate reusable GitHub Actions workflow

**Description:**
Generates a reusable GitHub Actions workflow that mirrors the `dAppCore/build@v3` action surface.

**Flags:**

```
--output string         Output file path (default: .github/workflows/release.yml)
--build-name string     Build name (binary name)
--build-platform string Build platforms (comma-separated)
--build-tags strings    Go build tags
--build-obfuscate      Enable obfuscation
--nsis                  Enable NSIS installer
--deno-build string    Deno build command
--wails-build-webview2 string WebView2 mode
--build-cache           Enable build cache
--archive-format string Archive format (gz, xz, zip)
--validate              Validate workflow syntax
--help, -h              Show help
```

**Examples:**

```bash
# Generate workflow
core build workflow --output .github/workflows/release.yml

# Generate with specific platforms
core build workflow --build-platform linux/amd64,linux/arm64,darwin/arm64

# Validate workflow
core build workflow --validate

# Full workflow generation
core build workflow \
    --build-name my-app \
    --build-platform linux/amd64,linux/arm64,darwin/amd64,windows/amd64 \
    --build-obfuscate \
    --nsis \
    --archive-format gz
```

---

## 📝 Usage Patterns

### Pattern 1: Simple Go CLI Tool

**Structure:**
```
my-cli/
├── cmd/
│   └── cli/
│       └── main.go
├── go.mod
└── .core/
    └── build.yaml
```

**Configuration:**
```yaml
# .core/build.yaml
version: 1
project:
  name: my-cli
  main: ./cmd/cli
  binary: cli
targets:
  - os: linux
    arch: amd64
  - os: darwin
    arch: arm64
  - os: windows
    arch: amd64
```

**Build:**
```bash
core build
```

### Pattern 2: Wails Desktop Application

**Structure:**
```
my-app/
├── wails.json
├── frontend/
│   ├── package.json
│   └── ...
├── cmd/
│   └── app/
│       └── main.go
├── go.mod
└── .core/
    └── build.yaml
```

**Configuration:**
```yaml
# .core/build.yaml
version: 1
project:
  name: my-app
  main: .
  binary: my-app
build:
  type: wails
  nsis: true
  webview2: download
sdk:
  spec: openapi.yaml
  languages: [typescript]
```

**Build:**
```bash
core build
# Automatically runs: npm run build, wails build
```

### Pattern 3: Multi-Platform Docker Service

**Structure:**
```
my-service/
├── Dockerfile
├── cmd/
│   └── service/
│       └── main.go
├── go.mod
└── .core/
    └── build.yaml
```

**Configuration:**
```yaml
# .core/build.yaml
version: 1
project:
  name: my-service
build:
  type: docker
  dockerfile: Dockerfile
  registry: ghcr.io
  image: my-org/my-service
  tags: [latest, v1.0]
  push: true
targets:
  - os: linux
    arch: amd64
  - os: linux
    arch: arm64
```

**Build:**
```bash
core build
# Builds and pushes multi-platform Docker images
```

### Pattern 4: macOS Application with Full Apple Support

**Structure:**
```
MyApp/
├── cmd/
│   └── macos/
│       └── main.go
├── Info.plist (generated)
├── go.mod
└── .core/
    └── build.yaml
```

**Configuration:**
```yaml
# .core/build.yaml
version: 1
project:
  name: MyApp
  main: ./cmd/macos
  binary: MyApp
build:
  type: go
  archive_format: zip
apple:
  team_id: TEAM_ID
  bundle_id: com.example.myapp
  cert_identity: "Developer ID Application: Lethean CIC (TEAM_ID)"
sign:
  enabled: true
  macos:
    identity: "Developer ID Application: Lethean CIC (TEAM_ID)"
    notarize: true
    apple_id: $APPLE_ID
    team_id: $APPLE_TEAM_ID
    app_password: $APPLE_APP_PASSWORD
```

**Build:**
```bash
core build apple --universal --sign --notarize --dmg
```

### Pattern 5: Full Release with Multiple Publishers

**Structure:**
```
my-app/
├── cmd/
│   └── app/
│       └── main.go
├── go.mod
├── openapi.yaml
└── .core/
    ├── build.yaml
    └── release.yaml
```

**Build Configuration:**
```yaml
# .core/build.yaml
version: 1
project:
  name: my-app
  main: ./cmd/app
  binary: app
targets:
  - os: linux
    arch: amd64
  - os: darwin
    arch: arm64
  - os: windows
    arch: amd64
build:
  obfuscate: true
  archive_format: gz
sign:
  enabled: true
sdk:
  spec: openapi.yaml
  languages: [typescript, python, go, php]
  diff: true
```

**Release Configuration:**
```yaml
# .core/release.yaml
version: 1
project:
  name: my-app
  repository: my-org/my-app
publishers:
  - type: github
    draft: false
  - type: docker
    registry: ghcr.io
    image: my-org/my-app
    tags: [latest, v{{.Version}}]
  - type: homebrew
    tap: my-org/homebrew-tap
    formula: my-app
  - type: npm
    package: @my-org/my-app
    access: public
```

**Release:**
```bash
core release --tag v1.0.0 --sign --publish --sdk --changelog
```

---

## 🧪 Testing

### Test Organization

All packages follow the **AX-7 Triplet Pattern**:

```
pkg/
└── build/
    ├── config.go                # Main implementation
    ├── config_test.go           # Unit tests (Good/Bad/Ugly)
    ├── config_example_test.go   # Usage examples
    └── stdlib_assert_test.go     # Standard library compliance
```

### Test Categories

| Category | Pattern | Purpose | Count |
|----------|---------|---------|-------|
| Good | `Test*_Good` | Happy path, expected inputs | 100+ |
| Bad | `Test*_Bad` | Error conditions, invalid inputs | 50+ |
| Ugly | `Test*_Ugly` | Edge cases, unusual conditions | 20+ |
| Behaviour | `Test*_Behaviour` | Integration, end-to-end | 10+ |
| Example | `Test*_Example` | Usage examples | 10+ |

### Test Examples

**Discovery Test:**

```go
// pkg/build/discovery_test.go
func TestDiscovery_Good(t *testing.T) {
    t.Parallel()
    
    // Test Go project detection
    result := build.Discover(storage.Local, "testdata/go-project")
    assert.True(t, result.OK)
    types := result.Value.([]build.ProjectType)
    assert.Contains(t, types, build.ProjectTypeGo)
    
    // Test Wails project detection
    result = build.Discover(storage.Local, "testdata/wails-project")
    assert.True(t, result.OK)
    types = result.Value.([]build.ProjectType)
    assert.Contains(t, types, build.ProjectTypeWails)
    assert.Contains(t, types, build.ProjectTypeGo)
}

func TestDiscovery_Bad(t *testing.T) {
    t.Parallel()
    
    // Test with non-existent directory
    result := build.Discover(storage.Local, "/nonexistent/path")
    assert.False(t, result.OK)
    assert.NotNil(t, result.Err)
}

func TestDiscovery_Ugly(t *testing.T) {
    t.Parallel()
    
    // Test with empty directory
    tmpDir := t.TempDir()
    result := build.Discover(storage.Local, tmpDir)
    assert.True(t, result.OK)
    types := result.Value.([]build.ProjectType)
    assert.Empty(t, types)
}
```

**Builder Test:**

```go
// pkg/build/builders/go_test.go
func TestGoBuilder_Build_Good(t *testing.T) {
    t.Parallel()
    
    builder := &GoBuilder{}
    cfg := &build.Config{
        Binary: "test-binary",
        Build: build.Build{
            Flags: []string{"-trimpath"},
            CGO: false,
        },
    }
    target := build.TargetConfig{OS: "linux", Arch: "amd64"}
    
    result := builder.Build(context.Background(), cfg, target)
    assert.True(t, result.OK)
}

func TestGoBuilder_Build_Bad(t *testing.T) {
    t.Parallel()
    
    builder := &GoBuilder{}
    cfg := &build.Config{
        Main: "/nonexistent/main.go",  // Invalid main path
    }
    target := build.TargetConfig{OS: "linux", Arch: "amd64"}
    
    result := builder.Build(context.Background(), cfg, target)
    assert.False(t, result.OK)
    assert.NotNil(t, result.Err)
}
```

**Publisher Test:**

```go
// pkg/release/publishers/github_test.go
func TestGitHubPublisher_Publish_Good(t *testing.T) {
    t.Parallel()
    
    publisher := &GitHubPublisher{
        Config: Publishers.GitHubConfig{
            Draft: false,
            Prerelease: false,
        },
    }
    
    cfg := &release.ReleaseConfig{
        Project: release.ReleaseProject{
            Name: "test-app",
            Repository: "test-org/test-repo",
        },
        Version: "1.0.0",
    }
    
    artifacts := []release.Artifact{
        {Name: "test-app_linux_amd64", Path: "dist/test-app_linux_amd64"},
    }
    
    // This would normally make actual API calls, but tests use mocks
    result := publisher.DryRun(context.Background(), cfg)
    assert.True(t, result.OK)
}
```

### Running Tests

```bash
# All tests in the module
cd go
go test ./...

# Specific package
cd go
go test ./pkg/build/...

# With coverage
cd go
go test -cover ./...

# Specific test
cd go
go test ./pkg/build -run TestDiscovery_Good

# Without workspace (CI mode)
GOWORK=off GOFLAGS=-mod=mod go test -count=1 -short ./...

# With race detector
cd go
go test -race ./...

# Verbose mode
cd go
go test -v ./pkg/build
```

### Test Coverage

```bash
# Generate coverage report
cd go
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# Open in browser
open coverage.html

# Text summary
go tool cover -func=coverage.out
```

---

## 📖 API Reference

### Type Index

| Type | Package | Description |
|------|---------|-------------|
| `BuildConfig` | `pkg/build` | Complete build configuration |
| `ReleaseConfig` | `pkg/release` | Complete release configuration |
| `ProjectType` | `pkg/build` | Detected project type enum |
| `TargetConfig` | `pkg/build` | Build target (OS/Arch) |
| `Builder` | `pkg/build/builders` | Project builder interface |
| `Pipeline` | `pkg/build` | Build pipeline orchestrator |
| `Publisher` | `pkg/release/publishers` | Release publisher interface |
| `Generator` | `pkg/sdk` | SDK generator interface |
| `Medium` | `pkg/storage` | Storage abstraction interface |
| `CacheConfig` | `pkg/build` | Build cache configuration |
| `SignConfig` | `pkg/build/signing` | Code signing configuration |
| `AppleConfig` | `pkg/build` | Apple-specific configuration |

### Function Index

| Function | Package | Description |
|----------|---------|-------------|
| `Discover()` | `pkg/build` | Detect project types in directory |
| `PrimaryType()` | `pkg/build` | Get primary detected project type |
| `LoadConfig()` | `pkg/build` | Load build configuration from file |
| `NewPipeline()` | `pkg/build` | Create new build pipeline |
| `NewBuilder()` | `pkg/build/builders` | Get builder for project type |
| `Detect()` | `pkg/build/builders/*` | Detect if builder can handle project |
| `Build()` | `pkg/build/builders/*` | Build project for target |
| `Publish()` | `pkg/release/publishers/*` | Publish artifacts to destination |
| `DryRun()` | `pkg/release/publishers/*` | Validate without publishing |
| `Generate()` | `pkg/sdk/generators/*` | Generate SDK for language |
| `SetupCache()` | `pkg/build` | Setup build cache |
| `GenerateArchive()` | `pkg/build` | Create archive for artifacts |
| `GenerateChecksums()` | `pkg/build` | Generate checksum files |
| `ComputeVersion()` | `pkg/release` | Resolve version from git |
| `GenerateChangelog()` | `pkg/release` | Generate changelog from commits |

### Constant Index

| Constant | Package | Description |
|----------|---------|-------------|
| `ProjectTypeGo` | `pkg/build` | Go project type |
| `ProjectTypeWails` | `pkg/build` | Wails project type |
| `ProjectTypeNode` | `pkg/build` | Node.js project type |
| `ProjectTypeDeno` | `pkg/build` | Deno project type |
| `ProjectTypePHP` | `pkg/build` | PHP project type |
| `ProjectTypePython` | `pkg/build` | Python project type |
| `ProjectTypeRust` | `pkg/build` | Rust project type |
| `ProjectTypeCPP` | `pkg/build` | C++ project type |
| `ProjectTypeDocker` | `pkg/build` | Docker project type |
| `ProjectTypeLinuxKit` | `pkg/build` | LinuxKit project type |
| `ProjectTypeTaskfile` | `pkg/build` | Taskfile project type |
| `ProjectTypeDocs` | `pkg/build` | MkDocs project type |
| `ArchiveFormatGZ` | `pkg/build` | Gzip archive format |
| `ArchiveFormatXZ` | `pkg/build` | XZ archive format |
| `ArchiveFormatZip` | `pkg/build` | Zip archive format |
| `ConfigFileName` | `pkg/build` | Configuration file name |
| `ConfigDir` | `pkg/build` | Configuration directory |

---

## 🔗 Related Documentation

### RFC Documents

| Document | Location | Description |
|----------|----------|-------------|
| Main RFC | [`RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.md) | Complete specification |
| Models RFC | [`RFC.models.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.models.md) | Data model definitions |
| Commands RFC | [`RFC.commands.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.commands.md) | Command specifications |
| Build Pipeline RFC | [`RFC.build-pipeline.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.build-pipeline.md) | Pipeline architecture |
| Release Pipeline RFC | [`RFC.release-pipeline.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.release-pipeline.md) | Release architecture |
| SDK Generation RFC | [`RFC.sdk-generation.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.sdk-generation.md) | SDK generation spec |
| Action Port RFC | [`RFC.action-port.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.action-port.md) | GitHub Action port spec |
| API Provider RFC | [`RFC.api-provider.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.api-provider.md) | API surface spec |
| CI Workflow RFC | [`RFC.ci-workflow.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.ci-workflow.md) | CI workflow spec |
| Imports RFC | [`RFC.imports.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.imports.md) | Import specifications |

### Repository Documentation

| Document | Location | Description |
|----------|----------|-------------|
| README.md | [`README.md`](file:///Users/snider/Code/core/go-build/README.md) | Repository overview |
| AGENTS.md | [`AGENTS.md`](file:///Users/snider/Code/core/go-build/AGENTS.md) | Agent guidance |
| CLAUDE.md | [`CLAUDE.md`](file:///Users/snider/Code/core/go-build/CLAUDE.md) | Claude-specific notes |
| LICENSE | [`LICENSE`](file:///Users/snider/Code/core/go-build/LICENSE) | EUPL-1.2 license |
| LICENCE | [`LICENCE`](file:///Users/snider/Code/core/go-build/LICENCE) | EUPL-1.2 license (alternate spelling) |

### Knowledge Pack Documentation

| Document | Location | Description |
|----------|----------|-------------|
| README.md | [`README.md`](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/build/README.md) | This document's companion |
| CoreGo INDEX | [`INDEX.md`](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) | CoreGo package catalog |

---

## 🏷️ Metadata

### Package Information

```yaml
name: go-build
module: dappco.re/go/build
repository: github.com/dappcore/go-build
spec: file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.md
version: v3.0.0
license: EUPL-1.2
status: Production
maintainer: Purberus <purberus@lthn.ai>
```

### Document Information

```yaml
title: go-build Package Index
type: INDEX
description: Complete index for the go-build package with all types, functions, and concepts
file: knowledge-packs/corego/pkg/build/INDEX.md
size: 36KB
lines: ~1000
generated_by: Purberus
commit: c92a626
date: 2026-06-17
```

### Related Packages

| Package | Relationship | Purpose |
|---------|--------------|---------|
| `go-config` | Dependency | Configuration management |
| `go-io` | Dependency | Storage abstractions |
| `go-process` | Dependency | Process management |
| `go-context` | Dependency | Context utilities |
| `dAppCore/build@v3` | GitHub Action | Action counterpart |
| `core/cli` | Related | CLI command registration |

---

## 📝 Changelog

| Date | Change | Commit |
|------|--------|--------|
| 2026-06-17 | Initial knowledge pack documentation | c92a626 |
| 2026-06-17 | Added to CoreGo knowledge pack | c92a626 |

---

*This INDEX.md file is part of the CoreGo Knowledge Pack, maintained by Purberus <purberus@lthn.ai>*  
*Knowledge Pack Version: CoreGo v1.2.0*  
*Last Updated: 2026-06-17*
