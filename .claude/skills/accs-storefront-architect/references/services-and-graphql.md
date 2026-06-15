# Adobe Commerce Backends and GraphQL Surfaces

Adobe Commerce ships a federated GraphQL stack: a high-performance read-only Catalog Service (and Live Search, Recommendations) that runs as SaaS on `commerce.adobe.io`, plus the monolith Adobe Commerce Core GraphQL endpoint that owns cart, checkout, customer and order data. EDS drop-ins call these endpoints from the shopper's browser through a single `fetchGraphQl` helper, routing per-call to `commerce-endpoint` or `commerce-core-endpoint` and stamping a scoped set of `Magento-*` (PaaS / ACCS) or `AC-*` (Adobe Commerce Optimizer) headers loaded out of `config.json` or the Configuration Service.

## Backend topology

Three supported Adobe Commerce backend topologies behind an EDS storefront. Drop-ins are identical across all three; only `config.json` (endpoints + headers) differs.

### Adobe Commerce PaaS (legacy "Adobe Commerce on Cloud" / on-prem)

- Self-managed (or managed-by-Adobe Cloud) instance of Adobe Commerce 2.4.7+.
- Merchant manually:
  - Installs the **Storefront Compatibility Package** — a PHP package that extends the Commerce GraphQL schema with the reads/writes that the cart, checkout, account, and order drop-ins require.
  - Connects each SaaS service through the **Services Connector** (authenticates Catalog Service, Live Search, and Product Recommendations using Admin-side API keys / Commerce IMS).
- Two GraphQL hosts in play:
  - `COMMERCE_CORE_ENDPOINT` — the monolith's own `/graphql` endpoint on the merchant's Commerce domain.
  - `COMMERCE_ENDPOINT` — Adobe-hosted SaaS GraphQL ("services" endpoint) at `https://commerce.adobe.io/...` fronting Catalog Service, Live Search, Product Recommendations.
- Headers include `x-api-key` and `Magento-Environment-Id` because the merchant must authenticate against the Adobe-hosted services side.

### Adobe Commerce as a Cloud Service (ACCS)

- Fully Adobe-managed SaaS. License bundles the Storefront and all merchandising services.
- Adobe sets up the hosted side: Services Connector, Storefront Compatibility schema, Catalog Service / Live Search / Recommendations provisioned automatically.
- 99.9% availability, auto-scaling, self-service provisioning via Commerce Cloud Manager, 30-day sandbox evaluation, automatic upgrades.
- GraphQL footprint is simpler than PaaS: a single Adobe-hosted GraphQL endpoint handles both catalog reads and core write operations. No `x-api-key` / `Magento-Environment-Id` on a vanilla ACCS storefront — auth is handled by Adobe at the edge.
- Storefront stays an EDS site built from the Adobe Commerce boilerplate; drop-ins are the same npm packages.

### Adobe Commerce Optimizer (ACO)

- Newest SaaS option, focused on read-side merchandising. License bundles the Storefront.
- Uses `AC-` prefixed headers (`AC-View-ID`, `AC-Price-Book-ID`, `AC-Policy-*`) instead of `Magento-` headers, because Optimizer was redesigned around catalog views / price books / attribute policies.
- Same drop-ins, same `fetchGraphQl` helper, different header contract.

### How the storefront connects

```
+------------------------------------------------------+
|  Edge Delivery Services (aem.live)                   |
|  - Renders authored docs (Google Drive / SharePoint) |
|  - Serves JS bundles for blocks + drop-ins           |
+-------------------+----------------------------------+
                    | browser-side fetches (HTTPS)
                    v
   +----------------+-------------------+
   | fetchGraphQl(query, vars, options) |
   | (from @dropins/tools/fetch-graphql)|
   +----------------+-------------------+
                    |
        +-----------+----------------+
        |                            |
        v                            v
+----------------------+   +---------------------------+
| COMMERCE_ENDPOINT    |   | COMMERCE_CORE_ENDPOINT    |
| (Adobe-hosted)       |   | (merchant Commerce inst., |
| Catalog Service,     |   |  via Storefront Compat)   |
| Live Search,         |   | Cart, Checkout, Customer, |
| Product Recs         |   | Orders, Wishlist, Account |
+----------------------+   +---------------------------+
```

EDS itself never proxies the GraphQL request. The browser hits the GraphQL hosts directly; EDS serves only HTML + JS. CORS, headers, and CDN caching are the storefront engineer's responsibility.

