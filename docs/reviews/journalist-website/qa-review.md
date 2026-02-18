# QA Review: Journalist Website Redesign Proposal

**Identifier:** qa-review__journalist-website-redesign
**Version:** v0.1.0
**Date:** 2026-02-11
**Status:** Draft
**Author:** QA Role
**Reviewed Document:** `v0.1.0__proposal__journalist-website-redesign.md`

---

## Proposal-by-Proposal Assessment

### Proposal 1: Visual Identity & Branding

The colour palette is well-specified with hex values that make automated contrast checking straightforward. Claimed WCAG ratios should be independently verified -- the Signal Blue on Paper (`#2563EB` on `#FAFAF8` = 4.6:1) sits right at the AA threshold and could fail with slight rendering differences. Font loading via Google Fonts CDN introduces a third-party dependency; test with CDN blocked to confirm `font-display: swap` fallbacks render acceptably. Edge case: users on high-contrast OS modes may override the palette entirely.

### Proposal 2: Homepage Redesign

The hero section assumes at least one article exists. What renders when the site has zero published content? The two-column 60/40 grid and three-column "More from the Newsroom" section need testing when categories are empty (e.g., zero investigations). The "View all" links must not appear for empty collections. The category grid counts ("12 pieces") must be dynamically accurate -- stale counts are a P1 visual defect. All grid layouts must be validated at the exact breakpoint boundary (768px).

### Proposal 3: Content Type Differentiation

Five badge variants with distinct background/text colour pairs are highly testable -- each can be verified by CSS class presence and computed style. The Unicode icon system (`U+1F4F0`, etc.) has inconsistent rendering across platforms; the proposal correctly recommends text glyphs as fallback. Verify that all five layout templates (`article.html`, `brief.html`, etc.) actually exist and render without errors. The left accent border (4px coloured) is the primary visual differentiator on cards and must be pixel-checked across all five types.

### Proposal 4: Navigation & Information Architecture

The active nav state (2px bottom border in Signal Blue) must correctly reflect the current section on every page, including topic pages and the search page, which do not map to a single category. Breadcrumbs with very long article titles will overflow on narrow viewports -- test with a 120-character title. Topic tag filtering needs keyboard accessibility: users must be able to tab through tags and activate with Enter/Space. Category landing pages need pagination or "load more" strategy when a collection exceeds 50 items.

### Proposal 5: Article Reading Experience

The 680px max-width content column is the core reading experience and must be tested across viewports to confirm centring and gutters. Code blocks with `overflow-x: auto` need testing with lines exceeding 200 characters. Tables inside the 680px column can be wider than their container -- the `.table-wrapper` scroll must be verified. Source citations with extremely long URLs must not break the layout (`word-break: break-all` is specified but needs visual confirmation). Blockquote nesting (blockquote inside blockquote) is unspecified -- test for graceful degradation.

### Proposal 6: Responsive & Mobile Design

Three breakpoints (640px, 1024px) with a hamburger menu below 640px. The hamburger menu overlay at opacity 0.98 must trap focus for keyboard/screen-reader users -- otherwise it is an accessibility blocker. Touch targets are specified at 48x48px minimum; every interactive element must be measured. The responsive table "Scroll" indicator (`::after` pseudo-element) may be missed by screen readers -- needs an `aria-label` or visually-hidden text alternative. Code blocks bleeding to screen edge (`margin-left: -16px`) risk clipping on devices with system-level safe areas (notched phones).

### Proposal 7: Interactive Features

Search, dark mode, and topic filtering all require JavaScript, but the proposal correctly mandates progressive enhancement -- the site must work fully with JS disabled. Lunr.js lazy-loaded on first interaction is good, but test the loading delay on slow connections (3G throttle). Dark mode must persist across page navigations and handle the flash-of-wrong-theme on initial load. Topic filtering via `.hidden` class toggling must update ARIA live regions so screen readers announce the filtered count. RSS badge styling in Amber (`#D97706`) on the dark masthead (`#1A1A2E`) needs contrast verification.

