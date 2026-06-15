+-------------------+
| Metadata          |
+----------+--------+
| Template | guides |
+----------+--------+
| Category | Build  |
+----------+--------+

+-------------------+
| Section Metadata  |
+---------+---------+
| style   | content |
+---------+---------+

![][image0]

# Web Performance, Keeping your Lighthouse Score 100.

The quality of the experience of websites is crucial to achieving the business goals of your website and the satisfaction of your visitors.

Adobe Experience Manager (AEM) is optimized to deliver excellent experiences and optimal web performance. With the [Real Use Monitoring (RUM)](https://www.aem.live/docs/rum) operations data collection, information is continuously collected from field use and offers a way to iterate on real use performance measurements without having to wait for the CRuX data to show effects of code and deployment changes. It is common for field data collected in RUM to deviate from the lab results, as the network, geo-location and processing power for real devices are much more diverse than the simulated conditions in a lab.

The [Google PageSpeed Insight Service](https://pagespeed.web.dev/) is proven to be a great lab measurement tool. It can be used to avoid the slow deterioration of the performance and experience score of your website.

## Server-sided vs. Client-sided rendering

All canonical content on a page is rendered into markup on the server. Decorations of the content via CSS and DOM only affect display and refined, specialized semantics for accessibility and beyond.\
Client-sided rendering (as in fetch JSON) and render significant portions of the page on the client are only used in situations where there is no canonical content of the page (e.g. blocks that list other pages, applications, etc.)

Redundant content that is semantically not a part of the canonical content of the page, is not included in the markup for performance considerations (slowing down `LCP`, introducing unnecessary blocking time measured via `TBT` for `INP` by proxy), this includes headers and footers, and fragments that are used redundantly on a large number of pages.

## Core Web Vitals (CWV) and Lighthouse via PageSpeed Insights.

The performance of a website impacts its rankings in search results, as well as actual end-user performance is reflected by the [Core Web Vitals](https://web.dev/explore/learn-core-web-vitals) (CWV) in the crUX report. CWV is the ultimate arbiter of what good vs. bad web performance (and technical user experience) is for the visitors to your website.\
\
CWV as metrics collected in the real-world (field data) are as much a function of the code, the network setup as well as the devices your visitors use. With PageSpeed insights, Google offers an isolated service that runs Google's Lighthouse scores, in a standardized configuration selected by Google based on global distribution of mobile and desktop devices.\
\
Lighthouse (LH) scores via PageSpeed Insights provide a valuable and reliable proxy in a lab environment that allows you to make relative assertions about changes in your code. An improved Lighthouse score on `LCP` and `CLS` will yield an improved CWV score. Conversely a worse or unchanged LH score will very likely have the corresponding effect on CWV.

\
To avoid using a project specific configuration for Lighthouse testing, we use the continuously-updated configurations seen as part of the mobile and desktop strategies referenced in the latest versions of the Google [PageSpeed Insights API](https://developers.google.com/speed/docs/insights/rest/v5/pagespeedapi/runpagespeed).

While there may be additional insight that some developers feel they can collect from other ways of measuring Lighthouse scores, to be able to have a meaningful and comparable performance conversation across projects, there needs to be a way to measure performance universally. The default PageSpeed Insight Service is the most authoritative, most widely accepted lab test when it comes to measuring your performance.\
\
However it is important to remember that the recommendations that you get from PageSpeed Insights do not necessarily lead to better results, especially the closer you get to a lighthouse score of `100`.\
\
Core Web Vitals (CWV) collected by the built-in [Operational Telemetry data](https://www.aem.live/docs/operational-telemetry) plays an important role in validating results quickly in the field. For minor changes, however, the variance of the results and the lack of sufficient data points (traffic) over a short period of time makes it impractical to get statistically relevant results in most cases.

## Getting Started with Web Performance

When you start your project with the Boilerplate as in the [Developer Tutorial](https://main--helix-website--adobe.hlx.page/developer/tutorial), you will get a very stable Lighthouse score on PageSpeed Insight for both Mobile and Desktop at `100`. On every component of the lighthouse score there is some buffer for the project code to consume and still be within the boundaries of a perfect `100` lighthouse score.

![][image1]

## Testing Your Pull Requests

It turns out that it is hard to improve your Lighthouse score once it is low, but it is not hard to keep it at `100` if you continuously test.

When you open a pull request (PR) on a project, the test URLs in the description of your project are used to run the PageSpeed Insights Service against. The AEM GitHub bot will automatically fail your PR if the score is below `100` with a little bit of buffer to account for some volatility of the results.

![][image2]

The results are for the mobile lighthouse score, as they tend to be harder to achieve than desktop.

## Three-Phase Loading (E-L-D)

Dissecting the payload that's on a web page into three phases makes it relatively straight-forward to achieve a clean lighthouse score and therefore set a baseline for a great customer experience.

The three phase loading approach divides the payload and execution of the page into three phases

- **Phase E (Eager):** This contains everything that's needed to get to the largest contentful paint (`LCP`).
- **Phase L (Lazy):** This contains everything that is controlled by the project and largely served from the same origin.
- **Phase D (Delayed):** This contains everything else such as third-party tags or assets that are not material to experience.

### Phase E: Eager

Before anything happens, it is important to note that the body must be hidden (with `display:none`) to make sure no images start downloading and to avoid initial `CLS`.

In the **eager** phase, the first action is to “decorate” the `DOM`: the loading sequence makes few adjustments, mainly adds CSS classes to icons, buttons, blocks and sections and creates the auto-blocks. See the [Markup, Sections, Blocks, and Auto Blocking](https://www.aem.live/developer/markup-sections-blocks) page for more details on the resulting markup.

The body can then already be displayed, considering that sections are not loaded yet and remain hidden.

Then, the **full first section** is loaded with a priority given to the first image of this section, the “`LCP` candidate”. In theory, the fewer blocks the first section has, the faster `LCP` can be loaded.

Once the `LCP` candidate and all blocks of the section are loaded, the first section can be displayed and the fonts can start loading asynchronously.

This ends the **eager** phase.

#### LCP

In general, the `LCP` is the “hero“ image displayed at the top of a page. It is crucial to make sure this image is loaded and displayed as soon as possible in the loading sequence (see the **Eager** phase).

Everything that's needed to be loaded for the true `LCP` to be displayed must be loaded. In a project, this usually consists of the markup, the CSS styles and JavaScript files.

In many cases the `LCP` element is contained in a block, where the block `.js` and and `.css` also have to be loaded.

It is a good rule of thumb to keep the aggregate payload before the `LCP` is displayed below 100kb, which usually results in an `LCP` event quicker than 1560ms (`LCP` scoring at 100 in PSI). Especially on mobile the network tends to be bandwidth constrained, so changing the loading sequence before `LCP` has minimal to no impact.

Loading from or connecting to a second origin before the `LCP` occurred is strongly discouraged as establishing a second connection (TLS, DNS, etc.) adds a significant delay to the `LCP`.

There are situations where the actual `LCP` element is not included in the markup that is transmitted to the client. This happens when there is an indirection or lookup (for example a service that’s called, a fragment that’s loaded or a lookup that needs to happen in a `.json`) for the `LCP` element.\
In those situations, it is important that the page loading waits with guessing the `LCP` candidate (currently the first image on the page) until the first block has made the necessary changes to the DOM.

There are other situations where the content contains 2 hero images, one for desktop, one for mobile. Same as above, it is important to make sure that the correct image is considered as the `LCP` candidate and the “hero” block might need to be adjusted to remove the unnecessary image from the `DOM` (remove the desktop image on mobile devices or vice versa) to not load a bandwidth consuming image or even worse, load the unnecessary image first as the `LCP` candidate.

Finally, the `LCP` can be something other than an image, a video, a long text… For all those cases, a deep understanding of the loading sequence and how the `LCP` candidate is computed is necessary to make the correct optimizations.

### Phase L: Lazy

In the **lazy** phase, the portion of the payload is loaded that doesn't affect total blocking time (`TBT`) and ultimately first input delay (FID).

This includes things like loading the next sections and their blocks (JavaScript and CSS) as well as loading all the remaining images according to their `loading="lazy"` attribute and other JavaScript libraries that are not blocking. The lazy phase is generally everything that happens in the various `blocks` you are going to create to cover the project needs.

In this phase it would still be advisable that the bulk of the payload come from the same origin and is controlled by the first party, so that changes can be made if needed to avoid negative impact on `TBT`, `TTI` and `FID`.

### Phase D: Delayed

In the **delayed** phase, the parts of the payload are loaded that don't have an immediate impact to the experience and/or are not controlled by the project and come from third parties. Think of marketing tooling, consent management, extended analytics, chat/interaction modules etc. which are often deployed through tag management solutions.

It is important to understand that for the impact on the overall customer experience to be minimized, the start of this phase needs to be significantly delayed. The delayed phase should be at least three seconds after the `LCP` event to leave enough time for the rest of the experience to get settled.

The delayed phase is usually handled in `delayed.js` which serves as an initial catch-all for scripts that cause `TBT`. Ideally, the `TBT` problems are removed from the scripts in question either by loading them outside of the main thread (in a web worker) or by just removing the actual blocking time from the code. Once the problems are fixed, those libraries can easily be added to the lazy phase and be loaded earlier.

Ideally there is no blocking time in your scripts, which is sometimes hard to achieve as commonly used technology like tag managers or build tooling create large JavaScript files that are blocking as the browser is parsing them. From a performance perspective it is advisable to remove those techniques, make sure your individual scripts are not blocking and load them individually as separate smaller files.

### Header and Footer

The header and specifically the footer of the page are not in the critical path to the `LCP`, which is why they are loaded asynchronously in their respective blocks. Generally, resources that do not share the same life cycle (meaning that they are updated with authoring changes at different times) should be kept in separate documents to make the caching chain between the origins and the browser simpler and more effective. Keeping those resources separate increases cache hit ratios and reduces cache invalidation and cache management complexity.

### Fonts

Since web fonts are often a strain on bandwidth and loaded from a different origin via a font service like <https://fonts.adobe.com> or <https://fonts.google.com>, it is largely impossible to load fonts before the `LCP`, this is why they are loaded right after.

By default, the [AEM Boilerplate](https://github.com/adobe/aem-boilerplate) implements the [font fallback](https://www.aem.live/developer/font-fallback) technique to avoid `CLS` when the font is loaded. It would be counterproductive to preload the fonts (via Early hints, h2-push or markup) and largely impact the performances.

## Bonus: Speed is Green

Building websites that are fast, small, and quick to render is not just a good idea to deliver exceptional experiences that convert better, it is also a good way to reduce carbon emissions.

## Common Sources of Performance Issues

Over time, we gathered a collection of anti-patterns that negatively impact performance, and need to be avoided to be compliant with the best practices in this document.

### Early hints / h2-push / pre-connect are part of the network budget

Instinctively, it would make sense to tell the browser to download as much as possible and as early as possible, even before the markup processing even starts. But remember, the ultimate goal is to have a stable page to show to the visitor as quickly as possible. `LCP` timing is a good proxy for that.

As a rule of thumb, to get an `LCP` to `100` on Mobile with PageSpeed Insight the network constraints are set up in a way that there can only be a single host with a network payload that's not exceeding 100kb, as the setup is largely bandwidth constrained. Early hints, h2-push and pre-connect consume that bandwidth, by downloading resources that are not required for `LCP` and therefore negatively impact the performance, and have to be removed.

Contrary to popular belief adding `<link rel="preload" …>` or `fetchpriority="high"` does not improve `LCP` performance measured by PageSpeedInsights, but in turn has a significant negative impact.\
\
Compare:

<https://main--prefetch--aem-sandbox.aem.live/index.html>\
<https://main--prefetch--aem-sandbox.aem.live/index-preload.html>\
<https://main--prefetch--aem-sandbox.aem.live/index-priority.html>

\
…using [PageSpeedInsights](https://pagespeed.web.dev/) or [Deep PSI](https://tools.aem.live/tools/deep-psi/deep-psi.html) for comparisons.

### Redirects for paths resolution

If a visitor requests `www.domain.com` and gets redirected to `www.domain.com/en` and then to `www.domain.com/en/home,` they get a penalty for each redirect, i.e. performance is negatively impacted. This is mostly visible in Core Web Vitals measured via RUM or CrUX as lab results in PageSpeed Insights by default exclude redirect overhead from the lab test.

### CDN client scripts injection

Our markup but also our `.aem.page` and `.aem.live` origins are optimized for performance and we are extremely careful with any part of the payload, as well as the loading sequence for resources.

Some CDN vendors and configurations inject scripts that are consuming bandwidth and create blocking time, before `LCP` with negative impacts performance. Those scripts should be disabled, or loaded appropriately in the loading sequence after `LCP`.

A comparison of a `.aem.live` origin of the PageSpeed Insight report, with the corresponding site that's fronted by a customers CDN (e.g. production site) will show the negative impact produced by a CDN outside of AEM's control.

### CDN TTFB and Protocol Implementation Impact

Depending on the CDN vendor, there are differences in protocol implementations and performance characteristics for the pure delivery of the HTTP payload. Additional tooling like WAF and other network infrastructure upstream of AEM may also negatively impact performance.\
A comparison of a `.aem.live` origin of the PageSpeed Insight report, with the corresponding site that's fronted by a customers CDN (e.g. production site) will show the negative impact produced by a CDN outside of AEM's control.

+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Pagination (Contained)                                                                                                                                                                 |
+--------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------+
| :icon-arrow: Previous                                                          | Up Next :icon-arrow:                                                                                  |
|                                                                                |                                                                                                       |
| ### [Indexing](https://main--helix-website--adobe.hlx.page/developer/indexing) | ### [Markup - Sections](https://main--helix-website--adobe.hlx.page/developer/markup-sections-blocks) |
+--------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------+

[image0]: https://main--helix-website--adobe.aem.page/media_11f686b25ac62733f99b1d7769df42c7427c474f0.jpg#width=1103&height=828

[image1]: https://main--helix-website--adobe.aem.page/media_1c0a833642edcfbc157cedaea970bc574bff0f4bb.png#width=1280&height=1350

[image2]: https://main--helix-website--adobe.aem.page/media_1f574c73155e1f08836f830b8b7bdf42573afcde7.png#width=1600&height=422
