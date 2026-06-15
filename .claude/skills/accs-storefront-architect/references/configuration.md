# Adobe Commerce + EDS Storefront — Configuration Reference

Dense technical reference for configuring an Adobe Commerce on Edge Delivery Services storefront: compatibility package, `configs.json`, multistore, AEM Assets, AEM Commerce Prerender, CDN routing, gated content, CORS.
Identifiers, header names, composer versions, environment variables, and file paths are preserved verbatim from Adobe's developer docs.

## Storefront compatibility package

### What it is

The Storefront Compatibility Package (SCP) is a composer-installable package that patches the Adobe Commerce codebase to support the EDS drop-in components (PDP, PLP, cart, checkout, account). It enhances the GraphQL schema with additional queries/mutations required by the drop-ins and bundles miscellaneous backend bugfixes. Drop-ins will not function correctly without it.

> "The Storefront Compatibility Package contains changes to the Adobe Commerce codebase that enable drop-in component functionality."

It applies to **Adobe Commerce only** — Magento Open Source is **not supported**. On Adobe Commerce as a Cloud Service (ACCS) the package is installed and updated automatically.

### Version matrix

| Commerce version | SCP version | Install method |
|------------------|-------------|----------------|
| 2.4.7            | `4.7.13`    | Composer (PaaS / on-prem) |
| 2.4.8            | `4.8.8`     | Composer (PaaS / on-prem); fixes up to `SCP-4.8.8` rolled into 2.4.9 |
| 2.4.9            | `4.9.0`     | Composer (PaaS / on-prem); requires Commerce Optimizer for SaaS |
| ACCS             | auto        | Installed and updated automatically; SaaS-only |

### Install steps — Adobe Commerce on cloud infrastructure (PaaS)

```bash
composer require adobe-commerce/storefront-compatibility:4.7.13   # 2.4.7
composer require adobe-commerce/storefront-compatibility:4.8.8    # 2.4.8
composer require adobe-commerce/storefront-compatibility:4.9.0    # 2.4.9

composer update "adobe-commerce/storefront-compatibility"

git add composer.json composer.lock
git commit -m "Install Storefront Compatibility Package"
git push
```

### Install steps — on-premises

```bash
composer require adobe-commerce/storefront-compatibility:4.8.8   # match version
bin/magento setup:upgrade
```

### Patch update procedure

```bash
composer update adobe-commerce/storefront-compatibility
bin/magento setup:upgrade && bin/magento cache:clean
```

### GraphQL surface added by SCP (per version)

**Common across versions (selection):**
- `clearWishlist` (mutation) — bulk wishlist removal
- `customerGroup` (query) — retrieve customer group data
- `customerSegments` (query) — retrieve customer segment data
- `deleteCustomerAddressV2` (mutation) — delete address by customer UID
- `updateCustomerAddressV2` (mutation) — update address by customer UID
- `exchangeExternalCustomerToken` (mutation) — exchange integration token for customer token + data
- `exchangeOtpForCustomerToken` (mutation) — admin seller-assisted buying: exchange OTP + email for customer token

**2.4.7 only (also added; later rolled into 2.4.8/2.4.9 core):**
- `guestOrder` (query) — guest order by order number, email, billing last name
- `guestOrderByToken` (query) — guest order via token
- `recaptchaFormConfig` (query) — reCAPTCHA configuration by form type
- `confirmCancelOrder` / `requestGuestOrderCancel` (mutations) — guest order cancellation
- `confirmReturn` / `requestGuestReturn` (mutations) — return requests
- `estimateTotals` (mutation) — cart totals estimate by address
- `generateCustomerToken` (mutation)
- `resendConfirmationEmail` (mutation)

**2.4.9 SaaS note:** `exchangeOtpForCustomerToken` is the headline addition. 2.4.9 is supported only on Adobe Commerce as a Cloud Service or Commerce Optimizer projects.

### B2B variant — Storefront Compatibility B2B Package

The B2B SCP layers on top of the base SCP and is **SaaS-only** (ACCS). It is installed and updated automatically.

**Prerequisite:** base Storefront Compatibility Package must be present.

**GraphQL additions:**