---

## Test Plan Outline

### Phase 1: Visual Checks (Foundation & Quick Wins)

| Test Area | Key Checks |
|-----------|------------|
| Colour palette | All 11 colours render correctly; contrast ratios verified with automated tooling (axe-core or Lighthouse) |
| Typography | IBM Plex family loads; fallback fonts render; weights 400/500/600/700 present; `font-display: swap` active |
| Spacing system | Spacing tokens (4px to 64px) applied consistently; no pixel rounding issues at fractional scaling |
| Masthead | Dark background, inverse text, Signal Blue accent line (2px), tagline in Fog, nav links in Ash with hover to Paper |
| Badges | All five content-type badges render with correct background and text colours; uppercase; correct font size (11px) |
| Metadata rendering | Date, author, reading time display; middle-dot separators; correct font and colour |
| Source citations | Styled citation block renders; long URLs break correctly; numbered items display |
| Responsive tables/code | Horizontal scroll on overflow; "Scroll" indicator visible; code blocks styled |
| RSS link | Visible in masthead and footer; Amber badge styling; links to valid feed URL |

### Phase 2: Layout Checks (Layouts & Navigation)

| Test Area | Key Checks |
|-----------|------------|
| Homepage hero | Most recent article promoted; badge, title, summary, metadata, topic tags all present |
| Two-column grid | 60/40 split on desktop; articles left, briefs right; stacks on mobile |
| Three-column section | Interviews, investigations, corrections each in a column; stacks below 768px |
| Category grid | Correct counts per category; dot colour matches content type; empty categories handled |
| Navigation bar | All links present; active state indicator on current section; keyboard navigable |
| Category landing pages | Reverse-chronological order; card display; item count accurate |
| Breadcrumbs | Correct hierarchy; current page not linked; truncation on long titles |
| Content-type layouts | All five layouts render without errors; accent borders correct; layout-specific features work |
| Topic filtering | Pills displayed; click filters list; "Show all" resets; active pill styled |
| Mobile hamburger | Menu opens/closes; focus trapped inside; slide-down animation; 48px touch targets |

### Phase 3: Interactive Checks (Search, Dark Mode, Filtering)

| Test Area | Key Checks |
|-----------|------------|
| Client-side search | `search.json` generated at build; Lunr.js loads on interaction; results display with highlighted matches |
| Search edge cases | Empty query; single character; special characters; no results state; 50+ results performance |
| Dark mode toggle | Theme switches; colours correct in dark palette; persists in localStorage; respects `prefers-color-scheme` |
| Dark mode flash | No flash-of-wrong-theme on page load; toggle state consistent across pages |
| Topic pages | Cross-collection listing; correct articles displayed; empty topic page handled |
| Print stylesheet | Clean output; no nav/footer; readable fonts; no background colours wasting ink |
| JS disabled | All content accessible; search/dark-mode/filtering degrade gracefully; no blank sections |

---

## Browser / Device Matrix

| Browser | Version | Platform | Priority |
|---------|---------|----------|----------|
| Chrome | Latest stable | Windows / macOS / Linux | P0 |
| Firefox | Latest stable | Windows / macOS / Linux | P0 |
| Safari | Latest stable | macOS | P0 |
| Edge | Latest stable | Windows | P1 |
| iOS Safari | Latest stable | iPhone SE, iPhone 15 | P0 |
| Android Chrome | Latest stable | Pixel 7 / Samsung Galaxy | P0 |
| Screen reader (NVDA) | Latest | Chrome on Windows | P1 |
| Screen reader (VoiceOver) | Latest | Safari on macOS / iOS | P1 |

Test at viewport widths: 320px, 375px, 414px, 640px, 768px, 1024px, 1280px, 1440px.

---

## Accessibility Checklist

