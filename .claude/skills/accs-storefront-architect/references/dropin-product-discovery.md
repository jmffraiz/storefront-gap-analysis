# Product Discovery (PLP + Search) Drop-In Reference

`@dropins/storefront-product-discovery` (v3.1.0): unified search + category browse + facets + sort + pagination, backed by Adobe Live Search.

## Purpose

Powers all shopper-facing discovery surfaces:
- Search results page (PLP for `?q=...`).
- Category landing pages (PLP for a category context).
- Quick search popovers.
- Search redirects (term → URL).

Replaces all prior search / PLP options (Luma front-end, OpenSearch, Algolia integrations). Live Search via Adobe Sensei is the only sanctioned backing service.

## Initialization

Endpoint: `commerce-endpoint` (Adobe-hosted services). Headers: `cs` group.

```javascript
import { initialize, search } from '@dropins/storefront-product-discovery/api.js';
import { initializers } from '@dropins/tools/initializer.js';

await initializers.mountImmediately(initialize, {
  langDefinitions: { en_US: { Search: { /* ... */ } } },
  context: { /* customer group, view id */ },
  scope: 'main',     // for multiple discovery surfaces on one page
});
```

## Functions

Single public function:

```typescript
search(request: SearchRequest | null, options?: SearchOptions): Promise<SearchResult>
```

Passing `null` as request resets state (documented signal for state-reset).

`SearchRequest`:
- `phrase: string` — search term ('' for browse).
- `page_size: number`.
- `current_page: number`.
- `filter: SearchClauseInput[]` — `{ attribute, eq | in | range: {from, to} }`.
- `sort: { attribute, direction }[]`.

`SearchResult`:
- `items: ProductSearchItem[]` with embedded `productView: ProductView`.
- `facets: Facet[]` with buckets.
- `page_info: { current_page, page_size, total_pages }`.
- `total_count: number`.
- `suggestions: string[]`, `related_terms: string[]`.
- `query_id: string` (for click-back analytics).

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `search/loading` | emit | `boolean` |
| `search/result` | emit | `SearchResult` |
| `search/error` | emit | `Error` |

Host subscribes to these to update pagination, breadcrumbs, page title, or to emit ACDL events.

## Slots

Slots vary by container. Main surfaces are `SearchResults` (product grid/list) and `Facets` (filter panel).

`SearchResults`:
- `ProductCardImage` — custom thumbnail (e.g., wishlist overlay).
- `ProductCardTitle`, `ProductCardPrice`, `ProductCardActions`.

`Facets`:
- `FacetItem` — per-bucket rendering.
- `FacetHeader`, `ActiveFilters`.

## Containers

### SearchResults

The product grid / list.

| Prop | Type | Default | Purpose |
|------|------|---------|---------|
| `scope` | string | — | Multi-surface isolation |
| `skeletonCount` | number | `8` | Skeleton items during loading |
| `imageWidth` | number | `400` | CDN transform width |
| `imageHeight` | number | `450` | CDN transform height |
| `displayMode` | `'grid' \| 'list'` | `'grid'` | Layout |
| `routeProduct` | function | — | URL builder per item |

### Facets

Filter panel — checkbox / range / pinned filters.

| Prop | Type | Purpose |
|------|------|---------|
| `scope` | string | Isolation |
| `expandedByDefault` | boolean | Initial open state per facet |
| `onFacetChange` | callback | Notify host of selection |

### SortBy

Sort dropdown.

| Prop | Type | Purpose |
|------|------|---------|
| `scope` | string | Isolation |
| `options` | array | Sort options ({attribute, direction, label}) |

### Pagination

Pagination control.

| Prop | Type | Purpose |
|------|------|---------|
| `scope` | string | Isolation |

## Federated search

Pattern for cross-source search (e.g., products + content + FAQ in one results page):

1. EDS `helix-query.yaml` indexes content (FAQ, articles).
2. The PLP block fires `Promise.all([searchProducts, searchContent])` to fan out.
3. Aggregates results into a tabbed UI (e.g., "Products | Articles | FAQs").
4. Tab content renders the appropriate result set; `search/result` event still drives product pagination.

## Search redirects

Adobe Commerce Admin lets merchants configure term→URL redirects (e.g., "iphone" → `/products/phones/apple`). The drop-in exposes:

```javascript
import { preloadSearchRedirects, getSearchRedirectDestination } from '@dropins/storefront-product-discovery/api.js';

await preloadSearchRedirects();
const dest = await getSearchRedirectDestination('iphone');
if (dest) window.location.assign(dest);
```

**Critical**: must be solved client-side. EDS native redirects strip query strings and collapse all `/search?q=...` URLs to one path — the storefront has to consult the redirect map before rendering results.

## Luma Bridge

Progressive migration aid for moving from Adobe Commerce Luma front-end to EDS. Lets the EDS storefront share session and cart cookies with a Luma site while cart/checkout/account stay on Luma.

Constraints:
- **PaaS-only** — Adobe Commerce on PaaS + ACO + EDS scenario.
- Positioned strictly as a migration aid.
- Install via composer in the Luma project; configure CORS to allow EDS origin.

Workflow: stand up EDS PLP/PDP, point Luma Bridge at it for product discovery, keep checkout on Luma until full cut-over.

## Data export validation

Silent failure mode: if Commerce's SaaS Data Export breaks, Catalog Service / Live Search returns stale or empty results — the drop-in renders fine but with wrong data.

Validation tools:
- **Data Management Dashboard** in Commerce Admin → shows export sync status.
- GraphQL Postman collection (Adobe ships) → query Catalog Service / Live Search directly.

Routine: after a catalog change, check the dashboard for sync lag; before launch, run the Postman collection against production.

## Architectural notes

### PLP performance

- Live Search response is cached at edge (Adobe-hosted).
- `imageWidth` / `imageHeight` tight for above-the-fold cards reduces CDN transform cost.
- Skeleton-render (`skeletonCount`) prevents CLS during loading.
- Pagination prefers `current_page` over infinite scroll for SEO indexability.

### SEO for category pages

- AEM Commerce Prerender does NOT cover PLPs (only PDPs).
- Category pages rely on authored sections + JSON-LD emitted from host metadata.
- Use canonical URLs to fold faceted variants (e.g., `?color=red`) back to the base PLP.
- `?q=` search result pages typically `noindex` (low SEO value, high duplication risk).

### Faceted URLs

Storefront's responsibility to:
1. Read query params (`?color=red&size=m`) on mount.
2. Translate to `SearchRequest.filter` clauses.
3. On facet change, push new query params (no hash routing).
4. On back/forward, re-fire `search(...)` with the URL state.

### Scope for multi-surface

`scope: 'quick-search'` isolates a popover search instance from the main PLP. Each scope has its own event channel and state — globals-scope containers would cross-react.

### Slot discipline

Don't append into `onChange` — wrap once, mutate inside. Especially relevant on product cards where wishlist toggle state changes per render.

### Live Search vs Core productSearch

- Always prefer Live Search (this drop-in's default).
- Fall back to Core `productSearch` (Commerce GraphQL) only when Live Search is unavailable for the license/edition (rare).

## Source URLs

- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/quick-start/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/initialization/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/functions/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/events/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/dictionary/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/slots/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/styles/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/containers/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/containers/search-results/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/containers/facets/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/containers/sort-by/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/containers/pagination/
- https://experienceleague.adobe.com/developer/commerce/storefront/how-tos/federated-search/
- https://experienceleague.adobe.com/developer/commerce/storefront/how-tos/search-redirects/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/discovery/luma-bridge/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/discovery/data-export-validation/
