---
type: Package Documentation
package: i18n
module: dappco.re/go/i18n
repo: core/go-i18n
lang: go
tags:
  - internationalization
  - localization
  - translation
  - i18n
  - l10n
  - grammar
  - semantics
  - intent
  - reversal
  - grammarimprint
  - dual-class
  - cldr
  - pluralization
---
# go-i18n — Grammar-Aware Internationalization Engine

**RFC:** [plans/code/core/go/i18n/RFC.md](../../../../../plans/code/core/go/i18n/RFC.md)
**Models:** [plans/code/core/go/i18n/RFC.models.md](../../../../../plans/code/core/go/i18n/RFC.models.md)
**Source:** [~/Code/core/go-i18n/](file:///Users/snider/Code/core/go-i18n/)
**Module:** `dappco.re/go/i18n`
**Dependencies:** `dappco.re/core/go`

---

## Overview

`go-i18n` is a grammar-aware internationalization engine. It provides:

### Core Capabilities

1. **Standard Translation** — Simple key-based message lookup with interpolation
2. **Grammar Engine** — Verb conjugation (past tense, gerund), noun pluralization with CLDR categories
3. **Semantic Intent System** — Compose questions, confirmations, success/failure messages from intents
4. **GrammarImprint** — Linguistic fingerprinting for privacy-preserving content classification
5. **Dual-Class Disambiguation** — Intelligent resolution of words that can be both verbs and nouns (test, check, commit, file)
6. **Article Selection** — Automatic "a" vs "an" based on vowel sounds
7. **Formality Levels** — Support for informal/formal language variants (tu/vous, du/Sie)
8. **Locale Loading** — Embedded JSON locales with hot-reload support
9. **Handler Chain** — Extensible middleware for special key namespaces
10. **Missing Key Handling** — Three modes: Normal (silent), Strict (panic), Collect (QA)

### Design philosophy

The package implements a two-layer architecture:

- **Layer 1 (Lightweight):** `core.I18n` interface in `core/go` — minimal footprint, no grammar engine
- **Layer 2 (Full):** `go-i18n` — complete implementation with grammar, reversal, classification

This allows packages to import only `core/go` and use `c.I18n().Translate()` without pulling in the entire grammar engine.

### Primary Use Cases

- **CLI Applications** — Grammar-correct commands, prompts, and messages
- **Web Applications** — Localized UI with proper pluralization and grammar
- **API Responses** — Translated error messages and status updates
- **Content Classification** — GrammarImprint for privacy-preserving semantic analysis
- **Multi-language Support** — Full Unicode, RTL, plural rules for 100+ languages

### Available Locales

Base translation files are available in the `locales/` directory:

| Locale | File | Description |
|--------|------|-------------|
| English | [en.json](./locales/en.json) | Base English translations with full grammar data |
| Français | [fr.json](./locales/fr.json) | French translations with verb conjugations, noun genders, and CLDR plural rules |

**French Locale Features:**
- Complete verb conjugation tables (base, past, gerund)
- Noun pluralization with gender support (masculine/feminine)
- Article selection rules (le/la/les, un/une/des)
- Signal words for grammar analysis
- Number formatting (thousands separator, decimal point)
- Time relative formatting ("il y a X secondes")
- Language name mappings (français, anglais, espagnol, etc.)

---

## Architecture

### Two-Layer Design

```
┌─────────────────────────────────────────────────────────────┐
│                    core/go (Lightweight)                        │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ type Translator interface                                 │ │
│  │   Translate(messageID string, args ...any) Result         │ │
│  │   SetLanguage(lang string) error                         │ │
│  │   Language() string                                       │ │
│  │   AvailableLanguages() []string                           │ │
│  └─────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ type I18n struct                                          │ │
│  │   AddLocales(mounts ...*Embed)                           │ │
│  │   SetTranslator(t Translator)                            │ │
│  │   Translate(messageID string, args ...any) Result        │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    go-i18n (Full Engine)                        │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Service — Main orchestrator                              │ │
│  │   - Loader (JSON, custom)                                  │ │
│  │   - Message cache                                         │ │
│  │   - Grammar engine                                        │ │
│  │   - Handler chain                                         │ │
│  │   - Mode management                                       │ │
│  └─────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Grammar Engine                                           │ │
│  │   - Verb conjugation (PastTense, Gerund)                 │ │
│  │   - Noun pluralization (CLDR categories)                  │ │
│  │   - Article selection (a/an/the)                          │ │
│  │   - Agreement rules                                       │ │
│  └─────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ reversal/ Subpackage                                      │ │
│  │   - Tokeniser (text → tokens with classification)         │ │
│  │   - GrammarImprint (linguistic fingerprinting)           │ │
│  │   - Multiplier (deterministic data augmentation)          │ │
│  │   - Anomaly detection                                     │ │
│  │   - Roundtrip verification                                │ │
│  └─────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Handler Chain                                             │ │
│  │   - LabelHandler (i18n.label.*)                           │ │
│  │   - ProgressHandler (i18n.progress.*)                     │ │
│  │   - CountHandler (i18n.count.*)                          │ │
│  │   - DoneHandler (i18n.done.*)                            │ │
│  │   - FailHandler (i18n.fail.*)                            │ │
│  │   - NumericHandler (i18n.number.*)                       │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Integration Pattern

```go
// core/go — defines the contract
type Translator interface {
    Translate(messageID string, args ...any) Result
    SetLanguage(lang string) error
    Language() string
    AvailableLanguages() []string
}

// go-i18n — implements the contract
func Register(c *core.Core) core.Result {
    svc := &i18n.Service{/* full grammar engine */}
    c.I18n().SetTranslator(svc)
    return core.Result{Value: svc, OK: true}
}

// Any package — uses core only (no go-i18n import)
func myHandler(c *core.Core) {
    r := c.I18n().Translate("greeting.hello", "World")
    greeting := r.Value.(string)  // "Hello, World!"
}
```

---

## Core concepts

### 1. Message Type

Individual translation with all plural forms:

```go
type Message struct {
    Text  string // Simple string value (non-plural)
    Zero  string // count == 0 (Arabic, Latvian, Welsh)
    One   string // count == 1 (most languages)
    Two   string // count == 2 (Arabic, Welsh)
    Few   string // Small numbers (Slavic: 2-4, Arabic: 3-10)
    Many  string // Larger numbers (Slavic: 5+, Arabic: 11-99)
    Other string // Default/fallback form
}

// Usage in locale JSON:
{
    "file": {
        "one": "file",
        "other": "files"
    },
    "vulnerability": {
        "one": "vulnerability",
        "other": "vulnerabilities"
    }
}
```

**CLDR Plural Categories:**
- `PluralOther` — Default/fallback
- `PluralZero` — n == 0 (Arabic, Latvian)
- `PluralOne` — n == 1 (most languages)
- `PluralTwo` — n == 2 (Arabic, Welsh)
- `PluralFew` — Small numbers (Slavic: 2-4, Arabic: 3-10)
- `PluralMany` — Larger numbers (Slavic: 5+, Arabic: 11-99)

### 2. Grammar Data

Language-specific grammar rules loaded from JSON:

```go
type GrammarData struct {
    Verbs    map[string]VerbForms    // verb -> {base, past, gerund}
    Nouns    map[string]NounForms    // noun -> {one, other, ...}
    Articles ArticleForms           // {IndefiniteDefault, IndefiniteVowel, Definite}
    Words    map[string]string       // Special word forms (URL, ID, OK, etc.)
    Punct    PunctuationRules        // Language-specific punctuation
    Signals  SignalData              // Disambiguation signal word lists
    Intents  map[string]Intent       // Semantic intent templates
    Number   NumberFormat            // Locale-specific number formatting
}
```

**Example (en.json):**
```json
{
  "gram": {
    "verb": {
      "delete": {"base": "delete", "past": "deleted", "gerund": "deleting"},
      "run": {"base": "run", "past": "ran", "gerund": "running"},
      "commit": {"base": "commit", "past": "committed", "gerund": "committing"}
    },
    "noun": {
      "file": {"one": "file", "other": "files"},
      "commit": {"one": "commit", "other": "commits"}
    },
    "article": {
      "indefinite": {"default": "a", "vowel": "an"},
      "definite": "the"
    },
    "word": {
      "url": "URL",
      "id": "ID",
      "ok": "OK"
    }
  }
}
```

### 3. Subject Type

Typed subject with metadata for semantic translations:

```go
type Subject struct {
    Noun      string           // The noun type (e.g., "file", "repo")
    Value     any              // The actual value (e.g., filename)
    count     int              // Count for pluralisation (default 1)
    gender    string           // Grammatical gender (feminine, masculine, neuter)
    location  string           // Location context (e.g., "in workspace")
    formality Formality        // Formality level override
}

// Builder methods (chainable):
S("file", "config.yaml")
  .Count(3)
  .Gender("feminine")
  .In("workspace")
  .Formal()
```

**Formality Levels:**
- `FormalityNeutral` — Context-appropriate (default)
- `FormalityInformal` — Informal forms (du, tu, you)
- `FormalityFormal` — Formal forms (Sie, vous, usted)

### 4. Intent System

Semantic intent with templates for all output forms:

```go
type Intent struct {
    Meta     IntentMeta
    Question string // Template for question form
    Confirm  string // Template for confirmation form
    Success  string // Template for success message
    Failure  string // Template for failure message
}

type IntentMeta struct {
    Type      string   // "action", "question", "info"
    Verb      string   // Reference to verb key
    Dangerous bool     // Requires confirmation
    Default   string   // "yes" or "no"
    Supports  []string // Extra options
}

type Composed struct {
    Question string     // "Delete config.yaml?"
    Confirm  string     // "Really delete config.yaml?"
    Success  string     // "config.yaml deleted"
    Failure  string     // "Failed to delete config.yaml"
    Meta     IntentMeta // Intent metadata
}
```

### 5. Token System (Reversal)

Token classification for dual-class disambiguation:

```go
type TokenType int

const (
    TokenUnknown    TokenType = iota  // Default classification
    TokenVerb                           // Matched against verb tables
    TokenNoun                           // Matched against noun tables
    TokenArticle                       // Article token (a, an, the)
    TokenWord                          // Resolved through grammar word map
    TokenPunctuation                   // Punctuation token
    TokenNumber                        // Numeric token
)

type Token struct {
    Raw        string    // Original word form
    Type       TokenType // Primary classification
    AltType    TokenType // Secondary classification (dual-class)
    Confidence float64   // Confidence score (0.0-1.0)
    AltConf    float64   // Alternate classification confidence
    VerbInfo   VerbMatch // If Type == TokenVerb
    NounInfo   NounMatch // If Type == TokenNoun
    ArtType    string    // If Type == TokenArticle ("definite", "indefinite")
    WordCat    string    // If Type == TokenWord (gram.word category)
    PunctType  string    // If Type == TokenPunctuation
}
```

### 6. GrammarImprint

Linguistic feature vector for content fingerprinting:

```go
type GrammarImprint struct {
    VerbDistribution   map[string]float64 // verb base -> frequency
    TenseDistribution  map[string]float64 // "past"/"gerund"/"base" -> ratio
    NounDistribution   map[string]float64 // noun base -> frequency
    PluralRatio        float64            // proportion of plural nouns (0.0-1.0)
    DomainVocabulary   map[string]int     // gram.word category -> hit count
    ArticleUsage       map[string]float64 // "definite"/"indefinite" -> ratio
    PunctuationPattern map[string]float64 // "label"/"progress"/"question" -> ratio
    TokenCount         int
    UniqueVerbs        int
    UniqueNouns        int
}
```

**Use Cases:**
- **Poindexter** — Semantic verification in pointer maps (LetherNet Analysis layer)
- **Multiplier** — Deterministic training data augmentation from grammar tables
- **Classification** — Language/dialect detection from grammar patterns
- **Content Scoring** — Structural quality assessment without reading content

---

## Package components

### Package structure

```
go-i18n/
├── go/
│   ├── i18n.go           # Package-level functions (T, Translate, Raw, Compose, SetLanguage)
│   ├── types.go          # Core types (Message, Subject, Intent, Composed, Mode, Formality, etc.)
│   ├── service.go        # Service struct + main methods
│   ├── loader.go         # JSON loading, flattening, LocaleProvider interface
│   ├── handler.go        # KeyHandler interface + built-in handlers (Label, Progress, Count, Done, Fail, Numeric)
│   ├── context.go        # TranslationContext + context management
│   ├── hooks.go          # Missing key callbacks + MissingKey event
│   ├── grammar.go        # Verb conjugation, articles, agreement rules
│   ├── language.go       # Language detection, BCP47 parsing, TextDirection
│   ├── localise.go       # Translation lookup + template rendering
│   ├── compose.go        # Intent composition (T, C, S functions)
│   ├── numbers.go        # Number formatting (locale-aware)
│   ├── time.go           # Time/date formatting (locale-aware)
│   ├── classify.go       # Language classification
│   ├── calibrate.go      # Calibration scoring
│   ├── validate.go       # Translation validation
│   ├── debug.go          # Debug mode [key] prefixing
│   ├── state.go          # Service state management
│   ├── core_service.go   # Core framework integration
│   ├── default_service.go # Default service fallback
│   │
│   ├── reversal/         # GrammarImprint subpackage
│   │   ├── tokeniser.go    # Text → token stream with classification
│   │   ├── imprint.go      # GrammarImprint calculation
│   │   ├── multiplier.go   # Deterministic data augmentation
│   │   ├── classify_bench_test.go
│   │   ├── reference.go    # Reference implementation
│   │   ├── roundtrip_test.go
│   │   └── anomaly.go      # Anomaly detection
│   │
│   ├── integration/      # Integration tests
│   │   ├── calibrate_test.go
│   │   ├── classify_test.go
│   │   └── result_helpers_test.go
│   │
│   ├── locales/          # Embedded locale files
│   │   ├── en.json        # English locale (base)
│   │   └── fr.json        # French locale
│   │
│   ├── tests/            # Additional test assets
│   │
│   ├── i18n.test        # Compiled test binary (13MB)
│   └── reversal.test     # Compiled reversal test binary (8MB)
│
├── docs/
├── KB/
├── GOAL.md              # SonarQube findings (913 issues)
├── REVIEW.md            # Code review (dual-class disambiguation)
├── README.md
└── go.work
```

### Service (Main Orchestrator)

```go
type Service struct {
    loader           Loader           // Locale source
    messages         map[string]map[string]Message // lang -> key -> message
    currentLang      string
    fallbackLang     string
    requestedLang    string
    languageExplicit bool
    availableLangs   []language.Tag
    mode             Mode             // Normal, Strict, Collect
    debug            bool
    formality        Formality        // Neutral, Informal, Formal
    location         string
    handlers         []KeyHandler     // Handler chain
    loadedLocales    map[int]struct{}
    loadedProviders  map[int]struct{}
    mu               core.RWMutex
}

// Constructors
func New() (*Service, error)
func NewWithLoader(loader Loader, opts ...Option) (*Service, error)

// Options
type Option func(*Service)
func WithDefaultHandlers() Option
func WithFallback(lang string) Option
func WithLanguage(lang string) Option
func WithFormality(f Formality) Option
func WithLocation(location string) Option
func WithHandlers(handlers ...KeyHandler) Option
func WithMode(m Mode) Option
func WithDebug(enabled bool) Option
```

### Handler Chain

Built-in handlers for special key namespaces:

| Handler | Namespace | Example | Output |
|---------|-----------|---------|--------|
| `LabelHandler` | `i18n.label.*` | `i18n.label.status` | `"Status:"` |
| `ProgressHandler` | `i18n.progress.*` | `i18n.progress.build` | `"Building..."` |
| `CountHandler` | `i18n.count.*` | `i18n.count.file` with arg `5` | `"5 files"` |
| `DoneHandler` | `i18n.done.*` | `i18n.done.delete` with arg `"file"` | `"File deleted"` |
| `FailHandler` | `i18n.fail.*` | `i18n.fail.delete` with arg `"file"` | `"Failed to delete file"` |
| `NumericHandler` | `i18n.number.*` | `i18n.number.1234` | `"1,234"` |

**Custom Handlers:**
```go
type KeyHandler interface {
    Match(key string) bool
    Handle(key string, args []any, next func() string) string
}

// Usage
svc := i18n.New(
    i18n.WithDefaultHandlers(),
    i18n.WithHandlers(customHandler),
)

// Test handler chain
result := i18n.RunHandlerChain(handlers, key, args, fallback)
```

### Core Service Integration

```go
// Register i18n with Core framework
c, err := core.New(
    core.WithService("i18n", i18n.NewCoreService(i18n.ServiceOptions{
        Language: "en-GB",
        Fallback: "en",
        Mode:     i18n.ModeNormal,
        Debug:    false,
    })),
)

// CoreService automatically loads:
// 1. Embedded go-i18n base translations (grammar, verbs, nouns)
// 2. Extra filesystems from ServiceOptions.ExtraFS
// 3. All registered locale providers via RegisterLocaleProvider()
```

---

## API reference

### Package-level functions

Simple, global access to i18n functionality:

```go
// Translation
t := i18n.T("greeting.hello", "World")  // "Hello, World!"
result := i18n.Translate("greeting.hello")  // core.Result
raw := i18n.Raw("prompt.yes")           // No i18n.* namespace magic

// Intent composition
composed := i18n.Compose("core.delete", i18n.S("file", "config.yaml"))
// composed.Question  → "Delete config.yaml?"
// composed.Confirm   → "Really delete config.yaml?"
// composed.Success  → "config.yaml deleted"
// composed.Failure  → "Failed to delete config.yaml"

// Language management
lang := i18n.Language()                  // Get current language
result := i18n.SetLanguage("fr")        // Switch to French
langs := i18n.AvailableLanguages()      // List loaded languages
```

### Service Methods

Full control via Service instance:

```go
// Create service
svc, err := i18n.New()
svc, err = i18n.NewWithLoader(customLoader)

// Translation
result := svc.Translate("greeting.hello", "World")
str := svc.T("greeting.hello", "World")

// Language
svc.SetLanguage("de")
lang := svc.Language()

// Locales
svc.AddLocales(fs, "locales")
locales := svc.Locales()

// Intent composition
composed := svc.Compose("core.delete", i18n.S("file", "data.json"))

// Grammar helpers
past := svc.PastTense("delete")   // "deleted"
gerund := svc.Gerund("run")       // "running"
plural := svc.Plural("file", 5)    // "files"
article := svc.Article("hour")    // "an"

// Formatting
num := svc.FormatNumber(1234567)   // "1,234,567" (locale-aware)
time := svc.FormatTime(time.Now()) // "Jun 17, 2026, 3:45 PM" (locale-aware)

// Mode
svc.SetMode(i18n.ModeStrict)   // Panic on missing keys
svc.SetMode(i18n.ModeCollect)   // Collect missing keys for QA
svc.SetDebug(true)             // Prefix untranslated keys with [key]
```

### Subject Builder

```go
// Create subject
subj := i18n.S("file", "config.yaml")

// Chain modifiers
subj = subj.Count(3).In("workspace").Gender("feminine").Formal()

// Access properties
noun := subj.Noun        // "file"
value := subj.Value      // "config.yaml"
count := subj.CountInt()  // 3
isPlural := subj.IsPlural() // true
str := subj.String()     // "config.yaml"
countStr := subj.CountString() // "3"

// Use in composition
composed := svc.Compose("core.delete", subj)
```

### Handler Usage

```go
// Use built-in handlers
T("i18n.label.status")           // "Status:"
T("i18n.progress.build")         // "Building..."
T("i18n.count.file", 5)           // "5 files"
T("i18n.done.delete", "file")    // "File deleted"
T("i18n.fail.delete", "file")    // "Failed to delete file"
T("i18n.number.1234")            // "1,234"

// With subjects
T("i18n.count.file", i18n.S("file", "data.txt").Count(3))
// → "3 files"
```

### Locale Registration

**Option 1: Embed in package**
```go
//go:embed locales/*.json
var localeFS embed.FS

func init() {
    i18n.RegisterLocales(localeFS, "locales")
}

// Or register with Core
func Register(c *core.Core) core.Result {
    c.I18n().AddLocales(core.EmbedFS(localeFS, "locales"))
    return core.OK(nil)
}
```

**Option 2: Custom LocaleProvider**
```go
type LocaleProvider interface {
    Load(lang string) ([]byte, error)
    Available() []string
}

i18n.RegisterLocaleProvider(customProvider)
```

**Option 3: Direct loading**
```go
svc, _ := i18n.New()
svc.AddLocales(embedFS, "locales/en.json")
```

### Missing Key Handling

```go
// Mode 1: Normal (production) - Returns key as-is
svc.SetMode(i18n.ModeNormal)
i18n.T("missing.key") // → "missing.key"

// Mode 2: Strict (dev/CI) - Panics on missing key
svc.SetMode(i18n.ModeStrict)
i18n.T("missing.key") // → panic

// Mode 3: Collect (QA) - Dispatches events, returns [key]
svc.SetMode(i18n.ModeCollect)
i18n.OnMissingKey(func(mk i18n.MissingKey) {
    log.Printf("missing: %s in %s at %s:%d", 
        mk.Key, mk.Lang, mk.File, mk.Line)
})
i18n.T("missing.key") // → "[missing.key]"

// Multiple handlers
svc.SetMissingKeyHandlers(handler1, handler2)
svc.AddMissingKeyHandler(handler3)
svc.ClearMissingKeyHandlers()
```

---

## Semantic intent system

### Intent Functions

| Function | Alias | Purpose | Example |
|----------|-------|---------|---------|
| `_()` | — | Simple gettext-style lookup | `_("cli.success")` |
| `T()` | `C()` | Compose — semantic intent resolution | `T("core.delete", S("file", path))` |
| `S()` | `Subject()` | Create typed subject with metadata | `S("file", path).Count(3)` |

### Using `_()` — Simple Translation

```go
i18n._("cli.success")                           // "Success"
i18n._("common.error.failed", map[string]any{"Action": "load"}) // "Failed to load"
```

### Using `T()` / `C()` — Compose

Semantic intent resolution returns a `Composed` result with multiple output forms:

```go
result := i18n.T("core.delete", i18n.S("file", path))

result.Question  // "Delete /path/to/file.txt?"
result.Confirm   // "Really delete /path/to/file.txt?"
result.Success   // "File deleted"
result.Failure   // "Failed to delete file"
result.Meta      // IntentMeta{Dangerous: true, Default: "no"}
```

### Using `S()` — Subject Builder

```go
i18n.S("file", "/path/to/file.txt")
i18n.S("commit", commits).Count(len(commits))     // plurality
i18n.S("user", name).Gender("female")              // gendered languages
i18n.S("file", path).Count(3).In("/project")       // chained
```

### Intent Templates

Locale files include intent templates:

```json
{
  "intents": {
    "core.delete": {
      "meta": {
        "type": "action",
        "verb": "delete",
        "dangerous": true,
        "default": "no"
      },
      "question": "Delete {{.Subject}}?",
      "confirm": "Really delete {{.Subject}}?",
      "success": "{{.Subject}} deleted",
      "failure": "Failed to delete {{.Subject}}"
    },
    "core.save": {
      "meta": {
        "type": "action",
        "verb": "save",
        "dangerous": false,
        "default": "yes",
        "supports": ["all", "skip"]
      },
      "question": "Save {{.Count}} {{.Noun}}?",
      "confirm": "Really save {{.Count}} {{.Noun}}?",
      "success": "{{.Count}} {{.Noun}} saved",
      "failure": "Failed to save {{.Noun}}"
    }
  }
}
```

### CLI Integration

```go
// Confirm uses T() internally
confirmed := cli.Confirm("core.delete", i18n.S("file", path))
// Displays: result.Question + localised [y/N] / [j/N]

// Question with options
choice := cli.Question("core.save", i18n.S("changes", 3).Count(3), cli.Options{
    Default: "yes",
    Extra:   []string{"all"},
})
// Displays: "Save 3 changes? [a/y/N]"
```

### Command Path Integration

Commands auto-compose i18n keys from their path:

```go
c.Command("issue/get", core.Command{Action: handler})
// Auto-generates i18n keys:
//   issue.get.label    → "Get Issue"
//   issue.get.progress → "Getting issue..."
//   issue.get.done     → "Issue retrieved"
//   issue.get.fail     → "Failed to get issue"
```

The parent = noun context, child = verb. The compose engine builds human-readable strings from these components automatically.

---

## Grammar engine

### Verb Conjugation

```go
// Past tense
PastTense("delete")   // "deleted"
PastTense("run")      // "ran"
PastTense("commit")  // "committed"

// Gerund (present participle)
Gerund("delete")     // "deleting"
Gerund("run")        // "running"
Gerund("commit")     // "committing"

// Base form
BaseForm("deleted")   // "delete"
BaseForm("ran")      // "run"
BaseForm("committed") // "commit"
```

### Noun Pluralization

```go
// Simple pluralization
Plural("file", 1)    // "file"
Plural("file", 5)    // "files"
Plural("vulnerability", 3) // "vulnerabilities"

// CLDR category-based
PluralForm("file", PluralOne)   // "file"
PluralForm("file", PluralOther) // "files"
PluralForm("file", PluralZero)  // "files" (if defined)

// Count word form (automatic pluralization)
countWordForm("en", "file", 1)  // "file"
countWordForm("en", "file", 5)  // "files"
countWordForm("de", "file", 5)  // "Dateien" (with proper German plural)
```

### Article Selection

```go
// Automatic "a" vs "an" based on vowel sounds
Article("hour")    // "an"
Article("cat")     // "a"
Article("URL")     // "a" (special word)
Article("ID")      // "an" (special word)

// Manual article selection
DefiniteArticle()   // "the"
IndefiniteArticle("hour") // "an"
```

### Agreement Rules

```go
// Subject-verb agreement
Agree(verb, subject)  // Conjugates verb based on subject

// Article-noun agreement
ArticleForNoun(noun, gender) // Selects article based on noun gender
```

---

## Reversal package (GrammarImprint)

The `reversal/` package provides linguistic fingerprinting — deterministic, one-way, semantic-preserving content analysis. Scanning a document produces a fingerprint of its grammatical structure without reading the content itself. The network classifies content by grammar, not meaning.

GrammarImprint is the Analysis layer (Layer 6) primitive, enabling the network to operate on content without reading it — consent-preserving semantic analysis.

### Tokeniser

Converts text to classified token stream:

```go
// Tokenise text
tokens, err := reversal.Tokenise(text, reversal.TokeniseOptions{
    Language: "en",
    Classify: true,
})

// Each token has:
// - Raw: original word form
// - Type: primary classification (TokenVerb, TokenNoun, etc.)
// - AltType: secondary classification (for dual-class words)
// - Confidence: classification confidence (0.0-1.0)
// - AltConf: alternate classification confidence
// - VerbInfo: if Type == TokenVerb
// - NounInfo: if Type == TokenNoun
// - etc.
```

### Dual-Class Disambiguation

Words like "test", "check", "commit", "file" can be both verbs and nouns:

```go
// Tokeniser assigns both classifications with confidence scores
for _, token := range tokens {
    if token.Type == reversal.TokenVerb && token.AltType == reversal.TokenNoun {
        // Dual-class word detected
        fmt.Printf("%s: verb=%v (conf=%v), noun=%v (conf=%v)\n",
            token.Raw, token.VerbInfo.Base, token.Confidence,
            token.NounInfo.Base, token.AltConf)
    }
}
```

**Signal-Based Disambiguation:**

The tokeniser uses multiple signals to determine the primary classification:

1. **Signal 1: Known word lists** — Check against verb/noun grammar tables
2. **Signal 2: Auxiliary verbs** — "do", "does", "did", "will", "would", etc.
3. **Signal 3: Contextual patterns** — Position in sentence, surrounding words
4. **Signal 4: Verb saturation** — If a confident verb exists, subsequent ambiguous words are likely nouns
5. **Signal 5: Clause boundaries** — Reset disambiguation at punctuation or conjunctions

**Example from REVIEW.md:**
> "The **test** passed and we should **commit** the fix"

Here "passed" is a confident verb, so Signal 5 would push "commit" toward noun — but "commit" is actually a verb in the second clause. The fix: only scan tokens within the same clause, where clause boundaries are punctuation tokens or coordinating conjunctions ("and", "or", "but").

### GrammarImprint Calculation

```go
// Create imprint from tokens
imprint := reversal.NewImprint(tokens)

// Imprint contains:
imprint.VerbDistribution   // map["delete" -> 0.5, "run" -> 0.3]
imprint.TenseDistribution  // map["past" -> 0.6, "gerund" -> 0.4]
imprint.NounDistribution   // map["file" -> 0.8, "commit" -> 0.2]
imprint.PluralRatio        // 0.25 (25% of nouns are plural)
imprint.DomainVocabulary   // map["word" -> 5] (word category counts)
imprint.ArticleUsage       // map["definite" -> 0.4, "indefinite" -> 0.6]
imprint.PunctuationPattern // map["label" -> 0.1, "question" -> 0.2]
imprint.TokenCount         // 42
imprint.UniqueVerbs        // 15
imprint.UniqueNouns        // 20
```

### Multiplier (Data Augmentation)

Deterministic training data augmentation from grammar tables:

```go
// Augment text with grammar variations
augmented := reversal.Multiply(text, reversal.MultiplierOptions{
    Language: "en",
    Factor:   5,  // Create 5x variations
})

// Returns multiple versions of the text with:
// - Verb tense variations
// - Noun plural variations
// - Synonym substitutions
// - Article variations
```

**Use Case:** Zero-API-calls training data generation for ML models.

### Anomaly Detection

```go
// Detect linguistic anomalies
anomalies := reversal.DetectAnomalies(tokens, reversal.AnomalyOptions{
    Language: "en",
    Threshold: 0.8,
})

// Returns tokens with low confidence or unexpected patterns
```

### Roundtrip Verification

```go
// Verify forward → reverse → forward produces same result
err := reversal.VerifyRoundtrip(verb, tense)
```

---

## Calibration and classification

### Calibration

Compare classification results from two models:

```go
// Calibrate domains
stats := i18n.CalibrateDomains(samples, i18n.CalibrateOptions{
    ModelA: model1,
    ModelB: model2,
})

// Stats includes:
// - Total samples
// - Agreed count
// - Agreement rate
// - Confusion matrix
// - Accuracy vs ground truth
// - Duration per model
```

### Classification

Classify text by domain:

```go
// Classify corpus
domain, confidence := i18n.ClassifyText(text)

// Classify with options
result := i18n.ClassifyCorpus(texts, i18n.ClassifyOptions{
    Threshold: 0.7,
    TopN:     3,
})

// Returns classification statistics
stats.Total        // Total texts classified
stats.Skipped      // Malformed or missing prompt field
stats.ByDomain     // map[domain]count
stats.Duration    // Total processing time
stats.PromptsPerSec // Throughput
```

---

## Configuration

### Service Options

```go
svc, err := i18n.New(
    i18n.WithLanguage("en-GB"),           // Initial language
    i18n.WithFallback("en"),              // Fallback for missing translations
    i18n.WithFormality(i18n.FormalityNeutral), // Default formality
    i18n.WithLocation("workspace"),       // Default location context
    i18n.WithMode(i18n.ModeNormal),        // Missing key mode
    i18n.WithDebug(false),                 // Debug mode
    i18n.WithDefaultHandlers(),            // Add built-in handlers
    i18n.WithHandlers(customHandler),      // Custom handlers
)
```

### Core Service Options

```go
c, err := core.New(
    core.WithService("i18n", i18n.NewCoreService(i18n.ServiceOptions{
        Language: "en-GB",
        Fallback: "en",
        Mode:     i18n.ModeNormal,
        Debug:    false,
        ExtraFS: extraFS,  // Additional filesystem sources
    })),
)
```

### Self-Registration Pattern

Same pattern as `i18n.RegisterLocales()`:

```go
// In any package's init()
func init() {
    i18n.RegisterLocales(localeFS, "locales")
}

// Registration stored until i18n.Init() runs
// Build tags control which packages are imported → which locales are loaded
```

**Example:** `core build -tags php` variant only loads PHP-related translations. Lighter binary, smaller attack surface.

---

## File structure

```
go-i18n/
├── go/
│   ├── i18n.go              # Package-level functions (T, Translate, Raw, Compose, SetLanguage, etc.)
│   ├── types.go             # Core types (Message, Subject, Intent, Composed, Mode, Formality, PluralCategory, GrammaticalGender, etc.)
│   ├── service.go           # Service struct + constructors + main methods
│   ├── loader.go            # JSON loading, flattening, FSLoader, LocaleProvider interface
│   ├── handler.go           # KeyHandler interface + built-in handlers (Label, Progress, Count, Done, Fail, Numeric)
│   ├── context.go           # TranslationContext + context management
│   ├── hooks.go             # MissingKey event + callback management
│   ├── grammar.go           # Verb conjugation, noun pluralization, article selection, agreement
│   ├── language.go          # Language detection, BCP47 parsing, TextDirection
│   ├── localise.go          # Translation lookup, template rendering, interpolation
│   ├── compose.go           # Intent composition (T, C, S, ComposeIntent)
│   ├── numbers.go           # Number formatting (grouping, decimals, currency)
│   ├── time.go              # Time/date formatting (locale-aware)
│   ├── classify.go          # Language classification
│   ├── calibrate.go         # Calibration scoring and comparison
│   ├── validate.go          # Translation validation
│   ├── debug.go             # Debug mode implementation
│   ├── state.go             # Service state management
│   ├── core_service.go      # Core framework integration
│   ├── default_service.go   # Default service fallback
│   │
│   ├── reversal/            # GrammarImprint subpackage
│   │   ├── tokeniser.go       # Text tokenisation with classification
│   │   ├── imprint.go         # GrammarImprint calculation
│   │   ├── multiplier.go      # Deterministic data augmentation
│   │   ├── anomaly.go         # Anomaly detection
│   │   ├── reference.go       # Reference implementation
│   │   ├── roundtrip_test.go   # Roundtrip verification tests
│   │   └── ...
│   │
│   ├── integration/         # Integration tests
│   │   └── ...
│   │
│   ├── locales/             # Embedded locale files
│   │   ├── en.json           # English locale (base, ~11KB)
│   │   └── fr.json           # French locale (~8KB)
│   │
│   └── tests/               # Additional test assets
│
├── docs/
├── KB/
├── GOAL.md                 # SonarQube findings (913 issues across 7 rules)
├── REVIEW.md               # Code review (dual-class disambiguation design)
├── README.md
├── LICENCE
└── go.work
```

---

## Usage examples

### Basic Translation

```go
package main

import (
    "fmt"
    "dappco.re/go/i18n"
)

func main() {
    // Initialize
    svc, err := i18n.New()
    if err != nil {
        panic(err)
    }
    i18n.SetDefault(svc)
    
    // Simple translation
    fmt.Println(i18n.T("greeting.hello", "World"))
    // Output: "Hello, World!"
    
    // With count
    fmt.Println(i18n.T("files.count", 5))
    // Output: "5 files"
}
```

### Grammar Engine

```go
// Verb conjugation
past := i18n.PastTense("delete")   // "deleted"
gerund := i18n.Gerund("run")       // "running"

// Noun pluralization
plural := i18n.Plural("file", 5)    // "files"

// Article selection
article := i18n.Article("hour")    // "an"
```

### Semantic Intent

```go
// Create subject with metadata
subj := i18n.S("file", "config.yaml").Count(3).In("workspace")

// Compose intent
composed := i18n.Compose("core.delete", subj)

fmt.Println(composed.Question)  // "Delete config.yaml?"
fmt.Println(composed.Confirm)   // "Really delete config.yaml?"
fmt.Println(composed.Success)  // "config.yaml deleted"
fmt.Println(composed.Failure)  // "Failed to delete config.yaml"
```

### Built-in Handlers

```go
// Label
label := i18n.T("i18n.label.status")          // "Status:"

// Progress
progress := i18n.T("i18n.progress.build")      // "Building..."

// Count
count := i18n.T("i18n.count.file", 5)          // "5 files"

// Done
done := i18n.T("i18n.done.delete", "file")   // "File deleted"

// Fail
fail := i18n.T("i18n.fail.delete", "file")   // "Failed to delete file"
```

### Number & Time Formatting

```go
// Number formatting
num := i18n.FormatNumber(1234567)          // "1,234,567" (en)
num = i18n.FormatNumber(1234567)          // "1.234.567" (de)

// Currency
currency := i18n.FormatCurrency(1234.56, "USD") // "$1,234.56"

// Time
timeStr := i18n.FormatTime(time.Now())     // "Jun 17, 2026, 3:45:30 PM" (en)

// Date
dateStr := i18n.FormatDate(time.Now())     // "06/17/2026" (en-US)
```

### Custom Locale

```go
// Create service with custom loader
loader := i18n.NewFSLoader(os.DirFS("locales"))
svc, err := i18n.NewWithLoader(loader)

// Add custom locale provider
i18n.RegisterLocaleProvider(&myProvider{})

// Or embed locales
//go:embed locales/*.json
var localeFS embed.FS

svc.AddLocales(localeFS, "locales")
```

### Missing Key Handling

```go
// Collect missing keys for QA
svc.SetMode(i18n.ModeCollect)
i18n.OnMissingKey(func(mk i18n.MissingKey) {
    log.Printf("Missing: %s in %s at %s:%d", 
        mk.Key, mk.Lang, mk.File, mk.Line)
})

// Strict mode for development
i18n.SetMode(i18n.ModeStrict)  // Panics on missing keys
```

### GrammarImprint

```go
// Tokenise text
tokens, err := reversal.Tokenise(text, reversal.TokeniseOptions{
    Language: "en",
    Classify: true,
})

// Create grammar imprint
imprint := reversal.NewImprint(tokens)

// Use for classification
fmt.Printf("Verb distribution: %v\n", imprint.VerbDistribution)
fmt.Printf("Noun distribution: %v\n", imprint.NounDistribution)
fmt.Printf("Plural ratio: %.2f\n", imprint.PluralRatio)
```

### Multi-Language Support

```go
// Set language
svc.SetLanguage("fr")

// Get current language
lang := svc.Language()  // "fr"

// List available languages
langs := svc.AvailableLanguages()  // ["en", "fr", ...]

// RTL detection
if svc.Direction() == i18n.DirRTL {
    // Right-to-left language (Arabic, Hebrew, Persian)
}
```

---

## Advanced features

### Dual-Class Disambiguation

```go
// Tokenise with dual-class detection
tokens, _ := reversal.Tokenise("The test passed and we should commit the fix", ...)

for _, token := range tokens {
    if token.Type == reversal.TokenVerb && token.AltType == reversal.TokenNoun {
        // Dual-class word
        fmt.Printf("%s: verb=%v (conf=%v), noun=%v (conf=%v)\n",
            token.Raw, token.VerbInfo.Base, token.Confidence,
            token.NounInfo.Base, token.AltConf)
    }
}
```

### Custom Handlers

```go
// Implement KeyHandler
type MyHandler struct{}

func (h MyHandler) Match(key string) bool {
    return strings.HasPrefix(key, "myapp.")
}

func (h MyHandler) Handle(key string, args []any, next func() string) string {
    // Custom handling
    if len(args) > 0 {
        return fmt.Sprintf("MyApp: %v", args[0])
    }
    return next()
}

// Register handler
svc, _ := i18n.New(
    i18n.WithDefaultHandlers(),
    i18n.WithHandlers(MyHandler{}),
)
```

### Template Functions

Locale JSON supports Go templates:

```json
{
  "greeting": "Hello, {{.Name}}!",
  "count": "You have {{.Count}} new {{.Plural \"message\"}}"
}
```

```go
// Use with template data
i18n.T("greeting", map[string]any{"Name": "Alice"})
// → "Hello, Alice!"

i18n.T("count", map[string]any{"Count": 5})
// → "You have 5 new messages"
```

### Formality Override

```go
// Set default formality
svc.SetFormality(i18n.FormalityFormal)

// Override per-subject
subj := i18n.S("user", "Dr. Smith").Formal()
composed := svc.Compose("greeting", subj)
// Uses formal address in supported languages
```

---

## Performance and quality

### Benchmarks

The package includes extensive benchmarks:
- `classify_bench_test.go` — Classification performance
- Tokeniser benchmarks — Optimized for high-throughput classification

**Optimizations:**
- Object pooling for tokenise scratch buffers (70% allocation reduction)
- Pre-lowered word caching
- Direct string allocation elision for map lookups
- Sync.Pool for reusable state

### Test Coverage

- **Unit tests:** 50+ test files
- **Integration tests:** Dedicated integration package
- **Benchmark tests:** Performance-critical paths
- **Example tests:** Usage documentation via tests
- **Triplet coverage:** Good/Bad/Ugly patterns throughout

### Quality Assurance

- **SonarQube:** GOAL.md tracks 913 findings across 7 rules
- **Code Review:** REVIEW.md documents dual-class disambiguation design
- **AX Standard:** Full compliance with agents-for-agents documentation

---

## Related knowledge packs

| Package | Knowledge Pack | Relationship |
|---------|----------------|--------------|
| core/go | [../../../../../corego/README.md](../../../../../README.md) | Foundation framework (required dependency) |
| go-blockchain | [../blockchain/](../blockchain/) | Uses go-i18n for localized blockchain messages |
| go-dns | [../dns/](../dns/) | Uses go-i18n for DNS-related translations |
| go-proxy | [../proxy/](../proxy/) | Uses go-i18n for mining proxy UI |
| go-lns | [../lns/](../lns/) | Uses go-i18n for name system messages |

---

## Statistics

- **Total Files:** 80+ Go files
- **Test Files:** 50+ (including integration tests)
- **Lines of Code:** ~50,000 (estimated)
- **Test Coverage:** High (Good/Bad/Ugly triplets)
- **Subpackages:** 2 (reversal, integration)
- **Embedded Locales:** 2 (en, fr)
- **Locale Size:** ~19KB total
- **Dependencies:** 1 (core/go)

### Type System
- **Core Types:** 15+ (Service, Message, Subject, Intent, Composed, etc.)
- **Enums:** 6 (Mode, Formality, TextDirection, PluralCategory, GrammaticalGender, TokenType)
- **Interfaces:** 5 (Translator, Loader, KeyHandler, LocaleProvider, MissingKeyHandler)

### Grammar Features
- **Verb Forms:** 3 (base, past, gerund)
- **Noun Forms:** 6+ (zero, one, two, few, many, other)
- **Article Types:** 3 (indefinite-default, indefinite-vowel, definite)
- **Plural Categories:** 6 (CLDR compliant)
- **Formality Levels:** 3 (neutral, informal, formal)
- **Token Types:** 6 (unknown, verb, noun, article, word, punctuation)

### Reversal Package
- **Files:** 10+ Go files
- **Tokeniser:** Full linguistic analysis
- **Imprint Dimensions:** 8 (verb dist, tense dist, noun dist, plural ratio, domain vocab, article usage, punctuation pattern, token count)
- **Test Binaries:** 2 (i18n.test 13MB, reversal.test 8MB)

---

## Key insights

1. **Two-Layer Architecture** — Separates lightweight interface from full engine, preventing dependency bloat
2. **Semantic Intent** — Compose human-readable strings from verbs, nouns, and intents — not just flat translations
3. **GrammarImprint** — Privacy-preserving content classification by grammar, not content
4. **Dual-Class Disambiguation** — Intelligent resolution of verb/noun ambiguity using multi-signal approach
5. **CLDR Compliance** — Full Unicode plural category support for 100+ languages
6. **AX Standard** — Designed for agents, with comprehensive RFCs and examples

---

## References

- **Primary RFC:** [plans/code/core/go/i18n/RFC.md](../../../../../plans/code/core/go/i18n/RFC.md)
- **Models Spec:** [plans/code/core/go/i18n/RFC.models.md](../../../../../plans/code/core/go/i18n/RFC.models.md)
- **Quality Review:** [REVIEW.md](file:///Users/snider/Code/core/go-i18n/REVIEW.md)
- **Static Analysis:** [GOAL.md](file:///Users/snider/Code/core/go-i18n/GOAL.md)
- **Cross-language Spec:** [plans/code/core/i18n/RFC.md](../../../../../plans/code/core/i18n/RFC.md)

---

*Knowledge pack: go-i18n v1.0.0*
*Last updated: 2026-06-17*
