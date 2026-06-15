# PLP — Gap Analysis

> **Mockup source:** [PLP.png](figmas/PLP.png) (Figma frames: `Mobile|Tablet|Desktop - PLP`, `Desktop - PLP (No Page Background Colour)`, `Mobile|Tablet|Desktop - PLP - Tabbed Content`, `Mobile|Tablet|Desktop - PLP (Out of Stock)`, `Mobile|Tablet|Desktop - PLP (Filter)`, `Mobile|Tablet|Desktop - PLP (Brochureware)`).
> **Variants/states included:** Default, Tabbed Content (sub-category tabs + sectioned grid), Out of Stock (card-level "Email me when in stock"), Filter (refine drawer/panel), Brochureware (non-transactional "Learn more" cards). Each variant is shown at three breakpoints (mobile, tablet, desktop).
> **Scope:** **IN:** the PLP listing itself — page title, sub-category tabs, sort, quick-filter pills, refine panel/drawer, product grid + cards (incl. subscription pricing, intensity meter, OOS notify-me, badges), in-grid promotions, pagination/"view more", brochureware card variant, FAQ accordion (brochureware variant), background-colour variant. **OUT:** header chrome (logo, mega-nav, search, basket icon, account), hero banner, trust strip, subscribe-to-save band, pre-toolbar promo strip, newsletter band, footer chrome, mini-basket overlay, PDP, cart, checkout, account, age-gate, cookie banner — all covered elsewhere or treated as authored chrome around the PLP.
> **Author:** Jose Maria Franco · **Date:** 2026-06-15

The storefront sits on **Adobe Commerce as a Cloud Service (ACCS) + Edge Delivery Services (EDS)** using the **AEM Boilerplate Commerce** repo. The PLP is hosted by the thin `blocks/product-list-page` EDS block that mounts the **`@dropins/storefront-product-discovery`** drop-in (containers: `SearchResults`, `Facets`, `SortBy`, `Pagination`) backed by Adobe **Live Search**. Add-to-basket on each card delegates to `@dropins/storefront-cart`.

### Mockup

![PLP — all variants and breakpoints](figmas/PLP.png)

**Complexity buckets** (used in §2):
- **Low** — theming / config / authored content / single slot fill.
- **Medium** — new block logic, multi-slot wiring, drawer/state coordination, or initializing one additional drop-in.
- **High** — multi-block coordination plus a new service dependency or external-system integration.

---

## 1. Feature decomposition

