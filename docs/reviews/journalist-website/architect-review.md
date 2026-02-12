# Architect Review: Journalist Website Redesign Proposal

**Identifier:** review__architect__journalist-website-redesign
**Version:** v0.1.0
**Date:** 2026-02-11
**Status:** Draft
**Reviewer:** Architect Role
**Reviewed Document:** v0.1.0__proposal__journalist-website-redesign.md

---

## Per-Proposal Assessment

### Proposal 1: Visual Identity & Branding

Feasibility is high. CSS custom properties for colours and spacing tokens map cleanly onto a Jekyll SCSS override file. The IBM Plex font family is a solid, well-maintained choice. One risk: the spacing system introduces 7 custom properties that must be adopted consistently across all subsequent proposals -- any deviation creates visual drift. Suggestion: define all tokens in a single `_sass/_variables.scss` partial and enforce imports.

### Proposal 2: Homepage Redesign

The hero + two-column + three-column + category grid layout is ambitious for a Jekyll site with no build-time data aggregation beyond Liquid. The hero "most recent across all collections" requires a Liquid sort across merged collections, which is feasible but produces slow build times as content grows past ~100 items. Risk: the 60/40 grid with heterogeneous card heights can produce awkward whitespace without careful min-height constraints. Suggestion: cap the "Latest Articles" and "Daily Briefs" columns at 3 items each to keep the fold predictable.

### Proposal 3: Content Type Differentiation

This is the highest-value proposal. Five layouts, five badge variants, and five accent colours are all achievable with Liquid's `page.type` front matter field and SCSS maps. Risk is low because each layout is a standalone `_layouts/*.html` file. The one concern is the Interview layout's "alternating background for Q&A" -- this requires either custom Liquid parsing of markdown or a convention in the content files (e.g., using blockquotes for answers). Suggestion: define the Q&A convention in a content-authoring guide before building the template.

### Proposal 4: Navigation & Information Architecture

Top nav, breadcrumbs, footer, and category landing pages are all standard Jekyll patterns. The topic tags linking to topic pages is the trickiest element -- Jekyll does not generate tag pages natively. This requires either `jekyll-archives` plugin (not available on GitHub Pages by default) or a manually maintained page per topic. Risk: topic page generation becomes a maintenance burden. Suggestion: use a `topics.html` page with anchor-linked sections rather than individual pages, at least in Phase 2.

### Proposal 5: Article Reading Experience

Well-specified and entirely CSS-driven. The 680px content column, heading scale, blockquote styles, and source citation block are all implementable as SCSS without any layout changes. The callout box (`<div class="callout">`) is the one element that crosses the "dual readability" line -- it requires HTML in content or layout injection. Risk: minimal. Suggestion: implement callouts as a Liquid include (`{% include callout.html %}`) to keep content files clean.

### Proposal 6: Responsive & Mobile Design

Three breakpoints (640/1024) are reasonable. The hamburger menu requires JavaScript, which contradicts the "works without JS" principle unless a CSS-only toggle (checkbox hack) is used. The "bleed to edge" code block treatment on mobile is a nice touch but requires negative margins that interact poorly with parent padding. Risk: the mobile hamburger is the most JS-dependent feature in the entire proposal. Suggestion: implement the hamburger as a CSS-only disclosure widget (`<details>/<summary>`) for progressive enhancement.

### Proposal 7: Interactive Features

The four features span a wide complexity range. RSS visibility is trivial. Topic filtering is straightforward DOM manipulation. Lunr.js search and dark mode are both medium-effort. Risk: Lunr.js indexing full body text will produce a large `search.json` (potentially 500KB+ at 50 articles), which is a performance concern on GitHub Pages without CDN gzip. Dark mode via CSS custom properties is well-proven. Suggestion: index only title, summary, and topics in Lunr.js -- skip body text.

---

## Cross-Cutting Assessments

### Phased Roadmap Assessment

The three-phase sequencing is well-structured. Phase 1 (foundation) correctly front-loads CSS overrides and visual quick wins before touching layouts. Phase 2 (layouts and navigation) correctly groups the five content-type layouts together so they can share a common base template. Phase 3 (interactivity) correctly defers JavaScript-dependent features.

One sequencing concern: the homepage redesign is in Phase 2, but it depends on content-type badges from Phase 1 and topic filtering from Phase 2. The homepage should be the last item in Phase 2, not the first, to avoid rework as component styles stabilise. Additionally, "Previous/next nav" is marked Complex in Phase 3 but is architecturally simpler than search -- it is a Liquid-only feature. Consider swapping it to Phase 2.

### Font Loading Performance

Loading three font families (IBM Plex Sans, Serif, Mono) with 7 total weight variants from Google Fonts CDN adds approximately 120-180KB of font data. On GitHub Pages, the initial page load will incur:

