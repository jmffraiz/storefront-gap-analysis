# ACCS + EDS Storefront Architecture

How EDS and Adobe Commerce fit together: delivery layer, drop-ins, blocks, event bus, GraphQL endpoints, backends, page-load flow.

## Topology

```
                       Authoring layer
            (DA.live / Google Drive / SharePoint /
             Universal Editor)
                            │
                            ▼
+----------------------------------------------------------+
|        Edge Delivery Services (aem.live / aem.page)      |
|        HTML, JS, CSS, images served from edge            |
|        - blocks/* decorate HTML structure                |
|        - scripts/aem.js core EDS bootstrap               |
|        - scripts/commerce.js Commerce config             |
|        - scripts/initializers/* mount drop-ins           |
+----------------------------------------------------------+
                            │
                            │ Browser-side fetches (HTTPS)
                            │
            ┌───────────────┴────────────────┐
            ▼                                ▼
+-------------------------+      +-------------------------+
| commerce-endpoint       |      | commerce-core-endpoint  |
| (Adobe-hosted SaaS)     |      | (Adobe Commerce GraphQL)|
| - Catalog Service       |      | - Cart, Checkout        |
| - Live Search           |      | - Customer, Order       |
| - Product Recs          |      | - Wishlist, Account     |
| READ-HEAVY              |      | STATE WRITES            |
+-------------------------+      +-------------------------+
```

EDS is the **delivery layer**. It never proxies GraphQL. The browser hits Commerce GraphQL hosts directly. CORS, CDN routing, and headers are the storefront engineer's responsibility.

## Page-load flow (six phases)

The boilerplate's `scripts/scripts.js` orchestrates this sequence — same shape EDS uses everywhere, with Commerce-specific phases added:

1. **`loadEager`** — synchronous decoration of above-the-fold content. Header skeleton, hero, LCP element. Lightweight; no third-party scripts; no drop-in initialization.
2. **`loadLazy`** — auto-block detection, decoration of below-the-fold sections, image lazy-loading boundaries, font swap.
3. **`loadDelayed`** — analytics, third-party scripts, observability. Fired ~3s after `loadLazy` or on user interaction.
4. **Block decoration** — each `blocks/{name}/{name}.js` invokes its `decorate(block)` function.
5. **Drop-in initialization** — within block decorators, drop-in initializers (`scripts/initializers/*`) call `setEndpoint`, `setFetchGraphQlHeaders`, and `initializers.mountImmediately(initialize, config)`.
6. **Event flow** — drop-ins publish `cart/*`, `checkout/*`, `pdp/*` on the event bus; other drop-ins and host blocks subscribe.

Implication: by the time `loadEager` finishes, the user sees rendered HTML (from authored content or AEM Commerce Prerender). Drop-ins hydrate during `loadLazy`. SEO crawlers see the prerendered HTML; clients see progressive enhancement.

## Blocks and the repository

EDS folder convention: every block is a self-contained folder under `blocks/`.

```
blocks/
└── commerce-cart/
    ├── commerce-cart.js   # decorate(block) — reads config, mounts drop-in
    ├── commerce-cart.css  # design-token-based styling
    └── _commerce-cart.json  # Universal Editor model
```

What's editable in authored content (DA.live / Google Doc / SharePoint):
- Page structure (sections separated by `---`)
- Block tables (block name in row 1, key/value pairs below)
- Block tables drive `readBlockConfig()` — key transforms to camelCase or stays kebab-case depending on read style
- Slot content (rich text, images, links) when surface to UE
- Page metadata (title, description, og tags via metadata block)

What's NOT editable (code-owned):
- `scripts/aem.js` — core EDS
- `scripts/scripts.js` — page lifecycle orchestration
- Drop-in package internals
- Block JS — but the block CAN read config from the authored content

## Drop-ins at a glance

A drop-in is the npm package (`@dropins/storefront-cart`, `@dropins/storefront-pdp`, etc.). It:
- Ships framework-agnostic Web Components.
- Exposes **containers** — composable renderable units (`CartSummaryList`, `OrderSummary`, `PaymentMethods`, `ProductGallery`).
- Exposes **functions** — imperative API (`addProductsToCart`, `setPaymentMethod`, `getRefinedProduct`).
- Publishes / subscribes on the **event bus** (`@dropins/tools/event-bus.js`).
- Reads from / writes to GraphQL via the **fetcher** (`@dropins/tools/fetch-graphql.js`).
- Uses **slots** (named extension points) for layered customization without forking.

Lifecycle inside a block:
```javascript
// blocks/commerce-cart/commerce-cart.js
import '../../scripts/initializers/cart.js';                       // sets endpoint + headers
import CartSummaryList from '@dropins/storefront-cart/containers/CartSummaryList.js';
import { render as provider } from '@dropins/storefront-cart/render.js';

export default async function decorate(block) {
  const config = readBlockConfig(block);
  await provider.render(CartSummaryList, {
    routeProduct: (item) => `/products/${item.url.urlKey}`,
    slots: { /* ... */ },
  })(block);
}
```

## Drop-in coordination

Multiple drop-ins on a page communicate exclusively via the event bus — no shared singleton store, no direct cross-imports.

