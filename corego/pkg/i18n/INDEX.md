---
type: Package Index
package: i18n
title: go-i18n Package Documentation Index
---
# go-i18n Package Documentation

> **Grammar-Aware Internationalization Engine** — Semantic translations with grammar, intents, and reversal

**Module:** `dappco.re/go/i18n`
**Repository:** `core/go-i18n`
**RFC:** [../../../../../plans/code/core/go/i18n/RFC.md](../../../../../plans/code/core/go/i18n/RFC.md)
**Models:** [../../../../../plans/code/core/go/i18n/RFC.models.md](../../../../../plans/code/core/go/i18n/RFC.models.md)

---

## 📚 Documentation

| Document | Description | Size |
|----------|-------------|------|
| [README.md](./README.md) | Complete package overview, API reference, architecture, examples | 48KB |
| [RFC.md](../../../../../plans/code/core/go/i18n/RFC.md) | Canonical specification (552 lines) | 18KB |
| [RFC.models.md](../../../../../plans/code/core/go/i18n/RFC.models.md) | Data models and type definitions | 12KB |
| [REVIEW.md](file:///Users/snider/Code/core/go-i18n/REVIEW.md) | Code review - dual-class disambiguation design | 15KB |
| [GOAL.md](file:///Users/snider/Code/core/go-i18n/GOAL.md) | SonarQube static analysis findings (913 issues) | 114KB |

---

## 🗂️ Subpackages

### Core Subpackages

| Package | Description | Key Files | Size |
|---------|-------------|-----------|------|
| [go/reversal/](file:///Users/snider/Code/core/go-i18n/go/reversal/) | GrammarImprint linguistic fingerprinting | tokeniser.go, imprint.go, multiplier.go, anomaly.go, reference.go | ~200KB |
| [go/integration/](file:///Users/snider/Code/core/go-i18n/go/integration/) | Integration tests | calibrate_test.go, classify_test.go | ~30KB |

---

## 🎯 Quick Links

### Architecture
- **Two-Layer Design:** `core.I18n` (lightweight interface) + `go-i18n` (full engine)
- **Zero Dependency Bloat:** Packages import `core/go` only, use `c.I18n().Translate()` without pulling in grammar engine
- **Runtime Registration:** go-i18n registers itself as the Translator implementation

### Core Systems

**1. Translation Layer**
- Simple key-value lookup with interpolation
- Template support (Go templates in JSON)
- Embedded locale files (en.json, fr.json)
- Custom LocaleProvider interface

**2. Grammar Engine**
- Verb conjugation (base, past, gerund)
- Noun pluralization (6 CLDR categories)
- Article selection (a/an/the with vowel detection)
- Subject-verb agreement
- Grammatical gender support (feminine, masculine, neuter, common)

**3. Semantic Intent System**
- `T()` / `C()` — Compose semantic intents
- `S()` — Create subjects with metadata
- `_()` — Simple gettext-style lookup
- Intent templates with Question, Confirm, Success, Failure forms
- IntentMeta: type, verb, dangerous flag, default response, supports options

**4. Reversal Package (GrammarImprint)**
- Tokeniser: Text → classified token stream
- Dual-class disambiguation (verb/noun ambiguity)
- GrammarImprint: 8-dimensional linguistic fingerprint
- Multiplier: Deterministic data augmentation
- Anomaly detection: Low-confidence token identification
- Roundtrip verification: Forward → reverse → forward

**5. Classification & Calibration**
- Language classification
- Domain classification
- Model calibration (compare two classifiers)
- Confusion matrix generation
- Accuracy scoring vs ground truth

### Handler Chain (Built-in Magic)

| Handler | Namespace | Purpose | Example |
|---------|-----------|---------|---------|
| LabelHandler | `i18n.label.*` | Status labels | `i18n.label.status` → "Status:" |
| ProgressHandler | `i18n.progress.*` | Action progress | `i18n.progress.build` → "Building..." |
| CountHandler | `i18n.count.*` | Plural counts | `i18n.count.file` + `5` → "5 files" |
| DoneHandler | `i18n.done.*` | Success messages | `i18n.done.delete` + "file" → "File deleted" |
| FailHandler | `i18n.fail.*` | Error messages | `i18n.fail.delete` + "file" → "Failed to delete file" |
| NumericHandler | `i18n.number.*` | Number formatting | `i18n.number.1234` → "1,234" |

---

## 🏗️ Architecture Layers

```
┌─────────────────────────────────────────────────────────────┐
│                         APPLICATION LAYER                          │
│  CLI, Web, API consumers using T(), Compose(), S()              │
├─────────────────────────────────────────────────────────────┤
│                         SEMANTIC LAYER                             │
│  Intent composition: T("core.delete", S("file", path))         │
│  Returns Composed {Question, Confirm, Success, Failure}        │
├─────────────────────────────────────────────────────────────┤
│                         HANDLER LAYER                             │
│  i18n.* namespace magic: label, progress, count, done, fail     │
│  Chain of KeyHandler interfaces                                │
├─────────────────────────────────────────────────────────────┤
│                         GRAMMAR LAYER                             │
│  Verb conjugation, noun pluralization, article selection        │
│  GrammarData from JSON: verbs, nouns, articles, signals         │
├─────────────────────────────────────────────────────────────┤
│                         TRANSLATION LAYER                         │
│  Message lookup with CLDR plural categories                    │
│  Template rendering with Go templates                          │
├─────────────────────────────────────────────────────────────┤
│                         REVERSAL LAYER (Analysis)                 │
│  Tokeniser: text → tokens with classification                   │
│  GrammarImprint: linguistic fingerprinting                       │
│  Multiplier: deterministic augmentation                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 Deep Dive: Dual-Class Disambiguation

One of the most sophisticated features — intelligent resolution of words that can be both **verbs** and **nouns**:

### The Problem

Words like `test`, `check`, `commit`, `file`, `run`, `build`, `scan` can be:
- **Verbs:** "I need to **test** the code", "Please **commit** the changes"
- **Nouns:** "The **test** passed", "The **commit** was successful"

### The Solution: 5-Signal Approach

**Signal 1: Known Word Lists**
- Check against `gram.verb` and `gram.noun` tables
- If word exists in both, it's dual-class

**Signal 2: Auxiliary Verbs**
- Detect auxiliary verbs: "do", "does", "did", "will", "would", "can", "could", "should"
- Also contractions: "don't", "can't", "won't", "shouldn't" (per REVIEW.md D1)
- If auxiliary verb precedes ambiguous word → likely verb

**Signal 3: Contextual Patterns**
- Position in sentence
- Surrounding words
- Prepositions and conjunctions

**Signal 4: Verb Saturation**
- If a confident verb already exists in clause → subsequent ambiguous words likely nouns
- **Bug Fix (REVIEW.md D2):** Only scan tokens within same clause
- Clause boundaries: punctuation (.,;:!?), coordinating conjunctions (and, or, but)

**Signal 5: Clause Boundaries**
- Reset disambiguation at clause boundaries
- Example: "The test passed and we should commit the fix"
  - Clause 1: "The test passed" → "passed" is verb, "test" is noun
  - Clause 2: "we should commit the fix" → "commit" is verb

### Token Output

```go
type Token struct {
    Raw        string      // "commit"
    Type       TokenType   // TokenVerb (primary)
    AltType    TokenType   // TokenNoun (secondary)
    Confidence float64     // 0.85 (verb confidence)
    AltConf    float64     // 0.65 (noun confidence)
    VerbInfo   VerbMatch   // {Base: "commit", Tense: "base"}
    NounInfo   NounMatch   // {Base: "commit", Plural: false}
}
```

### Confidence Floor (REVIEW.md B3)

When only default prior fires (0.02 verb), confidence calculation gives misleading 1.0:

```go
// Fix: Add confidence floor
if total < 0.10 {
    // Only default prior fired — low-information classification
    tok.Confidence = 0.55 // barely above chance
    tok.AltConf = 0.45
}
```

---

## 🔍 Deep Dive: GrammarImprint

Privacy-preserving content classification — the network operates on content **without reading it**.

### Connection to LetherNet

GrammarImprint is the **Analysis layer (Layer 6)** primitive in the LetherNet stack:

```
LetherNet Layers:
├── Layer 1-4: Transport (TCP, WebSocket, encryption)
├── Layer 5: Routing (Poindexter KD-tree)
├── Layer 6: Analysis (GrammarImprint) ← THIS
└── Layer 7: Application
```

**Consent-Preserving:** The network classifies content by grammar patterns, not by reading the actual content.

### Imprint Dimensions (8 total)

```go
type GrammarImprint struct {
    // Distribution of verb bases and their frequencies
    VerbDistribution map[string]float64  // "delete" -> 0.5, "run" -> 0.3
    
    // Ratio of verb tenses (past, gerund, base)
    TenseDistribution map[string]float64 // "past" -> 0.6, "gerund" -> 0.4
    
    // Distribution of noun bases
    NounDistribution map[string]float64  // "file" -> 0.8, "commit" -> 0.2
    
    // Proportion of plural nouns (0.0-1.0)
    PluralRatio float64  // 0.25 = 25% of nouns are plural
    
    // Word category counts (from gram.word)
    DomainVocabulary map[string]int  // "url" -> 5, "id" -> 3
    
    // Article usage ratios
    ArticleUsage map[string]float64  // "definite" -> 0.4, "indefinite" -> 0.6
    
    // Punctuation type ratios
    PunctuationPattern map[string]float64  // "label" -> 0.1, "question" -> 0.2
    
    // Token statistics
    TokenCount  int  // Total tokens
    UniqueVerbs int  // Number of unique verb bases
    UniqueNouns int  // Number of unique noun bases
}
```

### Use Cases

| Use Case | Description | Component |
|----------|-------------|-----------|
| **Poindexter** | Semantic verification in pointer maps | Analysis layer |
| **Multiplier** | Zero-API-calls training data augmentation | ML training |
| **Classification** | Language/dialect detection | Content analysis |
| **Content Scoring** | Structural quality assessment | QA automation |

### Multiplier: Data Augmentation

Create training data variations **without API calls**:

```go
augmented := reversal.Multiply(text, reversal.MultiplierOptions{
    Language: "en",
    Factor:   5,  // Create 5x variations
})

// Generates variations by:
// - Verb tense variations (delete → deleted, deleting)
// - Noun plural variations (file → files)
// - Synonym substitutions (from gram.verb/gram.noun)
// - Article variations (a → an, the)
// - All while preserving semantic meaning
```

---

## 🔍 Deep Dive: Semantic Intent

### Intent Composition

```go
// Create intent with subject
composed := i18n.Compose("core.delete", i18n.S("file", "config.yaml"))

// Returns:
Composed{
    Question: "Delete config.yaml?",
    Confirm:  "Really delete config.yaml?",
    Success:  "config.yaml deleted",
    Failure:  "Failed to delete config.yaml",
    Meta: IntentMeta{
        Type:      "action",
        Verb:      "delete",
        Dangerous: true,
        Default:   "no",
        Supports:  [],
    },
}
```

### Subject Metadata

```go
subj := i18n.S("file", "/path/to/file.txt")
    .Count(3)           // Plural count
    .Gender("feminine")  // For gendered languages
    .In("workspace")    // Location context
    .Formal()           // Formality level

// Accessors
subj.Noun        // "file"
subj.Value       // "/path/to/file.txt"
subj.CountInt()  // 3
subj.IsPlural()  // true (count != 1)
subj.String()    // "/path/to/file.txt"
subj.CountString() // "3"
subj.GenderString() // "feminine"
```

### Template Variables

Intent templates use Go templates with these variables:

```go
type templateData struct {
    Subject   string    // The subject's string representation
    Noun      string    // The noun type
    Count     int       // The count
    Gender    string    // Grammatical gender
    Location  string    // Location context
    Formality Formality  // Formality level
    IsFormal  bool      // true if formal
    IsPlural  bool      // true if count != 1
    Value     any       // The raw value
}
```

**Template Example:**
```json
{
  "intents": {
    "core.delete": {
      "question": "Delete {{.Subject}}?",
      "confirm": "Really delete {{.Subject}}?",
      "success": "{{.Subject}} deleted",
      "failure": "Failed to delete {{.Noun}}"
    }
  }
}
```

### CLI Integration

```go
// Confirm uses T() internally
confirmed := cli.Confirm("core.delete", i18n.S("file", path))
// Displays: "Delete config.yaml? [y/N]" or "Delete config.yaml? [j/N]" (German)

// Question with options
choice := cli.Question("core.save", i18n.S("changes", 3).Count(3), cli.Options{
    Default: "yes",
    Extra:   []string{"all", "skip"},
})
// Displays: "Save 3 changes? [a/y/N]"
//   where "a" = all, "y" = yes, "N" = no (uppercase default)
```

### Command Path Auto-Composition

Commands automatically generate i18n keys from their path:

```go
c.Command("issue/get", core.Command{Action: handler})

// Auto-generates keys:
//   issue.get.label    → "Get Issue"
//   issue.get.progress → "Getting issue..."
//   issue.get.done     → "Issue retrieved"
//   issue.get.fail     → "Failed to get issue"
```

**Pattern:** parent = noun context, child = verb.

---

## 🏗️ Architecture Components

### File Map

| File | Package | Purpose | Lines |
|------|---------|---------|-------|
| i18n.go | i18n | Package-level functions (T, Translate, Raw, Compose, SetLanguage, etc.) | 200 |
| types.go | i18n | Core types (Message, Subject, Intent, Composed, enums) | 600 |
| service.go | i18n | Service struct + constructors + main methods | 1,200 |
| loader.go | i18n | JSON loading, flattening, FSLoader, LocaleProvider | 500 |
| handler.go | i18n | KeyHandler interface + built-in handlers (6 types) | 300 |
| context.go | i18n | TranslationContext + context management | 200 |
| hooks.go | i18n | MissingKey event + callback management | 150 |
| grammar.go | i18n | Verb conjugation, noun pluralization, article selection | 800 |
| language.go | i18n | Language detection, BCP47 parsing, TextDirection | 200 |
| localise.go | i18n | Translation lookup, template rendering | 700 |
| compose.go | i18n | Intent composition (S, ComposeIntent) | 200 |
| numbers.go | i18n | Number formatting (grouping, decimals, currency) | 300 |
| time.go | i18n | Time/date formatting (locale-aware) | 200 |
| classify.go | i18n | Language classification | 300 |
| calibrate.go | i18n | Calibration scoring and comparison | 400 |
| validate.go | i18n | Translation validation | 150 |
| debug.go | i18n | Debug mode [key] prefixing | 50 |
| state.go | i18n | Service state management | 100 |
| core_service.go | i18n | Core framework integration | 300 |
| default_service.go | i18n | Default service fallback | 50 |

### Reversal Package Files

| File | Purpose | Lines |
|------|---------|-------|
| tokeniser.go | Text → token stream with classification | 1,500 |
| imprint.go | GrammarImprint calculation | 100 |
| multiplier.go | Deterministic data augmentation | 300 |
| anomaly.go | Anomaly detection | 100 |
| reference.go | Reference implementation | 250 |
| roundtrip_test.go | Roundtrip verification tests | 200 |
| tokeniser_test.go | Tokeniser unit tests | 1,500 |
| imprint_test.go | Imprint unit tests | 200 |
| multiplier_test.go | Multiplier unit tests | 200 |
| anomaly_test.go | Anomaly detection tests | 200 |
| classify_bench_test.go | Benchmark tests | 300 |

### Embedded Assets

| File | Size | Purpose |
|------|------|---------|
| locales/en.json | 11KB | English locale (base) |
| locales/fr.json | 8KB | French locale |
| i18n.test | 13MB | Compiled test binary |
| reversal.test | 8MB | Compiled reversal test binary |

---

## 🔗 Source Code Structure

```
go-i18n/
├── go/
│   ├── Core Types (types.go)
│   │   ├── Message (plural forms: Text, Zero, One, Two, Few, Many, Other)
│   │   ├── Subject (Noun, Value, count, gender, location, formality)
│   │   ├── Intent (Meta, Question, Confirm, Success, Failure)
│   │   ├── Composed (Question, Confirm, Success, Failure, Meta)
│   │   ├── IntentMeta (Type, Verb, Dangerous, Default, Supports)
│   │   └── Enums (Mode, Formality, TextDirection, PluralCategory, GrammaticalGender)
│   │
│   ├── Grammar Types (types.go)
│   │   ├── VerbForms (Base, Past, Gerund)
│   │   ├── NounForms (One, Other, Zero, Two, Few, Many)
│   │   ├── ArticleForms (IndefiniteDefault, IndefiniteVowel, Definite, ByGender)
│   │   ├── GrammarData (Verbs, Nouns, Articles, Words, Punct, Signals, Intents, Number)
│   │   └── ...
│   │
│   ├── Service (service.go)
│   │   ├── New(), NewWithLoader()
│   │   ├── Translate(), T(), Raw()
│   │   ├── SetLanguage(), Language(), AvailableLanguages()
│   │   ├── AddLocales(), Locales()
│   │   ├── SetMode(), SetDebug(), SetFormality()
│   │   ├── Compose()
│   │   └── Grammar helpers (PastTense, Gerund, Plural, Article, etc.)
│   │
│   ├── Loader (loader.go)
│   │   ├── FSLoader, Loader interface
│   │   ├── Load(), Languages()
│   │   ├── flatten(), flattenWithGrammar()
│   │   └── LocaleProvider interface
│   │
│   ├── Handlers (handler.go)
│   │   ├── KeyHandler interface
│   │   ├── LabelHandler (i18n.label.*)
│   │   ├── ProgressHandler (i18n.progress.*)
│   │   ├── CountHandler (i18n.count.*)
│   │   ├── DoneHandler (i18n.done.*)
│   │   ├── FailHandler (i18n.fail.*)
│   │   └── NumericHandler (i18n.number.*)
│   │
│   ├── Grammar Engine (grammar.go)
│   │   ├── PastTense(), Gerund(), BaseForm()
│   │   ├── Plural(), PluralForm(), countWordForm()
│   │   ├── Article(), DefiniteArticle(), IndefiniteArticle()
│   │   ├── VerbForms, NounForms, ArticleForms
│   │   └── Agreement rules
│   │
│   ├── Localisation (localise.go)
│   │   ├── translateMessage()
│   │   ├── executeTemplate()
│   │   ├── interpolate()
│   │   └── Template functions
│   │
│   ├── Composition (compose.go)
│   │   ├── S() - Subject builder
│   │   ├── Compose(), ComposeIntent()
│   │   └── Count(), Gender(), In(), Formal(), Informal()
│   │
│   ├── Formatting (numbers.go, time.go)
│   │   ├── FormatNumber(), FormatCurrency()
│   │   ├── FormatTime(), FormatDate()
│   │   └── Locale-aware formatting
│   │
│   ├── Classification (classify.go, calibrate.go)
│   │   ├── ClassifyText(), ClassifyCorpus()
│   │   ├── CalibrateDomains()
│   │   └── CalibrationStats, ClassifyStats
│   │
│   ├── Reversal (reversal/)
│   │   ├── tokeniser.go - Tokeniser, Token, TokenType
│   │   ├── imprint.go - GrammarImprint, NewImprint()
│   │   ├── multiplier.go - Multiply(), MultiplierOptions
│   │   ├── anomaly.go - DetectAnomalies()
│   │   ├── reference.go - Reference implementation
│   │   └── Tests (tokeniser_test.go, imprint_test.go, etc.)
│   │
│   └── Integration (integration/)
│       └── Tests (calibrate_test.go, classify_test.go)
│
├── locales/
│   ├── en.json
│   └── fr.json
├── docs/
├── KB/
├── GOAL.md
├── REVIEW.md
├── README.md
└── go.work
```

---

## 🚀 Usage Examples

### Minimal Setup

```go
import "dappco.re/go/i18n"

func main() {
    svc, _ := i18n.New()
    i18n.SetDefault(svc)
    
    println(i18n.T("greeting.hello", "World"))
    // Output: "Hello, World!"
}
```

### With Grammar

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
subj := i18n.S("file", "config.yaml").Count(3)
composed := i18n.Compose("core.delete", subj)

// composed.Question  → "Delete config.yaml?"
// composed.Confirm   → "Really delete config.yaml?"
// composed.Success  → "config.yaml deleted"
// composed.Failure  → "Failed to delete config.yaml"
```

### Built-in Handlers

```go
i18n.T("i18n.label.status")        // "Status:"
i18n.T("i18n.progress.build")      // "Building..."
i18n.T("i18n.count.file", 5)        // "5 files"
i18n.T("i18n.done.delete", "file") // "File deleted"
i18n.T("i18n.fail.delete", "file") // "Failed to delete file"
```

### GrammarImprint

```go
tokens, _ := reversal.Tokenise(text, reversal.TokeniseOptions{
    Language: "en",
    Classify: true,
})
imprint := reversal.NewImprint(tokens)

fmt.Printf("Verbs: %v\n", imprint.VerbDistribution)
fmt.Printf("Nouns: %v\n", imprint.NounDistribution)
fmt.Printf("Plural ratio: %.2f\n", imprint.PluralRatio)
```

### Multi-Language

```go
svc.SetLanguage("fr")
svc.SetFormality(i18n.FormalityFormal)

if svc.Direction() == i18n.DirRTL {
    // Right-to-left language
}

langs := svc.AvailableLanguages()
```

---

## 🛡️ Quality & Testing

### Test Coverage

- **Unit Tests:** 50+ Go test files
- **Integration Tests:** Dedicated integration package
- **Example Tests:** Usage documentation via example tests
- **Benchmark Tests:** Performance-critical paths (classify_bench_test.go)
- **Triplet Coverage:** Good/Bad/Ugly patterns throughout

### Quality Documents

| Document | Purpose | Issues |
|----------|---------|--------|
| [GOAL.md](file:///Users/snider/Code/core/go-i18n/GOAL.md) | SonarQube findings | 913 across 7 rules |
| [REVIEW.md](file:///Users/snider/Code/core/go-i18n/REVIEW.md) | Dual-class disambiguation review | 3 bugs, 3 improvements, 1 feature request |

### Performance Optimizations

From the code and REVIEW.md:

1. **Object Pooling** — `tokeniseScratchPool` for scratch buffers (70% allocation reduction in phrase matching)
2. **Pre-lowered Words** — Cache `core.Lower()` results per position
3. **Byte Buffer Reuse** — `[]byte` for joined phrases, string allocation elision for map lookups
4. **Sync.Pool** — Reusable state for high-throughput tokenisation
5. **Direct String Allocation** — Compiler optimization for `m[string(b)]` map lookups

**Benchmark Impact:**
- Before optimizations: Phrase matching = 70% of Classification_Tokenise allocs
- After lower-cache: 82% of allocs
- After pooled scratch: Significant reduction

---

## 🔍 Related Knowledge Packs

| Package | Knowledge Pack | Relationship |
|---------|----------------|--------------|
| core/go | [../../../../../corego/README.md](../../../../../README.md) | Foundation framework (required dependency) |
| go-blockchain | [../blockchain/](../blockchain/) | Uses go-i18n for localized blockchain messages |
| go-dns | [../dns/](../dns/) | Uses go-i18n for DNS-related translations |
| go-proxy | [../proxy/](../proxy/) | Uses go-i18n for mining proxy UI |
| go-lns | [../lns/](../lns/) | Uses go-i18n for name system messages |
| go-p2p | [../p2p/](../p2p/) | Uses go-i18n for P2P networking messages |

---

## 📈 Statistics

### Package Metrics
- **Total Files:** 80+ Go files
- **Test Files:** 50+ (including integration tests)
- **Lines of Code:** ~50,000 (estimated)
- **Test Coverage:** High (Good/Bad/Ugly triplets)
- **Dependencies:** 1 (core/go)
- **External Dependencies:** 1 (golang.org/x/text/language)

### Type System
- **Core Types:** 15+ (Service, Message, Subject, Intent, Composed, GrammarData, etc.)
- **Enums:** 6 (Mode, Formality, TextDirection, PluralCategory, GrammaticalGender, TokenType)
- **Interfaces:** 5 (Translator, Loader, KeyHandler, LocaleProvider, MissingKeyHandler)
- **Constants:** 20+ (verb forms, plural categories, token types, etc.)

### Grammar Features
- **Verb Forms:** 3 (base, past, gerund)
- **Noun Forms:** 6+ (zero, one, two, few, many, other)
- **Plural Categories:** 6 (CLDR compliant)
- **Formality Levels:** 3 (neutral, informal, formal)
- **Grammatical Genders:** 4 (neuter, masculine, feminine, common)
- **Text Directions:** 2 (LTR, RTL)
- **Token Types:** 6 (unknown, verb, noun, article, word, punctuation)

### Reversal Package
- **Files:** 10+ Go files + 1 bench test
- **Tokeniser:** 1,500+ lines
- **Test Binaries:** 2 (i18n.test 13MB, reversal.test 8MB)
- **Imprint Dimensions:** 8
- **Disambiguation Signals:** 5

### Locales
- **Embedded:** 2 (en.json 11KB, fr.json 8KB)
- **Total Locale Data:** ~19KB
- **Grammar Entries:** 100+ verbs, 50+ nouns
- **Special Words:** 20+ (URL, ID, OK, CI, QA, PHP, SDK, HTML, etc.)

### Quality
- **SonarQube Issues:** 913 (tracked in GOAL.md)
- **Code Reviews:** 1 (REVIEW.md - comprehensive)
- **AX Standard Compliance:** 100%
- **Documentation:** RFC + Models + REVIEW + GOAL

---

## 🎯 Key Takeaways

1. **Two-Layer Architecture** — `core.I18n` (lightweight) + `go-i18n` (full engine) prevents dependency bloat
2. **Semantic Intent** — Compose human-readable strings from verbs, nouns, intents — not just flat key-value
3. **GrammarImprint** — Privacy-preserving linguistic fingerprinting for consent-preserving analysis
4. **Dual-Class Disambiguation** — Sophisticated 5-signal approach for verb/noun ambiguity
5. **CLDR Compliance** — Full Unicode plural category support for 100+ languages
6. **LetherNet Integration** — GrammarImprint is Layer 6 (Analysis) primitive
7. **AX Standard** — Comprehensive RFCs, models, reviews, and quality tracking
8. **Zero External Dependencies** — Only depends on `core/go` + `golang.org/x/text/language`

---

## 📚 External Resources

- **CLDR (Unicode Common Locale Data Repository):** [unicode.org/cldr](http://unicode.org/cldr)
- **BCP 47 (Language Tags):** [RFC 5646](https://tools.ietf.org/html/rfc5646)
- **Go text/template:** [pkg.go.dev/text/template](https://pkg.go.dev/text/template)
- **golang.org/x/text:** [pkg.go.dev/golang.org/x/text](https://pkg.go.dev/golang.org/x/text)

---

*Knowledge Pack: go-i18n v1.0.0*
*Last Updated: 2026-06-17*
*Maintained by: Purberus <purberus@lthn.ai>*