Mutations: `assignChildCompany` / `unassignChildCompany` — company hierarchy; `importSharedRequisitionList` — clone a shared requisition list via token; `shareRequisitionListByToken` — generate a shareable token; `shareRequisitionListByEmail` — email the list to colleagues; `setQuoteTemplateExpirationDate`; `placeNegotiableQuoteOrderV2` — quote → order with standardized error handling.

Queries: `negotiableQuoteTemplates` — templates with expiration metadata; `sharedRequisitionList` — read-only shared list by token.

## Commerce configuration

### File location and resolution order

The storefront reads configuration from two sources, in this precedence:

1. `/config.json` at the repository root (if present, wins)
2. EDS Config Service: `GET https://admin.hlx.page/config/{ORG}/sites/{SITE}/public.json`
3. EDS CDN fallback

**Workflow guidance:**
- **Local / feature branches:** commit `config.json` for fast iteration
- **Production / `main`:** ensure `config.json` is **absent** so the Config Service is used (centralized, no redeploy to change)
- Optionally: `echo "config.json" >> .gitignore`
- CDN caches with `max-age=7200` (2 hours); clear browser session storage when validating changes

**Tools:**
- Config Generator UI: `https://da.live/app/adobe-commerce/storefront-tools/tools/config-generator/config-generator`
- Reference `config.json`: `https://main--aem-boilerplate-commerce--hlxsites.aem.live/config.json`

### Top-level schema

```json
{
  "public": {
    "default": {
      "commerce-core-endpoint": "{{COMMERCE_CORE_ENDPOINT}}",
      "commerce-endpoint": "{{COMMERCE_ENDPOINT}}",

      "headers": {
        "all": { "Store": "{{STORE_VIEW_CODE}}" },
        "cs":  { /* Catalog/Commerce Services headers, see below */ }
      },

      "analytics": {
        "aep-ims-org-id": "{{IMS_ORG_ID}}",
        "aep-datastream-id": "{{DATASTREAM_ID}}",
        "base-currency-code": "{{CURRENCY_CODE}}",
        "environment": "{{ENVIRONMENT_TYPE}}",
        "store-code": "{{STORE_CODE}}",
        "store-id": 0,
        "store-view-code": "{{STORE_VIEW_CODE}}"
      },

      "commerce-assets-enabled": false,
      "commerce-b2b-enabled": false,
      "commerce-companies-enabled": false
    },

    "/en-ca/": { /* per-store-view override, see Multistore */ },
    "/fr/":    { /* per-store-view override */ }
  }
}
```

### Endpoint keys

- `commerce-core-endpoint` — Adobe Commerce Core GraphQL: handles read/write, mutations (cart, checkout, customer)
- `commerce-endpoint` — Catalog Service GraphQL: read-only, optimized for product/category data

These are typically different hosts; the drop-ins route reads to the Catalog Service and writes to Core.

### Headers — Adobe Commerce PaaS / on-prem

```json
{
  "headers": {
    "all": { "Store": "{{STORE_VIEW_CODE}}" },
    "cs": {
      "Magento-Store-Code":      "{{STORE_CODE}}",
      "Magento-Store-View-Code": "{{STORE_VIEW_CODE}}",
      "Magento-Website-Code":    "{{WEBSITE_CODE}}",
      "x-api-key":               "{{API_KEY}}",
      "Magento-Environment-Id":  "{{ENVIRONMENT_ID}}"
    }
  }
}
```

### Headers — Adobe Commerce as a Cloud Service (ACCS)

Same shape, but **omit** `x-api-key` and `Magento-Environment-Id` — ACCS does not require them.

### Headers — Adobe Commerce Optimizer

Optimizer uses a different header taxonomy keyed on catalog view and price book:

```json
{
  "headers": {
    "cs": {
      "AC-View-ID":       "{{CATALOG_VIEW_ID}}",
      "AC-Price-Book-ID": "{{PRICE_BOOK_ID}}",
      "AC-Policy-{{*}}":  "{{ATTRIBUTE_VALUE}}"
    }
  }
}
```

`AC-Policy-*` is a wildcard family for catalog policy attributes (entitlement, segmentation).

### Feature flags

| Flag | Effect |
|------|--------|
| `commerce-assets-enabled` | Enables AEM Assets image substitution in drop-in image slots |
| `commerce-b2b-enabled` | Enables B2B drop-in features (companies, quotes, requisition lists) |
| `commerce-companies-enabled` | Specifically gates the companies sub-feature |

