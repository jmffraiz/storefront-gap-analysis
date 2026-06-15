# Checkout Drop-In Reference

Adobe Commerce `@dropins/storefront-checkout` (v3.2.0) provides composable containers, an event-driven state model, and headless APIs for guest, customer, and B2B checkout — single-step or multi-step.
The drop-in does not own a layout: integrators compose containers (`LoginForm`, `BillToShippingAddress`, `ShippingMethods`, `PaymentMethods`, `PlaceOrder`, etc.) inside a block and coordinate them via the shared event bus.

## Purpose

The checkout drop-in is responsible for:

- Capturing shopper identity (`LoginForm` — guest email or sign-in trigger).
- Capturing and persisting shipping/billing addresses, including the "billing same as shipping" toggle (`BillToShippingAddress`).
- Surfacing available shipping methods, persisting selection, emitting shipping estimates (`ShippingMethods`, `EstimateShipping`).
- Listing available payment methods, supporting custom PSP renderers (`PaymentMethods`, `PaymentOnAccount`, `PurchaseOrder`).
- Enforcing legal acceptance (`TermsAndConditions`) before order placement.
- Validating cart state at the boundary: out-of-stock detection, merged-cart notification, server errors (`OutOfStock`, `MergedCartBanner`, `ServerError`).
- Address verification handoff to a third-party AVS (`AddressValidation`).
- Final order submission with payment-method-aware button state and validation orchestration (`PlaceOrder`).
- B2B negotiable quote checkout (via `features.b2b.quotes` toggle and `quote-management/quote-data` event).

