# Dev Review: Journalist Website Redesign Proposal

**Identifier:** review__dev__journalist-website-redesign
**Version:** v0.1.0
**Date:** 2026-02-11
**Status:** Draft
**Reviewer:** Dev Role
**Reviewed Document:** v0.1.0__proposal__journalist-website-redesign.md

---

## Per-Proposal Assessment

### Proposal 1: Visual Identity & Branding

Implementation effort: low (1-2 hours). Create `_sass/_variables.scss` with all CSS custom properties for colours, spacing, and typography. Create `assets/css/style.scss` that imports Minima then overrides via the variables. The font `<link>` tags go into `_includes/head.html` (override Minima's default). Gotcha: Minima's default `$brand-color`, `$text-color` etc. must be explicitly overridden in SCSS -- CSS custom properties alone will not affect Minima's compiled values.

### Proposal 2: Homepage Redesign

Implementation effort: medium-high (3-4 hours). Requires a custom `_layouts/home.html` that replaces Minima's default. The hero section needs Liquid to merge all collections, sort by date, and take the first item: `{% assign all_posts = site.articles | concat: site.briefs | concat: site.interviews | sort: "date" | reverse %}`. The two-column and three-column grids are CSS Grid in a custom `_sass/_homepage.scss`. Gotcha: Liquid's `concat` filter does not exist in older Jekyll versions -- verify the GitHub Pages Jekyll version supports it (3.9.x does).

### Proposal 3: Content Type Differentiation

Implementation effort: medium (4-6 hours total for all five layouts). Each layout (`_layouts/article.html`, `_layouts/brief.html`, etc.) extends a shared `_layouts/post-base.html` that handles masthead, breadcrumbs, metadata, and footer. The badge component is an `_includes/badge.html` partial that reads `page.type` and maps to the colour. Gotcha: the Interview Q&A layout with "alternating backgrounds" has no specification for how Q&A pairs are marked up in the source markdown. This needs a convention (e.g., `> ` for answers, `**Q:**` prefix for questions) and corresponding Liquid/CSS treatment.

### Proposal 4: Navigation & Information Architecture

Implementation effort: medium (3-4 hours). Override `_includes/header.html` for the masthead nav. Breadcrumbs go in a shared `_includes/breadcrumbs.html` partial using `page.collection` and `page.title`. Category landing pages are individual markdown files (`articles.md`, `briefs.md`, etc.) with `layout: collection` and a Liquid loop. Gotcha: topic tag pages require either `jekyll-archives` (not on GitHub Pages whitelist) or manually created markdown files per topic. At scale, manual topic pages are untenable.

### Proposal 5: Article Reading Experience

Implementation effort: low-medium (2-3 hours). Almost entirely CSS. The content column (`max-width: 680px; margin: 0 auto;`), heading scale, code blocks, blockquotes, tables, and source citations are all SCSS in `_sass/_article.scss`. The source citation block needs a Liquid loop in the layout: `{% for source in page.sources %}`. Gotcha: the callout box spec says it lives in layouts, not content -- but there is no specification for how content authors signal "this paragraph is a callout." Needs a Liquid include or a front matter convention.

### Proposal 6: Responsive & Mobile Design

Implementation effort: medium (2-3 hours). Media queries at 640px and 1024px in `_sass/_responsive.scss`. The hamburger menu needs either a JS toggle or a CSS-only `<details>/<summary>` pattern. The mobile typography adjustments and responsive table/code treatments are straightforward SCSS. Gotcha: the "bleed to edge" code block on mobile (`margin-left: -16px`) assumes the parent has exactly `16px` padding. If the content column padding changes, this breaks. Use `calc(-1 * var(--space-md))` with the spacing tokens instead.

### Proposal 7: Interactive Features

Implementation effort varies. RSS visibility: trivial (15 minutes, add a link to `_includes/header.html` and `_includes/footer.html`). Topic filtering: low (1 hour, vanilla JS in `assets/js/filter.js`). Lunr.js search: medium (3-4 hours, `search.json` Liquid template + `assets/js/search.js`). Dark mode: medium (2-3 hours, CSS custom properties + toggle JS). Gotcha: the `search.json` template must escape content properly -- Liquid's `jsonify` filter handles this, but multiline content with special characters can produce invalid JSON if not careful.

---

## Cross-Cutting Assessments

### SCSS Architecture

The proposal references many CSS snippets but does not define a file structure. Recommended SCSS architecture for the `_sass/` directory:

```
_sass/
  _variables.scss       # CSS custom properties, colour maps, spacing tokens
  _typography.scss      # Font imports, heading scale, body text
  _masthead.scss        # Masthead and navigation styles
  _homepage.scss        # Hero, two-column, three-column, category grid
  _badge.scss           # Content-type badge component
  _article.scss         # Article reading experience (column, headings, code, tables)
  _brief.scss           # Brief-specific layout overrides
  _interview.scss       # Interview Q&A styles
  _investigation.scss   # Investigation layout styles
  _correction.scss      # Correction layout styles
  _breadcrumbs.scss     # Breadcrumb navigation
  _footer.scss          # Footer styles
  _responsive.scss      # All media queries, mobile overrides
  _dark-mode.scss       # Dark mode custom property overrides
  _search.scss          # Search page and overlay styles
  _callout.scss         # Callout box variants
```

The main `assets/css/style.scss` imports Minima first, then all custom partials. This ensures Minima's base styles are available for selective override without wholesale replacement.

### What the Actual First PR Looks Like (Phase 1 Scope)

Phase 1 should be a single PR titled something like "Foundation: custom styles, masthead, badges, metadata". The PR contains:

**New files:**
- `_sass/_variables.scss` -- all design tokens
- `_sass/_typography.scss` -- font stack, heading scale
- `_sass/_masthead.scss` -- dark masthead
- `_sass/_badge.scss` -- content-type badges
- `_sass/_article.scss` -- reading experience basics
- `_sass/_responsive.scss` -- initial breakpoints
- `_includes/badge.html` -- badge partial
- `_includes/metadata.html` -- date/author/reading-time partial
- `_includes/sources.html` -- source citation partial

**Modified files:**
- `assets/css/style.scss` -- import chain
- `_includes/head.html` -- font preconnect + stylesheet
- `_includes/header.html` -- masthead redesign + RSS link
- `_includes/footer.html` -- RSS link
- `_config.yml` -- possibly add collection output settings if not already present

**Not in Phase 1** (explicitly deferred):
- Custom `_layouts/*.html` for each content type (Phase 2)
- Homepage redesign (Phase 2)
- Category landing pages (Phase 2)
- Any JavaScript (Phase 2/3)

This scoping keeps Phase 1 to pure SCSS + Liquid includes, no layout overhauls, making it safe to merge and iterate on.

### Font Loading Strategy Recommendation

The proposal specifies Google Fonts CDN. For a GitHub Pages site, I would recommend self-hosting instead:

1. Download WOFF2 files for the 7 specified weights from Google Fonts.
2. Place them in `assets/fonts/`.
3. Define `@font-face` declarations in `_sass/_typography.scss` with `font-display: swap`.
4. Remove the external `<link>` tags entirely.

Benefits: eliminates two third-party DNS lookups, works offline during local development, no GDPR/privacy concerns from Google Fonts. The total WOFF2 payload for 7 weights of IBM Plex is approximately 100-140KB, well within budget for a static site.

If the team prefers the CDN approach for simplicity, at minimum add `<link rel="preconnect">` tags (already specified) and ensure the font CSS request is in `<head>` before the main stylesheet.

### Custom Liquid Templates Needed

The following `_includes/` partials are required by the proposal:

| Partial | Used By | Purpose |
|---------|---------|---------|
| `badge.html` | All layouts, homepage | Renders content-type badge from `page.type` |
| `metadata.html` | All layouts | Date, author, reading time, topic tags |
| `sources.html` | Article, investigation layouts | Source citation block from `page.sources` |
| `breadcrumbs.html` | All article-level layouts | Home > Collection > Title |
| `card.html` | Homepage, collection pages | Reusable article card (badge, title, summary, date) |
| `callout.html` | Article, investigation layouts | Callout box with variant support |
| `topic-pills.html` | Layouts, collection pages | Renders topic tags as clickable pills |
| `hero.html` | Homepage | Featured article hero section |

The following `_layouts/` are required:

| Layout | Extends | Purpose |
|--------|---------|---------|
| `post-base.html` | `default.html` | Shared structure for all content types |
| `article.html` | `post-base.html` | Full article reading experience |
| `brief.html` | `post-base.html` | Compact daily brief |
| `interview.html` | `post-base.html` | Q&A format |
| `investigation.html` | `post-base.html` | Urgent investigation format |
| `correction.html` | `post-base.html` | Minimal correction format |
| `collection.html` | `default.html` | Category landing page |
| `home.html` | `default.html` | Custom homepage |
| `search.html` | `default.html` | Search page (Phase 3) |

### Where the Spec Is Ambiguous and Needs More Detail

1. **Interview Q&A markup convention.** The proposal says "alternating background for questions vs. answers" but does not specify how the markdown distinguishes questions from answers. Without this, the Liquid template cannot parse them. Need: a defined markdown convention (e.g., headings for questions, blockquotes for answers) or structured front matter.

2. **Callout trigger mechanism.** The proposal says callouts live in layouts, not content, but does not specify how a content author signals "insert a callout here." Is it front matter? A Liquid tag in the layout that wraps specific content? This needs a decision before the investigation layout can be built.

3. **Topic page generation.** The proposal lists topic pages as a Phase 3 item but topic tags as clickable links in Phase 2. If tags are clickable in Phase 2, where do they link to? Need: either a single `/topics/` page with anchors (Phase 2) or explicit acknowledgement that tags are not linked until Phase 3.

4. **Collection merging for hero.** The proposal does not specify which collections are merged for the "most recent" hero. All five? Only articles and briefs? If an investigation is the most recent, should it be the hero? Need: a priority order or explicit "all collections" confirmation.

5. **Reading time calculation.** The proposal mentions "8 min read" in mockups but does not specify whether this is front matter or calculated. Need: confirmation that it is calculated via Liquid (`content | number_of_words | divided_by: 200`) or manually authored.

6. **Mobile breakpoint inconsistency.** Proposal 2 says the desktop layout kicks in at `min-width: 768px`, but Proposal 6 defines desktop as `>= 1024px` and tablet as `640px - 1023px`. These conflict. The Proposal 6 breakpoints should be authoritative -- confirm and update Proposal 2.

7. **Search overlay vs. page.** The proposal says search is both a dedicated `/search/` page and "optionally triggers an inline search overlay on desktop." This is two features. Need: a decision on which to build first. Recommendation: build the `/search/` page only; skip the overlay.

---

## Overall Verdict

The proposal is detailed, well-structured, and immediately actionable for Phase 1. The CSS specifications are precise enough to implement directly. The main gaps are in content-authoring conventions (Q&A format, callouts) and a few specification conflicts (breakpoints, topic link targets). These should be resolved before Phase 2 development begins.

### Top 3 Implementation Priorities

1. **Nail the SCSS token foundation.** Get `_variables.scss` right on the first pass -- every colour, every spacing value, every font declaration as a CSS custom property. Every subsequent SCSS file depends on these tokens. If the tokens are solid, the rest of the implementation is mechanical. This is the single highest-leverage file in the entire redesign.

2. **Build the `_includes/` partials before the layouts.** The badge, metadata, sources, card, and breadcrumb partials are reused across every layout and every page. Build and test them in isolation against the current Minima layout before creating custom `_layouts/`. This avoids duplication and ensures consistency from the start.

3. **Resolve the spec ambiguities before Phase 2.** The Interview Q&A convention, callout mechanism, topic link targets, and breakpoint conflict must have definitive answers before any Phase 2 code is written. File these as questions back to the Designer role and block Phase 2 layout work until they are answered.

---

*Review prepared by the Dev Role*
*Issues-FS__Dev__Role__Dev*
*Date: 2026-02-11*
