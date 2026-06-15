# Mini Basket — Gap Analysis

> **Mockup source:** [mini-basket.png](figmas/mini-basket.png) (Mobile / Tablet / Desktop — "Mini Basket - Vuse"); [Headers-Footers.png](figmas/Headers-Footers.png) for basket-icon trigger context.
> **Variants/states included:** Default (1 item, populated). Empty / OOS / multi-item / max-items not shown in mockup.
> **Scope:** **IN:** the mini-basket overlay that triggers on add-to-cart and on header basket-icon click — its shell, line items, totals, free-shipping progress, CTAs, and the "Add some flavour" recommendations strip. **OUT:** full cart page, checkout, PDP add-to-cart wiring, header/footer chrome (covered in the header gap analysis).
> **Author:** Jose Maria Franco · **Date:** 2026-06-15

The storefront sits on **Adobe Commerce as a Cloud Service (ACCS) + Edge Delivery Services (EDS)** using the **AEM Boilerplate Commerce** repo. The mini-basket is hosted by a thin EDS block (`blocks/commerce-mini-cart`) that mounts the **[`@dropins/storefront-cart`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/) [`MiniCart` container](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/mini-cart/)**, opened from the `commerce-header` block. The recommendations strip mounts a second drop-in ([`@dropins/storefront-recommendations`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/)) inside a mini-cart [slot](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/slots/).

### Mockups

**Mini Basket overlay** (Mobile / Tablet / Desktop) — primary source for this analysis:

![Mini Basket — Mobile, Tablet, Desktop](figmas/mini-basket.png)

**Header + Footer** — context for the basket-icon trigger (F1); full analysis in the header gap doc:

![Header and Footer states](figmas/Headers-Footers.png)

**Complexity buckets** (used in §2):
- **Low** — theming / config / authored content / single slot fill.
- **Medium** — new block logic, multi-slot wiring, drawer/state coordination, or initializing one additional drop-in.
- **High** — multi-block coordination plus a new service dependency or external-system integration.

---

## 1. Feature decomposition

| # | Feature | Description in the design | Variant/state | Type | Observations |
|---|---------|---------------------------|---------------|------|--------------|
| F1 | Overlay shell + trigger | Right-anchored overlay drawer with a back-arrow (←) and "Added to your basket" heading; opens automatically when a product is added and when the header basket icon is clicked. | Default | Cross-cutting | Drawer chrome (backdrop, slide-in, focus trap, body-scroll-lock, close/back affordance) is not part of any drop-in container; the host block owns it. Auto-open is driven by the `cart/product/added` event. |
| F2 | Basket total + item count | "BASKET TOTAL: £5.49 (1 item)" line below the heading. | Default | Commerce | Mockup mixes £ (total) and € (line price) — currency source must be reconciled (see open questions). |
| F3 | Free-shipping message (static) | Orange banner with truck icon: "Spend £50 to get free shipping". | Default | Authored content | Static authored content — **not** tied to cart subtotal or any threshold logic. A fixed promo banner rendered in a slot; copy/icon are merchant-authored. |
| F4 | Cart line item | Thumbnail, "Product name", attribute "12 MG/ML", "QUANTITY: 1", line price "€00.00", remove (X). | Default | Commerce | Quantity is shown as a static label ("QUANTITY: 1"), not a stepper. Single visible remove control. "12 MG/ML" is a product attribute (nicotine strength). |
| F5 | Remove item | (X) icon top-right of the line item. | Default | Commerce | Native to `MiniCart` via `enableItemRemoval`. No undo/confirm shown in mockup. |
| F6 | Checkout CTA | Full-width orange "CHECKOUT SECURELY" primary button. | Default | Commerce | Native footer CTA; routes to checkout. |
| F7 | Continue-shopping CTA | Outlined "CONTINUE SHOPPING" button below checkout. | Default | Commerce | Closes the overlay; not a native MiniCart button — secondary action wired by host block / slot. |
| F8 | "Add some flavour" recommendations strip | Section titled "ADD SOME FLAVOUR" with "VIEW ALL →" link and a horizontal strip of product cards. | Default | Commerce | Lives below the cart footer, inside the overlay. Backed by a recommendations source, not by cart data. |
| F9 | Recommendation product card | Image, "PROMOTION" badge, wishlist heart, "PROMO HERE" tag, product name, 5-star rating with "(5)" count, price "£19.50 →". | Default | Commerce | Composite: recs + promo badging + wishlist + ratings. Ratings/review count and promo badge are not provided by Catalog Service / Recommendations out of the box. |
| F10 | Wishlist toggle on rec card | Heart icon top-right of each rec card. | Default | Commerce | Requires the Wishlist drop-in; toggles add/remove on the recommended SKU. |
| F11 | Empty / OOS / max-items states | Not shown in mockup. | (missing) | Commerce | `MiniCart` + `EmptyCart` cover these but designs are absent — flagged as open. |