| # | Feature | Description in the design | Variant/state | Type | Observations |
|---|---------|---------------------------|---------------|------|--------------|
| F1 | Page title | "Shop Products" page heading sits above the grid on tabbed/brochureware variants; default variant has no title. | All | Cross-cutting | Page title is authored in the page document (front-matter / H1), not produced by the drop-in. |
| F2 | Sub-category tab navigation | Underlined tab row beneath the page title with two visible labels (e.g. "TOBACCO" / "MENTHOL FLAVOURS"); active tab has an underline accent. | Tabbed Content | Commerce | Drives which slice of the catalog is rendered in the grid below; selection should update the search request's `filter` / `categoryPath`. |
| F3 | Sort-by dropdown | Top of the toolbar: "SORT BY: Recommended ▾" — collapses on mobile to icon + label. | All | Commerce | Standard sort dropdown bound to the search request. |
| F4 | Quick-filter pill row | Horizontal scrollable pills next to the sort: "QUICK FILTERS" label then five labelled chips with dot-indicators (e.g. `Filter ●●●○○`). The active pill is highlighted (orange). | All | Commerce | Single-row, multi-select shortcut for the most-used facet values; mirrors a subset of the full Facets panel. |
| F5 | Product count | Right-aligned counter: "20 PRODUCTS" / "30 products" / "300 matching items" (inside the filter drawer). | All | Commerce | Bound to `SearchResult.total_count`. |
| F6 | Refine / filter drawer (mobile + tablet) | Modal drawer titled "Refine" with "CLEAR ALL" link; collapsible accordions per facet (Category, Cartridge Type, …); checkbox list per accordion with counts beside each label; "VIEW MORE" expands long lists; sticky footer with "300 matching items" + orange "SEE RESULTS" CTA. | Filter (mobile/tablet) | Commerce | Mobile/tablet show the refine UI as an overlay drawer; the trigger sits in the toolbar (icon next to Sort By). |
| F7 | Refine / filter sidebar (desktop) | Same anatomy as F6 but rendered inline as a left rail on desktop; adds **Colour** facet (swatch row) and **Price** facet (range slider with min/max numeric inputs `€100`–`€1,000`). | Filter (desktop) | Commerce | Colour swatches and a true range slider with manual inputs are not part of the OOTB facet rendering. |
| F8 | Product grid | 4-column responsive grid on desktop, 2-up on tablet, 1-up on mobile; in-grid promotional rows interrupt the flow (see F17, F18). | All | Commerce | Standard `SearchResults` grid layout. |
| F9 | Card — badge chip | Small chip on top of each card: light-blue "PROMO HERE" or "PROMO MIX" (also a "NEW" / "LABEL" tag on others). | All | Commerce | Merchant-controlled chip whose copy and color reflect a catalog attribute / promotion flag. |
| F10 | Card — product image | Centred product pack (NEO sticks with on-pack regulatory text in target locale). | All | Commerce | Pack imagery is delivered as a single SKU asset; on-pack legal copy is part of the asset, not overlaid. |
| F11 | Card — name + star rating + count | Product name, 5-star rating glyph, "(5)" count. | All | Commerce | **Ratings/review counts are not provided by Catalog Service or Live Search**; require a reviews platform. |
| F12 | Card — price | Single price line ("£5.49") rendered above the intensity meter and purchase-mode selector. | All | Commerce | Base price from Catalog Service / Live Search. |
| F13 | Card — intensity meter | "Intensity ●●●●○ 5" — five circle glyphs, the filled count reflecting strength on a 1–5 scale. | All | Commerce | Custom attribute display; intensity value is a product attribute exposed via Catalog Service. |
| F14 | Card — purchase-mode selector + subscription pricing | Two radio rows inside the card: ("One Time Purchase — £6.49 per pack") and ("Subscribe from — £4.29 per pack"), each with a tooltip (i) icon, plus a "WHAT IS A SUBSCRIPTION?" link. | All (transactional) | Commerce | Adobe Commerce / drop-ins **do not ship a subscription-products feature**; pricing model + frequency + life-cycle require an extension or external subscription service. |
| F15 | Card — quantity stepper + Add to Basket | "− 1 +" stepper next to a full-width orange "ADD TO BASKET" button. | All (transactional) | Commerce | Standard add-to-cart via `@dropins/storefront-cart`; quantity is per-card local state. |
| F16 | Card — Out-of-stock notify-me | OOS card swaps the purchase-mode block for "Currently unavailable" + orange outlined "EMAIL ME WHEN IN STOCK" CTA, plus a check-icon reassurance line "Hopefully not long now". | Out of Stock | Commerce | Adobe documents the pattern for PDP only ("Notify Me CTA"); on the PLP card it has to be added by the host block via a slot fill + custom back-in-stock endpoint. |
| F17 | In-page promotion (dark) | Full-width dark banner inserted between grid rows: image right, "In-page promotion" title, "Body content" copy, single outlined "ACTION" CTA. | Default / Tabbed | Authored content | An authored EDS section spliced into the grid flow at a configurable row position. |
| F18 | Page-breaker promotion (light) | Lighter in-grid card occupying a single grid cell — "This is a page breaker promotion" / "Body content" / outlined "ORDER YOUR RELEASE" CTA. | Default | Authored content | Card-sized authored promotion that replaces one product cell rather than the full row. |
| F19 | Pagination / progress + view more | "You've viewed 24 of 121" counter with a thin progress bar and a "VIEW MORE" outlined CTA below the grid. | All | Commerce | Replaces the OOTB numbered `Pagination`; "load more" pattern with progress indication. |
| F20 | Sectioned grid (Tabbed Content / Brochureware) | Brochureware variant groups the catalog by attribute under section headers "TOBACCO FLAVOURS", "MENTHOL FLAVOURS", "FRUIT FLAVOURS" — each section has its own 3-column grid. | Tabbed Content / Brochureware | Commerce | Multiple search queries (one per section) or one query with client-side grouping by attribute; not a single flat result set. |
| F21 | Brochureware card variant | Simplified card: image, name, rating, price, **"LEARN MORE"** link instead of purchase-mode selector / quantity / Add to Basket. | Brochureware | Commerce | Different `ProductActions` slot fill that swaps the add-to-cart cluster for a PDP link. |
| F22 | FAQ accordion ("Common questions") | List of 5 expandable rows ("Title" placeholders) above the newsletter band. | Brochureware | Authored content | Standard EDS `accordion` block authored on the brochureware variant only. |
| F23 | Background colour variant | "Desktop - PLP (No Page Background Colour)" frame is identical to the default desktop except the page background is white instead of light grey. | Default (alt) | Cross-cutting | Pure theming variant; no behavior change. |