## Commerce services

### Catalog Service

- **What it does**: Fast, read-only catalog reads — products, attributes, inventory, prices, categories — for PDP, PLP, search, navigation. Up to ~10x faster than equivalent reads through Core GraphQL because it bypasses the monolith stack and serves from a SaaS data store kept in sync by the **SaaS Data Export** extension.
- **GraphQL host**: `COMMERCE_ENDPOINT` (Adobe-hosted services endpoint). Same host serves Catalog Service, Live Search, and Recommendations — different operations on the same federated schema.
- **Schema simplification**: Maps the seven Commerce product types into two response types:
  - `SimpleProductView` (simple, virtual, downloadable, gift card)
  - `ComplexProductView` (configurable, bundle, grouped)
  - Both implement `ProductView` so PDP code can treat them uniformly.
- **Price precision**: 16 digits, 4 decimal places. Returns both regular and final prices with discount math already applied.
- **Required for**: Product Details drop-in (`@dropins/storefront-pdp`). Strongly recommended for any PLP / navigation surface.
- **Synchronization**: Real-time via the SaaS Data Export extension on PaaS; automatic on ACCS / ACO.

### Live Search

- **What it does**: AI-powered product search (Adobe Sensei-backed). Ranked results, facets, suggestions, "search as you type", typo tolerance (Levenshtein, max edit distance 1: insertion / deletion / substitution / transposition).
- **GraphQL host**: `COMMERCE_ENDPOINT`.
- **Operation**: `productSearch` query. The Live Search `productSearch` is distinct from the Core GraphQL `productSearch` — drop-ins consistently route the Live Search variant through the services endpoint.
- **Required for**: Product Discovery drop-in (`@dropins/storefront-product-discovery`) — search results, facets, sort, paging.
- **Personalization**: Adjusts rankings based on session signals when Data Connection is wired up.

### Product Recommendations

- **What it does**: AI-driven merchandising units. Powered by Adobe Sensei.
- **GraphQL host**: `COMMERCE_ENDPOINT`.
- **Operation**: `recommendationsByUnitIds` query. Units are configured in the Commerce Admin and identified by `unitId`.
- **Required for**: Recommendations drop-in (`@dropins/storefront-recommendations`).
- **Signals**: Sends `pageType`, `currentSku`, `cartSkus`, `userPurchaseHistory`, `userViewHistory` to bias the ranking.

### Payment Services

- **What it does**: First-party Adobe payment processor. Credit / debit cards, digital wallets, PayPal products, card vaulting. Settlement / refund reporting synced to Commerce Admin.
- **GraphQL host**: Exposed through **Core GraphQL** (`COMMERCE_CORE_ENDPOINT`). The Checkout drop-in sets the payment method via `setPaymentMethodOnCart`; Payment Services injects its own additional inputs into the `PaymentMethodInput`.
- **Required for**: Checkout drop-in only when the merchant uses Adobe Payment Services. Optional.

### Adobe Sensei

Not directly callable from the storefront. The AI engine behind Live Search ranking, Product Recommendations, and personalization signals. No GraphQL endpoint of its own — the storefront talks to Sensei indirectly via Live Search / Recommendations queries on `COMMERCE_ENDPOINT`.

### Data Connection (optional, but coupled)

- **What it does**: Forwards shopper behavior + order data to Adobe Experience Platform. Drives Sensei training, personalization, AJO segmentation.
- **GraphQL host**: None. Client-side via the Adobe Client Data Layer (ACDL); drop-ins push events (`SHOPPING_CART_VIEW`, `PRODUCT_VIEW`, etc.) into ACDL automatically.
- **Required for**: Real personalization in Live Search / Recommendations.

### Services Connector (PaaS only)

PHP / Admin component that authenticates a PaaS Commerce instance against Adobe-hosted services using Admin API keys / Adobe IMS. Issues the `x-api-key` and `Magento-Environment-Id` that the storefront stamps onto requests to `COMMERCE_ENDPOINT`. Not present on ACCS / ACO.

## Catalog Service GraphQL

### Major queries

- **`products(skus: [String!]!): [ProductView]`** — bulk fetch by SKU. Used by `getProductData` and `getProductsData` in the PDP drop-in.
- **`refineProduct(sku: String!, optionIds: [String!]!): ProductView`** — variant resolution. Used by `getRefinedProduct` whenever the shopper picks a swatch.
- **`attributeMetadata(...)`** — metadata about attributes for filtering and faceting.
- **`productSearch(...)`** — Live Search operation (see Live Search section).

