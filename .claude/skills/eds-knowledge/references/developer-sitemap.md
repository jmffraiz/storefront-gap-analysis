+------------------------+
| Metadata               |
+----------+-------------+
| Template | guides      |
+----------+-------------+
| Image    | ![][image0] |
+----------+-------------+
| Category | publish     |
+----------+-------------+

+-------------------+
| Section Metadata  |
+---------+---------+
| style   | content |
+---------+---------+

![][image0]

# Sitemaps

Create automatically generated sitemap files to be referenced from your `robots.txt`. This helps with SEO and the discovery of new content. AEM can generate three types of sitemaps: without any configuration, based solely on a query index or based on a manual sitemap configuration. A single sitemap must be limited to 50,000 URLs and 50MB (uncompressed) in size - see [Limits](https://main--helix-website--adobe.aem.page/docs/limits#sitemap-limits).

## Creating a Sitemap without any configuration

If you don’t do anything you will see your sitemap in `sitemap.xml` and have a sitemap index in `sitemap.json`. It will contain a list of all your published documents.

If you started with another type of sitemap and would like to switch to this type, you’ll have to delete the `helix-sitemap.yaml` configuration file - either manually defined in GitHub or automatically generated - and reindex your site.

### Domain name used in external URLs

To customize the domain used in creating external URLs, add a property named `cdn.prod.host` in your project configuration.

- If you are using the configuration service, [see here how to update the site configuration](https://www.aem.live/docs/config-service-setup#update-production-cdn).
- Otherwise, [see here for document-based configuration](https://main--helix-website--adobe.aem.page/docs/configuration).

## Generating a Sitemap configuration based on an index

Please see the document [Indexing](https://main--helix-website--adobe.hlx.page/developer/indexing) to learn more about indexing. In order to generate a sitemap configuration based on an index, please ensure that you have already set up an initial query index as explained there. This will generate a sitemap at the location:

`https://<branch>--<repo>--<owner>.hlx.page/sitemap.xml`

And a sitemap configuration at the following location:

`https://<branch>--<repo>--<owner>.hlx.page/helix-sitemap.yaml`

\
It is recommended that you create a `sitemap-index.xml` file that references all your sitemaps and keep that as part of your project code in your github repo. This way it is easy to add new sitemaps as the project expands.

## Manual setup of your Sitemap configuration

If you need more customization than your generated sitemap configuration file provides, you can copy its contents and paste it into a file named `helix-sitemap.yaml` in the root folder of your project.

Alternatively, if you are using the configuration service, you can also update the `sitemap.yaml` via the [site configuration](https://www.aem.live/docs/config-service-setup#update-sitemapyaml).

**Note:** When using a manually configured index and sitemap (e.g. your code repo includes a helix-query.yaml and helix-sitemap.yaml file) your index definition must include the robots property to ensure the sitemap excludes pages with `robots: noindex` metadata. When using auto-generated index definitions, simply follow the recommendations in the [indexing documentation](https://www.aem.live/developer/indexing#setting-up-properties-to-be-added-to-the-index) so those pages are excluded from the index.

The following sections contain the supported types of sitemaps.

### Simple Sitemap

The following is a simple `helix-sitemap.yaml`. It assumes a single index containing all the pages that need to appear in the sitemap.

```
 sitemaps:
   example:
     source: /query-index.json
     destination: /sitemap-en.xml
```

If you want last modification dates to be included in the URLs of your sitemap, add a `lastmod` property including a format to your configuration.

```
 sitemaps:
   example:
     source: /query-index.json
     destination: /sitemap-en.xml
     lastmod: YYYY-MM-DD
```

If your site does not have a `cdn.prod.host` defined or you want your sitemap URLs to use a different host name, you can specify an optional `origin`. Note that the origin value must include the protocol.

```
 sitemaps:
   example:
     origin: https://www.mysite.com
     source: /query-index.json
     destination: /sitemap-en.xml
     lastmod: YYYY-MM-DD
```

### Multiple Sitemaps

It is common to have sitemaps per section of the sites and/or per country or language. AEM supports sitemaps including the corresponding `hreflang` references. In the following example we assume that there is a one to one mapping between the indexes and the sitemaps XML files.

```
 sitemaps:
   example:
     lastmod: YYYY-MM-DD
     languages:
       en:
         source: /en/query-index.json
         destination: /sitemap-en.xml
         hreflang: en
       fr:
         source: /fr/query-index.json
         destination: /sitemap-fr.xml
         hreflang: fr
         alternate: /fr/{path}
```

If there are two pages in the english and french section that share a common suffix, they will be related, so e.g. if you have a page `/welcome` in the english section and a page `/fr/welcome` in the french section, the resulting entry in the `/sitemap-en.xml` will look like this:

```
<url>
  <loc>https://wwww.mysite.com/welcome</loc>
  <xhtml:link rel="alternate" hreflang="en" href="https://wwww.mysite.com/welcome"/>
  <xhtml:link rel="alternate" hreflang="fr" href="https://wwww.mysite.com/fr/welcome"/>
</url>
```

A similar entry will be available in `/sitemap-fr.xml`.

### Specifying the primary language manually

There might be situations where you have alternate versions of a page, but you’re unable to use a common suffix to identify them, possibly because you’re porting a legacy website that should not have its paths changed. In that situation, you can specify a `primary-language-url` for the alternate location, in the [metadata of the document](https://main--helix-website--adobe.aem.page/docs/authoring#metadata--seo).

Let’s assume our primary language is english, we have a page `/welcome` in the english section and `/fr/bienvenu` in the french section, and the latter is an alternate version of the former.

First, we add that information to the document at `/fr/bienvenu` in its metadata:\
![][image1]

This can also be added to a global `metadata` sheet, as shown in [Bulk Metadata](https://www.aem.live/docs/bulk-metadata).

Then, we add an indexed property `primary-language-url` to the french index:

```
 primary-language-url:
   select: head > meta[name="primary-language-url"]
   value: attribute(el, "content")
```

Finally, we re-publish the french page, and [rebuild the sitemap](https://www.aem.live/docs/admin.html#tag/sitemap/operation/generateSitemap).

### Specifying the default language

Another common requirement is to specify the default language for a sitemap with multiple languages. This can be achieved by adding a property `default` in the sitemap:

```
 sitemaps:
   example:
     default: en
     lastmod: YYYY-MM-DD
     languages:
       en:
         source: /en/query-index.json
         destination: /sitemap-en.xml
         hreflang: en
       fr:
         source: /fr/query-index.json
         destination: /sitemap-fr.xml
         hreflang: fr
         alternate: /fr/{path}
```

In the resulting sitemap, all entries from the english subtree will have an extra alternate entry with hreflang `x-default`.

### Specifying multiple hreflangs for one subtree

Sometimes, it is required to map multiple hreflangs to only one language subtree, e.g. consider we want the following to appear in the resulting sitemap:

```
<url>
 <loc>https://myhost/la/page</loc>
 <xhtml:link rel="alternate" hreflang="es-VE" href="https://myhost/la/page"/>
 <xhtml:link rel="alternate" hreflang="es-SV" href="https://myhost/la/page"/>
 <xhtml:link rel="alternate" hreflang="es-PA" href="https://myhost/la/page"/>
</url>
```

Every page in our sitemap source should appear exactly once, but have multiple alternate hreflangs associated with it. In order to achieve this, you should specify an array of languages in the `hreflang` property:

```
 sitemaps:
   example:
     lastmod: YYYY-MM-DD
     languages:
       la:
         source: /la/query-index.json
         destination: /sitemap-la.xml
         hreflang:
           - es-VE
           - es-SV
           - es-PA
```

### Multiple Indexes Aggregated Into One Sitemap

There are cases where it is easier to have a single larger sitemap than fragmented small sitemaps, especially as there is a limit of sitemaps that can be submitted to search engines per site.

The following example shows how to aggregate a number of separate indexes into a single sitemap.

```
 sitemaps:
   example:
     lastmod: YYYY-MM-DD
     languages:
       dk:
         source: /dk/query-index.json
         destination: /sitemap.xml
         hreflang: dk
         alternate: /dk/{path}
       no:
         source: /no/query-index.json
         destination: /sitemap.xml
         hreflang: no
         alternate: /no/{path}
```

Using the same destination it is possible to combine multiple small sitemaps into one larger sitemap.

### Including other sitemaps as input

In a mixed scenario, where not all languages in a sitemap are managed in AEM, you can include sitemaps from other language trees by specifying an XML path as source, as in:

```
sitemaps:
   example:
     lastmod: YYYY-MM-DD
     languages:
       en:
         source: /en/query-index.json
         destination: /sitemaps/sitemap-en.xml
         hreflang: en
       fr:
         source: https://www.mysite.com/legacy/sitemap-fr.xml
         destination: /sitemaps/sitemap-fr.xml
         hreflang: fr
         alternate: /fr/{path}
```

In this example, we use an external french sitemap to calculate all sitemap locations. AEM will determine alternates for english sitemap URLs by deconstructing the french counterparts in external sitemap using the `alternate` definition.

### Adding an extension to all locations in the sitemap

In a scenario, where you want all your locations to have an extension, e.g. `.html`, and you’re unable to generate a `helix-sitemap` sheet in your query index to derive a formula, you can add an extension to all languages or an individual language using an `extension` property:

```
sitemaps:
   example:
     lastmod: YYYY-MM-DD
     extension: .html
     languages:
       en:
         source: /en/query-index.json
         destination: /en/sitemap.xml
         hreflang: en
       fr:
         source: /fr/query-index.json
         destination: /fr/sitemap.xml
         hreflang: fr
         alternate: /fr/{path}
```

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Pagination (Contained)                                                                                                                                                             |
+-----------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
| :icon-arrow: Previous                                                                                     | Up Next :icon-arrow:                                                   |
|                                                                                                           |                                                                        |
| ### [Push Invalidation](https://main--helix-website--adobe.hlx.page/docs/setup-byo-cdn-push-invalidation) | ### [Launch](https://main--helix-website--adobe.hlx.page/docs/#launch) |
+-----------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------+

[image0]: https://main--helix-website--adobe.aem.page/media_191790b100d361466b2b0a3dc149a79ecc6511102.jpg#width=1103&height=828

[image1]: https://main--helix-website--adobe.aem.page/media_1aebb7da49a5deb3fb4824e7a625c23b856d41376.png#width=1202&height=172