It does NOT own:
- Cart line items rendering (provided by `@dropins/storefront-cart`).
- Order summary / totals (provided by `@dropins/storefront-cart`'s `OrderSummary`, into which `EstimateShipping` is slotted).
- Order confirmation page (provided by `@dropins/storefront-order`).
- Authentication UI/session (provided by `@dropins/storefront-auth` — checkout only listens to the `authenticated` event).

### Single vs multi-step

The same containers serve both modes — the difference is wiring:
- **Single-step**: all containers render simultaneously. Each container's `autoSync: true` (default) — every field change debounces and pushes to the backend immediately.
- **Multi-step**: containers set `autoSync: false`. Mutations only fire when the user clicks "Continue" on a step. Steps are coordinated by a step-module registry (`steps.js`) and visibility is class-driven (`.checkout-step-active`). The boilerplate's `demos` branch ships a reference multi-step implementation under `blocks/commerce-checkout-multi-step`.

## Initialization & extending

### Package

```javascript
import '@dropins/storefront-checkout';
import '../../scripts/initializers/checkout.js';
import ContainerName from '@dropins/storefront-checkout/containers/ContainerName.js';
import { render as CheckoutProvider } from '@dropins/storefront-checkout/render.js';
```

### Initializer config

`initializeCheckout(input: InitializeInput)` accepts:

```typescript
interface InitializeInput {
  langDefinitions?: LangDefinitions;
  models?: Record<string, { transformer: (data: any) => any }>;
  defaults?: {
    isBillToShipping?: boolean;
    selectedShippingMethod?: Selector<ShippingMethod>;
  };
  shipping?: { filterOptions?: Filter<ShippingMethod>; };
  features?: { b2b?: { quotes?: boolean; routeLogin?: () => string | void; }; };
}
```

| Key | Purpose |
|-----|---------|
| `langDefinitions` | Override dictionary strings per locale; deep-merged with defaults |
| `models` | Register transformers for `CartModel`, `CustomerModel`, `NegotiableQuoteModel`, `EstimateShippingModel` |
| `defaults.isBillToShipping` | Initial state of "billing same as shipping" |
| `defaults.selectedShippingMethod` | Selector to pre-pick a shipping method |
| `shipping.filterOptions` | Predicate to hide shipping methods from list |
| `features.b2b.quotes` | Enable negotiable-quote flow; listens to `quote-management/quote-data` |
| `features.b2b.routeLogin` | Returns URL for B2B login routing |

### GraphQL fragment extension (build-time)

```javascript
import { overrideGQLOperations } from '@dropins/build-tools/gql-extend.js';

overrideGQLOperations([
  {
    npm: '@dropins/storefront-checkout',
    operations: [`fragment CUSTOMER_FRAGMENT on Customer { custom_field }`],
  },
]);
```

Exposed fragments: `BILLING_CART_ADDRESS_FRAGMENT`, `SHIPPING_CART_ADDRESS_FRAGMENT`, `CHECKOUT_DATA_FRAGMENT`, `CUSTOMER_FRAGMENT`, `NEGOTIABLE_QUOTE_BILLING_ADDRESS_FRAGMENT`, `NEGOTIABLE_QUOTE_SHIPPING_ADDRESS_FRAGMENT`, `NEGOTIABLE_QUOTE_FRAGMENT`, `AVAILABLE_PAYMENT_METHOD_FRAGMENT`, `SELECTED_PAYMENT_METHOD_FRAGMENT`, `AVAILABLE_SHIPPING_METHOD_FRAGMENT`, `ESTIMATE_SHIPPING_METHOD_FRAGMENT`, `SELECTED_SHIPPING_METHOD_FRAGMENT`.

## Error handling pattern

Centralized capture with optimistic UI and rollback:

1. **Optimistic UI** — DOM updates immediately on user action.
2. **Backend dispatch** — drop-in fires `setShippingAddress` / `setPaymentMethod` / etc.
3. **Rollback on failure** — reverts to last known good state when possible.
4. **Surface via**:
   - Container callbacks: `onCartSyncError`, `onValidationError`, `onServerError`.
   - The `checkout/error` event for global observers.
   - The `ServerError` container with optional `autoScroll` and `onRetry`.

### Callback contracts

```typescript
onCartSyncError?: (error: { method?: any; error: Error; message: string }) => void;
onValidationError?: (error: { field: string; message: string }) => void;
onServerError?: (error: string) => void;
onRetry?: () => void;
```

### ServerError wiring

```javascript
CheckoutProvider.render(ServerError, {
  autoScroll: true,
  onRetry: () => $content.classList.remove('checkout__content--error'),
  onServerError: () => $content.classList.add('checkout__content--error'),
})($serverError);
```

## Event handling pattern + Events list

Events flow through `@dropins/tools/event-bus.js`. The bus is typed pub/sub with `on`, `emit`, and an `eager` replay option.

| Event | Direction | Payload |
|-------|-----------|---------|
| `authenticated` | listens | `boolean` |
| `cart/data` | listens | `Cart \| null` |
| `cart/initialized` | listens | `CartModel \| null` |
| `cart/merged` | listens | `{ oldCartItems: any[] }` |
| `cart/reset` | listens | `void` |
| `cart/updated` | listens | `CartModel \| null` |
| `checkout/error` | emits + listens | `CheckoutError` `{ message: string; code?: string }` |
| `checkout/initialized` | emits + listens | `Cart \| NegotiableQuote \| null` |
| `checkout/updated` | emits + listens | `Cart \| NegotiableQuote \| null` |
| `checkout/values` | emits | `ValuesModel` |
| `locale` | listens | locale identifier |
| `quote-management/quote-data` | listens | `{ quote: NegotiableQuoteModel; permissions: object }` |
| `shipping/estimate` | emits + listens | `ShippingEstimate` |

External vs internal:
- **External**: `authenticated`, `cart/*`, `locale`, `quote-management/quote-data`.
- **Internal**: `checkout/initialized`, `checkout/updated`, `checkout/values`, `shipping/estimate`, `checkout/error`.

The reference multi-step adds a non-typed `checkout/step/completed` event fired by the host block.

## Utilities

| Helper | Signature | Notes |
|--------|-----------|-------|
| `setAddressOnCart` | `(opts: { type: 'shipping' \| 'billing'; debounceMs: number; placeOrderBtn?: RenderAPI }) => (change: AddressFormChange) => void` | Debounced address writer; disables button when supplied |
| `estimateShippingCost` | `(opts: { debounceMs: number }) => (change: AddressFormChange) => void` | Debounced estimator |
| `isVirtualCart` | `(cart?: Cart \| null) => boolean` | All items virtual |
| `isEmptyCart` | `(cart?: Cart \| null) => boolean` | Cart empty or null |
| `getCartShippingMethod` | `(cart?: Cart \| null) => ShippingMethod \| null` | Selected shipping method |
| `getCartAddress` | `(cart: Cart, type?: 'shipping' \| 'billing') => CartAddress \| null` | Address extractor |
| `getCartPaymentMethod` | `(cart?: Cart \| null) => PaymentMethod \| null` | Selected payment method |
| `createFragment` | `(html: string) => DocumentFragment` | HTML to fragment |
| `createScopedSelector` | `(fragment: DocumentFragment) => (sel: string) => HTMLElement \| null` | Scoped `querySelector` |
| `scrollToElement` | `(el: HTMLElement) => void` | Smooth scroll + focus |
| `validateForm` | `(name: string) => boolean` | Native validation |
| `transformAddressFormValuesToCartAddressInput` | `(values: AddressFormValues) => CartAddressInput` | Form → GraphQL input |
| `transformCartAddressToFormValues` | `(address: CartAddress) => AddressFormValues` | GraphQL → form |

## Functions

```typescript
authenticateCustomer(authenticated = false): Promise<any>
estimateShippingMethods(input?: EstimateShippingInput): Promise<ShippingMethod[] | undefined>  // emits: shipping/estimate
getCart(): Promise<Cart>
getCheckoutAgreements(): Promise<CheckoutAgreement[]>
getCompanyCredit(): Promise<CompanyCredit | null>
getCustomer(): Promise<Customer | null>
getNegotiableQuote(input: GetNegotiableQuoteInput = {}): Promise<any>
getStoreConfig(): Promise<any>
initializeCheckout(input: InitializeInput): Promise<any>
isEmailAvailable(email: string): Promise<EmailAvailability>
resetCheckout(): any   // emits: checkout/updated (null)
setBillingAddress(input: BillingAddressInputModel): Promise<any>
setGuestEmailOnCart(email: string): Promise<any>
setPaymentMethod(input: PaymentMethodInputModel): Promise<any>
setShippingAddress(input: ShippingAddressInputModel): Promise<any>
setShippingMethods(input: Array<ShippingMethodInputModel>): Promise<any>
synchronizeCheckout(data: SynchronizeInput): Promise<any>
```

Models:

```typescript
interface Cart {
  id: string;
  email?: string;
  isVirtual: boolean;
  isEmpty: boolean;
  shippingAddresses: CartAddress[];
  billingAddress?: CartAddress;
  selectedPaymentMethod?: PaymentMethod;
  availablePaymentMethods?: PaymentMethod[];
  selectedShippingMethod?: ShippingMethod;
  availableShippingMethods?: ShippingMethod[];
}

interface ShippingMethod {
  code: string;
  carrier: { code: string; title: string };
  amount: Money;
  originalAmount?: Money;  // enables strikethrough
  title: string;
  available: boolean;
}
```

## Slots

| Container | Slot | Context | Purpose |
|-----------|------|---------|---------|
| `LoginForm` | `Heading` | `{ isAuthenticated: boolean; email?: string }` | Conditional heading |
| `LoginForm` | `Title` | default | Custom title |
| `LoginForm` | `Preferences` | `{ email: string; isEmailValid: boolean; isAuthenticated: boolean }` | Marketing opt-in when valid + unauth |
| `PaymentMethods` | `Title` | default | Custom title |
| `PaymentMethods` | `Methods` | `Record<string, PaymentMethodConfig>` | Per-code customization |
| `PlaceOrder` | `Content` | `ContentSlotContext` `{ selectedPaymentMethod, cartId }` | Per-method button content |
| `ShippingMethods` | `Title` | default | Custom title |
| `ShippingMethods` | `ShippingMethodItem` | `ShippingMethod` + `{ onSelect(); onRender(cb) }` | Custom method row |
| `TermsAndConditions` | `Agreements` | `{ appendAgreement(fn: () => AgreementConfig): void }` | Add custom agreements |

```typescript
interface PaymentMethodConfig {
  displayLabel?: boolean;
  enabled?: boolean;
  icon?: string;
  autoSync?: boolean;
  render?: (ctx: SlotContext) => void;
}
```

## Containers

All render via `CheckoutProvider.render(Container, props)(element)`. All expose `active: boolean` (default `true`) — when `false`, no render and no event subscription.

### AddressValidation

Modal-friendly chooser between entered + AVS-suggested address.
```typescript
interface AddressValidationProps {
  active?: boolean;
  suggestedAddress: Partial<CartAddressInput> | null;
  handleSelectedAddress?: (payload: { selection: 'suggested' | 'original'; address: CartAddressInput | null | undefined; }) => void;
}
```

### BillToShippingAddress

"Billing same as shipping" checkbox with conditional billing form trigger.
```typescript
interface BillToShippingAddressProps {
  active?: boolean; autoSync?: boolean;
  onCartSyncError?: (error: any) => void;
  onChange?: (checked: boolean) => void;
}
```
Hides automatically on empty/virtual cart. With `autoSync: true`, persists flag + copies shipping → billing.

### EstimateShipping

Read-only estimate, embedded into cart's `OrderSummary`. Consumer of `shipping/estimate` and `checkout/updated`.

### LoginForm

Email capture / sign-in trigger; tracks `authenticated`.
```typescript
interface LoginFormProps {
  active?: boolean; displayTitle?: boolean; displayHeadingContent?: boolean; autoSync?: boolean;
  onSignInClick?: (payload: { email: string }) => void;
  onSignOutClick?: () => void;
  onCartSyncError?: (error: any) => void;
  onValidationError?: (error: any) => void;
  slots?: { Heading?: SlotFn; Title?: SlotFn; Preferences?: SlotFn; };
}
```

### MergedCartBanner

Alert when sign-in merges anonymous cart. Subscribes to `cart/merged`; renders when `oldCartItems.length > 0`.

### OutOfStock

Detects unavailable items, provides removal + cart-return UI.
```typescript
interface OutOfStockProps {
  active?: boolean;
  onCartProductsUpdate?: (items: Array<{ uid: string; quantity: number }>) => void;
  routeCart?: () => string;
}
```

### PaymentMethods

Primary PSP integration point.
```typescript
interface PaymentMethodsProps {
  active?: boolean; displayTitle?: boolean; autoSync?: boolean;
  onCartSyncError?: (error: any) => void;
  onSelectionChange?: (method: PaymentMethod) => void;
  UIComponentType?: 'ToggleButton' | 'RadioButton';
  slots?: { Title?: SlotFn; Methods?: Record<string, PaymentMethodConfig>; };
}
```
PSPs needing multi-step flows MUST set `autoSync: false` per method.

### PaymentOnAccount

B2B account-credit method.
```typescript
interface PaymentOnAccountProps {
  initialReferenceNumber?: string;
  onReferenceNumberChange?: (value: string) => void;
  onReferenceNumberBlur?: (value: string) => void;
}
```
Pairs with `getCompanyCredit()`; surfaces exceed-limit warning.

### PlaceOrder

Submit button; orchestrates validation + tokenization + placement.
```typescript
interface PlaceOrderProps {
  active?: boolean; disabled?: boolean;
  handleValidation?: () => boolean | Promise<boolean>;
  handlePlaceOrder: (ctx: { paymentMethodCode: string; cartId: string }) => Promise<void>;  // REQUIRED
  slots?: { Content?: SlotFn };
}
```
`handlePlaceOrder` is the only required prop on any checkout container.

### PurchaseOrder

B2B PO method.

### ServerError

Auto-displays server errors with optional auto-scroll + retry. Subscribes to `checkout/error`.

### ShippingMethods

Lists + persists shipping selection; supports strikethrough pricing via `originalAmount`.
```typescript
interface ShippingMethodsProps {
  active?: boolean; displayTitle?: boolean; autoSync?: boolean;
  onCartSyncError?: (payload: { method: ShippingMethod; error: Error }) => void;
  onSelectionChange?: (method: ShippingMethod) => void;
  UIComponentType?: 'RadioButton' | 'ToggleButton';
  slots?: { Title?: SlotFn; ShippingMethodItem?: SlotFn; };
}
```
Filterable via `shipping.filterOptions`.

### TermsAndConditions

Acceptance checkboxes; blocks place order until manual agreements checked. Backed by `getCheckoutAgreements()`; requires **SCP 4.7.1-beta8+** for StoreConfig GraphQL.

## Payment methods integration

1. **Backend prerequisites**: PSP configured in Admin, extension installed, code appears in `availablePaymentMethods`.
2. **Import PSP SDK**: e.g., `import 'https://js.braintreegateway.com/web/dropin/1.43.0/js/dropin.min.js';`.
3. **Mount PSP UI via `Methods` slot with `autoSync: false`**:

```javascript
let braintreeInstance;
CheckoutProvider.render(PaymentMethods, {
  slots: {
    Methods: {
      braintree: {
        autoSync: false,
        render: (ctx) => {
          const $el = document.createElement('div');
          ctx.appendChild($el);
          braintreeWeb.dropin.create(
            { container: $el, authorization: clientToken },
            (err, instance) => { if (!err) braintreeInstance = instance; },
          );
        },
      },
    },
  },
})($paymentMethods);
```

4. **Tokenize + persist + place in `handlePlaceOrder`**:

```javascript
CheckoutProvider.render(PlaceOrder, {
  handlePlaceOrder: async ({ paymentMethodCode, cartId }) => {
    if (paymentMethodCode === 'braintree') {
      const { nonce } = await braintreeInstance.requestPaymentMethod();
      await setPaymentMethod({ code: 'braintree', braintree: { payment_method_nonce: nonce } });
    }
    await orderApi.placeOrder(cartId);
  },
})($placeOrder);
```

5. **Per-method button content** via `PlaceOrder.Content` slot keyed off `selectedPaymentMethod.code`.

Same pattern supports Adyen, Stripe, and any PSP exposing a JS Drop-in/Elements API. Backend-only methods (check/money order, free, bank transfer) keep `autoSync: true`.

## Address verification integration

1. User clicks Place Order.
2. `handleValidation` calls AVS service with the entered shipping address.
3. If normalized suggestion returned, open modal with `AddressValidation`.
4. User picks "original" or "suggested".
5. If "suggested", call `setShippingAddress` before resolving.
6. Resolve `handleValidation` to `true` to proceed, `false` to abort.

```javascript
async function validateShippingAddress() {
  const cart = await getCart();
  const current = getCartAddress(cart, 'shipping');
  const suggestion = await myAVS.normalize(current);
  if (!suggestion) return true;

  return new Promise((resolve) => {
    const $modal = openModal();
    CheckoutProvider.render(AddressValidation, {
      suggestedAddress: suggestion,
      handleSelectedAddress: async ({ selection, address }) => {
        if (selection === 'suggested' && address) await setShippingAddress({ address });
        $modal.close();
        resolve(true);
      },
    })($modal.content);
  });
}

CheckoutProvider.render(PlaceOrder, {
  handleValidation: validateShippingAddress,
  handlePlaceOrder: async ({ cartId }) => { await orderApi.placeOrder(cartId); },
})($placeOrder);
```

Key contracts: AVS responses MUST be transformed to `CartAddressInput`; session storage can bridge modal ↔ form when modal lives outside the checkout block.

## Multi-step pattern

Reference: `aem-boilerplate-commerce`, `demos` branch, `blocks/commerce-checkout-multi-step`.

File structure: `commerce-checkout.js` (decorator), `fragments.js` (DOM), `steps.js` (coordination), `steps/` (per-step modules), `containers.js` (drop-in registry), `components.js` (UI primitives), `constants.js`.

Each step module:
```javascript
{
  async display(data),
  async displaySummary(data),
  async continue(),
  isComplete(data),
  isActive(),
}
```

Navigation: linear with early returns — if current step incomplete, render and exit; else summary and advance. Virtual carts short-circuit shipping step.

`autoSync: false` everywhere; persistence deferred to step's `continue()`: validate → call `setGuestEmailOnCart` / `setShippingAddress` / `setShippingMethods` / `setBillingAddress` / `setPaymentMethod` → swap to summary class → emit custom `checkout/step/completed` → advance.

CSS-driven visibility:
```css
.checkout-step-content { display: none; }
.checkout-step-active .checkout-step-content { display: block; }
.checkout-step-summary { display: none; }
.checkout-step:not(.checkout-step-active) .checkout-step-summary { display: block; }
```

## BOPIS pattern

Reference: boilerplate `demos`, `blocks/commerce-checkout-bopis`.

Backend prerequisites: configure in-store delivery + pickup locations in Admin.

DOM: toggle row (Delivery vs In-Store Pickup), pickup locations panel (radio list, hidden initially), standard shipping form (visible by default).

```javascript
async function onToggle(type) {
  if (type === 'delivery') {
    // show shipping form, hide pickup options
  } else {
    // hide shipping form, show pickup locations
    await fetchPickupLocations();
  }
}
```

On location selection, call `setShippingAddress` with `pickupLocationCode` alongside or in place of street/city. `ShippingMethods` then auto-lists pickup-compatible methods.

## Styling

CSS classes (BEM) + design tokens. Override in `blocks/commerce-checkout/commerce-checkout.css`. No hardcoded values — consume platform tokens.

Container BEM roots:

| Container | Root |
|-----------|------|
| AddressValidation | `.checkout-address-validation` |
| BillToShippingAddress | `.checkout-bill-to-shipping-address` |
| EstimateShipping | `.checkout-estimate-shipping` |
| LoginForm | `.checkout-login-form` |
| MergedCartBanner | `.checkout__banner` |
| OutOfStock | `.checkout-out-of-stock` |
| PaymentMethods | `.checkout-payment-methods` |
| PaymentOnAccount | `.checkout-payment-on-account` |
| PlaceOrder | `.checkout-place-order` |
| PurchaseOrder | `.checkout-purchase-order` |
| ServerError | `.checkout-server-error` |
| ShippingMethods | `.checkout-shipping-methods` |
| TermsAndConditions | `.checkout-terms-and-conditions` |

Sub-elements follow `__element` / `--modifier`.

## Dictionary keys

Root namespace `Checkout`. Override at init time via `langDefinitions`.

Top-level namespaces: `Checkout.AddressValidation`, `Checkout.Addresses`, `Checkout.BillToShippingAddress`, `Checkout.EmptyCart`, `Checkout.EstimateShipping`, `Checkout.LoginForm`, `Checkout.MergedCartBanner`, `Checkout.OutOfStock`, `Checkout.PaymentMethods`, `Checkout.PaymentOnAccount`, `Checkout.PlaceOrder`, `Checkout.PurchaseOrder`, `Checkout.Quote`, `Checkout.ServerError`, `Checkout.ShippingMethods`, `Checkout.Summary`, `Checkout.TermsAndConditions`.

## Architectural notes

### Order of operations

1. **Boot**: `/scripts/initializers/checkout.js` → `initializeCheckout(...)`. Drop-in calls `getCart`, `getCustomer`, `getStoreConfig`, `getCheckoutAgreements`, and (if B2B) `getNegotiableQuote`. Emits `checkout/initialized`.
2. **Render**: Block decorator renders containers. Each subscribes to `checkout/initialized` (eager) + `checkout/updated`.
3. **Interact**: Optimistic local update → debounced mutation (when `autoSync: true`) → `checkout/updated` re-emission.
4. **Validate**: `handleValidation` runs (sync + async). All forms + terms + optional AVS modal.
5. **Tokenize + Place**: `handlePlaceOrder` — PSP tokens, `setPaymentMethod`, `orderApi.placeOrder(cartId)`.
6. **Post**: Order drop-in takes over for confirmation.

### Race conditions

- **Address vs shipping methods**: address change invalidates method list; drop-in re-emits `checkout/updated` with fresh `availableShippingMethods`; stale selections roll back via `onCartSyncError`. Multi-step (`autoSync: false`) must call `estimateShippingMethods` or re-fetch before continuing.
- **PSP tokenization vs cart mutation**: editing shipping mid-tokenization can void the token — disable `PlaceOrder` while tokenization is inflight.
- **Cart merge on sign-in**: `cart/merged` arrives asynchronously after `authenticated: true`; containers re-render on the subsequent `checkout/updated`.
- **Multiple shipping addresses (B2B)**: `Cart.shippingAddresses` is an array but the storefront drop-in treats it as single for B2C; B2B multi-shipping is out of scope.

### Recovery from failure

- **API failure**: containers roll back, emit `checkout/error`, call `onCartSyncError`. `ServerError` surfaces with `onRetry`.
- **Tokenization failure**: `handlePlaceOrder` should `throw` — `PlaceOrder` catches and surfaces via `checkout/error`.
- **Soft validation failure**: `handleValidation` returns `false`; integrator surfaces messages via `onValidationError`.
- **AVS modal cancel**: resolve `handleValidation` to `false` to abort or `true` to continue with original address.

### Eager subscription

The event bus's `eager: true` replays the last-emitted payload immediately. A container mounted **after** `checkout/initialized` still hydrates correctly. Rely on this rather than reading from a global cache.

### Pure-headless usage

For mobile/native or custom React without the drop-in UI, import only `@dropins/storefront-checkout/api.js` for the functions and skip containers. Re-implement event bus wiring as needed.

### `synchronizeCheckout` vs individual setters

`synchronizeCheckout(data)` is a multi-field commit (typically used for restoring partial sessions). Individual setters emit one `checkout/updated` each — prefer them for granular changes.

### `resetCheckout` semantics

Synchronous (returns `any`, not a promise). Emits `checkout/updated` with `null`. Clears in-memory checkout state — does NOT void the cart.

### `TermsAndConditions` store-config dependency

`getCheckoutAgreements` queries `StoreConfig`. Requires **SCP 4.7.1-beta8+**. Without it the container renders nothing.

### Place-order disablement triggers

Place Order button is disabled when:
- `disabled: true` explicitly.
- Async `handleValidation` is pending.
- Async `handlePlaceOrder` is pending.
- `setAddressOnCart` was wired with `placeOrderBtn` and an address mutation is inflight.

## Source URLs

- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/quick-start/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/initialization/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/extending/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/error-handling/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/event-handling/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/utilities/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/slots/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/styles/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/functions/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/events/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/dictionary/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/address-validation/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/bill-to-shipping-address/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/estimate-shipping/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/login-form/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/merged-cart-banner/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/out-of-stock/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/payment-methods/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/payment-on-account/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/place-order/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/purchase-order/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/server-error/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/shipping-methods/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/containers/terms-and-conditions/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/tutorials/add-payment-method/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/tutorials/address-integration/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/tutorials/validate-shipping-address/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/tutorials/buy-online-pickup-in-store/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/checkout/tutorials/multi-step/
