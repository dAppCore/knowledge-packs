---
type: Knowledge Pack
title: CorePlay Framework
description: Complete knowledge pack for CorePlay — the Play framework combining Go backend with TypeScript frontend
author: Mistral Vibe
version: 1.0.0
created: 2026-06-17T14:45:00Z
tags: [framework, play, golang, typescript, lethean, fullstack]
---

# CorePlay Knowledge Pack

> **"Go backend + TypeScript frontend = Fullstack application"**

CorePlay is the **Play framework** for Lethean, combining CoreGo backend with CoreTS frontend into a fullstack application framework.

---

## 🎯 Overview

**CorePlay** provides:
- Go backend (CoreGo)
- TypeScript frontend (CoreTS)
- Protobuf/gRPC communication
- Hot-reload development
- Production build pipeline

### Key Statistics

- **Repository:** `forge.lthn.sh/core/play`
- **Backend:** CoreGo (Go 1.22+)
- **Frontend:** CoreTS (Deno + TypeScript)
- **Communication:** Protobuf + gRPC
- **Build:** Multi-stage Docker builds

---

## 📚 Source of Truth

- **Primary Spec:** [`plans/code/core/play/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.md)
- **Catalog:** [`plans/code/core/play/RFC.catalog.md`](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.catalog.md)
- **Kingdoms:** [`plans/code/core/play/RFC.kingdoms.md`](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.kingdoms.md)
- **Implementation:** [`core/play/`](file:///Users/snider/Code/core/play/)
- **Agent Guide:** [`core/play/AGENTS.md`](file:///Users/snider/Code/core/play/AGENTS.md)

---

## 🏗️ Architecture

```
CorePlay (Fullstack Application)
├── Backend (Go)
│   ├── CoreGo Framework
│   ├── Protobuf Definitions
│   ├── gRPC Server
│   └── REST API (optional)
├── Frontend (TypeScript)
│   ├── CoreTS Framework
│   ├── Protobuf Client
│   ├── gRPC Client
│   └── React/Lit Components
├── Communication
│   ├── Protobuf (Type-safe)
│   └── gRPC (Efficient)
└── Build System
    ├── Development (Hot-reload)
    └── Production (Optimized)
```

---

## 📦 Core Components

### 1. Backend (Go)

**CoreGo + Play extensions:**

```go
// Protobuf service definition
service Greeter {
    rpc SayHello (HelloRequest) returns (HelloReply);
}

// gRPC server
func (s *server) SayHello(ctx context.Context, req *HelloRequest) (*HelloReply, error) {
    return &HelloReply{Message: "Hello " + req.Name}, nil
}

// REST gateway (optional)
func handleHello(w http.ResponseWriter, r *http.Request) {
    // Convert REST → gRPC
}
```

### 2. Frontend (TypeScript)

**CoreTS + Play extensions:**

```typescript
// Protobuf client
import { GreeterClient } from "./gen/proto";

const client = new GreeterClient("http://localhost:50051");
const response = await client.sayHello({ name: "World" });
console.log(response.message);
```

### 3. Communication Layer

**Protobuf + gRPC:**

- **Type-safe** — Generated from .proto files
- **Efficient** — Binary encoding
- **Bidirectional** — Streaming support
- **Cross-language** — Go ↔ TypeScript

---

## 🚀 Getting Started

### For Agents

1. **Read the RFC:** [`plans/code/core/play/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.md)
2. **Check catalog:** [`RFC.catalog.md`](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.catalog.md)
3. **Explore kingdoms:** [`RFC.kingdoms.md`](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.kingdoms.md)
4. **Browse code:** `ls core/play/`

### For Developers

1. **Clone the repo:** `git clone forge.lthn.sh/core/play`
2. **Define protobuf:** Create `.proto` files
3. **Generate code:** `go generate ./...` + `deno task generate`
4. **Run dev:** `task dev` (hot-reload both sides)
5. **Build prod:** `task build`

---

## 📊 Quick Stats

```
Backend files:         50+
Frontend files:        30+
Protobuf definitions:  10+
gRPC services:         10+
Build stages:          Multi-stage Docker
```

---

## 🎯 Use Cases

### When to Use CorePlay

✅ **Fullstack applications** — Go backend + TS frontend
✅ **Type-safe APIs** — Protobuf/gRPC communication
✅ **Hot-reload development** — Fast iteration
✅ **Production builds** — Optimized Docker images

### When NOT to Use CorePlay

❌ **Backend-only services** — Use CoreGo instead
❌ **Frontend-only apps** — Use CoreTS instead
❌ **Simple CLI tools** — Use CoreCLI instead
❌ **PHP applications** — Use CorePHP instead

---

## 🔗 Related Knowledge Packs

- [CoreGo](../corego/README.md) — Backend framework
- [CoreTS](../corets/README.md) — Frontend framework
- [CoreGUI](../coregui/README.md) — GUI framework
- [CoreCLI](../corecli/README.md) — CLI framework
- [CorePHP](../corephp/README.md) — PHP framework

---

## 💡 Agent Tips

1. **Protobuf first** — Always define API with protobuf
2. **Type-safe** — Never use raw JSON APIs
3. **gRPC for internal** — Use gRPC between Go and TS
4. **REST for external** — Use REST gateway for public APIs
5. **Hot-reload** — Use `task dev` for development

---

## 📝 Maintenance

This knowledge pack is maintained by Mistral Vibe. Updates triggered by:

- Changes to `plans/code/core/play/`
- Changes to `core/play/` repository
- New protobuf patterns
- gRPC updates

---

*Knowledge Pack v1.0.0*
*Created: 2026-06-17T14:45:00Z*
*Author: Mistral Vibe*
*Source: Lethean CorePlay Framework*
