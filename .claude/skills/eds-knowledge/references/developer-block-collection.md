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

# Block Collection

This is a collection of blocks considered a part of the AEM product and are recommended as blueprints for blocks in your project.

These blocks come from real production AEM projects. To be a part of this collection, a block needs to have a high use across a number of projects and provide enough abstract functionality and be general enough so it can be reused without having to change the underlying content model.

As the needs and designs of websites change, the block collection will change as well. Additions will be made to reflect emerging needs of projects, but blocks that are not used frequently enough will also be removed (deprecated).

There are few technical principles for the blocks in the collection:

- **Intuitive:** Content structure that’s intuitive and easy to author
- **Useable:** No dependencies, compatible with boilerplate
- **Responsive:** Works across all breakpoints
- **Context Aware:** Inherits CSS context such text and background colors
- **Localizable:** No hard-coded content
- **Fast:** No negative performance impact
- **SEO and A11y:** SEO friendly and accessible

All of the blocks can be considered as a basis for your own block development. It is very likely that you will change all the `.css` and `.js` code to meet your own project needs. The primary value of these blocks is the content structure they provide.\
\
Considering that the code of your block will be fully adapted to your project, there is no intent for the blocks in the collection to be backwards compatible to their respective older versions or to make them upgradable.

## Boilerplate

The most commonly used blocks (as well as default content types) are curated in the AEM Boilerplate and are a part of every AEM project. For a block to become a part of boilerplate it has to be used by the vast majority of all AEM projects.

