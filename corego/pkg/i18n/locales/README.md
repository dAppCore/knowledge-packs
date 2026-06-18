# Locale Files for go-i18n

> **Grammar-Aware Internationalization Engine** - Base translation files for the go-i18n package

## Overview

This directory contains JSON locale files that power the **go-i18n grammar engine**. These are not simple key-value translation files - they contain **grammatical data** that enables the engine to:

- Conjugate verbs in different tenses (base, past, gerund)
- Pluralize nouns with CLDR categories (one, other, zero, few, many)
- Handle gender-specific articles (masculine, feminine, neuter)
- Format numbers, dates, and times according to locale conventions
- Select appropriate auxiliary verbs and prepositions

## Available Locales

| Code | Language | Lines | Features |
|------|----------|-------|----------|
| `en` | English | 164 | Full (from go-i18n) |
| `fr` | French | 160 | Full (from go-i18n) |
| `es` | Spanish | 117 | Grammar-aware skeleton |
| `de` | German | 114 | Grammar-aware skeleton |
| `it` | Italian | 114 | Grammar-aware skeleton |
| `pt` | Portuguese | 111 | Grammar-aware skeleton |
| `ru` | Russian | 106 | Grammar-aware skeleton |
| `nl` | Dutch | 109 | Grammar-aware skeleton |
| `sv` | Swedish | 107 | Grammar-aware skeleton |
| `ja` | Japanese | 103 | Grammar-aware skeleton |
| `zh` | Chinese | 105 | Grammar-aware skeleton |
| `pl` | Polish | 114 | Grammar-aware skeleton |
| `ar` | Arabic | 106 | Grammar-aware skeleton |
| `ko` | Korean | 103 | Grammar-aware skeleton |
| `tr` | Turkish | 107 | Grammar-aware skeleton |
| `fi` | Finnish | 109 | Grammar-aware skeleton |
| `id` | Indonesian | 111 | Grammar-aware skeleton |

**Total: 16 languages**

## Structure

Each locale file follows the same structure:

### Top-Level Sections

```json
{
  "gram": { ... },     // Grammar data (verbs, nouns, articles, etc.)
  "prompt": { ... },   // CLI prompts (yes, no, continue, etc.)
  "time": { ... },     // Time formatting with pluralization
  "lang": { ... }      // Language name mappings
}
```

### Grammar Section (`gram`)

| Subsection | Purpose | Example |
|------------|---------|---------|
| `verb` | Verb conjugations | `{ "base": "run", "past": "ran", "gerund": "running" }` |
| `noun` | Noun forms with gender | `{ "one": "file", "other": "files", "gender": "m" }` |
| `article` | Article selection | `{ "indefinite": "a", "definite": "the" }` |
| `word` | Special words/acronyms | `{ "url": "URL", "id": "ID" }` |
| `punct` | Punctuation | `{ "label": ":", "progress": "..." }` |
| `signal` | Grammar signals | Determiners, auxiliaries, infinitives |
| `number` | Number formatting | `{ "thousands": ",", "decimal": "." }` |

### Why This Matters

Unlike flat translation systems, **go-i18n composes grammatically correct output**:

```go
// Instead of just translating "File not found"
// The engine can construct: "The file was not found"
// With proper verb conjugation, article selection, and noun agreement

// For pluralization:
// English: "1 file" vs "5 files"
// Arabic: Different forms for 0, 1, 2, 3-10, 11+
// The engine handles this automatically based on CLDR rules
```

## Usage

```go
// Load a locale
locales := i18n.LoadEmbedded("locales/en.json", "locales/fr.json")

// Set language
c.I18n().SetLanguage("fr")

// Translate with grammar
result := c.I18n().Translate("action.deploy", "server", "production")
// Returns: "Déploiement du serveur en production"
```

## Adding New Languages

To add a new language:

