# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A **scoping / fit-gap analysis workspace**, not a code repository. The goal is to size the development effort to rebuild BAT's multi-category ecommerce frontends on **Adobe Commerce as a Cloud Service (ACCS) + Edge Delivery Services (EDS)** using the **AEM Boilerplate Commerce** drop-ins (`@dropins/storefront-*`).

Inputs are Figma exports (large PNG mockups). Outputs are per-page gap-analysis markdown files that map each design feature to the closest existing drop-in / container / slot, classify coverage as `✅ / 🟡 / ❌`, and size complexity (Low / Medium / High) and risk.

## Workflow

The intended loop for each new page/feature:

1. Load a mockup PNG, a design (one image, multiple frames/breakpoints/variants inside it).
2. Crop and downsample to readable tiles with `sips` (the original PNGs are too large to read directly — see Image-handling notes).
3. Invoke the **`storefront-gapanalysis`** skill (slash command) with the Figma slices and a description of the feature to be built — it owns the per-page template and process.
4. Source Commerce facts from the **`storefront-knowledge`** skill (drop-in names, container props, slot names, events, services). Source generic EDS facts from **`eds-knowledge`**.

Always emit the completed gap analysis into `backlog/` — that is the deliverable store, not an archive.

## Skills (project-local)

Project-local config lives in `.claude/`:

- **`.claude/skills/storefront-knowledge/`** — Commerce-layer knowledge base (drop-ins, containers, slots, events, services, GraphQL endpoints, commerce blocks, SDK, configuration). `SKILL.md` plus `references/*.txt` — Adobe's official per-topic LLM bundles stored verbatim (30–225 KB each — **grep first, never read whole**).
- **`.claude/skills/eds-knowledge/`** — EDS delivery-layer knowledge (blocks, sections, authoring, metadata, indexing, Lighthouse).
- **`.claude/skills/storefront-gapanalysis/`** — the entry point, invoked as a slash command with Figma slices + a feature description. Owns the gap-doc process and template (`references/page-gap-analysis.template.md`); greps the `storefront-knowledge` bundles for Commerce facts and reads `eds-knowledge` for delivery-layer facts.

Phrasing discipline carried over from the knowledge skill: **"block"** = EDS block (`decorate(block)`); **"container"** = drop-in container; **"drop-in"** = the npm package; **"slot"** = named extension point inside a container. Use Adobe's official names verbatim (ACCS, ACO, Adobe Commerce PaaS, Storefront Compatibility Package, Catalog Service, Live Search, Product Recommendations, Payment Services, AEM Commerce Prerender, etc.).

## Gap-doc conventions

The section structure, table columns, legend, and complexity buckets are defined **once**, in the template at `.claude/skills/storefront-gapanalysis/references/page-gap-analysis.template.md` — do not restate or restructure them. The document shape is: **§1 Feature gap analysis** (a single combined table — one row per feature, with **Gap to close**, **Complexity**, and **Risk** as the primary columns), **§2 Assumptions and open questions**, **§3 Explicitly out of scope**. Beyond the template:

- Author is **Jose Maria Franco**.
- Mirror the existing examples in `backlog/` (e.g. `backlog/plp.md`, `backlog/minibasket.md`, `backlog/pdp.md`) for tone and granularity.
- Deliverables always go to `backlog/<page-or-feature-slug>.md`.

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
- Don't `Read` a `storefront-knowledge` bundle whole — grep with a narrow pattern, then read only the matching line range.
- Don't invent drop-in / container / slot / event / config-key names — verify against the `storefront-knowledge` bundles.
- Don't propose custom components when an Adobe container exists; don't mix the deprecated monolithic `ProductDetails` with the modular containers.
- Don't restructure the gap-doc template — the columns, legend, and §1/§2/§3 layout are load-bearing for downstream estimation.
