---
type: Package Documentation
package: cli
module: dappco.re/go/cli
repo: core/cli
lang: go
tags:
  - cli
  - commands
  - terminal
  - shell
  - ansi
  - semantic-output
  - zero-dependencies
  - cobra-alternative
  - glyph-system
  - hlcrf
  - build-variants
---
# core/cli — Semantic CLI Framework

> **"Zero external dependencies for output. Self-registering commands. Semantic output."**

**RFC:** [plans/code/core/go/cli/RFC.md](../../../../../plans/code/core/go/cli/RFC.md)
**Source:** [~/Code/core/cli/](file:///Users/snider/Code/core/cli/)
**Module:** `dappco.re/go/cli`
**Dependencies:** `dappco.re/core/go`

---

## 🎯 Overview

`core/cli` is a **semantic CLI framework** that replaces external dependencies (cobra, lipgloss, huh) with a zero-dependency, self-registering, AI-native command system. It's the foundation for all CoreGo CLI binaries.

### Design Principles

1. **Zero external dependencies for output** — Consuming code imports `cli` only, never `fmt`, `lipgloss`, or `huh`
2. **Self-registering commands** — Packages register via `init()`, build tags control what's compiled
3. **Semantic output** — `cli.Success()`, `cli.Error()`, `cli.Progress()` replace raw `fmt.Printf`
4. **Defence in depth** — Only compiled code exists in binary. No code = no vulnerabilities

### Primary Use Cases

- **Core CLI binary** — The main `core` command with all subcommands
- **Service CLIs** — Individual service binaries with semantic output
- **AI-assisted workflows** — Commands with `--ai` flag for opt-in AI assistance
- **Compatibility layers** — Drop-in replacements for other tools (ansible, terraform, kubectl)
- **Embedded in applications** — Any Go application can use `cli` for consistent output

### Key Innovation: Replaced Dependencies

| External Package | Transitive Deps | Replacement | Lines Saved |
|-----------------|-----------------|-------------|-------------|
| lipgloss | ~15 | `cli.NewStyle()` | ~100 lines ANSI |
| huh | ~30 | `cli.Prompt()`, `cli.Confirm()`, `cli.Select()` | Custom impl |
| cobra | ~20 | `cli.Command` | Custom impl |
| **Total** | **~65** | **core/cli** | **0 dependencies** |

---

## 🏗️ Architecture

### Component Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                           │
│  CLI binaries: core, core-dev, service binaries               │
├─────────────────────────────────────────────────────────────┤
│                    SEMANTIC OUTPUT LAYER                       │
│  Success, Error, Warn, Info, Debug, Echo                    │
│  Structured: Field, List, Table, Progress                     │
│  Prompts: Prompt, Confirm, Select                             │
├─────────────────────────────────────────────────────────────┤
│                    STYLING LAYER                               │
│  AnsiStyle API (Bold, Dim, Italic, Underline)                 │
│  Tailwind CSS colour palette                                 │
│  Predefined styles (Success, Error, Warn, Info, etc.)         │
├─────────────────────────────────────────────────────────────┤
│                    GLYPH SYSTEM                                │
│  ThemeUnicode (✓ ✗ ⚠ ℹ → • …)                               │
│  ThemeEmoji (✅ ❌ ⚠️ ℹ️ ➡️ • …)                                 │
│  ThemeASCII ([ok] [err] [warn] [info] -> * ...)                │
│  Auto-detection based on TERM and NO_COLOR                    │
├─────────────────────────────────────────────────────────────┤
│                    COMMAND REGISTRATION LAYER                  │
│  cli.RegisterCommands() in init()                            │
│  Command tree with self-registering packages                 │
│  Core.Command integration (Action system)                    │
├─────────────────────────────────────────────────────────────┤
│                    LAYOUT LAYER (HLCRF)                        │
│  Header, Content, Footer regions                              │
│  Left sidebar for tree/list views                             │
│  Port of RFC-001 HLCRF compositor                             │
├─────────────────────────────────────────────────────────────┤
│                    TRANSFORMER LAYER                           │
│  CLITransformerIn — foreign args → Core args                  │
│  CLITransformerOut — Core output → foreign format              │
│  Build-tag gated (ansible_compat, terraform_compat, etc.)    │
└─────────────────────────────────────────────────────────────┘
```

### Integration with Core Framework

```go
// Commands register via Core's Action system
func (s *MyService) OnStartup(ctx context.Context) core.Result {
    c := s.Core()
    c.Command("issue/get", core.Command{Action: s.cmdIssueGet})
    c.Command("issue/list", core.Command{Action: s.cmdIssueList})
    return core.Result{OK: true}
}

// Command path IS the route:
//   CLI:     core issue get
//   HTTP:    GET /issue/get
//   i18n:    issue.get.*
//   MCP:     tools/call issue/get
```

---

## 📚 Core Concepts

### 1. Semantic Output Functions

Replace raw `fmt.Printf` with semantic functions that convey meaning:

```go
// Status outputs
cli.Success("Build complete")      // ✓ Build complete (green, bold)
cli.Error("Connection failed")     // ✗ Connection failed (red, bold)
cli.Warn("Rate limited")          // ⚠ Rate limited (amber, bold)
cli.Info("Connecting...")          // ℹ Connecting... (blue, bold)
cli.Debug("Loaded 42 modules")      // ○ Loaded 42 modules (dim)

// With i18n integration
cli.Success(i18n.T("build.complete"))

// Raw output (no styling)
cli.Echo(key, args...)  // Just print, no semantic meaning
```

**Predefined Styles:**
```go
StyleSuccess = NewStyle().Bold().Foreground(ColourGreen500)
StyleError   = NewStyle().Bold().Foreground(ColourRed500)
StyleWarn    = NewStyle().Bold().Foreground(ColourAmber500)
StyleInfo    = NewStyle().Bold().Foreground(ColourBlue500)
StyleDebug   = NewStyle().Dim()
StyleHeader  = NewStyle().Bold().Foreground(ColourBlue300)
StyleKey     = NewStyle().Foreground(ColourBlue200)
StyleValue   = NewStyle().Foreground(ColourBlue50)
StyleCommand = NewStyle().Bold().Foreground(ColourCyan500)
StylePath    = NewStyle().Underline().Foreground(ColourViolet400)
```

### 2. AnsiStyle API (Zero-Dependency)

```go
// Create custom style
style := cli.NewStyle().Bold().Foreground("#3b82f6")
output := style.Render("Hello")  // bold blue text

// Chainable modifiers
cli.NewStyle().
    Bold().
    Dim().
    Italic().
    Underline().
    Foreground("#hex").
    Background("#hex").
    Render("text")

// Supported: bold, dim, italic, underline, 24-bit RGB/hex foreground and background
```

### 3. Colour Palette (Tailwind CSS)

```go
// Primary colours (500 shade)
ColourBlue500   = "#3b82f6"
ColourGreen500  = "#22c55e"
ColourRed500    = "#ef4444"
ColourAmber500  = "#f59e0b"
ColourPurple500 = "#a855f7"
ColourCyan500   = "#06b6d4"

// All Tailwind shades available: 50, 100, 200, 300, 400, 500, 600, 700, 800, 900
// Grey scale: Grey50 to Grey900
```

### 4. Glyph System

Three themes for different terminal capabilities:

```go
type GlyphTheme struct {
    Check    string   // ✓  ✅  [ok]
    Cross    string   // ✗  ❌  [err]
    Warning  string   // ⚠  ⚠️   [warn]
    Info     string   // ℹ  ℹ️   [info]
    Arrow    string   // →  ➡️   ->
    Bullet   string   // •  •   *
    Ellipsis string   // …  ...  ...
    Spinner  []string // [⠋⠙⠹⠸⠼⠴⠦⠧] or [|/-\] or [.oOo.]
}

// Predefined themes
ThemeUnicode = GlyphTheme{Check: "✓", Cross: "✗", Warning: "⚠", ...}
ThemeEmoji   = GlyphTheme{Check: "✅", Cross: "❌", Warning: "⚠️", ...}
ThemeASCII   = GlyphTheme{Check: "[ok]", Cross: "[err]", Warning: "[warn]", ...}
```

**Auto-Detection:**
```go
func init() {
    if os.Getenv("NO_COLOR") != "" || os.Getenv("TERM") == "dumb" {
        activeTheme = ThemeASCII
    } else if os.Getenv("CORE_EMOJI") == "1" {
        activeTheme = ThemeEmoji
    } else {
        activeTheme = ThemeUnicode
    }
}
```

**Inline Glyphs:**
```go
cli.Print(":check: Done")        // ✓ Done (resolved at render time)
cli.Print(":arrow: Next step")   // → Next step
```

### 5. Structured Output

```go
// Key-value pairs
cli.Field("Version", "0.8.0")          //   Version: 0.8.0
cli.Field("Status", "running")         //   Status: running

// Lists
cli.List("Modules", []string{"git", "agentic", "brain"})
//   Modules:
//     • git
//     • agentic
//     • brain

// Tables
cli.Table(headers, rows)  // aligned columns with borders

// Progress bars
p := cli.Progress("Building", 100)
p.Update(42)                           // Building [████░░░░░░] 42%
p.Done()                               // Building [██████████] ✓
```

### 6. Prompts (Replacing huh)

```go
// Text input with default
name := cli.Prompt("Project name", "my-project")

// Yes/No confirmation
ok := cli.Confirm("Continue?")

// Selection from list
choice := cli.Select("Choose", []string{"a", "b", "c"})  // numbered list
```

### 7. HLCRF Terminal Layout

Port of RFC-001 HLCRF compositor for structured terminal output:

**Simple Layout:**
```
┌─────────────────────────────────────┐
│ Header: command name + version      │  H
├─────────────────────────────────────┤
│ Content: primary output             │  C
│                                     │
│                                     │
├─────────────────────────────────────┤
│ Footer: status bar + elapsed time   │  F
└─────────────────────────────────────┘
```

**With Sidebar:**
```
┌─────────────────────────────────────┐
│ Header                              │
├──────────┬──────────────────────────┤
│ Left     │ Content                  │
│ (tree)   │ (detail)                 │
│          │                          │
├──────────┴──────────────────────────┤
│ Footer                              │
└─────────────────────────────────────┘
```

**Usage:**
```go
layout := cli.NewLayout()
layout.Header("core dev status")
layout.Content(func(w io.Writer) {
    cli.Table(w, headers, rows)
})
layout.Footer("3 repos dirty • 12s")
layout.Render()
```

### 8. Command Registration

**cobra is banned.** All command registration uses `cli.Command`:

```go
// Self-registering packages
// pkg/cli/commands.go
type CommandRegistration func(root *cli.Command)

func RegisterCommands(fn CommandRegistration)

// Package pattern
// pkg/php/cmd.go
package php

import "dappco.re/go/cli"

func init() {
    cli.RegisterCommands(AddCommands)
}

func AddCommands(root *cli.Command) {
    phpCmd := root.Command("php", "PHP tooling")
    phpCmd.Command("composer", "Dependency management")
    // ...
}
```

**Core.Command Integration (v0.8.0):**
```go
func (s *MyService) OnStartup(ctx context.Context) core.Result {
    c := s.Core()
    c.Command("issue/get", core.Command{Action: s.cmdIssueGet})
    c.Command("issue/list", core.Command{Action: s.cmdIssueList})
    return core.Result{OK: true}
}
```

### 9. Build Variants

Control which packages are compiled using build tags:

**Variant Files:**
```
cmd/
├── main.go
└── variants/
    ├── full.go      # default: all packages
    ├── ci.go        # CI/release only
    ├── php.go       # PHP tooling only
    ├── agent.go     # AI agent variant
    └── minimal.go   # core only
```

**Full Variant (Default):**
```go
//go:build !ci && !php && !minimal && !agent

package variants

import (
    _ "dappco.re/go/cli/pkg/ai"
    _ "dappco.re/go/cli/pkg/build"
    _ "dappco.re/go/cli/pkg/dev"
    _ "dappco.re/go/cli/pkg/doctor"
    _ "dappco.re/go/cli/pkg/go"
    _ "dappco.re/go/cli/pkg/php"
    _ "dappco.re/go/cli/pkg/sdk"
    _ "dappco.re/go/cli/pkg/setup"
)
```

**CI Variant:**
```go
//go:build ci

package variants

import (
    _ "dappco.re/go/cli/pkg/build"
    _ "dappco.re/go/cli/pkg/ci"
    _ "dappco.re/go/cli/pkg/doctor"
    _ "dappco.re/go/cli/pkg/sdk"
)
```

**Build Commands:**
```bash
go build .                # full variant (default)
go build -tags ci .       # CI variant
go build -tags php .      # PHP-only variant
go build -tags agent .    # agent variant
```

**Benefits:**
- Smaller attack surface — only compiled code exists in binary
- Faster builds — fewer packages to compile
- Self-documenting — variant file lists exactly what's included
- Defence in depth — no code = no vulnerabilities

### 10. Built-in Commands

All variants include cross-cutting commands:

**Configuration (`core config`):**
```bash
core config get <key>       # Get a configuration value
core config set <key> <val> # Set a configuration value
core config list            # List all configuration
core config path            # Show config file path
```

**Configuration Keys (dot-separated namespaces):**
- `dev.editor` — editor for commit messages (vi, nano, code)
- `dev.gpg` — GPG key ID for signing
- `dev.pager` — pager for diffs (less, more, delta)
- `build.variant` — default build variant (full, ci, php, minimal)
- `registry.forge` — Forge instance URL
- `registry.github` — GitHub API endpoint
- `lint.check` — run lint on commit (true/false)
- `lint.fail_on` — fail threshold (error, warning, info)

**Doctor (`core doctor`):**
```bash
core doctor                 # Full diagnostic scan
core doctor environment     # Check system information
core doctor commands        # List all available commands
core doctor checks          # Run diagnostic checks
core doctor install         # Install missing tools
```

**Doctor Environment:** Shows OS, Go, PHP, Python, Git, container runtimes, shell

**Doctor Commands:** Lists all core commands with status, binary location, version, variant

**Doctor Checks:**
- SSH key setup (SSH_AUTH_SOCK, keys loaded)
- Git credentials (configured user, email)
- Core installation (PATH, GOBIN)
- Container image availability (Go, Node, PHP)
- Network connectivity (Forge, GitHub, API endpoints)
- Disk space (workspace, cache, tmp)
- Permissions (workspace dirs, git repos)
- Language tools (golangci-lint, phpstan, biome availability)

**Doctor Install:**
- Creates `.core/` config directory
- Generates default `.core/lint.yaml`
- Installs required language tools
- Sets up shell completions
- Validates SSH and GPG keys

### 11. Operations Safety Taxonomy

| Category | Friction | Examples | SSH Gate |
|----------|----------|---------|----------|
| **Read** | None | `status`, `issues`, `prs`, `ci`, `diff`, `ahead`, `behind` | No |
| **Write-Local** | Low | `commit`, `branch`, `stash` | No |
| **Write-Remote** | Confirm | `push`, `pull`, `sync`, `merge`, `release` | Yes |
| **Destructive** | Double confirm | `reset`, `clean`, `force-push`, `delete` | Yes |

**SSH key passphrase is a FEATURE** — preserves human friction for remote operations.

### 12. AI-Native Convention

The `--ai` flag convention: every command works WITHOUT AI. AI is opt-in, never forced.

```
Human-first:     core status         → Just show me the state
AI-assisted:     core commit --ai    → AI drafts message, I approve
AI-delegated:    core dispatch 42    → Send to agent, notify when done
```

Human confirms all destructive operations. AI logs its work for human review.

### 13. FrankenPHP Bridge

Go calls PHP directly via FrankenPHP — no network hop, no separate server:

```
core dispatch 42
  → Go parses args
  → Go calls embedded PHP (AgentRouter)
  → PHP creates task, dispatches to agent
  → Go shows confirmation
```

Single binary does CLI + server + MCP + PHP. Laravel services available to all Go code.

---

## 🎯 API Reference

### Package-Level Functions

```go
// Semantic output
cli.Success(message)           // ✓ message (green)
cli.Error(message)            // ✗ message (red)
cli.Warn(message)             // ⚠ message (amber)
cli.Info(message)             // ℹ message (blue)
cli.Debug(message)            // ○ message (dim)
cli.Echo(message)             // raw output, no styling

// Structured output
cli.Field(key, value)         // Key: value
cli.List(key, items)          // Key:\n  • item1\n  • item2
cli.Table(headers, rows)      // aligned table with borders

// Prompts
cli.Prompt(label, default)    // text input
cli.Confirm(label)            // yes/no
cli.Select(label, options)   // select from list

// Progress
p := cli.Progress(label, total)
p.Update(current)             // update progress
p.Done()                       // complete with checkmark

// Styling
style := cli.NewStyle()
style = style.Bold().Foreground("#rrggbb")
output := style.Render(text)

// Glyphs (inline)
cli.Print(":check: Done")   // uses active theme's check glyph
```

### Command Types

```go
type Command struct {
    Use     string        // command name
    Short   string        // short description
    Long    string        // long description
    Action  func(*Command, []string) error  // command handler
    // ... more fields
}

// Registration
func RegisterCommands(fn CommandRegistration)
func AddCommands(root *Command)  // called by init()
```

### Layout (HLCRF)

```go
type Layout struct {
    header  string
    content func(io.Writer)
    footer  string
}

func NewLayout() *Layout
func (l *Layout) Header(h string)
func (l *Layout) Content(c func(io.Writer))
func (l *Layout) Footer(f string)
func (l *Layout) Render()
```

### Glyph Theme

```go
type GlyphTheme struct {
    Check    string
    Cross    string
    Warning  string
    Info     string
    Arrow    string
    Bullet   string
    Ellipsis string
    Spinner  []string
}

func SetGlyphTheme(theme GlyphTheme)
func GetGlyphTheme() GlyphTheme
```

### CLI Transformers (Compatibility Layer)

```go
// Transform foreign CLI args to Core args
type CLITransformerIn interface {
    Match(args []string) bool
    Transform(args []string) (command string, flags map[string]string, positional []string)
}

// Transform Core output to foreign format
type CLITransformerOut interface {
    FormatSuccess(result core.Result) string
    FormatError(err error) string
    FormatProgress(current, total int, message string) string
}

// Registration
func RegisterTransformer(t CLITransformerIn)
func RegisterOutputTransformer(t CLITransformerOut)
```

**Build Tag Convention:**
```
{tool}_compat     → full compatibility layer (in + out)
{tool}_compat_in  → args translation only (Core output format)
{tool}_compat_out → output translation only (Core args format)
```

**Example: Ansible Compatibility**
```go
//go:build ansible_compat

func init() {
    cli.RegisterTransformer(&AnsibleTransformer{})
}

func (t *AnsibleTransformer) Match(args []string) bool {
    return os.Args[0] == "ansible-playbook" || core.Contains(args, "-i")
}

func (t *AnsibleTransformer) Transform(args []string) (string, map[string]string, []string) {
    // ansible-playbook -i hosts site.yml --limit web --tags deploy
    // → core ansible run --inventory hosts --playbook site.yml --limit web --tags deploy
    return "ansible/run", flags, positional
}
```

Build and deploy:
```bash
go build -tags ansible_compat -o ansible-playbook
# Drop-in replacement — existing scripts work unchanged
```

---

## 📁 File Structure

```
core/cli/
├── cmd/
│   ├── main.go               # Entry point
│   └── variants/
│       ├── full.go           # Default variant (all packages)
│       ├── ci.go             # CI/release only
│       ├── php.go            # PHP tooling only
│       ├── agent.go          # AI agent variant
│       └── minimal.go        # Core only
│
├── go/
│   ├── pkg/
│   │   └── cli/
│   │       ├── command.go       # Command type and registration
│   │       ├── glyph.go         # Glyph themes (Unicode, Emoji, ASCII)
│   │       ├── style.go         # AnsiStyle API
│   │       ├── colours.go       # Tailwind CSS colour palette
│   │       ├── output.go        # Semantic output functions
│   │       ├── progress.go      # Progress bar implementation
│   │       ├── prompts.go       # Prompt, Confirm, Select
│   │       ├── layout.go        # HLCRF layout compositor
│   │       ├── table.go          # Table rendering
│   │       ├── list.go           # List rendering
│   │       ├── field.go          # Field (key-value) rendering
│   │       ├── service.go       # CLI service integration
│   │       ├── runtime.go       # CLI runtime management
│   │       ├── check.go          # Command availability checking
│   │       ├── app.go            # Application lifecycle
│   │       ├── log.go            # CLI logging
│   │       └── errors.go         # CLI error handling
│   │
│   └── internal/
│       └── term/
│           ├── term.go           # Terminal utilities
│           └── ...
│
├── docs/
├── README.md
├── LICENCE
├── CLAUDE.md
├── AGENTS.md
└── go.work
```

---

## 🚀 Usage Examples

### Minimal CLI

```go
package main

import "dappco.re/go/cli"

func main() {
    cli.Init(cli.Options{AppName: "myapp"})
    
    cli.Success("Hello, World!")
    cli.Info("This is an info message")
    cli.Warn("This is a warning")
    cli.Error("This is an error")
}
```

### With Structured Output

```go
cli.Field("Version", "1.0.0")
cli.Field("Status", "running")

cli.List("Features", []string{"Fast", "Reliable", "Secure"})

headers := []string{"Name", "Status", "Size"}
rows := [][]string{
    {"file1", "OK", "100KB"},
    {"file2", "Error", "200KB"},
}
cli.Table(headers, rows)
```

### With Progress

```go
p := cli.Progress("Processing", 100)
for i := 0; i < 100; i++ {
    p.Update(i)
    time.Sleep(50 * time.Millisecond)
}
p.Done()
```

### With Prompts

```go
name := cli.Prompt("Enter your name", "John Doe")
if cli.Confirm("Is this correct?") {
    choice := cli.Select("Choose an option", []string{"A", "B", "C"})
    cli.Success("You chose: %s", choice)
}
```

### Registering Commands

```go
// In init()
func init() {
    cli.RegisterCommands(func(root *cli.Command) {
        root.Command("greet", "Say hello").
            Action = func(cmd *cli.Command, args []string) error {
                if len(args) == 0 {
                    return cli.Error("Please provide a name")
                }
                cli.Success("Hello, %s!", args[0])
                return nil
            }
    })
}
```

### With Layout

```go
layout := cli.NewLayout()
layout.Header("System Status")
layout.Content(func(w io.Writer) {
    cli.Field(w, "Uptime", "42 hours")
    cli.Field(w, "Memory", "16 GB")
    cli.Field(w, "CPU", "8 cores")
})
layout.Footer("core status • 0.1s")
layout.Render()
```

### Build Variants

```bash
# Build full variant (all features)
go build .

# Build CI variant (only CI-related commands)
go build -tags ci .

# Build PHP variant (only PHP-related commands)
go build -tags php .

# Build minimal variant (core only)
go build -tags minimal .
```

### CLI Transformer

```go
// ansible_compat.go
//go:build ansible_compat

package compat

func init() {
    cli.RegisterTransformer(&AnsibleTransformer{})
}

type AnsibleTransformer struct{}

func (t *AnsibleTransformer) Match(args []string) bool {
    return len(args) > 0 && (args[0] == "ansible-playbook" || 
           core.Contains(args, "-i") || 
           core.Contains(args, "--inventory"))
}

func (t *AnsibleTransformer) Transform(args []string) (string, map[string]string, []string) {
    flags := make(map[string]string)
    positional := make([]string, 0)
    
    for i := 1; i < len(args); i++ {
        switch args[i] {
        case "-i", "--inventory":
            if i+1 < len(args) {
                flags["inventory"] = args[i+1]
                i++
            }
        default:
            if strings.HasPrefix(args[i], "--") {
                // Parse flag
            } else {
                positional = append(positional, args[i])
            }
        }
    }
    
    return "ansible/run", flags, positional
}
```

---

## 🔍 Advanced Features

### Custom Glyph Themes

```go
customTheme := cli.GlyphTheme{
    Check:   "✔",
    Cross:   "✖",
    Warning: "⚡",
    Info:    "ℹ",
    Arrow:   "→",
    Bullet:  "·",
    Ellipsis: "…",
    Spinner: []string{"⣾", "⣽", "⣻", "⢿", "⡿", "⣟", "⣯", "⣷"},
}

cli.SetGlyphTheme(customTheme)
```

### Custom Colour Palette

```go
// Override specific colours
cli.ColourBlue500 = "#myblue"

// Or use your own
style := cli.NewStyle().Foreground("#custom")
```

### Multiple Output Writers

```go
// Write to custom writer
cli.SuccessTo(writer, "message")
cli.ErrorTo(writer, "message")

// With styling
style := cli.NewStyle().Bold().Foreground(cli.ColourGreen500)
style.RenderTo(writer, "Custom output")
```

### Command Metadata

```go
cmd := root.Command("deploy", "Deploy application")
cmd.Short = "Deploy your application to production"
cmd.Long = `Deploy your application to production.

This command builds your application and deploys it to the configured
production environment. Use --dry-run to preview the deployment.`
cmd.Examples = []string{
    "core deploy",
    "core deploy --environment staging",
    "core deploy --ai",  // AI drafts deployment plan
}
cmd.Aliases = []string{"deploy", "push", "publish"}
```

### Safety Confirmations

```go
// Simple confirmation
if !cli.Confirm("Are you sure?") {
    return nil
}

// Dangerous operation (double confirm)
if !cli.Confirm("This will delete data. Are you sure?") {
    return nil
}
if !cli.Confirm("LAST CHANCE: Delete all data?") {
    return nil
}
```

---

## 📊 Performance & Quality

### Benchmarks

- Zero external dependencies = zero allocation overhead
- Inline glyph resolution = fast rendering
- Pooled buffers for progress bars
- Pre-computed style codes

### Test Coverage

- Good/Bad/Ugly triplets for all packages
- Example tests for API documentation
- Integration tests for command workflows

### Quality

- **AX Standard:** Full compliance
- **RFC:** 663 lines of comprehensive specification
- **Zero Transitive Dependencies:** Only depends on `core/go`

---

## 🔗 Related Knowledge Packs

| Package | Knowledge Pack | Relationship |
|---------|----------------|--------------|
| core/go | [../../../../../corego/README.md](../../../../../README.md) | Foundation framework (required dependency) |
| core/mcp | [../mcp/](../mcp/) | MCP server framework (uses CLI) |
| core/agent | [../agent/](../agent/) | Agent dispatch (uses CLI for commands) |
| core/api | [../api/](../api/) | REST framework (CLI ↔ HTTP bridge) |
| go-i18n | [../i18n/](../i18n/) | Internationalization (used for CLI translations) |

---

## 📈 Statistics

- **Total Files:** 50+ Go files
- **Lines of Code:** ~10,000 (estimated)
- **Test Coverage:** High (Good/Bad/Ugly triplets)
- **Dependencies:** 1 (core/go)
- **External Dependencies:** 0 (zero transitive dependencies!)

### Components
- **Output Functions:** 5 (Success, Error, Warn, Info, Debug)
- **Structured Output:** 4 (Field, List, Table, Progress)
- **Prompt Functions:** 3 (Prompt, Confirm, Select)
- **Glyph Themes:** 3 (Unicode, Emoji, ASCII)
- **Colour Constants:** 30+ (full Tailwind palette)
- **Style Modifiers:** 7 (Bold, Dim, Italic, Underline, FG, BG, Reset)
- **Layout Regions:** 3 (Header, Content, Footer)
- **Build Variants:** 5 (full, ci, php, agent, minimal)
- **Transformer Types:** 2 (In, Out)

### Removed Dependencies
- **lipgloss:** ~15 transitive deps replaced
- **huh:** ~30 transitive deps replaced
- **cobra:** ~20 transitive deps replaced
- **Total:** ~65 transitive dependencies eliminated

---

## 🎯 Key Takeaways

1. **Zero Dependencies** — No external packages for styling or prompts. Everything is custom-implemented in ~100 lines of ANSI code.
2. **Self-Registering** — Commands register via `init()` with build tags controlling inclusion.
3. **Semantic Output** — Functions convey meaning (Success, Error, Warn) not just formatting.
4. **Build Variants** — Different binaries with different feature sets, smaller attack surface.
5. **AI-Native** — Opt-in AI assistance with `--ai` flag, human always in control.
6. **Defence in Depth** — No code = no vulnerabilities. Build tags control attack surface.
7. **Compatibility Layer** — Drop-in replacements for other tools via CLI transformers.
8. **HLCRF Layout** — Structured terminal output with Header, Content, Footer, Left sidebar.
9. **Glyph System** — Three themes (Unicode, Emoji, ASCII) with auto-detection.
10. **Tailwind Palette** — Consistent colours with web UI.

---

## 📚 Learning Resources

- **Primary RFC:** [plans/code/core/go/cli/RFC.md](../../../../../plans/code/core/go/cli/RFC.md) (663 lines)
- **HLCRF Compositor:** [RFC-001-HLCRF-COMPOSITOR.md](file:///Users/snider/Code/meowmix/plans/rfc/RFC-001-HLCRF-COMPOSITOR.md)
- **AX Spec:** [RFC-025-AGENT-EXPERIENCE.md](file:///Users/snider/Code/core/docs/RFC-025-AGENT-EXPERIENCE.md)
- **IPC Worker Bundles:** [code/core/go/ipc/RFC.md](../../../../../plans/code/core/go/ipc/RFC.md) §5

---

*Knowledge Pack: core/cli v1.0.0*
*Last Updated: 2026-06-17*
*Maintained by: Purberus <purberus@lthn.ai>*