---

## 2. Gap analysis

Legend — **Coverage:** ✅ provided · 🟡 partial (slot / theme / extend) · ❌ none.

| # | Feature | Existing drop-in / block | Coverage | What it gives OOTB | Gap to close | Touch points | Dependencies | Complexity | Risk |
|---|---------|--------------------------|:--------:|--------------------|--------------|--------------|--------------|:----------:|:----:|
| F1 | Page title | `blocks/product-list-page` host + EDS page document | ✅ | Page title comes from the EDS page document (front-matter / `<h1>`); boilerplate ships the host block. | Author the page; restyle heading typography from design tokens. | `blocks/product-list-page/*`, page doc | — | Low | Low |
| F2 | Sub-category tab navigation | none native to product-discovery | ❌ | The drop-in has no tab control; tabs would have to drive `categoryPath` or a `filter` clause on the search request. | Build a tab strip in the host block (or as a sibling EDS block); on tab change call `search(...)` with the new filter and update the URL; persist active tab on reload. | `blocks/product-list-page/*`, history/URL handling | category UIDs / attribute values per tab | Medium | Med |
| F3 | Sort-by dropdown | `SortBy` container | ✅ | Dropdown driven by Live Search-exposed sort attributes; default labels from Commerce. | Mount inside the toolbar; theme to match; localise default option label. | `blocks/product-list-page/product-list-page.js`, placeholders | sort attributes configured in Commerce Admin | Low | Low |
| F4 | Quick-filter pill row | `Facets` container — partial | 🟡 | `Facets` renders the full facet panel; there is no native single-row, multi-select pill shortcut. | Build a pill rail in the host block bound to a curated subset of facet buckets (read from `search/result` event); selection mirrors the same `filter` clauses the full panel uses, so state stays in sync. | new partial in `product-list-page.js`, CSS | choice of which facets/buckets surface as pills (merchant config) | Medium | Med |
| F5 | Product count | `search/result` event payload | ✅ | `SearchResult.total_count` is on every result event. | Render a `<span>` in the toolbar driven by the event listener; pluralisation handled via placeholders. | `product-list-page.js`, placeholders | — | Low | Low |
| F6 | Refine drawer (mobile + tablet) | `Facets` container + host drawer shell | 🟡 | `Facets` renders the accordions, checkbox lists, and SelectedFacets/clear-all functionality; "VIEW MORE" expansion and the sticky "SEE RESULTS" footer are not native. | Build the drawer shell (overlay, slide-in, focus trap, body-scroll-lock, close affordance) in the host block — same pattern as the mini-basket overlay; mount `Facets` inside the drawer; add "VIEW MORE" per facet via `FacetBucket` / `Facet` slot; add a sticky footer with the running `total_count` and a "SEE RESULTS" button that closes the drawer. | `product-list-page.js`, new drawer shell, `Facets` slots (`Facet`, `FacetBucket`, `SelectedFacets`) | overlay shell reused from the cart drawer | Medium | Med |
| F7 | Refine sidebar (desktop) — Colour swatches + Price slider | `Facets` container | 🟡 | Standard facet renders checkboxes/buckets; Live Search supports `range` filters for price; **colour swatches and a dual-thumb range slider with manual inputs are not in the OOTB facet rendering**. | Mount `Facets` inline as left rail; use `FacetBucket` / `FacetBucketLabel` slots to swap the checkbox for a colour swatch when the bucket attribute is colour; build a price range slider component (SDK `Slider` or third party) inside a price-facet slot and translate values to a `range: {from, to}` filter. | `product-list-page.css`, `Facets` slots, slider component | colour-hex per swatch (Commerce attribute or mapping); accessible slider component | Medium | Med |
| F8 | Product grid | `SearchResults` container | ✅ | Grid/list `displayMode`, skeleton-render to prevent CLS, configurable `imageWidth`/`imageHeight`, automatic loading/empty/error states. | Restyle grid breakpoints; tune `imageWidth`/`imageHeight` for above-the-fold cards; set `skeletonCount`. | `product-list-page.js`, `product-list-page.css` | Live Search returning the catalog | Low | Low |
| F9 | Card — badge chip | `SearchResults` slot (`ProductImage` or new badge slot via host block) | 🟡 | No native badge slot — `ProductImage` slot wraps the thumbnail; `ProductName` / `ProductPrice` / `ProductActions` are the only other slots. | Render the chip via the `ProductImage` slot (absolute-positioned overlay) or by appending to the card root in `ProductName`; read the label/color from a product attribute or a promotion flag. | `SearchResults` slot, CSS | catalog attribute for badge label/colour (e.g. `pim_badge`) | Low | Low |
| F10 | Card — product image | `SearchResults` `ProductImage` slot / native | ✅ | Container renders the image with a CDN-transformed URL via `imageWidth`/`imageHeight`. | If `commerce-assets-enabled` is on, point at AEM Assets renditions; otherwise rely on Commerce media URLs. | `config.json` (`commerce-assets-enabled`) | AEM Assets DAM if used | Low | Low |
| F11 | Card — name + star rating + count | `SearchResults` `ProductName` slot + reviews provider | 🟡 | Name renders natively; **rating / review-count is not part of Catalog Service or Live Search**. | Integrate a reviews platform (e.g. Bazaarvoice / Yotpo / PowerReviews) — server-side or client-side fetch keyed by SKU; render stars + count via the `ProductName` (or a dedicated) slot; respect Core Web Vitals (lazy / batched fetch). | `SearchResults` slot, new reviews client | **third-party reviews provider** + per-SKU rating data | High | High |
| F12 | Card — price | `SearchResults` `ProductPrice` slot / native | ✅ | Price renders natively from Live Search data. | Restyle; ensure currency comes from the active store-view config (not hard-coded). | `product-list-page.css`, store config | currency setting per store-view | Low | Low |
| F13 | Card — intensity meter | `SearchResults` `ProductPrice` slot (or custom slot) | 🟡 | No native attribute-meter UI; the intensity value can be exposed via Live Search if added as a product attribute and returned in the search response. | Extend the Live Search attribute set to include `intensity` (1–5); render the 5-dot meter inside the card via a slot fill that reads the value. | Live Search attributes config (Commerce Admin), `SearchResults` slot, CSS | intensity attribute exposed on catalog | Low | Med |
| F14 | Card — purchase-mode selector + subscription pricing | none native | ❌ | Adobe Commerce / drop-ins **do not ship a subscription-products feature**. There is no native "subscribe / one-time" pricing model on cart or catalog. | Either (a) integrate a subscription-management extension/SaaS that exposes subscription SKUs + recurring price + frequency, or (b) author "subscription" as a configurable product option with discounted price. Either way the card needs a custom slot fill that renders the two-radio pricing block, the tooltip / "WHAT IS A SUBSCRIPTION?" link, and routes add-to-cart with the subscription metadata (which cart + checkout must also be ready for). | `SearchResults` `ProductActions`/`ProductPrice` slots, cart block, checkout block | **subscription service / extension** (e.g. Adobe Commerce subscription extension or third-party); pricing model decisions | High | High |
| F15 | Card — quantity stepper + Add to Basket | `SearchResults` `ProductActions` slot + `@dropins/storefront-cart` | 🟡 | No native add-to-cart inside the SearchResults card — the `ProductActions` slot is the documented extension point and `@dropins/storefront-cart`'s `addProductsToCart` is the call. | Render the stepper + button in `ProductActions`; wire `addProductsToCart` with the selected quantity (and subscription metadata from F14 if present); show busy / success state; emit `cart/product/added` so the mini-basket auto-opens (see mini-basket gap). | `SearchResults` slot, `@dropins/storefront-cart` init, CSS | cart drop-in initialised | Medium | Low |
| F16 | Card — OOS notify-me ("Email me when in stock") | none native on PLP | ❌ | Adobe documents the "Notify Me" pattern for the **PDP** (`@dropins/storefront-pdp`), not the PLP card. | Detect stock status on each result item (Live Search returns availability); replace the purchase-mode block in the `ProductActions` slot when `inStock === false` with the "EMAIL ME WHEN IN STOCK" CTA + reassurance line; on click, open a lightweight form (modal) or post to a back-in-stock endpoint; capture email + SKU. | `SearchResults` slot, host block modal, new `/api/back-in-stock` endpoint or marketing integration | **back-in-stock backend** (Commerce extension, Adobe Journey Optimizer, or marketing tool); inStock attribute exposed by Live Search | Medium | Med |
| F17 | In-page promotion (dark, full-width) | Authored EDS section spliced into grid | 🟡 | The grid is a single `SearchResults` container; promotional rows are not interleaved by the drop-in. | Either (a) render the grid then absolute-position / DOM-insert the authored promo after a specific row count, or (b) wrap the grid in a host-managed layout that pulls authored "promo slot" sections from the page document and slots them in between result rows; expose the row position as a `readBlockConfig` knob. | `product-list-page.js`, authored sections, CSS | — | Medium | Med |
| F18 | Page-breaker promotion (single-cell) | Authored content cell within grid | 🟡 | Same constraint as F17 — single-cell promo would need to replace one product cell. | Host block reserves a grid cell at a configured index and fills it with authored content; ensure the cell shape / aspect matches a product card so the grid doesn't reflow. | `product-list-page.js`, authored sections, CSS grid | — | Medium | Low |
| F19 | Pagination / "view more" with progress bar | `Pagination` container — partial | 🟡 | `Pagination` renders a numbered next/prev page control bound to `pageInfo.currentPage` / `totalPages`. There is no "load more" + progress-bar pattern OOTB. | Replace `Pagination` with a custom control: subscribe to `search/result`, request the next page via `search({ current_page: n+1 })`, append results into the grid, update the "You've viewed N of M" copy + progress bar; trade-off: SEO/indexability prefers numbered pagination — consider a hybrid. | `product-list-page.js` (custom control), `search/result` listener, CSS | SEO trade-off decision (open question) | Medium | Med |
| F20 | Sectioned grid (per sub-category section) | Multiple `search` calls or client grouping | 🟡 | A single `search` call returns a flat result list; the drop-in does not natively group by attribute under section headers. | Either (a) fire `Promise.all` with one `search` per section (each filtered by sub-category / attribute) and render each section's grid independently — fits the federated-search pattern documented for product-discovery; or (b) one `search` over the whole category then client-group by attribute. (a) is preferable for relevance/per-section sort. | new partial in `product-list-page.js`, multiple `SearchResults` mounts (one per section) with `scope` isolation | section attribute / category UIDs | Medium | Med |
| F21 | Brochureware card variant ("Learn more" only) | `SearchResults` `ProductActions` slot | ✅ | Slot lets us swap the entire actions area per card / per variant. | Bind a "brochureware" variant flag to the block (block-table key) so the host renders a "Learn more" link → PDP route instead of the stepper + add-to-basket cluster. | `SearchResults` slot, `_product-list-page.json` (block config), CSS | PDP routes resolvable for every SKU | Low | Low |
| F22 | FAQ accordion ("Common questions") | Generic EDS `accordion` block | ✅ | Standard EDS accordion authored in the page. | Author 5 rows; theme to match. | EDS `accordion` block | — | Low | Low |
| F23 | Background colour variant | Theme tokens | ✅ | The "No Page Background Colour" frame is a theming variant. | Expose page-background as a design-token override (or a body-class hook driven by metadata). | `styles/styles.css`, page metadata | — | Low | Low |

