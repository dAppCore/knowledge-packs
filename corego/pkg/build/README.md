# go-build — CoreGo Build Orchestration

> **Package:** `dappco.re/go/build`  
> **Repository:** [`github.com/dappcore/go-build`](https://github.com/dappcore/go-build)  
> **Spec:** [`plans/code/core/go/build/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>  
> **Status:** ✅ Production Ready  

---

## 📋 Overview

**go-build** is the comprehensive build orchestration, release automation, and code signing framework for the Lethean ecosystem. It serves as the backbone for `core build`, `core release`, `core sdk`, and `core ci` commands, providing a unified interface for building, packaging, signing, and publishing software across multiple platforms and languages.

### 🎯 Key Capabilities

| Category | Features |
|----------|----------|
| **Project Discovery** | Auto-detection for 11+ project types (Go, Wails, Node, Deno, PHP, Python, Rust, C++, Docker, LinuxKit, Taskfile, MkDocs) |
| **Build Orchestration** | Cross-platform matrix builds, build caching, obfuscation, Docker/LinuxKit image building |
| **Code Signing** | GPG, macOS codesign/notarisation, Windows SignTool |
| **Release Publishing** | GitHub, Docker, npm, Homebrew, Scoop, Chocolatey, AUR, LinuxKit |
| **SDK Generation** | OpenAPI-based SDK generation for TypeScript, Python, Go, PHP |
| **Apple Ecosystem** | Universal builds, DMG packaging, TestFlight/App Store submission, Xcode Cloud |
| **Workflow Automation** | Reusable GitHub Actions workflow generation |

---

## 🏗️ Architecture

### Module Structure

```
core/go-build/
├── go/                          # Go module root (dappco.re/go/build)
│   ├── cmd/                     # CLI command registration
│   │   ├── build/               # core build, core release commands
│   │   ├── ci/                  # core ci command
│   │   ├── sdk/                 # core sdk command
│   │   └── service/             # Build daemon/service
│   │
│   ├── pkg/                     # Library packages
│   │   ├── build/               # Core build logic
│   │   │   ├── apple/           # Apple-specific (plist, notarisation, DMG)
│   │   │   ├── builders/        # Project-type builders (12 types)
│   │   │   ├── images/          # LinuxKit image building
│   │   │   ├── installers/      # Installer script generation
│   │   │   ├── signing/         # Code signing (GPG, macOS, Windows)
│   │   │   └── templates/       # Workflow/runtimes templates
│   │   │
│   │   ├── release/             # Release orchestration
│   │   │   └── publishers/      # 8 publisher implementations
│   │   │
│   │   ├── sdk/                 # OpenAPI SDK generation
│   │   │   └── generators/      # Language-specific generators
│   │   │
│   │   ├── api/                 # Build provider API surface
│   │   ├── service/             # Build service daemon
│   │   └── storage/             # Storage abstractions
│   │
│   ├── internal/                # Non-public implementation support
│   └── tests/                   # Integration tests
│
├── docs/                        # Cross-language documentation
├── ui/                          # Language-specific UI assets
├── README.md
├── AGENTS.md
└── CLAUDE.md
```

### Core Design Principles

1. **Convention over Configuration** — Smart defaults with `.core/build.yaml` override
2. **Multi-Language Support** — Single toolchain for Go, Wails, Node, Deno, PHP, Python, Rust, C++, Docker
3. **Platform Agnostic** — Works on Linux, macOS, Windows, cross-compiles everywhere
4. **Agent-First** — Designed for AI agents to understand and extend
5. **Zero External Dependencies** — Uses CoreGo primitives exclusively

---

## 🔍 Package Deep Dive

### pkg/build/ — Core Build Logic

The `pkg/build` package is the heart of go-build, containing:

#### Configuration Management (`config.go`)

```go
// BuildConfig holds complete build configuration from .core/build.yaml
type BuildConfig struct {
    Version  int
    Project  Project
    Build    Build
    Apple    AppleConfig
    PreBuild PreBuild
    Targets  []TargetConfig
    Sign     signing.SignConfig
    SDK      *sdk.Config
    LinuxKit LinuxKitConfig
}

// Project metadata
type Project struct {
    Name        string
    Description string
    Main        string  // e.g., ./cmd/core
    Binary      string  // e.g., core
}

// Build settings
type Build struct {
    Type       string            // go, wails, docker, etc.
    CGO        bool
    Obfuscate  bool              // Use garble
    NSIS       bool              // Windows NSIS installer
    WebView2   string            // download|embed|browser|error
    Flags      []string          // Build flags
    LDFlags    []string          // Linker flags
    BuildTags  []string          // Go build tags
    ArchiveFormat string         // gz, xz, zip
    Env        []string
    Cache      CacheConfig
    Dockerfile string
    Registry   string
    Image      string
    Tags       []string
    BuildArgs  map[string]string
    Push       bool
    Load       bool
    LinuxKitConfig string
    Formats    []string          // iso, raw, qcow2, vmdk, etc.
}
```

#### Project Discovery (`discovery.go`)

**Marker-based detection for 11 project types:**

```go
// Marker files for project type detection
const (
    markerBuildConfig   = ".core/build.yaml"
    markerGoMod         = "go.mod"
    markerGoWork        = "go.work"
    markerMainGo        = "main.go"
    markerWails         = "wails.json"
    markerNodePackage   = "package.json"
    markerDenoJSON      = "deno.json"
    markerComposer      = "composer.json"
    markerMkDocs        = "mkdocs.yml"
    markerPyProject     = "pyproject.toml"
    markerCargo         = "Cargo.toml"
    markerDockerfile    = "Dockerfile"
)

// Discover detects project types in directory
func Discover(fs storage.Medium, dir string) core.Result

// Supported project types
const (
    ProjectTypeGo        = "go"
    ProjectTypeWails     = "wails"
    ProjectTypeNode      = "node"
    ProjectTypeDeno      = "deno"
    ProjectTypePHP       = "php"
    ProjectTypePython    = "python"
    ProjectTypeRust      = "rust"
    ProjectTypeCPP       = "cpp"
    ProjectTypeDocker    = "docker"
    ProjectTypeLinuxKit  = "linuxkit"
    ProjectTypeTaskfile  = "taskfile"
    ProjectTypeDocs      = "docs"
)
```

**Discovery Flow:**
1. Check for explicit type in `.core/build.yaml`
2. Check marker files in priority order (Wails first, then Go, etc.)
3. Check nested frontend manifests (`frontend/package.json`)
4. Check subtree for frontend projects
5. Return ordered list of detected types

#### Build Pipeline (`pipeline.go`)

```go
// Pipeline orchestrates the complete build process
type Pipeline struct {
    Config      *Config
    Options     *Options
    Builders    map[ProjectType]Builder
    Publishers  []release.Publisher
    // ...
}

// Run executes the build pipeline
func (p *Pipeline) Run(ctx context.Context) core.Result

// Build phases:
// 1. Setup - Validate config, resolve dependencies
// 2. PreBuild - Run frontend builds (npm, deno)
// 3. Build - Compile all targets
// 4. Archive - Create tar.gz/zip archives
// 5. Checksum - Generate SHA256 checksums
// 6. Sign - Code signing (GPG, macOS, Windows)
// 7. Package - Create installers (NSIS, DMG, etc.)
// 8. Publish - Upload to repositories
```

#### Cross-Platform Targets (`config.go`)

```go
// TargetConfig defines a build target
type TargetConfig struct {
    OS    string   // linux, darwin, windows
    Arch  string   // amd64, arm64, 386, etc.
    ARM   string   // optional ARM version (v6, v7, v8)
    Tags  []string // Additional build tags
}

// Common target matrix:
// - linux/amd64, linux/arm64
// - darwin/amd64, darwin/arm64  
// - windows/amd64, windows/arm64
```

#### Archive & Checksums (`archive.go`, `checksum.go`)

```go
// ArchiveFormat constants
const (
    ArchiveFormatGZ   = "gz"   // gzip (default)
    ArchiveFormatXZ   = "xz"   // xz
    ArchiveFormatZip  = "zip"  // zip
)

// GenerateArchive creates platform-specific archive
func GenerateArchive(cfg *Config, target TargetConfig) core.Result

// ChecksumAlgorithm constants
const (
    ChecksumSHA256 = "sha256"
    ChecksumSHA512 = "sha512"
)

// GenerateChecksums creates checksum files for all artifacts
func GenerateChecksums(dir string, files []string) core.Result
```

#### Build Caching (`cache.go`)

```go
// CacheConfig controls build cache setup
type CacheConfig struct {
    Enabled      bool
    Dir          string   // Cache metadata directory
    KeyPrefix    string   // Cache key prefix
    Paths        []string // Directories to cache
    RestoreKeys  []string // Fallback cache keys
}

// SetupCache creates cache directories and configures GitHub Actions cache
func SetupCache(cfg *CacheConfig) core.Result
```

### pkg/build/apple/ — Apple Ecosystem Support

Complete macOS/iOS build, signing, and distribution:

```go
// AppleConfig holds macOS Apple pipeline settings
type AppleConfig struct {
    TeamID       string
    BundleID     string
    Arch         string
    CertIdentity string
    ProfilePath  string
    KeychainPath string
}

// apple.go provides:
// - Universal binary creation (multi-arch)
// - Code signing with Developer ID certificates
// - Notarisation via Apple's API
// - DMG packaging
// - TestFlight/App Store submission
// - Info.plist generation
// - Entitlements generation
// - Xcode Cloud script generation
```

**Apple RealExec Integration:**
- Uses `security` CLI for certificate management
- Integrates with Apple's notarisation service
- Handles keychain operations securely

### pkg/build/builders/ — Project-Specific Builders

Each builder implements the `Builder` interface:

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

**Available Builders (12 types):**

| Builder | Description | Key Files |
|---------|-------------|-----------|
| `go.go` | Standard Go builds | `go build` with flags, tags, CGO |
| `wails.go` | Wails frontend + Go backend | wails.json, frontend detection |
| `node.go` | Node.js projects | package.json, npm/yarn/pnpm |
| `deno.go` | Deno projects | deno.json, deno task build |
| `php.go` | PHP projects | composer.json |
| `python.go` | Python projects | pyproject.toml, requirements.txt |
| `rust.go` | Rust projects | Cargo.toml, cargo build |
| `cpp.go` | C++ projects | CMakeLists.txt, make |
| `docker.go` | Docker image builds | Dockerfile, docker build |
| `linuxkit.go` | LinuxKit images | linuxkit.yml, kernel+initrd |
| `docs.go` | MkDocs documentation | mkdocs.yml |
| `taskfile.go` | Taskfile projects | Taskfile.yml |

**Go Builder (`builders/go.go`):**
```go
func (b *GoBuilder) Build(ctx context.Context, cfg *Config, target TargetConfig) core.Result {
    // Set GOOS, GOARCH, CGO_ENABLED
    // Apply build flags, ldflags, tags
    // Handle obfuscation with garble
    // Generate debug info if needed
    // Return compiled binary path
}
```

**Wails Builder (`builders/wails.go`):**
- Detects frontend framework (Angular, React, Vue, etc.)
- Builds frontend with npm/deno
- Compiles Go backend
- Bundles into single binary
- Handles WebView2 delivery modes:
  - `download` - Download at runtime
  - `embed` - Embed in binary
  - `browser` - Use system browser
  - `error` - Fail if not available

**Docker Builder (`builders/docker.go`):**
- Supports multi-stage builds
- Handles platform-specific builds
- Pushes to registries
- Loads for local development
- Supports BuildKit and traditional builds

**LinuxKit Builder (`builders/linuxkit.go`):**
- Builds immutable Linux images
- Supports multiple output formats:
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

```go
// SignConfig holds complete signing configuration
type SignConfig struct {
    Enabled bool
    GPG     GPGConfig
    MacOS   MacOSConfig
    Windows WindowsConfig
}

// GPGConfig for PGP/GPG signing
type GPGConfig struct {
    Key         string // Key ID or fingerprint
    Password    string // Optional password
    Armor       bool   // ASCII-armor output
    DetachSign  bool   // Detached signature
}

// MacOSConfig for macOS codesigning
type MacOSConfig struct {
    Identity       string // Certificate identity
    Notarize       bool   // Enable notarisation
    AppleID        string // Apple ID for notarisation
    TeamID         string // Team ID
    AppPassword    string // App-specific password
    Timestamp      bool   // Timestamp server
}

// WindowsConfig for Windows signing
type WindowsConfig struct {
    Signtool      bool   // Enable SignTool
    Certificate   string // Certificate path/name
    Password      string // Certificate password
    TimestampURL  string // Timestamp server URL
}
```

**Signing Process:**
1. **GPG Signing** — Creates detached `.sig` files for binaries
2. **macOS Signing** — Uses `codesign` with Developer ID certificates
3. **macOS Notarisation** — Submits to Apple's notarisation service
4. **Windows Signing** — Uses `signtool` with Authenticode certificates

### pkg/build/installers/ — Installer Generation

```go
// InstallerType constants
type InstallerType string

const (
    InstallerNSIS      InstallerType = "nsis"      // Windows NSIS
    InstallerDeb       InstallerType = "deb"       // Debian package
    InstallerRPM       InstallerType = "rpm"       // RPM package
    InstallerDMG       InstallerType = "dmg"       // macOS Disk Image
    InstallerShell     InstallerType = "shell"     // Shell script
    InstallerPowerShell InstallerType = "powershell" // PowerShell script
)

// GenerateInstaller creates platform-specific installer
func GenerateInstaller(cfg *Config, target TargetConfig) core.Result
```

**NSIS Installer:**
- Windows installer with modern UI
- Automatic dependency detection
- Start menu shortcuts
- Desktop shortcuts
- Uninstall support

**DMG Packaging:**
- macOS disk image creation
- Applications folder symlink
- Custom background
- Volume name customization
- EULA/license agreement

### pkg/release/ — Release Orchestration

```go
// ReleaseConfig holds release configuration from .core/release.yaml
type ReleaseConfig struct {
    Version   int
    Project   ReleaseProject
    Build     ReleaseBuild
    Changelog ChangelogConfig
    SDK       *sdk.Config
    Publishers []PublisherConfig
}

// ReleaseProject holds release project metadata
type ReleaseProject struct {
    Name        string
    Repository  string // e.g., dappcore/core
    Description string
}

// ChangelogConfig controls changelog generation
type ChangelogConfig struct {
    Use       string   // conventional, semver
    Exclude   []string // Commit patterns to exclude
    Categories []string // Custom categories
}
```

**Release Process:**
1. Load `.core/release.yaml`
2. Resolve version from git tags or semver
3. Generate changelog from commit history
4. Execute build pipeline
5. Generate checksums
6. Sign artifacts
7. Publish to configured destinations
8. Create GitHub release

### pkg/release/publishers/ — Release Publishers (8 types)

Each publisher implements the `Publisher` interface:

```go
type Publisher interface {
    // Name returns the publisher name
    Name() string
    
    // Type returns the publisher type (github, docker, npm, etc.)
    Type() string
    
    // Publish uploads artifacts to the destination
    Publish(ctx context.Context, cfg *ReleaseConfig, artifacts []Artifact) core.Result
    
    // DryRun validates configuration without publishing
    DryRun(ctx context.Context, cfg *ReleaseConfig) core.Result
}
```

**Publisher Types:**

| Publisher | Description | Repository Format |
|-----------|-------------|-------------------|
| `github.go` | GitHub Releases | GitHub API, drafts, prereleases |
| `docker.go` | Docker Images | Registry push, multi-platform |
| `npm.go` | npm Packages | npm registry, access control |
| `homebrew.go` | Homebrew Formulas | Tap repository, official formula |
| `scoop.go` | Scoop Manifests | Bucket repository |
| `chocolatey.go` | Chocolatey Packages | Package source, API key |
| `aur.go` | Arch User Repository | PKGBUILD, AUR helper |
| `linuxkit.go` | LinuxKit Images | Image push to hub |

**GitHub Publisher:**
- Creates GitHub releases
- Supports draft releases
- Handles prerelease detection (alpha/beta/rc)
- Uploads all artifacts with proper names
- Generates release notes from changelog

**Docker Publisher:**
- Multi-platform image builds
- Pushes to configured registry (ghcr.io, Docker Hub, etc.)
- Handles build args and tags
- Supports platform-specific builds

**Homebrew Publisher:**
- Generates formula files
- Supports official Homebrew repository PRs
- Handles bottle creation
- Validates formula syntax

**npm Publisher:**
- Publishes to npm registry
- Handles access control (public/private)
- Manages package.json updates
- Supports scopes (`@dappcore/core`)

### pkg/sdk/ — OpenAPI SDK Generation

```go
// Config holds SDK generation configuration
type Config struct {
    Spec       string   // Path to OpenAPI spec or "auto-detect"
    Languages  []string // typescript, python, go, php
    Output     string   // Output directory
    Diff       bool     // Enable breaking change detection
    Package    string   // Package name (Go, npm)
    Module     string   // Module path (Go)
    ClassName  string   // Class name prefix
}

// Generator interface for language-specific SDK generation
type Generator interface {
    Name() string
    Generate(ctx context.Context, cfg *Config, spec *openapi3.T) core.Result
}
```

**Supported Generators:**
- TypeScript (using openapi-typescript)
- Python (using openapi-python-client)
- Go (using oapi-codegen)
- PHP (using openapi-php-client)

**Breaking Change Detection:**
- Uses `oasdiff` to detect API changes
- Compares against previous spec
- Generates compatibility report
- Can fail build on breaking changes

### pkg/storage/ — Storage Abstractions

```go
// Medium defines the storage interface used throughout go-build
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
// Also supports: S3, SQLite, HTTP, and other go-io backends
```

---

## 🚀 Commands

### core build

**Basic Usage:**
```bash
# Simple passthrough (no .core/build.yaml)
core build                        # = go build .
core build --output ./bin         # = go build -o ./bin .
core build --targets linux/amd64  # Cross-compile

# With .core/build.yaml
core build                        # Matrix build all targets
core build --debug                # Enable debug output
core build --dry-run             # Show what would be built
```

**Flags:**
```
--config        Path to build config (default: .core/build.yaml)
--targets       Build targets (os/arch pairs)
--output        Output directory
--clean         Clean before building
--cache         Enable build cache
--obfuscate     Enable binary obfuscation
--sign          Enable code signing
--archive       Create archives
--checksum      Generate checksums
```

### core release

```bash
# Dry run (show what would be released)
core release --dry-run

# Create release
core release --tag v1.0.0

# Create draft release
core release --draft --tag v1.0.0

# Release with signing
core release --sign --tag v1.0.0

# Publish to specific destinations
core release --publish --tag v1.0.0
```

**Flags:**
```
--config       Path to release config (default: .core/release.yaml)
--tag          Version tag
--draft        Create as draft
--prerelease   Mark as prerelease
--dry-run      Validate without publishing
--sign         Enable code signing
--publish      Publish to configured destinations
--sdk          Generate SDKs
--changelog    Generate changelog
```

### core build apple

macOS-specific build commands:

```bash
# Build for Apple platforms
core build apple --targets darwin/amd64,darwin/arm64

# Sign and notarise
core build apple --sign --notarize

# Create DMG
core build apple --dmg

# Universal binary
core build apple --universal

# Xcode Cloud
core build apple --xcode-cloud
```

### core build workflow

Generate reusable GitHub Actions workflow:

```bash
# Generate workflow
core build workflow --output .github/workflows/release.yml

# Validate workflow
core build workflow --validate

# Generate with specific inputs
core build workflow --build-name core --build-platform linux/amd64,linux/arm64
```

**Workflow Inputs (aligned with dAppCore/build@v3 action):**
```yaml
inputs:
  core-version:        # Core version to use
  go-version:          # Go version
  node-version:       # Node version
  python-version:     # Python version
  wails-version:      # Wails version
  version:            # Release version
  build:              # Build command
  sign:               # Enable signing
  package:            # Enable packaging
  build-name:         # Binary name
  build-platform:     # Target platforms
  build-tags:         # Go build tags
  build-obfuscate:   # Enable obfuscation
  nsis:               # Enable NSIS installer
  deno-build:         # Deno build command
  wails-build-webview2: # WebView2 mode
  build-cache:        # Enable build cache
  archive-format:     # Archive format (gz, xz, zip)
```

### core sdk

OpenAPI SDK generation:

```bash
# Auto-detect OpenAPI spec
core sdk

# Specify spec file
core sdk --spec openapi.yaml

# Generate for specific languages
core sdk --languages typescript,python,go

# Enable breaking change detection
core sdk --diff

# Specify output directory
core sdk --output sdk/
```

### core ci

Run CI checks:

```bash
# Run all checks
core ci

# Run specific checks
core ci --checks vet,test,lint

# With coverage
core ci --coverage

# Short mode (skip slow tests)
core ci --short
```

---

## 📝 Configuration Files

### .core/build.yaml

Complete build configuration:

```yaml
version: 1

project:
  name: my-app
  description: "My Application"
  main: ./cmd/app
  binary: app

build:
  type: go                    # Override auto-detection
  cgo: false                  # Disable CGO
  obfuscate: false            # Use garble for obfuscation
  nsis: false                 # Windows NSIS installer
  webview2: download          # WebView2 delivery mode
  flags: ["-trimpath"]        # Build flags
  ldflags: ["-s", "-w"]       # Linker flags
  build_tags: ["netgo"]       # Go build tags
  archive_format: gz          # Archive format
  env: ["CGO_ENABLED=0"]      # Environment variables
  
  # Docker settings
  dockerfile: Dockerfile
  registry: ghcr.io
  image: owner/repo
  tags: [latest, v1.0]
  push: false
  
  # LinuxKit settings
  linuxkit_config: linuxkit.yml
  formats: [iso, qcow2]

cache:
  enabled: true
  dir: .core/cache
  key_prefix: my-app
  paths:
    - ~/.cache/go-build
    - ~/go/pkg/mod
  restore_keys:
    - go-

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

sign:
  enabled: true
  gpg:
    key: $GPG_KEY_ID
    password: $GPG_PASSWORD
  macos:
    identity: "Developer ID Application: Lethean CIC (TEAM_ID)"
    notarize: true
    apple_id: $APPLE_ID
    team_id: $APPLE_TEAM_ID
    app_password: $APPLE_APP_PASSWORD
  windows:
    signtool: false

sdk:
  spec: openapi.yaml
  languages: [typescript, python, go, php]
  output: sdk/
  diff: true

linuxkit:
  config: linuxkit.yml
  formats: [iso, raw]
```

### .core/release.yaml

Release-specific configuration:

```yaml
version: 1

project:
  name: my-app
  repository: owner/repo

build:
  targets:
    - os: linux
      arch: amd64
    - os: darwin
      arch: arm64
    - os: windows
      arch: amd64
  archive_format: gz

changelog:
  use: conventional
  exclude:
    - '^docs:'
    - '^test:'
    - '^ci:'

sdk:
  spec: openapi.yaml
  languages: [typescript, python, go]
  output: sdk/
  diff: true

publishers:
  - type: github
    draft: false
    prerelease: false

  - type: docker
    registry: ghcr.io
    image: owner/repo
    tags: [latest, v{{.Version}}]
    platforms:
      - linux/amd64
      - linux/arm64

  - type: homebrew
    tap: owner/homebrew-tap
    formula: my-app
    official:
      enabled: false

  - type: npm
    package: @owner/my-app
    access: public
```

---

## 🔧 Integration Points

### CoreGo Framework Integration

**Uses CoreGo primitives:**
- `core.Result` — Error handling with panic recovery
- `core.Action` — Action execution framework
- `core.IPC` — Internal IPC communication
- Storage abstractions from `go-io`

**Service Lock Support:**
```go
// go-build can run as a service in CoreGo
core.New(
    core.WithServiceLock(),  // Stops new services and admin modifications
    core.WithService("build", build.NewService()),
)
```

### dAppCore/build@v3 GitHub Action

The GitHub Action mirrors the CLI functionality:

```yaml
- uses: dAppCore/build@v3
  with:
    core-version: v3.0.0
    go-version: 1.22
    node-version: 20
    build: true
    sign: true
    package: true
    publish: true
    build-name: my-app
    build-platform: linux/amd64,linux/arm64,darwin/amd64,darwin/arm64,windows/amd64
```

**Action-BUILD Parity:**
- Both use the same discovery logic
- Both use the same option computation
- Both use the same setup planning
- Both expose the same control surface

### API Provider Surface (`pkg/api/`)

Exposes build capabilities via API for CoreGo runtime:

```go
// BuildProvider handles build requests via API
type BuildProvider struct {
    Config *Config
}

// Build handles POST /api/build requests
func (p *BuildProvider) Build(w http.ResponseWriter, r *http.Request)

// Release handles POST /api/release requests  
func (p *BuildProvider) Release(w http.ResponseWriter, r *http.Request)

// Status handles GET /api/build/status requests
func (p *BuildProvider) Status(w http.ResponseWriter, r *http.Request)
```

### Build Service Daemon (`pkg/service/`)

Local build daemon for long-running operations:

```go
// Service manages the build daemon
type Service struct {
    Config *Config
}

// Run starts the build service
func (s *Service) Run(ctx context.Context) core.Result

// Build executes a build request
func (s *Service) Build(ctx context.Context, req *BuildRequest) core.Result
```

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| Total Lines of Code | ~250K |
| Test Files | 100+ |
| Test Coverage | High |
| Go Version | 1.22+ |
| Module Path | dappco.re/go/build |
| Last Updated | 2026-06-17 |

---

## 🎯 Use Cases

### 1. Multi-Platform Go Application

Build a Go CLI tool for all platforms:

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
  - os: linux
    arch: arm64
  - os: darwin
    arch: amd64
  - os: darwin
    arch: arm64
  - os: windows
    arch: amd64

build:
  obfuscate: true
  archive_format: gz
```

```bash
core build
# Produces: dist/cli_linux_amd64, dist/cli_linux_arm64, etc.
```

### 2. Wails Desktop Application

Build a Wails app with frontend:

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
  webview2: embed

sdk:
  spec: openapi.yaml
  languages: [typescript]
```

```bash
core build
# Automatically: npm run build, wails build, create NSIS installer
```

### 3. Docker Image Release

Build and publish Docker images:

```yaml
# .core/build.yaml
version: 1
project:
  name: my-service

build:
  type: docker
  dockerfile: Dockerfile
  registry: ghcr.io
  image: owner/my-service
  tags: [latest, v1.0]
  push: true

targets:
  - os: linux
    arch: amd64
  - os: linux
    arch: arm64
```

```bash
core build
# Builds and pushes multi-platform Docker images
```

### 4. macOS Application with Notarisation

Complete macOS app build and notarisation:

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

```bash
core build apple --universal --sign --notarize --dmg
```

### 5. Full Release Pipeline

Complete build and release:

```yaml
# .core/release.yaml
version: 1

project:
  name: my-app
  repository: owner/repo

build:
  targets:
    - os: linux
      arch: amd64
    - os: darwin
      arch: arm64
    - os: windows
      arch: amd64

publishers:
  - type: github
    draft: false
  - type: docker
    registry: ghcr.io
    image: owner/my-app
  - type: homebrew
    tap: owner/homebrew-tap

sdk:
  spec: openapi.yaml
  languages: [typescript, python, go, php]
  diff: true
```

```bash
core release --tag v1.0.0 --sign --publish --sdk
# 1. Build all targets
# 2. Generate checksums
# 3. Sign artifacts
# 4. Generate SDKs
# 5. Create GitHub release
# 6. Publish Docker images
# 7. Update Homebrew tap
```

---

## 🔗 Dependencies

### CoreGo Dependencies

```go
import (
    "dappco.re/go"                    // Core framework
    "dappco.re/go/build"              // This package
    "dappco.re/go/build/pkg/build"   // Build logic
    "dappco.re/go/build/pkg/release"  // Release logic
    "dappco.re/go/build/pkg/sdk"      // SDK generation
)
```

### External Dependencies

```
# From go.mod
github.com/blang/semver/v4        - Semantic versioning
github.com/go-git/go-git/v5      - Git operations
github.com/google/go-github/v62 - GitHub API
github.com/markphelan/oasdiff     - OpenAPI diffing
github.com/pelletier/go-toml/v2   - TOML parsing
github.com/spf13/viper            - Configuration
get.ankama.com/goseco/po          - Gettext PO file handling
golang.org/x/oauth2               - OAuth2 for publishing
gopkg.in/yaml.v3                  - YAML parsing
```

---

## 🧪 Testing

### Test Structure

All packages follow the **AX-7 Triplet Pattern**:

```
<file>.go           # Main implementation
<file>_test.go      # Unit tests (Good/Bad/Ugly)
<file>_example_test.go  # Usage examples
<file>_behaviour_test.go  # Integration/behaviour tests
stdlib_assert_test.go    # Standard library compliance tests
```

### Test Categories

| Category | Purpose | Pattern |
|----------|---------|---------|
| Good | Happy path, expected inputs | `Test*_Good` |
| Bad | Error conditions, invalid inputs | `Test*_Bad` |
| Ugly | Edge cases, unusual conditions | `Test*_Ugly` |
| Behaviour | Integration, end-to-end | `Test*_Behaviour` |
| Example | Usage examples | `Test*_Example` |

### Running Tests

```bash
# All tests
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

# Without workspace
GOWORK=off GOFLAGS=-mod=mod go test -count=1 -short ./...
```

### Example Tests

**Discovery Example:**
```go
func TestDiscovery_Good(t *testing.T) {
    t.Parallel()
    
    result := build.Discover(storage.Local, "testdata/go-project")
    assert.True(t, result.OK)
    
    types := result.Value.([]build.ProjectType)
    assert.Contains(t, types, build.ProjectTypeGo)
}
```

**Build Example:**
```go
func TestPipeline_Good(t *testing.T) {
    t.Parallel()
    
    cfg := &build.Config{
        Binary: "test-binary",
        Targets: []build.TargetConfig{
            {OS: "linux", Arch: "amd64"},
        },
    }
    
    pipeline := build.NewPipeline(cfg)
    result := pipeline.Run(context.Background())
    assert.True(t, result.OK)
}
```

---

## 📚 API Reference

### Key Types

| Type | Description | Package |
|------|-------------|---------|
| `BuildConfig` | Complete build configuration | `pkg/build` |
| `ReleaseConfig` | Complete release configuration | `pkg/release` |
| `ProjectType` | Detected project type | `pkg/build` |
| `TargetConfig` | Build target (OS/Arch) | `pkg/build` |
| `Builder` | Project builder interface | `pkg/build/builders` |
| `Publisher` | Release publisher interface | `pkg/release/publishers` |
| `Generator` | SDK generator interface | `pkg/sdk` |

### Key Functions

| Function | Description | Package |
|----------|-------------|---------|
| `Discover()` | Detect project types | `pkg/build` |
| `PrimaryType()` | Get primary project type | `pkg/build` |
| `LoadConfig()` | Load build configuration | `pkg/build` |
| `NewPipeline()` | Create build pipeline | `pkg/build` |
| `NewBuilder()` | Get builder for project type | `pkg/build/builders` |
| `NewPublisher()` | Get publisher for type | `pkg/release/publishers` |
| `ComputeVersion()` | Resolve version from git | `pkg/release` |
| `GenerateChangelog()` | Generate changelog | `pkg/release` |

---

## 🎓 Examples

### Basic Go Build

```go
package main

import (
    "dappco.re/go/build"
    "dappco.re/go/build/pkg/build"
    "dappco.re/go/build/pkg/storage"
)

func main() {
    // Discover project type
    types, err := build.Discover(storage.Local, ".")
    if err != nil {
        panic(err)
    }
    
    // Load configuration
    cfg, err := build.LoadConfig(storage.Local, ".")
    if err != nil {
        panic(err)
    }
    
    // Create and run pipeline
    pipeline := build.NewPipeline(cfg)
    result := pipeline.Run(context.Background())
    if !result.OK {
        panic(result.Err)
    }
}
```

### Custom Builder

```go
package mybuilder

import (
    "dappco.re/go/build/pkg/build"
    "dappco.re/go/build/pkg/storage"
)

type MyBuilder struct{}

func (b *MyBuilder) Detect(fs storage.Medium, dir string) bool {
    // Check for custom marker files
    return build.FileExists(fs, build.AxJoin(dir, "my-project.json"))
}

func (b *MyBuilder) Build(ctx context.Context, cfg *build.Config, target build.TargetConfig) core.Result {
    // Custom build logic
    return core.Ok("build completed")
}

func (b *MyBuilder) Clean(cfg *build.Config) core.Result {
    // Clean build artifacts
    return core.Ok("cleaned")
}

// Register the builder
func init() {
    build.RegisterBuilder("my-project", &MyBuilder{})
}
```

### Custom Publisher

```go
package mypublisher

import (
    "dappco.re/go/build/pkg/release"
)

type MyPublisher struct {
    Config map[string]string
}

func (p *MyPublisher) Name() string {
    return "my-repo"
}

func (p *MyPublisher) Type() string {
    return "my-repo"
}

func (p *MyPublisher) Publish(ctx context.Context, cfg *release.ReleaseConfig, artifacts []release.Artifact) core.Result {
    // Upload to custom repository
    return core.Ok("published")
}

func (p *MyPublisher) DryRun(ctx context.Context, cfg *release.ReleaseConfig) core.Result {
    return core.Ok("dry run successful")
}

// Register the publisher
func init() {
    release.RegisterPublisher("my-repo", func(cfg map[string]string) release.Publisher {
        return &MyPublisher{Config: cfg}
    })
}
```

---

## 📖 Related Documentation

| Document | Location | Description |
|----------|----------|-------------|
| RFC | [`plans/code/core/go/build/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.md) | Complete specification |
| Models RFC | [`RFC.models.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.models.md) | Data model definitions |
| Commands RFC | [`RFC.commands.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.commands.md) | Command specifications |
| Build Pipeline RFC | [`RFC.build-pipeline.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.build-pipeline.md) | Pipeline architecture |
| Release Pipeline RFC | [`RFC.release-pipeline.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.release-pipeline.md) | Release architecture |
| SDK Generation RFC | [`RFC.sdk-generation.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.sdk-generation.md) | SDK generation spec |
| AGENTS.md | [`AGENTS.md`](file:///Users/snider/Code/core/go-build/AGENTS.md) | Agent guidance |
| CLAUDE.md | [`CLAUDE.md`](file:///Users/snider/Code/core/go-build/CLAUDE.md) | Claude-specific notes |

---

## 🏷️ Metadata

```yaml
name: go-build
module: dappco.re/go/build
repository: github.com/dappcore/go-build
spec: file:///Users/snider/Code/meowmix/plans/code/core/go/build/RFC.md
maintainer: Purberus <purberus@lthn.ai>
license: EUPL-1.2
status: Production
version: v3.0.0
last_updated: 2026-06-17
```

---

*Documentation generated by Purberus for the CoreGo Knowledge Pack*  
*Knowledge Pack Version: CoreGo v1.2.0*  
*Commit: c92a626*
