# Cart Drop-in Reference

Adobe Commerce `@dropins/storefront-cart` (v3.2.0): editable cart UI primitives covering full-cart, mini-cart, summary lines, coupons, gift cards, gift options, and shipping estimation.
Owns the pre-checkout cart state and emits the events that the Checkout drop-in subscribes to in order to bootstrap.

## Purpose

The Cart drop-in is the storefront's authoritative renderer and mutator for the pre-checkout cart state. It:

- Owns the lifecycle of the **guest cart** (`mask_id` cookie) and the **customer cart** (post-login, with merge on transition).
- Wraps the GraphQL cart mutations (`addProductsToCart`, `updateCartItems`, `applyCouponToCart`, `applyGiftCardToCart`, `setGiftOptionsOnCart`, `removeItemFromCart`, etc.) behind a stable, model-mapped API.
- Publishes `cart/*` events on the shared `@dropins/tools/event-bus` so other drop-ins (Header mini-cart count, PDP "Added" toasts, Checkout) stay in sync.
- Renders presentational containers (`CartSummaryGrid`, `CartSummaryList`, `CartSummaryTable`, `MiniCart`, `OrderSummary`, `Coupons`, `GiftCards`, `GiftOptions`, `EstimateShipping`, `EmptyCart`, `OrderSummaryLine`) that consume the `CartModel`.

**Contract with Checkout**: when checkout mounts, it listens to `cart/data` / `cart/initialized` to acquire the active cart payload. The Checkout drop-in re-emits `checkout/initialized` and `checkout/updated` (carrying `Cart | NegotiableQuote | null`), which the Cart drop-in listens to so that totals/shipping reflect checkout-stage mutations (selected shipping method, billing address) without re-fetching. Cart and Checkout share state via the event bus rather than a singleton store.

## Initialization

Cart bootstraps through the boilerplate initializer at `scripts/initializers/cart.js`. Import once from any block that renders a cart container; the initializer is idempotent.

```javascript
import '../../scripts/initializers/cart.js';
```

### Initializer signature

```javascript
await initializers.mountImmediately(initialize, {
  langDefinitions?: { [locale: string]: { [key: string]: string } },
  models?: { [modelName: string]: Model<any> },
  disableGuestCart?: boolean
});
```

| Param | Type | Required | Purpose |
|---|---|---|---|
| `langDefinitions` | `Record<string, Record<string, string>>` | No | Locale -> dictionary key tree. Deep-merged over defaults; `en_US` is the fallback. |
| `models` | `Record<string, Model<any>>` | No | Extension hook for transformers (most commonly `CartModel.transformer`). |
| `disableGuestCart` | `boolean` | No | When `true`, blocks anonymous cart creation. Use for B2B / login-walled storefronts. |

### Custom CartModel transformer

```javascript
const models = {
  CartModel: {
    transformer: (data) => ({
      customField: data?.custom_field,
      displayPrice: data?.price?.value ? `${data.price.value}` : 'N/A'
    })
  }
};
```

Transformers receive raw GraphQL responses; the returned shape is shallow-merged onto the default `CartModel`, so omitted fields fall through to defaults.

### Quick-start block pattern

```javascript
import '../../scripts/initializers/cart.js';
import CartSummaryGrid from '@dropins/storefront-cart/containers/CartSummaryGrid.js';
import { render as provider } from '@dropins/storefront-cart/render.js';

export default async function decorate(block) {
  await provider.render(CartSummaryGrid, {
    routeProduct: (item) => `/products/${item.url.urlKey}/${item.topLevelSku}`,
    routeEmptyCartCTA: () => '/'
  })(block);
}
```

### Default CartModel shape

```typescript
{
  id: string;
  totalQuantity: number;
  totalUniqueItems: number;
  items: Item[];
  miniCartMaxItems: Item[];
  total: { includingTax: Price; excludingTax: Price };
  subtotal: { excludingTax: Price; includingTax: Price; includingDiscountOnly: Price };
  appliedTaxes: TotalPriceModifier[];
  appliedDiscounts: TotalPriceModifier[];
  appliedCoupons?: Coupon[];
  addresses: { shipping?: { countryCode: string; zipCode?: string } };
  isGuestCart?: boolean;
  hasOutOfStockItems?: boolean;
  giftMessage: { recipientName: string; senderName: string; message: string };
  appliedGiftCards: AppliedGiftCardProps[];
}
```