### Notable types

- **`ProductView` (interface)**: `sku`, `name`, `urlKey`, `description`, `shortDescription`, `images`, `videos`, `attributes` (EAV-style key/value list), `metaTitle`, `metaDescription`, `metaKeyword`, `inStock`.
- **`SimpleProductView`** — adds `price`, `priceRange` for simple-typed products.
- **`ComplexProductView`** — adds `priceRange`, `options` (attribute groups with selectable values and UIDs).
- **`ProductPrice`** — `regular` and `final` components, each with `amount.value` and `amount.currency`. Discounts pre-applied.
- **`ProductViewImage` / `ProductViewVideo`** — media records with `url`, `label`, `roles` (e.g., `image`, `thumbnail`, `small_image`).

### How the storefront PDP uses Catalog Service

1. Boilerplate `scripts/initializers/pdp.js` calls `setEndpoint(COMMERCE_ENDPOINT)` and `setFetchGraphQlHeaders(getHeaders('cs'))` before mounting the PDP drop-in.
2. PDP drop-in calls `fetchProductData(sku)` on mount, issuing `products(skus: [sku])` against `COMMERCE_ENDPOINT`.
3. Response is run through the `ProductModel` transformer.
4. Each swatch click triggers `getRefinedProduct(sku, optionsUIDs)` against the same endpoint and re-renders price / inventory / images.
5. Boilerplate ships a build-time metadata generator that hits `products(skus: […])` on Catalog Service to generate `<meta>` tags and JSON-LD for SEO.

## Live Search GraphQL

Live Search invokes `productSearch` on `COMMERCE_ENDPOINT`. Drop-in API surface wraps it in a single `search(...)` function in `@dropins/storefront-product-discovery`.

### `productSearch` query parameters

- `phrase: String` — search term. Empty / null means "browse" mode (category landing).
- `page_size: Int` — results per page.
- `current_page: Int` — page number (1-indexed).
- `filter: [SearchClauseInput!]` — filter conditions. Each clause has an `attribute` and one of `eq`, `in`, `range` (with `from` / `to`). `inStock` is filterable.
- `sort: [ProductSearchSortInput!]` — `{attribute, direction}` pairs. Default is relevance.
- `context: QueryContextInput` — customer group code, view ID, contextual signals that bias ranking.

### Response shape