The code base for all the blocks in AEM Boilerplate is open-source and can be found on [GitHub adobe/aem-boilerplate](https://github.com/adobe/aem-boilerplate/tree/main/blocks)

Blocks in AEM Boilerplate can be discovered using the [sidekick library](https://www.aem.live/docs/sidekick-library) below, use the `copy` button to copy the corresponding content structure into your clipboard and paste into a document to see the content structure.

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Cards                                                                                                                                                                                                                                                                                                                                                                                               |
+---------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------+
| [Headings](https://main--helix-website--adobe.hlx.page/developer/block-collection/headings) | [Text](https://main--helix-website--adobe.hlx.page/developer/block-collection/text)       | [Images](https://main--helix-website--adobe.hlx.page/developer/block-collection/images)     | [Lists](https://main--helix-website--adobe.hlx.page/developer/block-collection/lists)                       |
|                                                                                             |                                                                                           |                                                                                             |                                                                                                             |
| _Default Content_                                                                           | _Default Content_                                                                         | _Default Content_                                                                           | _Default Content_                                                                                           |
|                                                                                             |                                                                                           |                                                                                             |                                                                                                             |
| Different levels of headings provide the semantic backbone of your document                 | Body text or copy with rich semantic formatting options                                   | Pictures bring your content alive                                                           | Ordered and unordered lists wherever they are needed                                                        |
+---------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------+
| [Links](https://main--helix-website--adobe.hlx.page/developer/block-collection/links)       | [Buttons](https://main--helix-website--adobe.hlx.page/developer/block-collection/buttons) | [Code](https://main--helix-website--adobe.hlx.page/developer/block-collection/code)         | [Sections](https://main--helix-website--adobe.hlx.page/developer/block-collection/sections)                 |
|                                                                                             |                                                                                           |                                                                                             |                                                                                                             |
| _Default Content_                                                                           | _Default Content_                                                                         | _Default Content_                                                                           | _Default Content_                                                                                           |
|                                                                                             |                                                                                           |                                                                                             |                                                                                                             |
| Reference other websites or your own content                                                | Call-to-action buttons and more                                                           | Highlight preformatted code snippets in your content                                        | Group content on your page into sections                                                                    |
+---------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------+
| [Icons](https://main--helix-website--adobe.hlx.page/developer/block-collection/icons)       | [Hero](https://main--helix-website--adobe.aem.page/developer/block-collection/hero)       | [Columns](https://main--helix-website--adobe.hlx.page/developer/block-collection/columns)   | [Cards](https://main--helix-website--adobe.hlx.page/developer/block-collection/cards)                       |
|                                                                                             |                                                                                           |                                                                                             |                                                                                                             |
| _Default Content_                                                                           | _Block_                                                                                   | _Block_                                                                                     | _Block_                                                                                                     |
|                                                                                             |                                                                                           |                                                                                             |                                                                                                             |
| Make your content more interesting with icons                                               | Hero treatment at the top of a page                                                       | Flexible way to handle multi-column layouts in a responsive way                             | List of cards with or without images and links                                                              |
+---------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------+
| [Header](https://main--helix-website--adobe.hlx.page/developer/block-collection/header)     | [Footer](https://main--helix-website--adobe.hlx.page/developer/block-collection/footer)   | [Metadata](https://main--helix-website--adobe.hlx.page/developer/block-collection/metadata) | [Section Metadata](https://main--helix-website--adobe.hlx.page/developer/block-collection/section-metadata) |
|                                                                                             |                                                                                           |                                                                                             |                                                                                                             |
| _Block_                                                                                     | _Block_                                                                                   | Add metadata to your page where needed                                                      | Highlight or structure all the content in a section                                                         |
|                                                                                             |                                                                                           |                                                                                             |                                                                                                             |
| Flexible header and navigation example                                                      | Simple extensible footer block                                                            |                                                                                             |                                                                                                             |
+---------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------+

## Block Collection

The block collection contains blocks that are commonly-used, but are not so common to be considered boilerplate. As a rule-of-thumb, to be included in the block collection a block must be used on more than half of all AEM projects.

The block collection can be the entry path into boilerplate code. Likewise if a block in the boilerplate is no longer used as much, it can be moved to this collection.

The code base for all the blocks in AEM Block Collection is open-source and can be found on [GitHub adobe/aem-block-collection](https://github.com/adobe/aem-block-collection/tree/main/blocks)

Blocks in AEM Block Collection can be discovered using the [sidekick library](https://www.aem.live/docs/sidekick-library) below, use the `copy` button to copy the corresponding content structure into your clipboard and paste into a document to see the content structure.

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Cards                                                                                                                                                                                                                                                                                                                                                                                   |
+-----------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------+
| [Embed](https://main--helix-website--adobe.hlx.page/developer/block-collection/embed)         | [Fragment](https://main--helix-website--adobe.hlx.page/developer/block-collection/fragment)       | [Table](https://main--helix-website--adobe.hlx.page/developer/block-collection/table)       | [Video](https://main--helix-website--adobe.hlx.page/developer/block-collection/video) |
|                                                                                               |                                                                                                   |                                                                                             |                                                                                       |
| _Block_                                                                                       | _Block_                                                                                           | _Block_                                                                                     | _Block_                                                                               |
|                                                                                               |                                                                                                   |                                                                                             |                                                                                       |
| A simple way to embed social media content into AEM pages                                     | Share pieces of content across multiple pages                                                     | A way to organize tabular data into rows and columns                                        | Display and playback videos directly from AEM                                         |
+-----------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------+
| [Accordion](https://main--helix-website--adobe.hlx.page/developer/block-collection/accordion) | [Breadcrumbs](https://main--helix-website--adobe.hlx.page/developer/block-collection/breadcrumbs) | [Carousel](https://main--helix-website--adobe.hlx.page/developer/block-collection/carousel) | [Modal](https://main--helix-website--adobe.hlx.page/developer/block-collection/modal) |
|                                                                                               |                                                                                                   |                                                                                             |                                                                                       |
| _Block_                                                                                       | _Block Add-on_                                                                                    | _Block_                                                                                     | _Autoblock_                                                                           |
|                                                                                               |                                                                                                   |                                                                                             |                                                                                       |
| A stack of descriptive labels that can be toggled to display related full content             | A list of page titles and relevant links showing the location of the current page in the          | A dynamic display tool that smoothly transitions through a series of images with optional   | A popup that appears over other site content                                          |
|                                                                                               | navigational hierarchy                                                                            | text content                                                                                |                                                                                       |
+-----------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------+
| [Quote](https://main--helix-website--adobe.hlx.page/developer/block-collection/quote)         | [Search](https://main--helix-website--adobe.hlx.page/developer/block-collection/search)           | [Tabs](https://main--helix-website--adobe.hlx.page/developer/block-collection/tabs)         | [Form](https://main--helix-website--adobe.hlx.page/developer/block-collection/form)   |
|                                                                                               |                                                                                                   |                                                                                             |                                                                                       |
| _Block_                                                                                       | _Block_                                                                                           | _Block_                                                                                     | _Block (Deprecated)_                                                                  |
|                                                                                               |                                                                                                   |                                                                                             |                                                                                       |
| A display of a quotation or a highlight of specific passage (or “pull quotes”) within a       | Allows users to find site content by entering a search term                                       | Segment information into multiple labeled (or “tabbed”) panels                              | A set of input controls grouped together that enables users to submit information     |
| document                                                                                      |                                                                                                   |                                                                                             |                                                                                       |
+-----------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------+

The block collection is continually evolving based on the feedback from the AEM community. If you think that there is a block that should be included in the block collection please speak to your AEM contact. Current candidates for inclusion in the block collection include:

- Consent Banner

If you have immediate need of a block that is not yet part of the collection, it is relatively easy to find AEM projects on GitHub that have example implementations for all of the above candidates.

## Block Party

The Block Party is a place for the AEM developer community to showcase what they have built on AEM sites. It also allows others to avoid reinventing the wheel and reuse these blocks / code snippets / integrations built by the community and tweak the code as necessary to fit their own projects. See [Block Party](https://main--helix-website--adobe.aem.page/developer/block-party/) for everything it has to offer.

**Note: While we love and support our AEM developer community, Adobe is not responsible for maintaining or updating the code that is showcased in Block Party. Please use the code at your own discretion.**

+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Pagination (Contained)                                                                                                                                                                                  |
+-----------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------+
| :icon-arrow: Previous                                                                                           | Up Next :icon-arrow:                                                                  |
|                                                                                                                 |                                                                                       |
| ### [Anatomy of a Project](https://main--helix-website--adobe.hlx.page/developer/anatomy-of-a-franklin-project) | ### [Block Party](https://main--helix-website--adobe.aem.page/developer/block-party/) |
+-----------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------+

[image0]: https://main--helix-website--adobe.hlx.page/media_18e872cbcb0d42cec5e1b7790d1b3fddfcea0dac8.jpeg#width=1103&height=827