### Initialization gotchas

- `mountImmediately` is not deferred — render before await completes will short-circuit on `null`.
- Guest cart creation auto-invoked by `initializeCart` unless `disableGuestCart: true`.
- `langDefinitions` merge mirrors existing nesting (`Cart.Cart.heading`, not `Cart.heading`).
- Transformer runs on every cart re-fetch or `events.emit('cart/data', ...)`.

## Functions

Imported from `@dropins/storefront-cart/api.js`. All mutations resolve to `CartModel | null` and most emit `cart/updated` + `cart/data` on success.

```typescript
addProductsToCart(items: {
  sku: string;
  parentSku?: string;
  quantity: number;
  optionsUIDs?: string[];
  enteredOptions?: { uid: string; value: string }[];
  customFields?: Record<string, any>;
}[]): Promise<CartModel | null>

applyCouponsToCart(couponCodes: string[], type: ApplyCouponsStrategy): Promise<CartModel | null>
applyGiftCardToCart(giftCardCode: string): Promise<CartModel | null>
createGuestCart(): Promise<any>
getCartData(): Promise<CartModel | null>
getCountries(): Promise<[CountryData]>
getEstimatedTotals(address: EstimateAddressShippingInput): Promise<CartModel | null>
getEstimateShipping(address: EstimateAddressInput): Promise<RawShippingMethodGraphQL | null>
getRegions(countryId: string): Promise<Array<{ code: string; name: string }>>
getStoreConfig(): Promise<StoreConfigModel | null>
initializeCart(): Promise<CartModel | null>
publishShoppingCartViewEvent(): any
refreshCart(): Promise<CartModel | null>
removeGiftCardFromCart(giftCardCode: string): Promise<CartModel | null>
resetCart(): Promise<CartModel | null>
setGiftOptionsOnCart(giftForm: GiftFormDataType): Promise<CartModel | null>
updateProductsFromCart(items: UpdateProductsFromCart): Promise<CartModel | null>
getCartDataFromCache(): CartModel | null
getCustomerCartPayload(): Promise<CartModel | null>
getGuestCartPayload(): Promise<CartModel | null>
```

### Function notes

- `ApplyCouponsStrategy`: `APPEND` (additive) or `REPLACE` (clears existing). Legacy Commerce accepts only one coupon — use `REPLACE` unless EE multi-coupon is enabled.
- `updateProductsFromCart` with `optionsUIDs` fires `addProductsToCart` then `updateCartItems` to swap configured variations.
- Quantity `0` via `updateProductsFromCart` = removal (uses `removeItemFromCart` internally).
- `getEstimateShipping` returns raw GraphQL (snake_case). Use `shipping/estimate` event for transformed payload.
- `getCartDataFromCache` is synchronous, returns last known model. Stale before `cart/initialized`.
- `publishShoppingCartViewEvent` is for Adobe Data Collection / commerce events SDK.
- `refreshCart()` forces fetch and re-emits events.
- `resetCart()` clears local cache and server-side cart cookie. Use on logout.

## Events

Bus: `import { events } from '@dropins/tools/event-bus.js';`

```javascript
events.on('cart/updated', (payload) => { /* CartModel | null */ }, { eager: true });
events.emit('cart/updated', newCart);
```

`{ eager: true }` replays the last emitted value synchronously on subscribe — essential for late-mounting blocks (header badge).

### Lifecycle events

| Event | Direction | Payload | Fires when |
|---|---|---|---|
| `cart/initialized` | Emit | `CartModel \| null` | First successful cart fetch after `initializeCart()`. |
| `cart/data` | Emit + Listen | `CartModel \| null` | On init and every state change. Current snapshot. |
| `cart/updated` | Emit + Listen | `CartModel \| null` | After any state-altering mutation. |
| `cart/reset` | Emit + Listen | `void` | Cart cleared (logout, explicit reset). |

### Product-level events

