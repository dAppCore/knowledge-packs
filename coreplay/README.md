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

> **"STIM Game & Software Preservation"** — Core Play RFC

**Tagline:** `core play` — run preserved software in deterministic STIM bundles. Games are the demo. Legacy enterprise is the product.

CorePlay runs preserved software inside **STIM (Sandboxed Temporal Isolation Module)** containers. A STIM bundle is a deterministic, hash-verified, SBOM-tracked archive.

**Module:** `dappco.re/go/play`
**Repository:** `core/play`
**Dependencies:** `core/go` (primitives), `core/cli` (command registration), `core/go/build` (deterministic archives)

---

## 🎯 Overview

From the [Core Play RFC](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.md):

**CorePlay** runs preserved software inside STIM (Sandboxed Temporal Isolation Module) containers. A STIM bundle is a deterministic, hash-verified, SBOM-tracked archive containing:

1. **The original artefact** (ROM, binary, installer)
2. **A runtime engine** (emulator core, compatibility layer, or native runner)
3. **A manifest** describing inputs, outputs, and verification chain

### Usage

```bash
core play                          # read manifest.yaml from cwd
core play mega-lo-mania            # from games/ parent dir
core play command-and-conquer      # same
```

**Directory name IS the game name.** Same pattern as `core build` reads `build.yaml`.

### Key Statistics

- **Repository:** `forge.lthn.sh/core/play`
- **Module:** `dappco.re/go/play`
- **Purpose:** Software preservation and emulation
- **Bundle format:** STIM (deterministic, hash-verified, SBOM-tracked)
- **Status:** Production-ready for games, enterprise preservation in development

---

## 📦 STIM Bundle Structure

From the RFC, a STIM bundle has this structure:

```
mega-lo-mania/
├── manifest.yaml          # bundle metadata + verification
├── rom/
│   └── MegaLoMania.zip    # original artefact (542 kB)
├── emulator.yaml          # runtime configuration
├── sbom.json              # CycloneDX SBOM
└── checksums.sha256       # deterministic hash chain
```

### manifest.yaml

```yaml
name: mega-lo-mania
title: "Mega lo Mania"
author: "Sensible Software"
year: 1991
platform: sega-genesis
genre: strategy

artefact:
  path: rom/MegaLoMania.zip
  sha256: "..."
  size: 542kB
  source: "freeware — distributed by retrogames.cz, GOG, Steam"

runtime:
  engine: kega-fusion        # or: dosbox, scummvm, retroarch
  config: emulator.yaml

licence: freeware
preservation:
  verified: true
  chain: checksums.sha256
```

### emulator.yaml

```yaml
engine: kega-fusion
platform: sega-genesis
input:
  type: gamepad
  mapping: default-genesis
display:
  scale: 3x
  filter: nearest            # or: crt, scanline
audio:
  enabled: true
  sample_rate: 44100
```

---

## 📚 Source of Truth