---

## 2. Gap analysis

Legend — **Coverage:** ✅ provided · 🟡 partial (slot / theme / extend) · ❌ none.

| # | Feature | Existing drop-in / block | Coverage | What it gives OOTB | Gap to close | Touch points | Dependencies | Complexity | Risk |
|---|---------|--------------------------|:--------:|--------------------|--------------|--------------|--------------|:----------:|:----:|
| F1 | Overlay shell + trigger | `commerce-header` block + [`MiniCart`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/mini-cart/) container | 🟡 | `MiniCart` renders the cart body as a header dropdown/drawer content; it does not own backdrop, slide-in animation, focus trap, or scroll-lock. | Build the overlay shell (panel, backdrop, back-arrow/close, a11y) in the host block; subscribe to [`cart/product/added`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/events/) (+ basket-icon click) to auto-open. Reuse the boilerplate header's existing mini-cart toggle as the base. | `blocks/commerce-header/*`, `blocks/commerce-mini-cart/*`, `cart/product/added` listener | event-bus `{ eager: true }` for badge sync | Medium | Med |
| F2 | Basket total + item count | [`MiniCart`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/mini-cart/) | 🟡 | `MiniCart` exposes subtotal ([`MiniCart.subtotal` dictionary key](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/dictionary/)) and `totalQuantity`/`totalUniqueItems` on `CartModel`; footer is opt-in (`hideFooter: true` by default). | Render "BASKET TOTAL: {price} ({count} item)" near the heading — a [slot](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/slots/) fill (e.g. `Heading`/`PreCheckoutSection`) or restyle the footer subtotal; map `totalQuantity` to the "(n item)" label with pluralization. | `commerce-mini-cart.js` slots, `langDefinitions` | — | Low | Low |
| F3 | Free-shipping message (static) | [`MiniCart` slot](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/slots/) + authored content | 🟡 | No native banner, but slots (`Heading` / `PreCheckoutSection`) accept arbitrary authored markup (see [Add messages to mini cart](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/tutorials/add-messages-to-mini-cart/)). | Drop a static authored banner into a slot; no cart/threshold logic. Theme the orange banner + truck icon. | `commerce-mini-cart.js` slot, block table / authored content, CSS | `--color-*` tokens for the orange banner | Low | Low |
| F4 | Cart line item | `MiniCart` (inherits [`CartSummaryList`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/cart-summary-list/) slots) | ✅ | Renders thumbnail, title, price, quantity, and per-item slots (`Thumbnail`, `ItemTitle`, `ItemPrice`, `ItemQuantity`, `ProductAttributes`, `ItemRemoveAction`). | Map "12 MG/ML" via `ProductAttributes` slot; render quantity as a static "QUANTITY: n" label (set `enableQuantityUpdate: false`); restyle to match. | `commerce-mini-cart.js` slots + CSS | nicotine-strength attribute present on Catalog Service product | Low | Low |
| F5 | Remove item | [`MiniCart`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/mini-cart/) | ✅ | `enableItemRemoval` (required prop) renders the remove control; optional `confirmBeforeDelete` / `undo`. | Set `enableItemRemoval: true`; restyle the (X) to top-right. Decide whether to enable undo/confirm (not in mockup). | `commerce-mini-cart.js` prop + CSS | — | Low | Low |
| F6 | Checkout CTA | [`MiniCart`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/mini-cart/) | ✅ | `routeCheckout()` + footer CTA; native primary button. | Wire `routeCheckout`, enable footer (`hideFooter: false`), theme to the orange primary. | `commerce-mini-cart.js` props + CSS | checkout route | Low | Low |
| F7 | Continue-shopping CTA | [`MiniCart`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/mini-cart/) | 🟡 | No native "continue shopping" button; footer ships checkout + cart links. | Add a secondary button via footer [slot](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/slots/) (`ProductListFooter`/`PreCheckoutSection`) that closes the overlay; theme as outlined button. | `commerce-mini-cart.js` slot + CSS | overlay-close handler from F1 | Low | Low |
| F8 | "Add some flavour" recs strip | [`@dropins/storefront-recommendations`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/) (or [`@dropins/storefront-product-discovery`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/) for "view all") | 🟡 | [Product Recommendations drop-in](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/containers/) renders Sensei-backed rec units (carousel/strip) with product cards; "VIEW ALL" links to a PLP/search. | Mount the recommendations unit inside a mini-cart slot (`ProductListFooter`/`PreCheckoutSection`); choose rec type (e.g. "added to cart" / cart-context unit); wire "VIEW ALL →" route. | new partial in `commerce-mini-cart.js`, recs block | **Product Recommendations** service enabled + Data Connection/AEP feeding behavioral data; rec unit configured in Commerce Admin | High | Med |
| F9 | Rec product card (badge + rating + price) | [Recommendations](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/slots/) card + custom badging/ratings | 🟡 | Rec card gives image, name, price, link. | Add "PROMOTION"/"PROMO HERE" badging (catalog attribute or authored) and 5-star rating + count — **ratings are not native to Catalog Service/Recommendations**; need a reviews provider. Restyle card. | rec card slot/override, CSS | promo flag source; **third-party reviews/ratings integration** (e.g. Bazaarvoice/Yotpo) for stars + count | High | High |
| F10 | Wishlist toggle on rec card | [`@dropins/storefront-wishlist`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/) | 🟡 | [Wishlist drop-in](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/) provides add/remove + state for a SKU (see [`WishlistToggle` container](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/containers/wishlist-toggle/)). | Mount a wishlist heart control per rec card, bound to the rec SKU; reflect logged-in vs guest behavior. | rec card slot, wishlist init | **Wishlist** drop-in initialized; customer auth state | Medium | Med |
| F11 | Empty / OOS / max-items states | `MiniCart` + [`EmptyCart`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/empty-cart/) | 🟡 | `EmptyCart` zero-state, `includeOutOfStockItems`/`hasOutOfStockItems`, `miniCartMaxItems` truncation with "view more". | Confirm designs for these states; theme `EmptyCart`, OOS line alert, and the max-items "view all" affordance. | `commerce-mini-cart.js`, `langDefinitions` | design for missing states | Low | Med |