| Event | Published by | Subscribed by |
|-------|-------------|--------------|
| `cart/data`, `cart/updated` | Cart | Mini-cart count, Checkout, Recommendations |
| `cart/merged` | Cart | Checkout (re-render after sign-in) |
| `cart/product/added` | Cart | Mini-cart toast, analytics |
| `authenticated` | User Auth | Cart (merge), Checkout, Account, Order |
| `auth/permissions`, `auth/group-uid` | User Auth | Companies, B2B drop-ins |
| `checkout/initialized`, `checkout/updated` | Checkout | Cart (re-render totals) |
| `checkout/error` | Checkout | ServerError container |
| `shipping/estimate` | Cart (EstimateShipping) / Checkout | Cross-listen |
| `pdp/data` | PDP | Recommendations, host CTAs |
| `pdp/valid`, `pdp/values` | PDP | Add-to-cart button states |
| `order/placed`, `order/data` | Order | Cart (reset), analytics |
| `search/loading`, `search/result`, `search/error` | Product Discovery | Pagination, sort, host UI |
| `wishlist/data`, `wishlist/alert` | Wishlist | Cart, PDP toggle |
| `quote-management/quote-data` | Quote Management | Checkout (B2B) |
| `requisitionList/alert` | Requisition List | Cart |
| `personalization/updated` | Personalization | Targeted blocks |
| `recommendations/data` | Recommendations | Host blocks |

Event bus API (`@dropins/tools/event-bus.js`):

```javascript
import { events } from '@dropins/tools/event-bus.js';

// Subscribe (eager: true replays last payload immediately)
events.on('cart/data', (cart) => { /* ... */ }, { eager: true });

// Emit
events.emit('checkout/updated', payload);

// Read last
const lastCart = events.lastPayload('cart/data');

// Scope (per-instance namespacing)
events.on('search/result', handler, { scope: 'quick-search' });

// Logging during dev
events.enableLogger(true);
```

## Backends

Three supported backend topologies — all run the same drop-ins, only `config.json` differs.

| Backend | Headers (`cs`) | SCP install | Use when |
|---------|----------------|-------------|----------|
| **Adobe Commerce as a Cloud Service (ACCS)** | `Magento-*` (no `x-api-key`, no `Magento-Environment-Id`) | Auto | Minimum ops, automatic upgrades, standard Commerce data model |
| **Adobe Commerce PaaS / on-prem** | `Magento-*` + `x-api-key` + `Magento-Environment-Id` | Composer manual | Custom monolith extensions, regulatory hosting |
| **Adobe Commerce Optimizer (ACO)** | `AC-View-ID`, `AC-Price-Book-ID`, `AC-Policy-*` | n/a (read-only) | Adobe merchandising on top of a different commerce platform |

Optionally, a fourth: split — Commerce Optimizer for reads + a separate Commerce core for writes. Set both `commerce-endpoint` (ACO) and `commerce-core-endpoint` (Adobe Commerce monolith).

## Commerce services

| Service | Endpoint host | What it provides | Required for |
|---------|---------------|------------------|--------------|
| **Catalog Service** | `commerce-endpoint` | Read-only product / category / inventory / price. ~10× faster than Core. SimpleProductView + ComplexProductView. | PDP drop-in (`@dropins/storefront-pdp`). Recommended for PLP. |
| **Live Search** | `commerce-endpoint` | Sensei-ranked search, facets, suggestions, typo tolerance | Product Discovery drop-in |
| **Product Recommendations** | `commerce-endpoint` | Sensei recommendation units by unit ID | Recommendations drop-in |
| **Payment Services** | `commerce-core-endpoint` (extension) | First-party PSP (cards, Apple Pay, PayPal) | Optional checkout PSP |
| **Adobe Sensei** | (none directly) | AI engine behind Live Search ranking + recs | Live Search, Recs |
| **Data Connection** | (ACDL → AEP) | Forwards shopper behavior to Adobe Experience Platform | Personalization, AEP-driven targeting |
| **Services Connector** | PaaS Admin module | Authenticates PaaS Commerce against Adobe SaaS services | PaaS only |

## Setting up a storefront

Sequence for a greenfield project:

1. **Create a repo from the AEM Boilerplate Commerce template** — `github.com/hlxsites/aem-boilerplate-commerce`.
2. **Connect AEM Code Sync** — links the repo to EDS preview/live publishing.
3. **Initialize authoring** — DA.live workspace OR Google Drive / SharePoint mountpoint.
4. **Stand up the Adobe Commerce backend** — ACCS instance OR PaaS + SCP composer install + Services Connector.
5. **Configure `config.json` locally** — endpoints, headers, analytics. Use the Config Generator UI at `https://da.live/app/adobe-commerce/storefront-tools/tools/config-generator/config-generator`.
6. **Run locally**: `npm install && npm start` (aem-cli serves on http://localhost:3000).
7. **Push to GitHub** — Code Sync auto-publishes to `*.aem.page` (preview) and `*.aem.live` (production).
8. **Move production config to the EDS Configuration Service** — delete `config.json` from `main`.

## Browser support and prerequisites

- Chrome / Edge: latest two versions.
- Safari: latest two versions on macOS + iOS.
- Firefox: latest two versions.
- ES2020+ baseline (drop-ins are modern Web Components — no legacy IE support).
- Node 22+ recommended for boilerplate dev (especially for AI agent skills).

## Source URLs

- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/before-you-start/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/architecture/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/architecture/how-a-page-loads/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/architecture/blocks-and-repo/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/architecture/drop-ins-at-a-glance/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/architecture/drop-ins-on-a-page/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/architecture/commerce-services-and-backends/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/backends/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/create-storefront/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/browser-compatibility/