| Event | Direction | Payload | Fires when |
|---|---|---|---|
| `cart/product/added` | Emit | `Item[] \| null` | Genuinely new SKU added. Does NOT fire on quantity increment. |
| `cart/product/updated` | Emit | `Item[] \| null` | Quantity increase on existing line or variation swap. |
| `cart/product/removed` | Emit | `void` | Line deleted. |
| `cart/merged` | Emit + Listen | `{ oldCartItems: Item[] \| null; newCart: CartModel \| null }` | Guest cart merged into customer cart on login. |

### Cross-drop-in integration events

| Event | Direction | Payload | Source |
|---|---|---|---|
| `checkout/initialized` | Listen | `Cart \| NegotiableQuote \| null` | Checkout drop-in. |
| `checkout/updated` | Listen | `Cart \| NegotiableQuote \| null` | Checkout drop-in. |
| `requisitionList/alert` | Listen | `void` | Requisition List drop-in (B2B). |
| `shipping/estimate` | Emit + Listen | `{ address: PartialAddress; shippingMethod: ShippingMethod \| null }` | EstimateShipping form. |

### Event gotchas

- `cart/product/added` does not fire for quantity-only changes — use `cart/product/updated` for those.
- `cart/merged` is the only deterministic guest -> logged-in transition signal.
- `cart/data` and `cart/updated` fire near-simultaneously; subscribe to one to avoid duplicate renders.
- Eager listeners receive last cached payload synchronously — defensive null-check.

## Slots

Slots accept `(ctx) => void` where `ctx` exposes:
- `ctx.appendChild(element)`
- `ctx.replaceWith(element)`
- `ctx.onChange(callback)` — re-runs when data inputs change
- `ctx.item`, `ctx.cart` — slot-specific data

**Best practice**: "Do not use context methods inside other context methods (for example, `appendChild()` inside `onChange()`)." Append once, mutate via `onChange`.

### Per-container slots

**`CartSummaryGrid`**: `Thumbnail`

**`CartSummaryList`**:
- Structural: `Heading`, `EmptyCart`, `Footer`, `CartSummaryFooter`
- Per-item: `Thumbnail`, `ProductAttributes`, `CartItem`, `ItemTitle`, `ItemPrice`, `ItemQuantity`, `ItemTotal`, `ItemSku`, `ItemRemoveAction`
- Banners: `UndoBanner`, `ConfirmDeleteBanner`, `RowTotalFooter`

**`CartSummaryTable`**:
- Cells: `Item`, `Price`, `Quantity`, `Subtotal`, `Thumbnail`
- Product info: `ProductTitle`, `Sku`, `Configurations`
- States: `ItemAlert`, `ItemWarning`, `Actions`, `UndoBanner`, `EmptyCart`

**`MiniCart`** — inherits `CartSummaryList` slots **plus**: `ProductList`, `ProductListFooter`, `PreCheckoutSection`

**`OrderSummary`**: `EstimateShipping`, `Coupons`, `GiftCards`

**`GiftOptions`**: `SwatchImage`

### Slot pattern

```javascript
await provider.render(CartSummaryList, {
  slots: {
    ItemTitle: (ctx) => {
      const el = document.createElement('a');
      el.textContent = ctx.item.name;
      el.href = `/products/${ctx.item.url.urlKey}`;
      ctx.replaceWith(el);
    }
  }
})(block);
```

## Containers

All containers render via `provider.render(Container, props)(domElement)`. Default exports from `@dropins/storefront-cart/containers/{Name}.js`.

### CartSummaryGrid

Grid layout, typically the full cart page.

| Prop | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `children` | `CartModel` | Yes | — | Cart data. |
| `initialData` | `CartModel \| null` | Yes | `null` | Preloaded model. |
| `routeProduct` | `(item) => string` | No | — | Product link href. |
| `routeEmptyCartCTA` | `() => string` | No | — | Empty-cart CTA href. |

### CartSummaryList

Vertical list. Used on cart pages and inside MiniCart.

