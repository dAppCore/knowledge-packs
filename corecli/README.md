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

> **"Semantic output, commands, AI-native convention"** — CoreCLI RFC

CoreCLI is the CLI framework for Lethean command-line tools, providing consistent command-line interfaces and semantic output.

---

## 🎯 Overview

**CoreCLI** provides:
- Consistent CLI conventions
- Semantic output formatting
- Command dispatch and routing
- AI-native command patterns

### Key Statistics

- **Repository:** `forge.lthn.sh/core/cli`
- **Framework:** Cobra-based (semantic wrapper)
- **Commands:** 50+ across repositories
- **Output:** Semantic JSON/structured formats

---

## 📚 Source of Truth

- **Primary Spec:** [`plans/code/core/cli/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/cli/RFC.md)
- **Migration Guide:** [`plans/code/core/cli/RFC.core-go-migration.md`](file:///Users/snider/Code/meowmix/plans/code/core/cli/RFC.core-go-migration.md)
- **Implementation:** [`core/cli/`](file:///Users/snider/Code/core/cli/)
- **Agent Guide:** [`core/cli/AGENTS.md`](file:///Users/snider/Code/core/cli/AGENTS.md)

---

## 🏗️ Architecture

```
CoreCLI (Command-Line Framework)
├── Cobra Core
│   ├── Command Tree
│   ├── Flags
│   └── Arguments
├── Semantic Output
│   ├── JSON
│   ├── YAML
│   └── Structured Text
├── AI-Native
│   ├── Intent Detection
│   ├── Natural Language
│   └── Agent Integration
└── Dispatch
    ├── Multi-Repo
    └── Provider Routing
```

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
