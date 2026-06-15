# <Page name> — Gap Analysis

> **Mockup source:** `<file>` (<Figma frame names>)
> **Variants/states included:** <Default / Filter / OOS / Empty / …>
> **Scope:** **IN:** <what this doc covers>. **OUT:** <what is excluded>.
> **Author:** <name> · **Date:** <YYYY-MM-DD>

The storefront sits on **Adobe Commerce as a Cloud Service (ACCS) + Edge Delivery Services (EDS)** using the **AEM Boilerplate Commerce** repo. Each page is hosted by a thin EDS block that mounts one or more `@dropins/storefront-*` drop-ins.

> **Documentation links:** Every drop-in package name, container name, and named Adobe service mentioned in this document must be hyperlinked to its official Experience League page. Use the URL reference table at the end of this document. Format: `[name](url)`.

**Complexity buckets** (used in §2):
- **Low** — theming / config / authored content / single slot fill.
- **Medium** — new block logic, multi-slot wiring, drawer/state coordination, or initializing one additional drop-in.
- **High** — multi-block coordination plus a new service dependency or external-system integration.

---

## 1. Feature decomposition

One row per discrete feature observed in the design.

| # | Feature | Description in the design | Variant/state | Type | Observations |
|---|---------|---------------------------|---------------|------|--------------|
| F1 | <short noun phrase> | <what the shopper sees / does, neutral of implementation> | <Default / Filter / OOS / All> | <Commerce / Authored content / Cross-cutting> | <caveats, ambiguities, edge cases> |
| F2 | | | | | |

---

## 2. Gap analysis

Legend — **Coverage:** ✅ provided · 🟡 partial (slot / theme / extend) · ❌ none.

| # | Feature | Existing drop-in / block | Coverage | What it gives OOTB | Gap to close | Touch points | Dependencies | Complexity | Risk |
|---|---------|--------------------------|:--------:|--------------------|--------------|--------------|--------------|:----------:|:----:|
| F1 | <name> | [`@dropins/storefront-…`](<doc-url>) / block / `none` | <✅ / 🟡 / ❌> | <containers, slots, events, props — concrete names, each linked to its doc page> | <one sentence: what to build / wrap / restyle> | <blocks/slots/files expected to change> | <backend prereqs, third-party SDKs, design-token additions> | <Low/Medium/High> | <Low/Med/High> |
| F2 | | | | | | | | | |

---

## 3. Overall assumptions and open questions

### 1. Assumptions 
   -
   -
   -

### 2. Open questions

Decisions blocking the estimate. Each carries a recommendation and the impact of each option.

1. **<Question>?**
   - *Recommendation:* <option>.
   - *Impact A — <option>:* <scope / complexity consequence>.
   - *Impact B — <option>:* <scope / complexity consequence>.

## 4. Explicitly out of scope

What the estimate must **not** include — protects against scope creep in the estimation session.

- <feature / capability / integration that is not part of this gap>.

---

## Documentation URL reference

Use these canonical URLs when hyperlinking drop-ins, containers, and services in the document above.

### Drop-ins (B2C)

| Drop-in / concept | Root URL |
|-------------------|----------|
| All drop-ins — introduction, extend/create, styling, branding, events | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/introduction/ |
| **Product Details** (`@dropins/storefront-pdp`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/ |
| **Product Discovery** (`@dropins/storefront-product-discovery`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/ |
| **Cart** (`@dropins/storefront-cart`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/ |
| **Checkout** (`@dropins/storefront-checkout`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/ |
| **User Account** (`@dropins/storefront-account`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/user-account/ |
| **User Auth** (`@dropins/storefront-auth`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/user-auth/ |
| **Order** (`@dropins/storefront-order`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/order/ |
| **Wishlist** (`@dropins/storefront-wishlist`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/ |
| **Payment Services** (`@dropins/storefront-payment-services`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/payment-services/ |
| **Personalization** (`@dropins/storefront-personalization`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/personalization/ |
| **Recommendations** (`@dropins/storefront-recommendations`) | https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/ |

### Drop-ins (B2B)

| Drop-in | Root URL |
|---------|----------|
| B2B drop-ins overview | https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/ |
| **Company Management** | https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/company-management/ |
| **Company Switcher** | https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/company-switcher/ |
| **Purchase Order** | https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/purchase-order/ |
| **Quote Management** | https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/quote-management/ |
| **Requisition List** | https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/requisition-list/ |
| **Quick Order** | https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/quick-order/ |

### Configuration & services

| Topic | URL |
|-------|-----|
| Storefront setup overview | https://experienceleague.adobe.com/developer/commerce/storefront/setup/ |
| Commerce configuration (`config.json`) | https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/commerce-configuration/ |
| Multistore setup | https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/multistore-setup/ |
| Storefront Compatibility Package (install) | https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/storefront-compatibility/install/ |
| B2B Compatibility Package | https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/storefront-compatibility-b2b/ |
| AEM Assets configuration | https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/aem-assets-configuration/ |
| AEM Commerce Prerender | https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/aem-prerender/ |
| CDN configuration | https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/content-delivery-network/ |
| CORS setup | https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/cors-setup/ |
| Gated content | https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/gated-content/ |
| Price Book setup | https://experienceleague.adobe.com/developer/commerce/storefront/setup/configuration/price-book-setup/ |
