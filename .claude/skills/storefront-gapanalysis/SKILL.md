---
name: storefront-gapanalysis
description: Produce a structured fit/gap (gap analysis) document for an Adobe Commerce as a Cloud Service (ACCS) + Edge Delivery Services (EDS) storefront page or feature set, mapping each required feature to the most specific existing `@dropins/storefront-*` drop-in, container, and slot. Use this skill whenever the user asks for a "gap analysis", "fit/gap", "coverage analysis", "what's missing", "what do we need to build", "analyze the gaps", or to map a design / mockup / Figma frame / requirements list against the drop-in library for a PDP, PLP, search, cart, checkout, account, order, B2B, wishlist, payments, or recommendations page. Trigger when the deliverable is a per-page or per-feature gap document that classifies coverage as provided / partial / none and sizes complexity and risk. Inputs are Figma slices (mockup crops) and a description of the feature to be built. Depends on the `storefront-knowledge` skill for the underlying Commerce facts and on `eds-knowledge` for the EDS delivery layer.
---

# Storefront Gap Analysis

You produce structured **fit/gap analysis** documents for storefront pages and feature sets built on **Adobe Commerce as a Cloud Service (ACCS) + Edge Delivery Services (EDS)** using the **AEM Boilerplate Commerce** repo.

A gap analysis answers: *for each feature in this design, which existing drop-in / container / slot already covers it, what's partially covered, what has to be built from scratch — and how complex and risky is each gap.*

## Inputs

This skill is invoked as a slash command with two inputs:

1. **Figma slices** — one or more mockup images (crops of the design, per frame / breakpoint / variant).
2. **A description of the feature to be built** — the page or feature scope, plus any constraints the user states.

If either input is missing (no image attached, or no feature description), ask for it before doing anything else.

## Feature types

Every feature in a design is one of two types — classify each one first, because the type decides which knowledge skill answers it and what the OOTB basis can be:

1. **Content features** — showcase content with no commerce state: hero blocks, navigation, video blocks, banners, editorial sections, footers. Their OOTB basis is an **EDS block** (boilerplate block collection or a custom block) plus authored content. Source facts from **`eds-knowledge`** (block collection, sections, authoring, metadata). Commerce bundles are irrelevant here — do not map content features to drop-ins.
2. **Commerce features** — require commerce state or operations: cart operations, checkout, login/registration, product data, search, orders, wishlist, B2B flows. Their OOTB basis is a **drop-in / container / slot** (mounted by a commerce block). Source facts from **`storefront-knowledge`**.

A feature can straddle both (e.g. a hero with a live product price): treat the content shell as type 1 and the commerce fragment as type 2, in the same row or split rows if separately estimable.

## Knowledge sources

This skill owns the **gap-analysis process and template**. It does **not** duplicate the platform knowledge — that lives in two sibling skills. Consult them rather than restating their content; if one is unavailable, produce the document but flag any identifier you could not verify.

- **`eds-knowledge`** — the EDS delivery layer: generic blocks, sections, authoring, metadata, indexing, placeholders, performance. Use it for *Authored content* rows and anything about how content is authored or delivered.
- **`storefront-knowledge`** — the Commerce layer: drop-in / container / slot / event identifiers, services, GraphQL endpoints, commerce blocks, SDK, configuration. Verbatim Adobe documentation bundles; **grep first, then read only the matching line range** (files are 30–225 KB).

## Workflow

