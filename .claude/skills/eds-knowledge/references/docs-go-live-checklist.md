+------------------------+
| Metadata               |
+----------+-------------+
| Template | guides      |
+----------+-------------+
| Category | Launch      |
+----------+-------------+
| Image    | ![][image0] |
+----------+-------------+

+-------------------+
| Section Metadata  |
+---------+---------+
| style   | content |
+---------+---------+

![][image0]

# Go-Live Checklist

The go-live checklist is a summary of best practices to consider when launching a website. These steps are generally good practices but have some aspects specific to Adobe Experience Manager.

## Steps Before Go-Live

### Content and Design QA

Make sure that your content and design conforms to the specifications and that you are happy with the website you see on your projects `.aem.live` domain. This may include checks for specific accessibility and SEO requirements of your project.

### Performance Validation

Every AEM project should produce a lighthouse score of 100 for mobile and desktop from [Google Pagespeed insights](https://pagespeed.web.dev/) on its respective `.aem.live` site.

See the document [Keeping it 100, Web Performance](https://www.hlx.live/developer/keeping-it-100) for more information.

### Analytics Validation

Make sure that all your analytics setup and the rest of your martech stack is firing as expected and visitor data is visible in your reporting dashboards.\
In any relaunch of a website the analytics instrumentation will change based on loading sequence and performance.

It is important to expect that the baseline of any metric captured by analytics will change. Contact the corresponding analysts to make sure that the adjustment of the baseline is understood and expected.

Metrics that may change their baselines as reported by analytics may include pageviews, conversion rates, bounce rates, time on page, etc. Depending on the change in loading patterns the baseline of the metrics may go up and down.\
Bottom of the funnel metrics like checkout, transactions or form submission that are captured by operational systems are not affected and are expected to stay flat past a lift-and-shift launch.

### RUM instrumentation

To be able to see performance impact quickly and reliably and to compare before / after launch metrics we recommend instrumenting your website before launch with [Real Use Monitoring](https://www.aem.live/docs/rum) (RUM), ideally as early as possible. [Adding RUM to your existing site is trivial](https://www.aem.live/developer/rum#how-to-add-rum-instrumentation-to-your-site) and can give you important operational insights even before launch.

### Legacy Redirects

In most migrations there are legacy URLs that are retired. Make sure those are reflected in your redirects spreadsheet (`redirects.xlsx` in sharepoint or `redirects` in google), found in your project content folder. Check Google Search Console for the most impactful backlinks (in terms of SEO) to create redirects for.

See the document [Redirects](https://www.hlx.live/docs/redirects) for more information.

### Sitemap & Robots

For most websites with a significant number of pages, a sitemap is desirable. AEM automatically generates sitemaps from the query index. For multilingual sites, adding hreflang to the sitemap ensures that the website correctly targets the appropriate geographic and language audience, which is essential for SEO and prevents issues like duplicate content across different language versions (aka SEO cannibalisation) and improves the search engine's ability to serve the right version of the content to the right users.

If you have a sitemap that’s generated for your site make sure it is discoverable from your `robots.txt`. ​​Note that `robots.txt` is (technically) case sensitive, and a good example is:

```
User-agent: *
Allow: /

Sitemap: https://<your-domain>/sitemap.xml
```

Note: `aem.page` and `aem.live` are kept hidden from crawlers intentionally, to avoid duplicate content. There is no need to set the `robots.txt` to `Disallow` crawlers during development.

It is important to understand that a `Disallow` in `robots.txt` is potentially helping with crawler budget issues under certain conditions, but does not prevent a page from being added to the google index, and conversely will make its removal via `noindex` impossible. URLs of pages that have a `Disallow` can still be discovered in a SERP.

An effective way to remove a URL from the index is to `Allow` crawlers and put a `noindex` `robots` tag on the page.

Adding a custom `robots.txt` is done via [config service](https://main--helix-website--adobe.aem.page/docs/config-service-setup#update-robotstxt).

See the documents [Indexing](https://www.hlx.live/developer/indexing) and [Sitemaps](https://www.hlx.live/developer/sitemap) for more information.

### Canonical URLs

Make sure canonical URLs return 2xx HTML response status code (not 3xx or 4xx) and that they are correctly implemented, which is crucial for preventing duplicate content issues across the site. Proper canonicalization helps search engines understand which versions of similar pages to index and display in search results, directly impacting SEO performance.

See the following external documentation for more information: [Consolidate duplicate urls](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls)

### Favicon

Adding a favicon to your site gives it a professional look in your visitor’s browsers.

See the document [Favicon](https://www.hlx.live/developer/favicon) for more information.

### Authentication for Authors

By default, authors don’t need to be logged in to use AEM Sidekick. If you decide you want to control who can preview and publish documents this can be configured.

See the document [Configuring Authentication for Authors](https://www.aem.live/docs/authentication-setup-authoring) for more information.

### SharePoint Access

If your content is in SharePoint, follow [this guide](https://www.aem.live/docs/setup-customer-sharepoint) to configure dedicated access which you control.

### CDN Configuration

One of the last steps in a go-live is usually to update your CDN to point to your `aem.live` endpoint.

You can either use your existing, self-managed CDN or an Adobe-managed CDN included in your license. [See here for details on supported CDN options](https://main--helix-website--adobe.aem.page/docs/byo-cdn-setup).

Ideally the CDN configuration is tested in a staging environment to make sure everything works as expected, which includes redirects from www to APEX and vice-versa.

### Push Invalidation Setup

Make sure push invalidation is properly set up according to the document [Configuring Push Invalidation for BYO Production CDN](https://www.hlx.live/docs/setup-byo-cdn-push-invalidation). Test the setup by publishing a small change and verifying that the change is visible on the production domain.

### Notify Engineering On-Call

The Edge Delivery Services team is closely monitoring all systems as part of our standard 24/7 operations. Multi-tenant incidents typically appear on [status.adobe.com](https://status.adobe.com). If we detect irregularities affecting a single tenant, we will reach out to the affected customer to help them address the problem.

By notifying us about your go-live, you can help us pay extra attention to your project during the go-live, and we can contact you via Teams or Slack more quickly in case of any anomalies we can assist with.

To notify us, please send an email to **aemgolives\@adobe.com** and include the following information:

- `aem.live` **URL(s)** that will be going live on Edge Delivery Services
- **Production domain**
- **Planned go-live date and time**
- **Primary contact person** for the go-live
- **Teams or Slack channel** for real-time collaboration with Engineering
  - If you do not yet have a collaboration channel set up yet, please refer to [Teams Collaboration](https://www.aem.live/docs/teams) for guidance.

## Post Go-Live Validation

### Performance Validation

Validate that the performance is still at a lighthouse score of 100 via [pagespeed insights](https://pagespeed.web.dev/) on the production environment. Introducing a CDN layer can have adverse performance effects that are usually visible on the protocol layer. Typical culprits are running HTTP/1.1 or ineffective origin caching as well as bot detection or other libraries injected by the CDN configuration.

### Google Search Console

If you have an active Google Search Console with your sitemap uploaded, it may be valuable to get a coverage report and make sure that indexing works as expected. The Google Search Console should be monitored for the weeks after a go-live to track the indexing status of new and updated pages, ensuring they are properly recognized by Google. It's crucial to check for total clicks, total impressions, backlinks changes and crawl errors, as these can significantly impact the site's SEO performance and authority.

### Martech and Analytics validation

Certain aspects of your martech stack may be tied to specific origins / hostnames, and operate differently on staging (or `.page` and `.live`) hostnames if not configured correctly. It is advisable to make sure that all the important tags in the martech stack fire correctly and the information is continuously collected after a go-live.

### 404 Report

After a website has been migrated there is usually a set of 404 Not Founds, which should be monitored after the go-live and redirected to popular page URLs. This information can be pulled from your site analytics and the respective Slack bot report. Monitoring this for the weeks after a go-live is recommended.

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Pagination (Contained)                                                                                                                                                |
+------------------------------------------------------------------------+----------------------------------------------------------------------------------------------+
| :icon-arrow: Previous                                                  | Up Next :icon-arrow:                                                                         |
|                                                                        |                                                                                              |
| ### [Launch](https://main--helix-website--adobe.hlx.page/docs/#launch) | ### [BYO CDN Setup Overview](https://main--helix-website--adobe.aem.page/docs/byo-cdn-setup) |
+------------------------------------------------------------------------+----------------------------------------------------------------------------------------------+

[image0]: https://main--helix-website--adobe.aem.page/media_1da4bd7d3a1161f686fa72258c51bd49249fa142a.png#width=1200&height=900
