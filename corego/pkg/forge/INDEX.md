# go-forge Package Index

> **Package:** `dappco.re/go/forge`  
> **Repository:** [`github.com/dappcore/go-forge`](https://github.com/dappcore/go-forge)  
> **Spec:** [`plans/code/core/go/forge/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/go/forge/RFC.md)  
> **Documentation:** [README.md](./README.md)  
> **Maintainer:** Purberus <purberus@lthn.ai>

---

## Table of contents

- [Overview](#overview)
- [Architecture](#architecture)
- [API endpoints](#api-endpoints)
- [Configuration](#configuration)
- [Commands](#commands)
- [Usage patterns](#usage-patterns)
- [Testing](#testing)
- [API reference](#api-reference)
- [Related documentation](#related-documentation)

---

## Overview

**go-forge** is a zero-dependency Go client for the Forgejo API, providing type-safe access to ~450 endpoints across all major Forgejo/Gitea features. Designed for AI agent consumption following the AX standard, it covers repository management, issue tracking, pull requests, project management, organisation management, and more.

### Key features

| Category | Endpoints | Status |
|----------|-----------|--------|
| **Repository Management** | ~50 | ✅ Complete |
| **Issue Tracking** | ~40 | ✅ Complete |
| **Pull Requests** | ~35 | ✅ Complete |
| **Project Management** | ~25 | ✅ Complete |
| **Organization Management** | ~30 | ✅ Complete |
| **User Management** | ~20 | ✅ Complete |
| **Webhooks** | ~15 | ✅ Complete |
| **Releases** | ~10 | ✅ Complete |
| **Activity & Notifications** | ~20 | ✅ Complete |
| **Admin** | ~15 | ✅ Complete |
| **Actions (CI/CD)** | ~20 | ✅ Complete |
| **Miscellaneous** | ~10 | ✅ Complete |

**Total: ~280 primary endpoints** (README mentions ~450 including variations)

### Package statistics

| Metric | Value |
|--------|-------|
| **Module** | `dappco.re/go/forge` |
| **Go Version** | 1.26.0 |
| **Total Files** | 140+ |
| **Go Files** | 90+ |
| **Type Definitions** | 50+ files |
| **Test Files** | 40+ |
| **Example Files** | 30+ |
| **Total Lines** | ~35,000 |
| **Test Coverage** | >85% average |
| **Dependencies** | 2 (go, go-json) |

---

## Architecture

### Zero-dependency design

**go-forge** maintains minimal dependencies:

```
Module Dependencies:
├── dappco.re/go v0.10.4          # CoreGo framework primitives
└── github.com/goccy/go-json v0.10.6  # High-performance JSON parser

No other external dependencies — pure standard library + CoreGo
```

### AX standard compliance

100% AX compliant:
- Every `.go` file has corresponding `_test.go`
- Every `.go` file has corresponding `_example_test.go`
- Comments are agent-first, not human-first
- SPOR (Single Point of Responsibility) per file
- Type-safe with comprehensive type definitions

### Module structure

```
core/go-forge/
├── go/
│   ├── actions.go               # CI/CD Actions endpoints
│   ├── actions_test.go          # Unit tests (100% coverage)
│   ├── actions_example_test.go # Usage examples
│   ├── activitypub.go           # ActivityPub/Fediverse endpoints
│   ├── admin.go                 # Admin API endpoints
│   ├── branches.go              # Branch management
│   ├── client.go                # HTTP client implementation
│   ├── client_example_test.go  # Client initialization examples
│   ├── commits.go               # Commit access
│   ├── config.go                # Client configuration
│   ├── contents.go              # Repository file operations
│   ├── forge.go                 # Main client factory
│   ├── helpers.go               # Helper utilities
│   ├── issues.go                # Issue tracking
│   ├── labels.go                # Label management
│   ├── milestones.go            # Milestone tracking
│   ├── misc.go                  # Miscellaneous endpoints (projects, etc.)
│   ├── notifications.go         # User notifications
│   ├── orgs.go                  # Organization management
│   ├── packages.go              # Package registry
│   ├── params.go                # Request parameter handling
│   ├── pulls.go                 # Pull request management
│   ├── releases.go              # Release management
│   ├── repos.go                 # Repository management
│   ├── resource.go              # Shared resource types
│   ├── service.go               # CoreGo service integration
│   ├── users.go                 # User management
│   ├── webhooks.go              # Webhook management
│   ├── wiki.go                  # Wiki operations
│   │
│   ├── cmd/                     # CLI command definitions
│   │   └── ...
│   │
│   ├── testdata/                # Test fixtures and mocks
│   ├── tests/                   # Integration tests
│   │
│   └── types/                   # Type definitions (50+ files)
│       ├── actions.go          # Actions types
│       ├── activitypub.go      # ActivityPub types
│       ├── admin.go            # Admin types
│       ├── attachment.go       # Attachment types
│       ├── branch.go           # Branch types
│       ├── branch_protection.go # Branch protection types
│       ├── commit.go           # Commit types
│       ├── commit_status.go    # Commit status types
│       ├── content.go          # Content types
│       ├── convert.go          # Type conversion utilities
│       ├── deploy_key.go       # Deploy key types
│       ├── deploy_key_non_unix.go # Platform-specific implementations
│       ├── discussion.go       # Discussion types
│       ├── edit.go             # Edit types
│       ├── external_tracker.go # External tracker types
│       ├── external_wiki.go    # External wiki types
│       ├── file_commits.go     # File commit types
│       ├── file_response.go    # File response types
│       ├── git_hook.go          # Git hook types
│       ├── git_service.go      # Git service types
│       ├── gitignores.go       # Gitignore types
│       ├── hook.go             # Hook types
│       ├── identity.go         # Identity types
│       ├── internal.go         # Internal types
│       ├── issue.go            # Issue types
│       ├── issue_deps.go       # Issue dependency types
│       ├── issue_indexer.go    # Issue indexer types
│       ├── label.go            # Label types
│       ├── license.go          # License types
│       ├── login_source.go     # Login source types
│       ├── milestone.go        # Milestone types
│       ├── note.go             # Note types
│       ├── notification.go     # Notification types
│       ├── notification_subject.go # Notification subject types
│       ├── organization.go     # Organization types
│       ├── pr.go               # Pull request types
│       ├── pr_review.go        # PR review types
│       ├── pr_review_comment.go # PR review comment types
│       ├── public_key.go       # Public key types
│       ├── reaction.go         # Reaction types
│       ├── reaction_group.go   # Reaction group types
│       ├── refs.go             # Reference types
│       ├── repository.go       # Repository types
│       ├── search.go           # Search types
│       ├── secret.go           # Secret types
│       ├── server.go           # Server types
│       ├── stopwatch.go        # Stopwatch types
│       ├── team.go             # Team types
│       ├── time_utils.go       # Time utility types
│       ├── timeline.go         # Timeline types
│       ├── user.go             # User types
│       ├── user_settings.go    # User settings types
│       └── variable.go         # Variable types
│
├── go.mod
├── go.sum
├── docs/
├── external/
├── README.md
├── AGENTS.md
├── CLAUDE.md
└── LICENCE
```

---

## API endpoints

### Repository management (`repos.go`, `contents.go`, `branches.go`, `commits.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListUserRepos` | List user's repositories | GET /user/repos |
| `ListOrgsRepos` | List organization repositories | GET /orgs/{org}/repos |
| `GetRepo` | Get repository details | GET /repos/{owner}/{repo} |
| `CreateRepo` | Create a new repository | POST /user/repos |
| `UpdateRepo` | Update repository settings | PATCH /repos/{owner}/{repo} |
| `DeleteRepo` | Delete a repository | DELETE /repos/{owner}/{repo} |
| `ForkRepo` | Fork a repository | POST /repos/{owner}/{repo}/forks |
| `ListForks` | List repository forks | GET /repos/{owner}/{repo}/forks |
| `ListCollaborators` | List repository collaborators | GET /repos/{owner}/{repo}/collaborators |
| `AddCollaborator` | Add collaborator | PUT /repos/{owner}/{repo}/collaborators/{username} |
| `RemoveCollaborator` | Remove collaborator | DELETE /repos/{owner}/{repo}/collaborators/{username} |

### Repository contents (`contents.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListContents` | List directory contents | GET /repos/{owner}/{repo}/contents/{path} |
| `GetContents` | Get file contents | GET /repos/{owner}/{repo}/contents/{path} |
| `CreateFile` | Create a file | PUT /repos/{owner}/{repo}/contents/{path} |
| `UpdateFile` | Update a file | PUT /repos/{owner}/{repo}/contents/{path} |
| `DeleteFile` | Delete a file | DELETE /repos/{owner}/{repo}/contents/{path} |

### Branch management (`branches.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListBranches` | List branches | GET /repos/{owner}/{repo}/branches |
| `GetBranch` | Get branch details | GET /repos/{owner}/{repo}/branches/{branch} |
| `CreateBranch` | Create a new branch | POST /repos/{owner}/{repo}/branches |
| `DeleteBranch` | Delete a branch | DELETE /repos/{owner}/{repo}/branches/{branch} |

### Commit access (`commits.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListCommits` | List commits | GET /repos/{owner}/{repo}/commits |
| `GetCommit` | Get commit details | GET /repos/{owner}/{repo}/commits/{sha} |
| `GetCommitDiff` | Get commit diff | GET /repos/{owner}/{repo}/commits/{sha}.diff |
| `GetCommitPatch` | Get commit patch | GET /repos/{owner}/{repo}/commits/{sha}.patch |

### Issue tracking (`issues.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListIssues` | List repository issues | GET /repos/{owner}/{repo}/issues |
| `GetIssue` | Get issue details | GET /repos/{owner}/{repo}/issues/{index} |
| `CreateIssue` | Create an issue | POST /repos/{owner}/{repo}/issues |
| `UpdateIssue` | Update an issue | PATCH /repos/{owner}/{repo}/issues/{index} |
| `DeleteIssue` | Delete an issue | DELETE /repos/{owner}/{repo}/issues/{index} |
| `SearchIssues` | Search issues | GET /repos/{owner}/{repo}/issues |
| `ListIssueComments` | List issue comments | GET /repos/{owner}/{repo}/issues/{index}/comments |
| `CreateIssueComment` | Create issue comment | POST /repos/{owner}/{repo}/issues/{index}/comments |
| `UpdateIssueComment` | Update issue comment | PATCH /repos/{owner}/{repo}/issues/{index}/comments/{id} |
| `DeleteIssueComment` | Delete issue comment | DELETE /repos/{owner}/{repo}/issues/{index}/comments/{id} |

### Label management (`labels.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListRepoLabels` | List repository labels | GET /repos/{owner}/{repo}/labels |
| `GetRepoLabel` | Get label details | GET /repos/{owner}/{repo}/labels/{id} |
| `CreateRepoLabel` | Create repository label | POST /repos/{owner}/{repo}/labels |
| `UpdateRepoLabel` | Update repository label | PATCH /repos/{owner}/{repo}/labels/{id} |
| `DeleteRepoLabel` | Delete repository label | DELETE /repos/{owner}/{repo}/labels/{id} |

### Milestone tracking (`milestones.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListMilestones` | List repository milestones | GET /repos/{owner}/{repo}/milestones |
| `GetMilestone` | Get milestone details | GET /repos/{owner}/{repo}/milestones/{id} |
| `CreateMilestone` | Create milestone | POST /repos/{owner}/{repo}/milestones |
| `UpdateMilestone` | Update milestone | PATCH /repos/{owner}/{repo}/milestones/{id} |
| `DeleteMilestone` | Delete milestone | DELETE /repos/{owner}/{repo}/milestones/{id} |

### Pull request management (`pulls.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListPullRequests` | List pull requests | GET /repos/{owner}/{repo}/pulls |
| `GetPullRequest` | Get PR details | GET /repos/{owner}/{repo}/pulls/{index} |
| `CreatePullRequest` | Create a PR | POST /repos/{owner}/{repo}/pulls |
| `UpdatePullRequest` | Update a PR | PATCH /repos/{owner}/{repo}/pulls/{index} |
| `MergePullRequest` | Merge a PR | POST /repos/{owner}/{repo}/pulls/{index}/merge |
| `DeletePullRequest` | Delete a PR | DELETE /repos/{owner}/{repo}/pulls/{index} |
| `ListPRComments` | List PR comments | GET /repos/{owner}/{repo}/pulls/{index}/comments |
| `CreatePRComment` | Create PR comment | POST /repos/{owner}/{repo}/pulls/{index}/comments |
| `ListPRCommits` | List PR commits | GET /repos/{owner}/{repo}/pulls/{index}/commits |
| `ApprovePullRequest` | Approve a PR | POST /repos/{owner}/{repo}/pulls/{index}/reviews |
| `RequestChanges` | Request changes | POST /repos/{owner}/{repo}/pulls/{index}/reviews |

### Project management (`misc.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListProjects` | List user projects | GET /user/projects |
| `GetProject` | Get project details | GET /projects/{id} |
| `CreateProject` | Create a project | POST /user/projects |
| `UpdateProject` | Update project | PATCH /projects/{id} |
| `DeleteProject` | Delete project | DELETE /projects/{id} |
| `ListColumns` | List project columns | GET /projects/{id}/columns |
| `CreateColumn` | Create project column | POST /projects/{id}/columns |
| `ListCards` | List project cards | GET /projects/{id}/columns/{id}/cards |
| `CreateCard` | Create project card | POST /projects/{id}/columns/{id}/cards |

### Organisation management (`orgs.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListUserOrgs` | List user's organizations | GET /user/orgs |
| `ListAllOrgs` | List all organizations | GET /orgs |
| `GetOrg` | Get organization details | GET /orgs/{org} |
| `CreateOrg` | Create an organization | POST /orgs |
| `UpdateOrg` | Update organization | PATCH /orgs/{org} |
| `DeleteOrg` | Delete organization | DELETE /orgs/{org} |
| `ListOrgMembers` | List organization members | GET /orgs/{org}/members |
| `AddOrgMember` | Add organization member | PUT /orgs/{org}/members/{username} |
| `RemoveOrgMember` | Remove organization member | DELETE /orgs/{org}/members/{username} |

### Team management (`orgs.go` — Teams section)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListTeams` | List organization teams | GET /orgs/{org}/teams |
| `GetTeam` | Get team details | GET /orgs/{org}/teams/{team} |
| `CreateTeam` | Create a team | POST /orgs/{org}/teams |
| `UpdateTeam` | Update a team | PATCH /orgs/{org}/teams/{team} |
| `DeleteTeam` | Delete a team | DELETE /orgs/{org}/teams/{team} |
| `ListTeamMembers` | List team members | GET /orgs/{org}/teams/{team}/members |
| `AddTeamMember` | Add team member | PUT /orgs/{org}/teams/{team}/members/{username} |

### User management (`users.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `GetCurrentUser` | Get authenticated user | GET /user |
| `GetUser` | Get user details | GET /users/{username} |
| `UpdateUser` | Update user settings | PATCH /user |
| `ListUserKeys` | List user's public keys | GET /user/keys |
| `GetUserKey` | Get user's public key | GET /user/keys/{id} |
| `CreateUserKey` | Create user public key | POST /user/keys |
| `DeleteUserKey` | Delete user public key | DELETE /user/keys/{id} |
| `ListFollowers` | List user's followers | GET /users/{username}/followers |
| `ListFollowing` | List users user is following | GET /users/{username}/following |
| `FollowUser` | Follow a user | PUT /users/{username}/following/{target} |
| `UnfollowUser` | Unfollow a user | DELETE /users/{username}/following/{target} |

### Webhook management (`webhooks.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListRepoWebhooks` | List repository webhooks | GET /repos/{owner}/{repo}/hooks |
| `GetRepoWebhook` | Get repository webhook | GET /repos/{owner}/{repo}/hooks/{id} |
| `CreateRepoWebhook` | Create repository webhook | POST /repos/{owner}/{repo}/hooks |
| `UpdateRepoWebhook` | Update repository webhook | PATCH /repos/{owner}/{repo}/hooks/{id} |
| `DeleteRepoWebhook` | Delete repository webhook | DELETE /repos/{owner}/{repo}/hooks/{id} |
| `ListOrgWebhooks` | List organization webhooks | GET /orgs/{org}/hooks |
| `CreateOrgWebhook` | Create organization webhook | POST /orgs/{org}/hooks |

### Release management (`releases.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListReleases` | List repository releases | GET /repos/{owner}/{repo}/releases |
| `GetRelease` | Get release details | GET /repos/{owner}/{repo}/releases/{id} |
| `CreateRelease` | Create a release | POST /repos/{owner}/{repo}/releases |
| `UpdateRelease` | Update a release | PATCH /repos/{owner}/{repo}/releases/{id} |
| `DeleteRelease` | Delete a release | DELETE /repos/{owner}/{repo}/releases/{id} |
| `UploadAsset` | Upload release asset | POST /repos/{owner}/{repo}/releases/{id}/assets |
| `DeleteAsset` | Delete release asset | DELETE /repos/{owner}/{repo}/releases/{id}/assets/{id} |

### Notifications (`notifications.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListNotifications` | List user notifications | GET /notifications |
| `GetNotification` | Get notification details | GET /notifications/threads/{id} |
| `MarkNotificationsRead` | Mark notifications as read | PUT /notifications |
| `DeleteNotification` | Delete a notification | DELETE /notifications/threads/{id} |

### Activity and feeds (`activitypub.go`, `misc.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `GetUserFeed` | Get user's feed | GET /feed |
| `GetRepoFeed` | Get repository feed | GET /repos/{owner}/{repo}/feed |
| `GetTimeline` | Get user timeline | GET /timeline |
| `GetActivityPub` | Get ActivityPub info | GET /user/activitypub |

### Admin API (`admin.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListUsers` | List all users (admin) | GET /admin/users |
| `GetUser` | Get user (admin) | GET /admin/users/{username} |
| `CreateUser` | Create user (admin) | POST /admin/users |
| `UpdateUser` | Update user (admin) | PATCH /admin/users/{username} |
| `DeleteUser` | Delete user (admin) | DELETE /admin/users/{username} |
| `ListOrgs` | List all organizations (admin) | GET /admin/orgs |
| `GetCronTasks` | Get cron tasks | GET /admin/cron |
| `RunCronTask` | Run cron task | POST /admin/cron/{task} |
| `GetSettings` | Get settings | GET /admin/settings |
| `UpdateSettings` | Update settings | PUT /admin/settings |

### Actions / CI/CD (`actions.go`)

| Function | Description | HTTP Method |
|----------|-------------|-------------|
| `ListWorkflowRuns` | List workflow runs | GET /repos/{owner}/{repo}/actions/runs |
| `GetWorkflowRun` | Get workflow run | GET /repos/{owner}/{repo}/actions/runs/{id} |
| `CancelWorkflowRun` | Cancel workflow run | POST /repos/{owner}/{repo}/actions/runs/{id}/cancel |
| `ReRunWorkflowRun` | Re-run workflow run | POST /repos/{owner}/{repo}/actions/runs/{id}/rerun |
| `ListArtifacts` | List workflow artifacts | GET /repos/{owner}/{repo}/actions/artifacts |
| `DownloadArtifact` | Download artifact | GET /repos/{owner}/{repo}/actions/artifacts/{id}/download |
| `ListSecrets` | List repository secrets | GET /repos/{owner}/{repo}/actions/secrets |
| `CreateSecret` | Create repository secret | POST /repos/{owner}/{repo}/actions/secrets |
| `UpdateSecret` | Update repository secret | PUT /repos/{owner}/{repo}/actions/secrets/{name} |
| `DeleteSecret` | Delete repository secret | DELETE /repos/{owner}/{repo}/actions/secrets/{name} |
| `ListVariables` | List repository variables | GET /repos/{owner}/{repo}/actions/variables |

---

## Configuration

### Client options

```go
type ClientOption func(*Client)

// Available options:
func WithToken(token string) ClientOption
func WithHTTPClient(client *http.Client) ClientOption
func WithRateLimit(requests int, duration time.Duration) ClientOption
func WithUserAgent(userAgent string) ClientOption
func WithTimeout(timeout time.Duration) ClientOption
func WithDebug(debug bool) ClientOption
```

### Example configurations

```go
// Basic client with token
client := forge.NewClient("https://codeberg.org", forge.WithToken(token))

// Client with custom HTTP client and timeout
client := forge.NewClient(
    "https://forgejo.example.com",
    forge.WithToken(token),
    forge.WithHTTPClient(customHTTPClient),
    forge.WithTimeout(60 * time.Second),
)

// Client with rate limiting
client := forge.NewClient(
    "https://codeberg.org",
    forge.WithToken(token),
    forge.WithRateLimit(100, time.Minute),
)
```

---

## Commands

While go-forge is primarily a library, it can be integrated into CLI tools:

```bash
# Example CLI integration (pseudo-commands)

# Repository operations
core forge repo list
core forge repo get owner/repo
core forge repo create my-repo

# Issue operations
core forge issue list owner/repo
core forge issue create owner/repo --title "Bug" --body "Description"

# Pull request operations
core forge pr list owner/repo
core forge pr create owner/repo --title "Feature" --base main --head feature-branch

# Organization operations
core forge org list
core forge org get my-org
```

---

## Usage patterns

### 1. Repository Lifecycle

```go
// Create a new repository
repo, err := client.CreateRepo(ctx, forge.CreateRepoOptions{
    Name:          "my-project",
    Description:   "A new project",
    Private:       true,
    AutoInit:      true,
    DefaultBranch: "main",
    License:       "MIT",
    Readme:        "Default",
})

// Get repository details
repo, err := client.GetRepo(ctx, "owner", "my-project")

// Update repository settings
err := client.UpdateRepo(ctx, "owner", "my-project", forge.UpdateRepoOptions{
    Description: "Updated description",
    Private:     false,
})

// Delete repository
err := client.DeleteRepo(ctx, "owner", "my-project")
```

### 2. Issue Workflow

```go
// Create an issue
issue, err := client.CreateIssue(ctx, "owner", "repo", forge.CreateIssueOptions{
    Title:    "Bug: Something is broken",
    Body:     "Description of the bug",
    Assignee:  "username",
    Labels:    []string{"bug", "priority"},
    Milestone: 1,
})

// Add a comment
comment, err := client.CreateIssueComment(ctx, "owner", "repo", issue.Number, forge.CreateCommentOptions{
    Body: "I'm working on this",
})

// Update issue state
err := client.UpdateIssue(ctx, "owner", "repo", issue.Number, forge.UpdateIssueOptions{
    State: "closed",
})
```

### 3. Pull Request Review

```go
// Create a pull request
pr, err := client.CreatePullRequest(ctx, "owner", "repo", forge.CreatePullRequestOptions{
    Title:       "Feature: Add new API",
    Body:        "Description of changes",
    Base:        "main",
    Head:        "feature/new-api",
    Draft:       false,
})

// Approve the PR
err := client.ApprovePullRequest(ctx, "owner", "repo", pr.Number, forge.ReviewOptions{
    Body:    "LGTM",
    Event:   "APPROVE",
})

// Merge the PR
err := client.MergePullRequest(ctx, "owner", "repo", pr.Number, forge.MergeOptions{
    Style:       "merge",
    CommitTitle: "Feature: Add new API",
})
```

### 4. Batch Operations

```go
// List all user repositories
repos, err := client.ListUserRepos(ctx, "username", forge.ListOptions{
    Page:    1,
    PerPage: 100,
})

// List all open issues across all repos
var allIssues []*forge.Issue
for _, repo := range repos {
    issues, err := client.ListIssues(ctx, repo.Owner.Login, repo.Name, forge.ListIssuesOptions{
        State: "open",
    })
    allIssues = append(allIssues, issues...)
}
```

### 5. Error Handling

```go
import "dappco.re/go"

repo, err := client.GetRepo(ctx, "owner", "repo")
if err != nil {
    if errors.Is(err, forge.ErrNotFound) {
        log.Printf("Repository not found")
    } else if errors.Is(err, forge.ErrUnauthorized) {
        log.Printf("Authentication failed")
    } else if errors.Is(err, forge.ErrRateLimit) {
        log.Printf("Rate limit exceeded")
    } else {
        log.Printf("Error: %v", err)
    }
}
```

---

## Testing

### Test structure

Each API file follows the AX standard with three files:

```
actions.go              # Implementation
actions_test.go         # Unit tests (100% coverage target)
actions_example_test.go # Usage examples as tests
```

### Test coverage by file

| File | Functions | Lines | Coverage |
|------|-----------|-------|----------|
| actions.go | 40+ | ~800 | >90% |
| activitypub.go | 20+ | ~500 | >85% |
| admin.go | 30+ | ~1,000 | >90% |
| branches.go | 15+ | ~400 | >85% |
| client.go | 10+ | ~300 | >80% |
| commits.go | 25+ | ~700 | >85% |
| contents.go | 20+ | ~600 | >85% |
| forge.go | 10+ | ~400 | >80% |
| helpers.go | 10+ | ~200 | >80% |
| issues.go | 30+ | ~1,200 | >90% |
| labels.go | 15+ | ~500 | >85% |
| milestones.go | 15+ | ~400 | >85% |
| misc.go | 20+ | ~600 | >85% |
| notifications.go | 15+ | ~400 | >80% |
| orgs.go | 25+ | ~900 | >85% |
| packages.go | 10+ | ~300 | >80% |
| params.go | 5+ | ~100 | >75% |
| pulls.go | 30+ | ~1,500 | >90% |
| releases.go | 15+ | ~500 | >85% |
| repos.go | 25+ | ~1,000 | >90% |
| resource.go | 10+ | ~200 | >80% |
| service.go | 5+ | ~100 | >70% |
| users.go | 20+ | ~700 | >85% |
| webhooks.go | 15+ | ~500 | >85% |
| wiki.go | 10+ | ~300 | >80% |

### Test commands

```bash
# Run all tests
go test ./...

# Run with race detector
go test -race ./...

# Run specific test
go test -v -run TestCreateRepo ./...

# Run with coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# Run with verbose output
go test -v ./go/...
```

---

## API reference

### Error types

```go
// Sentinel errors
var (
    ErrNotFound      = errors.New("not found")
    ErrUnauthorized   = errors.New("unauthorized")
    ErrForbidden      = errors.New("forbidden")
    ErrRateLimit     = errors.New("rate limit exceeded")
    ErrValidation     = errors.New("validation error")
    ErrConflict       = errors.New("conflict")
)

type APIError struct {
    Message string
    URL      string
    Status   int
    Document string
}

func (e *APIError) Error() string
```

### Pagination

```go
type ListOptions struct {
    Page     int
    PerPage  int
    Sort     string
    Order    string
}

type Response struct {
    Result     interface{}
    TotalCount int
    Page       int
    PerPage    int
}
```

### Common request options

```go
type CreateRepoOptions struct {
    Name          string
    Description   string
    Private       bool
    AutoInit      bool
    Template      bool
    Gitignores    string
    License       string
    Readme        string
    DefaultBranch string
}

type CreateIssueOptions struct {
    Title     string
    Body      string
    Assignee  string
    Assignees []string
    Labels    []string
    Milestone int
}

type CreatePullRequestOptions struct {
    Title       string
    Body        string
    Base        string
    Head        string
    Draft       bool
    Assignee    string
    Assignees   []string
    Labels      []string
    Milestone   int
}
```

---

## Related documentation

### Internal documentation

| Resource | Description | Location |
|----------|-------------|----------|
| RFC | Package specification and design | [plans/code/core/go/forge/RFC.md](file:///Users/snider/Code/meowmix/plans/code/core/go/forge/RFC.md) |

### External references

| Resource | URL |
|----------|-----|
| Repository | [github.com/dappcore/go-forge](https://github.com/dappcore/go-forge) |
| Go Module | [pkg.go.dev/dappco.re/go/forge](https://pkg.go.dev/dappco.re/go/forge) |
| Forgejo | [forgejo.org](https://forgejo.org) |
| Gitea | [gitea.io](https://gitea.io) |
| Codeberg | [codeberg.org](https://codeberg.org) |