---

## 3. Overall assumptions and open questions

### 1. Assumptions
   - The PLP is the boilerplate `product-list-page` block mounting the `@dropins/storefront-product-discovery` drop-in (Live Search-backed) — not a from-scratch component.
   - Live Search and Catalog Service are licensed and provisioned on the ACCS backend; falling back to Core `productSearch` is not planned.
   - Search redirects and tracking (Sensei behavioural events) are enabled per Adobe defaults via the boilerplate's initializers — no custom redirect work in scope here.
   - Product imagery is delivered via Catalog Service URLs unless `commerce-assets-enabled` is later turned on; AEM Assets DAM integration is **not** assumed in MVP.
   - "Brochureware" is a per-page variant (block-table flag) not a global brand switch — the same PLP can be rendered transactionally elsewhere.
   - In-page / page-breaker promotions are authored EDS sections, not driven by an external personalization / Target service.

### 2. Open questions

Decisions blocking the estimate. Each carries a recommendation and the impact of each option.

1. **Subscription pricing model (F14) — Adobe Commerce subscription extension, third-party subscription SaaS, or "configurable product" workaround?**
   - *Recommendation:* third-party subscription SaaS with a clear cart/checkout integration contract (e.g. Ordergroove / Recharge-equivalent), unless the merchant already has an Adobe-aligned subscription extension.
   - *Impact A — subscription extension/SaaS:* significant scope across catalog, PLP card, cart, and checkout; recurring billing, frequency UI, manage-subscription account screens; F14, F15 (basket wiring), cart, checkout all gain XL work.
   - *Impact B — configurable-product workaround (one-time vs subscribe as an option):* much smaller; loses true recurring billing and account-level subscription management; not viable if merchant truly needs recurring orders.