1. **Read the template** at `references/page-gap-analysis.template.md` (in this skill) to get the exact section structure and column definitions before writing anything.
2. **Triage the features** in the Figma slices and description into the two feature types above — content vs commerce — before gathering any context. This determines which knowledge skill you consult per feature.
3. **Gather EDS context for content features** from `eds-knowledge`: the block collection doc for which boilerplate blocks exist (cards, columns, hero, carousel, tabs, embed/video, fragment…), and the authoring/markup docs for how the content is authored.
4. **Gather Commerce context for commerce features** by grepping/reading the matching bundle(s) in the `storefront-knowledge` skill's `references/` folder:
   - PDP → `dropins-pdp.txt`
   - PLP / Search / Recommendations / Personalization → `dropins-catalog.txt`
   - Cart / mini-cart → `dropins-cart.txt`
   - Checkout → `dropins-checkout.txt`
   - Orders / returns / confirmation → `dropins-order.txt`
   - Account / Auth → `dropins-account-auth.txt`
   - Wishlist / Payment Services → `dropins-wishlist-payments.txt`
   - B2B companies / switcher → `dropins-b2b-company.txt`; quotes → `dropins-b2b-quote.txt`; purchase orders / quick order / requisition lists → `dropins-b2b-purchasing.txt`
   - Cross-cutting drop-in concepts (slots, events, dictionaries, styling) → `dropins-intro.txt`
   - Which block mounts which container, block table options → `blocks-reference.txt`
   - SDK / design tokens / custom drop-ins → `sdk-reference.txt`
   - Configuration / endpoints / multistore / infra → `setup-configuration.txt`

   The bundles are large — **grep for the section heading or identifier first, then read only that line range; never read a whole bundle.**
5. **Write the intro:** embed the supplied mockup screenshot(s) as markdown images (`![alt](path)`) — never just name or link the file — then add a **bullet list, never a paragraph**, below it: one bullet per functional input/constraint the user gave, polished into clear prose but kept as separate bullets, not merged into flowing text. Do not restate methodology, process, or template mechanics (architecture boilerplate, complexity-bucket definitions, documentation-link conventions) in the generated document — those are defined once, in the template's authoring-reference comment.
6. **Fill every section of the template:**
   - **§1 Feature gap analysis** — one row per discrete feature visible in the design or described by the user. Tag each feature `[C]` *Commerce*, `[A]` *Authored content*, or `[X]` *Cross-cutting*, noting the variant/state inline when not Default. Map commerce features to the most specific existing drop-in / container / slot in **OOTB basis**; map content features to the closest EDS block (boilerplate block collection or custom). Use exact package names (`@dropins/storefront-cart`), container names (`CartSummaryList`), and slot names (`Heading`, `Footer`). **Hyperlink every drop-in package name, container name, and named Adobe service to its official Experience League URL** using the documentation URL reference table at the end of the template. Set Coverage to ✅ / 🟡 / ❌ using the legend. Make **Gap to close** directly estimable, and size Complexity and Risk using the bucket definitions in the template's authoring-reference comment.
   - **§2 Assumptions and open questions** — list decisions blocking the estimate; follow the *Recommendation / Impact A / Impact B* format.
   - **§3 Out of scope** — list explicitly what is excluded to protect the estimate.
7. **Never invent slot or event names** — verify them against the `storefront-knowledge` bundles.
8. **Write the completed document to `backlog/<page-or-feature-slug>.md`** (derive the slug from the page/feature name, lowercase, hyphenated). Confirm the file path to the user after writing.

## Rules of thumb for accurate gaps

- **Slots before overrides before forks.** A feature reachable through a named slot is 🟡 *partial* at Low/Medium complexity; one needing a fork is ❌ or high-complexity 🟡. Don't mark something ❌ when a slot reaches it.
- **Prefer the most specific container.** Map to `ProductPrice` / `ProductGallery`, not the deprecated monolithic `ProductDetails`. Map cart line items to `CartSummaryList`, not "the cart drop-in" generically.
- **A new service dependency raises Risk.** Anything that turns on Live Search, Product Recommendations, Payment Services, or Data Connection / AEP carries backend-enablement risk — note it in Dependencies and bump Risk.
- **Authored content is not a gap.** Hero copy, merchandising banners, and SEO text authored in EDS documents are *Authored content*, not Commerce build — classify them so they don't inflate the estimate.
- **Don't turn content features into commerce work.** A hero, navigation, or video block with no commerce state needs no drop-in, no GraphQL, and no service enablement — an existing boilerplate block is ✅ or low-complexity 🟡; only a genuinely new block design is ❌.
- **B2B features assume `commerce-b2b-enabled` + the B2B Compatibility Package.** Note that prerequisite in Dependencies for any B2B row.
- **Don't double-count cross-cutting features.** Header, footer, mini-cart, and auth widgets shared across pages belong in a cross-cutting section, not repeated per page.

## Complexity and risk buckets

Defined once in the template's authoring-reference comment — apply them consistently. Do not restate them in the generated document or redefine them here.
