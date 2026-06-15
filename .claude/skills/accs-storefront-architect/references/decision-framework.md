# Decision Framework

Synthesis layer. When the user is making a non-trivial architectural choice on an ACCS + EDS storefront, walk through the relevant decision axis below: what each option provides, where it breaks down, what gates it.

## Backend topology: ACCS vs Adobe Commerce PaaS vs Adobe Commerce Optimizer vs on-prem

| Axis | ACCS | Adobe Commerce PaaS | Adobe Commerce Optimizer (ACO) | On-prem |
|------|------|---------------------|-------------------------------|---------|
| Hosting | Adobe-managed SaaS | Adobe-managed cloud or self-managed | Adobe-managed SaaS | Customer-managed |
| SCP install | Auto | Composer (matching version) | n/a (read-only Optimizer) | Composer |
| SCP B2B | Auto | Composer | n/a | Composer (read docs for compatibility) |
| Services Connector | n/a | Required | n/a | Required |
| `x-api-key` / `Magento-Environment-Id` | Not used | Required on `cs` headers | n/a | Required |
| Header taxonomy | `Magento-*` | `Magento-*` | `AC-*` (`AC-View-ID`, `AC-Price-Book-ID`, `AC-Policy-*`) | `Magento-*` |
| Upgrades | Automatic | Manual via composer | Automatic | Manual |
| 2.4.9 SaaS feature (e.g., `exchangeOtpForCustomerToken`) | Yes | No | n/a | No |
| Cart / checkout / customer writes | Auto-routed | `commerce-core-endpoint` | Read-only — needs a Commerce core elsewhere | `commerce-core-endpoint` |

**Pick ACCS when:** the merchant is ready to commit to Adobe-managed SaaS, wants minimum operational overhead, wants automatic SCP updates, and uses the standard Commerce data model.

**Pick PaaS when:** the merchant has heavy customizations to the monolith, needs custom extensions or modules that don't run on SaaS, has a long-running cloud deployment, or is mid-migration from on-prem.

**Pick ACO when:** the merchant is using a different commerce platform for transactions but wants Adobe's merchandising stack (Catalog Service, Live Search, Recommendations) as a read-side overlay. ACO is read-focused — pair it with a write-capable Commerce core when carting/checkout is required.

**Pick on-prem when:** regulatory / compliance demands self-hosting. Expect higher operational cost and slower service onboarding (Services Connector still required).

## Drop-in extend vs create from scratch

Adobe's stated rule of thumb (in `extend-or-create`): default to extend. Creating from scratch is for genuinely novel domain experiences that don't map to any of the eight B2C + six B2B drop-ins.

**Extend** when: the domain (cart, checkout, PDP, PLP, search, auth, account, order, wishlist, recommendations, payment, personalization, company, quote, PO, requisition, quick order) is covered by a drop-in. Use slots, events, custom containers in the same block, dictionaries, theming tokens, models / transformers, and `overrideGQLOperations`.

**Create from scratch** when: the domain has no drop-in equivalent (e.g., gift registry, store locator with proprietary mapping, loyalty dashboard). Build on the Drop-in SDK (`@dropins/tools`) so the new drop-in inherits the event bus, the GraphQL helper, design tokens, and recaptcha hooks.

The cost of "create from scratch" is upgrade tax. The cost of "extend" can be reaching for a fork when slots don't expose the needed point. Slots and events are upgrade-safe; forks are not.

## Slots vs overrides vs fork

Three layers of drop-in customization, in order of increasing cost.

1. **Slots** — named extension points in containers (`Heading`, `Footer`, `Methods`, `Title`, `ProductAttributes`, `ConfirmDeleteBanner`, etc.). Append, replace, or decorate via `(ctx) => void` callbacks. Upgrade-safe.
2. **Events + sibling components** — listen on the event bus for state, render your own UI alongside the drop-in container. Upgrade-safe.
3. **Model transformers** — register a `transformer: (raw) => shape` in the initializer to add or reshape fields on the model the container consumes. Upgrade-safe but shallow-merge gotcha.
4. **`overrideGQLOperations`** — extend the GraphQL fragment with custom backend fields at build time. Upgrade-safe.
5. **Fork the container** — copy the container source, modify, re-render at a custom mount point. Loses upgrade safety. Use when slots cannot reach the extension point.
6. **Write your own block** that mounts only the headless API surface — bypass containers entirely. Heaviest, most flexible.

When in doubt, start with slots. Escalate when slots cannot solve the problem.

## Catalog Service vs Core GraphQL for product data

Default to Catalog Service (`commerce-endpoint`) for reads. It is up to ~10× faster than Core GraphQL `products`, schema-simplified into `SimpleProductView` / `ComplexProductView`, with discount-applied prices.

Fall back to Core (`commerce-core-endpoint`) only when:
- The product query needs a field Catalog Service hasn't exposed yet.
- The shopper context demands a feature still gated behind the monolith (rare; SCP is closing the gap).

The PDP drop-in (`@dropins/storefront-pdp`) is wired to Catalog Service by default.