| Prop | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `children` | `CartModel` | Yes | — | Cart data. |
| `routeProduct` | `function` | No | — | Product URL. |
| `routeEmptyCartCTA` | `function` | No | — | Empty CTA URL. |
| `initialData` | `string` | No | `null` | Preloaded data. |
| `hideHeading` | `boolean` | No | `false` | Hide heading. |
| `hideFooter` | `boolean` | No | `false` | Hide footer. |
| `routeCart` | `function` | No | — | Cart page URL. |
| `onItemUpdate` | `function` | No | — | Item modification callback. |
| `onItemRemove` | `function` | No | — | Item removal callback. |
| `enableRemoveItem` | `boolean` | No | — | Show remove control. |
| `enableUpdateItemQuantity` | `boolean \| { removeOnZero: true }` | No | — | Quantity controls; auto-remove at 0. |
| `confirmBeforeDelete` | `boolean` | No | — | Inline confirmation. |
| `includeOutOfStockItems` | `boolean` | No | — | Show OOS items. |
| `quantityType` | `"stepper" \| "dropdown"` | No | `"stepper"` | Quantity widget. |
| `dropdownOptions` | `Array<{value, text}>` | When dropdown | — | Dropdown values. |
| `undo` | `boolean` | No | — | Undo banner. |

### CartSummaryTable

Table with columns Item / Price / Quantity / Subtotal / Actions.

| Prop | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `initialData` | `CartModel \| null` | No | `null` | Preload. |
| `className` | `string` | No | — | Custom class. |
| `routeProduct` | `function` | No | — | Product URL. |
| `allowQuantityUpdates` | `boolean` | No | `true` | Quantity controls. |
| `allowRemoveItems` | `boolean` | No | `true` | Remove buttons. |
| `onQuantityUpdate` | `function` | No | — | Callback. |
| `onItemRemove` | `function` | No | — | Callback. |
| `routeEmptyCartCTA` | `function` | No | — | Empty CTA URL. |
| `undo` | `boolean` | No | — | Undo banner. |

### MiniCart

Header dropdown / drawer.

| Prop | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `children` | `VNode[]` | No | — | Children. |
| `initialData` | `CartModel \| null` | No | `null` | Preload. |
| `hideFooter` | `boolean` | No | `true` | Footer hidden by default. |
| `slots` | `{ ProductList?, ... }` | No | — | Slot overrides. |
| `enableItemRemoval` | `boolean` | **Yes** | — | Allow removal. |
| `enableQuantityUpdate` | `boolean \| { removeOnZero?: boolean }` | No | — | Quantity controls. |
| `hideHeading` | `boolean` | No | — | Hide title. |
| `showDiscount` | `boolean` | No | — | Show discount lines. |
| `showSavings` | `boolean` | No | — | Show savings. |
| `undo` | `boolean` | No | — | Undo banner. |
| `confirmBeforeDelete` | `boolean` | No | — | Confirm before deletion. |
| `routeProduct(item)` | `function` | No | — | Product URL. |
| `routeCart()` | `function` | No | — | Cart page URL. |
| `routeCheckout()` | `function` | No | — | Checkout URL. |
| `routeEmptyCartCTA()` | `function` | No | — | Empty CTA URL. |

### OrderSummary

Right-rail totals, hosts coupons / gift cards / shipping estimate slots and the checkout CTA.

| Prop | Type | Required | Default | Purpose |
|---|---|---|---|---|
| `initialData` | `CartModel \| null` | No | `null` | Preload. |
| `routeCheckout` | `function` | No | — | Checkout URL. |
| `slots` | `Slot` | No | — | `EstimateShipping`, `Coupons`, `GiftCards`. |
| `errors` | `boolean` | **Yes** | — | Whether errors block checkout. |
| `showTotalSaved` | `boolean` | No | — | Show savings line. |
| `enableCoupons` | `boolean` | No | — | Show coupon section. |
| `enableGiftCards` | `boolean` | No | — | Show gift card section. |
| `updateLineItems` | `(lineItems) => OrderSummaryLineItem[]` | No | — | Transform line list. |

### Coupons

Coupon-code input. **No public props.** Must render into `OrderSummary.slots.Coupons`. Internally wires to `applyCouponsToCart`.

