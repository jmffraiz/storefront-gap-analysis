# Drop-ins — Shared Concepts

Cross-cutting reference for all Adobe Commerce storefront drop-ins (`@dropins/storefront-*`): what they are, how they're extended, the slot / event / styling / dictionary / linking contracts, and the build-time GraphQL extension hook.

## What drop-ins are

Framework-agnostic UI primitives, distributed as npm packages, that render Adobe Commerce surfaces (PDP, PLP, cart, checkout, etc.) inside any host — EDS blocks, React apps, custom Web Components — through:

- A shared **event bus** (`@dropins/tools/event-bus.js`) for cross-drop-in state.
- A shared **GraphQL fetcher** (`@dropins/tools/fetch-graphql.js`) for backend calls.
- A shared **initializer** (`@dropins/tools/initializer.js`) for lifecycle management.
- A shared **design token system** (`@dropins/tools/design-tokens.css`).

Drop-ins are NOT framework components (no React, no Preact peer dependency). They render imperatively into a DOM element you control.

### Drop-in vs container vs SDK component

| Term | Meaning |
|------|---------|
| **Drop-in** | The npm package (`@dropins/storefront-cart`, `@dropins/storefront-pdp`). |
| **Container** | A composite renderable inside a drop-in (`CartSummaryList`, `OrderSummary`, `ProductGallery`). Mounted via `provider.render(Container, props)(element)`. |
| **Slot** | A named extension point inside a container (`Heading`, `Footer`, `Methods`). Receives a `ctx` object with `appendChild`, `replaceWith`, `onChange`, etc. |
| **SDK component** | A lower-level primitive from `@dropins/storefront-elsie` (`Button`, `Card`, `Picker`, `Modal`). Used internally by containers; available to host blocks. |
| **Function** | An imperative API exported by `@dropins/storefront-<name>/api.js` (`addProductsToCart`, `setPaymentMethod`). Use when bypassing UI. |

## Extend vs create from scratch

