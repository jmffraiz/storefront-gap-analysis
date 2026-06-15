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

# Spreadsheets and JSON

In addition to translating Google Docs and Word documents into markdown and HTML markup, AEM also translates spreadsheets (Microsoft Excel workbooks and Google Sheets) into JSON files that can easily be consumed by your website or web application.

This enables many uses for content that is table-oriented or structured.

## Sheets and Sheet structure

The simplest example of a sheet consists of a table that uses the first row as column names and the subsequent rows as data. An example might look something like this.

![][image1]

After a preview and publish via the [sidekick](https://www.hlx.live/docs/sidekick), AEM translates this table to a JSON representation which is served to requests to the corresponding `.json` resource. The above example gets translated to:

```
{
  "total": 4,
  "offset": 0,
  "limit": 4,
  "columns": ["Source", "Destination"],
  "data": [
    {
      "Source": "/sidekick-extension",
      "Destination": "https://chromewebstore.google.com/detail/aem-sidekick/igkmdomcgoebiipaifhmpfjhbjccggml"
    },
    {
      "Source": "/github-bot",
      "Destination": "https://github.com/apps/helix-bot"
    },
    {
      "Source": "/install-github-bot",
      "Destination": "https://github.com/apps/helix-bot/installations/new"
    },
    {
      "Source": "/tutorial",
      "Destination": "/developer/tutorial"
    }
  ],
  ":type": "sheet"
}
```

AEM allows you to manage workbooks with multiple sheets.

- **If there is only one sheet,** AEM will by default use that sheet as the source of the information.
- **If there are multiple sheets,** AEM will deliver sheets that have names prefixed with `shared-`. This lets you keep sheets in the same workbook that will not be exposed.
- **If there is a sheet named** `shared-default`, AEM will deliver it as a single sheet, unless there is a [query parameter](#sheet) pointing to a different sheet.
- **If there are multiple sheets with** `shared-` **prefix**, AEM will deliver them in multi-sheet format. See [below](#multi-sheet-format) for an example.

**Note**: the `helix-` prefix is deprecated and the more neutral `shared-` prefix should be used.

See the [query parameter section](#query-parameters) for details on how to query a specific sheet.

## Multi-Sheet Format

If there are multiple sheets with `shared-` prefix, AEM will deliver them in multi-sheet format. Here's an example of a payload with 2 sheets:

```
{
  ":names": [
    "first",
    "second"
  ],
  ":type": "multi-sheet",
  ":version": 3,
  "first": {
    "total": 0,
    "offset": 0,
    "limit": 0,
    "data": [],
    "columns": []
  },
  "second": {
    "total": 0,
    "offset": 0,
    "limit": 0,
    "data": [],
    "columns": []
  }
}
```

- The `:names` property contains an array with all the names of the contained sheets.
- For each sheet name in `:names` there is a property in the payload. The value is the corresponding sheet data in [single sheet](#sheets-and-sheet-structure) format.

## Query Parameters

### Offset and Limit

Spreadsheets and JSON files can get very large. In such cases, AEM supports the use of `limit` and `offset` query parameters to indicate which rows of the spreadsheet are delivered. In case of a [multi-sheet](#multi-sheet-format), offset and limit are applied to all sheets in the payload.

As AEM always compresses the JSON, payloads are generally relatively small. Therefore by default AEM limits the number of rows it returns to 1000 if the `limit` query parameter is not specified. This is sufficient for many simple cases.

### Sheet

The `sheet` query parameter allows an application to specify one or multiple specific sheets in the spreadsheet or workbook. As an example `?sheet=jobs` will return the sheet named `shared-jobs` and `?sheet=jobs&sheet=articles` will return the data for the sheets named `shared-jobs` and `shared-articles`.

## Special Sheet Names

In certain use cases, AEM also writes to spreadsheets, where it expects specific sheet names:

- [The indexing service](https://main--helix-website--adobe.aem.page/developer/indexing) only writes to a sheet named `raw_index`, which may be delivered via JSON in a single sheet setup.

## Arrays

Native arrays are not supported as cell values, so they are delivered as strings.

`"tags": "[\"Adobe Life\",\"Responsibility\",\"Diversity & Inclusion\"]"`

You can turn them back into arrays in JavaScript using `JSON.parse()`.

+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Pagination (Contained)                                                                                                                                                           |
+-------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------+
| :icon-arrow: Previous                                                                                 | Up Next :icon-arrow:                                                     |
|                                                                                                       |                                                                          |
| ### [Markup - Sections](https://main--helix-website--adobe.hlx.page/developer/markup-sections-blocks) | ### [Publish](https://main--helix-website--adobe.hlx.page/docs/#publish) |
+-------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------+

[image0]: https://main--helix-website--adobe.hlx.page/media_10a516dc1e3a4c9b42aacb149e1bf202ea3e93b8c.jpeg#width=1103&height=827

[image1]: https://main--helix-website--adobe.hlx.page/media_1a0aa697c252fdd6e810b0c4fa8263d8243939426.png#width=1600&height=361
