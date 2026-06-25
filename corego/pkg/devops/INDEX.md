# go-devops Package Index

> **Package:** `dappco.re/go/devops`  
> **Repository:** [`github.com/dappcore/go-devops`](https://github.com/dappcore/go-devops)  
> **Spec:** [`plans/code/core/go/devops/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/devops/RFC.md)  
> **Documentation:** [README.md](./README.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>

---

## Table of contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Subpackages](#subpackages)
- [Configuration](#configuration)
- [Commands](#commands)
- [Usage patterns](#usage-patterns)
- [Testing](#testing)
- [API reference](#api-reference)
- [Related documentation](#related-documentation)
- [Statistics](#statistics)

---

## Overview

**go-devops** is the infrastructure and build automation library for the Lethean ecosystem. It provides a native Go Ansible playbook executor, multi-target build pipeline, code signing, release orchestration, container/VM management, SDK generation, and a developer toolkit — all designed for AI agent consumption and following the AX standard.

### Key features

| Category | Count | Description |
|----------|-------|-------------|
| **Project Types** | 6 | Go, Wails, Docker, C++, LinuxKit, Taskfile |
| **Builders** | 6 | One per project type with auto-detection |
| **Publisher Backends** | 8 | GitHub, Docker, Homebrew, npm, AUR, Scoop, Chocolatey, LinuxKit |
| **SDK Languages** | 4 | TypeScript, Python, Go, PHP |
| **Infrastructure APIs** | 3 | Hetzner Cloud, Hetzner Robot, CloudNS |
| **Container Runtimes** | 3 | QEMU, Hyperkit, LinuxKit |
| **Signing Methods** | 3 | macOS codesign, GPG, Windows signtool |
| **Ansible Modules** | ~30 | Pure Go implementations |
| **DevKit Tools** | 3 | Coverage analysis, secret scanning, complexity analysis |

### Package statistics

| Metric | Value |
|--------|-------|
| **Module** | `dappco.re/go/devops` |
| **Go Version** | 1.26.2 |
| **Total Go Files** | 139+ |
| **Library Files** | 25 |
| **CLI Files** | 50+ |
| **Test Files** | 50+ |
| **Total Lines** | ~15,000 |
| **Test Coverage** | >75% average |

---

## Architecture

### Multi-service design

**go-devops** follows Mantis #1336/#1379 design principles as a multi-Service repository:

- Each **independently-stateful subpackage** exposes `NewService(opts) + Register(c)`
- **Service names:** `dev`, `devkit`, `coolify` (and more)
- **Pure-utility subpackages** intentionally have no Service wiring
- **No shared mutable state** between services

### Service registration pattern

```go
// Each service registers itself
service := devops.NewService(opts)
if err := service.Register(container); err != nil {
    return err
}

// Container manages service lifecycle
container := core.New(core.WithServiceLock())
```

### Module dependencies

```
Primary (from go.mod):
├── code.gitea.io/sdk/gitea v0.24.1
├── dappco.re/go/agent v0.28.0
├── dappco.re/go/i18n v0.10.0
├── dappco.re/go/io v0.11.0
├── dappco.re/go/log v0.10.0
├── dappco.re/go/scm v0.17.0
├── github.com/kluctl/go-embed-python v0.0.0-3.13.1-20241219-1
├── golang.org/x/term v0.42.0
└── gopkg.in/yaml.v3 v3.0.1

Core Framework:
├── dappco.re/go v0.10.3
├── dappco.re/go/cli v0.10.0
└── dappco.re/go/process v0.14.0
```

---

## Subpackages

### Core packages

| Package | Description | Service | Files | Status |
|---------|-------------|---------|-------|--------|
| [ansible](file:///Users/snider/Code/core/go-devops/go/ansible) | Pure Go Ansible Executor | No | ~20 | Production |
| [build](file:///Users/snider/Code/core/go-devops/go/build) | Multi-Target Build Pipeline | No | ~15 | Production |
| [release](file:///Users/snider/Code/core/go-devops/go/release) | Release Orchestration | No | ~10 | Production |
| [deploy](file:///Users/snider/Code/core/go-devops/go/deploy) | Deployment Utilities | Yes | ~10 | Production |
| [devkit](file:///Users/snider/Code/core/go-devops/go/devkit) | Developer Toolkit | Yes | ~10 | Production |
| [snapshot](file:///Users/snider/Code/core/go-devops/go/snapshot) | Snapshot Management | No | ~5 | Production |
| [locales](file:///Users/snider/Code/core/go-devops/go/locales) | Localisation Support | No | ~2 | Production |

### Deploy subpackages

| Package | Description | Files | Status |
|---------|-------------|-------|--------|
| [deploy/coolify](file:///Users/snider/Code/core/go-devops/go/deploy/coolify) | Coolify Deployment | ~5 | Production |
| [deploy/python](file:///Users/snider/Code/core/go-devops/go/deploy/python) | Python Deployment | ~3 | Production |

### CLI commands

| Command | Description | Package | Files |
|---------|-------------|---------|-------|
| `core devops deploy` | Deployment management | [cmd/deploy](file:///Users/snider/Code/core/go-devops/go/cmd/deploy) | ~8 |
| `core devops setup` | Setup wizard | [cmd/setup](file:///Users/snider/Code/core/go-devops/go/cmd/setup) | ~15 |
| `core devops workspace` | Workspace management | [cmd/workspace](file:///Users/snider/Code/core/go-devops/go/cmd/workspace) | ~3 |
| `core devops vanity-import` | Vanity import management | [cmd/vanity-import](file:///Users/snider/Code/core/go-devops/go/cmd/vanity-import) | ~1 |

---

## Configuration

### File locations

| File | Purpose |
|------|---------|
| `.core/devops.yaml` | Main devops configuration |
| `.core/build.yaml` | Build pipeline configuration |
| `.core/release.yaml` | Release configuration |
| `playbooks/*.yml` | Ansible playbooks |
| `inventory.yml` | Ansible inventory |

### Configuration schema

```yaml
# .core/devops.yaml example
version: "1.0"

ansible:
  ssh:
    timeout: 30s
    retries: 3
    pool_size: 10
  
build:
  matrix:
    - goos: linux
      goarch: amd64
    - goos: darwin
      goarch: arm64
  cache:
    enabled: true
    dir: ".cache/build"
  
deploy:
  targets:
    production:
      type: coolify
      url: https://coolify.example.com
      token: ${{ env.COOLIFY_TOKEN }}
  
release:
  publishers:
    - github
    - docker
    - homebrew
  
signing:
  macos:
    identity: "Developer ID Application: Lethean"
    notarize: true
  gpg:
    key_id: "ABCDEF1234567890"
    passphrase: ${{ env.GPG_PASSPHRASE }}
  windows:
    cert_path: "cert.pfx"
    password: ${{ env.WINDOWS_CERT_PASSWORD }}

devkit:
  coverage:
    threshold: 80
    trending: true
  secrets:
    patterns:
      - "*.env"
      - "*.pem"
      - "*.key"
  complexity:
    threshold: 10
    max: 25
```

### Environment variables

| Variable | Description | Required |
|----------|-------------|----------|
| `COOLIFY_TOKEN` | Coolify API token | No |
| `HETZNER_TOKEN` | Hetzner Cloud API token | No |
| `ROBOT_USER` | Hetzner Robot username | No |
| `ROBOT_PASSWORD` | Hetzner Robot password | No |
| `CLOUDNS_API_KEY` | CloudNS API key | No |
| `GITHUB_TOKEN` | GitHub API token | No |
| `GPG_PASSPHRASE` | GPG signing passphrase | No |

---

## Commands

### Deployment (`core devops deploy`)

| Command | Description | Options |
|---------|-------------|---------|
| `deploy <target>` | Deploy to target | `--playbook`, `--inventory`, `--dry-run` |
| `deploy --list` | List targets | `-v` (verbose) |
| `deploy --validate` | Validate configuration | `--strict` |

### Setup (`core devops setup`)

| Command | Description | Options |
|---------|-------------|---------|
| `setup wizard` | Interactive setup | `--non-interactive` |
| `setup repo` | Repository configuration | `--template`, `--force` |
| `setup github-webhooks` | GitHub webhooks | `--repo`, `--url`, `--secret` |
| `setup github-security` | GitHub security config | `--repo`, `--token` |

### Workspace (`core devops workspace`)

| Command | Description | Options |
|---------|-------------|---------|
| `workspace config` | Show configuration | `--yaml`, `--json` |
| `workspace validate` | Validate workspace | `--strict` |

---

## Usage patterns

### 1. Ansible playbook execution

```go
import "forge.lthn.ai/core/go-devops/ansible"

ctx := context.Background()

// Parse and run playbook
pb, _ := ansible.ParsePlaybook("playbooks/deploy.yml")
inv, _ := ansible.ParseInventory("inventory.yml")
result := pb.Run(ctx, inv)
```

### 2. Multi-target build

```go
import "forge.lthn.ai/core/go-devops/build"

ctx := context.Background()

// Auto-detect project type and build
artifacts, err := build.Build(ctx, ".", 
    build.WithMatrix("linux/amd64", "darwin/arm64"),
    build.WithCache(true),
)
```

### 3. Release publishing

```go
import "forge.lthn.ai/core/go-devops/release"

ctx := context.Background()

cfg := release.Config{
    Publishers: []string{"github", "docker", "homebrew"},
    Sign:       true,
    Changelog:  true,
}

if err := release.Publish(ctx, cfg, artifacts...); err != nil {
    log.Fatal(err)
}
```

### 4. Secret scanning

```go
import "forge.lthn.ai/core/go-devops/devkit"

ctx := context.Background()

secrets, err := devkit.ScanSecrets(ctx, ".")
for _, s := range secrets {
    fmt.Printf("Secret at %s:%d: %s\n", s.File, s.Line, s.Type)
}
```

### 5. Coverage analysis

```go
import "forge.lthn.ai/core/go-devops/devkit"

ctx := context.Background()

report, err := devkit.AnalyzeCoverage(ctx, "coverage.out")
fmt.Printf("Coverage: %.2f%%\n", report.Percent)
```

---

## Testing

### Test coverage summary

| Package | Functions | Coverage |
|---------|-----------|----------|
| `ansible/` | ~100 | >80% |
| `build/` | ~80 | >85% |
| `deploy/` | ~50 | >75% |
| `devkit/` | ~60 | >80% |
| `release/` | ~70 | >80% |
| `snapshot/` | ~20 | >70% |

### Test commands

```bash
# All tests
go test ./...

# With race detector
go test -race ./...

# Specific package
go test ./go/devkit/...

# With coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# Verbose output
go test -v ./go/ansible/...
```

### AX standard compliance

All packages follow AX standard:
- Each `.go` file has corresponding `_test.go`
- Each `.go` file has corresponding `_example_test.go`
- Comments are agent-first, not human-first
- No external dependencies where CoreGo primitives exist

---

## API reference

### Ansible package

```go
// Core types
type Playbook struct {
    Name        string
    Description string
    Tasks       []Task
    Handlers    []Handler
    Vars        map[string]interface{}
}

type Task struct {
    Name     string
    Module   string
    Args     map[string]interface{}
    When     string
    Loop     []interface{}
    Retries  int
    Register string
}

// Core functions
func ParsePlaybook(path string) (*Playbook, error)
func ParseInventory(path string) (*Inventory, error)
func (pb *Playbook) Run(ctx context.Context, inv *Inventory) *PlaybookResult
func (pb *Playbook) Validate() error
```

### DevKit package

```go
type CoverageReport struct {
    Percent      float64
    TotalFuncs   int
    CoveredFuncs int
    TotalLines   int
    CoveredLines int
    Files        []CoverageFile
}

type SecretResult struct {
    File    string
    Line    int
    Column  int
    Type    string
    Content string
    Severity string
}

// Functions
func AnalyzeCoverage(ctx context.Context, profilePath string) (*CoverageReport, error)
func ScanSecrets(ctx context.Context, root string, opts ...ScanOption) ([]SecretResult, error)
func ScanComplexity(ctx context.Context, root string) ([]ComplexityResult, error)
```

### Build package

```go
type Builder interface {
    Name() string
    Detect(ctx context.Context, dir string) bool
    Build(ctx context.Context, dir string, opts BuildOptions) ([]Artifact, error)
}

type Artifact struct {
    Name     string
    Path     string
    GOOS     string
    GOARCH   string
    Type     string
    Checksum string
}

func Build(ctx context.Context, dir string, opts ...BuildOption) ([]Artifact, error)
func RegisterBuilder(b Builder)
func GetBuilder(projectType string) Builder
```

### Release package

```go
type Publisher interface {
    Name() string
    Publish(ctx context.Context, cfg Config, artifacts []Artifact) error
}

type Config struct {
    Publishers   []string
    Sign        bool
    Changelog   bool
    DryRun      bool
    Parallel    bool
}

func Publish(ctx context.Context, cfg Config, artifacts ...Artifact) error
func RegisterPublisher(p Publisher)
```

---

## Related documentation

### Internal documentation

| File | Description | Location |
|------|-------------|----------|
| Architecture | Ansible integration, build pipeline, infrastructure APIs | [docs/architecture.md](file:///Users/snider/Code/core/go-devops/docs/architecture.md) |
| Development Guide | Building, testing, coding standards | [docs/development.md](file:///Users/snider/Code/core/go-devops/docs/development.md) |
| Project History | Completed phases, known limitations | [docs/history.md](file:///Users/snider/Code/core/go-devops/docs/history.md) |
| Build System | Build pipeline details | [docs/build-system.md](file:///Users/snider/Code/core/go-devops/docs/build-system.md) |
| Publishers | Publisher backends documentation | [docs/publishers.md](file:///Users/snider/Code/core/go-devops/docs/publishers.md) |
| SDK Generation | SDK generation guide | [docs/sdk-generation.md](file:///Users/snider/Code/core/go-devops/docs/sdk-generation.md) |
| Index | Main documentation index | [docs/index.md](file:///Users/snider/Code/core/go-devops/docs/index.md) |
| Sync | Synchronisation mechanisms | [docs/sync.md](file:///Users/snider/Code/core/go-devops/docs/sync.md) |

### External references

| Resource | URL |
|----------|-----|
| Repository | [github.com/dappcore/go-devops](https://github.com/dappcore/go-devops) |
| Module | [pkg.go.dev/dappco.re/go/devops](https://pkg.go.dev/dappco.re/go/devops) |
| RFC | [plans/code/core/go/devops/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/devops/RFC.md) |
| Forgejo | [forge.lthn.ai/core/go-devops](forge.lthn.ai/core/go-devops) |

---

## Statistics

### File counts

```
Total Files:          200+
  └── Go Files:        139+
      ├── Library:      25
      ├── CLI:          50+
      └── Tests:        50+
  └── Documentation:   15+
      ├── Markdown:     12
      └── YAML:          3

Lines of Code:
  └── Go:              ~15,000
  └── Markdown:        ~3,000
  └── YAML:            ~500
  └── Total:           ~18,500
```

### Test coverage breakdown

| Category | Count | Coverage |
|----------|-------|----------|
| Unit Tests | 100+ | >80% |
| Example Tests | 25+ | N/A |
| Integration Tests | 10+ | >75% |

### Dependency analysis

```
Total Dependencies:    15+
  └── CoreGo:           8 (agent, i18n, io, log, process, cli, core, scm)
  └── External:         7 (gitea-sdk, go-embed-python, yaml, term, text, httpsig, cobra)
  └── Indirect:         30+
```
