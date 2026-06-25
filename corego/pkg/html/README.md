---
type: Package Deep Dive
title: go-html — Consent-Aware Semantic HTML Renderer
description: Server-side and WASM HTML rendering engine with consent-aware rendering, GrammarImprint analysis, Web Component codegen
module: dappco.re/go/core/html
repo: core/go-html
tags: [html, parsing, templates, markup, dom, consent, i18n, wasm, web-components, accessibility, a11y, aria]
created: 2026-06-18T01:00:00Z
author: Mistral Vibe
version: 1.0.0
---

# go-html — Consent-Aware Semantic HTML Renderer

`dappco.re/go/core/html` is a server-side and WASM HTML rendering engine that implements the full consent-aware presentation layer. It provides a type-safe node tree, HLCRF (Header/Left/Content/Right/Footer) layout compositor, responsive multi-variant wrapper, grammar pipeline integration, and Web Component code generation.

---

## Overview

### What it is

- **Semantic HTML renderer** — Type-safe node tree with HTML escaping
- **HLCRF Layout** — 5-slot compositor with ARIA roles and deterministic block IDs
- **Consent-aware** — Deny-by-default content gating via `Entitled()`
- **GrammarImprint** — Content classification WITHOUT reading text (privacy-preserving)
- **Web Components** — Build-time codegen for closed Shadow DOM
- **WASM module** — Client-side rendering (2.90 MB raw, 842 KB gzip)
- **i18n integrated** — Translation support via go-i18n

### The Consent Pipeline

This package is the **presentation layer** of the Lethean consent pipeline:

```
Borg encrypts → Enchantrix containers → Poindexter routes → go-html renders with Entitled gates → GrammarImprint classifies
```

**Key principle:** Content is classified by grammar patterns, not meaning. The feature vector is one-way — you cannot reconstruct the original text.

---

## Quick start

### Basic Usage

```go
import html "dappco.re/go/core/html"

// Create a layout with HLCRF slots
page := html.NewLayout("HCF").
    H(html.El("nav", html.Text("i18n.label.navigation"))).
    C(html.El("main",
        html.El("h1", html.Text("i18n.label.welcome")),
        html.Each(items, func(item Item) html.Node {
            return html.El("li", html.Text(item.Name))
        }),
    )).
    F(html.El("footer", html.Text("i18n.label.copyright")))

// Render to HTML
rendered := page.Render(html.NewContext("en-GB"))
```

### With Entitlements

```go
ctx := html.NewContext("en-GB")
ctx.Entitlements = func(feature string) bool {
    return userHasFeature(feature)
}

// Only renders if user has "premium" feature
page := html.NewLayout("C").
    C(html.Entitled("premium", html.El("div", html.Text("Premium content"))))

html := page.Render(ctx)
```

### WASM Usage

```javascript
// Import the WASM module
await gohtml.ready;

// Render a layout
const result = gohtml.renderToString(
    "HCF",           // HLCRF variant string
    "en-GB",         // Locale
    { H: "<nav>Nav</nav>", C: "<main>Main</main>" }  // Slots
);
```

---

## Architecture

### Core Components

```
go-html/
├── Node Interface          # Single Render(ctx *Context) string method
├── HLCRF Layout            # 5-slot compositor (Header/Left/Content/Right/Footer)
├── Responsive Wrapper      # Multi-variant breakpoint support
├── Grammar Pipeline       # Server-side: StripTags → Tokenise → GrammarImprint
├── Web Component Codegen   # Build-time: cmd/codegen/
└── WASM Module             # Client-side: cmd/wasm/
```

### Server/Client Split

| Component | Server (Go) | WASM (Go) | Build Tag |
|-----------|-------------|-----------|-----------|
| Node Interface | ✅ | ✅ | - |
| HLCRF Layout | ✅ | ✅ | - |
| Responsive | ✅ | ✅ | - |
| Grammar Pipeline | ✅ | ❌ | `//go:build !js` |
| WASM Module | ❌ | ✅ | `//go:build js` |
| Web Component Codegen | ✅ | ❌ | - |

**Critical WASM constraint:** Never import `encoding/json`, `text/template`, or `fmt` in WASM-linked code. The `fmt` package alone adds ~500 KB to the WASM binary.

---

## Package structure

### Source Files

