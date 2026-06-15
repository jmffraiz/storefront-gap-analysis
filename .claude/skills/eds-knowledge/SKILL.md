---
name: eds-knowledge
description: >-
  Adobe Edge Delivery Services (EDS / AEM EDS) reference knowledge base.
  Covers blocks, markup, sections, authoring, metadata, spreadsheets, JSON,
  indexing, sitemaps, redirects, placeholders, favicons, web performance,
  Lighthouse, Core Web Vitals, LCP, CLS, TBT, go-live checklist, CDN,
  developer tutorial, boilerplate, aem.live, aem.page, helix, and
  development collaboration best practices. Use when the request involves
  EDS, Edge Delivery Services, AEM blocks, AEM sites, aem.live, aem.page,
  helix, Franklin, or any Adobe Experience Manager web development topic.
---

# EDS Knowledge Base

Reference documentation for Adobe Edge Delivery Services (EDS).
Only load the specific document(s) that match the current request.

## How to use

1. Match the user's request against the **Document Index** below.
2. Read **only** the matching document(s) from the `references/` folder adjacent to this file.
3. If no single doc matches, check if multiple docs are relevant and load only those.
4. If the request is general EDS and no specific doc applies, briefly summarize the available topics and ask the user to narrow down.

## Document Index

| File | Topic | When to load |
|------|-------|-------------|
| [developer-tutorial.md](references/developer-tutorial.md) | Getting started, boilerplate setup, AEM CLI, local dev, first site | Setting up a new EDS project, boilerplate, `aem up`, tutorial steps |
| [developer-markup-sections-blocks.md](references/developer-markup-sections-blocks.md) | DOM structure, sections, default content, blocks, block options, auto blocking | Block development, markup structure, DOM decoration, auto-blocks, block options, `scripts.js` |
| [developer-block-collection.md](references/developer-block-collection.md) | Standard block catalog: cards, columns, hero, carousel, accordion, tabs, embed, fragment, etc. | Which blocks are available, boilerplate blocks, block collection, block party |
| [developer-spreadsheets.md](references/developer-spreadsheets.md) | Spreadsheets to JSON, sheets, multi-sheet format, query parameters, offset/limit | Spreadsheet/JSON APIs, structured data, sheet naming conventions, `shared-` prefix |
| [developer-indexing.md](references/developer-indexing.md) | Query index setup, `query-index.xlsx`, `helix-query.yaml`, robots/noindex | Indexing pages, query index, search, filtering, `helix-query.yaml`, noindex |
| [developer-keeping-it-100.md](references/developer-keeping-it-100.md) | Web performance, Lighthouse 100, E-L-D loading phases, LCP, CLS, TBT, CWV, fonts, delayed.js | Performance optimization, Lighthouse score, Core Web Vitals, loading phases, eager/lazy/delayed |
| [developer-sitemap.md](references/developer-sitemap.md) | Sitemap generation, `helix-sitemap.yaml`, hreflang, multi-language sitemaps | Sitemap config, `sitemap.xml`, hreflang, multi-language, SEO sitemaps |
| [developer-favicon.md](references/developer-favicon.md) | Adding favicon, repoless favicons | Favicon setup, `favicon.ico`, repoless |
| [docs-authoring.md](references/docs-authoring.md) | Content authoring in Word/Google Docs, images, links, sections, blocks, preview/publish | How authors create content, authoring workflow, preview, publish, delete content |
| [docs-bulk-metadata.md](references/docs-bulk-metadata.md) | Bulk metadata via `metadata.xlsx`, URL patterns, wildcards, metadata hierarchy | Bulk metadata, site-wide metadata, metadata sheet, template/theme assignment |
| [docs-placeholders.md](references/docs-placeholders.md) | Placeholders spreadsheet, Key/Text columns, i18n strings | Placeholders, localized strings, `placeholders.xlsx`, i18n |
| [docs-redirects.md](references/docs-redirects.md) | Redirects spreadsheet, Source/Destination, wildcard redirects, SEO migration | Redirects, URL mapping, `redirects.xlsx`, site migration SEO |
| [docs-go-live-checklist.md](references/docs-go-live-checklist.md) | Go-live steps: QA, performance, analytics, RUM, redirects, sitemap, CDN, push invalidation | Launching a site, go-live, pre-launch checklist, CDN setup, post-launch validation |
| [docs-dev-collab-and-good-practices.md](references/docs-dev-collab-and-good-practices.md) | GitHub workflow, PR etiquette, CSS/JS best practices, linting, mobile-first, content-first | Dev best practices, coding standards, PR workflow, CSS isolation, trunk-based dev |