### Analytics block

| Key | Source |
|-----|--------|
| `aep-ims-org-id` | IMS Org ID from Adobe Admin Console |
| `aep-datastream-id` | Datastream ID from Edge configuration |
| `base-currency-code` | Store view currency, e.g. `USD` |
| `environment` | Free-form environment tag (`prod`, `stage`) |
| `store-code` | Commerce store code |
| `store-id` | Numeric store ID |
| `store-view-code` | Commerce store view code |

### Reading config in storefront code

```javascript
import { getConfigValue, getHeaders } from '../../scripts/configs.js';

const endpoint = await getConfigValue('commerce-endpoint');
const headers  = await getHeaders('cs');                 // returns merged cs headers
const currency = await getConfigValue('analytics.base-currency-code');
const rootCat  = await getConfigValue('plugins.picker.rootCategory');
```

Dot notation is supported for nested keys. `getHeaders('cs')` merges the `all` block with the named group (`cs`).

### Secrets / sensitive values

`x-api-key`, `Magento-Environment-Id`, `aep-datastream-id`, and the IMS org are read by client-side JS — they are **not secrets** but are environment-scoped values. Genuine secrets (Commerce admin credentials, AEM admin tokens) belong in App Builder / repo secrets, never in `config.json`.

## Multistore

### Pattern: single repo, path-prefix per store view

One EDS repository serves multiple Commerce store views by mapping a root URL folder to each store view's headers. The `key` of each override **must match the root folder path of the corresponding store view** (with leading and trailing slashes).

```json
{
  "public": {
    "default": {
      "headers": {
        "all": { "Store": "en" },
        "cs": {
          "Magento-Store-Code":      "main_store",
          "Magento-Store-View-Code": "en",
          "Magento-Website-Code":    "base"
        }
      }
    },
    "/en-ca/": {
      "headers": {
        "all": { "Store": "en-ca" },
        "cs": {
          "Magento-Store-Code":      "ca-store",
          "Magento-Store-View-Code": "en-ca",
          "Magento-Website-Code":    "base"
        }
      }
    },
    "/fr-ca/": {
      "headers": {
        "all": { "Store": "fr-ca" },
        "cs": {
          "Magento-Store-Code":      "ca-store",
          "Magento-Store-View-Code": "fr-ca",
          "Magento-Website-Code":    "base"
        }
      }
    }
  }
}
```

### Locale folder convention

- Default root: `/en/` (US English in the boilerplate)
- Locale pattern: `language-code-region-code` (hyphenated), e.g. `/en-ca/`, `/fr-ca/`, `/de-de/`
- Per-store required files at each locale root: `index`, `store-switcher`, placeholder JSONs

### Folder mapping via the Config Service (EDS admin API)

EDS needs to know which content folder backs each path. Mapping is done by editing `folders.json` for the site.

**Workflow:**
1. Authenticate and obtain a token:
   ```
   POST https://admin.hlx.page/login
   ```
2. GET the current folder mappings:
   ```
   GET https://admin.hlx.page/config/{ORG}/sites/{SITE}/folders.json
   ```
3. POST the updated mappings (adding e.g. `/en-ca/products/`):
   ```json
   {
     "/en/products/":    "/en/products/default",
     "/en-ca/products/": "/en-ca/products/default",
     "/fr-ca/products/": "/fr-ca/products/default"
   }
   ```

> Critical: the POST **replaces** the entire `folders.json` object. Always start from the current GET response and add to it — never POST a partial document.

### Store switcher links

The store switcher document drives the language/region picker. To prevent EDS auto-localization rewriting the URL, append the `#nolocal` hash:

```
[Canada (EN)](https://site.com/en-ca/#nolocal)
[Canada (FR)](https://site.com/fr-ca/#nolocal)
[United States](https://site.com/en/#nolocal)
```

### Validation

- Browser session storage: the `config` key should contain the correct `Magento-Store-View-Code` for the current path
- Header value must exactly match the Commerce Admin store view code (case-sensitive)
- Folder names must be URL-safe and identical to the store view codes
- Cache `max-age=7200` applies — clear session storage during testing

### Multistore taxonomy variants