Decision criteria (from Adobe's `extend-or-create`):

| Customization | Approach |
|---------------|----------|
| Change a label, color, spacing | Dictionary + CSS tokens |
| Add UI to a known extension point | Slot |
| Listen to drop-in state for sibling component | Event bus subscription |
| Add a backend field to the drop-in's GraphQL | `overrideGQLOperations` + model transformer |
| Replace an internal rendering area | Slot with `replaceWith` |
| Change a container's logic | Fork the container (loses upgrade safety) |
| New commerce surface not covered by any drop-in | Create a new drop-in via SDK |

Adobe ships **eight B2C drop-ins** (PDP, Product Discovery, Cart, Checkout, User Auth, User Account, Order, Wishlist) plus **Payment Services**, **Personalization**, **Recommendations**, and **six B2B drop-ins** (Company Management, Company Switcher, Purchase Order, Quote Management, Requisition List, Quick Order). Default to extend; reserve "create" for genuinely novel commerce surfaces.

## Quick start

Every drop-in initialization follows the same pattern:

```javascript
import { initializers } from '@dropins/tools/initializer.js';
import { initialize } from '@dropins/storefront-<feature>/api.js';
import { setEndpoint, setFetchGraphQlHeaders } from '@dropins/tools/fetch-graphql.js';
import { getConfigValue, getHeaders } from '../configs.js';

setEndpoint(await getConfigValue('commerce-endpoint'));  // or commerce-core-endpoint
setFetchGraphQlHeaders(await getHeaders('cs'));          // or 'all'

await initializers.mountImmediately(initialize, {
  langDefinitions: { /* dictionary overrides */ },
  models: { /* transformer hooks */ },
  // drop-in-specific options
});
```

Then in a block:

```javascript
import Container from '@dropins/storefront-<feature>/containers/Container.js';
import { render as provider } from '@dropins/storefront-<feature>/render.js';

await provider.render(Container, {
  /* container-specific props */
  slots: { /* slot overrides */ },
})(domElement);
```

## Drop-ins inside EDS blocks (commerce blocks)

Adobe ships ~30 Commerce-specific blocks in the AEM Boilerplate Commerce repo. Each is a thin EDS block that mounts one or more drop-ins.

Mapping (illustrative):
- `blocks/product-details` → `@dropins/storefront-pdp` (per-area containers)
- `blocks/product-list-page` → `@dropins/storefront-product-discovery`
- `blocks/commerce-cart` → `@dropins/storefront-cart`
- `blocks/commerce-mini-cart` → `@dropins/storefront-cart` (MiniCart container)
- `blocks/commerce-checkout` → `@dropins/storefront-checkout` + `@dropins/storefront-order`
- `blocks/commerce-login` / `create-account` / `forgot-password` / `create-password` → `@dropins/storefront-auth`
- `blocks/commerce-account-sidebar` / `addresses` / `customer-information` / `orders-list` / `returns-list` → `@dropins/storefront-account`
- `blocks/commerce-wishlist` → `@dropins/storefront-wishlist`
- `blocks/product-recommendations` → `@dropins/storefront-recommendations`

DOM contract: the block reads its authored config table via `readBlockConfig(block)`, scaffolds DOM with `createContextualFragment`, mounts each drop-in container into the right scaffolded node, and wires shared state through `events.on(...)`.

## Styling

CSS custom properties (design tokens) are the only sanctioned customization. Hard-coded values break theming.

Token families:
- `--color-*` — neutrals (`--color-neutral-100` … `--color-neutral-900`), brand (`--color-brand-50` … `--color-brand-900`), positive / negative / warning semantic colors.
- `--spacing-*` — `xsmall`, `small`, `medium`, `large` and per-component overrides.
- `--type-*` — typography (font-family, font-size, line-height, weight per role).
- `--shape-*` — border-radius scale (`--shape-default-radius`, etc.).
- `--grid-*` — layout grid tokens.

Container CSS classes follow BEM: `.{drop-in}-{container}__{element}--{modifier}` (e.g., `.cart-cart-summary-list__heading`, `.checkout-payment-methods__spinner`).

Override location in the boilerplate: `blocks/{block-name}/{block-name}.css`.

## Branding

Brand theming hooks: logo, primary color, secondary color, accent, font family. Adobe documents these as CSS variables consumed by every drop-in. Set them globally in `styles/styles.css` to brand the entire storefront in one place.

```css
:root {
  --color-brand-500: #003366;     /* primary brand */
  --color-brand-700: #002244;     /* hover / active */
  --type-headline-1-font-family: 'YourBrandFont', serif;
}
```

## Labeling (dictionaries)

All user-facing strings live in dictionary trees, keyed by namespace per drop-in.

```javascript
await initializers.mountImmediately(initialize, {
  langDefinitions: {
    en_US: {
      Cart: { Cart: { heading: 'Bag' } },           // Cart drop-in
      Checkout: { LoginForm: { title: 'Email' } },  // Checkout drop-in
      PDP: { addToCart: 'Add to Bag' },             // PDP drop-in
    },
    fr_FR: { /* … */ },
  },
});
```

In the boilerplate, dictionaries are loaded from authored `placeholders/{feature}.json` (a spreadsheet) via `fetchPlaceholders` and passed to the initializer. This lets merchants edit copy without redeploying code.

Language fallback chain: container prop → `setGlobalLocale()` → `navigator.language` → `'en_US'`.

## Localizing links

`@dropins/tools/lib/linking.js` exposes `rootLink(path)` and `decorateLinks(element)`:

```javascript
import { rootLink, decorateLinks } from '@dropins/tools/lib/aem.js';

const cartHref = rootLink('/cart');               // '/en-ca/cart' when on /en-ca/...
decorateLinks(document.body);                     // rewrites all <a href="/..."> with prefix
```

This automatically threads the active locale folder onto outbound links, keeping multistore navigation correct.

## Slots

Slot signature:
```typescript
(ctx: SlotContext) => void
```

Where `SlotContext` exposes:
- `ctx.appendChild(element)` — append to slot host.
- `ctx.replaceWith(element)` — replace slot default content.
- `ctx.onChange(callback)` — re-run when slot inputs change. Reacts to `cart/data`, `pdp/data`, etc.
- `ctx.item`, `ctx.cart`, etc. — slot-specific data depending on container.

**Critical rule**: do not call `appendChild` inside `onChange`. Append the wrapper once; mutate its contents inside `onChange`. Else DOM compounds with every state change.

Default slots exist on most containers (`Heading`, `Footer`). Named slots vary per container — see each drop-in's reference for the full catalog.

## Layouts

Some drop-ins (notably User Account) ship multiple built-in layouts via the `Layout` container or `sidebar` extension. Most drop-ins are layout-agnostic: the host block arranges containers as it pleases.

## Events bus API

```javascript
import { events } from '@dropins/tools/event-bus.js';

// Subscribe
const sub = events.on('cart/data', handler, { eager: true });
sub.off();   // unsubscribe

// Emit
events.emit('checkout/values', payload);

// Last payload
const last = events.lastPayload('cart/data');

// Scope (per-instance)
events.on('pdp/data', handler, { scope: 'quick-view' });

// Debug
events.enableLogger(true);
```

`eager: true` replays the most recent payload synchronously when you subscribe — essential for late-mounting host components (mini-cart count, badges) that miss the initial emit.

## Common events (selected)

| Event | Direction | Drop-in |
|-------|-----------|---------|
| `authenticated` | listen | user-auth → all others |
| `cart/data`, `cart/updated`, `cart/initialized` | emit (cart), listen (others) | cart |
| `cart/product/added`, `cart/product/updated`, `cart/product/removed` | emit | cart |
| `cart/merged`, `cart/reset` | emit | cart |
| `checkout/initialized`, `checkout/updated` | emit | checkout |
| `checkout/error`, `checkout/values` | emit | checkout |
| `shipping/estimate` | emit + listen | cart, checkout |
| `pdp/data`, `pdp/values`, `pdp/valid`, `pdp/setValues` | emit | pdp |
| `search/loading`, `search/result`, `search/error` | emit | product-discovery |
| `wishlist/data`, `wishlist/alert`, `wishlist/initialized`, `wishlist/reset` | emit | wishlist |
| `order/data`, `order/placed`, `order/error` | emit | order |
| `auth/permissions`, `auth/group-uid`, `auth/error` | emit | user-auth |
| `auth/adobe-commerce-optimizer` | emit | user-auth (ACO only) |
| `recommendations/data` | emit | recommendations |
| `personalization/updated` | emit | personalization |
| `companyContext/changed` | emit | company-switcher |
| `quote-management/quote-data` | emit | quote-management |
| `requisitionList/alert` | emit | requisition-list |
| `payment-services/initialized/checkout`, `payment-services/initialized/product-detail` | emit | payment-services |
| `locale` | listen | most drop-ins |
| `error` | emit | all |

## Extending

Six mechanisms, from cheapest to most invasive:

1. **Slots** — `(ctx) => void`. Drop-in calls back into your code at known extension points.
2. **Events** — listen / emit. React to state without touching the drop-in's render tree.
3. **Custom containers in the same block** — render your own DOM alongside the drop-in.
4. **Dictionaries** — override strings.
5. **Models / transformers** — register `transformer: (data) => ({...})` to add or reshape fields.
6. **`overrideGQLOperations`** — extend the GraphQL fragment with backend fields at build time:

   ```javascript
   import { overrideGQLOperations } from '@dropins/build-tools/gql-extend.js';

   overrideGQLOperations([
     {
       npm: '@dropins/storefront-pdp',
       operations: [`fragment PRODUCT_FRAGMENT on ProductView { custom_field }`],
     },
   ]);
   ```

   Each drop-in publishes the named fragments it exposes. The replacement fragment must match the name exactly.

## Creating new drop-ins

For genuinely new commerce surfaces (gift registry, loyalty dashboard, store locator), build on the SDK:

- Scaffold: `@dropins/elsie` (the SDK CLI) generates a new drop-in skeleton with `VComponent`, `render()`, slots, dictionary, events.
- Lifecycle: implement `initialize()`, expose `containers`, register events, define `langDefinitions`.
- Distribution: publish as `@dropins/storefront-<your-feature>` (or private scope).

Custom drop-ins inherit:
- Design tokens.
- Event bus + GraphQL fetcher.
- Initializer lifecycle.
- Recaptcha integration.

But you own the upgrade path forever. Choose this only when nothing existing maps to your domain.

## Source URLs

- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/introduction/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/extend-or-create/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/quick-start/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/commerce-blocks/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/styling/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/branding/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/labeling/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/linking/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/dictionaries/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/slots/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/layouts/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/events/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/common-events/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/extending/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/all/creating/
