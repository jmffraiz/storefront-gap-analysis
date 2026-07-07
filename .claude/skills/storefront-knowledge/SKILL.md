---
name: storefront-knowledge
description: Adobe Commerce Storefront (ACCS + Edge Delivery Services) reference knowledge base, built from Adobe's official LLM documentation bundles. Covers drop-ins (`@dropins/storefront-*`), containers, slots, events, the AEM Boilerplate Commerce repo, Commerce blocks, the Storefront SDK / design tokens, Catalog Service / Live Search / Product Recommendations / Payment Services, GraphQL endpoints (`commerce-endpoint` vs `commerce-core-endpoint`), the Storefront Compatibility Package, AEM Commerce Prerender, AEM Assets, multistore / multilocale, CDN / CORS, B2B (companies, quotes, purchase orders, requisition lists, quick order), Universal Editor wiring, authoring, go-live, and performance. Use whenever the request involves Adobe Commerce storefronts on EDS, headless commerce on AEM, drop-in / container / slot / event identifiers, `@dropins/tools`, `fetchGraphQl`, the AEM Boilerplate Commerce repo, or any ACCS / PaaS / ACO storefront capability question.
---

# Storefront Knowledge Base

Reference documentation for **Adobe Commerce Storefront** on **Edge Delivery Services (EDS)** — the Commerce layer: drop-ins, services, GraphQL, blocks, SDK, configuration.
The references are Adobe's official per-topic LLM bundles, stored verbatim. Only load the specific document(s) that match the current request.

## How to use

1. Match the request against the **Document Index** below.
2. The bundles are large (30–225 KB). **Grep first** to locate the relevant section, then read only that line range:
   ```bash
   # Locate sections (each doc in a bundle starts with "# ")
   grep -nE "^# " references/dropins-cart.txt | grep -iE "coupon|gift"
   # Locate an exact identifier anywhere
   grep -n "CartSummaryList" references/dropins-cart.txt
   ```
3. Never read a whole bundle — read 80–300 lines around the grep hit.
4. **Never invent drop-in / container / slot / event / config-key names** — if you can't find an identifier in the matching bundle, say so.
5. For the generic EDS delivery layer underneath (blocks in general, sections, authoring mechanics, metadata, indexing, sitemaps, Lighthouse), use the sibling **`eds-knowledge`** skill instead. This skill covers only what is Commerce-specific.

## Document Index

| File | Topic | When to load |
|------|-------|-------------|
| [get-started.txt](references/get-started.txt) | Onboarding, architecture, backend options (ACCS / PaaS / ACO), performance, browser support | "How does it fit together", backend topology choice, first orientation |
| [boilerplate.txt](references/boilerplate.txt) | AEM Boilerplate Commerce: setup, configuration, blocks reference, customization, Universal Editor, updates | Repo layout, initializers, boilerplate blocks, Universal Editor wiring |
| [dropins-intro.txt](references/dropins-intro.txt) | Cross-drop-in concepts: extend/create, slots, events, dictionaries, styling, indexes for B2C and B2B drop-ins | "How do drop-ins work in general", slot API, event bus, any cross-cutting question |
| [dropins-pdp.txt](references/dropins-pdp.txt) | Product details drop-in: containers, slots, APIs | PDP, gallery, configurable options, price, attributes |
| [dropins-catalog.txt](references/dropins-catalog.txt) | Product discovery (Live Search), recommendations, personalization drop-ins | PLP, faceted browse, search, federated search, recs |
| [dropins-cart.txt](references/dropins-cart.txt) | Cart drop-in: containers, slots, events, customization APIs | Cart page, mini-cart, coupons, gift cards, estimate shipping |
| [dropins-checkout.txt](references/dropins-checkout.txt) | Checkout drop-in: containers, slots, events, customization APIs | Single/multi-step checkout, PSP integration, BOPIS |
| [dropins-order.txt](references/dropins-order.txt) | Order management, order confirmation, returns drop-in | Orders list, order details, returns, cancellation |
| [dropins-account-auth.txt](references/dropins-account-auth.txt) | User account and user authentication drop-ins | Sign-in, registration, addresses, payment methods, reCAPTCHA |
| [dropins-wishlist-payments.txt](references/dropins-wishlist-payments.txt) | Wishlist and Payment Services drop-ins | Wishlist UX, Adobe Payment Services, Apple Pay |
| [dropins-b2b-company.txt](references/dropins-b2b-company.txt) | B2B company management and company switcher drop-ins | Companies, roles, users, company switcher |
| [dropins-b2b-quote.txt](references/dropins-b2b-quote.txt) | B2B quote management drop-in: containers, slots, events, APIs | Negotiable quotes, quote lifecycle |
| [dropins-b2b-purchasing.txt](references/dropins-b2b-purchasing.txt) | B2B purchase order, quick order, requisition list drop-ins | POs and approvals, quick order pad, requisition lists |
| [blocks-reference.txt](references/blocks-reference.txt) | EDS block configurations for B2C and B2B commerce blocks | Which commerce block mounts which container; block table options |
| [sdk-reference.txt](references/sdk-reference.txt) | Storefront SDK: UI components, design system, Event Bus, GraphQL fetcher, slots API, CLI, utilities | Building custom drop-ins, design-token theming, low-level APIs |
| [setup-configuration.txt](references/setup-configuration.txt) | Configuration: endpoints, headers, CORS, gated content, multistore, prerender, CDN, AEM Assets, Storefront Compatibility Package | Any `config.json` / endpoint / header / multistore / infra question |
| [setup-go-live.txt](references/setup-go-live.txt) | Discovery, Luma Bridge, analytics, AEP, SEO, launch checklist | Go-live, migration, instrumentation, SEO |
| [merchants-authoring.txt](references/merchants-authoring.txt) | Merchant quick start, content, localization, storefront builder, content customizations | How merchants author commerce content, localization |
| [how-tos.txt](references/how-tos.txt) | Task-focused how-to guides | "How do I…" implementation tasks |
| [tutorials-reference.txt](references/tutorials-reference.txt) | Step-by-step tutorials (cart, checkout, order, account, PDP) | Worked customization examples, sizing a customization |

