---
type: Knowledge Pack
title: CoreCLI Framework
description: Complete knowledge pack for CoreCLI — the CLI framework for Lethean command-line tools
author: Mistral Vibe
version: 1.0.0
created: 2026-06-17T14:40:00Z
tags: [framework, cli, command-line, lethean, cobra]
---

# CoreCLI Knowledge Pack

> **"CLI scaffolding — Cobra-equivalent built on core/go primitives"**

**Official Site:** [dappco.re/go/cli/](https://dappco.re/go/cli/)

CoreCLI is the CLI runtime for every Core binary. It builds on core/go (`dappco.re/go`) — the Core standard library — and keeps its core path free of external dependencies.

**Official Description:** CLI scaffolding — Cobra-equivalent built on core/go primitives.

---

## 🎯 Overview

From the [CoreCLI Migration RFC](file:///Users/snider/Code/meowmix/plans/code/core/cli/RFC.core-go-migration.md):

**CoreCLI** is the CLI runtime for every Core binary. It builds on core/go (`dappco.re/go`) — the Core standard library — and keeps its core path free of external dependencies: every external import is unaudited attack surface against the SASE/HIPAA compliance story, so the library carries none beyond a thin terminal layer.

### Dependencies

The cli library imports:
- `core/go` — The Core standard library
- `golang.org/x/term` — TTY detection
- `golang.org/x/sys` — System utilities
- `mattn/go-runewidth` — Display-width calculation (self-contained module)

**No CLI framework** and **no styling library** on the dependency list.

ANSI styling and stripping are a zero-dependency implementation in `pkg/cli/ansi.go`. The Frame/TUI system lives in the opt-in `pkg/cli/frame` sub-module, which has its own `go.mod` so its dependencies never reach consumers that need only output and command registration.

### The Capability Map and Its Surfaces

The core/go Action registry is the **single capability map**: `c.Action("git.log", handler)` registers a named, entitlement-gated action with a description and an input schema.

The CLI, the REST API, and the MCP server are **projections** of this one map.

`cli.MountActions` projects the registry onto the command tree: every registered action is a command at the path derived from its dotted name (`git.log` → `core git log`), invoked through `Action.Run` so entitlement and enable/disable gating apply identically across surfaces.

`core.Entitled` is the access-control layer that decides which caller and which surface may invoke which action — **one map, gated per subject and per surface**.

Help text for projected actions is **generated, not hand-authored**. A binary supplies a `cli.HelpGenerator`; cmd/core's generator composes a readable description from the action name with go-i18n's grammar engine (`demo.echo` → "Echo the demo").

### Key Statistics

- **Repository:** `forge.lthn.sh/core/cli`
- **Built on:** `dappco.re/go/cli`
- **Dependencies:** Zero external CLI frameworks (pure core/go + thin term layer)
- **Status:** Page in flight — content fills in as the package converges

### Official Install

```bash
go get dappco.re/go/cli@latest
```

### Official Import

```go
import "dappco.re/go/cli"
```

---

## 📚 Source of Truth

- **Primary Spec:** [`plans/code/core/cli/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/cli/RFC.md)
- **Migration Guide:** [`plans/code/core/cli/RFC.core-go-migration.md`](file:///Users/snider/Code/meowmix/plans/code/core/cli/RFC.core-go-migration.md)
- **Implementation:** [`core/cli/`](file:///Users/snider/Code/core/cli/)
- **Agent Guide:** [`core/cli/AGENTS.md`](file:///Users/snider/Code/core/cli/AGENTS.md)

---

## 🏗️ Architecture

## 🏗️ Architecture

### Command Registration

Commands register on a core/go `*core.Core` through `core.Command` — **path-based routing** where the directory-style path (`config/get`) is the command path.

A binary composes its surface explicitly: each command package exposes an `AddXCommands(c *core.Core) core.Result` registrar, handed to `cli.Main` through `cli.WithCommands`.

**Important:** cmd binaries are `package main` and not importable, so they wire registrars directly rather than relying on `init()` self-registration.

`core.Command` carries a `CommandAction func(core.Options) core.Result`. Options is the universal input DTO — flags and positional arguments parse into it — and Result is the universal output. Argument validation happens inside the action.

### Output and Styling

`pkg/cli` provides:
- **Semantic output:** `Success`, `Error`, `Warn`, `Info`, `Debug`
- **Structured output:** `Field`, `List`, `Table`
- **Prompts:** `Prompt`, `Confirm`, `Select`
- **Glyph system:** Unicode/emoji/ASCII themes
- **HLCRF terminal layout:** Header, Left, Content, Right, Footer regions for composite TUIs

Display width and truncation account for wide and zero-width runes.

### Internationalization

Translatable strings resolve through `cli.T`, backed by the Core i18n service with a CLI-local fallback.

Command and action descriptions are i18n keys; the grammar engine renders readable text for keys without a catalog entry. **go-i18n is grammar- and phonetics-aware**, so generated text is article-correct ("an SSH", "a go.mod").

### Validation

The CLI test suite under `tests/cli/` builds the binary from source and asserts behaviour against the compiled artifact — **one Taskfile per command surface** (`config`, `doctor`, `pkg`, `version`, …), each exercising the real binary.

Package tests follow the `_Good` / `_Bad` / `_Ugly` convention.

---

## 📊 Quick Stats

```
Total commands:         50+ across repositories
Semantic output:       JSON, YAML, Structured Text
Terminal features:    ANSI styling, prompts, TUI frames
Internationalization:  go-i18n grammar engine
Test coverage:         CLI test suite with Taskfiles
```

---

## 📖 Quick Reference

### Command Registration

| Method | Purpose | Example |
|--------|---------|---------|
| `c.Command(path, action)` | Register command | `c.Command("git.log", handler)` |
| `cli.Main(c, opts...)` | Start CLI | `cli.Main(c, cli.WithCommands(...))` |
| `cli.WithCommands(reg...)` | Add registrars | `cli.WithCommands(AddGitCommands)` |

### Semantic Output

| Function | Purpose | Example |
|----------|---------|---------|
| `cli.Success(msg)` | Success message | `cli.Success("Done")` |
| `cli.Error(err, msg)` | Error message | `cli.Error(err, "Failed")` |
| `cli.Warn(msg)` | Warning message | `cli.Warn("Caution")` |
| `cli.Info(msg)` | Info message | `cli.Info("Starting...")` |
| `cli.Debug(msg)` | Debug message | `cli.Debug("State: %v", state)` |

### Structured Output

| Function | Purpose | Example |
|----------|---------|---------|
| `cli.Field(key, value)` | Key-value pair | `cli.Field("Name", "value")` |
| `cli.List(key, items)` | List of items | `cli.List("Tags", []string{...})` |
| `cli.Table(headers)` | Table output | `cli.Table([][]string{...})` |

### Prompts

| Function | Purpose | Example |
|----------|---------|---------|
| `cli.Prompt(msg)` | Text input | `name := cli.Prompt("Name:")` |
| `cli.Confirm(msg)` | Yes/No | `ok := cli.Confirm("Sure?")` |
| `cli.Select(msg, opts)` | Choice | `choice := cli.Select("Pick:", opts)` |

---

## 📦 Core Components

### 1. Command Structure

**Cobra-based command tree:**

```go
// Root command
var rootCmd = &cobra.Command{
    Use:   "app",
    Short: "Application description",
    Long:  "Detailed application description",
}

// Subcommands
var serveCmd = &cobra.Command{
    Use:   "serve",
    Short: "Start the server",
    RunE:  serveHandler,
}

func serveHandler(cmd *cobra.Command, args []string) error {
    // Semantic output
    return core.ResultFromError(serve()).Log().Error()
}
```

### 2. Semantic Output

**Always structured:**

```json
{
  "status": "success",
  "result": {
    "message": "Server started",
    "address": "localhost:8080"
  }
}

{
  "status": "error",
  "error": {
    "code": "PORT_IN_USE",
    "message": "Port 8080 already in use"
  }
}
```

### 3. AI-Native Patterns

**Natural language support:**

```bash
# Traditional
app serve --port 8080 --host 0.0.0.0

# Natural language (via agent)
app serve on port 8080 on all interfaces
```

---

## 💻 How-Tos

### 1. Registering a Command

```go
// In your command package
func AddGitCommands(c *core.Core) core.Result {
    c.Command("git.log", func(opts core.Options) core.Result {
        dir := opts.String("dir")
        // Parse additional flags and args from opts
        return c.Process().RunIn(ctx, dir, "git", "log")
    })
    return core.Ok(nil)
}

// In main.go
func main() {
    c := core.New()
    cli.Main(c,
        cli.WithCommands(AddGitCommands),
        cli.WithCommands(AddConfigCommands),
    )
}
```

### 2. Using Semantic Output

```go
// Success output
cli.Success("Operation completed successfully")

// Error output
cli.Error(err, "Failed to complete operation")

// Warning output
cli.Warn("This action may have side effects")

// Structured output
cli.Field("Name", "value")
cli.List("Items", []string{"a", "b", "c"})
cli.Table([][]string{
    {"Name", "Status"},
    {"Service1", "Running"},
    {"Service2", "Stopped"},
})
```

### 3. Adding Internationalization

```go
// In your command
desc := cli.T("git.log.desc")  // Translated via i18n

// With fallback
msg := cli.T("some.key", "Default message if key not found")
```

### 4. Creating Prompts

```go
// Simple prompt
name := cli.Prompt("Enter your name")

// Confirmation
ok := cli.Confirm("Are you sure?")

// Selection
choice := cli.Select("Choose an option", []string{"Option 1", "Option 2"})
```

### 5. Creating TUI Frames

```go
// Create a frame with HLCRF layout
frame := cli.NewFrame()
frame.Header().Write("My Application")
frame.Content().Write("Main content here")
frame.Footer().Write("Press q to quit")
frame.Render()
```

---

## 🚀 Getting Started

### For Agents

1. **Read the RFC:** [`plans/code/core/cli/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/cli/RFC.md)
2. **Check migration guide:** [`RFC.core-go-migration.md`](file:///Users/snider/Code/meowmix/plans/code/core/cli/RFC.core-go-migration.md)
3. **Explore code:** `ls core/cli/`

### For Developers

1. **Clone the repo:** `git clone forge.lthn.sh/core/cli`
2. **Add to your app:** Import `dappco.re/core/cli`
3. **Create commands:** Use `corecli.Command()` wrapper
4. **Semantic output:** Always use `core.Result`

---

## 📊 Quick Stats

```
Total CLI repos:         5+ (go, gui, agent, api, etc.)
Total commands:         50+
Output formats:         JSON, YAML, Structured Text
Framework:              Cobra + Semantic Wrapper
```

---

## 🎯 Use Cases

### When to Use CoreCLI

✅ **Command-line tools** — Use for CLI applications
✅ **Multi-repo commands** — Use for cross-repository operations
✅ **Semantic output** — Use for agent-readable output
✅ **AI-native CLI** — Use for natural language command support

### When NOT to Use CoreCLI

❌ **GUI applications** — Use CoreGUI instead
❌ **Backend services** — Use CoreGo instead
❌ **Simple scripts** — Use shell scripts instead

---

## 🎯 Use Cases

### When to Use CoreCLI

✅ **Command-line tools** — Use for any CLI application
✅ **Multi-repo dispatch** — Use for commands that span multiple repositories
✅ **Semantic output** — When you need structured, machine-readable output
✅ **Internationalized CLIs** — When you need multi-language support
✅ **TUI applications** — When you need terminal-based user interfaces
✅ **Provider routing** — When commands need to route to different providers

### When NOT to Use CoreCLI

❌ **GUI applications** — Use CoreGUI for desktop apps
❌ **Web applications** — Use CoreTS for frontend
❌ **Simple scripts** — Use plain Go for one-off scripts

---

## 💡 Agent Tips

### Core Principles

1. **Zero external dependencies** — CoreCLI has no CLI framework dependencies
2. **Path-based routing** — Command paths derive from action names (`git.log` → `core git log`)
3. **Universal types** — Commands use `core.Options` for input and `core.Result` for output
4. **Generated help** — Help text is generated from action names via go-i18n
5. **One capability map** — CLI, REST API, and MCP server all project from the same action registry

### AX-Specific Guidance

6. **Explicit registration** — Commands are registered explicitly, not via `init()`
7. **Semantic output** — Use `cli.Success`, `cli.Error`, etc. for consistent output
8. **Internationalized** — All user-facing text should use `cli.T()`
9. **Test with Taskfiles** — CLI tests build the binary and test the compiled artifact
10. **Follow Good/Bad/Ugly** — Package tests follow the triplet pattern

### Learning Resources

- **[CoreCLI Migration RFC](file:///Users/snider/Code/meowmix/plans/code/core/cli/RFC.core-go-migration.md)** — Primary specification
- **[Official Site](https://dappco.re/go/cli/)** — Official documentation
- **[go-i18n RFC](file:///Users/snider/Code/meowmix/plans/code/core/go-i18n/RFC.md)** — Internationalization details

---

## 🔗 Related Knowledge Packs

- [CoreGo](../corego/README.md) — Core Go framework
- [CoreGUI](../coregui/README.md) — GUI framework
- [CoreTS](../corets/README.md) — TypeScript framework
- [CorePlay](../coreplay/README.md) — Play framework
- [CorePHP](../corephp/README.md) — PHP framework

---

## 💡 Agent Tips

1. **Use semantic output** — Always return structured data
2. **Natural language** — Support intent-based commands
3. **Error codes** — Use consistent error codes
4. **Core.Result** — Always wrap output
5. **Cobra patterns** — Follow standard Cobra conventions

---

## 📝 Maintenance

This knowledge pack is maintained by Mistral Vibe. Updates triggered by:

- Changes to `plans/code/core/cli/`
- Changes to `core/cli/` repository
- New CLI patterns
- Cobra updates

---

*Knowledge Pack v1.0.0*
*Created: 2026-06-17T14:40:00Z*
*Author: Mistral Vibe*
*Source: Lethean CoreCLI Framework*