| Criterion | WCAG Ref | Test Method |
|-----------|----------|-------------|
| Colour contrast (text) | 1.4.3 AA | Automated (axe-core); manual spot-check Signal Blue on Paper |
| Colour contrast (non-text) | 1.4.11 AA | Badge borders, focus indicators, accent lines meet 3:1 ratio |
| Keyboard navigation | 2.1.1 A | Tab through all interactive elements; verify logical order |
| Focus indicators | 2.4.7 AA | Visible focus ring on all focusable elements; not just browser default |
| Alt text | 1.1.1 A | All images (if any) have descriptive alt text; decorative images use `alt=""` |
| Heading hierarchy | 1.3.1 A | No skipped heading levels; single H1 per page; logical nesting |
| Skip links | 2.4.1 A | "Skip to content" link as first focusable element on every page |
| Reduced motion | 2.3.3 AAA | Hamburger animation and dark-mode transitions respect `prefers-reduced-motion` |
| ARIA live regions | 4.1.3 AA | Topic filter and search results announce updates to screen readers |
| Touch target size | 2.5.8 AA | All interactive elements meet 48x48px minimum (44px WCAG minimum) |
| Language attribute | 3.1.1 A | `<html lang="en">` present |
| Link purpose | 2.4.4 A | "View all" links have context (e.g., "View all articles", not just "View all") |

---

## Content Edge Cases

| Scenario | Expected Behaviour | Risk |
|----------|-------------------|------|
| Title > 100 characters | Truncate with ellipsis on cards; full title on article page | Layout overflow on mobile cards |
| Empty category (0 articles) | Category grid shows "0 pieces"; "More from the Newsroom" omits empty types | Blank cards or broken grid |
| Missing metadata (no date, no author) | Graceful fallback; metadata line shows available fields only, no dangling separators | Rendering artefacts ("Feb 10 __ __") |
| 50+ articles in one collection | Category page remains performant; pagination or lazy-load needed | Page weight; scroll fatigue |
| Article with no summary | Hero and cards display title only; no empty summary block | Inconsistent card heights |
| Special characters in title | `<`, `>`, `&`, quotes, emoji, Unicode rendered correctly; no XSS | Broken HTML or injection |
| Very long topic tag | Tag pill expands gracefully; does not break card layout | Overflow or overlap |
| Article with 20+ topics | Tags wrap to multiple lines; container grows; no overflow | Card height inconsistency |
| No sources in front matter | Sources section hidden entirely; no empty "SOURCES" heading | Visual noise |
| Dark mode + badge colours | Badge colours remain distinguishable against dark backgrounds | Low contrast on dark surface |

---

## Overall Verdict

The proposal is thorough, well-specified, and implementation-ready. Colour values, font sizes, spacing tokens, and CSS snippets provide clear acceptance criteria that QA can validate against mechanically. The phased roadmap is sensible -- Phase 1 delivers visual improvement with low structural risk; Phase 2 introduces layout complexity; Phase 3 adds JavaScript-dependent features.

**Recommendation:** Approve for implementation with the priorities below addressed during each phase.

### Top 3 QA Priorities

1. **Accessibility compliance (all phases).** The Signal Blue accent (`#2563EB` on `#FAFAF8`) sits at exactly 4.6:1 -- the AA floor. Any rendering variance could drop it below threshold. Additionally, the hamburger menu must trap focus, topic filtering must announce changes via ARIA live regions, and skip links must be present from Phase 1. Accessibility must not be treated as Phase 3 polish.

2. **Empty-state and edge-case rendering (Phase 1-2).** The homepage layout assumes populated categories. Zero-content states (empty collections, missing metadata, no summary) must degrade gracefully without blank cards, orphaned headings, or dangling separators. These must be tested before the homepage redesign ships.

3. **Progressive enhancement integrity (Phase 3).** Search, dark mode, and filtering depend on JavaScript. The proposal mandates that the site works without JS, but this must be explicitly tested: disable JS and verify every page renders all content, navigation works, and no sections are blank. JS-off is not an afterthought -- it is the baseline.

---

*QA Review prepared by the QA Role*
*Issues-FS__Dev__Role__QA*
*Date: 2026-02-11*