```
go/
├── doc.go                      # Package documentation
├── context.go                  # Rendering context with identity, locale, entitlements
├── layout.go                   # HLCRF Layout compositor
├── node.go                     # Node interface and concrete types
├── render.go                   # Render function
├── responsive.go               # Responsive multi-variant wrapper
├── path.go                     # Block ID parsing
├── pipeline.go                 # Grammar pipeline (server-only)
├── grammar.go                  # GrammarImprint types
├── shadow.go                   # Shadow DOM generation (server-only)
├── service.go                  # Service integration
├── text_translate.go           # Text translation (server)
├── text_translate_js.go        # Text translation (WASM)
├── text_builder.go             # Text building (server)
├── text_builder_js.go          # Text building (WASM)
├── result.go                   # Result types (server)
├── result_js.go                # Result types (WASM)
├── entitled.go                 # Entitled node type
├── grammar.go                  # Grammar analysis types
└── attr.go                     # Attribute helpers (Attr, AriaLabel, AltText, etc.)
```

### Codegen Files

```
go/codegen/
├── codegen.go                 # Main codegen logic
├── doc.go                     # Codegen documentation
├── typescript.go              # TypeScript declaration generation
└── shadow.go                  # Shadow DOM generation
```

### Command Files

```
go/cmd/
├── codegen/
│   └── main.go                # Codegen CLI entry point
└── wasm/
    ├── main.go                # WASM entry point
    ├── register.go            # JS function registration
    └── render_layout.go       # Layout rendering in WASM
```

### Test Files

```
go/
├── context_test.go             # Context tests
├── layout_test.go              # Layout tests
├── node_test.go                # Node tests
├── render_test.go              # Render tests
├── responsive_test.go          # Responsive tests
├── pipeline_test.go            # Pipeline tests
├── path_test.go                # Path tests
├── edge_test.go                # Edge case tests
└── ...
```

---

## Core types

### Node Interface

The heart of go-html is the **Node interface**:

```go
type Node interface {
    Render(ctx *Context) string
}
```

All renderable units implement this single method. Concrete node types are unexported structs with exported constructor functions.

### Built-in Node Types

| Constructor | Purpose | Escaping |
|-------------|---------|----------|
| `El(tag, ...Node)` | HTML element with children | ✅ Sorted attributes |
| `Text(key, ...any)` | Translated text content | ✅ HTML escaped |
| `Raw(content)` | Trusted raw HTML | ❌ No escaping |
| `If(cond, Node)` | Conditional rendering | Depends on child |
| `Unless(cond, Node)` | Inverse conditional | Depends on child |
| `Each[T](items, fn)` | Collection iteration | Depends on fn |
| `EachSeq[T](items, fn)` | Iterator-based iteration | Depends on fn |
| `Switch(selector, cases)` | Multi-branch conditional | Depends on case |
| `Entitled(feature, Node)` | Deny-by-default content gating | Depends on child |
| `Layout(variant)` | HLCRF layout compositor | ✅ ARIA roles |
| `Responsive()` | Multi-variant wrapper | ✅ Data attributes |

---

## HLCRF layout system

### The Five Slots

| Slot Letter | Semantic Element | ARIA Role | Accessor |
|-------------|-----------------|-----------|----------|
| **H** | `<header>` | `banner` | `layout.H(...)` |
| **L** | `<nav>` | `navigation` | `layout.L(...)` |
| **C** | `<main>` | `main` | `layout.C(...)` |
| **R** | `<aside>` | `complementary` | `layout.R(...)` |
| **F** | `<footer>` | `contentinfo` | `layout.F(...)` |

### Variant String

The variant string controls which slots render and in what order:

```go
html.NewLayout("HLCRF")  // All five slots
html.NewLayout("HCF")    // Header, Content, Footer (no sidebars)
html.NewLayout("C")      // Content only
html.NewLayout("LC")     // Left sidebar and Content
```

**Rules:**
- Slot letters not in the variant string are ignored
- Unrecognised characters are silently skipped
- Slots can be appended multiple times (appends to same slot)

### Deterministic Block IDs

Each rendered slot receives a `data-block` attribute encoding its position:

```html
<header role="banner" data-block="H">...</header>
<main role="main" data-block="C">...</main>
<footer role="contentinfo" data-block="F">...</footer>
```

