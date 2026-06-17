---
type: Package Index
title: go-html Package Index
description: Complete index of go-html package components and API surface
module: dappco.re/go/core/html
---

# go-html — Package Index

> **Repository:** `core/go-html`
> **Module:** `dappco.re/go/core/html`
> **Type:** Library
> **Status:** Production
> **Lines:** ~643 (source)

---

## 📚 Quick Links

- **[README.md](./README.md)** — Complete package documentation
- **[RFC Specification](file:///Users/snider/Code/meowmix/plans/code/core/go/html/RFC.md)** — Technical specification
- **[CLAUDE.md](file:///Users/snider/Code/core/go-html/CLAUDE.md)** — Implementation details
- **[AGENTS.md](file:///Users/snider/Code/core/go-html/AGENTS.md)** — Agent guidance

### Local Documentation

- [Architecture](file:///Users/snider/Code/core/go-html/docs/architecture.md) — Full internals
- [Development Guide](file:///Users/snider/Code/core/go-html/docs/development.md) — Building and testing
- [Project History](file:///Users/snider/Code/core/go-html/docs/history.md) — Phases and limitations

### Sub-Specifications

- [Models RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/html/RFC.models.md) — Data models
- [Imports RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/html/RFC.imports.md) — Import structure
- [Catalog RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/html/RFC.catalog.md) — Component catalog

---

## 🗂️ File Structure

### Source Files (Core Package)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `doc.go` | ~50 | Package documentation | ✅ Complete |
| `context.go` | ~100 | Rendering context with identity, locale, entitlements | ✅ Complete |
| `node.go` | ~150 | Node interface and concrete types (El, Text, Raw, If, Unless, Each, EachSeq, Switch, Layout, Responsive) | ✅ Complete |
| `render.go` | ~100 | Render function | ✅ Complete |
| `layout.go` | ~150 | HLCRF Layout compositor | ✅ Complete |
| `responsive.go` | ~100 | Multi-variant responsive wrapper | ✅ Complete |
| `path.go` | ~50 | Block ID parsing | ✅ Complete |
| `attr.go` | ~50 | Attribute helpers (Attr, AriaLabel, AltText, TabIndex, AutoFocus, Role) | ✅ Complete |
| `entitled.go` | ~50 | Entitled node type with deny-by-default gating | ✅ Complete |

### Server-Side Only Files (`//go:build !js`)

| File | Purpose | Status |
|------|---------|--------|
| `pipeline.go` | Grammar pipeline (StripTags, Imprint, CompareVariants) | ✅ Complete |
| `grammar.go` | GrammarImprint types and analysis | ✅ Complete |
| `shadow.go` | Shadow DOM generation | ✅ Complete |
| `text_translate.go` | Text translation (server) | ✅ Complete |
| `text_builder.go` | Text building (server) | ✅ Complete |
| `result.go` | Result types (server) | ✅ Complete |
| `service.go` | Service integration | ✅ Complete |

### WASM-Only Files (`//go:build js`)

| File | Purpose | Status |
|------|---------|--------|
| `text_translate_js.go` | Text translation (WASM) | ✅ Complete |
| `text_builder_js.go` | Text building (WASM) | ✅ Complete |
| `result_js.go` | Result types (WASM) | ✅ Complete |

### Codegen Package

| File | Purpose | Status |
|------|---------|--------|
| `codegen/codegen.go` | Main codegen logic | ✅ Complete |
| `codegen/doc.go` | Codegen documentation | ✅ Complete |
| `codegen/typescript.go` | TypeScript declaration generation | ✅ Complete |
| `codegen/shadow.go` | Shadow DOM generation | ✅ Complete |

### Command Package

| File | Purpose | Status |
|------|---------|--------|
| `cmd/codegen/main.go` | Codegen CLI entry point | ✅ Complete |
| `cmd/wasm/main.go` | WASM entry point | ✅ Complete |
| `cmd/wasm/register.go` | JS function registration | ✅ Complete |
| `cmd/wasm/render_layout.go` | Layout rendering in WASM | ✅ Complete |

### Test Files

| File | Purpose | Coverage |
|------|---------|----------|
| `context_test.go` | Context tests | ✅ Good/Bad/Ugly |
| `layout_test.go` | Layout tests | ✅ Good/Bad/Ugly |
| `node_test.go` | Node tests | ✅ Good/Bad/Ugly |
| `render_test.go` | Render tests | ✅ Good/Bad/Ugly |
| `responsive_test.go` | Responsive tests | ✅ Good/Bad/Ugly |
| `pipeline_test.go` | Pipeline tests | ✅ Good/Bad/Ugly |
| `path_test.go` | Path tests | ✅ Good/Bad/Ugly |
| `edge_test.go` | Edge case tests | ✅ Good/Bad/Ugly |

### Documentation Files

| File | Purpose |
|------|---------|
| `docs/architecture.md` | Full architecture documentation |
| `docs/development.md` | Development guide |
| `docs/history.md` | Project history |

---

## 🔧 Public API Surface

### Node Interface

```go
type Node interface {
    Render(ctx *Context) string
}
```

**Implementations:**
- `El(tag, children...)` — HTML element
- `Text(key, args...)` — Translated text
- `Raw(content)` — Trusted raw HTML
- `If(cond, node)` — Conditional rendering
- `Unless(cond, node)` — Inverse conditional
- `Each[T](items, fn)` — Collection iteration
- `EachSeq[T](items, fn)` — Iterator iteration
- `Switch(selector, cases)` — Multi-branch conditional
- `Entitled(feature, node)` — Entitlement gating
- `Layout(variant)` — HLCRF layout
- `Responsive()` — Responsive wrapper

### Context Type

```go
type Context struct {
    Identity     string
    Locale       string
    Entitlements func(feature string) bool
    Data         map[string]any
    Metadata     map[string]any
}
```

**Constructors:**
- `NewContext()` — Default context
- `NewContextWithService(svc)` — Context with translator

**Methods:**
- `SetLocale(locale)` — Set locale
- `SetEntitlements(fn)` — Set entitlement function
- `SetService(svc)` — Set translator service

### Layout Type

**Constructor:**
- `NewLayout(variant)` — Create layout with variant string

**Slot Methods (return `*Layout` for chaining):**
- `H(nodes...)` — Header slot
- `L(nodes...)` — Left slot
- `C(nodes...)` — Content slot
- `R(nodes...)` — Right slot
- `F(nodes...)` — Footer slot

**Variant strings:**
- `"HLCRF"` — All five slots
- `"HCF"` — Header, Content, Footer
- `"C"` — Content only
- `"LC"` — Left, Content
- Any combination of H, L, C, R, F

### Responsive Type

**Constructor:**
- `NewResponsive()` — Create responsive wrapper

**Methods:**
- `Variant(name, layout, media?)` — Add variant with optional media query
- `VariantSelector(name)` — Get CSS attribute selector

### Attribute Helpers

All apply recursively through wrapper nodes:

- `Attr(node, key, value)` — Set arbitrary attribute
- `AriaLabel(node, label)` — Set aria-label
- `AltText(node, text)` — Set alt attribute
- `TabIndex(node, index)` — Set tabindex
- `AutoFocus(node)` — Set autofocus
- `Role(node, role)` — Set role

### Grammar Pipeline (Server-Only)

- `StripTags(html)` — Remove HTML tags
- `Imprint(node, ctx)` — Grammar analysis
- `CompareVariants(resp, ctx)` — Compare variant similarity

### Web Component Codegen

- `cmd/codegen/` — Generate Web Component classes

### WASM Module

- `gohtml.renderToString(variant, locale, slots)` — Render in browser

---

## 🏷️ Node Type Catalog

### Content Nodes

| Node | Purpose | Escaping | Children |
|------|---------|----------|-----------|
| `El` | HTML element | ✅ Attributes | ✅ Yes |
| `Text` | Translated text | ✅ HTML | ❌ No |
| `Raw` | Trusted HTML | ❌ No | ❌ No |

### Control Flow Nodes

| Node | Purpose | Condition | Children |
|------|---------|-----------|-----------|
| `If` | Conditional render | Function | ✅ Yes |
| `Unless` | Inverse conditional | Function | ✅ Yes |
| `Switch` | Multi-branch | Selector function | ✅ Yes |

### Iteration Nodes

| Node | Purpose | Input | Children |
|------|---------|-------|-----------|
| `Each[T]` | Slice iteration | `[]T` | Function |
| `EachSeq[T]` | Iterator iteration | `iter.Seq[T]` | Function |

### Layout Nodes

| Node | Purpose | Variant | Children |
|------|---------|---------|-----------|
| `Layout` | HLCRF compositor | String | Slot nodes |
| `Responsive` | Multi-variant wrapper | - | Layout variants |

### Accessibility Nodes

| Node | Purpose | Attribute | Children |
|------|---------|-----------|-----------|
| `Entitled` | Content gating | Feature string | ✅ Yes |

---

## 🎨 HLCRF Layout Catalog

### Slot Definitions

| Letter | Element | ARIA Role | Semantic Meaning |
|--------|---------|-----------|------------------|
| H | `<header>` | `banner` | Page header |
| L | `<nav>` | `navigation` | Left sidebar/nav |
| C | `<main>` | `main` | Main content |
| R | `<aside>` | `complementary` | Right sidebar |
| F | `<footer>` | `contentinfo` | Page footer |

### Common Variant Strings

| Variant | Slots | Use Case |
|---------|-------|----------|
| `"HLCRF"` | All 5 | Full desktop layout |
| `"HCF"` | H, C, F | Standard layout without sidebars |
| `"C"` | C only | Minimal content-only |
| `"LC"` | L, C | Left sidebar + content |
| `"CR"` | C, R | Content + right sidebar |
| `"HLCF"` | H, L, C, F | Header, left, content, footer |
| `"HCF"` | H, C, F | Classic header-content-footer |
| `"HLCRF"` | All 5 | Full 5-slot layout |

### Deterministic Block IDs

Block IDs follow the slot structure:

| Structure | Block ID |
|-----------|----------|
| Root H slot | `H` |
| Root C slot | `C` |
| Nested in L.0 | `L.0` |
| Nested in C.2.1 | `C.2.1` |
| Deep nesting (10 levels) | `C.0.0.0.0.0.0.0.0.0` |

**Properties:**
- No `fmt.Sprintf` (keeps WASM size down)
- String concatenation only
- Stable across renders
- Unique per position

---

## ⚡ WASM Integration

### JavaScript API

```javascript
gohtml.renderToString(variant, locale, slots)
```

**Parameters:**
- `variant` (string): HLCRF variant string (e.g., `"HCF"`)
- `locale` (string): BCP 47 locale (e.g., `"en-GB"`)
- `slots` (object): `{ H?: string, L?: string, C?: string, R?: string, F?: string }`

**Returns:** HTML string

### Size Budget

| Metric | Limit | Current | Status |
|--------|-------|---------|--------|
| Raw binary | 3.5 MB | 2.90 MB | ✅ Under limit |
| Gzip compressed | 1 MB | 842 KB | ✅ Under limit |

### Build Commands

```bash
# Manual build
GOOS=js GOARCH=wasm go build -ldflags="-s -w" -o gohtml.wasm ./cmd/wasm/

# Makefile target (includes size check)
make wasm
```

### Critical Constraints

**Never import in WASM-linked code:**
- `encoding/json` — Adds ~1 MB
- `text/template` — Adds ~800 KB
- `fmt` — Adds ~500 KB

**Use instead:**
- String concatenation
- Custom formatting functions
- Minimal local helpers

---

## 🧠 Grammar Pipeline

### Components

| Component | Purpose | Location |
|-----------|---------|----------|
| `StripTags` | Remove HTML tags | `pipeline.go` |
| `Tokenise` | Tokenize text | `go-i18n/reversal` |
| `GrammarImprint` | Feature vector | `grammar.go` |
| `Imprint` | Full pipeline | `pipeline.go` |
| `CompareVariants` | Similarity scoring | `pipeline.go` |

### GrammarImprint Features

| Feature | Description | Weight |
|---------|-------------|--------|
| `VerbDistribution` | Verb frequencies by base form | 30% |
| `TenseDistribution` | Past/gerund/base ratios | 20% |
| `NounDistribution` | Noun frequencies by base form | 25% |
| `PluralRatio` | Plural vs singular ratio | - |
| `DomainVocabulary` | Category hit counts | - |
| `ArticleUsage` | Definite/indefinite ratios | 15% |
| `PunctuationPattern` | Label/progress/question ratios | 10% |
| `TokenCount` | Total token count | - |
| `UniqueVerbs` | Unique verb count | - |
| `UniqueNouns` | Unique noun count | - |

### Similarity Scoring

`CompareVariants()` returns pairwise similarity scores:

```go
scores := html.CompareVariants(responsive, ctx)
// Returns: map[string]float64{
//     "desktop:tablet": 0.85,
//     "desktop:mobile": 0.72,
//     "tablet:mobile": 0.88,
// }
```

**Use case:** Detect semantically divergent responsive variants

---

## 🎭 Web Component Codegen

### CLI Usage

```bash
# From JSON slot map
echo '{"H":"nav-bar","C":"main-content"}' | go run ./cmd/codegen/

# From file
cat layout.json | go run ./cmd/codegen/ > components.js

# With TypeScript declarations
cat layout.json | go run ./cmd/codegen/ --typescript > components.ts
```

### Generated Output

**JavaScript:**
```javascript
class NavBar extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'closed' });
        this.shadowRoot.innerHTML = `<nav>...</nav>`;
    }
}
customElements.define('nav-bar', NavBar);
```

**TypeScript:**
```typescript
class NavBar extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'closed' });
        this.shadowRoot.innerHTML = `<nav>...</nav>`;
    }
}
customElements.define('nav-bar', NavBar);
```

### Features

- **Closed Shadow DOM** — Hermetic isolation
- **Deterministic** — Same input → same output
- **Minimal** — Only generates needed code
- **TypeScript support** — Optional type declarations

---

## 📊 Statistics

### Code Metrics

```
Total Go files:           25+
Total source lines:    ~643
Total test lines:      ~2,000+
Public API symbols:     ~20
WASM binary size:       2.90 MB raw, 842 KB gzip
Node types:             11
Layout variants:       31 (practical subset of 119 possible)
Attribute helpers:      6 (Attr, AriaLabel, AltText, TabIndex, AutoFocus, Role)
```

### Test Coverage

| Category | Files | Tests | Status |
|----------|-------|-------|--------|
| Node types | 11 | 50+ | ✅ Complete |
| Layout | 1 | 30+ | ✅ Complete |
| Responsive | 1 | 20+ | ✅ Complete |
| Grammar pipeline | 3 | 15+ | ✅ Complete |
| WASM size | 1 | 1 | ✅ Enforced |
| Edge cases | 1 | 20+ | ✅ Complete |

### Performance

| Operation | Time | Notes |
|-----------|------|-------|
| Node render | <1μs | Single node |
| Layout render | <10μs | Full HLCRF |
| Responsive render | <50μs | All variants |
| StripTags | <10μs | Per KB |
| GrammarImprint | <100μs | Full analysis |
| WASM render | <1ms | Browser-side |

---

## 🏷️ Tags & Categories

### Technology Tags

- `html` — Primary tag
- `parsing` — HTML parsing/generation
- `templates` — Template engine
- `markup` — Markup generation
- `dom` — DOM manipulation
- `consent` — Consent-aware rendering
- `i18n` — Internationalization
- `wasm` — WebAssembly
- `web-components` — Web Component generation
- `accessibility` — a11y support
- `aria` — ARIA roles

### Feature Tags

- `rendering` — HTML rendering
- `layout` — Page layout
- `responsive` — Responsive design
- `grammar` — Grammar analysis
- `codegen` — Code generation
- `type-safe` — Type safety
- `xss-safe` — XSS prevention
- `entitlements` — Access control

### Node Tags

- `el` — HTML element
- `text` — Text node
- `raw` — Raw HTML
- `if` — Conditional
- `unless` — Inverse conditional
- `each` — Iteration
- `switch` — Multi-branch
- `entitled` — Access control
- `layout` — Page layout
- `responsive` — Responsive layout

---

## 🔗 Dependencies

### Internal Dependencies

| Package | Purpose | Import Path |
|---------|---------|-------------|
| go-i18n | i18n translation and reversal | `dappco.re/go/core/i18n` |
| go-inference | Grammar classification | `forge.lthn.ai/core/go-inference` (indirect) |
| go-log | Structured logging | `forge.lthn.ai/core/go/log` |
| go-io | I/O operations | `dappco.re/go/core/io` |
| Core framework | Result pattern, error handling | `dappco.re/go` |

### External Dependencies

| Package | Purpose |
|---------|---------|
| `html` | Standard library HTML escaping |
| `testing` | Standard library testing |

### Requirements

- **Go version:** 1.26+ (uses `range` over integers, `iter.Seq`, `maps.Keys`, `slices.Collect`)
- **WASM:** `GOOS=js GOARCH=wasm` with `-s -w` linker flags

---

## 🎯 Usage Patterns

### Pattern 1: Basic Page Layout

```go
html.NewLayout("HCF").
    H(navNode).
    C(mainNode).
    F(footerNode)
```

### Pattern 2: Consent-Aware Rendering

```go
ctx := html.NewContext("en-GB")
ctx.Entitlements = entitlementCheck

html.NewLayout("C").
    C(html.Entitled("premium", premiumContent))
```

### Pattern 3: Responsive Design

```go
html.NewResponsive().
    Variant("desktop", desktopLayout).
    Variant("tablet", tabletLayout).
    Variant("mobile", mobileLayout)
```

### Pattern 4: i18n Integration

```go
html.Text("greeting.hello")  // Looks up translation
html.Text("welcome.user", username)  // With interpolation
```

### Pattern 5: WASM Client Rendering

```javascript
gohtml.renderToString("HCF", "en-GB", { H: "<nav>...</nav>", C: "<main>...</main>" })
```

---

## 📋 Compliance Summary

### Coding Standards

✅ **UK English:** colour, organisation, centre, behaviour, licence, serialise
✅ **All types annotated:** Use `any` not `interface{}`
✅ **Tests use testify:** assert/require
✅ **Licence:** EUPL-1.2 with SPDX identifier
✅ **Safe-by-default:** HTML escaping on Text nodes and attributes

### Safety Rules

✅ **XSS prevention:** `html.EscapeString()` on Text and attributes
✅ **Void element handling:** 13 void elements never emit closing tags
✅ **Entitlement deny-by-default:** Content absent from DOM when not entitled
✅ **Deterministic output:** Sorted attributes, reproducible block IDs
✅ **No fmt in WASM:** Uses string concatenation only

### Build Constraints

✅ **WASM size gate:** < 3.5 MB raw, < 1 MB gzip
✅ **No banned imports:** encoding/json, text/template, fmt
✅ **Go version:** 1.26+

### Test Organization

✅ **File-aware tests:** Each production file owns its test file
✅ **Table-driven subtests:** Uses `t.Run()`
✅ **Good/Bad/Ugly pattern:** Comprehensive test coverage
✅ **Integration tests:** Initialize i18n before Text node tests

### Verification

```sh
GOWORK=off go mod tidy
GOWORK=off go vet ./...
GOWORK=off go test -count=1 ./...
gofmt -l .
bash /Users/snider/Code/core/go/tests/cli/v090-upgrade/audit.sh .
```

---

## 📝 Maintenance Information

- **Author**: Mistral Vibe (Purberus <purberus@lthn.ai>)
- **Created**: 2026-06-18T01:00:00Z
- **Last Updated**: 2026-06-18T01:00:00Z
- **Version**: 1.0.0
- **Licence**: EUPL-1.2
- **Repository**: `forge.lthn.sh/core/go-html`
- **Module**: `dappco.re/go/core/html`

### Key Contacts

- **Project Lead:** Hades (Lethean)
- **Maintainer:** Purberus <purberus@lthn.ai>
- **Consumes:** go-i18n, go-inference
- **Used by:** Core GUI, Core IDE, Core Agent

---

*Package index generated: 2026-06-18T01:00:00Z*
