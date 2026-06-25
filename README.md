# Lethean knowledge packs

Curated reference material for the Lethean (dAppCore) frameworks. Each pack is a
self-contained knowledge base an agent or a developer can read to understand a
framework, find the right package, and work with it correctly.

These packs are written for agents: factual, navigable, and kept close to the
source. They complement the canonical specifications in `plans/` rather than
replacing them.

## Available packs

| Framework | Description | Repository |
|-----------|-------------|------------|
| [CoreGo](corego/README.md) | Zero-dependency Go framework | `forge.lthn.sh/core/go` |
| [CoreGUI](coregui/README.md) | Wails v2 GUI framework | `forge.lthn.sh/core/gui` |
| [CoreTS](corets/README.md) | TypeScript frontend framework | `forge.lthn.sh/core/ts` |
| [CoreCLI](corecli/README.md) | CLI framework | `forge.lthn.sh/core/cli` |
| [CorePlay](coreplay/README.md) | Fullstack framework (Go + TS) | `forge.lthn.sh/core/play` |
| [CorePHP](corephp/README.md) | PHP framework | `forge.lthn.sh/core/php` |

## Pack structure

Each pack follows a consistent layout:

```
knowledge-packs/<framework>/
├── README.md     Framework overview and usage
├── INDEX.md      Package or component catalogue (optional)
├── SPEC.md       Framework specification, where not held in plans/
└── examples/     Example code and patterns (where present)
```

A README covers the framework's purpose, architecture, core components, a
getting-started path, when to use it (and when not to), and links to related
packs.

## Relationship to plans

Knowledge packs sit between the canonical specifications and the OKF bundle.

| Location | Purpose | Audience |
|----------|---------|----------|
| `plans/` | Canonical specifications — RFCs, design documents | Humans and agents |
| `knowledge-packs/` | Curated, agent-oriented reference | Agents and humans |
| `skills-okf/` | OKF bundle — interoperable knowledge format | Agents and tools |

Source-of-truth order: `plans/` → `knowledge-packs/` → `skills-okf/`.

## Framework comparison

| Framework | Language | Purpose | Notes |
|-----------|----------|---------|-------|
| CoreGo | Go | Backend primitives | Zero-dependency, SPOR, Result pattern |
| CoreGUI | Go + TS | Desktop apps | Wails v2, provider mounting, window management |
| CoreTS | TypeScript | Frontend | Deno, React, Lit, GrammarImprint |
| CoreCLI | Go | CLI tools | Cobra, semantic output |
| CorePlay | Go + TS | Fullstack | Protobuf, gRPC, hot reload |
| CorePHP | PHP | Web apps | Laravel-like, modular, 25+ repos |

## Reading order

1. Start with CoreGo. It is the foundation the other frameworks build on.
2. Check a framework's purpose before reaching for it; use the one that fits the job.
3. Follow the RFC links to the canonical spec when you need authoritative detail.
4. Cross-reference related packs, linked at the foot of each README.

## Usage

Each README is plain Markdown — read it directly, or load it as context. Packs are
addressed by framework name under `knowledge-packs/<framework>/`.

## Maintenance

Updated when frameworks are added or change, or when patterns settle. Original
drafts by Mistral Vibe.

## Related

- `plans/` — canonical specifications
- `skills-okf/` — OKF knowledge bundle
- `scripts/` — automation scripts
