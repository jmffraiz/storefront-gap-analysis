# User Account, User Auth, Order — Drop-in Reference

Three drop-ins that together cover authentication, profile, addresses, payment tokens, order history, returns, and order management — including seller-assisted buying.

# User Auth Drop-in

`@dropins/storefront-auth` (v3.2.0). Handles sign-in, sign-up, password reset, password update, email confirmation, reCAPTCHA, and the canonical `authenticated` event that other drop-ins listen for.

## Purpose

- Sign-in (`SignIn`).
- Sign-up (`SignUp`).
- Combined entry (`AuthCombine`) — toggles between sign-in and sign-up.
- Forgot password flow (`ResetPassword`).
- New-password creation (`UpdatePassword`).
- Success notification banner.
- reCAPTCHA gating on auth-related mutations.

## Initialization

Endpoint: `commerce-core-endpoint`. Headers: `all` group.

```javascript
await initializers.mountImmediately(initialize, {
  langDefinitions: { en_US: { Auth: { /* ... */ } } },
  adobeCommerceOptimizer: false,  // true for ACO Price Book propagation
  recaptcha: {
    enabled: true,
    type: 'v3',     // or 'enterprise'
    siteKey: '...',
  },
});
```

## Functions

- `getCustomerToken(email, password): Promise<token>` — sets `auth_dropin_user_token` cookie + `Authorization: Bearer` header.
- `revokeCustomerToken(): Promise<void>` — clears both.
- `getCustomerData(): Promise<Customer>`.
- `createCustomer(input): Promise<Customer>` (uses `createCustomerV2`).
- `createCustomerAddress(input): Promise<Address>`.
- `requestPasswordResetEmail(email): Promise<void>`.
- `resetPassword(token, newPassword): Promise<void>`.
- `confirmEmail(token): Promise<void>`.
- `resendConfirmationEmail(email): Promise<void>`.
- `getAttributesForm(formCode): Promise<AttributesForm>` — EAV metadata for dynamic forms.
- `verifyToken(): Promise<boolean>` — validates current token and emits `authenticated`.
- `getCustomerRolePermissions()` (B2B).
- `getAdobeCommerceOptimizerData()` — returns `{ priceBookId, groupUid }` for ACO.

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `authenticated` | emit | `boolean` |
| `auth/adobe-commerce-optimizer` | emit | `{ adobeCommerceOptimizer: { priceBookId } }` |
| `auth/error` | emit | `Error` |
| `auth/group-uid` | emit | `string` (customer group UID) |
| `auth/permissions` | emit | `string[]` (B2B ACL resources) |

`authenticated` is the canonical session-bridging signal — Cart, Checkout, Account, Order, and B2B drop-ins all listen for it.

## Containers

- `AuthCombine` — sign-in + sign-up combined UI.
- `SignIn` — email/password form.
- `SignUp` — registration form (EAV-driven via `getAttributesForm`).
- `ResetPassword` — request reset email form.
- `UpdatePassword` — set new password from reset link.
- `SuccessNotification` — post-action confirmation banner.

All containers accept `slots`, `recaptcha` toggle, `routeRedirect` for post-action navigation.

## reCAPTCHA integration

Hook: `@dropins/tools/recaptcha.js`. Order:
1. `setEndpoint(endpoint)` — recaptcha config endpoint.
2. `setConfig({ siteKey, type })`.
3. `initReCaptcha()` — bootstraps grecaptcha.
4. `verifyReCaptcha(action)` — returns a token to attach to the mutation.

Adobe Commerce v3 and Enterprise both supported.

## Seller-Assisted Buying (SAB) consent slot

`AuthCombine` exposes `RemoteShoppingAssistanceConsent` slot with input `allowRemoteShoppingAssistance` — used during sign-up to capture buyer consent to seller-assisted buying.

## Architectural notes

- Token cookie: `auth_dropin_user_token`. Same-origin first-party preferred (use Fastly VCL); cross-origin requires `SameSite=None; Secure`.
- SAB session cookie (separate): `auth_dropin_admin_session`.
- Init Auth BEFORE Account / Order so the `authenticated` event reaches them via `eager: true`.

# User Account Drop-in

`@dropins/storefront-account` (v3.2.0). Authenticated user surfaces: profile, addresses, payment tokens, order history, seller-assisted buying activity.

## Purpose

- Customer information edit (name, email, password) — `CustomerInformation`.
- Address book CRUD — `Addresses`, `AddressForm`, `AddressValidation`.
- Stored payment methods — `PaymentMethods`.
- Order history list — `OrdersList`.
- Seller-assisted buying activity & settings — `SellerAssistedBuyingActivity`, `SellerAssistedBuyingSettings`.
- Account navigation sidebar (via `commerce-account-sidebar` block).

## Initialization

Endpoint: `commerce-core-endpoint`. Headers: `all` group.

```javascript
await initializers.mountImmediately(initialize, {
  langDefinitions: { en_US: { Account: { /* ... */ } } },
});
```

## Functions

- `updateCustomer(input)`, `updateCustomerEmail(input)`, `updateCustomerPassword(currentPassword, newPassword)`.
- `getCustomer()`, `getCustomerAddress(id)`.
- `createCustomerAddress(input)`, `updateCustomerAddress(id, input)`, `deleteCustomerAddress(id)`.
- `removeCustomerAddress(id)`.
- `getCustomerPaymentTokens()`, `deletePaymentToken(token)`.
- `getOrderHistoryList(options)`.
- `getCountries()`, `getRegions(countryId)`.
- `getStoreConfig()`.
- `getAttributesForm(formCode)`.
- `getAdminAssistanceActions()` — SAB.

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `error` | emit | `Error` |
| `account/customerPaymentTokens` | listen | `PaymentToken[]` |
| `companyContext/changed` | listen | new company context (B2B) |

