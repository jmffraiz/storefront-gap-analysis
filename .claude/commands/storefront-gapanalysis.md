---
description: Produce a structured fit/gap analysis document for an ACCS + EDS storefront page or feature set, mapping each required feature to the most specific existing `@dropins/storefront-*` drop-in, container, and slot.
argument-hint: [page-or-feature-name]
---

# Storefront Gap Analysis

Produce a structured **fit/gap analysis** document for the storefront page or feature set: **$ARGUMENTS**

If `$ARGUMENTS` is empty, ask the user which page / feature / mockup to analyze before doing anything else.

A gap analysis answers: *for each feature in this design, which existing drop-in / container / slot already covers it, what's partially covered, what has to be built from scratch — and how complex and risky is each gap.*

## Knowledge sources

All Commerce knowledge (drop-in reference files, container/slot/event names, service facts, `llms-full.txt`) lives in the **`accs-storefront-architect`** skill — consult it rather than restating its content. If that skill is unavailable, produce the document but flag any identifier you could not verify.

## Workflow

1. **Read the template** at `.claude/templates/page-gap-analysis.template.md` to get the exact section structure and column definitions before writing anything.
2. **Gather Commerce context** by spawning the `storefront-architect` subagent (`Agent` tool, `subagent_type: "storefront-architect"`). Ask it for the containers, slots, events, and service dependencies relevant to the page type. Grep `llms-full.txt` directly only for identifiers the subagent could not confirm.
3. **Fill every section of the template:**
   - **§1 Feature decomposition** — one row per discrete feature visible in the design or described by the user. Classify each as *Commerce*, *Authored content*, or *Cross-cutting*.
   - **§2 Gap analysis** — map each feature to the most specific existing drop-in / container / slot. Use exact package names (`@dropins/storefront-cart`), container names (`CartSummaryList`), and slot names (`Heading`, `Footer`). **Hyperlink every drop-in package name, container name, and named Adobe service to its official Experience League URL** using the documentation URL reference table at the end of the template. Set Coverage to ✅ / 🟡 / ❌ using the legend. Size Complexity and Risk using the bucket definitions in the template preamble.
   - **§3 Assumptions and open questions** — list decisions blocking the estimate; follow the *Recommendation / Impact A / Impact B* format.
   - **§4 Out of scope** — list explicitly what is excluded to protect the estimate.
4. **Never invent slot or event names** — verify them against the reference files or `llms-full.txt`.
5. **Output the completed document as a fenced markdown code block** so the user can copy it directly.

## Rules of thumb for accurate gaps

- **Slots before overrides before forks.** A feature reachable through a named slot is 🟡 *partial* at Low/Medium complexity; one needing a fork is ❌ or high-complexity 🟡. Don't mark something ❌ when a slot reaches it.
- **Prefer the most specific container.** Map to `ProductPrice` / `ProductGallery`, not the deprecated monolithic `ProductDetails`. Map cart line items to `CartSummaryList`, not "the cart drop-in" generically.
- **A new service dependency raises Risk.** Anything that turns on Live Search, Product Recommendations, Payment Services, or Data Connection / AEP carries backend-enablement risk — note it in Dependencies and bump Risk.
- **Authored content is not a gap.** Hero copy, merchandising banners, and SEO text authored in EDS documents are *Authored content*, not Commerce build — classify them so they don't inflate the estimate.
- **B2B features assume `commerce-b2b-enabled` + the B2B Compatibility Package.** Note that prerequisite in Dependencies for any B2B row.
- **Don't double-count cross-cutting features.** Header, footer, mini-cart, and auth widgets shared across pages belong in a cross-cutting section, not repeated per page.

## Complexity and risk buckets

Defined in the template preamble — apply them consistently. Do not redefine here.
