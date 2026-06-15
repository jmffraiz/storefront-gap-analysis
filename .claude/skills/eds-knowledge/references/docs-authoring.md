+------------------------------------------------------------------------+
| Metadata                                                               |
+-------------+----------------------------------------------------------+
| Title       | Authoring and Publishing Content                         |
+-------------+----------------------------------------------------------+
| Description | How to author, preview and publish content using the AEM |
|             | Sidekick.                                                |
+-------------+----------------------------------------------------------+
| Image       | ![][image0]                                              |
+-------------+----------------------------------------------------------+
| Category    | publish                                                  |
+-------------+----------------------------------------------------------+
| Template    | guides                                                   |
+-------------+----------------------------------------------------------+

+-------------------+
| Section Metadata  |
+---------+---------+
| style   | content |
+---------+---------+

![][image0]

# Authoring and Publishing Content

## Authoring Content

### You already know the most important part.

If you use Microsoft Word or Google Docs, then you already know how to create content.

Your documents in Word or Google Docs become your pages on your website. Your headings in your documents will become headings on your website. Bold, italic, underlining, lists, images, etc. will all appear on your website.

### Images and Videos

To add an image to your document, drag the image into the page. Word and Google Docs automatically add it as normal. Your image will be resized to fit the browser window of your visitor. Any resizing you do in Word or Google Docs will have no effect.

It is a good idea to set an alternative text for all images you add to the document, as this increases accessibility and helps search engines find your content.

To do this, use the built-in features of Document Authoring, Microsoft Word or Google Docs. See the documentation of either product for more details.

- [Document Authoring](https://docs.da.live/authors/guides/editing-docs#images)
- [Microsoft Word](https://support.microsoft.com/en-us/office/add-alternative-text-to-a-shape-picture-chart-smartart-graphic-or-other-object-44989b2a-903c-4d9a-b742-6a75b451c669)
- [Google Docs](https://support.google.com/docs/answer/6199477?hl=en)

Microsoft Word and Google Docs do not allow you to just drag and drop videos, but you can add videos via SharePoint or Google Drive, preview and publish them [using the Sidekick](https://main--helix-website--adobe.aem.page/docs/sidekick) and add the resulting URL as a link to a [suitable block](https://main--helix-website--adobe.aem.page/developer/block-collection/video) in your document.

### Links

Links are an important part of every website and you can add them both in Word and Google Docs. If you are creating a link within your website, enter the URL as is, even if the page you are linking to is not public yet (e.g. a preview or live URL).

Links pointing to other pages on the same site will automatically be adjusted to be relative to your site.

You can link to headings or sections within a page by appending an anchor value to the URL. Heading elements have automatic lower-cased IDs where spaces are replaced with dashes. For example, if a page `/about-us` has a heading “Our History”, the URL linking to it would be `https://<your_host>/about-us#our-history`.

### Sections

On some websites you have sections or blades that change background color or otherwise indicate breaks in the content. Creating a section break in both Microsoft Word and Google Docs can be done using `---` (three hyphens) on a single line. In Google Docs you can also create sections by inserting a Horizontal Line element into the page (select “Insert → Horizontal Line” from the menu).

## Blocks

Blocks are a way to work with more structured content and add special functionality to your site. Which blocks are available to your site depends on what your development team has implemented and differs from site to site. The only block that is common to all sites is the metadata block described previously.

Regardless of site, the structure of a block is always the same: it is a table with a merged first row that serves as the block name (header row). The header row may have specific formatting like a background color to increase their discoverability and differentiation in a document.

Blocks usually contain content, configuration, or references to other pieces of content, be it from other documents, spreadsheets, or both.

![][image1]

As you can see from this example, you are free to put any kind of content into the cells of a block, and it is up to the block to either render the content or ignore it. If the site you are working on uses blocks extensively, then you will probably have a reference list of blocks you can use.

Blocks can have variants in parenthesis. For example, a `Columns` block can have a `(highlight)`option which passes a layout hint to the block display logic.

See the document [The Block Collection](https://www.hlx.live/developer/block-collection) to learn more about out-of-the-box blocks.

## Metadata

See the document [Page Metadata](https://main--helix-website--adobe.aem.page/docs/metadata) for instructions on how to manage metadata for your pages.

## Structured Data in Spreadsheets

You can put content into spreadsheets and then the spreadsheet is automatically turned into an API that your developers can use. This allows you to use spreadsheets like a headless CMS for use in data tables, navigation, or feature comparisons, for example.

See the document [Spreadsheets and JSON](https://www.hlx.live/developer/spreadsheets) for more information.

## Preview and Publish Content

Once a document is created in Google Drive or Sharepoint, you can preview the corresponding web page and eventually publish the content to your production website.

The preview function is used to share pages with stakeholders before they are published and available to the general public on your website.

In order to preview, publish, or delete content, [use the Sidekick](https://main--helix-website--adobe.aem.page/docs/sidekick) that can be installed as a browser extension.

### Preview

In Word or Google Docs, open the Sidekick, then click the “Preview” button. This will open a new browser window (check for the popup warning) that has the preview version of your site.

Although you can copy and share the URL of this preview, it is not meant for production. It does not have your domain name on it and is invisible to search engines. If the content is ready for publication, you can publish. If you need to make changes, open the Sidekick on the preview page and click “Edit” to go back to Word or Google Docs.

### Publish

Publishing makes your content visible to everyone on the internet. To publish something, open the sidekick on a preview page (or follow the instructions above to open the preview again), then click “Publish”. After a few seconds, a new browser window will open, with your page on your public website.

Once your content has been published, it is visible to everyone on the internet, and search engines will be able to find it.

### Delete

Generally, deleting published content and therefore removing publicly accessible resources from the web can be problematic because of inbound links from search, social, bookmarks and other referring sites. If a page is deleted that was once published, it is recommended to use redirects to make sure that incoming traffic for the deleted page is sent to the next best place. See the document [Redirects](https://www.hlx.live/docs/redirects) for more information.

If you want to remove published content or just delete it from your site as part of a clean-up, doing so is a two-step process.

1. First, delete the source document.
   - Alternatively you can rename the page or move it to a different folder, for example your `drafts` folder.

1) Then open the page you want to delete on the preview site, open the sidekick and sign in. You will now see a `...` menu with two options: `Delete` and `Unpublish`.
   - `Unpublish` removes it from the public production website, but keeps the preview.
   - `Delete` removes the preview, too.

Deleting or unpublishing something is permanent and cannot be undone easily. If you want to undo a deletion, you have to restore the original document and then preview and publish it again.

+------------------------------------------------------------------------------------------------------------------------------------------------------+
| Pagination (Contained)                                                                                                                               |
+--------------------------------------------------------------------------+---------------------------------------------------------------------------+
| :icon-arrow: Previous                                                    | Up Next :icon-arrow:                                                      |
|                                                                          |                                                                           |
| ### [Publish](https://main--helix-website--adobe.hlx.page/docs/#publish) | ### [Metadata](https://main--helix-website--adobe.aem.page/docs/metadata) |
+--------------------------------------------------------------------------+---------------------------------------------------------------------------+

[image0]: https://main--helix-website--adobe.aem.page/media_1cf7bb3a1af050eff35416bc16502895c1f5a166e.jpg#width=1103&height=827

[image1]: https://main--helix-website--adobe.aem.page/media_1d1b790457ebb452685739cbd3ab6374db01ebf8e.png#width=964&height=804