`ProductSearchResult`:
- `items: [ProductSearchItem]` — each embeds a `productView: ProductView` (structurally compatible with Catalog Service's `products` query) plus `product` (legacy shape) and `highlights`.
- `facets: [Facet]` — each has `attribute`, `title`, `type` (`PINNED`, `INTERVAL`, `STATS`, etc.), and `buckets` (`ScalarBucket` / `RangeBucket` / `StatsBucket`).
- `page_info: { current_page, page_size, total_pages }`
- `total_count: Int`
- `suggestions: [String]` — typo / spell suggestions.
- `related_terms: [String]`
- `query_id: String` — opaque ID for click-tracking back to Live Search.

### Facets, paging, sort

- Facets configured in Commerce Admin (which attributes are filterable / facetable, weights). Popover "search as you type" returns `name`, `sku`, `category_ids`. URL convention (`?color=red`) is the storefront's job to map into `filter` clauses.
- Paging: `page_size` + `current_page`. The drop-in's `search/result` event delivers `page_info` for Pagination.
- `inStock` is filterable but not facetable.

### When to use Live Search vs Adobe Commerce GraphQL `productSearch`

- Always prefer Live Search for shopper-facing search / browse / PLP. Ranked, personalized, faster, and wired into Product Discovery.
- Fall back to Core `productSearch` only when (a) Live Search is unavailable for the edition, or (b) you need legacy schema fields that Live Search hasn't exposed yet (rare).
- Core `productSearch` runs against the monolith — same filters but no Sensei ranking, shares request cost with cart and checkout traffic.

## Adobe Commerce GraphQL (monolith)

Catalog Service does not replace Core GraphQL. The monolith stays authoritative for everything stateful: carts, checkout, customers, orders, wishlists, payment tokens. Drop-ins route those operations to `COMMERCE_CORE_ENDPOINT`.

### Cart drop-in (`@dropins/storefront-cart`)

Mutations:
- `addProductsToCart` — `addProductsToCart()` and PDP Add-to-Cart. Accepts `sku`, `quantity`, `parentSku`, `optionsUIDs`, `enteredOptions`.
- `updateCartItems` — `updateProductsFromCart()`. Changes quantity, swaps configurable variants (remove + add atomically).
- `applyCouponsToCart` / `removeCouponFromCart` — `applyCouponsToCart()` with `couponCodes` array and `ApplyCouponsStrategy` of `APPEND` or `REPLACE`.
- `applyGiftCardToCart` / `removeGiftCardFromCart` — with `giftCardCode`.
- `setGiftOptionsOnCart` — `setGiftOptionsOnCart()`, takes `GiftFormDataType`.
- `mergeCarts` — `getCustomerCartPayload()` after sign-in.
- `estimateTotals` / `estimateShippingMethods` — `getEstimatedTotals()` / `getEstimateShipping()`.

Queries: `customerCart`, `cart(cart_id: …)`, `storeConfig`, `countries`, `country(id: …)`.

### Checkout drop-in (`@dropins/storefront-checkout`)

Mutations:
- `setShippingAddressesOnCart` — `ShippingAddressInputModel`.
- `setBillingAddressOnCart` — `BillingAddressInputModel`.
- `setShippingMethodsOnCart` — `setShippingMethods()`, `ShippingMethodInputModel[]`.
- `setPaymentMethodOnCart` — `setPaymentMethodOnCart()`, `PaymentMethodInputModel`.
- `setGuestEmailOnCart` — `setGuestEmailOnCart()`.
- `placeOrder` — `placeOrder()`. Atomic helper `setPaymentMethodAndPlaceOrder()` fuses `setPaymentMethodOnCart` + `placeOrder`.
- `estimateShippingMethods` — `getEstimateShipping()`.

Functions: `authenticateCustomer()`, `getCart()`, `getCheckoutAgreements()`, `getCustomer()`, `getCompanyCredit()` (B2B), `getNegotiableQuote()` (B2B), `getStoreConfig()`, `initializeCheckout()`, `resetCheckout()`, `synchronizeCheckout()`, `isEmailAvailable(email)`, `setShippingMethods()`.

### User Auth drop-in (`@dropins/storefront-auth`)

- `generateCustomerToken` — `getCustomerToken()` (email + password). Sets `auth_dropin_user_token` cookie, stamps `Authorization: Bearer <token>` on subsequent GraphQL calls.
- `revokeCustomerToken` — `revokeCustomerToken()` on sign-out.
- `customer` (query) — `getCustomerData()`.
- `createCustomerV2` (or legacy `createCustomer`) — `createCustomer()`.
- `createCustomerAddress` — `createCustomerAddress()`.
- `requestPasswordResetEmail` — `requestPasswordResetEmail()`.
- `resetPassword` — `resetPassword()`.
- `confirmEmail`, `resendConfirmationEmail` — for account confirmation.
- `attributesForm` — `getAttributesForm()` for EAV form metadata.
- `getCustomerRolePermissions` (B2B) and `getAdobeCommerceOptimizerData` (returns Optimizer's `price book ID`).
- `verifyToken()` validates and emits the `authenticated` event.

### User Account drop-in (`@dropins/storefront-account`)

Queries: `customer`, `countries`, `country`, `attributesForm`, `storeConfig`, `customerPaymentTokens`.

Mutations: `updateCustomerV2`, `updateCustomerEmail`, `changeCustomerPassword`, `createCustomerAddress` / `updateCustomerAddress` / `deleteCustomerAddress`, `deletePaymentToken`.

Functions: `getAdminAssistanceActions()`, `getAttributesForm()`, `getCountries()`, `getCustomer()`, `getCustomerAddress()`, `getCustomerPaymentTokens()`, `deletePaymentToken()`, `getOrderHistoryList()`, `getRegions()`, `getStoreConfig()`, `removeCustomerAddress()`, `updateCustomer()`, `updateCustomerAddress()`, `updateCustomerEmail()`, `updateCustomerPassword()`.

### Order drop-in (`@dropins/storefront-order`)

Mutations: `cancelOrder`, `confirmCancelOrder`, `requestReturn`, `requestGuestReturn`, `confirmGuestReturn`, `requestGuestOrderCancel`, `placeOrder`, `placeNegotiableQuoteOrder` (B2B), `reorderItems`, `setPaymentMethodAndPlaceOrder` (atomic).

Queries: `guestOrderByToken`, `getGuestOrder`, `customer`, `attributesList`, `attributesForm`, `storeConfig`. Order-detail lookup via `getOrderDetailsById(orderNumber)`. Returns: `getCustomerOrdersReturn()`.

### How drop-ins call it (operationally)

- Each `scripts/initializers/<dropin>.js` calls `setEndpoint(COMMERCE_CORE_ENDPOINT)` (or `COMMERCE_ENDPOINT` for read-heavy drop-ins) and `setFetchGraphQlHeaders(getHeaders('all'))` before `initializers.mountImmediately(initialize, {...config})`.
- Once configured, every call inside the drop-in goes through `fetchGraphQl(query, options)` and inherits the configured endpoint + headers.
- Cart and Checkout listen for the `authenticated` event (emitted by user-auth) and swap the cart context from guest to customer (running `mergeCarts` if a guest cart ID was held).

## Authentication

### Customer token (shopper auth)

- Returned by `generateCustomerToken` against Core GraphQL.
- Stored in the `auth_dropin_user_token` cookie by the user-auth drop-in.
- Sent on subsequent Core GraphQL requests as `Authorization: Bearer <token>`. SDK auto-injects this when `verifyToken()` succeeds.
- Revoked by `revokeCustomerToken`. Cookie cleared, bearer header removed.

### Guest cart token

- Issued by `createEmptyCart` / `createGuestCart` (Core GraphQL). UUID-like string.
- Stored client-side (typically `localStorage` keyed by store-view).
- Passed as `cart_id` inside mutation variables on every guest cart operation. **Not** a request header — travels in GraphQL variables.
- After sign-in, folded into the customer cart via `mergeCarts(source_cart_id, destination_cart_id)`.

### Service-side API key (PaaS only)

- `x-api-key` on requests to `COMMERCE_ENDPOINT` (Catalog Service / Live Search / Recommendations). Provisioned via the Services Connector in Commerce Admin. Pinned to a `Magento-Environment-Id` (Adobe-Commerce-side tenant identifier issued at service onboarding).
- ACCS / ACO do not require this header — Adobe handles service-side auth at the platform layer.

### API Mesh / Edge Gateway

- **API Mesh** (Adobe Developer App Builder) composes Catalog Service + Core GraphQL + third-party APIs behind one endpoint. Storefront points `COMMERCE_ENDPOINT` (and optionally `COMMERCE_CORE_ENDPOINT`) at the Mesh URL.
- **Fastly VCL** (EDS CDN) is the production-recommended way to deliver GraphQL endpoints under the storefront origin, eliminating CORS preflights.

## SDK GraphQL helper

Lives in `@dropins/tools/fetch-graphql.js`.

### `fetchGraphQl` signature

```ts
fetchGraphQl(
  query: string,
  options?: {
    method?: 'GET' | 'POST';            // default 'POST'
    cache?: RequestCache;
    signal?: AbortSignal;
    variables?: Record<string, unknown>;
    headers?: Record<string, string>;   // merged on top of helper-level headers
    endpoint?: string;                  // overrides the configured endpoint
  }
): Promise<{ data: unknown; errors?: GraphQlError[] }>;
```

Module-level configuration:
- `setEndpoint(url: string)` — default endpoint for subsequent calls.
- `setFetchGraphQlHeader(name: string, value: string)` — set / replace one header.
- `setFetchGraphQlHeaders(headers: Record<string, string>)` — bulk-replace.
- `removeFetchGraphQlHeader(name: string)` — drop a header (e.g., `Authorization` on sign-out).
- `getHeaders(scope: 'all' | 'cs')` — boilerplate helper in `scripts/configs.js`.
- `getConfigValue(path: string)` — boilerplate helper for reading any dot-notation path from `config.json`.

### Boilerplate initializer pattern (Cart example)

```js
// scripts/initializers/cart.js
import { initializers } from '@dropins/tools/initializer.js';
import {
  setEndpoint,
  setFetchGraphQlHeaders,
} from '@dropins/tools/fetch-graphql.js';
import { initialize } from '@dropins/storefront-cart';
import { getConfigValue, getHeaders } from '../configs.js';

setEndpoint(await getConfigValue('commerce-core-endpoint'));
setFetchGraphQlHeaders(await getHeaders('all'));

await initializers.mountImmediately(initialize, {
  disableGuestCart: false,
  langDefinitions: { default: { 'AddToCart': 'Add to Bag' } },
  models: {},
});
```

PDP / Product Discovery / Recommendations initializers are structurally identical but call `setEndpoint(await getConfigValue('commerce-endpoint'))` and `setFetchGraphQlHeaders(await getHeaders('cs'))`.

### `initializers.mountImmediately(initialize, config)`

Universal entry point. Each drop-in exports an `initialize` `Initializer` with its own typed config:
- PDP: `langDefinitions`, `models` (each model has `initialData`, `transformer`, `fallbackData`), `scope`, `defaultLocale`, `globalLocale`, `sku`, `acdl`, `anchors`, `persistURLParams`, `preselectFirstOption`, `optionsUIDs`.
- Cart: `langDefinitions`, `models`, `disableGuestCart`.
- Auth / Account / Order: own typed configs.

Global helpers:
- `initializers.setImageParamKeys(params)` — maps logical image params to CDN-specific query keys. Defaults tuned for Fastly.
- `initializers.setGlobalLocale(locale)` — `'en-US'`, `'fr-FR'`, etc. Precedence: component prop → `setGlobalLocale()` → `navigator.language` → `'en-US'`.

### Drop-in event bus (`@dropins/tools/event-bus.js`)

- `events.on('<event>', (payload) => {}, options?)` — `eager: true` fires immediately with the last-seen payload; `scope: '<scope>'` namespaces events.
- `events.emit('<event>', payload, options?)`.
- `events.lastPayload('<event>', options?)`.
- `events.enableLogger(true)` — verbose dev logging.

## Headers contract

### `config.json` shape (Adobe Commerce PaaS)

```json
{
  "public": {
    "default": {
      "commerce-core-endpoint": "{{COMMERCE_CORE_ENDPOINT}}",
      "commerce-endpoint": "{{COMMERCE_ENDPOINT}}",
      "headers": {
        "all": {
          "Store": "{{STORE_VIEW_CODE}}"
        },
        "cs": {
          "Magento-Store-Code": "{{STORE_CODE}}",
          "Magento-Store-View-Code": "{{STORE_VIEW_CODE}}",
          "Magento-Website-Code": "{{WEBSITE_CODE}}",
          "x-api-key": "{{API_KEY}}",
          "Magento-Environment-Id": "{{ENVIRONMENT_ID}}"
        }
      }
    }
  }
}
```

### `config.json` shape (Adobe Commerce as a Cloud Service)

```json
{
  "public": {
    "default": {
      "commerce-core-endpoint": "{{COMMERCE_CORE_ENDPOINT}}",
      "commerce-endpoint": "{{COMMERCE_ENDPOINT}}",
      "headers": {
        "all": { "Store": "{{STORE_VIEW_CODE}}" },
        "cs": {
          "Magento-Store-Code": "{{STORE_CODE}}",
          "Magento-Store-View-Code": "{{STORE_VIEW_CODE}}",
          "Magento-Website-Code": "{{WEBSITE_CODE}}"
        }
      }
    }
  }
}
```

`x-api-key` and `Magento-Environment-Id` are omitted on ACCS — Adobe authenticates server-side.

### `config.json` shape (Adobe Commerce Optimizer)

```json
{
  "public": {
    "default": {
      "commerce-endpoint": "{{COMMERCE_ENDPOINT}}",
      "headers": {
        "all": {
          "AC-View-ID": "{{CATALOG_VIEW_ID}}",
          "AC-Price-Book-ID": "{{PRICE_BOOK_ID}}"
        }
      }
    }
  }
}
```

`AC-Policy-*` headers (optional) expose attribute-based filtering (per-shopper / per-segment).

### Required and optional headers (verbatim)

| Header                    | Scope        | PaaS | ACCS | ACO | Purpose |
|---------------------------|--------------|------|------|-----|---------|
| `Content-Type`            | both         | required | required | required | `application/json`; set by `fetchGraphQl`. |
| `Store`                   | `all`        | required | required | n/a | Store view code; selects store view on Core GraphQL. |
| `Magento-Store-Code`      | `cs`         | required | required | n/a | Store code on services endpoint. |
| `Magento-Store-View-Code` | `cs`         | required | required | n/a | Store view code on services endpoint. |
| `Magento-Website-Code`    | `cs`         | required | required | n/a | Website code on services endpoint. |
| `Magento-Environment-Id`  | `cs`         | required | not sent | n/a | Adobe-issued services-side tenant ID. |
| `x-api-key`               | `cs`         | required | not sent | n/a | API key from Services Connector. |
| `Authorization`           | `all`        | conditional | conditional | conditional | `Bearer <customer_token>` after sign-in. Managed by user-auth drop-in. |
| `Magento-Customer-Group`  | `cs`         | optional | optional | n/a | Customer group code; biases Catalog Service prices for shopper segments. |
| `AC-View-ID`              | `all`        | n/a | n/a | required | Optimizer catalog view identifier. |
| `AC-Price-Book-ID`        | `all`        | n/a | n/a | optional | Optimizer price book identifier. |
| `AC-Policy-*`             | `all`        | n/a | n/a | optional | Attribute-based filtering policies. |

### Header resolution flow

1. `getConfigValue(path)` reads from `config.json` if present (local / branch testing) or otherwise falls back to the **EDS Configuration Service**.
2. `getHeaders('cs' | 'all')` materializes the merged header map. `all` headers go on every request; `cs` headers are added when the request targets `commerce-endpoint`.
3. The initializer calls `setFetchGraphQlHeaders(...)`. From that point, every `fetchGraphQl(...)` request includes those headers.
4. The user-auth drop-in mutates the header set at runtime: `setFetchGraphQlHeader('Authorization', 'Bearer ' + token)` after sign-in; `removeFetchGraphQlHeader('Authorization')` after sign-out.

Do not commit production `config.json` to `main` — use the EDS Configuration Service for production.

## CORS implications

Drop-ins make cross-origin GraphQL requests from the EDS-served HTML origin (`*.aem.live`, `*.aem.page`, or the merchant's custom domain via EDS CDN) to:
- `COMMERCE_CORE_ENDPOINT` (e.g., `https://commerce.example.com/graphql` on PaaS).
- `COMMERCE_ENDPOINT` (e.g., `https://commerce.adobe.io/...`).

CORS preflight (`OPTIONS`) fires because requests are non-simple — they send `Content-Type: application/json` plus custom `Magento-*` / `x-api-key` / `Authorization` / `Store` headers.

### CORS configuration on PaaS

- Commerce instance must allow the storefront origin.
- Allowed methods: `GET`, `POST`, `OPTIONS`.
- Allowed headers must include: `Content-Type`, `Authorization`, `X-Requested-With`, plus every `Magento-*`, `x-api-key`, `Store` header in use.
- Wildcard origin (`*`) cannot be combined with credentialed requests (cookies / `Authorization`).
- After CORS config changes on PaaS, run `cache:flush`.

### CORS on ACCS / ACO

Adobe configures CORS at the platform level for the Adobe-hosted services endpoint. The merchant declares allowed origins through Commerce Cloud Manager / Optimizer admin.

### Production recommendation: same-origin delivery

Avoid CORS in production by serving the GraphQL endpoint under the storefront origin via **Fastly VCL** (EDS CDN). Pattern: `https://www.example.com/graphql` is VCL-rewritten to proxy to `COMMERCE_CORE_ENDPOINT` / `COMMERCE_ENDPOINT`. Eliminates preflights, simplifies cookies, allows first-party cookies for the customer token.

### Preflight verification

A working CORS config returns:
- `Access-Control-Allow-Origin: <storefront-origin>` matching `Origin`
- `Access-Control-Allow-Methods: GET, POST, OPTIONS`
- `Access-Control-Allow-Headers: <list including every custom header in use>`
- `Access-Control-Allow-Credentials: true` (when cookies / auth are sent cross-origin)

## Source URLs

- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/architecture/commerce-services-and-backends/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/backends/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/graphql/
- https://experienceleague.adobe.com/developer/commerce/storefront/reference/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/
- https://experienceleague.adobe.com/developer/commerce/storefront/get-started/architecture/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/commerce-configuration/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/cors-setup/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/initializer/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/events/
- https://experienceleague.adobe.com/en/docs/commerce/cloud-service/overview
- https://experienceleague.adobe.com/en/docs/commerce/catalog-service/overview
- https://experienceleague.adobe.com/en/docs/commerce/live-search/overview
- https://experienceleague.adobe.com/en/docs/commerce/product-recommendations/guide-overview
- https://experienceleague.adobe.com/en/docs/commerce/payment-services/overview