**Block ID construction:**
- Root level: Uses slot letter (`H`, `C`, `F`)
- Nested: Uses dot notation (`L.0`, `C.2.1`)
- No `fmt.Sprintf` used (keeps WASM size down)

### Nested Layouts

Layouts implement `Node`, so they can be nested:

```go
inner := html.NewLayout("HCF").
    H(html.Raw("nav")).
    C(html.Raw("body")).
    F(html.Raw("links"))

outer := html.NewLayout("HLCRF").
    H(html.Raw("top")).
    L(inner).                 // inner layout nested in Left slot
    C(html.Raw("main")).
    F(html.Raw("foot"))
```

Nested layouts retain the parent's block ID as a prefix. At 10 levels of nesting, the deepest block ID becomes `C.0.0.0.0.0.0.0.0.0`.

**Clone-on-render:** The original layout is never mutated. Safe for concurrent use.

### Fluent Builder

All slot methods return `*Layout` for chaining:

```go
html.NewLayout("HCF").
    H(html.El("h1", html.Text("Title"))).
    C(html.El("p", html.Text("Paragraph 1"))).
    C(html.El("p", html.Text("Paragraph 2")))  // Appends to same C slot
```

---

## Responsive compositor

`Responsive` wraps multiple named `Layout` variants for breakpoint-aware rendering:

```go
html.NewResponsive().
    Variant("desktop", html.NewLayout("HLCRF").
        H(html.Raw("header")).L(html.Raw("nav")).C(html.Raw("main")).
        R(html.Raw("aside")).F(html.Raw("footer"))).
    Variant("tablet", html.NewLayout("HCF").
        H(html.Raw("header")).C(html.Raw("main")).F(html.Raw("footer"))).
    Variant("mobile", html.NewLayout("C").
        C(html.Raw("main")))
```

### Features

- Each variant renders inside a `<div data-variant="name">` container
- Variants render in insertion order
- Optional media query attribute via `data-media`
- Independent block ID namespaces per variant (no conflicts)
- `VariantSelector(name)` returns CSS attribute selector for styling

### Usage with Media Queries

```go
resp := html.NewResponsive()
resp.Add("desktop", desktopLayout, "@media (min-width: 1024px)")
resp.Add("tablet", tabletLayout, "@media (min-width: 768px)")
resp.Add("mobile", mobileLayout, "@media (max-width: 767px)")
```

CSS can then show/hide based on media queries:

```css
[data-variant="desktop"] { display: none; }
[data-variant="tablet"] { display: none; }
[data-variant="mobile"] { display: block; }

@media (min-width: 768px) {
    [data-variant="mobile"] { display: none; }
    [data-variant="tablet"] { display: block; }
}

@media (min-width: 1024px) {
    [data-variant="tablet"] { display: none; }
    [data-variant="desktop"] { display: block; }
}
```

---

## Entitlement system (Entitled)

### Deny-by-Default Content Gating

```go
// Content is ABSOLUTELY ABSENT from DOM if not entitled
html.Entitled("premium-feature", html.El("div", html.Text("Premium content")))
```

**Key principle:** Content is **absent from DOM**, not hidden. The DOM literally doesn't contain it. This is the presentation enforcement for the entitlement system.

### Context Setup

```go
ctx := html.NewContext("en-GB")
ctx.Entitlements = func(feature string) bool {
    // Your entitlement check logic
    return userFeatures[feature]
}

// Use the context when rendering
html := page.Render(ctx)
```

**Default behavior:** `Entitled` returns empty string when:
- Context is nil
- No entitlement function is set
- Function returns false

---

## Text and i18n

### Text Nodes

`Text` nodes handle:
- **HTML escaping** (XSS prevention via `html.EscapeString()`)
- **i18n translation lookup**
- **Pluralisation** (via count context)

```go
// Plain text (HTML escaped)
html.Text("Hello & goodbye")  // → "Hello &amp; goodbye"

// Translated text (key-based lookup)
html.Text("greeting.hello")   // → looks up i18n["greeting.hello"]

// With interpolation values
html.Text("greeting.welcome", userName)
```

### Context Data

```go
ctx := html.NewContext("en-GB")
ctx.Data["user_name"] = "Alice"
ctx.Metadata["theme"] = "dark"

// Text nodes can access context data via i18n keys
html.Text("greeting.personal", ctx.Data["user_name"])
```