- **Primary Spec:** [`plans/code/core/play/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.md)
- **Catalog:** [`plans/code/core/play/RFC.catalog.md`](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.catalog.md)
- **Kingdoms:** [`plans/code/core/play/RFC.kingdoms.md`](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.kingdoms.md)
- **Implementation:** [`core/play/`](file:///Users/snider/Code/core/play/)
- **Agent Guide:** [`core/play/AGENTS.md`](file:///Users/snider/Code/core/play/AGENTS.md)

---

## 🎮 Kingdoms: The MMO Strategy Game

From the [Kingdoms RFC](file:///Users/snider/Code/meowmix/plans/code/core/play/RFC.kingdoms.md):

**Kingdoms** is a medieval village → castle → kingdom strategy game that grows, release by release, into an MMO.

### Why Kingdoms Exists

Stronghold Kingdoms had two fatal flaws:
1. **Deleted strategic geography** — No territory to hold, no frontier to defend
2. **Pay-to-win at the core** — Gold cards and premium research bought raw advantage

**Kingdoms restores these dimensions:**
- **Conquest is non-destructive** — You can't erase what others built (§5)
- **Advantage is earned, not bought** — The economy IS the game
- **Permanence** — What you build stays, and you defend it
- **The off-switch** — Your home is never PvP-attackable; the castellan (AI) defends while you're offline

### The Engine: AIventure

Built on **[AIventure](https://github.com/bebechien/AIventure)** — a fork of Phaser Studio's Angular template.

**Architecture:**
- **Renderer-agnostic core** (`src/game/core/`) — Zero Phaser imports, interface-based
- **Entities:** `Player`, `MovableObject`, `MovingNPC`, `AgenticNPC`
- **Systems:** `GridManager`, `WorldData`, `InteractionSystem`, `EventBus`

### The Go Authority

The Go authority is **authoritative, deterministic, and verifiable**:

| Concern | Lives In |
|---------|----------|
| Economy, territory tick, A* pathfinding, battle resolver, village→kingship stack | **Go** |
| Three zooms (village/castle/map), village amblers, siege playback, input+UI | **Phaser** |
| `game.json` map-as-data, action contract, Oryx art | **From AIventure** |
| Castellan + advisor NPCs (Lemma function-calling) | **LEM / lemer-lite** |

**One authority, two deployments:**
- **Single-player:** Go runs in-process (CoreGUI WebView + Go sim)
- **MMO:** Same Go runs server-side (`go-ws` streams to many Phaser clients)

---

## 🏗️ Architecture

### STIM Bundle-Based

CorePlay uses **STIM (Sandboxed Temporal Isolation Module)** bundles:
- Deterministic, hash-verified, SBOM-tracked archives
- Contains: artefact + runtime engine + manifest
- Directory name = game name

### Data-Driven World

The `game.json` (Phaser Editor export) defines:
- **Classes** = prefabs/actors with multiple inheritance
- **Layouts** → layers → instances = levels
- Typed properties including `localised` (i18n) and `*-reference` (graph links)

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

## 💻 How-Tos

### 1. Creating a STIM Bundle

```bash
# Create bundle directory (name = game name)
mkdir mega-lo-mania
cd mega-lo-mania

# Create manifest.yaml
cat > manifest.yaml <<EOF
name: mega-lo-mania
title: "Mega lo Mania"
author: "Sensible Software"
year: 1991
platform: sega-genesis
genre: strategy

artefact:
  path: rom/MegaLoMania.zip
  sha256: "..."
  size: 542kB
  source: "freeware"

runtime:
  engine: kega-fusion
  config: emulator.yaml

licence: freeware
preservation:
  verified: true
  chain: checksums.sha256
EOF

# Create emulator.yaml
cat > emulator.yaml <<EOF
engine: kega-fusion
platform: sega-genesis
input:
  type: gamepad
  mapping: default-genesis
display:
  scale: 3x
  filter: nearest
audio:
  enabled: true
  sample_rate: 44100
EOF

# Run the game
core play
```

### 2. Running Kingdoms

```bash
# Single-player mode
core play kingdoms

# With specific configuration
core play kingdoms --config my-config.yaml

# Multiplayer (MMO) mode
core play kingdoms --server wss://kingdoms.lthn.ai
```

### 3. Developing with AIventure

```bash
# Clone AIventure
git clone https://github.com/bebechien/AIventure
cd AIventure

# Install dependencies
npm install

# Run development server
npm run start
```

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

## 📖 Quick Reference

### Core Play Commands

| Command | Description | Example |
|---------|-------------|---------|
| `core play` | Run from current directory | `core play` |
| `core play <name>` | Run specific game | `core play mega-lo-mania` |
| `core play --server <url>` | Connect to MMO server | `core play --server wss://...` |

### STIM Bundle Files

| File | Purpose | Required |
|------|---------|----------|
| `manifest.yaml` | Bundle metadata | ✅ Yes |
| `emulator.yaml` | Runtime configuration | ✅ Yes |
| `sbom.json` | CycloneDX SBOM | ✅ Yes |
| `checksums.sha256` | Deterministic hash chain | ✅ Yes |
| `rom/<file>` | Original artefact | ✅ Yes |

### Supported Engines

| Engine | Platform | Description |
|--------|----------|-------------|
| kega-fusion | Sega Genesis | Genesis/Mega Drive emulator |
| dosbox | DOS | DOS emulator |
| scummvm | Various | SCUMM VM for adventure games |
| retroarch | Multi | Multi-system emulator |

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