---

## 3. Overall assumptions and open questions

### 1. Assumptions
   - The mini-basket is the boilerplate `commerce-mini-cart` block mounting the `MiniCart` container, opened from `commerce-header` — not a from-scratch component.
   - Quantity is intentionally read-only in the overlay ("QUANTITY: 1" label); quantity editing happens on the full cart page.
   - The free-shipping banner is static authored content (fixed copy, no subtotal/threshold logic); the "PROMOTION"/"PROMO HERE" badges are merchant-driven values, not hard-coded.
   - The "ADD SOME FLAVOUR" strip is a single recommendations unit, not a hand-curated cross-sell list.
   - One coupon model and no coupon/gift-card entry inside the mini-basket (none shown) — those live on the cart page.

### 2. Open questions

1. **Which currency is authoritative — £ or €?**
   - *Recommendation:* single store currency from Commerce config; treat the € line price as a mockup artifact.
   - *Impact A — single currency:* no work; totals and line prices both come from `CartModel` in store currency.
   - *Impact B — multi-currency / price-book switching:* adds currency-context handling and possibly per-locale store views — pushes F2/F4 up a complexity band.

2. **What backs the 5-star ratings and review count on rec cards?**
   - *Recommendation:* confirm whether a reviews provider is in scope; if not, drop ratings from MVP.
   - *Impact A — third-party reviews (Bazaarvoice/Yotpo/etc.):* XL integration, async rating fetch per SKU, caching — dominates F9 cost.
   - *Impact B — no ratings at launch:* F9 drops to a restyle of the native rec card (S).

