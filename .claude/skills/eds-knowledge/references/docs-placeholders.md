+------------------------+
| Metadata               |
+----------+-------------+
| Template | guides      |
+----------+-------------+
| Image    | ![][image0] |
+----------+-------------+
| Category | Publish     |
+----------+-------------+

+-------------------+
| Section Metadata  |
+---------+---------+
| style   | content |
+---------+---------+

![][image0]

# Placeholders

In most websites, there are strings or variables that will be used throughout the site. Especially in sites that need to support multiple languages, it is not a good idea to hard code such values. Instead placeholders can be used and managed centrally.

Note: depending on your site content structure, placeholders might not be in use.

Placeholders can be managed as a spreadsheet that is either in the root folder of the project or in the locales root folder in the case of a multilingual site.

- Name the file `placeholders` for Google Docs.
- Name the file `placeholders.xlsx` for SharePoint.

The spreadsheet has to contain at least two columns titled `Key` and `Text`.

![][image1]

- The `Key` column is an identifier that is transformed automatically to be easily accessible via code.
- The `Text` is the literal text (or string) for a placeholder with a given key.

After making changes to your placeholder spreadsheet, you can preview your changes via the sidekick and have your stakeholders check that the new placeholders are working on your `.page` preview website before publishing the placeholder changes to your production website. See the [Sidekick documentation](https://www.hlx.live/docs/sidekick#sharing-the-sidekick) for more information about switching between environments.

Are you a developer and curious to learn how to use placeholders in your code? [Look here](https://www.hlx.live/developer/placeholders).

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Pagination (Contained)                                                                                                                                                              |
+-------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------+
| :icon-arrow: Previous                                                   | Up Next :icon-arrow:                                                                                      |
|                                                                         |                                                                                                           |
| ### [Slack Bot](https://main--helix-website--adobe.hlx.page/docs/slack) | ### [Push Invalidation](https://main--helix-website--adobe.hlx.page/docs/setup-byo-cdn-push-invalidation) |
+-------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------+

[image0]: https://main--helix-website--adobe.hlx.page/media_1924a42826eff0f60ff46c462d9fe3749e6a7bb66.png#width=1600&height=1200

[image1]: https://main--helix-website--adobe.hlx.page/media_1ca9282b18333fd9acfa2c200017a3cebebd631e1.png#width=1384&height=672
