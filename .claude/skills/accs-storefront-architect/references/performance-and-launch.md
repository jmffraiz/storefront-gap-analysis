# Performance, SEO, Analytics, Launch

Targets, techniques, instrumentation, and go-live checklist for ACCS + EDS storefronts.

## Performance targets

Adobe sets explicit minimums — these are required, not aspirational:

| Metric | Target |
|--------|--------|
| Lighthouse Performance (mobile) | ≥ 90 |
| Lighthouse Accessibility | ≥ 90 |
| Lighthouse Best Practices | ≥ 90 |
| Lighthouse SEO | ≥ 90 |
| LCP (Largest Contentful Paint) | ≤ 2.5s on mobile 4G |
| INP (Interaction to Next Paint) | ≤ 200ms |
| CLS (Cumulative Layout Shift) | ≤ 0.1 |
| TBT (Total Blocking Time) | ≤ 200ms |
| First-byte (HTML) | ≤ 100ms (edge-cached) |

Page-type expectations:
- **PDP**: LCP element is typically the main product image. Prerender + AEM Assets CDN transform with explicit `width`/`height` to hit target.
- **PLP**: LCP is usually the first product card. Skeleton-render to avoid CLS during search load.
- **Homepage**: LCP is hero image / authored content. Standard EDS performance.
- **Cart / Checkout**: LCP usually the cart summary. Pre-fetch on prior page hover.

## Performance techniques

### Deferred drop-in loading

`@dropins/tools/initializer.js` provides `initializers.register()` (defers) vs `initializers.mountImmediately()` (immediate). Reserve `mountImmediately` for above-the-fold drop-ins (PDP main containers, PLP, cart on cart page). Defer mini-cart, wishlist toggle, recommendations rails until `loadLazy` or user interaction.

### Prerender PDPs (AEM Commerce Prerender)

App Builder service that generates static HTML for each SKU and publishes via the EDS BYOM API. Result: crawlers, AI assistants, link previews see fully-formed HTML. Hydration progresses without blocking LCP.

Mandatory for B2C SEO-driven catalogs. Optional for B2B catalogs hidden behind login.

### Image optimization

- AEM Assets CDN: explicit `width`, `height` query params drive responsive transforms.
- Fastly / EDS-served images: `?width=400&height=450&format=webp` or similar.
- Drop-ins accept `imageWidth` / `imageHeight` (Product Discovery, Recommendations) and `imageParams` (PDP gallery) for tight transforms.
- `loading="lazy"` on below-the-fold images is default.
- `fetchpriority="high"` for the LCP element.

### Font loading

Boilerplate uses `font-display: swap` and preloads critical fonts. Limit web font count — every variant is a separate request.

### Third-party isolation

`scripts.js` `loadDelayed` phase fires after main content paint or on first user interaction — gates analytics, chat widgets, marketing pixels. Never run third-party scripts in `loadEager` or `loadLazy`.

### GraphQL caching

- Catalog Service responses cached at Adobe edge.
- GET `/graphql` queries (read-only safelisted) cacheable at customer CDN (Fastly VCL).
- Same-origin GraphQL via Fastly VCL eliminates CORS preflight overhead (~50–100ms saved per call).

### Event-bus efficiency

- `{ eager: true }` for header badges and counters — saves a render cycle.
- One subscription per surface, not per component.
- Unsubscribe (`.off()`) when a block unmounts (rare in EDS but applies to SPA-style routing).

### Bundle size

- Drop-ins are ~50–150 KB gzipped each.
- `@dropins/tools` ~30 KB.
- Only import the containers you mount — drop-ins are tree-shakeable at the container level.

## PageSpeed common issues + fixes

| Issue | Fix |
|-------|-----|
| LCP > 2.5s on PDP | Enable AEM Commerce Prerender; preload LCP image; reduce hero image weight; set explicit `width`/`height` |
| LCP > 2.5s on PLP | Use Live Search edge cache; preload first row's images via `<link rel="preload">`; ensure SearchResults skeleton matches final card dimensions |
| CLS > 0.1 | Set explicit `width`/`height` on images; reserve space for late-loading drop-ins with skeleton; avoid web font swap-induced shifts |
| INP > 200ms | Defer drop-in mounts; debounce search input (default 300ms); avoid heavy synchronous work in event-bus subscribers |
| TBT > 200ms | Move analytics to `loadDelayed`; avoid third-party blocking scripts; tree-shake unused container imports |
| Lighthouse SEO failure | Add prerendered HTML; check meta tags; ensure JSON-LD on PDPs |
| Lighthouse Accessibility failure | Verify alt text on images; color contrast tokens; ARIA labels on icon buttons |