```javascript
provider.render(OrderSummary, {
  routeCheckout: () => '#checkout',
  slots: {
    Coupons: (ctx) => {
      const wrap = document.createElement('div');
      provider.render(Coupons)(wrap);
      ctx.appendChild(wrap);
    }
  },
  showTotalSaved: true
})('.cart__order-summary');
```

### GiftCards

Gift card code input / applied-card list. Wraps `applyGiftCardToCart` + `removeGiftCardFromCart`. Configure in Commerce Admin **Marketing > Gift Card Accounts**.

### GiftOptions

Gift wrap / message form. Two views.

| Prop | Type | Purpose |
|---|---|---|
| `dataSource` | `'cart' \| 'order'` | `cart` for cart/checkout, `order` for order detail. |
| `view` | `'product' \| 'order'` | Item-level or order-level form. |
| `isEditable` | `boolean` | `true` on cart, `false` on order confirmation. |
| `item` | `object` | Item shape (see below). |
| `onGiftOptionsChange` | `Function` | Custom integration hook. |

Item shape:
```javascript
{
  giftWrappingAvailable: boolean,
  giftMessageAvailable: boolean,
  giftWrappingPrice?: Price,
  giftMessage?: { recipientName?, senderName?, message? },
  productGiftWrapping: GiftWrappingConfigProps[]
}
```

### EstimateShipping

Country / state / postal code form.

| Prop | Type | Required | Purpose |
|---|---|---|---|
| `showDefaultEstimatedShippingCost` | `boolean` | **Yes** | Show estimated cost before form completion. |

Renders in `OrderSummary.slots.EstimateShipping`. Emits `shipping/estimate` event with `{ address, shippingMethod }`.

### EmptyCart

Zero-state.

| Prop | Type | Purpose |
|---|---|---|
| `routeCTA` | `function` | Start-shopping CTA URL builder. |

### OrderSummaryLine

Single key/value line. Programmatic alternative for use with `updateLineItems`.

| Prop | Type | Required | Purpose |
|---|---|---|---|
| `label` | `VNode \| string` | **Yes** | Left-hand label. |
| `price` | `VNode` | **Yes** | Right-hand price element. |
| `classSuffixes` | `string[]` | No | BEM suffixes. |
| `labelClassSuffix` | `string` | No | Label class suffix. |
| `testId` | `string` | No | Test hook. |

## Styling

BEM class names per container, design tokens via CSS custom properties.

### Design tokens

```css
.cart-estimate-shipping {
  gap: var(--spacing-xsmall);
  color: var(--color-neutral-800);
}
```

Token families: `--color-*` (neutral / brand / positive / negative), `--spacing-*` (xsmall, small, medium, large), typography, shape, grid.

### Container BEM roots

| Container | Selector roots |
|---|---|
| CartSummaryGrid | `.cart-cart-summary-grid`, `.cart-cart-summary-grid__content`, `.cart-cart-summary-grid__content--empty`, `.cart-cart-summary-grid__empty-cart`, `.cart-cart-summary-grid__item-container` |
| CartSummaryList | `.cart-cart-summary-list`, `.cart-cart-summary-list__heading`, `.cart-cart-summary-list__content`, `.cart-cart-summary-list__empty-cart` |
| CartSummaryTable | `.cart-cart-summary-table__row`, `.cart-cart-summary-table__cell-qty`, `.cart-cart-summary-table__item`, `.cart-cart-summary-table__item-name`, `.cart-cart-summary-table__item-configurations` |
| OrderSummary | `.cart-order-summary`, `.cart-order-summary__entry`, `.cart-order-summary__label`, `.cart-order-summary__price`, `.cart-order-summary__total` |
| MiniCart | `.cart-mini-cart`, `.cart-mini-cart__footer`, `.cart-mini-cart__products` |
| GiftOptions | `.cart-gift-options-view` |
| Coupons | `.cart-coupons__accordion-section`, `.coupon-code-form__*` |

### Boilerplate override location

Overrides go in `blocks/commerce-cart/commerce-cart.css` (cart page) and `blocks/commerce-mini-cart/commerce-mini-cart.css` (mini-cart).

## Dictionary keys

i18n tree at `en_US`. Override via `langDefinitions` (deep-merged).