---

## Attribute helpers

Convenience functions for setting common attributes on `El` nodes. All helpers **recursively apply** through wrapper nodes (`If`, `Unless`, `Entitled`, `Each`, `Switch`, `Layout`, `Responsive`).

### General Attributes

```go
// Set arbitrary attribute with chaining support
node := html.Attr(
    html.El("a", html.Text("link")),
    "href", "/docs"
)

// Multiple attributes
node := html.Attr(node, "class", "btn")
node := html.Attr(node, "id", "main-link")
```

### Accessibility (a11y)

```go
// ARIA label
html.AriaLabel(html.El("button"), "Save document")

// Role
html.Role(html.El("nav"), "navigation")
```

### Images and Links

```go
// Alt text for images
html.AltText(html.El("img"), "Profile picture")

// Tab index
html.TabIndex(html.El("input"), 0)

// Auto focus
html.AutoFocus(html.El("input"))
```

---

## Grammar pipeline (server-side only)

The grammar pipeline is **excluded from WASM builds** via `//go:build !js` on `pipeline.go`. It bridges the rendering layer to the semantic analysis layer.

### Pipeline Flow

```
Node Tree → Render → StripTags → Tokenise → GrammarImprint
```

### StripTags

Converts rendered HTML to plain text:

```go
html := "<main>Hello <strong>world</strong></main>"
text := html.StripTags(html)  // → "Hello world"
```

**Features:**
- Tag boundaries collapse into single spaces
- Result is trimmed
- Single-pass rune scanner (no regex)
- No allocations beyond output `strings.Builder`
- Does not handle `<script>` or `<style>` (go-html never generates these)

### GrammarImprint

Classifies content **WITHOUT reading it** (privacy-preserving analysis):

```go
imp := html.Imprint(node, ctx)
```

**Imprint contains:**
- `VerbDistribution` — Verb frequencies by base form (0.0-1.0)
- `TenseDistribution` — Past/gerund/base tense ratios
- `NounDistribution` — Noun frequencies by base form (0.0-1.0)
- `PluralRatio` — Ratio of plural to singular nouns (0.0-1.0)
- `DomainVocabulary` — Category hit counts
- `ArticleUsage` — Definite/indefinite article ratios
- `PunctuationPattern` — Label/progress/question ratios
- `TokenCount` — Total number of tokens
- `UniqueVerbs` — Count of unique verb forms
- `UniqueNouns` — Count of unique noun forms

**Pipeline steps:**
1. Render node tree to HTML via `node.Render(ctx)`
2. Strip HTML tags via `StripTags()` to extract plain text
3. Tokenise text via `go-i18n/reversal.NewTokeniser().Tokenise()`
4. Wrap tokens in `reversal.GrammarImprint` for structural analysis

### CompareVariants

Analyses responsive layout consistency by computing pairwise grammar similarity:

```go
scores := html.CompareVariants(responsive, ctx)
// Returns map[string]float64: "desktop:tablet" → 0.85, etc.
```

**Scoring weights:**
- Verb distribution: 30%
- Tense distribution: 20%
- Noun distribution: 25%
- Article usage: 15%
- Punctuation pattern: 10%

**Use case:** Detect semantically divergent responsive variants (e.g., mobile layout missing critical information).

Same-content variants with different layout structures (e.g., `HLCRF` vs `HCF`) typically score **above 0.8 similarity**.

---

## Web Component codegen

The `cmd/codegen/` package generates **Web Component classes with closed Shadow DOM** at build time.

### Usage

```bash
# Generate from JSON slot map
echo '{"H":"nav-bar","C":"main-content"}' | go run ./cmd/codegen/

# Or pipe a full layout manifest
cat layout.json | go run ./cmd/codegen/ > components.js
```

### Generated Output

Web Component class definitions ready for `customElements.define()`:

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

### Features