## SEO

### Indexing

- **PDPs**: indexed via AEM Commerce Prerender (server-rendered HTML, full JSON-LD).
- **PLPs**: indexed via authored EDS pages backed by `helix-sitemap.yaml`.
- **Search results (`?q=...`)**: typically `noindex` to avoid duplicate content.
- **Faceted PLPs**: use canonical tag pointing to the base category PLP.

### Sitemap

`helix-sitemap.yaml` declares the sitemap source. Adobe Commerce Prerender publishes to a routed location:

```yaml
sitemaps:
  default:
    source: /sitemap.json
    destination: /sitemap-content.xml
```

For multistore, generate per-locale sitemaps via the EDS Configuration Service.

### Metadata

Each prerendered PDP includes:
- `<title>` from product `metaTitle` (with fallback to `name`).
- `<meta name="description">` from `metaDescription`.
- `<meta name="keywords">` from `metaKeyword`.
- `<meta name="sku" content="ORIGINAL_CASING_SKU">` — drop-in reads this on init.
- Open Graph + Twitter card tags.
- JSON-LD `Product` schema (price, availability, brand, reviews if available).

PLP / category pages: metadata authored in EDS content (per-page metadata block).

### Structured data

`@type: Product` JSON-LD on PDPs. `@type: BreadcrumbList` for category navigation. `@type: WebSite` + `@type: SearchAction` on homepage for sitelinks search box.

## Analytics instrumentation

### Adobe Client Data Layer (ACDL)

Drop-ins automatically push to `window.adobeDataLayer` (or `window.dataLayer` if not present):
- `commerce.shoppingCartView` — Cart drop-in's `publishShoppingCartViewEvent()`.
- `commerce.productView` — PDP drop-in.
- `commerce.productAddToCart` — fires on `cart/product/added`.
- `commerce.productCheckoutStep` — Checkout drop-in's step transitions.
- `commerce.searchResults` — Product Discovery.

These events feed:
- Adobe Experience Platform (via Edge SDK + Datastream).
- Google Tag Manager (via dataLayer if configured).
- Custom analytics through bus listeners.

### Adobe Experience Platform (AEP) integration

- IMS Org + Datastream configured in `config.json` `analytics` block.
- Edge SDK (Web SDK) auto-loaded when `aep-datastream-id` is set.
- Sends events to AEP → Edge Network → Sensei (powers Live Search ranking, Recommendations).

### Custom analytics via event bus

```javascript
import { events } from '@dropins/tools/event-bus.js';

events.on('cart/product/added', (items) => {
  myAnalytics.track('Product Added', { items });
});
events.on('order/placed', (order) => {
  myAnalytics.track('Order Completed', { orderId: order.number, total: order.total });
});
```

### Magento Storefront Events (collector)

Adobe ships:
- `@adobe/magento-storefront-events-sdk` — events SDK.
- `@adobe/magento-storefront-event-collector` — collects + forwards to Live Search / Recs for AI training.

These are auto-loaded by the boilerplate. They power Sensei's personalization feedback loop — turning user behavior into better search ranking and recommendation relevance.

## Launch checklist

