# Drop-in SDK Reference

`@dropins/tools` (the shared SDK) + `@dropins/storefront-elsie` (the UI primitives library — Adobe's design-system components). Reference for building custom drop-ins, theming with design tokens, and using SDK utilities.

## SDK fundamentals

The SDK provides framework-agnostic primitives. Drop-ins (`@dropins/storefront-*`) build on top of it. Custom drop-ins should too.

**Core packages:**
- `@dropins/tools/event-bus.js` — pub/sub for cross-drop-in state.
- `@dropins/tools/fetch-graphql.js` — Commerce GraphQL fetcher with header injection.
- `@dropins/tools/initializer.js` — `initializers.mountImmediately` + `register` lifecycle.
- `@dropins/tools/recaptcha.js` — reCAPTCHA v3 / Enterprise integration.
- `@dropins/tools/lib/aem/assets.js` — AEM Assets image helpers.
- `@dropins/tools/lib/aem.js` — `rootLink`, `decorateLinks`, locale handling.
- `@dropins/storefront-elsie` — UI component library (`Button`, `Card`, `Modal`, `Picker`, `Field`, `Icon`, etc.).
- `@dropins/build-tools/gql-extend.js` — `overrideGQLOperations` for build-time GraphQL fragment extension.

**CLI:**
- `@dropins/elsie` — scaffolds new drop-ins (`elsie new my-feature`).

## Reference primitives

### `events` (event-bus.js)

```typescript
events.on<T>(name: string, handler: (payload: T) => void, options?: { eager?: boolean; scope?: string }): Subscription
events.emit<T>(name: string, payload: T, options?: { scope?: string }): void
events.lastPayload<T>(name: string, options?: { scope?: string }): T | undefined
events.enableLogger(enabled: boolean): void
```

`Subscription` exposes `.off()` to unsubscribe.

### `fetchGraphQl` (fetch-graphql.js)

```typescript
fetchGraphQl(query: string, options?: {
  method?: 'GET' | 'POST';
  cache?: RequestCache;
  signal?: AbortSignal;
  variables?: Record<string, unknown>;
  headers?: Record<string, string>;
  endpoint?: string;
}): Promise<{ data: unknown; errors?: GraphQlError[] }>
```

Module configuration:
- `setEndpoint(url)`.
- `setFetchGraphQlHeader(name, value)`.
- `setFetchGraphQlHeaders(headers)`.
- `removeFetchGraphQlHeader(name)`.

### `initializers` (initializer.js)

```typescript
initializers.mountImmediately(initialize, config): Promise<void>
initializers.register(initialize, config): void
initializers.setImageParamKeys(params): void
initializers.setGlobalLocale(locale): void
```

`mountImmediately` runs the initializer now. `register` defers until something explicitly triggers it.

`setImageParamKeys` maps logical params (`width`, `height`, `auto`, `quality`, `crop`, `fit`) to CDN-specific query keys. Defaults tuned for Fastly:

```javascript
initializers.setImageParamKeys({
  width: 'width',
  height: 'height',
  // or transform: width: (val) => ['w', val * 2]  for HD displays
});
```

`setGlobalLocale('fr-FR')` sets the locale for all drop-ins. Resolution precedence: component prop → `setGlobalLocale()` → `navigator.language` → `'en-US'`.

### `render` (per drop-in)

```typescript
provider.render(Container, props): (element: HTMLElement) => Promise<void>
```

Curried — first call configures, second mounts.

### `VComponent`

The SDK's underlying renderable. Drop-in containers extend `VComponent`. When building custom drop-ins, your containers do too.

### `Slot`

The slot primitive — its context exposes `appendChild`, `replaceWith`, `onChange`, plus container-specific data.

### `recaptcha.js`

```typescript
setEndpoint(endpoint: string): void
setConfig({ siteKey: string; type: 'v3' | 'enterprise' }): void
initReCaptcha(): Promise<void>
verifyReCaptcha(action: string): Promise<string>  // returns token
```

Call order: `setEndpoint` → `setConfig` → `initReCaptcha` → `verifyReCaptcha(action)`.

### `rootLink` and `decorateLinks`

```typescript
rootLink(path: string): string                       // prefixes with active locale folder
decorateLinks(element: HTMLElement): void            // rewrites all <a href="/..."> in subtree
```

## Design tokens

CSS custom properties defined in `@dropins/tools/design-tokens.css`. Override at `:root` in `styles/styles.css` for global theming.

### Token families

**Colors:**
- Neutrals: `--color-neutral-50`, `--color-neutral-100` … `--color-neutral-900`, `--color-neutral-950`.
- Brand: `--color-brand-50` … `--color-brand-900`.
- Semantic: `--color-positive-500`, `--color-negative-500`, `--color-warning-500`, `--color-info-500` (with 50–900 scales each).
- Background: `--color-background-default`, `--color-background-emphasis`, `--color-background-subtle`.
- Text: `--color-text-default`, `--color-text-subtle`, `--color-text-inverse`.

**Spacing:**
- `--spacing-xxsmall` ≈ 4px.
- `--spacing-xsmall` ≈ 8px.
- `--spacing-small` ≈ 12px.
- `--spacing-medium` ≈ 16px.
- `--spacing-large` ≈ 24px.
- `--spacing-xlarge` ≈ 32px.
- `--spacing-xxlarge` ≈ 48px.

**Typography:**
- `--type-headline-1-font-family`, `--type-headline-1-font-size`, `--type-headline-1-line-height`, `--type-headline-1-font-weight`.
- Same set for `headline-2` … `headline-6`, `body-1`, `body-2`, `caption`, `code`.
- Default font: `--type-headline-1-default-font`, `--type-body-1-default-font`.

**Shapes (border-radius):**
- `--shape-default-radius`, `--shape-medium-radius`, `--shape-large-radius`, `--shape-pill-radius`.

**Grid:**
- `--grid-columns-mobile`, `--grid-columns-tablet`, `--grid-columns-desktop`.
- `--grid-gap-small`, `--grid-gap-medium`, `--grid-gap-large`.
- `--grid-max-width`.

> Exact token values vary per SDK release. Inspect `@dropins/tools/design-tokens.css` in your `node_modules` (or import it and read the CSSOM) for the authoritative list.

## Components catalog (Elsie SDK)

`@dropins/storefront-elsie` — accessible, themable UI primitives used by all drop-in containers. Available for custom blocks too.

| Component | Purpose |
|-----------|---------|
| `Accordion` | Collapsible sections. |
| `ActionButton` | Icon + label button for inline actions. |
| `ActionButtonGroup` | Set of `ActionButton`s with shared styling. |
| `AlertBanner` | Banner with semantic variant (positive / negative / warning / info). |
| `Breadcrumbs` | Crumb trail. |
| `Button` | Primary CTA — variants `primary`, `secondary`, `tertiary`, `destructive`. |
| `Card` | Bordered container. |
| `CartItem` | Standard cart line representation. |
| `CartList` | List of `CartItem`s. |
| `Checkbox` | Form checkbox with label. |
| `ColorSwatch` | Selectable color circle (for PDP options). |
| `ContentGrid` | Responsive grid for product cards / cms tiles. |
| `Divider` | Horizontal rule. |
| `Field` | Wraps an input with label, help text, error. |
| `Header` | Section header with title + actions. |
| `Icon` | Inline icon (named library). |
| `IllustratedMessage` | Empty-state visual + message. |
| `Image` | Optimized image with width/height transforms. |
| `ImageSwatch` | Selectable image swatch. |
| `Incrementer` | Quantity stepper (+/−). |
| `InlineAlert` | Compact alert inside forms. |
| `Input` | Text input. |
| `InputDate` | Date input. |
| `InputFile` | File picker. |
| `InputPassword` | Password input with show/hide. |
| `Modal` | Dialog with backdrop, focus trap, ESC handling. |
| `Pagination` | Page numbers + prev/next. |
| `Picker` | Searchable single-select. |
| `Portal` | Renders children into a different DOM node. |
| `Price` | Money formatting with currency. |
| `PriceRange` | Range display (from-to). |
| `ProductItemCard` | Product card for grids. |
| `ProgressSpinner` | Loading indicator. |
| `RadioButton` | Form radio. |
| `Skeleton` | Loading skeleton box. |
| `Tag` | Pill tag (filter chips, status). |
| `TextArea` | Multi-line text input. |
| `TextSwatch` | Text-only swatch (sizes). |
| `ToggleButton` | Two-state button. |

### Component usage pattern

```javascript
import { Button } from '@dropins/storefront-elsie/Button.js';
import { render } from '@dropins/storefront-elsie/render.js';

const instance = render(Button, {
  children: 'Add to Cart',
  variant: 'primary',
  size: 'medium',
  onClick: () => addToCart(sku),
})(buttonContainer);

// Later, update props imperatively:
instance.setProps({ children: 'Added!', disabled: true });
```

Notify Me CTA pattern (PDP): hold the instance ref, update props on state-bus events.

## Utilities (`@dropins/tools/utilities/*`)

| Utility | Signature | Purpose |
|---------|-----------|---------|
| `classList` | `classList(...args)` | Conditional class string composer (like `clsx`). |
| `debounce` | `debounce(fn, ms)` | Standard debounce. |
| `deepmerge` | `deepmerge(target, ...sources)` | Recursive object merge. |
| `getCookie` | `getCookie(name)` | Read a cookie by name. |
| `getFormErrors` | `getFormErrors(formEl)` | Collect HTML5 form validation errors. |
| `getFormValues` | `getFormValues(formEl)` | Collect form values as an object. |
| `getPathValue` | `getPathValue(obj, 'a.b.c')` | Read a nested path. |

## CLI (`@dropins/elsie`)

```bash
# Scaffold a new drop-in
npx @dropins/elsie new my-feature

# Generate a new container in an existing drop-in
npx @dropins/elsie generate container MyContainer

# Validate a drop-in's design-token compliance
npx @dropins/elsie audit
```

## Architectural notes

### When to build with SDK vs raw DOM

- **Use SDK components** for forms, modals, alerts, pickers, grids, cards, anywhere you need design-token-themed UI consistent with Adobe drop-ins.
- **Use raw DOM** for layout structure inside a block (the block's own scaffolding) — no point wrapping a `<div>` flex container in a component.

### Theming approach

Set design tokens once at `:root` in `styles/styles.css`. All drop-ins inherit. Drop-in-specific BEM classes can scope variants (e.g., `.pdp-product { --spacing-medium: 20px; }`).

### Performance budget for custom drop-ins

A custom drop-in should:
- Lazy-load below-the-fold containers.
- Use the Elsie `Skeleton` component during data fetch.
- Cap event subscriptions per container — eager-subscribe once per surface, not per slot.
- Share GraphQL fetches across containers in the same drop-in (use a module-level cache).
- Stay within ~50KB gzipped per drop-in to keep main bundle lean.

### Custom drop-in scaffolding

```bash
npx @dropins/elsie new storefront-loyalty
cd storefront-loyalty
npm install
# Edit src/ — initializer, containers, models, GraphQL queries
npm run build
npm publish      # or npm link for dev
```

Then in the boilerplate's `package.json`:
```json
{ "dependencies": { "@dropins/storefront-loyalty": "^1.0.0" } }
```

And in `scripts/initializers/loyalty.js`:
```javascript
import { initializers } from '@dropins/tools/initializer.js';
import { initialize, setEndpoint } from '@dropins/storefront-loyalty/api.js';

setEndpoint(await getConfigValue('commerce-core-endpoint'));
await initializers.mountImmediately(initialize, { /* ... */ });
```

### GraphQL extension via `overrideGQLOperations`

When a drop-in's GraphQL fragment doesn't include a field you need, extend it at build time:

```javascript
// build.mjs
import { overrideGQLOperations } from '@dropins/build-tools/gql-extend.js';

overrideGQLOperations([
  {
    npm: '@dropins/storefront-pdp',
    operations: [
      `fragment PRODUCT_FRAGMENT on ProductView {
        custom_attribute_xyz
      }`,
    ],
  },
]);
```

The fragment name must match the drop-in's exposed fragment exactly (check the drop-in's documentation for the list).

### Verified vs. inferred design tokens

The Experience League SDK pages are JS-rendered and may not yield the exact token names verbatim via static fetch. The authoritative source is `@dropins/tools/design-tokens.css` in your `node_modules` after `npm install`. Confirm token names from there when authoring CSS overrides.

## Source URLs

- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/get-started/cli/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/events/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/graphql/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/initializer/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/links/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/render/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/recaptcha/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/slots/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/reference/vcomponent/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/design/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/components/overview/
- https://experienceleague.adobe.com/developer/commerce/storefront/sdk/utilities/