3. **Is the recommendations strip powered by Adobe Product Recommendations (Sensei) or a static cross-sell?**
   - *Recommendation:* Adobe Product Recommendations with a cart-context unit.
   - *Impact A — Product Recommendations:* requires the service + Data Connection/AEP; richer but adds service enablement and a populated behavioral dataset.
   - *Impact B — static/related-products cross-sell:* simpler, no AEP dependency, but loses personalization and the "added to cart" relevance.

4. **Should removal use confirm/undo, and what are the empty/OOS/max-items designs?**
   - *Recommendation:* enable `undo` (non-destructive) and request the missing-state designs before estimating F11.
   - *Impact A — undo + designed states:* small added scope, predictable.
   - *Impact B — left undefined:* F5/F11 estimates carry design risk and likely rework.

## 4. Explicitly out of scope

- Full cart page (`commerce-cart` block) — coupons, gift cards, gift options, shipping estimation, quantity editing.
- Checkout flow and place-order.
- PDP add-to-cart button wiring (only the resulting `cart/product/added` auto-open is in scope).
- Header and footer chrome beyond the basket-icon trigger (see header gap analysis).
- B2B variations (negotiable quotes, requisition lists, `disableGuestCart`).
- Reviews/ratings platform integration itself (treated as a dependency, not built here).
- Multi-currency / price-book infrastructure unless open question #1 resolves to multi-currency.

---

## 5. Drop-in documentation references

**Cart drop-in** ([`@dropins/storefront-cart`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/)) — primary drop-in backing F1–F7 and F11:
- [Quick start](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/quick-start/) · [Initialization](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/initialization/) · [Functions](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/functions/) · [Events](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/events/) · [Dictionary](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/dictionary/) · [Slots](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/slots/) · [Styles](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/styles/)
- Containers: [`MiniCart`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/mini-cart/) · [`CartSummaryList`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/cart-summary-list/) · [`EmptyCart`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/empty-cart/) · [`OrderSummary`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/order-summary/)
- Tutorials: [Add messages to mini cart](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/tutorials/add-messages-to-mini-cart/) · [Add product lines to cart summary](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/tutorials/add-product-lines-to-cart-summary/) · [Configure cart summary](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/tutorials/configure-cart-summary/)

**Product Recommendations drop-in** ([`@dropins/storefront-recommendations`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/)) — backs F8 / F9:
- [Quick start](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/quick-start/) · [Initialization](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/initialization/) · [Containers](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/containers/) · [Slots](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/slots/) · [Functions](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/functions/) · [Events](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/events/) · [Dictionary](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/dictionary/) · [Styles](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/styles/)

**Wishlist drop-in** ([`@dropins/storefront-wishlist`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/)) — backs F10:
- [Quick start](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/quick-start/) · [Initialization](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/initialization/) · [Containers](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/containers/) · [Slots](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/slots/) · [Functions](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/functions/) · [Events](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/events/) · [Dictionary](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/dictionary/)

**Product Discovery drop-in** ([`@dropins/storefront-product-discovery`](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/)) — referenced from F8 for the "VIEW ALL →" target (PLP / federated search):
- [Quick start](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/quick-start/) · [Containers](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-discovery/containers/) · [Federated search](https://experienceleague.adobe.com/developer/commerce/storefront/how-tos/federated-search/) · [Search redirects](https://experienceleague.adobe.com/developer/commerce/storefront/how-tos/search-redirects/)

**Cross-cutting**:
- [Drop-ins overview](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/) · [Drop-in SDK](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/sdk/) · [Event bus](https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/events/) · [AEM Boilerplate Commerce repo](https://github.com/hlxsites/aem-boilerplate-commerce)
