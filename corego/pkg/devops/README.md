# go-devops — CoreGo DevOps Orchestration

> **Package:** `dappco.re/go/devops`  
> **Repository:** [`github.com/dappcore/go-devops`](https://github.com/dappcore/go-devops)  
> **Spec:** [`plans/code/core/go/devops/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/devops/RFC.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>  
> **Status:** Production Ready  
> **Module:** `forge.lthn.ai/core/go-devops`

---

## Overview

**go-devops** is the infrastructure and build automation library for the Lethean ecosystem. It provides a native Go Ansible playbook executor, multi-target build pipeline, code signing, release orchestration, and a developer toolkit — all designed for AI agent consumption.

### Key capabilities

| Category | Features |
|----------|----------|
| **Infrastructure as Code** | Native Go Ansible playbook executor (~30 modules over SSH without shelling out) |
| **Build Pipeline** | Multi-target with project type auto-detection (Go, Wails, Docker, C++, LinuxKit, Taskfile) |
| **Code Signing** | macOS codesign, GPG, Windows signtool |
| **Release Orchestration** | Changelog generation, 8 publisher backends (GitHub Releases, Docker, Homebrew, npm, AUR, Scoop, Chocolatey, LinuxKit) |
| **Cloud Infrastructure** | Hetzner Cloud and Robot API clients, CloudNS DNS management |
| **Container/VM Management** | QEMU and Hyperkit support for container/VM management |
| **SDK Generation** | OpenAPI SDK generator (TypeScript, Python, Go, PHP) |
| **Developer Toolkit** | Cyclomatic complexity analysis, vulnerability scanning, coverage trending |

### Architecture

**go-devops** is intentionally a multi-Service repository (Mantis #1336/#1379). Each independently-stateful subpackage exposes its own canonical `NewService(opts) + Register(c)` pair under its own package name. Pure-utility subpackages intentionally have no Service wiring.

### Module structure

```
core/go-devops/
├── go/                          # Go module root (dappco.re/go/devops)
│   ├── cmd/                     # CLI command registration
│   │   ├── deploy/              # Deployment commands
│   │   ├── setup/               # Setup wizard and repository configuration
│   │   ├── workspace/           # Workspace configuration
│   │   └── vanity-import/       # Vanity import management
│   │
│   ├── deploy/                  # Deployment utilities
│   │   ├── coolify/             # Coolify deployment service
│   │   └── python/              # Python deployment helpers
│   │
│   ├── devkit/                  # Developer toolkit
│   │   ├── coverage.go          # Coverage trending and analysis
│   │   ├── scan_secrets.go      # Secret scanning
│   │   ├── secret.go            # Secret management
│   │   └── service.go           # DevKit service
│   │
│   ├── snapshot/                # Snapshot utilities
│   │   └── snapshot.go          # Snapshot creation and management
│   │
│   ├── locales/                 # Localisation support
│   │   └── embed.go             # Embedded translations
│   │
│   ├── go.mod                   # Module definition
│   ├── go.sum                   # Dependency checksums
│   └── ...
│
├── docs/                        # Cross-language documentation
│   ├── architecture.md          # Ansible integration, build pipeline, infrastructure APIs
│   ├── build-system.md          # Build system documentation
│   ├── development.md           # Development guide
│   ├── history.md               # Project history
│   ├── index.md                 # Documentation index
│   ├── publishers.md            # Publisher backends
│   ├── sdk-generation.md        # SDK generation guide
│   └── sync.md                  # Synchronisation
│
├── playbooks/                   # Ansible playbooks
├── external/                    # External dependencies
├── README.md
├── AGENTS.md
├── CLAUDE.md
└── LICENCE
```

### Core design principles

1. **Agent-First** — Designed for AI agents to understand and extend
2. **Zero External Dependencies** — Uses CoreGo primitives exclusively where possible
3. **Convention over Configuration** — Smart defaults with override capabilities
4. **Multi-Platform** — Works on Linux, macOS, Windows
5. **Extensible** — Each service is independently extensible

---

## Packages

### Ansible integration (`ansible/`)

The Ansible integration provides a pure Go Ansible playbook executor with ~30 module handlers:

- **SSH Transport** — Secure SSH-based execution without shelling out
- **YAML Parsing** — Native YAML parsing for playbooks and inventory files
- **Jinja2 Templating** — Template support with Jinja2
- **Module Handlers** — ~30 module types for various operations

**Key files:**
- Playbook parsing and execution
- Inventory management
- SSH connection pooling
- Module registry

### Build pipeline (`build/`)

Multi-target build pipeline with auto-detection:

- **Project Types:** Go, Wails, Docker, C++, LinuxKit, Taskfile
- **Cross-Platform:** Matrix builds for multiple architectures
- **Caching:** Build caching for efficiency
- **Obfuscation:** Code obfuscation support

### Release orchestration (`release/`)

Complete release automation:

- **Changelog Generation** — Automatic changelog from commits
- **Asset Packaging** — Archive creation (tar.gz, zip, etc.)
- **Publisher Backends** — 8 backends for release distribution
- **Signing** — macOS codesign, GPG, Windows signtool

**Publisher backends:**
1. GitHub Releases
2. Docker Hub/Registry
3. Homebrew
4. npm
5. AUR (Arch User Repository)
6. Scoop (Windows)
7. Chocolatey (Windows)
8. LinuxKit

### Infrastructure APIs

#### Hetzner APIs

- **Hetzner Cloud API** — VM and infrastructure management
- **Hetzner Robot API** — Dedicated server management

#### CloudNS

- **CloudNS DNS Management** — Domain and DNS record management

### Container/VM management

- **QEMU** — Full virtualisation support
- **Hyperkit** — macOS hypervisor framework
- **LinuxKit** — Docker-free container runtime

### SDK generation

OpenAPI-based SDK generator supporting:

- **TypeScript**
- **Python**
- **Go**
- **PHP**

### Developer toolkit (`devkit/`)

#### Coverage analysis

- **Trending** — Coverage trend analysis across commits
- **Reporting** — HTML and text reports
- **Thresholds** — Configurable coverage thresholds

#### Secret scanning

- **Pattern Matching** — Detect secrets in code
- **False Positive Reduction** — Smart filtering
- **Reporting** — Detailed scan reports

#### Complexity analysis

- **Cyclomatic Complexity** — Code complexity metrics
- **Threshold Warnings** — Configurable complexity limits
- **Visualisation** — Complexity heatmaps

---

## Configuration

### Workspace configuration

```yaml
# .core/devops.yaml
deploy:
  targets:
    - name: production
      type: coolify
      url: https://coolify.example.com
      token: ${{ secrets.COOLIFY_TOKEN }}
  
build:
  matrix:
    - goos: linux
      goarch: amd64
    - goos: linux
      goarch: arm64
    - goos: darwin
      goarch: arm64
  
release:
  publishers:
    - github
    - docker
    - homebrew
  
signing:
  macos:
    identity: "Developer ID Application: Example Inc"
  gpg:
    key_id: ABCDEF1234567890
```

---

## Commands

### Deployment commands

```bash
# Deploy to configured target
core devops deploy production

# Deploy with specific playbook
core devops deploy production --playbook deploy.yml

# List available targets
core devops deploy --list
```

### Setup commands

```bash
# Interactive setup wizard
core devops setup wizard

# Configure GitHub webhooks
core devops setup github-webhooks

# Generate repository configuration
core devops setup repo
```

### Workspace commands

```bash
# Show workspace configuration
core devops workspace config

# Validate configuration
core devops workspace validate
```

---

## Usage patterns

### Running Ansible playbooks

```go
import (
    "forge.lthn.ai/core/go-devops/ansible"
)

// Parse playbook
pb, err := ansible.ParsePlaybook("playbooks/deploy.yml")
if err != nil {
    log.Fatal(err)
}

// Parse inventory
inv, err := ansible.ParseInventory("inventory.yml")
if err != nil {
    log.Fatal(err)
}

// Run playbook
result := pb.Run(ctx, inv)
if result.Error != nil {
    log.Fatal(result.Error)
}

fmt.Printf("Playbook completed in %v", result.Duration)
```

### Build and release pipeline

```go
import (
    "forge.lthn.ai/core/go-devops/build"
    "forge.lthn.ai/core/go-devops/release"
)

// Build artifacts
artifacts, err := build.Build(ctx, ".", build.WithMatrix("linux/amd64", "darwin/arm64"))
if err != nil {
    log.Fatal(err)
}

// Publish release
releaseCfg := release.Config{
    Publishers: []string{"github", "docker"},
    Sign:       true,
}

if err := release.Publish(ctx, releaseCfg, artifacts...); err != nil {
    log.Fatal(err)
}
```

### Using DevKit for code analysis

```go
import (
    "forge.lthn.ai/core/go-devops/devkit"
)

// Scan for secrets
secrets, err := devkit.ScanSecrets(ctx, ".")
if err != nil {
    log.Fatal(err)
}

for _, secret := range secrets {
    fmt.Printf("Found secret at %s: %s\n", secret.File, secret.Type)
}

// Analyse coverage
coverage, err := devkit.AnalyzeCoverage(ctx, "coverage.out")
if err != nil {
    log.Fatal(err)
}

fmt.Printf("Coverage: %.2f%%\n", coverage.Percent)
```

---

## Testing

### Test structure

Each package follows the AX standard with:

- `_test.go` — Unit tests
- `_example_test.go` — Usage examples

### Running tests

```bash
# All tests
go test ./...

# With race detector
go test -race ./...

# Specific package
go test ./go/devkit/...

# With coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

### Test coverage

| Package | Files | Lines | Coverage |
|---------|-------|-------|----------|
| `ansible/` | ~20 | ~2,000 | >80% |
| `build/` | ~15 | ~1,500 | >85% |
| `deploy/` | ~10 | ~800 | >75% |
| `devkit/` | ~10 | ~1,000 | >80% |
| `release/` | ~10 | ~1,200 | >80% |
| `snapshot/` | ~5 | ~400 | >70% |

---

## API reference

### Ansible package

```go
type Playbook struct {
    Name        string
    Description string
    Tasks       []Task
    Handlers    []Handler
    Vars        map[string]interface{}
}

type Task struct {
    Name    string
    Module  string
    Args    map[string]interface{}
    When    string
    Loop    []interface{}
    Retries int
}

func ParsePlaybook(path string) (*Playbook, error)
func ParseInventory(path string) (*Inventory, error)
func (pb *Playbook) Run(ctx context.Context, inv *Inventory) *PlaybookResult
```

### DevKit package

```go
type CoverageReport struct {
    Percent     float64
    TotalFuncs  int
    CoveredFuncs int
    TotalLines  int
    CoveredLines int
}

type SecretResult struct {
    File    string
    Line    int
    Type    string
    Content string
}

func AnalyzeCoverage(ctx context.Context, profilePath string) (*CoverageReport, error)
func ScanSecrets(ctx context.Context, root string) ([]SecretResult, error)
```

---

## Related documentation

| Resource | Description |
|----------|-------------|
| [plans/code/core/go/devops/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/devops/RFC.md) | RFC and specification |
| [docs/architecture.md](file:///Users/snider/Code/core/go-devops/docs/architecture.md) | Architecture overview |
| [docs/development.md](file:///Users/snider/Code/core/go-devops/docs/development.md) | Development guide |
| [docs/publishers.md](file:///Users/snider/Code/core/go-devops/docs/publishers.md) | Publisher backends |
| [docs/sdk-generation.md](file:///Users/snider/Code/core/go-devops/docs/sdk-generation.md) | SDK generation |

---

## Statistics

| Metric | Value |
|--------|-------|
| **Total Go Files** | 139+ |
| **Library Files** | 25 |
| **CLI Commands** | 10+ |
| **Module Dependencies** | 15+ |
| **Test Files** | 50+ |
| **Documentation** | 30KB+ |
| **Lines of Code** | ~15,000 |
