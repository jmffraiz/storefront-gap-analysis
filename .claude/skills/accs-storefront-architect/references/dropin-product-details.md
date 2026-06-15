# Product Details (PDP) Drop-In Reference

`@dropins/storefront-pdp` (v3.0.2): renders the product detail page surface — gallery, header, price, options/swatches, quantity, attributes, descriptions, gift-card and downloadable options.
Modern architecture is **modular per-area containers** (mount each independently). The legacy monolithic `ProductDetails` container is deprecated.

## Purpose & responsibilities

The PDP drop-in renders:
- Product gallery (carousel, thumbnails, zoom, video).
- Header (name, brand, SKU, variant labels).
- Price (regular, final, range, savings).
- Options (configurable swatches: color/size/image/text).
- Quantity stepper.
- Short description, full description, custom attributes.
- Downloadable links + gift-card options when applicable.

The host owns:
- "Add to Cart" / "Notify Me" CTAs (NOT a slot).
- Wishlist toggle (separate drop-in).
- Recommendations rails (separate drop-in).
- SEO metadata (host's `metadata.js`).
- URL routing.

Render is client-side only. For SEO, host either renders prerendered HTML (via AEM Commerce Prerender) or emits structured data from build-time metadata.

## Initialization

Package: `@dropins/storefront-pdp`. Initializer source: `scripts/initializers/pdp.js`.

```javascript
import { initializers } from '@dropins/tools/initializer.js';
import { initialize, setEndpoint, setFetchGraphQlHeaders } from '@dropins/storefront-pdp/api.js';

setEndpoint(await getConfigValue('commerce-endpoint'));     // Catalog Service
setFetchGraphQlHeaders(await getHeaders('cs'));

await initializers.mountImmediately(initialize, {
  sku: getProductSku(),                          // from <meta name="sku">
  langDefinitions: { en_US: { PDP: { /* ... */ } } },
  models: { /* transformers */ },
  scope: 'main',                                 // for multiple PDPs on one page (e.g., quick view)
  defaultLocale: 'en-US',
  globalLocale: 'en-US',
  acdl: true,                                    // push to Adobe Client Data Layer
  anchors: { /* deep-link targets */ },
  persistURLParams: true,
  preselectFirstOption: false,
  optionsUIDs: [],                               // pre-selected option UIDs
});
```

**Critical**: read SKU from `<meta name="sku">` rather than the URL slug. URL parsing lower-cases the SKU and breaks case-sensitive backends. AEM Commerce Prerender writes this meta tag automatically.

## Functions

From `@dropins/storefront-pdp/api.js`:

```typescript
fetchProductData(sku: string): Promise<ProductModel | null>
getFetchedProductData(): ProductModel | null
getProductConfigurationValues(): ValuesModel
getProductData(sku: string): Promise<ProductModel | null>
getProductsData(skus: string[]): Promise<ProductModel[]>
getRefinedProduct(sku: string, optionsUIDs: string[]): Promise<ProductModel | null>
isProductConfigurationValid(): boolean
setProductConfigurationValid(valid: boolean): void
setProductConfigurationValues(values: ValuesModel): void
```

### Notes

- `fetchProductData` issues `products(skus: [sku])` against Catalog Service and caches the response.
- `getProductData` is the public read accessor.
- `getRefinedProduct(sku, optionsUIDs)` resolves a variant when the shopper selects swatches. Drop-in calls this automatically on selection.
- `setProductConfigurationValues` mutates the in-memory state and emits `pdp/values`.

## Events

Bus: `@dropins/tools/event-bus.js`.

| Event | Direction | Payload | Fires when |
|-------|-----------|---------|------------|
| `pdp/data` | emit + listen | `ProductModel \| null` | Product loaded or refined |
| `pdp/setValues` | listen | `ValuesModel` | External component requests value change |
| `pdp/valid` | emit | `boolean` | Configuration validity changes |
| `pdp/values` | emit + listen | `ValuesModel` | Selected option UIDs / quantity changed |

`ValuesModel` shape: `{ sku, quantity, optionsUIDs: string[], errors: ... }`.

## Slots

Modular containers expose limited slots. Primary CTAs (Add to Cart, Notify Me) are NOT slots — host owns them.

| Container | Slot | Purpose |
|-----------|------|---------|
| `ProductAttributes` | `Attributes` | Custom attribute rendering |
| `ProductGallery` | `CarouselThumbnail` | Custom thumbnail rendering |
| `ProductGallery` | `CarouselMainImage` | Custom main image rendering |
| `ProductOptions` | `Swatches` | Custom swatch group |
| `ProductOptions` | `SwatchImage` | Custom image swatch |

## Containers

All accept `scope` prop (for multi-PDP-per-page like quick view). Render via `provider.render(Container, props)(element)`.

### ProductHeader

Product name, SKU, brand, variant name.

| Prop | Type | Purpose |
|------|------|---------|
| `scope` | string | Multi-PDP isolation |
| `showVariantSku` | boolean | Show resolved variant SKU |
| `showVariantName` | boolean | Show variant name (triggers `getRefinedProduct`) |

### ProductPrice

Price (regular, final), range for ComplexProductView, savings.

| Prop | Type | Purpose |
|------|------|---------|
| `scope` | string | Multi-PDP isolation |

### ProductGallery

Carousel + thumbnails + zoom + video.

| Prop | Type | Purpose |
|------|------|---------|
| `scope` | string | Multi-PDP isolation |
| `controls` | object | Show/hide chevrons, indicators |
| `zoom` | boolean / object | Enable image zoom; configure on-hover or click |
| `videos` | boolean | Enable inline video |
| `imageParams` | object | `width`, `height` for CDN transform |
| `loop` | boolean | Carousel loops |
| `slidesPerView` | number | Visible thumbs |

The only container with rich props.

### ProductOptions

Configurable option groups (swatches / dropdowns / radio).

| Prop | Type | Purpose |
|------|------|---------|
| `scope` | string | Multi-PDP isolation |
| `onValues` | callback | Fired on selection change with new values |
| `onErrors` | callback | Fired on validation errors |
| `transformer` | function | Transform options before render |

The only container with callbacks.

### ProductAttributes

Custom attribute key/value list.

### ProductDescription / ProductShortDescription

Rich-text descriptions.

### ProductQuantity

Quantity stepper.

### ProductDownloadableOptions

Downloadable links selector (when product is downloadable type).

### ProductGiftCardOptions / ProductGiftcardOptions

Gift card forms (sender, recipient, message, amount).

### ProductDetails (DEPRECATED)

Single monolithic container — kept for backward compat. Migrate to per-area containers.

## Styling

BEM class roots: `.pdp-product`, `.pdp-carousel`, `.pdp-price`, `.pdp-swatches`, `.pdp-gift-card-options`.

State modifiers: `--hidden`, `--active`, `--disabled`, `--out-of-stock`.

Override in `blocks/product-details/product-details.css`. Use design tokens — never hard-coded values.

## Dictionary keys

~30 keys under the `PDP.*` namespace. Override at init time via `langDefinitions`:

```javascript
langDefinitions: {
  en_US: {
    PDP: {
      header: { title: 'Product' },
      price: { from: 'From' },
      options: { selectColor: 'Select Color' },
      quantity: { label: 'Qty' },
      // ...
    },
  },
}
```

## Notify Me CTA pattern

Host-owned button driven by three eager subscriptions:

```javascript
import { events } from '@dropins/tools/event-bus.js';

let buttonInstance;

events.on('pdp/data', updatePrimaryCTA, { eager: true });
events.on('pdp/valid', updatePrimaryCTA, { eager: true });
events.on('cart/data', updatePrimaryCTA, { eager: true });

function updatePrimaryCTA() {
  const product = events.lastPayload('pdp/data');
  const valid = events.lastPayload('pdp/valid');
  if (!product) return;
  if (product.inStock && valid) {
    buttonInstance.setProps({ children: 'Add to Cart', disabled: false, onClick: addToCart });
  } else {
    buttonInstance.setProps({ children: 'Notify Me', disabled: false, onClick: notifyMe });
  }
}
```

Centralize state-driven button updates in one function. Subscriptions are `eager` so a late-mounting CTA hydrates correctly.

## Architectural notes

### Client-side render only

The drop-in does not SSR. SEO is the host's responsibility — emit metadata in `<head>` via build-time generation or AEM Commerce Prerender.

### Cart integration

The PDP drop-in does not call `addProductsToCart`. The host listens to `pdp/data` + `pdp/values` and calls the Cart drop-in's `addProductsToCart({sku, parentSku, quantity, optionsUIDs})`.

### Scope for multi-PDP per page

`scope: 'quick-view'` (any string) isolates a second PDP instance from the main PDP. Each instance has its own event subscriptions, model cache, and slot wiring. Useful for quick-view modals, comparison views.

### SEO caveats

- `<meta name="sku">` from prerender → drop-in reads it on init.
- JSON-LD must be in the prerendered HTML (PDP drop-in itself doesn't emit structured data).
- Open Graph tags must be in the prerendered HTML.

### Performance

- Gallery thumbnails are lazy.
- Built-in skeleton during data fetch.
- Eager subscriptions to `pdp/data` from host CTAs save a render tick on initial load.
- `imageParams: { width, height }` drives the CDN transform — set them tight for above-the-fold images.

## Source URLs

- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/quick-start/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/initialization/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/styles/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/functions/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/slots/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/events/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/dictionary/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/tutorials/notify-me-cta/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-attributes/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-description/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-downloadable-options/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-gallery/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-gift-card-options/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-giftcard-options/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-header/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-options/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-price/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-quantity/
- https://experienceleague.adobe.com/developer/commerce/storefront/dropins/product-details/containers/product-short-description/
