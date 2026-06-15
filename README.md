# Storefront Gap Analysis

Scoping and fit-gap analysis workspace for rebuilding BAT's multi-category ecommerce frontends on **Adobe Commerce as a Cloud Service (ACCS) + Edge Delivery Services (EDS)** using the [AEM Boilerplate Commerce](https://github.com/hlxsites/aem-boilerplate-commerce) drop-ins (`@dropins/storefront-*`).

## Purpose

Each gap-analysis document maps a Figma design to the closest existing drop-in / container / slot, classifies coverage (`✅ provided · 🟡 partial · ❌ none`), and sizes complexity (Low / Medium / High) and risk. The output feeds downstream estimation and project scoping.

## Documents

### Active

_None in progress._

### Archived ([`backlog/`](backlog/))

| File | Page / Feature |
|---|---|
| [`backlog/plp.md`](backlog/plp.md) | Product Listing Page |
| [`backlog/minibasket-gap-analysis.md`](backlog/minibasket-gap-analysis.md) | Mini Basket overlay |
| [`backlog/vuse-pdp-gap-analysis.md`](backlog/vuse-pdp-gap-analysis.md) | Vuse PDP — Device + Consumable |

## Workflow (adding a new page)

1. Add the Figma export PNG to `figmas/`.
2. Run the `/storefront-gapanalysis` slash command in Claude Code — provide the design and any notes.
3. Save the output as a new top-level `.md` (e.g. `pdp.md`); once retired or superseded, move it into `backlog/`.

## Claude Code artifact setup

All Claude Code configuration is project-local under `.claude/`. Three layers cooperate to produce a gap-analysis document:

```
.claude/
├── commands/
│   └── storefront-gapanalysis.md       # /storefront-gapanalysis slash command — owns the process
├── templates/
│   └── page-gap-analysis.template.md   # canonical section layout, columns, legend, complexity buckets
├── skills/
│   ├── accs-storefront-architect/      # Commerce-layer knowledge base (drop-ins, services, GraphQL)
│   │   ├── SKILL.md
│   │   └── references/                 # curated topic docs + llms-full.txt (grep, never read whole)
│   └── eds-knowledge/                  # EDS delivery-layer knowledge (blocks, sections, Lighthouse)
└── agents/
    └── storefront-architect.md         # Opus-backed architect subagent
```

| Artifact | Type | Purpose |
|---|---|---|
| `/storefront-gapanalysis` | Slash command | Entry point. Runs the per-page workflow: read template → gather Commerce context → fill §1 features, §2 gaps, §3 assumptions, §4 out-of-scope → output as a fenced code block. |
| `page-gap-analysis.template.md` | Template | Section structure, column definitions, coverage legend, complexity buckets, Experience League URL reference table. Load-bearing — do not restructure. |
| `accs-storefront-architect` | Skill | Authoritative source for drop-in / container / slot / event / service identifiers. Consulted via the `storefront-architect` subagent. |
| `eds-knowledge` | Skill | Authoritative source for EDS facts (blocks, sections, authoring, metadata, Lighthouse, Core Web Vitals). |
| `storefront-architect` | Subagent | Opus-backed architect that pulls from both knowledge skills. Spawned by the slash command for Commerce context; also invokable directly for "should we…" / architecture-review questions. |

### How the pieces fit at runtime

1. User runs `/storefront-gapanalysis <feature-description>` and attaches the design.
2. The command reads `.claude/templates/page-gap-analysis.template.md` to lock in the structure.
3. The command spawns the `storefront-architect` subagent to gather drop-in / container / slot / service facts for the page type. The subagent reads the curated reference files in `accs-storefront-architect` (and greps `llms-full.txt` only as fallback).
4. The command fills the template, hyperlinks every drop-in / container / service to Experience League, and emits the completed document as a fenced markdown block.

### Editing the configuration

- **Change the process or rules of thumb** → edit `.claude/commands/storefront-gapanalysis.md`.
- **Change the document structure or columns** → edit `.claude/templates/page-gap-analysis.template.md` (this also changes every future gap doc).
- **Add or correct Commerce facts** → edit files under `.claude/skills/accs-storefront-architect/references/`.
- **Change architect behavior** → edit `.claude/agents/storefront-architect.md`.

## Figma exports

Source PNGs live in `figmas/`.

## Author

Jose Maria Franco — Netcentric