1. **Copy an existing file** as a template (e.g., `cp es.json xx.json`)
2. **Translate all values** maintaining the JSON structure
3. **Add to lang mappings** - Include the language code in the `lang` section
4. **Validate JSON** - Ensure the file is valid JSON
5. **Test** - Verify with the go-i18n grammar engine

### Minimum Viable Locale

A minimal locale file requires:
- `gram.verb` - At least common verbs (be, have, do, go, make)
- `gram.noun` - Common nouns (file, repo, commit, error)
- `gram.article` - Article rules
- `prompt` - Basic CLI prompts
- `time` - Relative time formatting
- `lang` - Language name mappings

## Language-Specific Notes

### English (`en.json`) & French (`fr.json`)
- **Source**: Copied from `core/go-i18n/go/locales/`
- **Status**: Full implementation
- **Features**: Complete verb conjugations, noun genders, signal words

### Romance Languages (`es.json`, `pt.json`, `it.json`)
- **Status**: Skeleton with core grammar
- **Features**: Verb conjugations (present, past, gerund), noun genders (m/f)
- **Notes**: Need expansion for irregular verbs and plural forms

### Germanic Languages (`de.json`, `nl.json`, `sv.json`)
- **Status**: Skeleton with core grammar
- **Features**: Strong/weak verb conjugations, noun genders (m/f/n)
- **Notes**: German needs cases (nominative, accusative, etc.)

### Slavic Languages (`ru.json`)
- **Status**: Skeleton
- **Features**: Basic verb/noun forms
- **Notes**: Russian has complex aspect system (perfective/imperfective)

### East Asian Languages (`ja.json`, `zh.json`, `ko.json`)
- **Status**: Skeleton
- **Features**: Basic verb forms, no articles, no gender
- **Notes**: Chinese/Japanese/Korean don't have verb conjugation in the same way

### Uralic Languages (`fi.json`)
- **Status**: Skeleton
- **Features**: Verb conjugations, noun cases (15+ in Finnish)
- **Notes**: Finnish is agglutinative with complex morphology

### Austronesian Languages (`id.json`)
- **Status**: Skeleton
- **Features**: Reduplication for plurals, no verb conjugation
- **Notes**: Indonesian uses affixation and reduplication

### New Additions
- **Polish (`pl.json`)**: Slavic language with complex cases (7 cases) and aspect system
- **Arabic (`ar.json`)**: Semitic language with root-based morphology, right-to-left script
- **Korean (`ko.json`)**: Agglutinative language with SOV word order
- **Turkish (`tr.json`)**: Agglutinative language with vowel harmony
- **Finnish (`fi.json`)**: Uralic language with 15 noun cases
- **Indonesian (`id.json`)**: Austronesian language with simple grammar, reduplication

## Local Inference Idea

The user mentioned: **"reduce the lang down with local inference"**

This is a fascinating approach! Instead of maintaining complete grammar tables for every language, you could:

1. **Use a base language** (English) as the source of truth
2. **Apply transformation rules** per-language to generate forms
3. **Use ML models** to predict conjugations/pluralizations
4. **Cache results** for performance

**Benefits:**
- Smaller locale files
- Easier maintenance
- Support for more languages
- Adaptive to new words

**Trade-offs:**
- Requires inference infrastructure
- May have lower accuracy
- Latency for first-time translations

## Next Steps

1. ✅ **Created skeleton locales** for 16 languages
2. ⏳ **Expand vocabulary** - Add more verbs, nouns, adjectives
3. ⏳ **Add irregular forms** - Handle exceptions (e.g., "go/went/gone")
4. ⏳ **Add more languages** - Hindi, Thai, Vietnamese, Hebrew, etc.
5. ⏳ **Test with go-i18n** - Validate all locales work with the engine
6. ⏳ **Implement local inference** - Reduce locale size with ML

## See Also

- [go-i18n RFC](../../../../../plans/code/core/go/i18n/RFC.md)
- [go-i18n Repository](file:///Users/snider/Code/core/go-i18n/)
- [CLDR Plural Rules](http://cldr.unicode.org/index/cldr-spec/plural-rules)
