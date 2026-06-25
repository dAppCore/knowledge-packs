# go-git — Multi-Repository Git Operations

> **Package:** `dappco.re/go/git`  
> **Repository:** [`github.com/dappcore/go-git`](https://github.com/dappcore/go-git)  
> **Spec:** [`plans/code/core/go/git/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/git/RFC.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>  
> **Status:** Production Ready  
> **Module:** `dappco.re/go/git`

---

## Overview

**go-git** is a minimal in-house Git wrapper for multi-repository operations. It provides parallel git status checks, push/pull operations across multiple repositories, and integrates with the CoreGo framework as a service. Designed for AI agent consumption following the AX standard.

### Key capabilities

| Category | Features | Description |
|----------|----------|-------------|
| **Status Checking** | Parallel execution | Check git status across multiple repos simultaneously |
| **Push Operations** | Batch + individual | Push to single or multiple repositories |
| **Pull Operations** | Batch + individual | Pull from single or multiple repositories |
| **CoreGo Service** | Full integration | Works as a CoreGo service with queries and actions |
| **Result Handling** | Comprehensive | Detailed error reporting with git stderr |
| **Path Validation** | Security-first | Validates paths, prevents directory traversal |

### Architecture

### Module structure

```
core/go-git/
├── go/                          # Go module root (dappco.re/go/git)
│   ├── git.go                    # Core git operations
│   ├── git_test.go               # Unit tests
│   ├── git_example_test.go      # Usage examples
│   ├── service.go               # CoreGo service integration
│   ├── service_test.go          # Service tests
│   ├── service_example_test.go # Service usage examples
│   │
│   ├── cmd/                     # CLI commands (if any)
│   ├── tests/                   # Integration tests
│   │   └── cli/                 # CLI test utilities
│   │
│   ├── go.mod                   # Module definition
│   └── go.sum                   # Dependency checksums
│
├── docs/                        # Documentation
├── README.md
├── AGENTS.md
├── CLAUDE.md
├── CONTRIBUTING.md
└── LICENCE
```

### Core design principles

1. **Minimal Wrapper** — Thin layer over git CLI, not a full git implementation
2. **Parallel Execution** — All multi-repo operations run in parallel
3. **CoreGo Integration** — Full service support with queries and actions
4. **Security-First** — Path validation prevents directory traversal attacks
5. **AX Standard** — Each `.go` file has `_test.go` and `_example_test.go`

---

## Packages

### Core operations (`git.go`)

#### Status Checking

- **`Status`** — Check git status for multiple repositories in parallel
- **`StatusIter`** — Iterator version for streaming results
- **`RepoStatus`** — Struct containing status for a single repository

#### Push Operations

- **`Push`** — Push changes to a single repository
- **`PushMultiple`** — Push to multiple repositories in parallel
- **`PushMultipleIter`** — Iterator version for streaming push results
- **`PushResult`** — Struct containing push result for a single repository

#### Pull Operations

- **`Pull`** — Pull changes from a single repository
- **`PullMultiple`** — Pull from multiple repositories in parallel
- **`PullMultipleIter`** — Iterator version for streaming pull results
- **`PullResult`** — Struct containing pull result for a single repository

#### Helper Functions

- **`IsNonFastForward`** — Check if error is due to non-fast-forward merge
- **Error handling** — Comprehensive error types with git stderr

### CoreGo service (`service.go`)

The `Service` provides git operations as a CoreGo service with:

#### Query Handlers

- **`QueryStatus`** — Request status for multiple paths
- **`QueryDirtyRepos`** — Get repos with uncommitted changes
- **`QueryAheadRepos`** — Get repos with unpushed commits
- **`QueryBehindRepos`** — Get repos with unpulled commits

#### Action Handlers

- **`TaskPush`** — Push to a single repository
- **`TaskPull`** — Pull from a single repository
- **`TaskPushMultiple`** — Push to multiple repositories
- **`TaskPullMultiple`** — Pull from multiple repositories

#### Service Methods

- **`Status()`** — Get last status result
- **`All()`** — Iterator over all last known statuses
- **`Dirty()`** — Iterator over repos with uncommitted changes
- **`Ahead()`** — Iterator over repos with unpushed commits
- **`Behind()`** — Iterator over repos with unpulled commits
- **`DirtyRepos()`** — Get repos with uncommitted changes
- **`AheadRepos()`** — Get repos with unpushed commits
- **`BehindRepos()`** — Get repos with unpulled commits

---

## Configuration

### Service options

```go
// ServiceOptions configures the git service
type ServiceOptions struct {
    WorkDir string  // Working directory (optional, for path validation)
}

// Create a new git service
serviceFactory := git.NewService(git.ServiceOptions{
    WorkDir: "/path/to/workdir",
})

// Register with Core
core.New(
    core.WithServiceLock(),
    core.WithService(serviceFactory),
)
```

### Path validation

The service validates all paths:
- Must be absolute paths
- Must be within WorkDir (if configured)
- Prevents directory traversal attacks
- Resolves symlinks for security

---

## Commands

While go-git is primarily a library, it can be integrated into CLI tools:

```bash
# Example CLI integration (pseudo-commands)

# Check status of all repositories
core git status /path/to/repos

# Push to all repositories
core git push /path/to/repos

# Pull from all repositories
core git pull /path/to/repos
```

---

## Usage patterns

### 1. Basic Status Checking

```go
import "dappco.re/go/git"

ctx := context.Background()

// Check status for multiple repositories
statuses := git.Status(ctx, git.StatusOptions{
    Paths: []string{
        "/home/user/Code/core/agent",
        "/home/user/Code/core/api",
    },
})

for _, status := range statuses {
    fmt.Printf("%s: branch=%s, dirty=%v, ahead=%d, behind=%d\n",
        status.Name, status.Branch, status.IsDirty(), status.Ahead, status.Behind)
}
```

### 2. Streaming Status with Iterator

```go
ctx := context.Background()

// Stream status results as they become available
for status := range git.StatusIter(ctx, git.StatusOptions{
    Paths: []string{"/path/to/repo1", "/path/to/repo2"},
    Names: map[string]string{
        "/path/to/repo1": "My Project",
        "/path/to/repo2": "Another Project",
    },
}) {
    if status.Error != nil {
        log.Printf("Error checking %s: %v", status.Name, status.Error)
        continue
    }
    fmt.Printf("%s: %d untracked, %d modified\n", status.Name, status.Untracked, status.Modified)
}
```

### 3. Push to Multiple Repositories

```go
ctx := context.Background()

// Push to multiple repositories in parallel
result := git.PushMultiple(ctx,
    []string{"/path/to/repo1", "/path/to/repo2"},
    map[string]string{
        "/path/to/repo1": "Repo 1",
        "/path/to/repo2": "Repo 2",
    },
)

if !result.OK {
    log.Fatal(result.Error)
}

// Get individual results
pushResults := result.Value.([]git.PushResult)
for _, pr := range pushResults {
    if pr.Error != nil {
        log.Printf("Failed to push %s: %v", pr.Name, pr.Error)
    } else {
        log.Printf("Successfully pushed %s", pr.Name)
    }
}
```

### 4. Pull from Multiple Repositories

```go
ctx := context.Background()

// Pull from multiple repositories
result := git.PullMultiple(ctx,
    []string{"/path/to/repo1", "/path/to/repo2"},
    nil, // Use default names (path names)
)

if !result.OK {
    // Handle error
}

pullResults := result.Value.([]git.PullResult)
for _, pr := range pullResults {
    if pr.Error != nil {
        if git.IsNonFastForward(pr.Error) {
            log.Printf("%s: non-fast-forward, manual merge needed", pr.Name)
        } else {
            log.Printf("%s: pull failed: %v", pr.Name, pr.Error)
        }
    }
}
```

### 5. Using CoreGo Service

```go
import (
    "dappco.re/go"
    "dappco.re/go/git"
)

// Create Core with git service
c := core.New(
    core.WithServiceLock(),
    core.WithService(git.NewService(git.ServiceOptions{
        WorkDir: "/home/user/Code",
    })),
)

// Query status
statuses := c.Query(git.QueryStatus{
    Paths: []string{"/home/user/Code/core/agent"},
}).Value.([]git.RepoStatus)

// Perform action
c.Action("git.push", core.Options{
    "path": "/home/user/Code/core/agent",
})

// Get service and use its methods
service := c.GetService().(*git.Service)
// Access last status
lastStatus := service.Status()

// Get repos with uncommitted changes
dirtyRepos := service.DirtyRepos()

// Stream repos with unpushed commits
for repo := range service.Ahead() {
    fmt.Printf("%s has unpushed commits\n", repo.Name)
}
```

### 6. Error Handling

```go
ctx := context.Background()

result := git.Push(ctx, "/path/to/repo")
if !result.OK {
    switch {
    case git.IsNonFastForward(result.Error):
        log.Println("Non-fast-forward: rebase or merge required")
    case errors.Is(result.Error, os.ErrNotExist):
        log.Println("Repository does not exist")
    default:
        if gitErr, ok := result.Value.(*git.GitError); ok {
            log.Printf("Git error: %v\nStderr: %s", gitErr.Err, gitErr.Stderr)
        }
    }
}
```

---

## Testing

### Test structure

Each file follows the AX standard with:
- `_test.go` — Unit tests with mocked git commands
- `_example_test.go` — Usage examples as tests

### Test coverage

| File | Functions | Lines | Coverage |
|------|-----------|-------|----------|
| `git.go` | 9 public functions | ~540 | >80% |
| `service.go` | 15+ public methods | ~480 | >80% |

### Test commands

```bash
# All tests
go test ./...

# With race detector
go test -race ./...

# Specific file
go test -v ./go/git/...

# With coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

### AX standard compliance

All packages follow AX standard:
- `git.go` → `git_test.go` + `git_example_test.go`
- `service.go` → `service_test.go` + `service_example_test.go`

---

## API reference

### Core types

```go
// RepoStatus represents the git status of a single repository
type RepoStatus struct {
    Name      string  // Repository name or path
    Path      string  // Filesystem path
    Modified  int     // Number of modified files
    Untracked int     // Number of untracked files
    Staged    int     // Number of staged files
    Ahead     int     // Commits ahead of upstream
    Behind    int     // Commits behind upstream
    Branch    string  // Current branch
    Error     error   // Error if status check failed
}

// Methods on RepoStatus
func (s *RepoStatus) IsDirty() bool      // Has uncommitted changes
func (s *RepoStatus) HasUnpushed() bool  // Has unpushed commits
func (s *RepoStatus) HasUnpulled() bool  // Has unpulled commits

// PushResult represents the result of a push operation
type PushResult struct {
    Name  string
    Path  string
    Error error
}

// PullResult represents the result of a pull operation
type PullResult struct {
    Name  string
    Path  string
    Error error
}

// GitError represents a git command error
type GitError struct {
    Args   []string  // Operation arguments
    Err    error     // Underlying error
    Stderr string    // Git stderr output
}

func (e *GitError) Error() string
```

### Status options

```go
// StatusOptions configures the status check
type StatusOptions struct {
    Paths []string            // Repository paths to check
    Names map[string]string   // Optional: map paths to display names
}

// Main status functions
func Status(ctx core.Context, opts StatusOptions) []RepoStatus
func StatusIter(ctx core.Context, opts StatusOptions) iter.Seq[RepoStatus]
```

### Push/pull functions

```go
// Single repository operations
func Push(ctx core.Context, path string) core.Result
func Pull(ctx core.Context, path string) core.Result

// Multiple repository operations
func PushMultiple(ctx core.Context, paths []string, names map[string]string) core.Result
func PushMultipleIter(ctx core.Context, paths []string, names map[string]string) iter.Seq[PushResult]
func PullMultiple(ctx core.Context, paths []string, names map[string]string) core.Result
func PullMultipleIter(ctx core.Context, paths []string, names map[string]string) iter.Seq[PullResult]

// Helper function
func IsNonFastForward(err error) bool
```

### Service types

```go
// Query types for service
type QueryStatus struct {
    Paths []string
    Names map[string]string
}
type QueryDirtyRepos struct{}
type QueryAheadRepos struct{}
type QueryBehindRepos struct{}

// Task types for service
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

// Service type
type Service struct {
    *core.ServiceRuntime[ServiceOptions]
    opts       ServiceOptions
    lastStatus []RepoStatus
}

// Service factory
func NewService(opts ServiceOptions) func(*core.Core) core.Result

// Service methods
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
| RFC | Package specification | [plans/code/core/go/git/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/git/RFC.md) |
| LLM Index | AI training data | [llms.txt](file:///Users/snider/Code/core/go-git/llms.txt) |

### External references

| Resource | URL |
|----------|-----|
| Repository | [github.com/dappcore/go-git](https://github.com/dappcore/go-git) |
| Module | [pkg.go.dev/dappco.re/go/git](https://pkg.go.dev/dappco.re/go/git) |
| Git | [git-scm.com](https://git-scm.com) |

