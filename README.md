# Storefront Gap Analysis

Scoping and fit-gap analysis workspace for rebuilding BAT's multi-category ecommerce frontends on **Adobe Commerce as a Cloud Service (ACCS) + Edge Delivery Services (EDS)** using the [AEM Boilerplate Commerce](https://github.com/hlxsites/aem-boilerplate-commerce) drop-ins (`@dropins/storefront-*`).

## Purpose

Each gap-analysis document maps a Figma design to the closest existing drop-in / container / slot, classifies coverage (`✅ provided · 🟡 partial · ❌ none`), and sizes complexity (Low / Medium / High) and risk. The output feeds downstream estimation and project scoping.

## Documents

Completed gap-analysis documents live in [`backlog/`](backlog/). For example:

| File | Page / Feature |
|---|---|
| [`backlog/plp.md`](backlog/plp.md) | Product Listing Page |
| [`backlog/minibasket.md`](backlog/minibasket.md) | Mini Basket overlay |
| [`backlog/pdp.md`](backlog/pdp.md) | Vuse PDP — Device + Consumable |

## Workflow (adding a new page)

1. Add the Figma export PNG to `figmas/` and crop/resample it into readable slices.
2. Invoke the **`storefront-gapanalysis`** skill as a slash command, providing the Figma slices and a description of the feature to be built.
3. The skill writes the completed document directly to `backlog/<page-slug>.md`.

## Claude Code artifact setup

All Claude Code configuration is project-local under `.claude/`. Three skills cooperate to produce a gap-analysis document:

```
.claude/
└── skills/
    ├── storefront-gapanalysis/         # entry point — owns the gap-doc process and template
    │   ├── SKILL.md
    │   └── references/
    │       └── page-gap-analysis.template.md   # canonical layout, columns, legend, complexity buckets
    ├── storefront-knowledge/           # Commerce-layer knowledge base (drop-ins, services, GraphQL, blocks, SDK)
    │   ├── SKILL.md
    │   └── references/                 # Adobe's official llms.txt topic bundles, verbatim (grep, never read whole)
    └── eds-knowledge/                  # EDS delivery-layer knowledge (blocks, sections, Lighthouse)
```

| Artifact | Type | Purpose |
|---|---|---|
| `storefront-gapanalysis` | Skill | Entry point (slash command). Inputs: Figma slices + feature description. Runs the per-page workflow: read template → gather Commerce context → fill §1 feature gap analysis (single combined table), §2 assumptions/open questions, §3 out-of-scope → write to `backlog/<page-slug>.md`. |
| `page-gap-analysis.template.md` | Template | Section structure, the combined §1 table columns (`Feature · OOTB basis · Coverage · Gap to close · Dependencies & touch points · Complexity · Risk`), coverage legend, complexity buckets, Experience League URL reference table. Load-bearing — do not restructure. |
| `storefront-knowledge` | Skill | Authoritative source for drop-in / container / slot / event / service identifiers. Built from Adobe's official per-topic LLM bundles (`…/commerce/storefront/llms.txt`); grepped directly by the gap-analysis skill. |
| `eds-knowledge` | Skill | Authoritative source for EDS facts (blocks, sections, authoring, metadata, Lighthouse, Core Web Vitals). |

### How the pieces fit at runtime

1. User invokes `/storefront-gapanalysis` with the Figma slices and a feature description.
2. The skill reads `references/page-gap-analysis.template.md` to lock in the structure.
3. The skill greps/reads the matching topic bundle(s) in `storefront-knowledge` for drop-in / container / slot / service facts, and the matching `eds-knowledge` document for delivery-layer facts.
4. The skill fills the template — one combined §1 table with per-feature coverage, gap to close, complexity, and risk — hyperlinks every drop-in / container / service to Experience League, and writes the completed document to `backlog/<page-slug>.md`.

### Editing the configuration

- **Change the process or rules of thumb** → edit `.claude/skills/storefront-gapanalysis/SKILL.md`.
- **Change the document structure or columns** → edit `.claude/skills/storefront-gapanalysis/references/page-gap-analysis.template.md` (this also changes every future gap doc).
- **Refresh Commerce facts** → re-download the bundles under `.claude/skills/storefront-knowledge/references/` (command in that skill's SKILL.md).

## Figma exports

Source PNGs live in `figmas/`.

## Author

Jose Maria Franco — Netcentric
