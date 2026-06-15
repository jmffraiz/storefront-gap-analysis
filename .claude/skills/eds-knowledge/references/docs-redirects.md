+------------------------+
| Metadata               |
+----------+-------------+
| Template | guides      |
+----------+-------------+
| Image    | ![][image0] |
+----------+-------------+
| Category | launch      |
+----------+-------------+

+-------------------+
| Section Metadata  |
+---------+---------+
| style   | content |
+---------+---------+

![][image0]

# Redirects

Every website has the need for redirects. For example if you relocate or delete content, you want your users to still be able to find it or the next best thing. See the document [Authoring and Publishing Content](https://www.hlx.live/docs/authoring#delete) for more information on deleting content.

You can intuitively manage redirects as a spreadsheet called `redirects` (or `redirects.xlsx`) in the root of your project folder.

The spreadsheet has to contain at least two columns titled `Source` and `Destination`.

![][image1]

- The `Source` is relative to the domain of your website, so it only contains the relative path.
- The `Destination` can be either a fully qualified URL if you are redirecting to a different website, or it can be a relative path if you are redirecting within your own website.

After making changes to your redirects spreadsheet, you can preview your changes via the sidekick and have your stakeholders check that the redirects are working on your `.page` preview website before publishing the redirect changes to your production website. See the [Sidekick documentation](https://www.hlx.live/docs/sidekick#environment-switcher) for more information about switching between environments.\
\
Redirects take precedence over existing content, which means that if you have an existing page with a given URL, defining a redirect for that same URL will serve the redirect for that page and “hide” the existing page. Conversely if a redirect that has been set up on an existing page is removed, the existing page will be served again, unless the page was unpublished.

Remember that if your redirect workbook has multiple pages (worksheets), then the redirects will only work on the sheet that is called `helix-default`. This allows you to manage more complex redirects through spreadsheet formulas. The [spreadsheets and JSON documentation page](https://main--helix-website--adobe.hlx.page/developer/spreadsheets) has all the details.

**Wildcards Redirects (CDN level)**

Wildcard redirects have a set of issues around (a) unmanaged complexity and accumulate tech debt over time, (b) introducing potential redirect loops and also tend to (c)  `301` into `404`. For those reasons we generally recommend avoiding patterns-based wildcard redirects, and instead create redirects based on actual "usage" data. For this reason, they **are not supported** via the `redirects` sheet.

However there are some cases where individually managed redirects are inflating the list of redirects, and for practical or perceived initial simplicity reasons it may be compelling to use pattern based redirects. For example, you might want to apply a redirect to all pages under a specific path, regardless of the exact URL.

This is where using wildcards **in your CDN** can be helpful. Wildcards allow you to match multiple URLs under a common path, simplifying the redirection of entire sections of your site.

**Example:** If you want to redirect all URLs under `/old-path/` to `/new-path/`, including any subpages (e.g., `/old-path/page1`, `/old-path/page2`), you can configure a wildcard redirect in your CDN.

- Input URL pattern: `/old-path/*`
- Redirect to: `/new-path/$1`

The wildcard (`*`) captures anything after `/old-path/`, and `$1` represents the captured part of the URL, ensuring the structure is maintained in the new location.

Keep in mind that redirecting this way usually redirects the entire namespace and hence creates redirects (`301`) into `404` for an infinite number of URLs and may therefore lead to undesirable results.

**Note:** The specifics of configuring these redirects depend on the CDN provider you are using. Different vendors may have their own syntax, interface, and capabilities for handling wildcards and advanced redirect rules. Always consult the documentation of your specific CDN for guidance on how to implement wildcard redirects.

## Site Migrations and SEO

When migrating your site to AEM Edge Delivery Services, this may necessitate some changes to the URLs your site uses. This can have an impact on SEO, so it is important that you plan for this carefully to avoid any disruption. We recommend the following steps to handle the most common scenarios:

- Add your most popular URLs to the redirects sheet to ensure these are handled properly via 301 redirects
- If the change to URLs follows a common pattern, for example, removing `.html` from all urls, set this up as a wildcard redirect at your CDN
- If the change includes more complex situations than a common pattern, best practice is to map the old and new URLs 1-to-1.
- If you are pruning content as part of your migration, avoid the temptation to redirect these urls to a common destination such as your home page; See the “Avoid irrelevant redirects” note in Google's [documentation on site moves](https://developers.google.com/search/docs/crawling-indexing/site-move-with-url-changes#start-site-move) for more information
- Ensure your [sitemap](https://www.aem.live/developer/sitemap) is up to date
- After go-live, [monitor 404s via Operational Telemetry](https://www.aem.live/docs/operational-telemetry) and add missing redirects as needed

For more information, see Google's [documentation on site moves](https://developers.google.com/search/docs/crawling-indexing/site-move-with-url-changes).

+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Pagination (Contained)                                                                                                                                      |
+------------------------------------------------------------------------------+------------------------------------------------------------------------------+
| :icon-arrow: Previous                                                        | Up Next :icon-arrow:                                                         |
|                                                                              |                                                                              |
| ### [Favicon](https://main--helix-website--adobe.hlx.page/developer/favicon) | ### [Resources](https://main--helix-website--adobe.hlx.page/docs/#resources) |
+------------------------------------------------------------------------------+------------------------------------------------------------------------------+

[image0]: https://main--helix-website--adobe.aem.page/media_11f1e812b5708f947436b2cd918bcec9cf5e2d6b7.jpg#width=1103&height=828

[image1]: https://main--helix-website--adobe.aem.page/media_12885b8790f1f9a84da8ff0af200da6e5f34983d1.png#width=1600&height=422