### Top-level groups

| Group | Purpose |
|---|---|
| `Cart` | Cart shell — `heading`, `editCart`, `viewAll`, `viewMore`. |
| `CartSummaryTable` | Table headers — `item`, `price`, `qty`, `subtotal`. |
| `MiniCart` | `heading`, `subtotal`, `cartLink`, `checkoutLink`. |
| `EmptyCart` | Empty state. |
| `PriceSummary` | Totals, taxes, shipping, discounts, gift cards. |
| `CartItem` | Per-line strings. |
| `EstimateShipping` | Form labels. |
| `OutOfStockMessage` | OOS alerts. |
| `GiftOptions` | Wrap/message/receipt. |

### Notable defaults

| Key path | Default value |
|---|---|
| `Cart.heading` | `Shopping Cart ({count})` |
| `MiniCart.subtotal` | `Subtotal` |
| `EmptyCart.heading` | `Your cart is empty` |
| `CartItem.lowInventory` | `Only {count} left!` |
| `GiftOptions.order.giftReceipt` | `Use gift receipt` |

### Override pattern

```javascript
await initialize({
  langDefinitions: {
    en_US: {
      Cart: {
        Cart: { heading: 'My Custom Title' }
      }
    }
  }
});
```

**Gotcha**: nesting must mirror existing tree (`Cart.Cart.heading`, not `Cart.heading`).

## Tutorials condensed

### 1. Configure Cart Summary

Quantity stepper → dropdown:
```javascript
const DROPDOWN_MAX_QUANTITY = 20;
const dropdownOptions = Array.from(
  { length: parseInt(DROPDOWN_MAX_QUANTITY, 10) },
  (_, i) => ({ value: `${i + 1}`, text: `${i + 1}` })
);
provider.render(CartSummaryList, {
  quantityType: 'dropdown',
  dropdownOptions
})(block);
```

Content-driven config (EDS row keys `show-discount` / `show-savings`):
```javascript
const {
  'show-discount': showDiscount = 'false',
  'show-savings': showSavings = 'false'
} = readBlockConfig(block);

provider.render(CartSummaryList, {
  showDiscount: showDiscount === 'true',
  showSavings: showSavings === 'true'
})(block);
```

### 2. Add product lines to Cart Summary

Custom attribute via `ProductAttributes` slot:
```javascript
slots: {
  ProductAttributes: (ctx) => {
    const attrs = ctx.item?.productAttributes;
    attrs?.forEach((attr) => {
      if (attr.code === 'shipping_notes') {
        const el = document.createElement('div');
        el.innerText = `${attr.code}: ${attr.value}`;
        ctx.appendChild(el);
      }
    });
  }
}
```

Canonical pattern: append wrapper once, mutate content inside `onChange`. Never append new children inside `onChange`.

### 3. Order Summary lines

`OrderSummary.updateLineItems(lineItems)` to add / remove / reorder / group rows:

```javascript
provider.render(OrderSummary, {
  updateLineItems: (lineItems) => {
    const filtered = lineItems.filter(li => li.key !== 'tax');
    filtered.push({
      key: 'fpt',
      sortOrder: 50,
      content: OrderSummaryLine({
        label: `FPT(${labels.join(',')})`,
        price: Price({ amount: total })
      })
    });
    return filtered;
  }
})(block);
```

### 4. Gift options on PDP

**Critical constraint**: "Gift options must be applied after adding the product to the cart" — no API attaches gift options during initial `addProductsToCart`.

```javascript
CartProvider.render(GiftOptions, {
  item: cartItem ?? predefinedConfig,
  view: 'product',
  onGiftOptionsChange: async (data) => {
    sessionStorage.setItem('updatedGiftOptions', JSON.stringify(data));
  }
})($giftOptions);

const result = await addProductsToCart([{ sku, quantity }]);
const stored = JSON.parse(sessionStorage.getItem('updatedGiftOptions') || '{}');
const giftOptions = {
  gift_message: { to: stored.recipientName, from: stored.senderName, message: stored.message },
  gift_wrapping_id: stored.isSelected ? stored.wrappingId : null
};
const itemUid = result.items.find(i => i.sku === sku).uid;
await updateProductsFromCart([{ uid: itemUid, quantity, giftOptions }]);
```