## Mental model

1. **EDS is the delivery layer; drop-ins are the commerce layer.** EDS serves authored documents as HTML; it does not own commerce state.
2. **Two GraphQL endpoints:** `commerce-endpoint` → Adobe-hosted services (Catalog Service, Live Search, Recommendations; read-heavy); `commerce-core-endpoint` → Commerce core GraphQL (cart, checkout, customer, orders — state).
3. **Containers compose; drop-ins coordinate.** A "drop-in" is the npm package (`@dropins/storefront-cart`); a "container" is a renderable composite inside it (`CartSummaryList`); the host **block** decides what to mount. A "slot" is a named extension point inside a container.
4. **Slots before overrides before forks.** Forks are upgrade tax.
5. **Authoring is document-driven.** Block tables drive container props via `readBlockConfig()`; merchant-controllable behavior belongs in the block table.
6. **The Storefront Compatibility Package** bridges a PaaS instance to the drop-ins; on ACCS it (and the B2B variant) is auto-installed. B2B drop-ins additionally assume `commerce-b2b-enabled`.
7. **The monolithic `ProductDetails` container is deprecated** — use the modular per-area containers.

## Phrasing discipline

- Use Adobe's official names verbatim: **Adobe Commerce as a Cloud Service (ACCS)**, **Adobe Commerce Optimizer (ACO)**, **Adobe Commerce (PaaS)**, **Storefront Compatibility Package**, **AEM Boilerplate Commerce**, **Edge Delivery Services (EDS)**, **Universal Editor**, **Document Authoring (DA.live)**, **Catalog Service**, **Live Search**, **Product Recommendations**, **Payment Services**, **Data Connection**, **API Mesh**.
- Reserve **"block"** for EDS blocks (`decorate(block)`) and **"container"** for drop-in containers. Don't mix them.

## Refreshing the references

The bundles are generated by Adobe from the official docs. To refresh, re-download from the index at `https://experienceleague.adobe.com/developer/commerce/storefront/llms.txt`:

```bash
cd references
for f in get-started boilerplate how-tos setup-configuration setup-go-live \
  dropins-intro dropins-cart dropins-checkout dropins-order dropins-pdp \
  dropins-account-auth dropins-catalog dropins-wishlist-payments \
  dropins-b2b-quote dropins-b2b-company dropins-b2b-purchasing \
  blocks-reference merchants-authoring sdk-reference tutorials-reference; do
  curl -sSf -o "$f.txt" "https://experienceleague.adobe.com/developer/commerce/storefront/_llms-txt/$f.txt"
done
```