## Containers

- `Addresses` — address-book listing with CRUD actions.
- `AddressForm` — single-address create/edit form.
- `AddressValidation` — AVS modal pair (also available in Checkout).
- `CustomerInformation` — name/email/password edit.
- `OrdersList` — paginated order history with status badges.
- `PaymentMethods` — stored card list.
- `SellerAssistedBuyingSettings` — toggle SAB on customer.
- `SellerAssistedBuyingActivity` — list of admin-assisted actions.

## Sidebar

The `commerce-account-sidebar` block (boilerplate) renders a nav list with active-state. Hosts the Account drop-in's `Layout` / `Sidebar` container.

## Tutorials

- **Customize layout** — override slots in `OrdersList` / `Addresses` for branded headers, custom empty states.
- **Stored payment methods (Adobe Payment Services)** — wire Payment Services delete callbacks.
- **Validate address** — same AVS pattern as Checkout's `AddressValidation`.

# Order Drop-in

`@dropins/storefront-order` (v3.2.0). Order detail, status tracking, returns, cancellation, reorder, guest order lookup, negotiable-quote-order placement.

## Purpose

- Order header (number, date, status).
- Order product list with reorder action.
- Order cost summary (subtotal, tax, shipping, discount, total).
- Customer details (shipping/billing address, contact).
- Shipping status (tracking, carrier, ETA).
- Order comments (CSR/customer comments).
- Order cancellation (both customer-initiated and guest variant).
- Returns request (full flow) + return detail + returns list.
- Order search (guest order lookup by number + email + last name).

## Initialization

Endpoint: `commerce-core-endpoint`.

```javascript
await initializers.mountImmediately(initialize, {
  langDefinitions: { en_US: { Order: { /* ... */ } } },
});
```

## Functions

- `placeOrder(cartId)`, `setPaymentMethodAndPlaceOrder(input)` (atomic).
- `placeNegotiableQuoteOrder(input)` (B2B).
- `cancelOrder(orderId, reason)`, `confirmCancelOrder(token)`.
- `requestGuestOrderCancel(input)` (guest), `confirmCancelOrder(token)` (guest two-step).
- `requestReturn(input)`, `requestGuestReturn(input)`, `confirmGuestReturn(token)`.
- `reorderItems(orderId)` — clones items into current cart.
- `getOrderDetailsById(orderNumber)`.
- `getGuestOrder(number, email, lastName)`, `guestOrderByToken(token)`.
- `getCustomerOrdersReturn()` — returns list.
- `getStoreConfig()`, `getAttributesForm()`, `getAttributesList()`.

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `order/data` | emit | `OrderModel` |
| `order/placed` | emit | `OrderModel` |
| `order/error` | emit | `Error` |
| `cart/reset` | emit | `void` (after order placed) |
| `companyContext/changed` | listen | new company context (B2B) |

## Containers (12)

- `OrderHeader` — order number, date, status.
- `OrderProductList` — line items, reorder button per item.
- `OrderCostSummary` — totals.
- `CustomerDetails` — shipping + billing addresses.
- `ShippingStatus` — tracking detail, carrier, ETA.
- `OrderStatus` — status banner with phase pills.
- `OrderComments` — list + add comment.
- `OrderCancelForm` — cancellation request form.
- `OrderSearch` — guest order lookup form.
- `CreateReturn` — return request form (line selection + reasons).
- `OrderReturns` — return list within an order.
- `ReturnsList` — all returns for the customer.

## Guest cancellation pattern (two-step)

1. `requestGuestOrderCancel({orderId, email, lastName, reason})` — Commerce sends confirmation email with token.
2. User clicks link → page calls `confirmCancelOrder(token)`.

Guest returns follow the same two-step pattern (`requestGuestReturn` → `confirmGuestReturn`).

## Seller-Assisted Buying integration

When `adminAssistedOrder` field is true on the order, the `OrderStatusContent` container shows a badge and label via:
- CSS: `.order-order-status-content__admin-assisted` class.
- Dictionary: `Order.OrderStatusContent.adminAssistedLabel`.

## Architectural notes

### Auth tokens, session bridging

- Cart, Checkout, Account, Order all listen for `authenticated` from Auth.
- Account/Order use `Authorization: Bearer` header injected by the Auth drop-in's `setFetchGraphQlHeader` call.
- SAB cookies: `auth_dropin_admin_session` distinct from `auth_dropin_user_token`.

### Returns flow

1. Customer initiates from `OrderProductList` row action or from `ReturnsList`.
2. `CreateReturn` form: select items, reasons, quantities.
3. `requestReturn(input)` → Commerce creates return record.
4. `OrderReturns` renders within order detail; `ReturnsList` shows all returns.

### Seller-assisted buying use cases

- CSR signs in to customer's account on customer's behalf — emits `auth/permissions` with admin role.
- Customer toggles `allowRemoteShoppingAssistance` to opt-in/out.
- Order placed by admin shows `adminAssistedOrder=true` for transparency.

### Init order

Initialize User Auth first so `authenticated` event is published. Account and Order initializers subscribe with `eager: true` and bootstrap when the event fires.

### Banner integration

The boilerplate's header renders a SAB banner via `renderSellerAssistedBuyingBanner.js` when the SAB session cookie is set.

## Source URLs (all three drop-ins)

User Account:
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/user-account/ (+ all sub-pages)

User Auth:
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/user-auth/ (+ all sub-pages)

Order:
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/order/ (+ all sub-pages)
