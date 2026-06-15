# Wishlist, Payment Services, Personalization, Recommendations — Drop-in Reference

Four B2C drop-ins that cover saved-items, Adobe-first-party payment, AEP-driven content personalization, and Sensei product recommendations.

# Wishlist Drop-in

`@dropins/storefront-wishlist` (v3.2.0). Saved-items list + add/remove toggle + cross-page item state.

## Purpose

- Multi-list wishlist (per customer).
- Per-product wishlist toggle (heart icon on PDP / PLP / Cart).
- Empty-state UX + alert banners.
- Guest wishlist (when enabled) backed by local storage; merge on login.

## Initialization

```javascript
await initializers.mountImmediately(initialize, {
  langDefinitions: { en_US: { Wishlist: { /* ... */ } } },
  isGuestWishlistEnabled: true,
  storeCode: 'default',
  models: { /* WishlistModel transformer */ },
});
```

## Functions

- `addProductsToWishlist(input): Promise<WishlistModel>`.
- `removeProductsFromWishlist(input): Promise<WishlistModel>`.
- `updateProductsInWishlist(input): Promise<WishlistModel>`.
- `getWishlists()`, `getWishlistById(id)`, `getDefaultWishlist()`, `getGuestWishlist()`.
- `initializeWishlist()`, `mergeWishlists(sourceId, destId)` (post-login).
- `resetWishlist()` (logout).
- Storage helpers for guest wishlist persistence.
- `getStoreConfig()`.

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `wishlist/initialized` | emit | `WishlistModel` |
| `wishlist/data` | emit | `WishlistModel` |
| `wishlist/alert` | emit | `{ message, type }` |
| `wishlist/reset` | emit | `void` |

## Containers

- `Wishlist` — list of saved items with line actions.
- `WishlistAlert` — banner for add/remove confirmations.
- `WishlistItem` — single row (used inside list and in standalone cards).
- `WishlistToggle` — heart icon button for any product context (PDP / PLP / cart).

## Slot

`image` slot per item for custom thumbnail rendering.

## BEM roots

`.wishlist`, `.wishlist__list`, `.wishlist-item`, `.wishlist-toggle`, `.wishlist-alert`.

## Notes

- `WishlistToggle` listens to `wishlist/data` and re-renders state.
- On login, the boilerplate's auth initializer calls `mergeWishlists` so the guest list folds in.

# Payment Services Drop-in

`@dropins/storefront-payment-services` (v3.1.1). Adobe Payment Services UI (Apple Pay, Credit Card) — slots into the Checkout drop-in's `PaymentMethods` container.

## Purpose

- Apple Pay button + sheet (PCI-compliant; no card data touches storefront).
- Hosted credit card form via Adobe Payment Services.
- Payment vault hooks (stored cards via Account drop-in).

## Requirements

- Adobe Commerce Payment Services extension v2.10.0+ on the backend.
- Apple Pay merchant ID + domain verification (`/.well-known/apple-developer-merchantid-domain-association`) — sandbox-redirect vs production-rewrite distinction matters.

## Initialization

```javascript
await initializers.mountImmediately(initialize, {
  apiUrl: 'https://payment.commerce.adobe.com',
  getCustomerToken: () => cookieGet('auth_dropin_user_token'),
  storeViewCode: 'default',
  langDefinitions: { en_US: { PaymentServices: { /* ... */ } } },
});
```

## Functions

None public — entirely event-driven and container-rendered.

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `authenticated` | listen | `boolean` |
| `payment-services/initialized/checkout` | emit | init signal for checkout context |
| `payment-services/initialized/product-detail` | emit | init signal for PDP context |

## Containers

### ApplePay

| Prop | Type | Purpose |
|------|------|---------|
| `location` | `'checkout' \| 'pdp' \| 'cart'` | Determines flow + button styling |
| `getCartId` | function | Returns cart ID for express checkout from PDP |
| `createCart` | function | Creates a cart when needed for express checkout |
| `onButtonClick` | callback | Pre-click hook |
| `onSuccess` | callback | Post-success hook |
| `onError` | callback | Error hook |

### CreditCard

Embedded card form (Adobe-hosted iframe — PCI compliance).
| Prop / Method | Purpose |
|---------------|---------|
| `creditCardFormRef.current.validate()` | Trigger validation |
| `creditCardFormRef.current.submit()` | Tokenize + submit |

## Slots

No slots — PCI compliance requires Adobe-hosted rendering.

## `PaymentMethodCode` enum

Identifies methods within the Checkout's `PaymentMethods.Methods` slot map. Used when slotting Payment Services-specific renders.

## BEM roots

`.payment-services-apple-pay`, `.payment-services-credit-card`.

# Personalization Drop-in

`@dropins/storefront-personalization` (v3.1.1). AEP / Adobe Commerce-driven content personalization — targeted blocks visible to specific audiences.

## Purpose

- Display content variants conditional on customer group, customer segment, or cart price rule context.
- Refresh personalization state on cart / order changes.

## Backing primitives (Adobe Commerce)

- Customer groups.
- Customer segments.
- Cart price rules (segment-conditioned promotions).

## Initialization

