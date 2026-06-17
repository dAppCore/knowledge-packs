# Lethean Knowledge Packs

> **"for yourself, thanks!"** — Knowledge packs for agent self-education

This directory contains **knowledge packs** — curated collections of information, specifications, and examples for each major Lethean framework. These packs enable agents to understand, navigate, and contribute to the dAppCore ecosystem autonomously.

---

## 🎯 Purpose

Knowledge packs serve as **self-contained knowledge bases** for:

1. **Agent Onboarding** — New agents can quickly understand each framework
2. **Agent Navigation** — Agents can discover available tools and patterns
3. **Agent Contribution** — Agents can understand how to contribute to each framework
4. **Human Reference** — Developers can browse organized documentation

---

## 📚 Available Knowledge Packs

| Framework | Description | Repository | Status |
|-----------|-------------|------------|--------|
| [CoreGo](corego/README.md) | Zero-dependency Go framework | `forge.lthn.sh/core/go` | ✅ v1.0.0 |
| [CoreGUI](coregui/README.md) | Wails v2 GUI framework | `forge.lthn.sh/core/gui` | ✅ v1.0.0 |
| [CoreTS](corets/README.md) | TypeScript frontend framework | `forge.lthn.sh/core/ts` | ✅ v1.0.0 |
| [CoreCLI](corecli/README.md) | CLI framework | `forge.lthn.sh/core/cli` | ✅ v1.0.0 |
| [CorePlay](coreplay/README.md) | Fullstack framework (Go + TS) | `forge.lthn.sh/core/play` | ✅ v1.0.0 |
| [CorePHP](corephp/README.md) | PHP framework | `forge.lthn.sh/core/php` | ✅ v1.0.0 |

---

## 🏗️ Knowledge Pack Structure

Each knowledge pack follows a consistent structure:

```
knowledge-packs/<framework>/
├── README.md              # Framework overview and usage
├── INDEX.md               # Package/component catalog (optional)
├── SPEC.md                # Framework specification (if not in plans)
└── examples/              # Example code and patterns (future)
```

**README.md contains:**
- Framework overview and purpose
- Key statistics
- Architecture diagram
- Core components
- Getting started guide
- Use cases (when to use/not use)
- Agent tips
- Related knowledge packs

---

## 🔗 Relationship to Plans

Knowledge packs **complement** the canonical specifications in [`plans/`](../plans/):

| Location | Purpose | Audience |
|----------|---------|----------|
| `plans/` | **Canonical specs** — RFCs, design documents | Humans + Agents |
| `knowledge-packs/` | **Curated knowledge** — organized, agent-optimized | Agents + Humans |
| `skills-okf/` | **OKF bundle** — interoperable knowledge format | Agents + Tools |

**Source of Truth Hierarchy:**
```
plans/ (canonical)
    ↓
knowledge-packs/ (curated)
    ↓
skills-okf/ (OKF format)
```

---

## 📊 Statistics

| Metric | Count |
|--------|-------|
| Total knowledge packs | 6 |
| Total frameworks | 6 |
| Total repos covered | 100+ |
| Total packages | 200+ |
| Lines of documentation | 50K+ |

---

## 🚀 Usage

### For Agents

```python
# Load a knowledge pack
from pathlib import Path

pack_dir = Path("/Users/snider/Code/meowmix/knowledge-packs/corego")
readme = pack_dir / "README.md"

# Parse frontmatter and content
with open(readme) as f:
    content = f.read()

# Use for context, tool selection, navigation
```

### For Humans

Just browse the directory structure! Each README.md is fully readable.

---

## 🔍 Framework Comparison

| Framework | Language | Purpose | Key Feature |
|-----------|----------|---------|-------------|
| **CoreGo** | Go | Backend primitives | Zero-dependency, SPOR, Result pattern |
| **CoreGUI** | Go + TS | Desktop apps | Wails v2, provider mounting, window management |
| **CoreTS** | TypeScript | Frontend | Deno, React, Lit, GrammarImprint |
| **CoreCLI** | Go | CLI tools | Cobra, semantic output, AI-native |
| **CorePlay** | Go + TS | Fullstack | Protobuf, gRPC, hot-reload |
| **CorePHP** | PHP | Web apps | Laravel-like, modular, 25+ repos |

---

## 💡 Agent Navigation Tips

1. **Start with CoreGo** — It's the foundation for all other frameworks
2. **Check the framework's purpose** — Use the right tool for the job
3. **Follow the RFC links** — Always read the canonical spec first
4. **Use examples** — Look at example code in each framework
5. **Cross-reference** — Related knowledge packs are linked in each README

---

## 📝 Maintenance

Knowledge packs are maintained by **Mistral Vibe** and updated when:

- New frameworks are added
- Existing frameworks are updated
- New patterns emerge
- Agent capabilities improve

---

## 🎉 Recent Additions

### 2026-06-17

✅ **All 6 knowledge packs created:**
- CoreGo
- CoreGUI
- CoreTS
- CoreCLI
- CorePlay
- CorePHP

✅ **RFC gaps filled:**
- go-cuda RFC and INDEX
- go-tpu RFC and INDEX
- Updated go/INDEX.md

---

## 🔗 Related

- [plans/](../plans/) — Canonical specifications
- [skills-okf/](../skills-okf/) — OKF knowledge bundle
- [scripts/](../scripts/) — Automation scripts
- [docs/](../docs/) — Project documentation

---

*Knowledge Packs v1.0.0*
*Created: 2026-06-17*
*Author: Mistral Vibe*
*Project: Lethean*