The same path-prefix mechanism handles:
- **Locale splits** — `/en/`, `/en-ca/`, `/fr-ca/`
- **Brand splits** — `/brand-a/`, `/brand-b/` (different `Magento-Store-Code`)
- **Region splits** — `/us/`, `/eu/`, `/apac/` (different `Magento-Website-Code`)
- **B2B/B2C splits** — separate store views, different `commerce-b2b-enabled` flag per override

## Price Book setup

Used to drive **Adobe Commerce Optimizer** Price Book ID propagation per logged-in customer.

### Header injected

```
AC-Price-Book-ID: {{PRICE_BOOK_ID}}
```

This header is set on the Catalog Service GraphQL fetch (the `cs` headers group).

### Enable Optimizer in the auth initializer

`scripts/initializers/auth.js`:

```javascript
initializers.register(initialize, {
  adobeCommerceOptimizer: true,
  // ...other auth options
});
```

### Replace customer-group header with Price Book header

`scripts/initializers/index.js` — replace the default `setCustomerGroupHeader` function with `setAdobeCommerceOptimizerHeader`:

```javascript
import { events } from '@dropins/tools/event-bus.js';
import { CS_FETCH_GRAPHQL } from '@dropins/tools/fetch-graphql.js';

function setAdobeCommerceOptimizerHeader({ adobeCommerceOptimizer }) {
  const priceBookId = adobeCommerceOptimizer?.priceBookId;
  if (priceBookId) {
    CS_FETCH_GRAPHQL.setFetchGraphQlHeader('AC-Price-Book-ID', priceBookId);
  } else {
    CS_FETCH_GRAPHQL.removeFetchGraphQlHeader('AC-Price-Book-ID');
  }
}

events.on('auth/adobe-commerce-optimizer', setAdobeCommerceOptimizerHeader, { eager: true });
```

The `eager: true` option ensures the handler is registered before any authentication event fires.

### API used

- `CS_FETCH_GRAPHQL.setFetchGraphQlHeader(name, value)` — set/replace a header on Catalog Service GraphQL
- `CS_FETCH_GRAPHQL.removeFetchGraphQlHeader(name)` — remove it (used on logout / no price book)

### Event

`auth/adobe-commerce-optimizer` fires on customer login. Payload includes `adobeCommerceOptimizer.priceBookId`.

### Troubleshooting checklist

- `adobeCommerceOptimizer: true` set in the auth initializer
- Default Price Book exists in the catalog view
- Customer record has a valid assigned Price Book
- Backend (Optimizer) recognizes the `AC-Price-Book-ID` header
- Price Book has active pricing rules
- `eager: true` set on the event subscription

## AEM Assets integration

DAM-backed product imagery: serve product images from AEM Assets with CDN optimization, falling back to Commerce-hosted images when no AEM asset matches the SKU.

### Enable in `config.json`

```json
{
  "public": {
    "default": {
      "commerce-assets-enabled": true
    }
  }
}
```

The flag is per-store override-able (you can enable AEM Assets only for certain locales/brands).

### Drop-in import

```javascript
import { tryRenderAemAssetsImage } from '@dropins/tools/lib/aem/assets.js';
```

### Use inside a drop-in image slot

```javascript
DropinImageSlot: (ctx) => {
  const { data, defaultImageProps } = ctx;
  tryRenderAemAssetsImage(ctx, {
    imageProps: defaultImageProps,
    params: {
      width:  defaultImageProps.width,
      height: defaultImageProps.height,
    },
  });
}
```

### Helper surface (`@dropins/tools/lib/aem/assets.js`)

| Function | Purpose |
|----------|---------|
| `isAemAssetsEnabled()` | Boolean: reads `commerce-assets-enabled` from config |
| `generateAemAssetsOptimizedUrl(assetUrl, alias, params)` | Builds a CDN-optimized AEM Assets URL |
| `makeAemAssetsImageSlot(config)` | Factory for a slot renderer |
| `tryRenderAemAssetsImage(ctx, config)` | Attempts AEM Assets render, falls back transparently |

### Parameters

- `alias` — product SKU; used as the matching key against AEM Assets
- `imageProps` — default props from the drop-in context (src, alt, sizes)
- `params.width` / `params.height` — render-target dimensions (drive AEM Assets transform)
- `wrapper` — optional anchor element to wrap the `<img>` for linked images