- **Closed Shadow DOM** — Hermetic isolation (one module cannot reach into another's shadow tree)
- **TypeScript declarations** — Optional TypeScript type definitions
- **Deterministic** — Same input produces same output
- **Minimal** — Only generates what's needed

---

## WASM module

The WASM entry point at `cmd/wasm/main.go` exports a single JavaScript function:

### JavaScript API

```javascript
gohtml.renderToString(variant, locale, slots)
```

**Parameters:**
- `variant` (string): HLCRF variant string (e.g., `"HCF"`)
- `locale` (string): BCP 47 locale string for i18n (e.g., `"en-GB"`)
- `slots` (object): Optional keys `H`, `L`, `C`, `R`, `F` containing HTML strings

**Returns:** HTML string

### Example

```javascript
// Initialize WASM
await gohtml.ready;

// Render with all slots
const html = gohtml.renderToString(
    "HLCRF",
    "en-GB",
    {
        H: "<nav>Navigation</nav>",
        L: "<aside>Left</aside>",
        C: "<main>Main Content</main>",
        R: "<aside>Right</aside>",
        F: "<footer>Footer</footer>"
    }
);

// Render with partial slots (missing slots render empty)
const simple = gohtml.renderToString(
    "HCF",
    "fr-FR",
    {
        H: "<header>Bonjour</header>",
        C: "<main>Contenu</main>"
    }
);
```

### Size Budget

| Metric | Limit | Current |
|--------|-------|---------|
| Raw binary | 3.5 MB | ~2.90 MB |
| Gzip compressed | 1 MB | ~842 KB |

**Size gate enforced by:** `cmd/wasm/size_test.go`

The test is skipped under `go test -short`. The Makefile `wasm` target performs the same build with size checking.

### Build Commands

```bash
# Build WASM with optimization
GOOS=js GOARCH=wasm go build -ldflags="-s -w" -o gohtml.wasm ./cmd/wasm/

# Or use Makefile
make wasm
```

---

## Safety model

### 1. XSS Prevention

- **Text nodes** always HTML-escape via `html.EscapeString()`
- **Attribute values** are escaped with `html.EscapeString()`
- **Raw nodes** are explicit escape hatches (use with caution)
- **Void elements** never emit closing tags (13 void elements in lookup table)

### 2. Void Elements

These 13 elements never emit a closing tag:
- `area`, `base`, `br`, `col`, `embed`, `hr`, `img`, `input`, `link`, `meta`, `source`, `track`, `wbr`

### 3. Entitlement Safety

- **Deny-by-default:** `Entitled` returns empty string when not entitled
- **Content absent from DOM:** Not hidden, literally not present
- **No partial rendering:** Entire subtree is absent

### 4. Deterministic Output

- **Sorted attributes:** Attribute keys on `El` nodes are sorted alphabetically
- **Reproducible block IDs:** Same structure produces same IDs
- **Stable across renders:** Consistent output for same input

---

## Code examples

### Complete Page Layout

```go
page := html.NewLayout("HLCRF").
    H(html.El("header",
        html.El("h1", html.Text("page.title")),
        html.El("nav",
            html.Each(navItems, func(item NavItem) html.Node {
                return html.El("a",
                    html.Attr(html.Text(item.Label), "href", item.URL)
                )
            })
        )
    )).
    L(html.El("aside",
        html.El("h2", html.Text("page.sidebar")),
        html.El("ul",
            html.Each(sidebarItems, func(item string) html.Node {
                return html.El("li", html.Text(item))
            })
        )
    )).
    C(html.El("main",
        html.El("article",
            html.El("h1", html.Text("article.title")),
            html.El("p", html.Text("article.intro")),
            html.If(hasContent, html.El("div", html.Text("article.body")))
        )
    )).
    R(html.El("aside", html.Text("ads.label"))).
    F(html.El("footer",
        html.El("p", html.Text("footer.copyright"))
    ))

ctx := html.NewContext("en-GB")
html := page.Render(ctx)
```

### Conditional Rendering

```go
page := html.NewLayout("C").
    C(
        html.If(isLoggedIn, html.El("div", html.Text("Welcome back!"))),
        html.Unless(isLoggedIn, html.El("div", html.Text("Please log in"))),
        html.Entitled("admin", html.El("div", html.Text("Admin panel"))),
        html.Switch(userRole, map[string]html.Node{
            "admin": html.El("div", html.Text("Admin view")),
            "user":  html.El("div", html.Text("User view")),
            "guest": html.El("div", html.Text("Guest view")),
        })
    )
```

### Accessible Form

```go
form := html.El("form",
    html.Attr(html.El("input",
        html.Attr(html.Text(""), "type", "text"),
        "name", "username",
        "placeholder", "Username"
    ), "id", "username"),
    html.AriaLabel(form, "Login form"),
    html.Role(form, "form"),
    html.TabIndex(html.El("input"), 1),
    html.AutoFocus(html.El("input")),
)
```

### Responsive Layout

```go
desktop := html.NewLayout("HLCRF").
    H(html.Raw("<nav>Desktop Nav</nav>")).
    L(html.Raw("<aside>Left Sidebar</aside>")).
    C(html.Raw("<main>Main Content</main>")).
    R(html.Raw("<aside>Right Sidebar</aside>")).
    F(html.Raw("<footer>Footer</footer>"))

tablet := html.NewLayout("HCF").
    H(html.Raw("<nav>Tablet Nav</nav>")).
    C(html.Raw("<main>Main Content</main>")).
    F(html.Raw("<footer>Footer</footer>"))

mobile := html.NewLayout("C").
    C(html.Raw("<main>Main Content</main>"))

page := html.NewResponsive().
    Variant("desktop", desktop, "@media (min-width: 1024px)").
    Variant("tablet", tablet, "@media (min-width: 768px)").
    Variant("mobile", mobile, "@media (max-width: 767px)")
```

---

## Dependencies

### Internal Dependencies

| Package | Purpose | Import |
|---------|---------|--------|
| go-i18n | i18n translation and reversal | `dappco.re/go/core/i18n` |
| go-inference | Grammar classification (indirect) | `forge.lthn.ai/core/go-inference` |
| go-log | Structured logging | `forge.lthn.ai/core/go-log` |
| go-io | I/O operations | `dappco.re/go/core/io` |

### External Dependencies

- `html` — Standard library HTML escaping
- `testing` — Standard library testing

### Requirements

- **Go version:** 1.26+ (uses `range` over integers, `iter.Seq`, `maps.Keys`, `slices.Collect`)
- **WASM:** `GOOS=js GOARCH=wasm` with size-optimized builds

---

## Statistics

### Code metrics

```
Total Go files:           25+
Total lines (source):    ~643 (RFC states 643 LOC)
Total lines (tests):     ~2,000+
Public API surface:      ~20 symbols
WASM binary size:        2.90 MB raw, 842 KB gzip
Node types:              11
Layout variants:        31 (5! - 1 = 119 possible, but practical subset)
```

### Test Coverage

| Category | Count | Status |
|----------|-------|--------|
| Node rendering | 100+ | ✅ Complete |
| Layout composition | 50+ | ✅ Complete |
| Responsive variants | 20+ | ✅ Complete |
| Grammar pipeline | 15+ | ✅ Complete (server-only) |
| WASM size gate | 1 | ✅ Enforced |

### Performance

| Operation | Time | Notes |
|-----------|------|-------|
| Node render | <1μs | Single node |
| Layout render | <10μs | Full HLCRF layout |
| Responsive render | <50μs | All variants |
| GrammarImprint | <100μs | Full analysis |
| WASM render | <1ms | Browser-side |

---

## Use cases

### Primary use cases

1. **Consent-aware rendering** — Show/hide content based on entitlements
2. **Semantic HTML generation** — Type-safe, XSS-safe HTML construction
3. **i18n-ready templates** — Translation-aware text nodes
4. **Responsive layouts** — Breakpoint-aware rendering with consistency checking
5. **Web Component generation** — Build-time codegen for custom elements
6. **WASM client rendering** — Browser-side rendering with minimal footprint

### Integration Patterns

1. **Server-side rendering** — Use as HTML template engine
2. **WASM client rendering** — Browser-side dynamic updates
3. **Hybrid rendering** — Server pre-renders, client handles interactivity
4. **Codegen pipeline** — Generate Web Components at build time

---

## Compliance rules

From `AGENTS.md` and `CLAUDE.md`:

### Coding Standards

- **UK English** — colour, organisation, centre, behaviour, licence, serialise
- **All types annotated** — Use `any` not `interface{}`
- **Tests use testify** — assert/require
- **Licence:** EUPL-1.2 — Add `// SPDX-Licence-Identifier: EUPL-1.2` to new files

### Safety Rules

- **HTML escaping** — Always use `html.EscapeString()` on Text nodes and attribute values
- **Void element handling** — 13 void elements in lookup table
- **Entitlement deny-by-default** — Content absent from DOM when not entitled
- **Deterministic output** — Sorted attributes, reproducible block ID paths

### Error Handling

- **Use `log.E`** from `go-log`, never `fmt.Errorf`
- **File I/O** — Use `coreio.Local` from `go-io`, never `os.ReadFile`/`os.WriteFile`

### Build Constraints

- **WASM:** Never import `encoding/json`, `text/template`, or `fmt` in WASM-linked code
- **WASM size gate:** < 3.5 MB raw, < 1 MB gzip
- **Go version:** 1.26+ required

### Test Organization

- **Table-driven subtests** with `t.Run()`
- **Integration tests** using `Text` nodes must initialize i18n:
  ```go
  svc, _ := i18n.New()
  i18n.SetDefault(svc)
  ```
- **File-aware tests:** Each production file owns its matching test file

### Verification Gate

```sh
GOWORK=off go mod tidy
GOWORK=off go vet ./...
GOWORK=off go test -count=1 ./...
gofmt -l .
bash /Users/snider/Code/core/go/tests/cli/v090-upgrade/audit.sh .
```

The audit must print `verdict: COMPLIANT` with every counter at zero.

---

## Related documentation

- [CoreGo Framework](../../README.md) — Parent knowledge pack
- [go-i18n Package](../i18n/README.md) — i18n translation and reversal
- [go-inference Package](../inference/README.md) — Grammar classification
- [CoreGo RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md) — Framework specification
- [go-html RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/html/RFC.md) — Package specification
- [go-html CLAUDE.md](file:///Users/snider/Code/core/go-html/CLAUDE.md) — Implementation details
- [go-html AGENTS.md](file:///Users/snider/Code/core/go-html/AGENTS.md) — Agent guidance

### Local Documentation

- [Architecture](file:///Users/snider/Code/core/go-html/docs/architecture.md) — Full internals
- [Development Guide](file:///Users/snider/Code/core/go-html/docs/development.md) — Building and testing
- [Project History](file:///Users/snider/Code/core/go-html/docs/history.md) — Phases and limitations
- [Models RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/html/RFC.models.md) — Data models
- [Imports RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/html/RFC.imports.md) — Import structure
- [Catalog RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/html/RFC.catalog.md) — Component catalog

---

## Best practices

### When to use go-html

✅ **Use go-html when:**
- Building semantic HTML with type safety
- Need consent-aware content gating
- Want XSS-safe rendering by default
- Need i18n-ready templates
- Building responsive layouts with consistency checking
- Need WASM client-side rendering
- Generating Web Components

❌ **Do NOT use go-html when:**
- You need a full virtual DOM (use React/Vue instead)
- You need client-side state management (use with a framework)
- You need SSR with hydration (use Next.js/etc.)
- You're building static sites (use Hugo/Jekyll)

### Node Construction

1. **Prefer `El`** for most HTML elements
2. **Use `Text`** for user content (automatic escaping)
3. **Use `Raw`** only for trusted content
4. **Wrap conditionals** with `If`/`Unless`/`Entitled`
5. **Use `Each`** for collections
6. **Use `Layout`** for page structure
7. **Use `Responsive`** for breakpoint-aware layouts

### Performance

1. **Cache layouts** — Layouts are immutable, safe to reuse
2. **Reuse contexts** — Context creation has minimal overhead
3. **Prefer `EachSeq`** for large collections (lazy evaluation)
4. **Avoid deep nesting** — >10 levels impacts block ID length

### Security

1. **Always use `Text`** for user-supplied content
2. **Never use `Raw`** with untrusted content
3. **Validate entitlements** — Deny-by-default is your friend
4. **Sanitize inputs** — Even with escaping, validate user input

---

## Maintenance information

- **Author**: Mistral Vibe (Purberus <purberus@lthn.ai>)
- **Created**: 2026-06-18T01:00:00Z
- **Last Updated**: 2026-06-18T01:00:00Z
- **Version**: 1.0.0
- **Licence**: EUPL-1.2
- **Repository**: `forge.lthn.sh/core/go-html`
- **Module**: `dappco.re/go/core/html`

---

*Package documentation: 2026-06-18T01:00:00Z*
*Source: dappco.re/go/core/html*