## Live Search vs Core productSearch

Default to Live Search (via Product Discovery drop-in, `commerce-endpoint`). It provides Sensei-ranked relevance, typo tolerance (Levenshtein max distance 1), suggestions, related terms, and tracked `query_id` for click-back analytics. It is the only documented path for PLP / search in the boilerplate.

Fall back to Core `productSearch` only when Live Search is unavailable for the edition or licensing tier.

## Adobe Payment Services vs third-party PSP

Adobe Payment Services is bundled with ACCS-style licensing and integrates directly with the Checkout drop-in's `PaymentMethods.Methods.payment_services` slot.

**Pick Adobe Payment Services when:**
- Settlement + refund reporting must show in Commerce Admin natively.
- Apple Pay / Google Pay / PayPal / Venmo / standard cards cover the merchant's needs.
- The team wants minimum integration work.

**Pick a third-party PSP (Braintree, Adyen, Stripe, etc.) when:**
- The merchant has an existing PSP relationship and wants to preserve it.
- The merchant needs payment methods, currencies, or processor relationships Adobe doesn't cover.
- The PSP exposes a JS Drop-in / Elements API that can mount via `PaymentMethods.Methods.<code>` slot with `autoSync: false`.

Pattern for third-party PSPs: mount the PSP SDK in the slot, tokenize in `handlePlaceOrder`, then `setPaymentMethod` + `placeOrder`. See `references/dropin-checkout.md` for the canonical Braintree-style code.

## AEM Commerce Prerender vs client-only PDPs

**Prerender PDPs when:**
- SEO matters (most B2C). Crawlers and AI agents read full markup including JSON-LD without executing JS.
- LCP must be ≤ 2.5s on slow connections.
- Link previews (Slack, social, AI assistants) need product imagery and title.

**Skip prerender when:**
- The catalog is small and not SEO-driven (e.g., logged-in B2B catalog).
- The team can't operate the App Builder action (every 5 min change-check, every 60 min full sweep, 1-year JWT rotation).

When you do prerender, read the SKU from `<meta name="sku">` (Original case) rather than the URL slug (case-folded).

## Single-step vs multi-step checkout

**Single-step** is the default. Containers `autoSync: true` push each field change to the backend immediately. Lower complexity, fewer race conditions, faster perceived flow.

**Multi-step** when:
- The merchant requires it for brand/UX reasons.
- The checkout has more form fields than fit in a single screen (B2B, complex shipping rules).
- Reduced perceived complexity per step is more valuable than fewer total interactions.

Multi-step pattern: `autoSync: false` everywhere; step modules under `steps/`; class-driven visibility (`.checkout-step-active`); persistence in step's `continue()`. Boilerplate reference: `demos` branch, `blocks/commerce-checkout-multi-step`.

## Same-origin GraphQL (Fastly VCL) vs CORS

Adobe's strong recommendation: **avoid CORS in production.** Serve `/graphql` (and other Commerce paths) under the storefront origin via Fastly VCL rules. Benefits: no preflight, first-party cookies, simpler debugging, no `Access-Control-*` headers.

Use CORS only when the merchant lacks edge-routing capability (e.g., non-Fastly CDN, sandbox preview environment). Configure exact origins (no `*` with credentials), allow `OPTIONS`, include every custom header (`Magento-*`, `x-api-key`, `Authorization`).

## AEM Assets DAM vs Commerce-hosted product imagery

**Enable `commerce-assets-enabled: true` when:**
- The merchant manages product imagery in AEM Assets (DAM-backed workflow, asset metadata, brand-controlled cropping).
- A reliable SKU → asset alias is in place.
- AEM Assets CDN optimization (responsive resize, format negotiation) is needed.

**Leave disabled (or per-store override) when:**
- Imagery is uploaded directly to Commerce.
- The merchant doesn't operate AEM Assets.
- No asset matches the SKU — the drop-in falls back to the Commerce-hosted image transparently when on.

The slot-side wiring (`tryRenderAemAssetsImage`) only invokes if the flag is on.

## Same-store multilocale vs multi-store per-locale

**Same store, multiple store views** (locales sharing a store): use `headers.all.Store` and per-path overrides keyed by folder. Catalog and customers shared. Best for translation-only locale splits.

**Multiple stores** (region-specific catalog, currency, taxes): different `Magento-Store-Code` per override. Catalogs and price books can diverge. Best for region splits.

**Multi-website**: different `Magento-Website-Code`. Top-level isolation — separate domains, separate sitemaps, separate customer scopes. Pick this when regions need fully isolated experiences (e.g., US vs EU storefronts).

Same EDS repository handles all three via per-path overrides in `config.json` and `folders.json` mapping at the EDS Configuration Service.

## B2B Compatibility Package on or off

**Turn B2B on (`commerce-b2b-enabled: true`, install B2B SCP) when any of:**
- Companies (organizations with users, hierarchies, credit, roles).
- Negotiable quotes.
- Purchase orders with approval rules.
- Requisition lists (shared lists between buyers).
- Quick order (CSV / multi-SKU / variant grid).