2. **Reviews and ratings (F11) — which platform, and is it in MVP?**
   - *Recommendation:* defer to phase 2 unless a provider is already chosen; if in MVP, prefer a provider with a documented JS SDK + per-SKU bulk fetch.
   - *Impact A — third-party reviews (Bazaarvoice / Yotpo / PowerReviews) in MVP:* XL scope — per-SKU rating + count for the card, lazy/batched fetch to protect CWV, full PDP review widget (separate doc); F11 dominates.
   - *Impact B — drop ratings from MVP:* F11 collapses to a name-only restyle; revisit when a provider is selected.

3. **Back-in-stock notifications (F16) — Adobe Journey Optimizer / Commerce extension / lightweight in-house service?**
   - *Recommendation:* Adobe Journey Optimizer (AJO) or the merchant's existing ESP, called via a thin server endpoint; avoid building stock-watcher logic in the browser.
   - *Impact A — AJO/ESP:* moderate work — email capture form + server endpoint + journey config; clean separation.
   - *Impact B — bespoke in-house service:* additional backend ownership, stock-watch jobs, dedupe, unsubscribe.

4. **Pagination model (F19) — load-more progress bar (as designed) vs numbered pagination?**
   - *Recommendation:* numbered pagination for SEO + a "load more" affordance on top, OR keep load-more but emit canonical paginated URLs.
   - *Impact A — load-more only:* matches design as-is; risks PLP-level SEO (only page 1 indexed); breaks deep-linking to a specific page.
   - *Impact B — numbered (or hybrid):* small UX change; preserves indexability and deep linking; aligns with the drop-in's stance on `current_page` over infinite scroll.