### 5. Mini-cart messages

Listen to `cart/product/added` and `cart/product/updated` for toasts:
```javascript
import { events } from '@dropins/tools/event-bus.js';

events.on('cart/product/added', () => showMessage('Item added to your cart'), { eager: true });
events.on('cart/product/updated', () => showMessage('Cart updated'), { eager: true });
```

### 6. Edit product variation in cart

Adds Edit button for configurable items, toggled via block config (`enable-updating-product`). `updateProductsFromCart` with `optionsUIDs` internally does `addProductsToCart` then `updateCartItems`.

## Architectural notes

### Cart persistence

- **Guest cart**: `createGuestCart()` → Commerce returns `mask_id`. Stored in cookie (`Magento_Cart` / `commerce_cart`). Survives reloads.
- **Customer cart**: tied to customer record, persists server-side across sessions until checkout completes.
- **Cache**: `getCartDataFromCache()` returns last in-memory model.
- **Refresh**: `refreshCart()` forces fresh GraphQL fetch and re-emits all `cart/*` events. Use sparingly.

### Anonymous → logged-in transition

1. Customer cart fetched server-side post-login.
2. Commerce merges guest cart into customer cart (`mergeCarts` mutation).
3. Drop-in emits `cart/merged` with `{ oldCartItems, newCart }`.
4. Subsequent `cart/data` / `cart/updated` reflect merged state.

`cart/merged` is the only deterministic signal for the transition — use it for analytics or merged-line UI callouts.

### B2B / negotiable quotes

- `disableGuestCart: true` is the typical B2B mode (login-walled).
- `checkout/initialized` / `checkout/updated` carry `Cart | NegotiableQuote | null` — type-check.
- `requisitionList/alert` listened for bulk add-to-cart notifications.
- Cart drop-in does not render quote UI itself — quote UX lives in Checkout drop-in.

### Performance considerations

- Lazy thumbnails in `CartSummaryTable` and others by default.
- Built-in skeletons — no manual loader wrapper needed.
- Initializer idempotent — safe to import from any cart-touching block.
- `{ eager: true }` saves a render cycle for header badges.
- Avoid `refreshCart()` in hot paths — forces full re-render of every subscriber.
- Append once, mutate inside `onChange` — avoid compounding DOM growth.
- `MiniCart.hideFooter` defaults to `true` — footer with totals/CTAs is opt-in.

### Common integration pitfalls

- **Coupons multi-apply**: legacy Commerce allows only one coupon — use `REPLACE` unless EE multi-coupon enabled.
- **`getEstimateShipping` raw GraphQL**: snake_case (`carrier_code`, `method_code`). Use `shipping/estimate` event for camelCase.
- **Slot `onChange` re-runs on every `cart/data`** — diff inside callback to avoid wasteful work.
- **`enableUpdateItemQuantity: { removeOnZero: true }`** removes silently — pair with `confirmBeforeDelete: true` or `UndoBanner` slot.
- **Custom `CartModel` transformer** is shallow-merged. Spread default first:
  ```javascript
  transformer: (data) => ({ ...defaultTransform(data), customField: data?.custom_field })
  ```
- **`OrderSummary.errors` is required** — forgetting it disables the Checkout CTA permanently.
- **GiftOptions on PDP** never auto-persists — always re-apply via `updateProductsFromCart` after `addProductsToCart`.

## Source URLs

- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/quick-start/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/initialization/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/slots/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/styles/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/functions/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/events/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/dictionary/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/cart-summary-grid/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/cart-summary-list/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/cart-summary-table/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/coupons/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/empty-cart/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/estimate-shipping/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/gift-cards/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/gift-options/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/mini-cart/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/order-summary/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/containers/order-summary-line/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/tutorials/configure-cart-summary/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/tutorials/add-product-lines-to-cart-summary/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/tutorials/order-summary-lines/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/tutorials/gift-options/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/tutorials/add-messages-to-mini-cart/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/cart/tutorials/enable-product-variation-updates-in-cart/
