+------------------------+
| Metadata               |
+----------+-------------+
| Template | guides      |
+----------+-------------+
| Image    | ![][image0] |
+----------+-------------+
| Category | Build       |
+----------+-------------+

+-------------------+
| Section Metadata  |
+---------+---------+
| style   | content |
+---------+---------+

![][image0]

# Indexing

Adobe Experience Manager offers a way to keep an index of all the published pages in a particular section of your website. This is commonly used to build lists, feeds, and enable search and filtering use cases for your pages or content fragments.

AEM keeps this index in a spreadsheet and offers access to it using JSON. Please see the document [Spreadsheets and JSON](https://main--helix-website--adobe.hlx.page/developer/spreadsheets) for more information.

We will introduce the concept of creating a query index by previewing an Excel workbook or Google spreadsheet first. Note, that if you already have a custom query definition in a file called `helix-query.yaml` in your GitHub repository, it is no longer possible to create indexes that way. Every new index will have to be manually added to that `helix-query.yaml`.

## Setting Up an Initial Query Index

In this section we’ll create a query index in the root folder that will index all documents in your backend.

1. After setting up your `fstab.yaml` with a mountpoint that points into your SharePoint site or Google Drive, go to the root folder.
2. Depending on your backend, create either a workbook named `query-index.xlsx` for SharePoint or a spreadsheet named `query-index` for Google Drive.
3. In that spreadsheet or workbook, create a sheet named `raw_index`.

## Setting Up Properties to be Added to the Index

1. In your `query-index` document, add a header line and in the first column add `path` as the header name.
2. In the following columns of the header line, add all other properties you need extracted from the rendered HTML page.

In the following example in Google Drive, the extracted fields are `title`, `image`, `description`, and `lastModified`.

![][image1]

\
Pages are indexed when they are published. To remove pages from index, they have to be unpublished.

For simple scenarios without custom index definition, pages that have `robots` metadata property set to `noindex` will automatically be omitted from indexing by AEM. (There are a few special scenarios here, for more details see the section [Special Scenarios for Robots](https://main--helix-website--adobe.aem.page/developer/indexing#special-scenarios-for-robots)).

\
The following table summarizes the properties that are available and from where in the HTML page they’re extracted.

+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table                                                                                                                                                     |
+===========================================================================================================================================================+
| +----------------+--------------------------------------------------------------------------------------------------------------------------------------+ |
| | Name           | Description                                                                                                                          | |
| +================+======================================================================================================================================+ |
| | `author`       | Returns the content of the meta tag named `author` in the `head` element.                                                            | |
| +----------------+--------------------------------------------------------------------------------------------------------------------------------------+ |
| | `title`        | Returns the content of the `og:title` meta property in the `head` element.                                                           | |
| +----------------+--------------------------------------------------------------------------------------------------------------------------------------+ |
| | `date`         | Returns the content of the meta tag named `publication-date` in the `head` element.                                                  | |
| +----------------+--------------------------------------------------------------------------------------------------------------------------------------+ |
| | `image`        | Returns the content of the `og:image` meta property in the `head` element.                                                           | |
| +----------------+--------------------------------------------------------------------------------------------------------------------------------------+ |
| | `category`     | Returns the content of the meta tag named `category` in the `head` element.                                                          | |
| +----------------+--------------------------------------------------------------------------------------------------------------------------------------+ |
| | `tags`         | Returns the content of the meta tag named `article:tag` in the `head` element as an array.                                           | |
| |                |                                                                                                                                      | |
| |                | See the document [Spreadsheets and JSON](https://www.hlx.live/developer/spreadsheets#arrays) for more information on array-handling. | |
| +----------------+--------------------------------------------------------------------------------------------------------------------------------------+ |
| | `description`  | Returns the content of the meta tag named `description` in the `head` element.                                                       | |
| +----------------+--------------------------------------------------------------------------------------------------------------------------------------+ |
| | `robots`       | Returns the content of the meta tag named `robots` in the `head` element.                                                            | |
| +----------------+--------------------------------------------------------------------------------------------------------------------------------------+ |
| | `lastModified` | Returns the value of the `Last-Modified` response header for the document.                                                           | |
| +----------------+--------------------------------------------------------------------------------------------------------------------------------------+ |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------+

For every other header added, the indexer will try to find a meta tag with a corresponding name.

## Activate Your Index

To activate your index, preview the spreadsheet using the sidekick. This will create an index configuration.

## Checking Your Index

The Admin Service has an API endpoint where you can check the index representation of your page. Given your GitHub owner, repository, branch and owner, and a resource path to a page, its endpoint is:

`https://admin.hlx.page/index/<owner>/<repo>/<branch>/<path>`

You should get a JSON response where the data node contains the index representation of the page.

## Debugging Your Index Configuration

The AEM CLI has a feature where it will print the index record whenever you change your query configuration, which assists in finding the correct CSS selectors:

`$ aem up --print-index`

Please see the [AEM CLI GitHub documentation](https://github.com/adobe/helix-cli) for more information and watch this [video](https://www.hlx.live/media_15501e6b07101255cb6ef99077f218666809ec354.mp4) to learn more about this feature.

## Setting Up More Index Configurations

You can define your own custom index configurations by creating your own `helix-query.yaml. `This allows you to have more than one index configuration in the same `helix-query.yaml`, where parts of your sites are indexed into different Excel workbooks or Google spreadsheets. See the document [Indexing reference](https://www.hlx.live/docs/indexing-reference) for more information.

## Special Scenarios for Robots

There are a few nuances on how pages get indexed by AEM in conjunction with indexing setup for your site. Let’s look at them:

In the following 2 situations, setting `robots` to `noindex` on the page metadata would **not** prevent it from being indexed by AEM:

- You have added a  `robots` column in `query-index.xlsx`
- You have a `helix-query.yaml` in your Github repository i.e. you have defined a **custom index definition.**

### Recommendations

1. **If you do not have a custom index definition**, it is recommended to **not** add a `robots` column to your index sheet unless you have a requirement for doing so.\
   Adding `robots` column to your index sheet would cause a page to be indexed by AEM even though it may have `robots` metadata set to `noindex.`
2. **If you do have a custom index definition**, pages would get indexed by AEM irrespective of setting `robots` to `noindex` on the page metadata. If you want to prevent this from happening,  you can use spreadsheet filters to omit pages from index that have `robots` metadata set to` noindex`. For more details, see the section titled  "[Enforcing](https://main--helix-website--adobe.hlx.page/developer/indexing#enforcing-noindex-configuration-with-custom-index-definitions) `noindex` [configuration with custom index definitions](https://main--helix-website--adobe.hlx.page/developer/indexing#enforcing-noindex-configuration-with-custom-index-definitions)" below.

### Enforcing “noindex” configuration with custom index definitions

If you have defined your own custom index definitions in` helix-query.yaml`, setting the `robots` property to `noindex` is not effective in preventing the pages from getting indexed. In order to enforce`  noindex  `configuration is such situations, do the following:

1. Create a sheet named “`helix-default`” in your` query-index.xlsx` . After this, your `query-index.xlsx` spreadsheet should have 2 sheets `“raw_index`” and `“helix-default`”. The `“raw_index`” sheet is there to have all the raw indexed data.
2. Modify your custom `helix-query.yaml` (it must be in your project’s Github repository) and add the `robots` property so that it gets indexed.
3. Now set up your `“helix-default`” sheet in the `query-index.xlsx` spreadsheet to get automatically filled up using Excel formula which ensures that all the rows in `raw_index` which have` robots` property set as` noindex`, do not get copied over to the ` helix-default` sheet. This can be done by using an Excel formula like this `=FILTER(Table1,NOT(Table1[robots]="noindex"))`
4. Now your helix-default sheet has only the rows from` raw_index` that do not have` robots` property set to` noindex`.
5. Ensure that you publish the pages that you want to get indexed.
6. Now if you fetch the index as usual like:` https://<branch>--<repo>-<org>.hlx.page/query-index.json`, you’d only get data from` helix-default` sheet i.e. entries that are not explicitly prevented from getting indexed through the `robot `property set as` noindex`.

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Pagination (Contained)                                                                                                                                                |
+--------------------------------------------------------------------------+--------------------------------------------------------------------------------------------+
| :icon-arrow: Previous                                                    | Up Next :icon-arrow:                                                                       |
|                                                                          |                                                                                            |
| ### [Forms](https://main--helix-website--adobe.hlx.page/developer/forms) | ### [Keeping it 100](https://main--helix-website--adobe.hlx.page/developer/keeping-it-100) |
+--------------------------------------------------------------------------+--------------------------------------------------------------------------------------------+

[image0]: https://main--helix-website--adobe.hlx.page/media_154896ddb0d10ee236adc3592217d30238ede804c.jpeg#width=1103&height=828

[image1]: https://main--helix-website--adobe.hlx.page/media_10b8811f4b7a024f7fe26a86ee8fd1a79ae54285a.png#width=1600&height=1308