1. DNS lookup + TLS handshake to `fonts.googleapis.com` (preconnect mitigates this)
2. CSS fetch from Google (small, cacheable)
3. Font file downloads from `fonts.gstatic.com` (the bulk of the payload)

With `font-display: swap`, users see system fonts first, then a reflow when web fonts load. This is acceptable but produces a visible layout shift (CLS impact). Risk: on slow connections, the serif-to-serif swap (Georgia to IBM Plex Serif) is subtle, but the sans-serif swap (system-ui to IBM Plex Sans) may cause noticeable nav/badge reflow.

Recommendations:
- Self-host the fonts as WOFF2 files in the Jekyll `assets/fonts/` directory to eliminate the third-party dependency and reduce connection overhead.
- Subset the fonts to Latin characters only (saves ~40% file size).
- Consider dropping IBM Plex Mono and using the system monospace stack -- code blocks are not frequent enough to justify a third font family.

### Lunr.js Search Approach

Lunr.js is the correct choice for a static site. The proposed lazy-loading strategy (load on first search interaction) is sound. Concerns:

- **Index size**: Indexing body text for 50 articles will produce a JSON payload of 300-500KB. On GitHub Pages without server-side gzip, this is served uncompressed. Users on mobile will feel this.
- **Build time**: Generating `search.json` via Liquid is slow for large collections. At 100+ posts, build times may exceed GitHub Pages' 10-minute limit.
- **Alternative**: Consider generating a pre-built Lunr index at build time (via a custom Jekyll plugin) rather than building it client-side. However, custom plugins are not supported on GitHub Pages -- so client-side index building is the only option without moving to GitHub Actions.

Recommendation: Index title, summary, and topics only. Provide a "full text search" toggle that loads the full index on demand.

### Dark Mode via CSS Custom Properties

The approach is architecturally sound. Defining all colours as CSS custom properties and toggling via a `.dark-mode` class on `<html>` is the standard pattern. The proposed dark palette has reasonable contrast ratios.

Concerns:
- The proposal says "apply before first paint to prevent flash" -- this requires an inline `<script>` in `<head>` that reads `localStorage` before the body renders. This is a well-known pattern but must be placed before the stylesheet link to avoid FOUC.
- The dark accent colour (`#60A5FA`) against the dark background (`#121220`) gives a contrast ratio of approximately 5.8:1 -- passing AA but worth verifying for all combinations.
- Dark mode affects every colour in the system. It should not be attempted until all Phase 1 and Phase 2 styles are stable, which the roadmap correctly sequences.

### Build Pipeline Impact

GitHub Pages uses `jekyll build` with a limited plugin whitelist. The proposal's requirements are compatible:

- **Custom layouts**: Supported natively.
- **SCSS compilation**: Supported via `jekyll-sass-converter` (included by default).
- **Collection iteration**: Supported natively.
- **Lunr.js JSON generation**: Supported via a Liquid template (e.g., `search.json` with front matter).
- **`jekyll-feed`**: Already included.
- **`jekyll-archives`** (for topic pages): NOT in the GitHub Pages whitelist. Topic pages must be generated manually or via GitHub Actions.

The main build-time risk is Liquid template complexity. Merging and sorting multiple collections on the homepage (`site.articles + site.briefs + site.interviews...`) produces O(n log n) Liquid operations that scale poorly. At 200+ items, expect build times over 30 seconds.

Recommendation: If build times become problematic, migrate from GitHub Pages' built-in Jekyll to a GitHub Actions workflow with `jekyll build` -- this removes the plugin whitelist restriction and allows custom plugins for search index generation and topic page creation.

---

## Overall Verdict

The proposal is thorough, well-structured, and implementable within the Jekyll + GitHub Pages constraint. The design decisions are grounded in content needs rather than aesthetic preference, which aligns with the project's ethos. The phased roadmap is realistic.

### Top 3 Recommendations

1. **Self-host fonts.** Eliminate the Google Fonts dependency. Serve WOFF2 files from `assets/fonts/`, subset to Latin. This removes a third-party call, improves load performance, and avoids layout shift from the sans-serif swap. Consider dropping IBM Plex Mono in favour of the system monospace stack.

2. **Simplify topic pages.** Do not build individual topic pages in Phase 2. Instead, use a single `/topics/` page with anchor sections per topic, generated via Liquid. Defer per-topic pages to Phase 3, and only if content volume justifies them. This avoids the `jekyll-archives` dependency and reduces maintenance overhead.

3. **Constrain the Lunr.js index.** Index only title, summary, and topics -- not full body text. This keeps `search.json` under 50KB, eliminates client-side index build latency, and is sufficient for discovery in a corpus of 50-100 items. Full-text search can be added later if needed.

---

*Review prepared by the Architect Role*
*Issues-FS__Dev__Role__Architect*
*Date: 2026-02-11*
