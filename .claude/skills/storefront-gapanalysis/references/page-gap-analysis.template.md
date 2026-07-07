<!--
Authoring reference — for whoever fills in this template. Do NOT copy this comment block into the generated document; it is defined once, here.

Architecture: the storefront sits on Adobe Commerce as a Cloud Service (ACCS) + Edge Delivery Services (EDS) using the AEM Boilerplate Commerce repo (https://github.com/hlxsites/aem-boilerplate-commerce). Each page is hosted by a thin EDS block that mounts one or more `@dropins/storefront-*` drop-ins.

Documentation links: every drop-in package name, container name, and named Adobe service mentioned in the document must be hyperlinked to its official Experience League page, using the URL reference table at the end of this file. Format: `[name](url)`.

Complexity buckets (used in §1 Complexity column):
- Low — theming / config / authored content / single slot fill.
- Medium — new block logic, multi-slot wiring, drawer/state coordination, or initializing one additional drop-in.
- High — multi-block coordination plus a new service dependency or external-system integration.
-->

# <Page name> — Gap Analysis

> **Author:** <name> · **Date:** <YYYY-MM-DD>

![<alt text: what the mockup shows — page/frames/breakpoints, and what scope is highlighted>](<relative path to mockup image>)

Description of the highlighted scope as a **bullet list, never a paragraph** — one bullet per functional input/constraint the user gave for this feature, polished but not merged or summarized together, no process/methodology notes:

- <polished restatement of one functional input/constraint the user gave for this feature>
- <...>

---

## 1. Feature gap analysis

One row per discrete feature observed in the design. Rows ordered by page position; group rows by variant/state where it helps readability.

Legend — **Coverage:** ✅ provided · 🟡 partial (slot / theme / extend) · ❌ none.

Column guidance:
- **Feature** — short noun phrase + one clause of what the shopper sees/does; note variant/state in *italics* when it is not Default (e.g. *OOS only*). Type tag: `[C]` Commerce · `[A]` Authored content · `[X]` Cross-cutting.
- **OOTB basis** — the drop-in / container / slot that provides the closest starting point, with concrete names (containers, slots, events, props), each linked to its doc page. `none` if greenfield.
- **Gap to close** — the deliverable: what to build / wrap / restyle / configure, stated so it can be estimated directly. This is the primary column.
- **Dependencies & touch points** — backend prereqs, third-party SDKs, design tokens, plus the blocks/slots/files expected to change. Omit when trivial.

| # | Feature | OOTB basis | Coverage | Gap to close | Dependencies & touch points | Complexity | Risk |
|---|---------|------------|:--------:|--------------|-----------------------------|:----------:|:----:|
| F1 | <noun phrase — what the shopper sees/does> `[C/A/X]` *<variant if not Default>* | [`@dropins/storefront-…`](<doc-url>) → <container / slot / event names> or `none` | <✅ / 🟡 / ❌> | <what to build / wrap / restyle — estimable in one read> | <prereqs · files/blocks that change> | <Low/Medium/High> | <Low/Med/High> |
| F2 | | | | | | | |

---

## 2. Overall assumptions and open questions

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

## 3. Explicitly out of scope

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
