# go-scm — Source Control Management

> **Package:** `dappco.re/go/scm`  
> **Repository:** [`github.com/dappcore/go-scm`](https://github.com/dappcore/go-scm)  
> **Spec:** [`plans/code/core/go/scm/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/scm/RFC.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>  
> **Status:** ✅ Production Ready  

---

## 📋 Overview

**go-scm** is the source control management package for the Core ecosystem. It provides comprehensive multi-repo management, git-based marketplace operations, Forge/Gitea integration, and data collection capabilities.

### 🎯 Key Features

| Category | Features |
|----------|----------|
| **Multi-Repo Management** | repos.yaml registry, health checks, batch operations across 51+ repos |
| **Package Manifests** | manifest.yaml → core.json compilation, ed25519 signing |
| **Git-Based Marketplace** | Package discovery, installation, version resolution via git tags |
| **Forge/Gitea Integration** | Full API clients for repos, PRs, issues, mirroring, sync |
| **Data Collection** | GitHub, BitcoinTalk, market data, academic papers |
| **Job Runner** | Forgejo CI runner integration with event-driven architecture |
| **Agent CI** | Agent-driven CI workflows with security validation |
| **Plugin System** | Dynamic SCM extension loading |
| **Git Abstraction** | Unified git operations layer |

---

## 🏗️ Architecture

### Repository Structure

```
core/go-scm/
├── go/                          # Go module root (dappco.re/go/scm)
│   ├── agentci/                 # Agent-driven CI workflows
│   │   ├── config.go            # Agent CI configuration
│   │   ├── security.go          # Security validation
│   │   └── clotho.go            # Workflow orchestration
│   │
│   ├── collect/                 # Data collection (6 subpackages)
│   │   ├── bitcointalk.go       # BitcoinTalk forum data
│   │   ├── github.go            # GitHub repository data
│   │   ├── market.go            # Market/price data
│   │   ├── papers.go            # Academic papers
│   │   ├── process.go           # Parallel processing
│   │   ├── events.go            # Collection events
│   │   ├── state.go             # Persistent state
│   │   └── ratelimit.go         # API rate limiting
│   │
│   ├── forge/                   # Forgejo API client
│   │   ├── client.go            # HTTP client
│   │   ├── repos.go             # Repository operations
│   │   ├── prs.go               # Pull request operations
│   │   ├── orgs.go              # Organization operations
│   │   └── auth.go              # Authentication
│   │
│   ├── gitea/                   # Gitea API client
│   │   ├── client.go            # HTTP client
│   │   ├── repos.go             # Repository operations
│   │   ├── prs.go               # Pull request operations
│   │   ├── issues.go            # Issue operations
│   │   ├── mirror.go            # Repository mirroring
│   │   ├── sync.go              # Repository sync
│   │   └── config.go            # Configuration
│   │
│   ├── git/                     # Git abstraction layer
│   │   ├── git.go               # Core git operations
│   │   ├── service.go           # Git service
│   │   └── ...                  # Git utilities
│   │
│   ├── jobrunner/              # Forgejo CI job runner
│   │   ├── types.go             # Signal/Source/Handler interfaces
│   │   ├── poller.go            # Event poller
│   │   ├── journal.go           # Event journal
│   │   ├── forgejo/             # Forgejo integration
│   │   │   ├── source.go        # Forgejo webhook source
│   │   │   └── signals.go       # Signal type definitions
│   │   └── handlers/            # Event handlers
│   │       ├── dispatch.go      # Dispatch handler
│   │       ├── merge.go         # Merge handler
│   │       ├── resolve.go       # Resolve handler
│   │       └── ...
│   │
│   ├── manifest/                # Package manifest system
│   │   ├── manifest.go          # Manifest parsing/validation
│   │   ├── compile.go           # manifest.yaml → core.json
│   │   ├── sign.go              # Ed25519 signing
│   │   └── verify.go            # Signature verification
│   │
│   ├── marketplace/             # Git-based package marketplace
│   │   ├── index.go             # Package index
│   │   ├── search.go            # Package search
│   │   ├── install.go           # Package installation
│   │   ├── update.go            # Package updates
│   │   ├── publish.go           # Package publishing
│   │   └── scanner.go           # Marketplace scanner
│   │
│   ├── plugin/                  # Plugin system
│   │   ├── registry.go          # Plugin discovery/lifecycle
│   │   ├── loader.go            # Plugin loading
│   │   ├── manifest.go          # Plugin manifest parsing
│   │   ├── installer.go         # Plugin installation
│   │   └── config.go            # Plugin configuration
│   │
│   ├── repos/                   # Multi-repo registry
│   │   ├── registry.go          # Repository registry
│   │   ├── service.go           # Registry service
│   │   ├── workconfig.go        # Work configuration
│   │   ├── kbconfig.go          # Knowledge base configuration
│   │   ├── gitstate.go          # Git state tracking
│   │   └── service_sync.go      # Sync service
│   │
│   ├── pkg/                     # API surface
│   │   └── api/                 # REST API
│   │       ├── provider.go      # API provider
│   │       ├── embed.go          # Embedded UI assets
│   │       └── ...
│   │
│   ├── cmd/                     # CLI commands (6 commands)
│   │   ├── scm/                  # Main SCM commands
│   │   │   └── main.go
│   │   ├── forge/                # Forge CLI
│   │   │   └── main.go
│   │   ├── gitea/                # Gitea CLI
│   │   │   └── main.go
│   │   ├── collect/              # Collection CLI
│   │   │   └── main.go
│   │   ├── verify/               # Verification commands
│   │   │   └── cmd.go
│   │   ├── compile/              # Manifest compilation
│   │   │   └── cmd.go
│   │   └── sign/                 # Manifest signing
│   │       └── cmd.go
│   │
│   ├── scm.go                   # Main SCM package
│   ├── service.go               # Core service
│   ├── scm_test.go              # Unit tests
│   └── scm_example_test.go      # Example tests
│
├── go.work
├── go.work.sum
├── docs/
├── external/
├── playbooks/
├── README.md
├── AGENTS.md
├── CLAUDE.md
├── LICENCE
└── sonar-project.properties
```

### Module Information

- **Module Path:** `dappco.re/go/scm`
- **Go Version:** 1.22+
- **Test Files:** 50+
- **Total Lines:** ~150K+

---

## 🔍 Package Deep Dive

### scm/ — Main SCM Package

The root package provides the main SCM functionality and service integration.

**scm.go:**
```go
// Package scm provides source control management for the Core ecosystem.
// It handles multi-repo management, git-based marketplace, and Forge/Gitea integration.
package scm

// Main entry points
func New() *SCM
func (s *SCM) Init() core.Result
func (s *SCM) Close() core.Result
```

**service.go:**
```go
// SCMService integrates with Core framework
type SCMService struct {
    Config *Config
}

func NewCoreService(cfg *Config) *SCMService
func (s *SCMService) Name() string
func (s *SCMService) Run(ctx context.Context) core.Result
```

### manifest/ — Package Manifest System

**Purpose:** Parse, validate, sign manifest.yaml and compile to core.json build artifacts.

**Manifest Format (Source):**
```yaml
# .core/manifest.yaml
code: go-io
name: Core I/O
version: 0.3.0
description: Mandatory I/O abstraction layer
licence: EUPL-1.2
sign: <ed25519 signature>
sign_key: <ed25519 public key>
```

**Core.json Format (Build Artifact):**
```json
{
  "code": "go-io",
  "name": "Core I/O",
  "version": "0.3.0",
  "description": "Mandatory I/O abstraction layer",
  "licence": "EUPL-1.2",
  "sign": "...",
  "sign_key": "...",
  "build": {
    "targets": ["linux/amd64", "darwin/arm64"],
    "checksums": "SHA-256"
  }
}
```

**Key Files:**

| File | Purpose | Key Types/Functions |
|------|---------|---------------------|
| `manifest.go` | Parse and validate manifest.yaml | `Manifest`, `Parse()`, `Validate()` |
| `compile.go` | Compile manifest.yaml → core.json | `Compile()`, `CompileToCoreJSON()` |
| `sign.go` | Ed25519 signature operations | `Sign()`, `VerifySignature()` |
| `verify.go` | Manifest verification | `Verify()`, `VerifyIntegrity()` |

**Manifest Type:**
```go
type Manifest struct {
    Code        string `yaml:"code" json:"code"`
    Name        string `yaml:"name" json:"name"`
    Version     string `yaml:"version" json:"version"`
    Description string `yaml:"description" json:"description"`
    Licence     string `yaml:"licence" json:"licence"`
    Sign        string `yaml:"sign" json:"sign"`
    SignKey     string `yaml:"sign_key" json:"sign_key"`
    // Extended fields
    Element     *ElementSpec `yaml:"element,omitempty" json:"element,omitempty"`
    Daemons     []DaemonSpec  `yaml:"daemons,omitempty" json:"daemons,omitempty"`
    Namespace   string         `yaml:"namespace,omitempty" json:"namespace,omitempty"`
}

// ElementSpec for GUI rendering
type ElementSpec struct {
    Tag    string `yaml:"tag" json:"tag"`
    Source string `yaml:"source" json:"source"`
}

// DaemonSpec for long-running processes
type DaemonSpec struct {
    Name     string `yaml:"name" json:"name"`
    Binary   string `yaml:"binary" json:"binary"`
    Args     []string `yaml:"args" json:"args"`
    Health   string `yaml:"health" json:"health"`
    Default  bool   `yaml:"default" json:"default"`
}
```

**Signing Process:**
1. Parse manifest.yaml
2. Validate structure and required fields
3. Compute SHA-256 hash of content
4. Sign hash with ed25519 private key
5. Store signature in `sign` field
6. Store public key in `sign_key` field

**Verification Process:**
1. Parse manifest
2. Extract signature and public key
3. Recompute hash of content (excluding sign fields)
4. Verify signature with public key
5. Return verification result

**Why Git Tags ARE Versions:**
- No separate version registry
- Git tags are the source of truth
- `v0.3.0` tag exists → version 0.3.0 exists
- Tag is signed → version is verified
- `go get dappco.re/go/io@v0.3.0` resolves via tag
- Marketplace indexes tags, not a database

### repos/ — Multi-Repo Registry

**Purpose:** Manage multiple repositories via repos.yaml registry with health checks and batch operations.

**Registry File Locations:**
1. `.core/repos.yaml` — Project-specific registry
2. `~/.core/repos.yaml` — System-wide registry

**Registry Format:**
```yaml
# .core/repos.yaml or ~/.core/repos.yaml
repos:
  - path: core/go
    remote: ssh://git@forge.lthn.ai:2223/core/go.git
    branch: dev
    workspace: core/go
  - path: core/go-io
    remote: ssh://git@forge.lthn.ai:2223/core/go-io.git
    branch: dev
    workspace: core/go-io
  # ... 51 repos
```

**Registry Entry:**
```go
type RepoEntry struct {
    Path      string `yaml:"path"`
    Remote    string `yaml:"remote"`
    Branch    string `yaml:"branch"`
    Workspace string `yaml:"workspace,omitempty"`
    // Additional fields
    Sync      bool   `yaml:"sync,omitempty"`
    Push      bool   `yaml:"push,omitempty"`
    Pull      bool   `yaml:"pull,omitempty"`
}

type Registry struct {
    Version string      `yaml:"version"`
    Repos   []RepoEntry `yaml:"repos"`
}
```

**Discovery:**
- System registry discovered via env `CORE_REPOS`
- Falls back to conventional location `~/.core/repos.yaml`
- Project registry at `.core/repos.yaml`
- Operations work across all discovered repos

**Key Files:**

| File | Purpose |
|------|---------|
| `registry.go` | Registry loading and management |
| `service.go` | Registry service with health checks |
| `workconfig.go` | Work configuration for batch operations |
| `kbconfig.go` | Knowledge base configuration |
| `gitstate.go` | Git state tracking (dirty, ahead/behind) |
| `service_sync.go` | Sync service for repository synchronization |

**Operations:**

| Operation | Command | Purpose |
|-----------|---------|---------|
| Health Check | `core dev health` | Status across all repos (clean/dirty, synced/ahead/behind) |
| Work | `core dev work` | Full workflow: status → commit → push |
| Commit | `core dev commit` | Claude-assisted commits for dirty repos |
| Push | `core dev push` | Push repos with unpushed commits |
| Pull | `core dev pull` | Pull repos that are behind |
| Impact | `core dev impact {pkg}` | Show dependency graph impact |

**Repository States:**
- **Clean** — No uncommitted changes
- **Dirty** — Has uncommitted changes
- **Synced** — Up to date with remote
- **Ahead** — Local commits not pushed
- **Behind** — Remote has commits not pulled

### marketplace/ — Git-Based Marketplace

**Purpose:** Git-based package registry with no separate server. Packages are discovered via git repositories with manifest.yaml files.

**Design Principles:**
1. Each package has `.core/manifest.yaml`
2. Manifests are indexed by marketplace scanner
3. `core pkg install {name}` clones and verifies
4. ed25519 signatures verified before installation
5. Version resolution via git tags

**Key Files:**

| File | Purpose |
|------|---------|
| `index.go` | Package index management |
| `search.go` | Package search functionality |
| `install.go` | Package installation |
| `update.go` | Package updates |
| `publish.go` | Package publishing |
| `scanner.go` | Marketplace scanning |

**Marketplace Index:**
- Scans repositories for manifest.yaml files
- Builds index of available packages
- Resolves package names to repository URLs
- Caches index for performance

**Package Identification:**
- Package name = `code` field from manifest.yaml
- Package version = git tag
- Package repository = git remote URL

**Commands:**

| Command | Purpose |
|---------|---------|
| `core pkg search {query}` | Search marketplace index |
| `core pkg install {name}` | Clone, verify signature, install |
| `core pkg update {name}` | Pull latest, verify, update |
| `core pkg list` | List installed packages |
| `core pkg publish` | Push manifest to marketplace index |

**Installation Process:**
1. Search marketplace index for package name
2. Resolve package to repository URL
3. Clone repository
4. Verify manifest.yaml signature
5. Copy package to destination
6. Return installation result

### forge/ — Forgejo API Client

**Purpose:** Full Forgejo API client for repository, PR, organization, and authentication operations.

**Key Files:**

| File | Purpose | API Coverage |
|------|---------|--------------|
| `client.go` | HTTP client and configuration | Connection management |
| `repos.go` | Repository operations | CRUD, forking, mirroring |
| `prs.go` | Pull request operations | CRUD, reviews, merging |
| `orgs.go` | Organization operations | CRUD, membership |
| `auth.go` | Authentication | Token, SSH key management |

**Client Type:**
```go
type Client struct {
    Endpoint string
    Token    string
    Timeout time.Duration
}

func NewClient(endpoint, token string) *Client
func (c *Client) Do(ctx context.Context, method, path string, body, result any) core.Result
```

**Repository Operations:**
- List repositories
- Get repository details
- Create repository
- Update repository
- Delete repository
- Fork repository
- Mirror repository (from GitHub, etc.)
- Get repository contents
- Get repository branches
- Get repository tags
- Get repository commits

**Pull Request Operations:**
- List PRs
- Get PR details
- Create PR
- Update PR
- Close PR
- Merge PR
- Add PR review
- Get PR reviews
- Get PR comments
- Add PR comment

**Organization Operations:**
- List organizations
- Get organization details
- Create organization
- Update organization
- Delete organization
- List organization members
- Add organization member
- Remove organization member

### gitea/ — Gitea API Client

**Purpose:** Full Gitea API client for repository, PR, issue, mirroring, and sync operations.

**Key Files:**

| File | Purpose | API Coverage |
|------|---------|--------------|
| `client.go` | HTTP client and configuration | Connection management |
| `repos.go` | Repository operations | CRUD, forking, mirroring |
| `prs.go` | Pull request operations | CRUD, reviews, merging |
| `issues.go` | Issue operations | CRUD, labels, milestones |
| `mirror.go` | Repository mirroring | Create, update, delete |
| `sync.go` | Repository synchronization | Sync status, trigger |
| `config.go` | Configuration | Server settings |

**Gitea-Specific Features:**
- Issue tracker management
- Milestone management
- Label management
- Webhook management
- Repository mirroring (one-way sync from external sources)
- Repository sync (two-way sync with external sources)

**Mirror Operations:**
- Create mirror repository
- Update mirror settings
- Delete mirror repository
- Sync mirror repository
- List mirror repositories

**Sync Operations:**
- Trigger repository sync
- Get sync status
- List sync operations
- Configure sync settings

### git/ — Git Abstraction Layer

**Purpose:** Unified git operations abstraction for consistent git interactions.

**Key Files:**

| File | Purpose |
|------|---------|
| `git.go` | Core git operations |
| `service.go` | Git service with context-aware support |

**Git Service:**
```go
type GitService struct {
    Config *GitConfig
}

func NewGitService(cfg *GitConfig) *GitService
func (s *GitService) Clone(ctx context.Context, url, path string, opts CloneOptions) core.Result
func (s *GitService) Fetch(ctx context.Context, path string, opts FetchOptions) core.Result
func (s *GitService) Pull(ctx context.Context, path string, opts PullOptions) core.Result
func (s *GitService) Push(ctx context.Context, path string, opts PushOptions) core.Result
func (s *GitService) Status(ctx context.Context, path string) (GitStatus, core.Result)
```

**Operations:**
- Clone repositories from remote URLs
- Fetch/pull/push with branch tracking
- Branch creation and switching
- Status queries (dirty, ahead/behind)
- Signature verification
- SSH key management
- Tag management
- Commit management

**CloneOptions:**
```go
type CloneOptions struct {
    Branch    string
    Depth     int
    Recurse   bool
    SSHKey    string
    Auth     GitAuth
}
```

**GitAuth:**
```go
type GitAuth struct {
    Method string // "basic", "token", "ssh", "none"
    Token  string // Token for token auth
    User   string // Username for basic auth
    Pass   string // Password for basic auth
}
```

### collect/ — Data Collection System

**Purpose:** Collect external data from various sources including GitHub, BitcoinTalk, market data, and academic papers.

**Key Files:**

| File | Purpose |
|------|---------|
| `bitcointalk.go` | BitcoinTalk forum data collection |
| `github.go` | GitHub repository metadata collection |
| `market.go` | Market/price data collection |
| `papers.go` | Academic papers collection |
| `process.go` | Parallel processing of collection tasks |
| `events.go` | Collection event types |
| `state.go` | Persistent state management |
| `ratelimit.go` | API rate limiting |

**Collection Sources:**

| Source | Command | Data Collected |
|--------|---------|---------------|
| GitHub | `core collect github` | Repository metadata, stars, contributors, forks, issues, PRs |
| BitcoinTalk | `core collect bitcointalk` | Forum threads, posts, community sentiment |
| Market | `core collect market` | Price data, volume, exchanges, trading pairs |
| Papers | `core collect papers` | Academic papers, research publications |

**Collection Process:**
1. Parse collection configuration
2. Load persistent state (last run, rate limits)
3. Dispatch collection jobs to worker agents
4. Apply rate limiting per source
5. Store collected data
6. Update persistent state

**Event Types:**
```go
type CollectionEvent struct {
    Type      string `json:"type"`
    Source    string `json:"source"`
    Timestamp time.Time `json:"timestamp"`
    Data      any    `json:"data"`
}
```

**Rate Limiting:**
- Per-source rate limits
- Configurable limits
- Token bucket algorithm
- Automatic backoff

### jobrunner/ — Forgejo CI Job Runner

**Purpose:** Forgejo CI integration with event-driven architecture for running CI jobs.

**Architecture:**
```
Forge webhook → Job Runner picks up → Executes handlers → Reports status
```

**Key Files:**

| File | Purpose |
|------|---------|
| `types.go` | Signal, Source, Handler interfaces |
| `poller.go` | Event poller for sources |
| `journal.go` | Event journal for audit trails |
| `forgejo/source.go` | Forgejo webhook source |
| `forgejo/signals.go` | Forgejo signal types |
| `handlers/dispatch.go` | Dispatch handler |
| `handlers/merge.go` | Merge handler |
| `handlers/resolve.go` | Resolve handler |

**Signal Interface:**
```go
type Signal interface {
    Type() string
    Source() string
    Payload() any
}
```

**Source Interface:**
```go
type Source interface {
    Name() string
    Poll(ctx context.Context) ([]Signal, error)
    Listen(ctx context.Context, handler Handler) error
}
```

**Handler Interface:**
```go
type Handler interface {
    Type() string
    Handle(ctx context.Context, signal Signal) core.Result
}
```

**Poller Features:**
- Multiple sources (Forgejo, GitHub Actions, etc.)
- Handler chain (first match wins)
- Dry-run mode for testing
- Cycle tracking (completed poll-dispatch iterations)
- Dynamic source/handler addition

**Journal:**
- Records all collection events
- Provides audit trail
- Persistent storage
- Queryable interface

### agentci/ — Agent-Driven CI Workflows

**Purpose:** Agent-driven CI workflows with security validation and orchestration.

**Key Files:**

| File | Purpose |
|------|---------|
| `config.go` | Configuration management for agent CI |
| `security.go` | Security validation for agent operations |
| `clotho.go` | Workflow orchestration (Clotho = one of the Fates) |

**Configuration:**
```go
type AgentCIConfig struct {
    Enabled        bool
    Workspace      string
    BuildTimeout   time.Duration
    TestTimeout    time.Duration
    DeployTimeout  time.Duration
    Security       SecurityConfig
}

type SecurityConfig struct {
    RequireApproval bool
    MaxConcurrency  int
    AllowedActions  []string
}
```

**Clotho Orchestrator:**
- Manages workflow lifecycle
- Handles task dependencies
- Provides rollback capabilities
- Integrates with external systems

**Security Features:**
- Action approval workflow
- Concurrency limits
- Allowed action list
- Audit logging

### plugin/ — Plugin System

**Purpose:** Dynamic loading of SCM extensions via plugin manifests.

**Key Files:**

| File | Purpose |
|------|---------|
| `registry.go` | Plugin discovery and lifecycle management |
| `loader.go` | Load plugins from manifest declarations |
| `manifest.go` | Plugin manifest parsing |
| `installer.go` | Download and install plugins |
| `config.go` | Plugin configuration management |
| `plugin.go` | Plugin interface and runtime |

**Plugin Discovery:**
- Loaded from manifest Namespace declarations
- Each CoreApp can declare plugins it provides
- Plugins are loaded on demand
- Lifecycle: Discover → Load → Configure → Run → Unload

**Plugin Manifest:**
```yaml
# .core/manifest.yaml
namespace: my-namespace
plugins:
  - name: my-plugin
    description: My custom SCM plugin
    version: 1.0.0
    entry: ./bin/my-plugin
    config: ./config/my-plugin.yaml
    permissions: ["read", "write", "network"]
```

**Plugin Interface:**
```go
type Plugin interface {
    Name() string
    Version() string
    Description() string
    Initialize(ctx context.Context, config any) error
    Run(ctx context.Context) error
    Stop(ctx context.Context) error
}
```

**Registry:**
```go
type Registry struct {
    Plugins map[string]*PluginInfo
}

func (r *Registry) Discover() error
func (r *Registry) Load(name string) (Plugin, error)
func (r *Registry) Unload(name string) error
func (r *Registry) Configure(name string, config any) error
```

### pkg/api/ — REST API Surface

**Purpose:** REST API for SCM operations with embedded UI assets.

**Key Files:**

| File | Purpose |
|------|---------|
| `provider.go` | API provider implementing Core API service |
| `embed.go` | Embedded HTML/CSS/JS assets |

**API Endpoints:**
```
GET  /api/scm/repos              - List registered repositories
GET  /api/scm/repos/{repo}       - Get repository details
POST /api/scm/repos/{repo}/sync  - Sync repository
GET  /api/scm/repos/status        - Get repository status
POST /api/scm/manifests/compile   - Compile manifest
POST /api/scm/manifests/verify    - Verify manifest
GET  /api/scm/marketplace/packages - List marketplace packages
GET  /api/scm/marketplace/search   - Search marketplace
```

**UI Assets:**
- Embedded static files
- Angular-based frontend
- Real-time updates via WebSocket
- Responsive design

---

## 🚀 Commands

### Command Structure

```
core dev       - Multi-repo operations
core pkg       - Marketplace operations
core forge     - Forge operations
core gitea     - Gitea operations
core collect   - Data collection operations
core verify    - Manifest verification
core compile   - Manifest compilation
core sign      - Manifest signing
```

### core dev — Multi-Repo Operations

**Purpose:** Manage multiple repositories in batch.

**Registration:** `cmd/scm/main.go`

**Commands:**

**1. Health:**
```bash
core dev health [--all] [--json] [--verbose]
```
- Shows status across all registered repos
- Displays clean/dirty, synced/ahead/behind for each
- `--all` includes system-wide registry
- `--json` outputs as JSON
- `--verbose` shows detailed git status

**2. Work:**
```bash
core dev work [--dry-run] [--push] [--auto-commit]
```
- Full workflow: status → commit → push
- Only operates on repos that need action
- `--dry-run` shows what would be done
- `--push` also pushes after commit
- `--auto-commit` generates commit messages automatically

**3. Commit:**
```bash
core dev commit [--all] [--message STRING] [--dry-run] [--ai]
```
- Commit changes in dirty repos
- `--all` commits all dirty repos
- `--message` specifies commit message
- `--dry-run` shows what would be committed
- `--ai` uses Claude for commit message generation

**4. Push:**
```bash
core dev push [--all] [--dry-run] [--force]
```
- Push repos with unpushed commits
- `--all` pushes all repos with unpushed commits
- `--dry-run` shows what would be pushed
- `--force` force pushes (use with caution)

**5. Pull:**
```bash
core dev pull [--all] [--dry-run] [--rebase]
```
- Pull repos that are behind
- `--all` pulls all repos that are behind
- `--dry-run` shows what would be pulled
- `--rebase` uses rebase instead of merge

**6. Impact:**
```bash
core dev impact {pkg} [--depth INT] [--json]
```
- Show dependency graph impact for a package
- `--depth` limits traversal depth
- `--json` outputs as JSON

### core pkg — Marketplace Operations

**Purpose:** Interact with the git-based marketplace.

**Registration:** `cmd/compile/main.go` + marketplace handlers

**Commands:**

**1. Search:**
```bash
core pkg search {query} [--limit INT] [--json]
```
- Search marketplace index for packages
- `--limit` limits number of results
- `--json` outputs as JSON

**2. Install:**
```bash
core pkg install {name} [--version STRING] [--force] [--no-verify]
```
- Install package from marketplace
- `--version` specifies version (git tag)
- `--force` overwrites existing installation
- `--no-verify` skips signature verification (NOT RECOMMENDED)

**3. Update:**
```bash
core pkg update [--all] [--dry-run]
```
- Update installed packages
- `--all` updates all packages
- `--dry-run` shows what would be updated

**4. List:**
```bash
core pkg list [--json] [--long]
```
- List installed packages
- `--json` outputs as JSON
- `--long` shows detailed information

**5. Publish:**
```bash
core pkg publish [--repo PATH] [--manifest PATH] [--dry-run]
```
- Push manifest to marketplace index
- `--repo` specifies repository path
- `--manifest` specifies manifest file
- `--dry-run` validates without publishing

### core forge — Forge Operations

**Purpose:** Interact with Forgejo/Forge instances.

**Registration:** `cmd/forge/main.go`

**Commands:**

**1. Repos:**
```bash
core forge repos [--org STRING] [--all] [--json]
```
- List repositories in organization
- `--org` specifies organization
- `--all` lists all repositories (not just org)
- `--json` outputs as JSON

**2. Orgs:**
```bash
core forge orgs [--json]
```
- List organizations
- `--json` outputs as JSON

**3. PRs:**
```bash
core forge prs [--repo STRING] [--state open|closed|merged] [--json]
```
- List pull requests
- `--repo` filters by repository
- `--state` filters by PR state
- `--json` outputs as JSON

**4. Auth:**
```bash
core forge auth [--token STRING] [--test]
```
- Authenticate with Forge
- `--token` specifies authentication token
- `--test` tests current authentication

### core gitea — Gitea Operations

**Purpose:** Interact with Gitea instances.

**Registration:** `cmd/gitea/main.go`

**Commands:**

**1. Repos:**
```bash
core gitea repos [--json]
```
- List repositories
- `--json` outputs as JSON

**2. PRs:**
```bash
core gitea prs [--repo STRING] [--state open|closed|merged] [--json]
```
- List pull requests
- Filters by repository and state
- `--json` outputs as JSON

**3. Issues:**
```bash
core gitea issues [--repo STRING] [--state open|closed] [--labels STRING] [--json]
```
- List issues
- Filters by repository, state, labels
- `--json` outputs as JSON

**4. Mirror:**
```bash
core gitea mirror [--repo STRING] [--remote STRING] [--interval DURATION] [--create]
```
- Configure repository mirroring
- `--repo` specifies repository to mirror
- `--remote` specifies remote repository URL
- `--interval` sets sync interval
- `--create` creates new mirror

**5. Sync:**
```bash
core gitea sync [--repo STRING] [--now]
```
- Trigger repository synchronization
- `--repo` specifies repository to sync
- `--now` triggers immediate sync

**6. Config:**
```bash
core gitea config [--get STRING] [--set STRING=VALUE] [--list]
```
- Manage Gitea configuration
- `--get` gets configuration value
- `--set` sets configuration value
- `--list` lists all configuration

### core collect — Data Collection Operations

**Purpose:** Collect external data from various sources.

**Registration:** `cmd/collect/main.go`

**Commands:**

**1. Dispatch:**
```bash
core collect dispatch [--source STRING] [--all] [--parallel INT] [--dry-run]
```
- Dispatch collection jobs to worker agents
- `--source` specifies data source
- `--all` collects from all sources
- `--parallel` sets concurrency level
- `--dry-run` shows what would be collected

**2. GitHub:**
```bash
core collect github [--repo STRING] [--org STRING] [--all] [--limit INT] [--json]
```
- Collect GitHub repository data
- `--repo` specifies repository
- `--org` specifies organization
- `--all` collects from all accessible repos
- `--limit` limits number of repos
- `--json` outputs as JSON

**3. BitcoinTalk:**
```bash
core collect bitcointalk [--thread STRING] [--board STRING] [--all] [--limit INT] [--json]
```
- Collect BitcoinTalk forum data
- `--thread` specifies thread ID
- `--board` specifies board name
- `--all` collects from all boards
- `--limit` limits number of threads
- `--json` outputs as JSON

**4. Market:**
```bash
core collect market [--exchange STRING] [--symbol STRING] [--all] [--json]
```
- Collect market/price data
- `--exchange` specifies exchange
- `--symbol` specifies trading pair
- `--all` collects from all exchanges
- `--json` outputs as JSON

**5. Papers:**
```bash
core collect papers [--query STRING] [--source STRING] [--all] [--limit INT] [--json]
```
- Collect academic papers
- `--query` specifies search query
- `--source` specifies data source
- `--all` collects from all sources
- `--limit` limits number of papers
- `--json` outputs as JSON

### core verify — Manifest Verification

**Purpose:** Verify manifest.yaml signatures and integrity.

**Registration:** `cmd/verify/cmd.go`

**Commands:**

**1. Verify:**
```bash
core verify [--manifest PATH] [--key PATH] [--json]
```
- Verify manifest signature
- `--manifest` specifies manifest file (default: .core/manifest.yaml)
- `--key` specifies public key file
- `--json` outputs as JSON

**2. Batch Verify:**
```bash
core verify batch [--dir PATH] [--recursive] [--json]
```
- Verify all manifests in directory
- `--dir` specifies directory to scan
- `--recursive` scans subdirectories
- `--json` outputs as JSON

### core compile — Manifest Compilation

**Purpose:** Compile manifest.yaml to core.json build artifacts.

**Registration:** `cmd/compile/cmd.go`

**Commands:**

**1. Compile:**
```bash
core compile [--manifest PATH] [--output PATH] [--pretty]
```
- Compile manifest.yaml to core.json
- `--manifest` specifies manifest file
- `--output` specifies output file
- `--pretty` formats JSON output

**2. Batch Compile:**
```bash
core compile batch [--dir PATH] [--recursive] [--overwrite]
```
- Compile all manifests in directory
- `--dir` specifies directory to scan
- `--recursive` scans subdirectories
- `--overwrite` overwrites existing files

### core sign — Manifest Signing

**Purpose:** Sign manifest.yaml with ed25519 keys.

**Registration:** `cmd/sign/cmd.go`

**Commands:**

**1. Sign:**
```bash
core sign [--manifest PATH] [--key PATH] [--output PATH] [--overwrite]
```
- Sign manifest with private key
- `--manifest` specifies manifest file
- `--key` specifies private key file
- `--output` specifies output file
- `--overwrite` overwrites existing file

**2. Generate Keys:**
```bash
core sign keygen [--output PATH] [--overwrite]
```
- Generate new ed25519 key pair
- `--output` specifies output directory
- `--overwrite` overwrites existing keys

---

## 📝 Configuration

### Registry Files

**Project Registry:** `.core/repos.yaml`
```yaml
repos:
  - path: core/go
    remote: ssh://git@forge.lthn.ai:2223/core/go.git
    branch: dev
    workspace: core/go
  - path: core/go-io
    remote: ssh://git@forge.lthn.ai:2223/core/go-io.git
    branch: dev
    workspace: core/go-io
```

**System Registry:** `~/.core/repos.yaml`
```yaml
repos:
  - path: core/go
    remote: ssh://git@forge.lthn.ai:2223/core/go.git
    branch: dev
  - path: dappcore/knowledge-packs
    remote: https://github.com/dAppCore/knowledge-packs.git
    branch: main
```

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CORE_REPOS` | Path to system-wide repos.yaml | `~/.core/repos.yaml` |
| `CORE_HOME` | Core home directory | `~/.core` |
| `FORGE_ENDPOINT` | Forge API endpoint | `https://forge.lthn.ai` |
| `FORGE_TOKEN` | Forge authentication token | (none) |
| `GITEA_ENDPOINT` | Gitea API endpoint | `https://gitea.lthn.ai` |
| `GITEA_TOKEN` | Gitea authentication token | (none) |

### Manifest Configuration

**manifest.yaml:**
```yaml
# Required fields
code: go-scm
name: Source Control Management
version: 1.0.0
description: Multi-repo management and marketplace
licence: EUPL-1.2

# Optional fields
sign: <ed25519 signature>
sign_key: <ed25519 public key>

# GUI element specification
-element:
  tag: "core-scm-widget"
  source: "./dist/scm.js"

# Daemon specification
daemons:
  sync:
    binary: "./bin/sync"
    args: ["--interval", "5m"]
    health: "/healthz"
    default: true

# Namespace for plugin discovery
namespace: scm

# Plugin declarations
plugins:
  - name: my-scm-plugin
    description: Custom SCM extension
    version: 1.0.0
    entry: "./bin/my-plugin"
    config: "./config/plugin.yaml"
```

---

## 🔗 Dependencies

### CoreGo Dependencies

```go
import (
    "dappco.re/go"                    // Core framework
    "dappco.re/go/scm"                // This package
    "dappco.re/go/io"                 // Storage abstractions
    "dappco.re/go/log"                // Logging
    "dappco.re/go/config"             // Configuration management
)
```

### External Dependencies

```
# From go.mod
github.com/go-git/go-git/v5      - Git operations
github.com/google/go-github/v62 - GitHub API
```

### Related Packages

| Package | Relationship | Purpose |
|---------|--------------|---------|
| `go-config` | Dependency | Configuration (manifests, repos.yaml) |
| `go-build` | Related | Build artifacts and compilation |
| `go-ai` | Related | AI integration for commit messages |

---

## 🧪 Testing

### Test Structure

All packages follow the **AX-7 Triplet Pattern**:

```
manifest/
├── manifest.go                # Main implementation
├── manifest_test.go           # Unit tests
├── manifest_example_test.go   # Usage examples
└── stdlib_assert_test.go       # Standard library compliance
```

### Test Coverage

**Key Test Files:**
- `scm_test.go` — Main SCM package tests
- `service_test.go` — Service tests
- `manifest/manifest_test.go` — Manifest parsing tests
- `manifest/sign_test.go` — Signing tests
- `manifest/verify_test.go` — Verification tests
- `repos/registry_test.go` — Registry tests
- `repos/service_test.go` — Registry service tests
- `forge/*_test.go` — Forge API tests
- `gitea/*_test.go` — Gitea API tests
- `git/*_test.go` — Git abstraction tests
- `collect/*_test.go` — Collection tests
- `jobrunner/*_test.go` — Job runner tests
- `cmd/*/*_test.go` — CLI command tests

### Running Tests

```bash
# All tests
cd go
go test ./...

# Specific package
cd go
go test ./manifest/...

# With coverage
cd go
go test -cover ./...

# Specific test
cd go
go test ./manifest -run TestParse_Good

# Verbose mode
cd go
go test -v ./scm
```

---

## 📖 API Reference

### Type Index

| Type | Package | Description |
|------|---------|-------------|
| `Manifest` | `manifest` | Package manifest structure |
| `ElementSpec` | `manifest` | GUI element specification |
| `DaemonSpec` | `manifest` | Daemon specification |
| `Registry` | `repos` | Repository registry |
| `RepoEntry` | `repos` | Repository entry |
| `Client` | `forge` | Forgejo API client |
| `Client` | `gitea` | Gitea API client |
| `GitService` | `git` | Git service |
| `CollectionEvent` | `collect` | Collection event |
| `Signal` | `jobrunner` | Job signal interface |
| `Source` | `jobrunner` | Job source interface |
| `Handler` | `jobrunner` | Job handler interface |
| `SCMService` | `scm` | SCM core service |

### Function Index

| Function | Package | Description |
|----------|---------|-------------|
| `Parse()` | `manifest` | Parse manifest.yaml |
| `Validate()` | `manifest` | Validate manifest structure |
| `Compile()` | `manifest` | Compile manifest to core.json |
| `Sign()` | `manifest` | Sign manifest with ed25519 |
| `Verify()` | `manifest` | Verify manifest signature |
| `Load()` | `repos` | Load repository registry |
| `HealthCheck()` | `repos` | Check repository health |
| `NewClient()` | `forge` | Create Forgejo client |
| `NewClient()` | `gitea` | Create Gitea client |
| `Clone()` | `git` | Clone repository |
| `Dispatch()` | `collect` | Dispatch collection jobs |
| `Poll()` | `jobrunner` | Poll for job signals |
| `Handle()` | `jobrunner` | Handle job signal |

---

## 🎓 Examples

### Example 1: Load and Verify Manifest

```go
package main

import (
    "dappco.re/go/scm/manifest"
)

func main() {
    // Load manifest from file
    manifestResult := manifest.Parse(".core/manifest.yaml")
    if !manifestResult.OK {
        panic(manifestResult.Err)
    }
    m := manifestResult.Value.(*manifest.Manifest)
    
    // Validate manifest
    if err := manifest.Validate(m); err != nil {
        panic(err)
    }
    
    // Verify signature
    verifyResult := manifest.Verify(m)
    if !verifyResult.OK {
        panic(verifyResult.Err)
    }
    
    println("Manifest is valid and verified:", m.Name, m.Version)
}
```

### Example 2: Compile Manifest to core.json

```go
package main

import (
    "dappco.re/go/scm/manifest"
)

func main() {
    // Load manifest
    manifestResult := manifest.Parse(".core/manifest.yaml")
    if !manifestResult.OK {
        panic(manifestResult.Err)
    }
    
    // Compile to core.json
    compileResult := manifest.Compile(manifestResult.Value.(*manifest.Manifest))
    if !compileResult.OK {
        panic(compileResult.Err)
    }
    
    // Get compiled JSON
    jsonData := compileResult.Value.([]byte)
    
    // Save to file
    if err := os.WriteFile("core.json", jsonData, 0644); err != nil {
        panic(err)
    }
    
    println("Manifest compiled to core.json")
}
```

### Example 3: Load Repository Registry

```go
package main

import (
    "dappco.re/go/scm/repos"
)

func main() {
    // Load system registry
    registryResult := repos.Load("~/.core/repos.yaml")
    if !registryResult.OK {
        panic(registryResult.Err)
    }
    registry := registryResult.Value.(*repos.Registry)
    
    println("Loaded registry with", len(registry.Repos), "repositories")
    
    // Check health of all repos
    healthResult := repos.HealthCheck(registry)
    if !healthResult.OK {
        panic(healthResult.Err)
    }
    
    health := healthResult.Value.(repos.HealthReport)
    println("Health check complete:", health.Summary)
}
```

### Example 4: Forgejo API Client

```go
package main

import (
    "context"
    "dappco.re/go/scm/forge"
)

func main() {
    // Create client
    client := forge.NewClient("https://forge.lthn.ai", os.Getenv("FORGE_TOKEN"))
    
    // List repositories
    reposResult := client.ListRepos(context.Background(), "core", 0, 100)
    if !reposResult.OK {
        panic(reposResult.Err)
    }
    
    repositoryList := reposResult.Value.([]forge.Repository)
    println("Found", len(repositoryList), "repositories in 'core' organization")
    
    // List PRs
    prsResult := client.ListPRs(context.Background(), "core/go", "open", 0, 50)
    if !prsResult.OK {
        panic(prsResult.Err)
    }
    
    prList := prsResult.Value.([]forge.PullRequest)
    println("Found", len(prList), "open PRs in 'core/go'")
}
```

### Example 5: Git Operations

```go
package main

import (
    "context"
    "dappco.re/go/scm/git"
)

func main() {
    // Create git service
    service := git.NewGitService(&git.GitConfig{
        Timeout: 30 * time.Second,
    })
    
    // Clone repository
    cloneResult := service.Clone(context.Background(), "https://forge.lthn.ai/core/go.git", "./core-go", git.CloneOptions{
        Branch: "dev",
        Depth:  1,
    })
    if !cloneResult.OK {
        panic(cloneResult.Err)
    }
    
    println("Repository cloned successfully")
    
    // Get status
    statusResult := service.Status(context.Background(), "./core-go")
    if !statusResult.OK {
        panic(statusResult.Err)
    }
    
    status := statusResult.Value.(git.GitStatus)
    println("Repository status:", status.String)
}
```

---

## 🔗 Related Documentation

### RFC Documents

| Document | Location | Description | Size |
|----------|----------|-------------|------|
| Main RFC | [`RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/scm/RFC.md) | Complete specification | 160KB |
| Models RFC | [`RFC.models.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/scm/RFC.models.md) | Data model definitions | 129KB |
| Commands RFC | [`RFC.commands.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/scm/RFC.commands.md) | Command specifications | 673 bytes |
| Catalog RFC | [`RFC.catalog.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/scm/RFC.catalog.md) | Tool/handler catalog | 96KB |
| Imports RFC | [`RFC.imports.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/scm/RFC.imports.md) | Import specifications | 705 bytes |
| CLAUDE.md | [`CLAUDE.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/scm/CLAUDE.md) | Claude-specific notes | 1.7KB |

### Repository Documentation

| Document | Location | Description |
|----------|----------|-------------|
| README.md | [`README.md`](file:///Users/snider/Code/core/go-scm/README.md) | Repository overview | 2.1KB |
| AGENTS.md | [`AGENTS.md`](file:///Users/snider/Code/core/go-scm/AGENTS.md) | Agent guidance | 966 bytes |
| CLAUDE.md | [`CLAUDE.md`](file:///Users/snider/Code/core/go-scm/CLAUDE.md) | Claude-specific notes | 5.1KB |
| LICENCE | [`LICENCE`](file:///Users/snider/Code/core/go-scm/LICENCE) | EUPL-1.2 license | 14KB |
| sonar-project.properties | [`sonar-project.properties`](file:///Users/snider/Code/core/go-scm/sonar-project.properties) | SonarQube configuration | 531 bytes |

### Knowledge Pack Documentation

| Document | Location | Description |
|----------|----------|-------------|
| README.md | [`README.md`](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/scm/README.md) | This document's companion | 40KB+ |
| CoreGo INDEX | [`INDEX.md`](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) | CoreGo package catalog | 240+ |

---

## 🏷️ Metadata

```yaml
name: go-scm
module: dappco.re/go/scm
repository: github.com/dappcore/go-scm
spec: file:///Users/snider/Code/meowmix/plans/code/core/go/scm/RFC.md
version: v1.0.0
license: EUPL-1.2
status: Production
maintainer: Purberus <purberus@lthn.ai>
```

---

*Documentation generated by Purberus for the CoreGo Knowledge Pack*  
*Knowledge Pack Version: CoreGo v1.2.0*  
*Commit: f566c02*