5. **Quick filters (F4) — merchant-curated subset or auto-pick from facet response?**
   - *Recommendation:* merchant-curated list (block-table or authored config) drawn from the same facet attributes.
   - *Impact A — curated:* low complexity; predictable; needs an authoring surface.
   - *Impact B — auto:* zero authoring but unpredictable order/relevance; pills may shift per result set.

6. **Tabbed Content (F2) and Sectioned grid (F20) — how are sub-categories defined: catalog categories or product attribute values?**
   - *Recommendation:* catalog categories with stable UIDs (best for sort/relevance per section); attribute-based grouping for cosmetic shelves only.
   - *Impact A — categories:* one `search` per tab/section with `categoryPath`; simple, isolatable, cacheable.
   - *Impact B — attribute grouping:* one `search` over the parent category + client grouping; can't sort independently per section.

7. **In-grid promotions (F17, F18) — authored only, or merchandised via Adobe Target / personalization?**
   - *Recommendation:* authored only for MVP; revisit personalization in a later phase.
   - *Impact A — authored:* one EDS section per promo slot; cheap.
   - *Impact B — personalized:* requires `@dropins/storefront-personalization` (AEP-backed) and tagged audiences; bigger scope.

8. **Multi-currency / multi-locale — single store-view or per-country store-views?**
   - *Recommendation:* per-country store-views (`Magento-Store-View-Code` per locale) — pack imagery already shows localised on-pack warnings (Czech text visible), so locale isolation matters.
   - *Impact A — per-country store-views:* config-driven, no code changes per locale; placeholder spreadsheets per language.
   - *Impact B — single store-view:* simpler now, expensive to retrofit later.

