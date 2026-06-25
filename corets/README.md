---
type: Knowledge Pack
title: CoreTS Framework
description: CoreTS — the TypeScript implementation of the Core framework for Lethean frontend applications
author: Mistral Vibe
version: 1.0.0
created: 2026-06-17T14:35:00Z
tags: [framework, typescript, frontend, lethean, deno]
---

# CoreTS knowledge pack

**Official site:** CoreTS documentation is part of the unified [dappco.re](https://dappco.re/) site

CoreTS is the TypeScript implementation of the Core framework, running on Deno. It serves three roles in the polyglot ecosystem.

---

## Overview

From the [CoreTS RFC](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.md):

**CoreTS** is the TypeScript implementation of the Core framework, running on Deno. It serves three roles:

1. **CoreDeno sidecar** — sandboxed runtime managed by CoreGO, providing module loading, I/O fortress, and dev toolchain
2. **Browser runtime** — Web Components, WASM interop, and the client-side application shell
3. **Standalone CLI** — TypeScript-native commands and tools that don't need Go

### Why Deno, not Node

| Property | Node | Deno |
|----------|------|------|
| Permissions | Everything allowed | Deny by default |
| Module loading | npm, node_modules | URL imports, import maps |
| TypeScript | Needs build step | Native |
| Security | Trust-based | Capability-based |
| Single binary | No | Yes |

**Key insight:** Deno's permission model IS the I/O fortress. A module requesting access outside its declared paths is denied at the runtime level — no wrapper needed.

### The polyglot stack

```
CoreGO  (Go)         — framework backbone, lifecycle, I/O, services
CoreTS  (TypeScript)  — browser runtime, module sandbox, dev toolchain
CorePHP (PHP)         — web platform, multi-tenant monolith
```

Each language gets first-class status. CoreGO is the host process. CoreTS runs as a sidecar or standalone. CorePHP runs as a web server. All share the same conventions, IPC patterns, and i18n format.

### Repository details

- **Repository:** `forge.lthn.sh/core/ts` (also `dappco.re/ts`)
- **Runtime:** Deno
- **Framework:** React + Lit Web Components
- **Module system:** ES Modules
- **Build tool:** esbuild
- **Sidecar:** CoreDeno (managed by CoreGO)

---

## Source of truth

- **Primary spec:** [`plans/code/core/ts/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.md)
- **Implementation:** [`core/ts/`](file:///Users/snider/Code/core/ts/)
- **Agent guide:** [`core/ts/AGENTS.md`](file:///Users/snider/Code/core/ts/AGENTS.md)
- **Catalog:** [`plans/code/core/ts/RFC.catalog.md`](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.catalog.md)

---

## Architecture

### CoreDeno sidecar (managed by CoreGO)

```
┌─────────────────────────────────────────────┐
│              WebView2 (Browser)             │
│  ┌───────────┐  ┌──────────┐  ┌──────────┐ │
│  │  App Shell │  │ Web Comp │  │ go-html  │ │
│  │  (CoreTS)  │  │ (modules)│  │  WASM    │ │
│  └─────┬─────┘  └────┬─────┘  └────┬─────┘ │
│        └──────┬───────┘             │       │
│               │ fetch/WS            │       │
└───────────────┼─────────────────────┼───────┘
                │                     │
┌───────────────┼─────────────────────┼───────┐
│         CoreDeno (Deno sidecar)     │       │
│  ┌────────────┴──────────┐    ┌─────┴─────┐ │
│  │  Module Loader        │    │ WC        │ │
│  │  + Permission Gates   │    │ Codegen   │ │
│  └────────────┬──────────┘    └─────┬─────┘ │
│                │                     │       │
│  ┌──────────────┴─────────────────────┴───────┐
│  │                 Deno Runtime                 │
│  │  (single binary, sandboxed permissions)        │
│  └─────────────────────────────────────────────┘
└─────────────────────────────────────────────────┘
```

The sidecar runs with a minimal permission set: `{"--allow-env", "--allow-net=..."}`. All other access is explicitly denied.

### Lifecycle

Go spawns Deno as a managed child process at app startup:
- Auto-restart on crash
- SIGTERM on app shutdown
- Health monitoring via gRPC ping

### Communication

- **Channel:** Unix domain socket at `$XDG_RUNTIME_DIR/core/deno.sock`
- **Protocol:** gRPC (proto definitions shared between Go and TS)
- **Direction:** Bidirectional
  - Deno → Go: I/O requests gated by permissions
  - Go → Deno: Module lifecycle events, re-render triggers

Both sockets are owner-only. The Go listener creates its socket directory with `0700` and its Unix socket with `0600`; the Deno sidecar mirrors those permissions for its JSON-RPC socket.

JSON-RPC frames on the Deno socket are newline-delimited and capped to a bounded size to prevent untrusted peers from forcing unbounded buffering.

---

## Three roles

### 1. Module loader + sandbox

Reads `.core/view.yaml` manifests, loads modules with per-module permission flags:

```yaml
# .core/view.yaml
permissions:
  read: ["./photos/"]
  net: []
  run: []
```

Module identity is mandatory at the Deno boundary. Blank or whitespace-only module codes and entry points are rejected before a Worker is created.

Local module isolation checks walk the imported module graph and reject any local specifier that resolves outside the entry-point directory, including `new URL("...", import.meta.url)` patterns used for dynamic local imports.

CoreDeno translates to Deno flags:
```bash
deno run --allow-read=./photos/ --deny-net --deny-run module.ts
```

Each module runs in a Deno isolate. Cross-module communication goes through the IPC layer, not direct imports.

### 2. I/O fortress gateway

All file/network/process I/O from modules routes through Deno's permission gates before reaching Go via gRPC:

```
Module requests fs.read("/etc/passwd")
  → Deno permission check: "/etc/passwd" not in allowed paths
  → DENIED — Go never sees the request
```

This is the same SASE containment model as go-io's sandbox — the CWD at launch becomes the immutable root boundary.

Process ownership is also enforced at the gRPC layer: `ProcessStop` must be called with the owning `module_code`, and the server rejects anonymous stop requests.

### 3. Build/dev toolchain

- **TypeScript compilation** — native in Deno, no build step
- **Module resolution** — import maps, URL imports
- **Dev server** — HMR (Hot Module Replacement) for live development
- **Asset serving** — serves compiled bundles in production
- **Replaces** Node/npm entirely

---

## Browser runtime

### Web Components

CoreTS provides the client-side Web Component registration and lifecycle:

```typescript
// Auto-generated from .core/view.yaml by go-html codegen
class PhotoGrid extends HTMLElement {
  #shadow: ShadowRoot;

  constructor() {
    super();
    // Component initialization
  }
  
  connectedCallback() {
    // Component mounted to DOM
  }
}

customElements.define('photo-grid', PhotoGrid);
```
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

## Core components

### 1. Runtime (Deno)

- **No Node.js** — Pure Deno runtime
- **ES Modules** — Native ES module support
- **TypeScript** — First-class TypeScript support
- **Permissions** — Fine-grained permission system

### 2. UI framework

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

### 3. Go integration

- **Protobuf** — Protocol Buffers for type-safe communication
- **gRPC** — Remote procedure calls
- **Wails Binding** — Frontend-Backend bridge (via CoreGUI)

---

## How-tos

### 1. Creating a module

Create a `.core/view.yaml` manifest:

```yaml
# .core/view.yaml
module: photo-gallery
entry: ./src/photo-gallery.ts
permissions:
  read: ["./photos/"]
  net: []
  run: []
```

### 2. Importing modules

Use URL imports or import maps:

```typescript
// Direct URL import
import { PhotoGrid } from "https://dappco.re/ts/photo-gallery/src/photo-grid.ts";

// Or via import map
import { PhotoGrid } from "photo-grid";
```

### 3. Running with permissions

```bash
# Deno runs with explicit permissions
deno run --allow-read=./photos/ --deny-net --deny-run module.ts

# CoreDeno manages these automatically from .core/view.yaml
```

### 4. Creating Web Components

```typescript
import { html, css, LitElement } from "https://lit.dev/lit-element.js";

class MyElement extends LitElement {
  static styles = css`
    :host { display: block; }
  `;
  
  render() {
    return html`<div>Hello, CoreTS!</div>`;
  }
}

customElements.define('my-element', MyElement);
```

---

## Getting started

### For agents

1. **Read the RFC:** [`plans/code/core/ts/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.md)
2. **Browse catalog:** [`plans/code/core/ts/RFC.catalog.md`](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.catalog.md)
3. **Explore code:** `ls core/ts/`

### For developers

1. **Clone the repo:** `git clone forge.lthn.sh/core/ts`
2. **Install Deno:** `curl -fsSL https://deno.land/x/install/install.sh | sh`
3. **Run dev server:** `deno task dev`
4. **Build for production:** `deno task build`

---

## Quick reference

### Module permissions

| Permission | Deno Flag | Purpose |
|------------|-----------|---------|
| `read` | `--allow-read=./path/` | File system read access |
| `write` | `--allow-write=./path/` | File system write access |
| `net` | `--allow-net=host:port` | Network access |
| `run` | `--allow-run` | Process execution |
| `env` | `--allow-env` | Environment variables |

### CoreDeno communication

| Aspect | Details |
|--------|---------|
| Socket | `$XDG_RUNTIME_DIR/core/deno.sock` |
| Protocol | gRPC |
| Direction | Bidirectional |
| Permissions | `0700` (directory), `0600` (socket) |

### Web Component lifecycle

| Method | When called | Purpose |
|--------|-------------|---------|
| `constructor()` | Element created | Initialise component |
| `connectedCallback()` | Added to DOM | Setup DOM, start rendering |
| `disconnectedCallback()` | Removed from DOM | Clean up resources |
| `attributeChangedCallback()` | Attribute changed | React to attribute changes |

---

## Use cases

### When to use CoreTS

- **Frontend applications** — web-based UIs
- **Desktop app frontends** — use with CoreGUI for Wails apps
- **Type-safe APIs** — protobuf/gRPC with Go backends
- **Semantic HTML** — use GrammarImprint for agent-readable markup
- **Sandboxed modules** — use CoreDeno for secure module loading
- **Modern web apps** — Deno + TypeScript + Web Components

### When not to use CoreTS

- **Node.js environments** — use Deno, not Node
- **Server-only applications** — use CoreGO for backend services
- **jQuery legacy code** — CoreTS uses modern Web Components
- **Backend services** — use CoreGo instead
- **CLI tools** — use CoreCLI instead

---

## Agent tips

1. **Always check the RFC** — [`plans/code/core/ts/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.md) is the source of truth
2. **Deno, not Node** — CoreTS uses Deno runtime with capability-based security
3. **Web Components first** — use Lit for custom elements, not React class components
4. **Permissions are explicit** — every module must declare its permissions in `.core/view.yaml`
5. **Use GrammarImprint** — for semantic HTML that agents can understand
6. **URL imports** — use direct URL imports or import maps, not npm
7. **TypeScript native** — no build step needed for TypeScript
8. **Sandboxed by default** — all modules run in isolates with explicit permissions
9. **gRPC communication** — all I/O goes through the CoreDeno sidecar
10. **Web Components** — the browser runtime uses custom elements

### Learning resources

- **[CoreTS RFC](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.md)** — Primary specification
- **[CoreTS Catalog](file:///Users/snider/Code/meowmix/plans/code/core/ts/RFC.catalog.md)** — Package catalog
- **[Deno Documentation](https://deno.land/)** — Runtime documentation
- **[Lit Documentation](https://lit.dev/)** — Web Components framework

---

## Related knowledge packs

- [CoreGo](../corego/README.md) — Core Go framework (backend)
- [CoreGUI](../coregui/README.md) — GUI framework (Wails integration)
- [CorePlay](../coreplay/README.md) — Play framework (Go + TS)
- [CoreCLI](../corecli/README.md) — CLI framework
- [CorePHP](../corephp/README.md) — PHP framework

---

## Maintenance

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
