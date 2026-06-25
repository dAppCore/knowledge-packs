# go-git Package Index

> **Package:** `dappco.re/go/git`  
> **Repository:** [`github.com/dappcore/go-git`](https://github.com/dappcore/go-git)  
> **Spec:** [`plans/code/core/go/git/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/git/RFC.md)  
> **Documentation:** [README.md](./README.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>

---

## Table of contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Packages](#packages)
- [Configuration](#configuration)
- [Commands](#commands)
- [Usage patterns](#usage-patterns)
- [Testing](#testing)
- [API reference](#api-reference)
- [Related documentation](#related-documentation)

---

## Overview

**go-git** is a minimal in-house Git wrapper providing multi-repository operations with parallel execution. It serves as a thin abstraction over the git CLI, offering status checking, push/pull operations, and full CoreGo service integration — all designed for AI agent consumption following the AX standard.

### Key features

| Category | Features | Count |
|----------|----------|-------|
| **Status Operations** | Parallel status checks, dirty/ahead/behind detection | 4 functions |
| **Push Operations** | Single, multiple, streaming | 4 functions |
| **Pull Operations** | Single, multiple, streaming | 4 functions |
| **CoreGo Service** | Full query/action support | 8 query/task types |
| **Service Methods** | Status access, filtering | 8 methods |
| **Path Validation** | Security checks, symlink resolution | Built-in |

### Package statistics

| Metric | Value |
|--------|-------|
| **Module** | `dappco.re/go/git` |
| **Go Version** | 1.26.0 |
| **Total Files** | 10+ |
| **Go Files** | 8 |
| **Test Files** | 4 |
| **Example Files** | 4 |
| **Total Lines** | ~2,000 |
| **Test Coverage** | >80% |
| **Dependencies** | 1 (dappco.re/go) |

---

## Architecture

### Minimal wrapper design

**go-git** is intentionally minimal:
- Thin layer over git CLI (uses `/usr/bin/env git`)
- Not a full git implementation
- Focuses on multi-repository workflows
- Parallel execution for all batch operations

### AX standard compliance

100% AX compliant:
- `git.go` → `git_test.go` + `git_example_test.go`
- `service.go` → `service_test.go` + `service_example_test.go`
- Comments are agent-first
- SPOR (Single Point of Responsibility) per file

### Module structure

```
core/go-git/
├── go/
│   ├── git.go                    # Core operations (539 lines)
│   ├── git_test.go               # Unit tests
│   ├── git_example_test.go      # Usage examples
│   ├── service.go               # CoreGo service (482 lines)
│   ├── service_test.go          # Service unit tests
│   ├── service_example_test.go # Service usage examples
│   ├── go.mod                   # Module definition
│   └── go.sum                   # Dependencies
│
├── docs/                        # Documentation
├── tests/                       # Integration tests
│   └── cli/
│       └── git/
│           └── main.go          # CLI test utility
├── README.md
├── AGENTS.md
├── CLAUDE.md
├── CONTRIBUTING.md
└── LICENCE
```

### CoreGo integration

```
Service Lifecycle:
1. NewService() creates service factory
2. Register with Core container
3. OnStartup() registers query/action handlers
4. Service provides both Query and Action handlers
5. Full path validation for security
```

---

## Packages

### Core operations (`git.go`)

#### Status checking

| Function | Description | Parallel | Iterator |
|----------|-------------|----------|----------|
| `Status` | Check status for multiple repos | ✅ Yes | ❌ No |
| `StatusIter` | Stream status results | ✅ Yes | ✅ Yes |

**`RepoStatus` Struct Fields:**
- `Name` — Repository display name
- `Path` — Filesystem path
- `Modified` — Count of modified files
- `Untracked` — Count of untracked files
- `Staged` — Count of staged files
- `Ahead` — Commits ahead of upstream
- `Behind` — Commits behind upstream
- `Branch` — Current branch name
- `Error` — Error if check failed

**`RepoStatus` Methods:**
- `IsDirty()` — Has uncommitted changes
- `HasUnpushed()` — Has unpushed commits
- `HasUnpulled()` — Has unpulled commits

#### Push operations

| Function | Description | Parallel | Iterator |
|----------|-------------|----------|----------|
| `Push` | Push to single repo | ❌ No | ❌ No |
| `PushMultiple` | Push to multiple repos | ✅ Yes | ❌ No |
| `PushMultipleIter` | Stream push results | ✅ Yes | ✅ Yes |

**`PushResult` Struct:**
- `Name` — Repository display name
- `Path` — Filesystem path
- `Error` — Error if push failed

#### Pull operations

| Function | Description | Parallel | Iterator |
|----------|-------------|----------|----------|
| `Pull` | Pull from single repo | ❌ No | ❌ No |
| `PullMultiple` | Pull from multiple repos | ✅ Yes | ❌ No |
| `PullMultipleIter` | Stream pull results | ✅ Yes | ✅ Yes |

**`PullResult` Struct:**
- `Name` — Repository display name
- `Path` — Filesystem path
- `Error` — Error if pull failed

#### Helper functions

| Function | Description |
|----------|-------------|
| `IsNonFastForward` | Check if error is non-fast-forward merge |

#### Error types

| Type | Description |
|------|-------------|
| `GitError` | Git command error with stderr |

### CoreGo service (`service.go`)

#### Query types

| Query | Description | Handler |
|-------|-------------|---------|
| `QueryStatus` | Request status for paths | `handleQuery` |
| `QueryDirtyRepos` | Get repos with uncommitted changes | `handleQuery` |
| `QueryAheadRepos` | Get repos with unpushed commits | `handleQuery` |
| `QueryBehindRepos` | Get repos with unpulled commits | `handleQuery` |

#### Task types

| Task | Description | Handler |
|------|-------------|---------|
| `TaskPush` | Push to single repository | `handleTaskMessage` |
| `TaskPull` | Pull from single repository | `handleTaskMessage` |
| `TaskPushMultiple` | Push to multiple repositories | `handleTaskMessage` |
| `TaskPullMultiple` | Pull from multiple repositories | `handleTaskMessage` |

#### Service methods

| Method | Returns | Description |
|--------|---------|-------------|
| `Status()` | `[]RepoStatus` | Get last status result |
| `All()` | `iter.Seq[RepoStatus]` | Iterator over all statuses |
| `Dirty()` | `iter.Seq[RepoStatus]` | Iterator over dirty repos |
| `Ahead()` | `iter.Seq[RepoStatus]` | Iterator over repos with unpushed |
| `Behind()` | `iter.Seq[RepoStatus]` | Iterator over repos with unpulled |
| `DirtyRepos()` | `[]RepoStatus` | Get dirty repos as slice |
| `AheadRepos()` | `[]RepoStatus` | Get repos with unpushed as slice |
| `BehindRepos()` | `[]RepoStatus` | Get repos with unpulled as slice |

---

## Configuration

### Service options

```go
type ServiceOptions struct {
    WorkDir string  // Optional: working directory for path validation
}

// Create service factory
factory := git.NewService(git.ServiceOptions{
    WorkDir: "/home/user/Code",
})

// Register with Core
c := core.New(
    core.WithServiceLock(),
    core.WithService(factory),
)
```

### Path validation rules

1. **Absolute Paths Required** — All paths must be absolute
2. **WorkDir Constraint** — If WorkDir is set, paths must be within it
3. **Symlink Resolution** — All paths are resolved through symlinks
4. **Directory Traversal Prevention** — Paths cannot escape WorkDir

```go
// Validation examples:
✅ "/home/user/Code/repo"          // Valid absolute path
❌ "~/Code/repo"                   // Not absolute
❌ "/tmp/../etc/passwd"            // Directory traversal blocked
❌ "/home/user/Code/../../../etc"  // Escape WorkDir blocked
```

---

## Commands

### CLI integration

```bash
# Pseudo-commands for CLI integration

# Status operations
core git status /path/to/repo1 /path/to/repo2
core git status --all /home/user/Code

# Push operations
core git push /path/to/repo
core git push --all /home/user/Code

# Pull operations
core git pull /path/to/repo
core git pull --all /home/user/Code

# Query operations
core git dirty-repos      # List repos with uncommitted changes
core git ahead-repos      # List repos with unpushed commits
core git behind-repos      # List repos with unpulled commits
```

---

## Usage patterns

### 1. Status Checking

```go
// Check status for specific repositories
statuses := git.Status(ctx, git.StatusOptions{
    Paths: []string{
        "/home/user/Code/core/agent",
        "/home/user/Code/core/api",
    },
})

// Iterate over results
for _, status := range statuses {
    if status.Error != nil {
        log.Printf("Error: %v", status.Error)
        continue
    }
    fmt.Printf("%s (%s): modified=%d, untracked=%d\n",
        status.Name, status.Branch, status.Modified, status.Untracked)
}
```

### 2. Streaming Status

```go
// Stream results as they become available
for status := range git.StatusIter(ctx, git.StatusOptions{
    Paths: []string{"/repo1", "/repo2"},
    Names: map[string]string{"/repo1": "Project 1", "/repo2": "Project 2"},
}) {
    if status.HasUnpushed() {
        fmt.Printf("%s has unpushed commits\n", status.Name)
    }
}
```

### 3. Batch Push with Error Handling

```go
result := git.PushMultiple(ctx,
    []string{"/repo1", "/repo2"},
    map[string]string{"/repo1": "Repo 1", "/repo2": "Repo 2"},
)

if !result.OK {
    log.Fatal(result.Error)
}

for _, pr := range result.Value.([]git.PushResult) {
    if pr.Error != nil {
        log.Printf("Push failed for %s: %v", pr.Name, pr.Error)
    } else {
        log.Printf("Push succeeded for %s", pr.Name)
    }
}
```

### 4. CoreGo Service Integration

```go
// Create Core with git service
c := core.New(
    core.WithServiceLock(),
    core.WithService(git.NewService(git.ServiceOptions{
        WorkDir: "/home/user/Code",
    })),
)

// Query status via Core
result := c.Query(git.QueryStatus{
    Paths: []string{"/home/user/Code/repo"},
})
statuses := result.Value.([]git.RepoStatus)

// Get service and access state
service := c.GetService().(*git.Service)
lastStatus := service.Status()

// Stream repos needing attention
for repo := range service.Dirty() {
    fmt.Printf("%s has uncommitted changes\n", repo.Name)
}
```

### 5. Filtering Repositories

```go
// Get repos with unpushed commits
service := c.GetService().(*git.Service)
aheadRepos := service.AheadRepos()

// Get repos with unpulled commits
behindRepos := service.BehindRepos()

// Get repos with uncommitted changes
dirtyRepos := service.DirtyRepos()

// Or use iterators
for repo := range service.Ahead() {
    fmt.Printf("Push needed: %s\n", repo.Name)
}
```

---

## Testing

### Test structure

Each Go file has corresponding test files:

```
git.go                    # Implementation
git_test.go               # Unit tests (20+ tests)
git_example_test.go      # Usage examples (8+ examples)

service.go                # Service implementation
service_test.go          # Service unit tests
service_example_test.go # Service examples
```

### Test coverage by function

| Function | Lines | Coverage |
|----------|-------|----------|
| `Status` | ~50 | >80% |
| `StatusIter` | ~30 | >80% |
| `getStatus` | ~60 | >80% |
| `getAheadBehind` | ~40 | >80% |
| `Push` | ~15 | >80% |
| `Pull` | ~15 | >80% |
| `PushMultiple` | ~20 | >80% |
| `PullMultiple` | ~20 | >80% |
| `IsNonFastForward` | ~5 | >80% |

### Service test coverage

| Method | Coverage |
|--------|----------|
| `NewService` | >80% |
| `OnStartup` | >80% |
| `handleQuery` | >80% |
| `handleTask` | >80% |
| `handleTaskMessage` | >80% |
| `validatePath` | >80% |
| `validatePaths` | >80% |
| All accessor methods | >80% |

### Test commands

```bash
# Run all tests
go test ./...

# Run with race detector
go test -race ./...

# Run specific package
go test -v ./go/...

# Run with coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# Run specific test
go test -v -run TestStatus ./...
```

---

## API reference

### Core types

```go
// RepoStatus - git status for a single repository
type RepoStatus struct {
    Name      string
    Path      string
    Modified  int
    Untracked int
    Staged    int
    Ahead     int
    Behind    int
    Branch    string
    Error     error
}

// Methods
func (s *RepoStatus) IsDirty() bool
func (s *RepoStatus) HasUnpushed() bool
func (s *RepoStatus) HasUnpulled() bool

// PushResult - result of a push operation
type PushResult struct {
    Name  string
    Path  string
    Error error
}

// PullResult - result of a pull operation
type PullResult struct {
    Name  string
    Path  string
    Error error
}

// GitError - git command error
type GitError struct {
    Args   []string
    Err    error
    Stderr string
}
func (e *GitError) Error() string

// StatusOptions - configuration for status operations
type StatusOptions struct {
    Paths []string
    Names map[string]string
}
```

### Core functions

```go
// Status operations
func Status(ctx core.Context, opts StatusOptions) []RepoStatus
func StatusIter(ctx core.Context, opts StatusOptions) iter.Seq[RepoStatus]

// Push operations
func Push(ctx core.Context, path string) core.Result
func PushMultiple(ctx core.Context, paths []string, names map[string]string) core.Result
func PushMultipleIter(ctx core.Context, paths []string, names map[string]string) iter.Seq[PushResult]

// Pull operations
func Pull(ctx core.Context, path string) core.Result
func PullMultiple(ctx core.Context, paths []string, names map[string]string) core.Result
func PullMultipleIter(ctx core.Context, paths []string, names map[string]string) iter.Seq[PullResult]

// Helper
func IsNonFastForward(err error) bool
```

### Service types

```go
// ServiceOptions
type ServiceOptions struct {
    WorkDir string
}

// Query types
type QueryStatus struct {
    Paths []string
    Names map[string]string
}
type QueryDirtyRepos struct{}
type QueryAheadRepos struct{}
type QueryBehindRepos struct{}

// Task types
type TaskPush struct {
    Path string
    Name string
}
type TaskPull struct {
    Path string
    Name string
}
type TaskPushMultiple struct {
    Paths []string
    Names map[string]string
}
type TaskPullMultiple struct {
    Paths []string
    Names map[string]string
}

// Service
type Service struct {
    *core.ServiceRuntime[ServiceOptions]
    opts       ServiceOptions
    lastStatus []RepoStatus
}

// Factory
func NewService(opts ServiceOptions) func(*core.Core) core.Result

// Methods
func (s *Service) Status() []RepoStatus
func (s *Service) All() iter.Seq[RepoStatus]
func (s *Service) Dirty() iter.Seq[RepoStatus]
func (s *Service) Ahead() iter.Seq[RepoStatus]
func (s *Service) Behind() iter.Seq[RepoStatus]
func (s *Service) DirtyRepos() []RepoStatus
func (s *Service) AheadRepos() []RepoStatus
func (s *Service) BehindRepos() []RepoStatus
```

---

## Related documentation

### Internal documentation

| Resource | Description | Location |
|----------|-------------|----------|
| RFC | Package specification and design | [plans/code/core/go/git/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/git/RFC.md) |
| LLM Index | AI agent training data | [llms.txt](file:///Users/snider/Code/core/go-git/llms.txt) |

### External references

| Resource | URL |
|----------|-----|
| Repository | [github.com/dappcore/go-git](https://github.com/dappcore/go-git) |
| Module | [pkg.go.dev/dappco.re/go/git](https://pkg.go.dev/dappco.re/go/git) |
| Git Documentation | [git-scm.com/doc](https://git-scm.com/doc) |