**Leave off when:**
- B2C-only.
- Limited B2B needs (account-level pricing only) — the Optimizer Price Book pattern may suffice.

B2B drop-ins assume the B2B SCP is present. ACCS auto-installs it; PaaS requires composer install.

## Drop-in selection cheat sheet

| Use case | Drop-in |
|----------|---------|
| Product detail page | `@dropins/storefront-pdp` |
| Product list page, search | `@dropins/storefront-product-discovery` |
| Cart, mini-cart | `@dropins/storefront-cart` |
| Checkout | `@dropins/storefront-checkout` |
| Sign-in, register, password reset | `@dropins/storefront-auth` |
| My Account, addresses, payment tokens, seller-assisted buying | `@dropins/storefront-account` |
| Order detail, order history, cancellation, returns | `@dropins/storefront-order` |
| Wishlist | `@dropins/storefront-wishlist` |
| Recommendations rails | `@dropins/storefront-recommendations` |
| Personalized content blocks (audience-targeted) | `@dropins/storefront-personalization` |
| Adobe Payment Services UI (Apple Pay, cards) | `@dropins/storefront-payment-services` |
| Companies (B2B) | `@dropins/storefront-company-management` |
| Switching active company (B2B) | `@dropins/storefront-company-switcher` |
| Negotiable quotes (B2B) | `@dropins/storefront-quote-management` |
| Purchase orders + approval rules (B2B) | `@dropins/storefront-purchase-order` |
| Shared requisition lists (B2B) | `@dropins/storefront-requisition-list` |
| Quick order by SKU / CSV (B2B) | `@dropins/storefront-quick-order` |

## When to fork vs extend: decision rubric

Ask in order:
1. Is the customization a copy / styling / labeling change? → CSS + dictionaries.
2. Is it a small DOM insertion (add a CTA, swap a heading)? → slot.
3. Is it a behavioral side-effect tied to drop-in state? → event listener + sibling component.
4. Does it need an extra backend field? → `overrideGQLOperations` + transformer.
5. Does it replace the rendering of a specific area but not the whole container? → slot with `replaceWith`.
6. Is it a structural change to a container's internal logic? → fork the container.
7. Is it an entirely new commerce surface (gift registry, locator, loyalty)? → custom drop-in via SDK.

Forks and custom drop-ins are the only options where you take on upgrade responsibility. Everything above is upgrade-safe.

## Performance budget — what each piece costs

| Element | Approximate cost |
|---------|------------------|
| EDS HTML (cached at edge) | ≤ 100 ms TTFB |
| `@dropins/tools` initial load | ~30 KB gzipped |
| Per-drop-in package | ~50–150 KB gzipped |
| Catalog Service GraphQL roundtrip | ~50–150 ms (Adobe-hosted) |
| Core GraphQL roundtrip | ~100–300 ms (merchant origin) |
| Live Search productSearch | ~80–200 ms |
| AEM Commerce Prerender HTML | served as static, ≤ 100 ms TTFB |

Targets to hit (per `references/performance-and-launch.md`):
- LCP ≤ 2.5 s on mobile 4G.
- INP ≤ 200 ms.
- CLS ≤ 0.1.
- Lighthouse performance ≥ 90 on mobile.

To stay in budget:
- Prerender PDPs (HTML at the edge).
- Lazy-mount drop-ins not above the fold.
- Use `commerce-endpoint` (Catalog Service) for catalog reads.
- Avoid `refreshCart()` in hot paths.
- Eager-subscribe headers and badges to `cart/data`.
- Keep media via AEM Assets CDN with width/height transforms.

## Migration: Luma → EDS storefront

Adobe ships **Luma Bridge** (Product Discovery drop-in) for progressive migrations: shared session and cart between a Luma storefront and an EDS storefront, while account / checkout stay on Luma until you cut over.

Constraints:
- Luma Bridge is PaaS-only (EDS + Adobe Commerce + Adobe Commerce Optimizer scenario).
- Positioned as a progressive bridge, not a long-term architecture.

Typical sequence:
1. Stand up the EDS storefront with Catalog Service / Live Search / Product Discovery.
2. Enable Luma Bridge so the EDS site reuses Luma's cart/session cookies.
3. Migrate PDP + PLP traffic first (read-heavy, high SEO value).
4. Migrate cart, then checkout, then account.
5. Retire Luma.

## Last resort signals

If the user is heading toward any of these, raise it explicitly:
- **CORS in production** — push toward same-origin via Fastly VCL.
- **Custom search UI** — push toward Product Discovery + Live Search.
- **Custom rec engine** — push toward Product Recommendations + Sensei.
- **Building PDP with raw Core GraphQL** — push toward Catalog Service + the PDP drop-in.
- **Forking a drop-in container to add a button** — push toward a slot.
- **Committing `config.json` for prod while using Configuration Service** — push toward removing the file.
- **Hand-rolling B2B companies / quotes / POs** — push toward the B2B drop-ins + B2B SCP.
- **Skipping prerender for B2C catalog** — push toward AEM Commerce Prerender for SEO + LCP.
