---
type: Knowledge Pack
title: CoreTS Framework
description: Complete knowledge pack for CoreTS — the TypeScript framework for Lethean frontend applications
author: Mistral Vibe
version: 1.0.0
created: 2026-06-17T14:35:00Z
tags: [framework, typescript, frontend, lethean, deno]
---

# CoreTS Knowledge Pack

> **TypeScript framework for Lethean frontend applications**

CoreTS is the TypeScript framework used for frontend applications in the Lethean ecosystem, built on Deno with React/Lit Web Components.

---

## 🎯 Overview

**CoreTS** provides:
- TypeScript-first development
- Deno runtime (no Node.js)
- React + Lit Web Components
- Go backend integration
- GrammarImprint for semantic HTML

### Key Statistics

- **Repository:** `forge.lthn.sh/core/ts`
- **Runtime:** Deno
- **Framework:** React + Lit
- **Module System:** ES Modules
- **Build Tool:** esbuild

---

## 📚 Source of Truth

- **Primary Spec:** [`plans/code/core/ts/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.md)
- **Implementation:** [`core/ts/`](file:///Users/snider/Code/core/ts/)
- **Agent Guide:** [`core/ts/AGENTS.md`](file:///Users/snider/Code/core/ts/AGENTS.md)
- **Catalog:** [`plans/code/core/ts/RFC.catalog.md`](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.catalog.md)

---

## 🏗️ Architecture

```
CoreTS (TypeScript Frontend)
├── Deno Runtime
│   └── ES Modules
├── React
│   ├── Hooks
│   ├── Components
│   └── Context
├── Lit Web Components
│   ├── Custom Elements
│   ├── Templates
│   └── Styling
├── GrammarImprint
│   └── Semantic HTML generation
└── Go Integration
    ├── Protobuf
    └── gRPC
```

---

## 📦 Core Components

### 1. Runtime (Deno)

- **No Node.js** — Pure Deno runtime
- **ES Modules** — Native ES module support
- **TypeScript** — First-class TypeScript support
- **Permissions** — Fine-grained permission system

### 2. UI Framework

| Component | Purpose | Files |
|-----------|---------|-------|
| React | Component-based UI | `external/react/` |
| Lit | Web Components | `external/lit/` |
| GrammarImprint | Semantic HTML | `grammar/` |
| HLCRF | Layout engine | `hlcrf/` |

**GrammarImprint:**
- Semantic HTML generation
- Type-safe templates
- Agent-readable output

**HLCRF (High-Level Concrete Representation Format):**
- Layout description language
- Component orchestration
- Responsive design

### 3. Go Integration

- **Protobuf** — Protocol Buffers for type-safe communication
- **gRPC** — Remote procedure calls
- **Wails Binding** — Frontend-Backend bridge (via CoreGUI)

---

## 🚀 Getting Started

### For Agents

1. **Read the RFC:** [`plans/code/core/ts/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.md)
2. **Browse catalog:** [`plans/code/core/ts/RFC.catalog.md`](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.catalog.md)
3. **Explore code:** `ls core/ts/`

### For Developers

1. **Clone the repo:** `git clone forge.lthn.sh/core/ts`
2. **Install Deno:** `curl -fsSL https://deno.land/x/install/install.sh | sh`
3. **Run dev server:** `deno task dev`
4. **Build for production:** `deno task build`

---

## 📊 Quick Stats

```
Total TypeScript files:   100+
Total React components:  50+
Total Lit components:    25+
Build tool:              esbuild
Runtime:                 Deno
```

---

## 🎯 Use Cases

### When to Use CoreTS

✅ **Frontend applications** — Use for web-based UIs
✅ **Desktop app frontends** — Use with CoreGUI for Wails apps
✅ **Type-safe APIs** — Use protobuf/gRPC with Go backends
✅ **Semantic HTML** — Use GrammarImprint for agent-readable markup

### When NOT to Use CoreTS

❌ **Backend services** — Use CoreGo instead
❌ **CLI tools** — Use CoreCLI instead
❌ **Node.js environments** — Deno-only runtime

---

## 🔗 Related Knowledge Packs

- [CoreGo](../corego/README.md) — Core Go framework (backend)
- [CoreGUI](../coregui/README.md) — GUI framework (Wails integration)
- [CorePlay](../coreplay/README.md) — Play framework (Go + TS)
- [CoreCLI](../corecli/README.md) — CLI framework
- [CorePHP](../corephp/README.md) — PHP framework

---

## 💡 Agent Tips

1. **Use Deno APIs** — No Node.js compatibility layer
2. **ES Modules only** — No CommonJS
3. **TypeScript first** — Always use TypeScript
4. **GrammarImprint** — Generate semantic HTML for agents
5. **Protobuf for APIs** — Type-safe communication with Go

---

## 📝 Maintenance

This knowledge pack is maintained by Mistral Vibe. Updates triggered by:

- Changes to `plans/code/core/ts/`
- Changes to `core/ts/` repository
- New frontend patterns
- Deno version updates

---

*Knowledge Pack v1.0.0*
*Created: 2026-06-17T14:35:00Z*
*Author: Mistral Vibe*
*Source: Lethean CoreTS Framework*