```javascript
await initializers.mountImmediately(initialize, {
  langDefinitions: { en_US: { Personalization: { /* ... */ } } },
});
```

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `cart/initialized` | listen | trigger re-evaluation |
| `cart/updated` | listen | trigger re-evaluation |
| `order/placed` | listen | refresh post-purchase context |
| `personalization/updated` | emit | `{ audiences, segments }` |

## Functions

- `fetchPersonalizationData()`.
- `getPersonalizationData()`.
- `savePersonalizationData()`.
- `getStoreConfig()`.

## Slot

`Content` slot per targeted block — defines what to render when the audience matches.

## Containers

### TargetedBlock

| Prop | Type | Purpose |
|------|------|---------|
| `type` | string | Audience type (customer group / segment / rule UID) with fallback chain |
| `slots.Content` | function | Render callback for matched audience |

## StoreConfig flags

`StoreConfigModel` exposes flags that toggle personalization features per store view.

# Recommendations Drop-in

`@dropins/storefront-recommendations` (v4.0.1). Sensei-driven product recommendation rails.

## Purpose

- Render recommendation units (configured in Commerce Admin) — "Recommended for you", "Customers also bought", "Trending", etc.
- Emit click-through events for Sensei learning loop.
- Multiple units per page (PDP, cart, home).

## Backing service

Adobe Commerce Product Recommendations (Sensei AI). Configured via Commerce Admin: define units with `unitId`, page type, position.

## Initialization

```javascript
await initializers.mountImmediately(initialize, {
  langDefinitions: { en_US: { Recommendations: { /* ... */ } } },
  models: {
    RecommendationUnitModel: {
      transformer: (data) => ({ /* ... */ }),
    },
  },
});
```

## Functions

- `getRecommendationsByUnitIds(unitIds: string[]): Promise<RecommendationUnit[]>`.
- `publishRecsItemAddToCartClick(item)` — sends click signal to Sensei.

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `recommendations/data` | emit | `RecommendationUnit[]` |

## Slots (6)

`Heading`, `Footer`, `Title`, `Sku`, `Price`, `Thumbnail` — per-product-card customization.

## Containers

### ProductList

| Prop | Type | Purpose |
|------|------|---------|
| `recId` | string | Recommendation unit ID |
| `currentProduct` | ProductView | Context for recs (sent as signal to Sensei) |
| `routeProduct` | function | URL builder per item |

GraphQL query routing depends on whether product price is present in the response — handles configurable products with hidden prices gracefully.

## Architectural notes

### Backing service per drop-in

| Drop-in | Service | Falls back when off |
|---------|---------|---------------------|
| Wishlist | Core GraphQL `wishlist` | Cannot operate without backend support |
| Payment Services | Adobe Payment Services extension | Use a different PSP via Checkout's `PaymentMethods` slot |
| Personalization | Adobe Commerce customer groups / segments + AEP | Renders default (un-targeted) content |
| Recommendations | Adobe Commerce Product Recommendations (Sensei) | Renders nothing or falls back to a "Popular products" static unit |

### Performance considerations

- Lazy-load Payment Services SDK (Apple Pay / iframe) only when checkout reaches payment step.
- Batch recommendations — call `getRecommendationsByUnitIds(['a', 'b', 'c'])` instead of three separate calls.
- Skeleton-render TargetedBlocks to prevent CLS during audience evaluation.
- Recommendations are cached at edge (Adobe-hosted); page-level cache hit is fast.

### Cross-cutting integration

- Wishlist toggle on PDP listens to PDP's `pdp/data` event for current SKU.
- Recommendations on PDP read `currentProduct` from PDP's `pdp/data`.
- Personalization re-evaluates on `cart/updated` — segment membership can change with cart contents (price rules).
- Apple Pay express checkout from PDP needs `getCartId` / `createCart` callbacks because no cart exists yet.

### Event ordering

Cart and Recommendations both consume PDP data. Personalization reacts to Cart events. Order is:
1. PDP emits `pdp/data`.
2. Recommendations subscribe (eager) → fetch via `getRecommendationsByUnitIds`.
3. User adds to cart → Cart emits `cart/updated`.
4. Personalization subscribes → re-evaluates audiences.

### Storage scoping

Guest Wishlist uses `localStorage` keyed by `storeCode` to keep multistore separation.

### Auth coordination

All four drop-ins listen for `authenticated`:
- Wishlist: merges guest list on sign-in.
- Payment Services: switches to vaulted cards.
- Personalization: re-fetches with customer context.
- Recommendations: includes customer signals in unit fetch.

### Dictionary deep-merge

Override pattern same as other drop-ins. Each drop-in has its own root namespace (`Wishlist`, `PaymentServices`, `Personalization`, `Recommendations`).

## Source URLs (all four drop-ins)

Wishlist: https://experienceleague.adobe.com/developer/commerce/storefront/dropins/wishlist/ (+ sub-pages)
Payment Services: https://experienceleague.adobe.com/developer/commerce/storefront/dropins/payment-services/ (+ sub-pages)
Personalization: https://experienceleague.adobe.com/developer/commerce/storefront/dropins/personalization/ (+ sub-pages)
Recommendations: https://experienceleague.adobe.com/developer/commerce/storefront/dropins/recommendations/ (+ sub-pages)
