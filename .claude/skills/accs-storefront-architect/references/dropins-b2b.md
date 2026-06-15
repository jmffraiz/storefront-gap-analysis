# B2B Drop-ins Reference

Six drop-ins layered on top of the B2C drop-in framework: Company Management, Company Switcher, Purchase Order, Quote Management, Requisition List, Quick Order.

> **B2B Compatibility Package required.** All B2B drop-ins depend on Adobe Commerce's B2B Compatibility Package being installed on the backend AND B2B features enabled at `Stores > Settings > Configuration > General > B2B Features` (Enable Company, Enable Company Registration, Enable Quotes, Enable Purchase Order, Enable Quick Order, etc.). Visibility of features on the storefront is gated by company role ACL resources (`Magento_Company::*`, `Magento_NegotiableQuote::*`, `Magento_PurchaseOrder::*`) propagated through the `auth/permissions` event.

## B2B model

| Term | Meaning |
|------|---------|
| **Company** | Top-level B2B account containing users, addresses, credit, hierarchy. |
| **Company user** | A customer associated with a company. |
| **Company admin** | Privileged user (initial seat holder by default) with full company permissions. |
| **Role** | Named permission bundle (e.g., "Buyer", "Approver", "Admin"). Roles map to ACL resources. |
| **Company hierarchy** | Tree of teams / divisions under a parent company. Users belong to a node. |
| **Company credit** | Credit limit + available balance — surfaces in checkout via `PaymentOnAccount`. |
| **Negotiable quote** | Pre-order document with custom pricing, negotiated between buyer and sales rep. Becomes an order on acceptance. |
| **Purchase order (PO)** | Buyer-initiated order document requiring internal approval (via approval rules) before becoming a Commerce order. |
| **Approval rule** | Conditions (amount, SKU, shipping) that send a PO through approver(s) before submission. |
| **Requisition list** | Saved set of items for bulk re-ordering. Shareable across company users. |
| **Quick order** | Multi-line bulk-add UI (CSV upload, multiple SKU input, variant grid). |

# Company Management Drop-in

`@dropins/storefront-company-management`. Surfaces for company self-service: registration, invitations, profile, hierarchy, users, roles, credit.

## Containers

- `CompanyRegistration` — public company sign-up form (separate from individual customer sign-up).
- `AcceptInvitation` — landing page for invited company users to set password.
- `CompanyProfile` — company info edit (name, tax ID, contact).
- `CompanyCredit` — credit limit + balance display.
- `CompanyHierarchy` — tree view + edit.
- `CompanyStructure` — top-level structure including roles + permissions wiring.
- `CompanyUsers` — user list, invite, suspend, role assignment.
- `RolesAndPermissions` — role CRUD with ACL resource picker.
- `CustomerCompanyInfo` — read-only company info shown inside a customer's account page.

## Initialization

```javascript
await initializers.mountImmediately(initialize, {
  langDefinitions: { /* ... */ },
});
```

## Functions

Per-container fetchers/mutators — registration, invitation accept, user CRUD, role CRUD, credit read.

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `company-management/data` | emit | `CompanyModel` |
| `auth/permissions` | listen | updated ACL set |

# Company Switcher Drop-in

`@dropins/storefront-company-switcher`. For users who belong to multiple companies (consultants, multi-tenant buyers).

## Container

`CompanySwitcher` — dropdown of accessible companies, switching emits `companyContext/changed`.

## Initialization

```javascript
await initializers.mountImmediately(initialize, {
  langDefinitions: { /* ... */ },
});
```

## Event

| Event | Direction | Payload |
|-------|-----------|---------|
| `companyContext/changed` | emit | `{ companyId, companyName }` |

Cart, Checkout, Account, Order all listen — switching company resets cart context (typically to a fresh cart for that company).

# Purchase Order Drop-in

`@dropins/storefront-purchase-order`. Approval-driven order documents.

## Containers