### Behavior

- Flag on + matching asset in AEM → AEM Assets image with CDN optimization params
- Flag on + no match → Commerce-hosted image is rendered unchanged
- Flag off + URL points to AEM Assets → likely 400 / broken image (Commerce won't transform AEM URLs); keep the flag aligned with where the product imagery actually lives

## AEM Commerce Prerender

App Builder application that generates server-rendered HTML for PDPs and publishes the markup to EDS via the BYOM (Bring Your Own Markup) API. Crawlers, AI agents, and link previews see complete product markup before JS executes.

> SEO benefit: search engines and AI systems read full product data without executing the SPA. Improves LCP for first paint and unlocks high-quality structured data.

### Repository

- Source: `https://github.com/adobe-rnd/aem-commerce-prerender`
- Management UI: `https://prerender.aem-storefront.com`
- Runbook: `https://github.com/adobe-rnd/aem-commerce-prerender/blob/main/docs/RUNBOOK.md`

### What is prerendered

Product Detail Pages only (catalog browse / PLP, account, checkout remain dynamic). Each SKU yields a published HTML doc at the templated URL.

### Install

```bash
git clone https://github.com/adobe-rnd/aem-commerce-prerender.git
cd aem-commerce-prerender
npm install
npm run setup     # interactive wizard, writes .env
npm run deploy    # deploys to Adobe App Builder
```

### Environment variables (`.env`)

| Variable | Purpose | Example |
|---|---|---|
| `ORG` | GitHub organization / username | `adobe` |
| `SITE` | Repository name | `your-storefront` |
| `CONTENT_URL` | AEM content endpoint | `https://main--your-storefront--adobe.aem.live` |
| `PRODUCTS_TEMPLATE` | Product template URL | `https://main--your-storefront--adobe.aem.live/products/default` |
| `PRODUCT_PAGE_URL_FORMAT` | URL pattern with tokens | `/{locale}/products/{urlKey}` |
| `LOCALES` | Comma-separated locales | `en-us,fr-fr` |
| `AEM_ADMIN_API_AUTH_TOKEN` | Long-lived AEM admin token (1-year expiration) | (generated) |

### Scheduled actions

| Action | Schedule | Purpose |
|---|---|---|
| `fetch-all-products` | every 60 min | discover new catalog items |
| `check-product-changes` | every 5 min | detect updates, publish |
| `mark-up-clean-up` | every 60 min | unpublish removed products |

### Manual invocation

```bash
aio rt action invoke aem-commerce-ssg/fetch-all-products
aio rt action invoke aem-commerce-ssg/check-product-changes
aio rt action invoke aem-commerce-ssg/mark-up-clean-up
```

### Storefront-side integration

#### PDP block location

`blocks/product-details/product-details.js`

#### SKU retrieval pattern

The prerendered HTML includes:
```html
<meta name="sku" content="ORIGINAL_CASING_SKU">
```

Read from the meta tag, **not** the URL — URL parsing lower-cases the SKU and breaks case-sensitive backends:

```javascript
const sku = document.querySelector('meta[name="sku"]')?.content;
```

#### Drop-in initialization

```javascript
import { initializers } from '@dropins/tools/initializer.js';
import { initialize }   from '@dropins/storefront-pdp';

const sku = getProductSku();
await initializers.mountImmediately(initialize, { sku });
```

### Customization points

| Element | Location | Purpose |
|---|---|---|
| HTML templates | `actions/pdp-renderer/templates/product-details.hbs` | Markup structure |
| JSON-LD / structured data | `actions/pdp-renderer/ldJson.js` | SEO structured data |
| GraphQL queries | `actions/queries.js` | Product data retrieval |
| Rendering logic | `actions/pdp-renderer/render.js` | Data transformation |

### Token rotation

The `AEM_ADMIN_API_AUTH_TOKEN` is a JWT valid for **1 year**.
1. Decode at `https://jwt.io` to read `exp`.
2. Generate a replacement via the AEM project admin panel.
3. Update `.env`, redeploy: `npm run deploy`.

### Build issue: missing `maxVersion`

Each entry in `app.config.yaml`'s `productDependencies` block must declare both `minVersion` and `maxVersion`:

```yaml
productDependencies:
  - code: PRODUCT_CODE
    minVersion: MIN_VERSION
    maxVersion: MAX_VERSION
```

## CDN

The CDN sits in front of both EDS and the Commerce backend, splitting traffic by path. Boilerplate guidance assumes **Fastly** (with provided VCL snippets), but the same routing rules apply to any equivalent edge.

### Backends

| Backend | Host | Purpose |
|---|---|---|
| EDS | `main--aem-boilerplate-commerce--hlxsites.aem.live` | Storefront markup, JS, CSS, content |
| Catalog Service | `https://catalog-service.adobe.io/graphql` | Product/category GraphQL (prod) |
| Catalog Service (sandbox) | `https://catalog-service-sandbox.adobe.io/graphql` | Non-prod GraphQL |
| Commerce Core | Adobe Commerce origin | `/graphql` mutations, `/rest`, `/media`, `/checkout`, `/customer`, `/admin`, `/static` |

### VCL snippet priorities

Lower priority numbers execute first.

| Subroutine | Priority | Purpose |
|---|---|---|
| `recv` | 4 | Redirects (e.g. apex → www) |
| `recv` | 100 | Backend routing |
| `pass` / `miss` | 30 | Backend selection on uncached requests |
| `fetch` | 30 | Response handling |
| `deliver` | 30 | Outbound headers / cookie scrubbing |

Custom VCL snippets live under: `var/vcl_snippets_custom/` (perm `775`).

### Routing rules — full storefront pattern

Default: route to EDS. Carve out Commerce paths.

**Commerce request detection (regex):**
```
^/(graphql|rest|oauth|media|checkout|customer|admin|static)($|/)
```

**Header flag set on Commerce-bound requests:**
```
x-commerce: true
```

**Notable rules:**
- `/graphql` → Commerce Core (mutations) — GET queries should hit cache (HIT) for read-only safelisted operations
- `/media/**` → Commerce (product imagery) — even when content is mostly EDS-served
- All other paths → EDS

### Host header management

EDS requires the request `Host` header to be the EDS-side host:
```
Host: main--aem-boilerplate-commerce--hlxsites.aem.live
```

### Required EDS-side headers on the backend definition

```
X-BYO-CDN-Type: fastly
X-Push-Invalidation: enabled
```

### Cache configuration

- **Surrogate keys:** validated via the `surrogate-key` response header. Required for granular invalidation.
- **Compression:** JS and HTML responses must return `Content-Encoding: gzip` or `br`.
- **GraphQL:** GET requests to `/graphql` (read-only safelisted) should be cacheable and hit the edge cache; POST mutations bypass.

### Cookie scrubbing (API Mesh compatibility)

When using API Mesh, retain only these cookies on backend requests; strip all others:
- `PHPSESSID`
- `X-Magento-Vary`
- `form_key`
- `private_content_version`
- `mage-messages`
- `persistent_shopping_cart`
- `fastly_geo_store`

### Commerce Admin settings

- **Stores → Configuration → General → Web → Url Options → Auto-redirect to Base URL:** set to **No**. The CDN handles canonical redirects via VCL.
- `base_url` and `secure_base_url` must include the full canonical domain (with `www` where relevant).

### Validation

```bash
# Confirm EDS-side static asset is served gzipped from the CDN
curl -sI -H 'Fastly-Debug: 1' https://domain.com/scripts/aem.js

# Confirm GraphQL GET is cacheable
curl -sI 'https://domain.com/graphql?query=...'
```

Look for `x-cache: HIT`, `content-encoding: br|gzip`, and the `surrogate-key` header.

## Gated content

Restrict EDS pages to authenticated Commerce customers, with a fallback redirect to login when the visitor lacks a valid token. Enforcement runs at the **edge** (Fastly Compute reference solution), not in the client.

### Authentication model

> "Gated content is displayed only if the customer is logged in and their authentication token is valid."

The edge service validates the customer token by calling Adobe Commerce GraphQL (e.g. the `customer` query) — any auth-confirming query/mutation works.

### Gated URL inventory — `header` document

URLs to gate are listed in a spreadsheet inside the EDS `header` document with three columns:

| Column | Purpose |
|---|---|
| `url` | Path to restricted content; wildcards supported |
| `key` | Custom evaluation identifier |
| `value` | Comparable value (typically `True`) |

The edge fetches this metadata to determine whether the incoming path is gated.

### Edge flow

1. Request hits CDN.
2. Edge compute fetches header metadata from EDS.
3. If path matches a gated URL pattern → require valid customer token.
4. Edge validates the token via Commerce GraphQL (`customer` query).
5. On valid token → proxy to EDS, serve content.
6. On missing/invalid token → redirect to the designated fallback (login page).

### Reference implementation

Adobe ships a downloadable Fastly Compute starter: `fastly-compute-sample.zip`. Contains starter kit configuration and `index.js` (request routing + auth logic for gated URLs).

> The sample is **reference-only and unsupported by Adobe**. Productionizing requires hardening, logging, and a chosen edge platform.

### Patterns

- **Member-only landing pages** — fully restricted; redirect to login.
- **Tiered access** — extend `key`/`value` to encode customer group requirements; edge logic compares against group claims in the token response.
- **Paywall preview** — render a public stub via a separate non-gated URL; the gated URL serves the full document only post-auth.

## CORS

CORS only matters when the storefront origin differs from the Commerce backend origin. Adobe's strongest recommendation:

> "CORS should be a last resort for production. Prefer same-origin delivery."

Serve both from the same domain via CDN proxy routing (see the CDN section) and CORS disappears as a concern.

### When CORS is unavoidable

The `/graphql` endpoint is the primary endpoint requiring CORS configuration.

### Required response headers

| Header | Value |
|---|---|
| `Access-Control-Allow-Origin` | Exact storefront origin (no wildcard with credentials) |
| `Access-Control-Allow-Methods` | `GET, POST, OPTIONS` |
| `Access-Control-Allow-Headers` | `Content-Type, Authorization, X-Requested-With` |

### Example origins

- Local dev: `http://localhost:3000`
- Production: `https://your-storefront.com`

Origins **must include protocol and port** and have **no trailing slash**.

### Implementation options

1. **Web server (Nginx/Apache)** — recommended for production
2. **Community module** — `graycore/magento2-cors`
3. **Custom Adobe Commerce module**

After any change: `bin/magento cache:flush`.

### Constraints

- **No `*` with credentials:** `Access-Control-Allow-Origin: *` is invalid when `credentials: include` is used. Specify exact origins.
- **OPTIONS required:** preflight requests must be answered. Include `OPTIONS` in `Access-Control-Allow-Methods`.
- **Docker / multi-host:** add every hostname variant in use.

### Preflight test

```bash
curl -I -X OPTIONS https://your-commerce-backend.com/graphql \
  -H "Origin: YOUR_STOREFRONT_URL" \
  -H "Access-Control-Request-Method: POST"
```

Expect a `2xx` response carrying `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`.

### Troubleshooting checklist

| Symptom | Cause | Fix |
|---|---|---|
| `No 'Access-Control-Allow-Origin' header is present` | CORS not configured / origin not allowed / cache stale | Enable CORS, add origin, `bin/magento cache:flush` |
| `Access-Control-Allow-Origin ... not equal to the supplied origin` | Origin missing from allowed list, or trailing slash mismatch | Add **exact** origin (protocol + port, no slash) |
| `must not be the wildcard '*' when credentials mode is 'include'` | Using `*` with credentialed requests | Replace with explicit origin |
| Preflight 405 | Web server blocking `OPTIONS` | Allow `OPTIONS` at the server layer |
| Module appears installed but headers missing | Module disabled or cache stale | `bin/magento module:status \| grep -i cors`, then `cache:flush` |
| Intermittent failures | Multiple hostnames in use (Docker) | Add every variant to allowed origins |

## Source URLs

- https://experienceleague.adobe.com/developer/commerce/storefront/setup/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/multistore-setup/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/storefront-compatibility/install/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/storefront-compatibility/v249/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/storefront-compatibility/v248/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/storefront-compatibility/v247/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/storefront-compatibility-b2b/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/commerce-configuration/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/price-book-setup/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/aem-assets-configuration/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/aem-prerender/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/content-delivery-network/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/gated-content/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/cors-setup/
- https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/cors-troubleshooting/
