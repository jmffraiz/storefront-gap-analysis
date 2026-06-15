# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A **scoping / fit-gap analysis workspace**, not a code repository. The goal is to size the development effort to rebuild BAT's multi-category ecommerce frontends on **Adobe Commerce as a Cloud Service (ACCS) + Edge Delivery Services (EDS)** using the **AEM Boilerplate Commerce** drop-ins (`@dropins/storefront-*`).

Inputs are Figma exports (large PNG mockups). Outputs are per-page gap-analysis markdown files that map each design feature to the closest existing drop-in / container / slot, classify coverage as `✅ / 🟡 / ❌`, and size complexity (Low / Medium / High) and risk.

## Workflow

The intended loop for each new page/feature:

1. Load a mockup PNG, a design (one image, multiple frames/breakpoints/variants inside it).
2. Crop and downsample to readable tiles with `sips` (the original PNGs are too large to read directly — see Image-handling notes).
3. Run the **`/storefront-gapanalysis`** slash command — it owns the per-page template and process, providing the design and user input.
4. Source Commerce facts from the **`accs-storefront-architect`** skill (drop-in names, container props, slot names, events, services). Source generic EDS facts from **`eds-knowledge`**.

Always emit the gap analysis as a new top-level `.md`; archived/superseded docs live in `backlog/`.

## Skills and agent (project-local)

Project-local config lives in `.claude/`:

- **`.claude/skills/accs-storefront-architect/`** — Commerce-layer knowledge base (drop-ins, services, GraphQL endpoints, ACCS vs PaaS, Universal Editor wiring). `SKILL.md` plus `references/*.md` per topic and `references/llms-full.txt` (2.8 MB — **grep first, never read whole**).
- **`.claude/skills/eds-knowledge/`** — EDS delivery-layer knowledge (blocks, sections, authoring, metadata, indexing, Lighthouse).
- **`.claude/commands/storefront-gapanalysis.md`** — slash command that owns the gap-doc process. Reads the template at `.claude/templates/page-gap-analysis.template.md` and pulls Commerce facts from `accs-storefront-architect` (via the `storefront-architect` subagent).
- **`.claude/agents/storefront-architect.md`** — opus-backed architect agent that pulls from both knowledge skills above. Invoke via the `storefront-architect` subagent type for design / "should we…" / architecture-review questions.

Phrasing discipline carried over from the architect skill: **"block"** = EDS block (`decorate(block)`); **"container"** = drop-in container; **"drop-in"** = the npm package; **"slot"** = named extension point inside a container. Use Adobe's official names verbatim (ACCS, ACO, Adobe Commerce PaaS, Storefront Compatibility Package, Catalog Service, Live Search, Product Recommendations, Payment Services, AEM Commerce Prerender, etc.).

## Gap-doc conventions (match the existing files)

When producing a new gap analysis, mirror the archived examples in `backlog/` (e.g. `backlog/plp.md`, `backlog/minibasket-gap-analysis.md`, `backlog/vuse-pdp-gap-analysis.md`):

- **Header block** with mockup source link, variants/states included, in/out scope, author (Jose Maria Franco), date.
- One-paragraph **architecture lead** naming the host block, drop-in, and key container(s).
- **§1 Feature decomposition** table: `# · Feature · Description in the design · Variant/state · Type · Observations`. Type is one of `Cross-cutting | Commerce | Authored content`.
- **§2 Gap analysis** table with the legend `✅ provided · 🟡 partial · ❌ none` and columns: `Feature · Existing drop-in / block · Coverage · What it gives OOTB · Gap to close · Touch points · Dependencies · Complexity · Risk`. Complexity buckets are defined at the top of each doc (Low / Medium / High) — keep that definition block.
- **§3 Overall assumptions and open questions** — list assumptions, then numbered open questions each with *Recommendation* + *Impact A / Impact B* branches.
- Link drop-in/container/slot names to the matching Adobe Experience League page. Use exact identifiers (`@dropins/storefront-cart`, `MiniCart`, `ProductActions`, `cart/product/added`, `commerce-endpoint`, etc.).

## Image-handling notes

The Figma exports are very large. Reading them directly will fail or be unreadable. Use `sips` to crop and resample — the local permission allowlist (`.claude/settings.local.json`) is already populated with the common variants of this pattern, e.g.:

```sh
# Whole-image downsample for a first overview
sips --resampleWidth 1900 /Users/2245328/storefront-gap-analysis/figmas/PLP.png --out figmas/sips/plp-overview.png

# Crop a single frame (cropOffset is y x; cropToHeightWidth is height width)
sips --cropToHeightWidth 3000 1600 --cropOffset 200 1400 /Users/2245328/storefront-gap-analysis/figmas/PLP.png --out figmas/sips/s1d-top.png
sips --resampleWidth 1500 figmas/sips/s1d-top.png --out figmas/sips/s1d-top-r.png

# Confirm dimensions
sips -g pixelWidth -g pixelHeight figmas/sips/plp-overview.png
```

Workflow: get pixel dimensions first → downsample whole image to find the frame grid → crop per frame at native resolution → resample the crop down to ~1200–1900px wide before reading. Adding new sips invocations triggers a permission prompt; that's expected.

## What NOT to do

- Don't try to `Read` the raw `*.png` files at full resolution — crop/resample first.
- Don't `Read` or `Grep` the skill `llms-full.txt` whole — grep with a narrow pattern.
- Don't invent drop-in / container / slot / event / config-key names — verify against the `accs-storefront-architect` references.
- Don't propose custom components when an Adobe container exists; don't mix the deprecated monolithic `ProductDetails` with the modular containers.
- Don't restructure the gap-doc template — the columns, legend, and §1/§2/§3 layout are load-bearing for downstream estimation.
