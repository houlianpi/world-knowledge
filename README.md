# World Knowledge

Agent-friendly knowledge base for macOS UI automation.

Designed for [goal-driven-automation](https://github.com/houlianpi/goal-driven-automation) — an AI-powered macOS automation testing framework.

## Overview

```
world-knowledge/
├── SCHEMA.md     # Constraints & conventions (human-maintained)
├── INDEX.md      # Navigation index (always updated)
├── LOG.md        # Changelog (append-only)
│
├── apps/         # Application registry
├── patterns/     # Reusable interaction patterns
├── elements/     # UI element locators
├── cases/        # Successful execution paths
├── failures/     # Known pitfalls & solutions
│
├── raw/          # Immutable raw assets (imported JSON, traces)
└── _archive/     # Deprecated content
```

## Design Principles

Inspired by [Karpathy's LLM Wiki](https://x.com/karpathy/status/1912536934191718830) concept:

1. **Compile, Not RAG** — Pre-compile knowledge to Markdown, inject fragments on-demand
2. **Three-Layer Architecture** — `raw/` (immutable) → Pages (agent-written) → `SCHEMA.md` (constraints)
3. **Session Orientation** — Every session reads `SCHEMA` + `INDEX` + `LOG` first
4. **Cross-Reference Network** — Each page links to ≥2 others via `[[id]]`
5. **Confidence Evolution** — Scores adjust from execution feedback

## Quick Start

### For Agents

```
1. Read SCHEMA.md → Understand constraints
2. Read INDEX.md → Find relevant pages  
3. Read specific pages → Extract knowledge
4. Execute with knowledge
5. Update pages → Record execution results
6. Update INDEX.md + LOG.md
```

### For Humans

- Edit `SCHEMA.md` to change conventions
- Run lint to check integrity
- Review `LOG.md` for recent changes
- Archive low-confidence pages periodically

## Related

- [goal-driven-automation](https://github.com/houlianpi/goal-driven-automation) — Main project
- [Issue #45](https://github.com/houlianpi/goal-driven-automation/issues/45) — Original design discussion

## License

MIT