- `ApprovalRulesList` — list configured rules.
- `ApprovalRuleForm` — create/edit rule.
- `ApprovalRuleDetails` — read rule.
- `CompanyPurchaseOrders` — all POs across the company (admin view).
- `CustomerPurchaseOrders` — current user's POs.
- `RequireApprovalPurchaseOrders` — POs pending current user's approval.
- `PurchaseOrderApprovalFlow` — step-by-step approval timeline.
- `PurchaseOrderConfirmation` — post-submission confirmation.
- `PurchaseOrderStatus` — status banner with phase indicators.
- `PurchaseOrderCommentForm` / `PurchaseOrderCommentsList` — discussion thread.
- `PurchaseOrderHistoryLog` — audit log of state transitions.

## Flow

1. Buyer creates a cart → clicks "Create Purchase Order" at checkout (via Checkout's `PurchaseOrder` payment container).
2. PO matches an `ApprovalRule` (amount > threshold, certain SKU, etc.) → routed to approver(s).
3. Approver(s) approve or reject. On full approval → PO converts to a Commerce order.

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `purchase-order/data` | emit | `PurchaseOrderModel` |

# Quote Management Drop-in

`@dropins/storefront-quote-management`. Negotiable quotes between buyer and seller rep.

## Containers

- `RequestNegotiableQuoteForm` — initiate a quote request from a cart.
- `QuotesListTable` — list of all quotes (states: Submitted, Pending, Declined, Open, Updated, Closed, Ordered, Expired).
- `ManageNegotiableQuote` — single-quote workspace (line items, comments, accept/decline).
- `ItemsQuoted` — line item table.
- `OrderSummary` (quote-flavored) — totals with negotiated pricing.
- `OrderSummaryLine` — custom row.
- `QuoteCommentsList` — buyer ↔ seller thread.
- `QuoteHistoryLog` — audit.
- `ShippingAddressDisplay` — addresses tied to the quote.

Quote templates:
- `QuoteTemplatesListTable` — saved templates for repeat negotiations.
- `ManageNegotiableQuoteTemplate`, `ItemsQuotedTemplate`, `QuoteTemplateCommentsList`, `QuoteTemplateHistoryLog`.

## Flow

1. Buyer adds items to cart → requests a quote via `RequestNegotiableQuoteForm`.
2. Seller rep adjusts pricing / terms in Commerce Admin.
3. Buyer accepts → checkout with the negotiated cart (Checkout's `features.b2b.quotes: true` listens for `quote-management/quote-data`).
4. Place order via `placeNegotiableQuoteOrder` (Order drop-in).

## Event

| Event | Direction | Payload |
|-------|-----------|---------|
| `quote-management/quote-data` | emit | `{ quote, permissions }` |

# Requisition List Drop-in

`@dropins/storefront-requisition-list`. Saved shopping lists shareable across company users.

## Containers

- `RequisitionListSelector` — picker (used on PDP / PLP "Add to Requisition List").
- `RequisitionListForm` — create / edit list.
- `RequisitionListHeader` — list metadata.
- `RequisitionListGrid` — line items grid.
- `RequisitionListView` — read-only view (when shared).
- `ShareRequisitionListContent` — share dialog (by email or by token).
- `SharedRequisitionList` — read-only public view via token.

## Flow

- Save a current cart as a list, OR add individual items from PDP/PLP.
- Share via email (`shareRequisitionListByEmail`) or via token URL (`shareRequisitionListByToken` / `importSharedRequisitionList`).
- "Add all to cart" mass-adds the list.

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `requisitionList/alert` | emit | `{ message }` (cart listens for bulk-add toast) |

# Quick Order Drop-in

`@dropins/storefront-quick-order`. Bulk add UI for B2B buyers who already know SKUs.

## Containers

- `QuickOrderItems` — single-line SKU + quantity input grid.
- `QuickOrderMultipleSku` — multi-SKU textarea (paste from spreadsheet).
- `QuickOrderCsvUpload` — CSV import.
- `QuickOrderVariantsGrid` — variant-aware quick add for configurable products.

## Flow

1. Type or paste SKUs → autocomplete validates.
2. Confirm → bulk `addProductsToCart`.

## Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `quick-order/data` | emit | `{ items, errors }` |

# Architectural notes

## When to enable B2B

Turn B2B on (`commerce-b2b-enabled: true`, install B2B SCP) when any of:
- Companies (org accounts).
- Negotiable quotes.
- Purchase orders with approval rules.
- Requisition lists shared across buyers.
- Quick order workflows.

Leave off for pure B2C — keep the storefront leaner and avoid B2B-only fields in cart/checkout models.

## Compatibility package required

Adobe Commerce backend needs:
1. Storefront Compatibility Package (base) — drop-in support.
2. Storefront Compatibility B2B Package — B2B drop-in support.

On ACCS: both auto-installed. On PaaS: composer install both.

B2B SCP adds GraphQL: `assignChildCompany`, `unassignChildCompany`, `importSharedRequisitionList`, `shareRequisitionListByToken`, `shareRequisitionListByEmail`, `setQuoteTemplateExpirationDate`, `placeNegotiableQuoteOrderV2`, `negotiableQuoteTemplates`, `sharedRequisitionList`.

## Payment-on-Account vs Purchase Order

Both are B2B-specific payment methods on Checkout — distinct concepts:

- **Payment on Account** (`PaymentOnAccount` container in Checkout) — pay from company credit line; checks `getCompanyCredit()`; supports reference number.
- **Purchase Order** (`PurchaseOrder` container in Checkout) — submits a PO that goes through approval rules; not a direct charge.

A B2B checkout often offers both — buyer picks based on auth scope and order context.

## Integration with Checkout's PaymentOnAccount / PurchaseOrder

The Checkout drop-in's `PaymentMethods` container exposes the slots; Payment-on-Account and Purchase-Order containers slot in via `PaymentMethods.Methods.<method-code>`:

```javascript
CheckoutProvider.render(PaymentMethods, {
  slots: {
    Methods: {
      payment_on_account: { render: (ctx) => provider.render(PaymentOnAccount, {...})(ctx) },
      purchase_order:     { render: (ctx) => provider.render(PurchaseOrder,    {...})(ctx) },
    },
  },
});
```

## Role-based UI visibility

B2B drop-ins read the `auth/permissions` event (emitted by User Auth) and render only what the user is authorized for. Examples:
- Without `Magento_Company::manage_users`, `CompanyUsers` is hidden.
- Without `Magento_PurchaseOrder::manage_purchase_orders`, `RequireApprovalPurchaseOrders` is hidden.
- Without `Magento_NegotiableQuote::view`, the quotes nav entry is hidden.

Always trust the permission event over assumptions about user roles.

## Slot strategy

Same `(ctx) => void` pattern as B2C drop-ins. Append-once + `onChange`-mutate rule applies.

## Event-bus discipline

B2B drop-ins are heavy event publishers. Three principles:
1. Always subscribe with `eager: true` for state-driven hosts (badges, header counts).
2. Namespace via `scope` when rendering multiple instances on one page.
3. Treat `companyContext/changed` as a global reset signal — cart and order should re-fetch.

## Localization

Each B2B drop-in has its own dictionary namespace (`CompanyManagement`, `PurchaseOrder`, `QuoteManagement`, `RequisitionList`, `QuickOrder`, `CompanySwitcher`). Override via `langDefinitions` per usual.

## Versioning

B2B drop-ins evolve faster than B2C — Adobe iterates on quote and PO flows. Pin versions in `package.json` and review changelogs before bumping majors.

## Source URLs

All B2B drop-ins are under: https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/

Per drop-in landing pages:
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/company-management/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/company-switcher/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/purchase-order/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/quote-management/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/requisition-list/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins-b2b/quick-order/
