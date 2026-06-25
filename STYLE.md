# Knowledge-pack writing style

Reference material for agents. Calm, factual, terse. Bridge-comms discipline:
**value added only** — if a line doesn't carry information an agent needs, cut it.

## Voice
- UK English (colour, organisation, centre, licence).
- Sentence-case headings.
- No exclamation marks. No emojis anywhere, including in headings.
- No hype, boosterism, or self-congratulation. Drop "battle-tested", "powerful",
  "seamless", "robust", "blazing-fast", "leverage", "utilise", "game-changing",
  "best-in-class", motivational framing, and epigraph quotes that praise the
  document or the system.
- State things plainly. One claim per sentence. Prefer the shorter word.

## Keep, unchanged
- Every fact, version, name, path, module, repo, command, and figure.
- All code blocks, YAML, JSON, and tables — verbatim.
- The structural sections an agent uses: overview/purpose, architecture, package
  structure, components, APIs, CLI, patterns, use cases, related links.
- Frontmatter (it is metadata) — strip hype only from a `description:` value.

## Cut
- Emoji section markers: `## 🎯 Overview` → `## Overview`.
- Epigraph or pull-quote lines that hype the doc or the system.
- "Statistics" / "Quick Stats" flex sections (repo, package, or line counts kept
  for their own sake) and celebratory "Recent Additions" changelogs.
- Marketing adjectives and redundant restatement (say it once).

## Rules
- Tone and filler only. Never change a fact, a path, a code sample, or the
  meaning. If unsure whether something is fact or filler, keep it.
- Do not invent. Do not add content that is not already there.
