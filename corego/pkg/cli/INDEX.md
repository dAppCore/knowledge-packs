---
type: Package Index
package: cli
module: forge.lthn.ai/core/cli
repo: core/cli
title: go-cli Package Index
description: Unified command-line interface for the Core ecosystem with Cobra, Bubbletea, and full i18n support
tags:
  - cli
  - command-line
  - cobra
  - tui
  - bubbletea
  - lipgloss
  - i18n
  - localization
  - daemon
  - service-management
---
# go-cli Package Index

> **Unified Command-Line Interface for the Core Ecosystem**

**Repository:** `core/cli`  
**Module:** `forge.lthn.ai/core/cli`  
**Binary:** `core`  
**Status:** ✅ Production-Ready  
**License:** EUPL-1.2  
**Test Pattern:** Good/Bad/Ugly (AX Standard)  
**Last Updated:** 2026-06-17  
**Maintainer:** Purberus <purberus@lthn.ai>

---

## 📚 Documentation

| Document | Description | Path |
|----------|-------------|------|
| README | Complete package documentation | [README.md](README.md) |
| Repository README | Original package README | [~/Code/core/cli/README.md](file:///Users/snider/Code/core/cli/README.md) |
| CLAUDE.md | Development guidance for Claude Code | [~/Code/core/cli/CLAUDE.md](file:///Users/snider/Code/core/cli/CLAUDE.md) |

---

## 🎯 Package Overview

`go-cli` is the **unified command-line interface** for the Core ecosystem, providing a single `core` binary that wraps common development tooling (testing, linting, building, releasing, multi-repo management) behind a consistent interface. It is built on **Cobra** for command parsing with a **Bubbletea-based TUI framework** and full **i18n/localization** support.

### Core Capabilities

1. **Unified Development Workflow** — Single command for all development tasks across Go, PHP, and Wails projects
2. **Environment Verification** — `core doctor` for diagnosing development environment issues
3. **Package Management** — Install, remove, search, and manage Core packages
4. **Project Scaffolding** — Generate new projects with proper structure and configuration
5. **Build & CI** — Consistent build, test, lint, and release workflows
6. **Multi-Repo Operations** — Execute commands across multiple repositories
7. **Daemon Management** — Start, stop, list, and manage background services
8. **Interactive TUI** — Rich terminal user interface for complex workflows
9. **Internationalization** — Full i18n support with grammar-aware messaging

### Design Philosophy

- **Token-Efficient** — Minimize boilerplate and cognitive overhead for developers
- **Composable** — Commands register explicitly onto Core framework instance
- **Unified** — Single binary with consistent patterns across all commands
- **Intuitive** — Clear command structure with helpful output
- **Extensible** — External modules can register commands without coupling
- **Localizable** — Full i18n support for global developer community
- **Observable** — Rich output with progress tracking and styled formatting
- **Testable** — Complete test triplet coverage (_test.go + _example_test.go)

### Binary Information

- **Binary Name:** `core`
- **Install:** `go install forge.lthn.ai/core/cli/cmd/core@latest`
- **Build:** `go build -o core ./cmd/core/`
- **Module Path:** `forge.lthn.ai/core/cli`

---

## 🏗️ Architecture

### Component Layers

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Binary Layer                                             │
│  core binary (forge.lthn.ai/core/cli/cmd/core)                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Main Entry (cmd/core/main.go)                          │
│  cli.Main() — Initializes Core + registers commands                        │
│  Explicit registration via cli.WithCommands()                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                    CLI Runtime (pkg/cli)                                   │
│  runtime.go, command.go, service.go, output.go, errors.go              │
│  prompt.go, tracker.go, daemon.go, stream.go, tree.go                  │
│  layout.go, check.go, ansi.go, styles.go, glyph.go                       │
│  i18n.go, render.go, utils.go, io.go, log.go                              │
│  action_mount.go                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Frame TUI (pkg/cli/frame)                               │
│  Bubbletea-based framework with HLCRF layout                             │
│  frame.go, frame_model.go, layout.go, frame_components.go               │
│  io.go, style.go                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                    i18n Layer (pkg/cli/locales + pkg/i18n)                  │
│  Translation files for all supported languages                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Command Layer (cmd/core/)                                │
│  Built-in: config, doctor, help, pkgcmd                                  │
│  External: build, go, qa, + more from ecosystem                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                    Core Framework Layer                                     │
│  dappco.re/go — ServiceRuntime, ACTION system, Result pattern             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Repository Structure

```
core/cli/
├── go/                                  # Go module root
│   ├── cmd/core/                        # Binary entry point
│   │   ├── main.go                      # Main entry + command registration
│   │   ├── actions_demo.go              # Demo actions for testing
│   │   │
│   │   ├── config/                      # Configuration management commands
│   │   │   ├── cmd.go                   # Main config command
│   │   │   ├── cmd_get.go               # Get subcommand
│   │   │   ├── cmd_set.go               # Set subcommand
│   │   │   ├── cmd_list.go              # List subcommand
│   │   │   └── cmd_path.go              # Path subcommand
│   │   │
│   │   ├── doctor/                      # Diagnostic commands
│   │   │   ├── cmd_doctor.go            # Main doctor command
│   │   │   ├── cmd_environment.go       # Environment check
│   │   │   ├── cmd_commands.go          # Commands check
│   │   │   ├── cmd_checks.go            # Specific checks
│   │   │   └── cmd_install.go           # Installation checks
│   │   │
│   │   ├── help/                        # Help system
│   │   │   └── cmd.go                   # Help command with grammar
│   │   │
│   │   └── pkgcmd/                     # Package management
│   │       ├── cmd_pkg.go               # Main pkg command
│   │       ├── cmd_commands.go          # Commands subcommand
│   │       ├── cmd_install.go           # Install subcommand
│   │       ├── cmd_list.go              # List subcommand
│   │       ├── cmd_manage.go            # Manage subcommand
│   │       ├── cmd_remove.go            # Remove subcommand
│   │       └── cmd_search.go            # Search subcommand
│   │
│   ├── pkg/cli/                        # CLI Runtime Package
│   │   ├── runtime.go                  # Lifecycle management
│   │   ├── command.go                  # Command builders
│   │   ├── service.go                  # Service management
│   │   ├── output.go                   # Styled output functions
│   │   ├── errors.go                   # Error handling
│   │   ├── prompt.go                   # Interactive prompting
│   │   ├── tracker.go                  # Task tracker with spinners
│   │   ├── daemon.go                   # Daemon mode detection
│   │   ├── stream.go                   # Streaming output
│   │   ├── tree.go                     # Tree rendering
│   │   ├── layout.go                   # Layout utilities
│   │   ├── check.go                    # Pre-condition checking
│   │   ├── ansi.go                     # ANSI utilities
│   │   ├── styles.go                   # Lipgloss styles
│   │   ├── glyph.go                    # Emoji/symbol shortcodes
│   │   ├── render.go                   # Rendering utilities
│   │   ├── utils.go                    # General utilities
│   │   ├── io.go                       # I/O utilities
│   │   ├── log.go                      # Logging utilities
│   │   ├── strings.go                  # String utilities
│   │   ├── i18n.go                     # i18n integration
│   │   └── action_mount.go             # Action registry mounting
│   │   │
│   │   ├── frame/                     # Bubbletea TUI Framework
│   │   │   ├── frame.go                # Main frame implementation
│   │   │   ├── frame_model.go          # Frame model
│   │   │   ├── frame_components.go     # Component definitions
│   │   │   ├── layout.go               # Layout management
│   │   │   ├── io.go                   # Frame I/O
│   │   │   └── style.go               # Frame styles
│   │   │
│   │   └── locales/                   # i18n Translation Files
│   │       └── ...
│   │
│   ├── internal/term/                 # Terminal utilities
│   │   └── term.go
│   │
│   ├── go.mod
│   ├── go.sum
│   └── go.work
│
├── .core/                             # Runtime configuration
│   └── config.yaml
├── docs/                              # Documentation
│   └── user-guide.md
├── README.md                          # Repository README
├── CLAUDE.md                          # Development guidance
├── LICENCE                           # EUPL-1.2 license
└── Taskfile.yml                      # Task definitions
```

---

## 🎯 Core Commands

### Built-in Commands

#### 1. `core config` — Configuration Management

**Description:** Manage Core CLI configuration with YAML-based files and environment variable overrides.

**Subcommands:**
- `core config get <key>` — Get configuration value
- `core config set <key> <value>` — Set configuration value
- `core config list` — List all configuration
- `core config path` — Show configuration file path

**Configuration Priority:**
1. Command-line flags
2. Environment variables (`CORE_CONFIG_*` prefix)
3. Configuration file (`~/.core/config.yaml`)
4. Default values

#### 2. `core doctor` — Environment Diagnosis

**Description:** Diagnose development environment issues and verify prerequisites.

**Subcommands:**
- `core doctor` — Run all diagnostic checks
- `core doctor environment` — Check environment variables and paths
- `core doctor commands` — Verify required commands are available
- `core doctor install` — Check installation and dependencies
- `core doctor checks` — Run specific diagnostic checks

**Checks:**
- Go version and toolchain
- Node.js and npm/yarn/pnpm
- Docker and container runtime
- Git and GitHub CLI
- Core dependencies
- Environment variables
- File system permissions

#### 3. `core help` — Help System

**Description:** Get help with grammar-aware i18n support.

**Features:**
- Context-sensitive help
- Grammar-aware messages
- Localized output
- Command usage examples

#### 4. `core pkg` — Package Management

**Description:** Manage Core ecosystem packages.

**Subcommands:**
- `core pkg install <package>` — Install a package
- `core pkg remove <package>` — Remove a package
- `core pkg list` — List installed packages
- `core pkg search <query>` — Search for packages
- `core pkg manage` — Manage package versions
- `core pkg commands` — List package commands

### External Commands (Ecosystem)

Commands from external modules that integrate with the CLI:

- `core build` — Build & release commands (from `dappco.re/go/build`)
- `core go` — Go development commands (from `dappco.re/go/lint`)
- `core qa` — Quality assurance commands (from `dappco.re/go/lint`)
- `core agent` — Agent orchestration commands (from `dappco.re/go/agent`)

### Action Projection

The action registry is projected onto the CLI as commands, providing a third surface for the same capability map that the API and MCP layers project.

- Runs last so explicit commands win any path collision
- Uses go-i18n-generated help text
- Grammar-aware messaging

---

## 🏗️ CLI Runtime (pkg/cli)

### Key Components

#### Runtime (`runtime.go`)
- Singleton lifecycle management (Init, Shutdown, Execute)
- Core framework instance creation
- Service attachment with dependency injection
- Signal handling (SIGINT, SIGTERM, SIGHUP)
- Cleanup and graceful shutdown

#### Command Builders (`command.go`)
- `NewCommand()` — Create command with error return
- `NewGroup()` — Create command group
- `NewRun()` — Create simple command without error
- Flag helpers: `StringFlag()`, `BoolFlag()`, `IntFlag()`
- Type aliases for Cobra types (Command ≈ cobra.Command)

#### Service Management (`service.go`)
- Service lifecycle management
- Dependency injection
- Start/Stop ordering
- Error propagation

#### Styled Output (`output.go`)
**Semantic Output Functions:**
- `Success()` — Green checkmark for success messages
- `Error()` — Red cross for error messages
- `Warn()` — Yellow for warnings
- `Info()` — Blue for information
- `Dim()` — Gray for secondary info
- `Progress()` — Progress indicator
- `Label()` — Key-value pair display
- `Section()` — Section header
- `Hint()` — Suggestion/hint
- `Echo()` — Raw output without styling

**All functions have verbose variants:**
- `SuccessVerb()`, `ErrorVerb()`, etc.

#### Error Handling (`errors.go`)
- `Err()` — Create new error
- `Wrap()` — Wrap error with context
- `WrapVerb()` — Wrap with verbose output
- `Exit()` — Exit with error
- Re-exports: `Is()`, `As()`, `Join()` from errors package

#### Task Tracker (`tracker.go`)
- Concurrent task execution with spinners
- Progress bars for progress-tracked tasks
- TTY-aware (live updates vs static output)
- Error aggregation
- Task timeout support

#### Daemon Management (`daemon.go`)
**Modes:**
- `ModeInteractive` — Running in terminal with TTY
- `ModePipe` — Output piped to another command
- `ModeDaemon` — Running as daemon (detached)

**Daemon Service Management:**
- Reads `.core/manifest.yaml` from project directory
- Walks up directory tree to find manifest
- Daemons run with `CORE_DAEMON=1` environment variable
- Tracked in `~/.core/daemons/`

#### Tree Rendering (`tree.go`)
- Hierarchical data rendering
- Multiple styles: ASCII, Unicode, Rounded, Double
- Customizable appearance

#### Layout Utilities (`layout.go`)
- Grid layout
- Flex layout (row/column)
- Border rendering
- Alignment options

#### ANSI Utilities (`ansi.go`)
- Color constants (Red, Green, Blue, Yellow, Cyan, Magenta)
- Style constants (Bold, Dim, Italic, Underline)
- `ApplyStyle()` — Apply style to text
- `HasColorSupport()` — Check terminal color support
- `StripANSI()` — Remove ANSI codes

#### Styles (`styles.go`)
**Predefined Styles:**
- `StyleSuccess` — Green, bold
- `StyleError` — Red, bold
- `StyleWarn` — Yellow, bold
- `StyleInfo` — Blue
- `StyleDim` — Gray
- `StyleHeader` — Bold, underline

**Custom Style Creation:**
```go
style := cli.NewStyle().
    Foreground(cli.ColorRed).
    Bold(true).
    Italic(true)
```

#### Glyph System (`glyph.go`)
**Emoji/Symbol Shortcodes:**
- `:check:` — ✓ (success)
- `:cross:` — ✗ (failure)
- `:warn:` — ⚠ (warning)
- `:info:` — ℹ (information)
- `:arrow:` — → (arrow)
- `:star:` — ★ (star)
- `:dot:` — ● (dot)
- `:circle:` — ○ (circle)

**Features:**
- Theme support (Unicode, Emoji, ASCII)
- Auto-detection based on TERM and NO_COLOR
- Registration of custom glyphs
- Fallback to text when glyph unavailable

#### Prompt Utilities (`prompt.go`)
- `PromptConfirm()` — Yes/No prompt
- `PromptInput()` — Text input prompt
- `PromptPassword()` — Hidden input prompt
- `PromptSelect()` — Single selection from options
- `PromptMultiSelect()` — Multiple selection from options
- Support for default values and validation

#### i18n Integration (`i18n.go`)
- `T()` — Get translation
- `TN()` — Get translation with pluralization
- `TC()` — Get translation with context
- `HasTranslation()` — Check if translation exists
- `SetLocale()` — Set current locale

---

## 🎨 Frame TUI Framework

Bubbletea-based terminal user interface framework with HLCRF (Header, Left, Content, Right, Footer) layout.

### HLCRF Layout

```
┌─────────────────────────────────────────────────────────────┐
│                    Header Region                              │
├───────────────┬─────────────────┬───────────────┬─────────┤
│   Left Region  │   Content Region │  Right Region  │ Footer   │
│   (sidebar)    │    (main)        │   (sidebar)    │         │
├───────────────┴─────────────────┴───────────────┴─────────┘
```

### Frame Components

1. **List** — Scrollable list with selection
2. **Tree** — Hierarchical tree view
3. **Table** — Data table with headers
4. **Form** — Interactive form with fields
5. **Text** — Styled text display
6. **Progress** — Progress bar
7. **Spinner** — Loading spinner
8. **Modal** — Modal dialog
9. **Tabs** — Tab navigation
10. **Menu** — Context menu

### Navigation

- Focus management between regions
- Focus stack for navigation history
- Push/Pop focus operations

---

## 🌐 i18n Support

### Features

- **Locale System:** Support for multiple languages
- **Grammar-Aware:** Proper pluralization and verb conjugation
- **Translation Files:** YAML-based translation catalogs
- **Context Support:** Context-aware translations

### Grammar System

**Action Messages:**
```go
cli.ActionCreated("file")  // "file was created" / "files were created"
cli.ActionDeleted("item")   // "item was deleted" / "items were deleted"
cli.ActionUpdated("user")  // "user was updated" / "users were updated"
```

**Progress Messages:**
```go
cli.ProgressStarting("Build")  // "Starting build..."
cli.ProgressCompleted("Build") // "Build completed"
cli.ProgressFailed("Build")   // "Build failed"
```

**With Count:**
```go
cli.ActionFound("file", 1)   // "1 file was found"
cli.ActionFound("file", 5)   // "5 files were found"
```

---

## 🚀 Quick Start

### Installation

```bash
# Install from source
go install forge.lthn.ai/core/cli/cmd/core@latest

# Or build from source
cd ~/Code/core/cli/go
go build -o core ./cmd/core/
```

### Basic Usage

```bash
# Verify environment
core doctor

# Run tests
core go test

# Build project
core build

# Get help
core help

# Manage configuration
core config get log.level
core config set log.level debug

# Manage packages
core pkg list
core pkg search process
core pkg install dappco.re/go/process

# Run as daemon
CORE_DAEMON=1 core serve
```

---

## 🧪 Testing

### Test Structure

Complete test triplet coverage following AX Standard:
- `*_test.go` — Unit and integration tests
- `*_example_test.go` — Usage examples as tests

**Naming Convention:**
- `TestGood*` — Happy path tests
- `TestBad*` — Expected error conditions
- `TestUgly*` — Panics and edge cases

### Running Tests

```bash
# All tests
cd ~/Code/core/cli/go
go test -v ./...

# Specific package
go test -v ./pkg/cli/...

# With coverage
go test -cover ./...

# With race detector
go test -race ./...
```

---

## 📊 Statistics

| Metric | Value |
|--------|-------|
| **Total Go files** | ~100 |
| **Test files** | ~80 |
| **Example test files** | ~40 |
| **Lines of code** | ~40,000 |
| **Built-in commands** | 4 (config, doctor, help, pkg) |
| **External commands** | 3+ (build, go, qa, + ecosystem) |
| **TUI components** | 10+ |
| **i18n locales** | 5+ |
| **Configuration keys** | 50+ |

---

## 🔗 Related Packages

| Package | Relationship | Path |
|---------|--------------|------|
| [CoreGO Framework](file:///Users/snider/Code/core/go/) | Foundation framework | N/A |
| [go-agent](../agent/) | Agent orchestration | [../agent/](file:///Users/snider/Code/meowmix/knowledge-packs/corego/pkg/agent/) |
| [go-build](file:///Users/snider/Code/core/go-build/) | Build commands | Ecosystem |
| [go-lint](file:///Users/snider/Code/core/go-lint/) | Lint commands | Ecosystem |
| [CoreGO INDEX](../../INDEX.md) | Complete package catalog | [../../INDEX.md](file:///Users/snider/Code/meowmix/knowledge-packs/corego/INDEX.md) |

---

## 📈 Quality Metrics

- ✅ **Cobra Integration** — Full Cobra command framework support
- ✅ **Bubbletea TUI** — Rich terminal user interface
- ✅ **Lipgloss Styles** — Comprehensive styling support
- ✅ **Test Coverage** — Good/Bad/Ugly pattern for all scenarios
- ✅ **i18n Support** — Full internationalization with grammar
- ✅ **Documentation** — Complete README + INDEX
- ✅ **Core Integration** — Full CoreGO framework support
- ✅ **Extensible** — External modules can register commands
- ✅ **Token-Efficient** — Minimal boilerplate for developers

---

## 📝 Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-06-17 | Complete knowledge pack documentation | Purberus |
| 2026-05-XX | CLAUDE.md architecture documentation | Maintainer |
| 2026-04-XX | i18n system integrated | Maintainer |
| 2026-03-XX | Bubbletea TUI framework added | Maintainer |
| 2026-02-XX | Initial CLI creation | Maintainer |

---

## 🎯 Tags

```yaml
# Core Capabilities
- cli
- command-line-interface
- cobra
- developer-tools
- build-tool
- tui
- terminal-ui
- bubbletea
- lipgloss

# Features
- command-management
- flag-parsing
- argument-handling
- error-handling
- output-formatting
- styled-output
- ansi-colors
- progress-tracking
- task-tracker
- spinner
- daemon-management

# i18n
- internationalization
- localization
- grammar-aware
- pluralization

# Architecture
- composable
- extensible
- modular
- dependency-injection
- lifecycle-management

# Quality
- production-ready
- high-coverage
- testable
- documented

# Ecosystem
- corego
- wails
- dappcore
- lethean
- forgejo
```

---

## 📚 References

1. **Repository** — [~/Code/core/cli/](file:///Users/snider/Code/core/cli/)
2. **CLAUDE.md** — [~/Code/core/cli/CLAUDE.md](file:///Users/snider/Code/core/cli/CLAUDE.md)
3. **Repository README** — [~/Code/core/cli/README.md](file:///Users/snider/Code/core/cli/README.md)
4. **Cobra** — [github.com/spf13/cobra](https://github.com/spf13/cobra)
5. **Bubbletea** — [github.com/charmbracelet/bubbletea](https://github.com/charmbracelet/bubbletea)
6. **Lipgloss** — [github.com/charmbracelet/lipgloss](https://github.com/charmbracelet/lipgloss)
7. **CoreGO Framework** — [CoreGO INDEX](../../INDEX.md)

---

*Package index generated: 2026-06-17T21:30:00Z*
*Knowledge Pack: CoreGo v1.2.0*
*Repository: forge.lthn.ai/core/cli*
*Maintainer: Purberus <purberus@lthn.ai>*