## 4. Explicitly out of scope

What the estimate must **not** include — protects against scope creep in the estimation session.

- Header chrome (logo, mega-nav, search box, account, basket icon trigger) — covered by the header gap analysis.
- **Hero banner** above the PLP grid — authored EDS section, treated as page chrome, not PLP functionality.
- **Trust strip** (e.g. "Tested by Scientists", "Standard Delivery included on all orders", "28 Money Back Guarantee") — authored row between hero and grid; not PLP functionality.
- **Subscribe-to-save promo band** between hero and grid — authored content.
- **Pre-toolbar promo strip** above the hero ("Refer your favorites" / "All Products") — authored row.
- **Newsletter signup band** between grid and footer — authored chrome; integration with the chosen ESP / CDP is its own gap.
- Footer chrome and payment icons — covered by the footer gap analysis.
- Mini-basket overlay (covered in `minibasket-gap-analysis.md`); only `cart/product/added` emission from F15 is in scope here.
- PDP and PDP-side back-in-stock notify-me, configurable options, and gallery.
- Cart page, checkout, place-order, payment, address.
- Account, orders, returns, addresses, subscriptions self-service.
- Age-gate / `(18+)` consent enforcement, cookie banner, and consent management platform integration.
- Reviews-platform integration itself (Bazaarvoice / Yotpo / PowerReviews) — treated as a dependency (open question #2).
- Subscription service / recurring billing platform itself — treated as a dependency (open question #1).
- Back-in-stock backend (AJO / ESP / custom) itself — treated as a dependency (open question #3).
- Adobe Target / Personalization-driven merchandising of in-grid promotions (open question #7).
- AEM Assets DAM integration for product imagery, unless `commerce-assets-enabled` is later turned on.
- B2B variations (companies, quotes, requisition lists, quick order).
- Search-results page (`?q=…`) — overlaps with PLP code path but has its own UX surfaces (suggestions, redirects, federated search) not shown in this mockup.
- SEO indexing of faceted URLs, canonical URL strategy, and JSON-LD emission (covered in performance/SEO doc).

---

## 5. References

### Curated skill references (read while preparing this doc)
- `accs-storefront-architect/references/dropin-product-discovery.md` — `@dropins/storefront-product-discovery` v3.1.0: containers (`SearchResults`, `Facets`, `SortBy`, `Pagination`), `search()` API, `search/result` event, facet/slot inventory, faceted-URL pattern, federated search.
- `accs-storefront-architect/references/boilerplate.md` — `blocks/product-list-page` block contract; drop-in initializer pattern; `readBlockConfig` knob authoring; UE instrumentation; CSS / JS / slot customization order.
- `accs-storefront-architect/references/architecture.md` — drop-in vs block layering; event bus contract.
- `accs-storefront-architect/references/dropins-overview.md` — slots-before-overrides decision rule; extend-vs-create guidance.
- `accs-storefront-architect/references/services-and-graphql.md` — Live Search vs Core `productSearch`; `commerce-endpoint` vs `commerce-core-endpoint`; header sets.
- `accs-storefront-architect/references/configuration.md` — `config.json` keys (`commerce-assets-enabled`, store-view headers); multistore patterns.
- `accs-storefront-architect/references/llms-full.txt` — direct verbatim reads for:
  - `SearchResults` container slot signatures (`ProductActions`, `ProductPrice`, `ProductName`, `ProductImage`, `NoResults`, `Header`, `Footer`) — lines ≈40168–40300, 41107–41200.
  - `Facets` container slot signatures (`Facet`, `SelectedFacets`, `Facets`, `FacetBucket`, `FacetBucketLabel`) — lines ≈40991–41105.
  - `SortBy` / `Pagination` container props — lines ≈40311–40400, 40006–40050.
  - Notify Me CTA pattern (documented for PDP, adapted for PLP card in F16) — lines ≈39684–39830.

### Adobe official documentation (canonical URLs)
- Product Discovery overview — `https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/`
- `SearchResults` container — `https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/containers/search-results/`
- `Facets` container — `https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/containers/facets/`
- `SortBy` container — `https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/containers/sort-by/`
- `Pagination` container — `https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/containers/pagination/`
- Drop-in slots reference — `https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/slots/`
- Drop-in events reference — `https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/events/`
- Federated search pattern — `https://experienceleague.adobe.com/developer/commerce/storefront/how-tos/federated-search/`
- Search redirects — `https://experienceleague.adobe.com/developer/commerce/storefront/how-tos/search-redirects/`
- Boilerplate blocks reference — `https://experienceleague.adobe.com/developer/commerce/storefront/boilerplate/blocks-reference/`
- Customizing blocks (slots, events, CSS) — `https://experienceleague.adobe.com/developer/commerce/storefront/boilerplate/customizing-blocks/`
- AEM Boilerplate Commerce repo — `https://github.com/hlxsites/aem-boilerplate-commerce`
- Notify Me CTA tutorial (PDP-side reference pattern) — `https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/events/` + boilerplate PR `https://github.com/hlxsites/aem-boilerplate-commerce/pull/1116`

### Project artefacts
- `_templates/page-gap-analysis.template.md` — the structural template this document follows.
- `minibasket-gap-analysis.md` — sibling gap doc consulted for tone/structure parity and for the overlay-shell pattern reused by F6 (refine drawer).
- `figmas/PLP.png` — primary design source.