Pre-go-live items (from Adobe's launch page):

### Config

- [ ] `config.json` absent from `main` branch (or kept intentionally for sandbox).
- [ ] Production config in EDS Configuration Service.
- [ ] Endpoints point to production hosts.
- [ ] Headers correct for backend (PaaS / ACCS / ACO).
- [ ] `x-api-key` rotated within 90 days.

### Backend

- [ ] Storefront Compatibility Package matches Commerce version (PaaS only).
- [ ] B2B Compatibility Package installed if `commerce-b2b-enabled` (PaaS only).
- [ ] Services Connector configured (PaaS only).
- [ ] Catalog Service, Live Search, Product Recommendations, Payment Services provisioned.
- [ ] Sensei training catalog warmed.

### EDS

- [ ] DNS pointed via EDS-recommended CDN setup.
- [ ] `X-BYO-CDN-Type: fastly` header (if using customer Fastly).
- [ ] `X-Push-Invalidation: enabled`.
- [ ] Configuration Service `default-site.json` and `folders.json` deployed.
- [ ] Multistore folder mappings (if multistore).

### Performance

- [ ] Lighthouse mobile ≥ 90 on all four categories.
- [ ] LCP ≤ 2.5s on PDP, PLP, homepage, cart.
- [ ] INP ≤ 200ms.
- [ ] CLS ≤ 0.1.
- [ ] AEM Commerce Prerender running for PDPs.

### SEO

- [ ] Robots.txt allows crawl.
- [ ] Sitemap accessible.
- [ ] Meta tags + JSON-LD validated on PDP samples.
- [ ] Canonical tags on faceted PLPs.
- [ ] `noindex` on search-results pages.

### Analytics

- [ ] AEP datastream live.
- [ ] ACDL events firing.
- [ ] Event collector sending to Sensei.
- [ ] Custom analytics integrated.

### Security / Compliance

- [ ] CORS configured (or same-origin via VCL — preferred).
- [ ] reCAPTCHA enabled on auth flows.
- [ ] Customer token cookie `SameSite=None; Secure` if cross-origin.
- [ ] GDPR / CCPA cookie consent.
- [ ] PCI compliance — payment forms use Adobe Payment Services iframe or PSP-hosted.

### Functional smoke tests

- [ ] Anonymous browse → PDP → add to cart → checkout → place order.
- [ ] Sign up → confirm email → sign in.
- [ ] Forgot password → reset.
- [ ] Cart merge on sign-in.
- [ ] Multistore: locale switcher preserves cart.
- [ ] B2B: company registration → quote → PO → approval.
- [ ] Returns flow.
- [ ] Guest order lookup.

### Operational

- [ ] AEM Commerce Prerender scheduled actions deployed (`fetch-all-products`, `check-product-changes`, `mark-up-clean-up`).
- [ ] `AEM_ADMIN_API_AUTH_TOKEN` expiration tracked (1-year JWT).
- [ ] Drop-in NPM versions pinned; auto-update PR workflow configured.
- [ ] Cypress / e2e tests for critical journeys.
- [ ] Monitoring on Lighthouse score regressions (e.g., via SpeedCurve / Calibre).

## FAQ highlights

Per Adobe's troubleshooting FAQ — surprising answers worth remembering:

- **"Can drop-ins run on plain Adobe Commerce without EDS?"** Yes — drop-ins are framework-agnostic. EDS is the recommended host but not required.
- **"Do I need both `commerce-endpoint` and `commerce-core-endpoint`?"** Always set `commerce-endpoint` (for Catalog Service / Live Search / Recs). Set `commerce-core-endpoint` for write operations (cart, checkout, customer). On ACCS, both may point to similar hosts.
- **"Can I use Magento Open Source with the drop-ins?"** No — Storefront Compatibility Package is Adobe Commerce only.
- **"How often are drop-ins released?"** Weekly NPM PR via the bundled GitHub Action. Semver — minor / patch are non-breaking.
- **"Can I run a React app inside an EDS page?"** Yes — EDS doesn't restrict client-side frameworks. Most teams use vanilla / Web Components for consistency with drop-ins.
- **"Does Live Search work without Adobe Sensei?"** No — Live Search is a Sensei product. Without it, you fall back to Core `productSearch` (unranked).
- **"Can I use a non-Adobe PSP?"** Yes — slot any PSP into Checkout's `PaymentMethods.Methods.<code>` slot with `autoSync: false`, tokenize in `handlePlaceOrder`. See `references/dropin-checkout.md` for the Braintree example.
- **"Why is my SKU coming through lowercased?"** Reading from URL slug instead of `<meta name="sku">`. Switch to the meta tag set by AEM Commerce Prerender.
- **"How do I serve different prices to different customers?"** Use Adobe Commerce customer groups + segments (Personalization drop-in), OR Adobe Commerce Optimizer Price Books with `AC-Price-Book-ID` header (see `references/configuration.md`).

## Source URLs

- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/performance/
- https://experienceleague.adobe.com/developer/commerce/storefront/troubleshooting/pagespeed-issues/
- https://experienceleague.adobe.com/developer/commerce/storefront/troubleshooting/faq/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/launch/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/analytics/instrumentation/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/analytics/adobe-experience-platform/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/seo/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/seo/indexing/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/seo/metadata/
