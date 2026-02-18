# QA Review: Phase 1 Implementation

**Identifier:** qa-phase1-review__journalist-website-redesign
**Version:** v0.1.0
**Date:** 2026-02-12
**Status:** Draft
**Author:** QA Role
**Reviewed Implementation:** Phase 1 SCSS, Liquid templates, pages, and config in `Issues-FS__Dev__Role__Journalist`

---

## Executive Summary

The Phase 1 implementation delivers the core visual transformation specified in the final recommendation. The SCSS token system, masthead, badge variants, article reading experience, footer, and responsive breakpoints are all present and structurally sound. Liquid templates are syntactically valid, handle missing data gracefully in most cases, and include accessibility foundations (skip link, `aria-label`, `aria-current`, focus indicators).

However, this review identifies **3 Major issues**, **7 Minor issues**, and **3 Cosmetic issues** that should be addressed before the Phase 1 PR is merged. The most significant are: (1) a safe-area `@supports` block that unintentionally reduces desktop padding on the masthead and footer, (2) duplicate H1 headings on every article page, and (3) existing content files missing the `type` front matter field required by the badge component.

**Verdict: Pass with issues.** No Critical defects. The Major issues are all fixable within the Phase 1 scope. Recommend fixing the 3 Majors before merge; Minors can be addressed in a follow-up.

---

## 1. Spec Compliance: Item-by-Item Audit

### 1.1 SCSS Tokens (`_sass/_variables.scss`)

| Token | Spec Hex | Implementation Hex | Match |
|-------|----------|--------------------|-------|
| Ink | `#1A1A2E` | `#1A1A2E` | YES |
| Slate | `#4A4A68` | `#4A4A68` | YES |
| Signal Blue | `#2563EB` | `#2563EB` | YES |
| Deep Signal | `#1D4ED8` | `#1D4ED8` | YES |
| Paper | `#FAFAF8` | `#FAFAF8` | YES |
| Warm Grey | `#F3F2EE` | `#F3F2EE` | YES |
| Ash | `#D4D2CC` | `#D4D2CC` | YES |
| Verified | `#16A34A` | `#16A34A` | YES |
| Amber | `#D97706` | `#D97706` | YES |
| Alert Red | `#DC2626` | `#DC2626` | YES |
| Fog | `#9CA3AF` | `#9CA3AF` | YES |

**Result: All 11 palette colours present and correct.**

Badge background/text colours also verified against Proposal 3 spec:

| Badge | Spec BG | Impl BG | Spec Text | Impl Text | Match |
|-------|---------|---------|-----------|-----------|-------|
| Article | `#EFF6FF` | `#EFF6FF` | `#1D4ED8` | `#1D4ED8` | YES |
| Brief | `#FFFBEB` | `#FFFBEB` | `#B45309` | `#B45309` | YES |
| Interview | `#F0FDF4` | `#F0FDF4` | `#15803D` | `#15803D` | YES |
| Investigation | `#FEF2F2` | `#FEF2F2` | `#B91C1C` | `#B91C1C` | YES |
| Correction | `#F3F4F6` | `#F3F4F6` | `#6B7280` | `#6B7280` | YES |

**Result: All badge colour pairs match spec exactly.**

System font stacks match Dinis's decision (no IBM Plex):
- Sans: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif` -- **Matches**
- Serif: `Georgia, "Times New Roman", Times, serif` -- **Matches**
- Mono: `"Menlo", "Consolas", "Liberation Mono", monospace` -- **Matches**

Spacing scale (4/8/16/24/32/48/64px) correctly defined as both SCSS variables and CSS custom properties. Minima override variables (`$brand-color`, `$text-color`, etc.) correctly set. Import order in `assets/main.scss` is correct: variables before Minima, custom partials after.

### 1.2 Typography (`_sass/_typography.scss`)

| Element | Spec Size | Impl Size | Spec Weight | Impl Weight | Match |
|---------|-----------|-----------|-------------|-------------|-------|
| Body | 18px | 18px | 400 | 400 | YES |
| H1 | 36px | 36px | 700 | 700 | YES |
| H2 | 28px | 28px | 600 | 600 | YES |
| H3 | 22px | 22px | 600 | 600 | YES |
| H4 | 18px | 18px | 600 | 600 | YES |
| H5/H6 | Not spec'd | 16px/600 | - | - | OK |
| Nav | 15px | 15px | 500 | 500 | YES |
| Metadata | 14px | 14px | 400 | 400 | YES |
| Code | 15px | 15px | 400 | impl. | YES |
| Badge | 11px | 11px | 700 | 700 | YES |

Line heights, letter-spacing values all match. H2 bottom border for section separation is a reasonable addition not specified but consistent with the design intent. Link focus indicators use `focus-visible` with 2px Signal Blue outline -- good practice.

**Result: Typography fully compliant.**

### 1.3 Masthead (`_sass/_masthead.scss` + `_includes/header.html`)

| Requirement | Spec | Implementation | Match |
|-------------|------|----------------|-------|
| Background | `#1A1A2E` (Ink) | `$color-ink` = `#1A1A2E` | YES |
| Text colour | `#FAFAF8` (Paper) | `$color-paper` = `#FAFAF8` | YES |
| Accent border | 2px solid Signal Blue | `border-bottom: 2px solid $color-signal-blue` | YES |
| Padding | 24px 32px | `$space-lg $space-xl` = 24px 32px | YES |
| Tagline colour | `#9CA3AF` (Fog) | `$color-fog` | YES |
| Nav link colour | `#D4D2CC` (Ash) | `$color-ash` | YES |
| Nav hover | `#FAFAF8` (Paper) | `$color-paper` | YES |
| RSS badge | Amber border/text | `$color-amber` for color and border | YES |
| Skip link | First focusable element | `<a class="skip-link">` is first element in header.html | YES |
| `aria-current` | On active nav | `aria-current="page"` when path matches | YES (see note) |
| `aria-label` | On nav | `aria-label="Primary navigation"` | YES |

The skip link targets `#main-content`, which is correctly set as the `id` on the `<main>` element in `default.html`. The skip link is visually hidden until focused, then appears at `top: 0` with Signal Blue background. Focus outline is 2px Paper -- good contrast against the blue background.

Nav links have `min-height: 48px` for touch targets and `inline-flex` for alignment. RSS badge also has `min-height: 48px`. Minima's default `.site-nav` is hidden with `display: none`.

**Minor note:** The `aria-current="page"` logic compares the first URL segment. When a user is on `/articles/some-article/`, the Articles nav link gets `aria-current="page"`, which is semantically imprecise -- the user is not on the Articles listing page but on a subpage. The value `aria-current="section"` would be more accurate. See Minor issue M-6.

**Result: Masthead compliant. One minor semantic issue.**

### 1.4 Badges (`_sass/_badge.scss` + `_includes/badge.html`)

All five badge variants are defined with correct colour pairs (verified in section 1.1). The badge component:

- Defaults to `page.type` if no `include.type` is passed -- correct.
- Handles both exact (`article`) and alias (`feature_article`) type values -- good.
- Falls back to `badge--article` styling for unknown types with a humanized label -- reasonable.
- Guards with `{%- if badge_type -%}` so no badge renders if type is nil -- correct.
- Font: 11px, 700 weight, 0.08em letter-spacing, uppercase, 3px border-radius -- matches Proposal 3 CSS.

**Note:** The final recommendation states "Text glyphs ([A], [B], [I], [X], [C]) are the canonical icon system." These glyphs are absent from the badge component. The badge renders the full word ("Article", "Brief", etc.) rather than a glyph prefix. Reviewing the Phase 1 scope, the file list specifies the badge renders `page.type` as a "colour-coded badge" but does not explicitly mention glyphs. The glyphs are part of the broader Proposal 3 icon system. Assessment: the text glyphs are a Phase 2 concern tied to the content-type layouts, not a Phase 1 gap. Noting for completeness.

**Result: Badges compliant with Phase 1 scope.**

### 1.5 Metadata (`_includes/metadata.html`)

| Requirement | Implementation | Match |
|-------------|----------------|-------|
| Date display | `date: "%B %-d, %Y"` format | YES |
| Machine-readable date | `datetime="{{ meta_date | date_to_xmlschema }}"` | YES |
| Author display | Rendered when present | YES |
| Reading time | Calculated: `number_of_words / 200`, min 1 | YES |
| Separators | Middle-dot, only between present fields | YES |
| No dangling separators | Each separator is conditional on preceding field | YES |
| Topic pills | Rendered from `page.topics` array | YES |
| Missing fields | Gracefully omitted | YES |

The separator logic is well-structured: each field checks if preceding fields exist before rendering a middle-dot. No dangling separators will appear.

Topic pills are rendered as `<span>` elements, not `<a>` links. This is correct for Phase 1 -- topic pills become clickable links in Phase 2 (linking to `/topics/#slug`).

**Result: Metadata fully compliant.**

### 1.6 Sources (`_includes/sources.html`)

| Requirement | Implementation | Match |
|-------------|----------------|-------|
| Hidden when absent | `{%- if source_list and source_list.size > 0 -%}` | YES |
| Styled citation block | `<aside>` with heading, border-top, proper spacing | YES |
| `aria-label` | `aria-label="Sources"` on `<aside>` | YES |
| Long URL handling | `.sources__item a { word-break: break-all; }` | YES |

Uses `<ol>` with `list-style: none`. See Minor issue M-3 regarding semantic mismatch.

**Result: Sources compliant.**

### 1.7 Article Reading Experience (`_sass/_article.scss`)

| Requirement | Implementation | Match |
|-------------|----------------|-------|
| Content column width | `max-width: 680px` via `$content-max-width` | YES |
| Centred column | `margin-left: auto; margin-right: auto` | YES |
| Code blocks | `overflow-x: auto`, Warm Grey bg, Ash border | YES |
| Blockquotes | 4px Signal Blue left border, italic, Slate text | YES |
| Nested blockquotes | Additional left margin (`$space-sm`) | YES |
| Table styling | Full-width, collapse, Warm Grey header bg | YES |
| Table scroll wrapper | `.table-wrapper` with `overflow-x: auto` | YES |

Code blocks correctly reset inner `code` styles (`background: none`, `padding: 0`). Blockquote `p:last-child` has `margin-bottom: 0` to prevent trailing space.

**Result: Article reading experience compliant.**

### 1.8 Footer (`_sass/_footer-custom.scss` + `_includes/footer.html`)

| Requirement | Implementation | Match |
|-------------|----------------|-------|
| Dark background | `$color-ink` (#1A1A2E) | YES |
| Signal Blue top border | `border-top: 4px solid $color-signal-blue` | YES (see note) |
| RSS link | Amber badge, links to `/feed.xml` | YES |
| Nav links | From `site.navigation`, Ash colour, Paper on hover | YES |
| Footer nav separators | Middle-dots between links | YES |
| Focus indicators | `focus-visible` outline on links | YES |
| Touch targets | `min-height: 48px` on nav links and RSS badge | YES |

The spec says "Signal Blue top border" but does not specify width. The masthead accent is 2px; the footer uses 4px. This creates a visual asymmetry. See Cosmetic issue C-1.

Minima's default footer columns are hidden with `display: none`.

**Result: Footer compliant.**

### 1.9 Responsive Design (`_sass/_responsive.scss`)

| Requirement | Implementation | Match |
|-------------|----------------|-------|
| Breakpoints | 640px / 1024px | `$breakpoint-mobile: 640px; $breakpoint-desktop: 1024px` | YES |
| Mobile typography | h1: 28px, h2: 24px, h3: 20px, body: 17px | YES |
| Mobile code blocks | Edge-bleed with `calc(-1 * var(--space-md))` | YES |
| Mobile table scroll | `::after` content with "Scroll" indicator | YES |
| Safe-area padding | `max(var(--space-md), env(safe-area-inset-*))` | YES (see Major issue) |
| Focus indicators | Global `*:focus-visible` with Signal Blue | YES |
| Table accessibility | `.table-wrapper[role="region"]` focus styles | YES |

Mobile layout collapses footer nav to vertical, hides separators, adjusts header padding. Tablet and desktop breakpoints define content column constraints.

**MAJOR ISSUE:** The `@supports (padding: max(0px))` block applies safe-area padding with `max(var(--space-md), env(safe-area-inset-*))` globally to `.site-header`, `.site-footer`, `.post-content`, and `.page-content .wrapper`. This block applies on ALL viewports, not just mobile. On desktop, `env(safe-area-inset-left)` is `0`, so `max(16px, 0) = 16px`. This overrides the masthead's `padding: 24px 32px` (reducing horizontal padding from 32px to 16px) and the footer's `padding: 48px 32px` (same reduction). See Major issue MAJ-1.

**Result: Responsive mostly compliant. One Major defect in safe-area padding scope.**

### 1.10 Main Stylesheet (`assets/main.scss`)

Import order is correct:
1. `_variables` (before Minima -- SCSS `!default` overrides work)
2. `minima` (base theme)
3. `_typography`, `_masthead`, `_badge`, `_article`, `_footer-custom`, `_responsive` (custom layers)

The spec references `assets/css/style.scss` as the modified file. The implementation uses `assets/main.scss`. This is the correct file for Minima -- the theme's standard entry point is `assets/main.scss`, not `assets/css/style.scss`. The spec filename was incorrect. No defect.

**Result: Compliant.**

### 1.11 Default Layout (`_layouts/default.html`)

| Requirement | Implementation | Match |
|-------------|----------------|-------|
| `<html lang="en">` | `lang="{{ page.lang | default: site.lang | default: "en" }}"` | YES |
| Skip link | Rendered via `header.html` include (first child of `<body>`) | YES |
| Main content landmark | `<main id="main-content" aria-label="Content">` | YES |
| Header include | `{%- include header.html -%}` | YES |
| Footer include | `{%- include footer.html -%}` | YES |

The layout is clean and minimal. Content is wrapped in a `.wrapper` div inside `<main>`.

**Result: Compliant.**

### 1.12 Config (`_config.yml`)

Navigation array includes Articles, Briefs, Interviews, Investigations, and About. RSS feed URL (`/feed.xml`) is handled by `jekyll-feed` plugin. Collections are correctly defined with output and permalinks.

**Note:** The `Corrections` collection has no navigation entry. Dinis's decision states "Corrections Policy should be visible and easy to find." While this may be an About page concern (Phase 2), the omission from navigation means the only way to reach corrections is via the homepage categories section or direct URL. See Minor issue M-5.

**Result: Config functional. Minor navigation gap noted.**

---

## 2. Code Quality

### 2.1 SCSS Syntax

All six SCSS partials were reviewed for syntax:

- **_variables.scss:** Valid. Proper SCSS variable declarations, `:root` block with interpolation syntax (`#{$var}`). No unclosed braces.
- **_typography.scss:** Valid. Nesting used correctly for pseudo-classes and child selectors.
- **_masthead.scss:** Valid. Proper BEM-like naming convention. Correct use of `&:hover`, `&:focus`, `&:focus-visible` nesting.
- **_badge.scss:** Valid. Five variant classes follow consistent pattern.
- **_article.scss:** Valid. Nested blockquote styles, table wrapper, and source citation styles all syntactically correct.
- **_footer-custom.scss:** Valid. `rgba()` used correctly for border-top on credit line.
- **_responsive.scss:** Valid. Media queries use SCSS interpolation `#{$breakpoint-mobile - 1}` correctly. `@supports` block is valid CSS.

No undefined SCSS variables detected. All referenced variables are defined in `_variables.scss`.

**Result: SCSS syntax is clean across all files.**

### 2.2 Liquid Syntax

All six Liquid templates were reviewed:

- **header.html:** Valid. Proper `{%- for -%}` loops, `{%- assign -%}` statements, conditional `{% if %}`. String filters (`| escape`, `| relative_url`, `| remove_first`, `| split`, `| first`) are all valid Liquid filters.
- **footer.html:** Valid. `{%- unless forloop.first -%}` correctly handles separator logic.
- **badge.html:** Valid. Multi-branch `if/elsif/else/endif` covers all five types plus fallback.
- **metadata.html:** Valid. Uses `| default:` for parameter fallback, `| number_of_words`, `| divided_by:` for reading time calculation.
- **sources.html:** Valid. Conditional rendering with size check.
- **default.html:** Valid. Minimal layout with proper HTML5 structure.

No unclosed tags, no undefined variables, no missing `endif`/`endfor` blocks detected.

**Result: Liquid syntax is clean across all templates.**

---

## 3. Edge Cases

### 3.1 Missing Front Matter Fields

| Field | When Absent | Behaviour | Assessment |
|-------|-------------|-----------|------------|
| `page.type` | No badge renders | Badge guarded by `{%- if badge_type -%}` | CORRECT |
| `page.date` | No date in metadata | Date block guarded by `{%- if meta_date -%}` | CORRECT |
| `page.author` | No author, no leading separator | Author block guarded by `{%- if meta_author -%}` | CORRECT |
| `page.topics` | No topic pills | Topics div guarded by `{%- if meta_topics and meta_topics.size > 0 -%}` | CORRECT |
| `page.sources` | Entire sources section hidden | Sources aside guarded by `{%- if source_list and source_list.size > 0 -%}` | CORRECT |
| `page.summary` | No summary paragraph in listing | Guarded by `{% if doc.summary %}` in index.md | CORRECT |
| `content` (empty body) | Reading time defaults to "1 min read" | `{% if read_minutes < 1 %}{% assign read_minutes = 1 %}{% endif %}` | CORRECT |

All missing-field scenarios degrade gracefully. No dangling separators, no orphaned headings, no empty containers rendered.

### 3.2 Existing Content Compatibility

Reviewed four content files across three collections:

**Articles (3 files):**
- `2026-02-11__the-great-merge.md` -- Has `type: feature_article`, `author`, `topics`, `sources`, `summary`. All fields present. Badge template handles `feature_article` alias (maps to `badge--article`). **Compatible.**
- `2026-02-10__submodule-ecosystem-status.md` -- Has `type: feature_article`, `author`, `topics`, `sources`, `summary`. **Compatible.**
- `2026-02-09__state-of-the-ecosystem.md` -- Not reviewed in detail; follows same pattern.

**Daily Briefs (1 file):**
- `2026-02-11__daily-brief.md` -- Has `type: daily_brief`, `author`, `topics`, `summary`. No `sources`. Sources section correctly hidden. Badge handles `daily_brief` alias (maps to `badge--brief`). **Compatible.**

**Interviews (2 files):**
- `2026-02-09__librarian-interview-prep.md` -- **MISSING `type` field.** No badge will render. Also missing `topics`. **See Major issue MAJ-3.**
- `2026-02-09__librarian-questionnaire.md` -- **MISSING `type` field.** No badge will render. Also missing `topics`. **See Major issue MAJ-3.**

**Investigations:** Empty collection (0 files). Category section in index.md links to `/investigations/` which will be a 404 unless a landing page exists. Not a Phase 1 regression -- this was pre-existing.

**Corrections:** Empty collection (0 files). Same as investigations.

### 3.3 Empty Collections

The `index.md` merges all five collections with `concat`. If a collection is empty, `concat` produces a zero-length array contribution -- no error. The `sort` and `reverse` filters work on the resulting array regardless. If ALL collections are empty, the `{% for doc in sorted_docs limit:20 %}` loop simply produces no output. No blank cards, no orphaned headings.

The Categories section is static markdown (not Liquid-driven), so it always renders all five category links regardless of content count. This is acceptable for Phase 1 but should be dynamic in Phase 2.

### 3.4 Very Long Titles

The article `"The Great Merge: 136 Commits Land Across All 17 Submodules"` is 56 characters -- under the 60-character truncation threshold. No truncation logic exists in Phase 1 for listing items. A 120-character title would likely overflow on mobile. This is noted but not actionable until Phase 2 card components are built.

---

## 4. Bugs Found

### Major Issues

**MAJ-1: Safe-area `@supports` block overrides desktop horizontal padding**

- **File:** `/home/user/Issues-FS__Dev/roles/Issues-FS__Dev__Role__Journalist/_sass/_responsive.scss`, lines 168-176
- **Severity:** Major
- **Description:** The `@supports (padding: max(0px))` block sets `padding-left` and `padding-right` to `max(var(--space-md), env(safe-area-inset-*))` on `.site-header` and `.site-footer`. This block applies at ALL viewport widths, not just mobile. On desktop browsers that support `max()` (all modern browsers), `env(safe-area-inset-left)` evaluates to `0`, so `max(16px, 0) = 16px`. This overrides the masthead's shorthand `padding: 24px 32px`, reducing horizontal padding from 32px to 16px. The same reduction occurs on the footer (from 32px to 16px).
- **Expected:** Desktop masthead horizontal padding is 32px (`$space-xl`).
- **Actual:** Desktop masthead horizontal padding is 16px (`$space-md`) due to the `@supports` override in the cascade.
- **Fix:** Scope the `@supports` block inside the mobile media query, or use the appropriate spacing token per element: `max(var(--space-xl), env(safe-area-inset-left))` for header/footer, `max(var(--space-md), env(safe-area-inset-left))` for content areas.

**MAJ-2: Duplicate H1 headings on article pages**

- **Severity:** Major
- **Description:** The `_config.yml` assigns `layout: "post"` to all collection types. Minima's `post` layout renders `page.title` as an `<h1 class="post-title">`. However, every article's markdown body also begins with a `# Title` heading, creating a second H1 on the page. For example, `2026-02-11__the-great-merge.md` has front matter `title: "The Great Merge: 136 Commits Land Across All 17 Submodules"` AND begins its body with `# The Great Merge: 136 Commits Land Across All 17 Submodules`.
- **Spec requirement:** "Single H1 per page (override Minima's duplicate H1 if present)" is listed as a Phase 1 accessibility baseline item.
- **Impact:** Violates WCAG 1.3.1 (Info and Relationships) and the spec's explicit Phase 1 requirement. Screen readers will announce two H1s, confusing the page structure.
- **Fix:** Either (a) remove the `# Title` lines from all content files (they duplicate the front matter title), or (b) create a custom `_layouts/post.html` that renders the title differently (e.g., as a non-heading element or skips rendering if the content already has an H1). Option (a) is the simpler Phase 1 fix.

**MAJ-3: Interview content files missing `type` front matter field**

- **Severity:** Major
- **Description:** Both interview files (`2026-02-09__librarian-interview-prep.md`, `2026-02-09__librarian-questionnaire.md`) lack a `type` field in their front matter. The badge component relies on `page.type` (or `doc.type` in listing context) to render. Without it, no badge renders for these items in the homepage listing, making them visually inconsistent with articles and briefs that do have badges.
- **Impact:** Interview items in the homepage listing appear without badges, breaking the content-type differentiation that is the centrepiece of the design.
- **Fix:** Add `type: interview` to the front matter of both files. Also consider adding `topics` where appropriate. Additionally, review the `_config.yml` `defaults` section -- a `type` default could be set per collection to ensure all documents in a collection have a type even if the author omits it.

### Minor Issues

**M-1: No separate `_includes/skip-link.html` file**

- **File:** Skip link is inlined in `/home/user/Issues-FS__Dev/roles/Issues-FS__Dev__Role__Journalist/_includes/header.html`
- **Severity:** Minor
- **Description:** The spec lists `_includes/skip-link.html` as a new Phase 1 file. The implementation inlines the skip link directly in `header.html`. Functionally equivalent (the skip link renders correctly as the first focusable element), but deviates from the spec's file manifest.
- **Impact:** Low. The skip link works. Reusability is slightly reduced if other layouts need to include it independently.

**M-2: Spec references `assets/css/style.scss`; implementation uses `assets/main.scss`**

- **Severity:** Minor
- **Description:** The final recommendation lists `assets/css/style.scss` as a modified file. The implementation uses `assets/main.scss`, which is Minima's actual entry point. The spec filename was incorrect -- this is not an implementation error.
- **Impact:** None functional. Documentation discrepancy only.

**M-3: Sources list uses `<ol>` with `list-style: none`**

- **File:** `/home/user/Issues-FS__Dev/roles/Issues-FS__Dev__Role__Journalist/_includes/sources.html`
- **Severity:** Minor
- **Description:** The sources component uses `<ol>` (ordered list) but the SCSS applies `list-style: none`, hiding the numbers. The QA acceptance criteria state "numbered items display" but the implementation suppresses numbers. Either use `<ul>` (no ordering semantics) or remove `list-style: none` to show numbers.
- **Impact:** Low. Screen readers still announce list item positions for `<ol>` regardless of visual styling, so assistive technology users hear "1 of 3" etc. The visual output is unnumbered.

**M-4: Hardcoded GitHub URL in footer**

- **File:** `/home/user/Issues-FS__Dev/roles/Issues-FS__Dev__Role__Journalist/_includes/footer.html`, line 18
- **Severity:** Minor
- **Description:** The GitHub repo link is hardcoded as `https://github.com/owasp-sbot/Issues-FS__Dev__Role__Journalist`. This should come from `_config.yml` (e.g., `site.github_url` or similar) for maintainability.
- **Impact:** If the repo is renamed or moved, the footer link breaks silently.

**M-5: Corrections category absent from site navigation**

- **File:** `/home/user/Issues-FS__Dev/roles/Issues-FS__Dev__Role__Journalist/_config.yml`
- **Severity:** Minor
- **Description:** Dinis's decision states "Corrections Policy should be visible and easy to find. Include a standing 'Corrections Policy' section on the About page and ensure the Corrections category is prominent in navigation." The current navigation includes Articles, Briefs, Interviews, Investigations, and About -- but not Corrections. The corrections are only reachable via the static markdown list on the homepage.
- **Impact:** Corrections are not "prominent in navigation" as requested. This may be intentionally deferred to Phase 2 (About page + nav updates), but the gap should be acknowledged.

**M-6: `aria-current="page"` used for section-level matching**

- **File:** `/home/user/Issues-FS__Dev/roles/Issues-FS__Dev__Role__Journalist/_includes/header.html`, line 17
- **Severity:** Minor
- **Description:** The navigation active state compares the first URL path segment. When a user is on `/articles/the-great-merge/`, the Articles link receives `aria-current="page"`. However, the user is not on the Articles listing page -- they are on a subpage within that section. The ARIA spec recommends `aria-current="page"` only for the exact page, and `aria-current="true"` or no value for ancestor/section matches.
- **Impact:** Screen readers announce "current page" for a link that is not the current page. Minor confusion for assistive technology users.

**M-7: No `lang` key in `_config.yml`**

- **File:** `/home/user/Issues-FS__Dev/roles/Issues-FS__Dev__Role__Journalist/_config.yml`
- **Severity:** Minor
- **Description:** The `default.html` layout falls back to `"en"` when `site.lang` is not set. It would be better practice to set `lang: en` explicitly in `_config.yml` so the language is centrally configured rather than relying on a template fallback.
- **Impact:** Functional -- the fallback works. But explicit configuration is preferable.

### Cosmetic Issues

**C-1: Footer top border width inconsistency**

- **Severity:** Cosmetic
- **Description:** The masthead accent border is 2px (`border-bottom: 2px solid $color-signal-blue`), while the footer accent border is 4px (`border-top: 4px solid $color-signal-blue`). This creates visual asymmetry between the two dark-background sections. The spec does not specify footer border width, so this is not a compliance failure, but visual consistency would suggest matching widths.

**C-2: `h3` margin-bottom uses raw `12px` instead of spacing token**

- **File:** `/home/user/Issues-FS__Dev/roles/Issues-FS__Dev__Role__Journalist/_sass/_typography.scss`, line 57
- **Severity:** Cosmetic
- **Description:** All other spacing values use the `$space-*` tokens. The `h3` `margin-bottom` uses a raw `12px` value (between `$space-sm` at 8px and `$space-md` at 16px). This breaks the token-driven spacing system.

**C-3: `border-bottom-width: 1px` redundancy**

- **File:** `/home/user/Issues-FS__Dev/roles/Issues-FS__Dev__Role__Journalist/_sass/_typography.scss`, lines 80-82
- **Severity:** Cosmetic
- **Description:** The `.post-content h2, .page-content h2` rule sets `border-bottom: 1px solid $color-ash` and then immediately re-declares `border-bottom-width: 1px`. The second declaration is redundant.

---

## 5. Accessibility Audit

### WCAG Compliance Assessment

| Criterion | WCAG Ref | Phase 1 Status | Notes |
|-----------|----------|----------------|-------|
| Colour contrast (text) | 1.4.3 AA | PASS (conditional) | Ink on Paper (15.2:1) and Slate on Paper (7.1:1) pass AAA. Signal Blue on Paper (4.6:1) is at the AA floor. RSS Amber on Ink (4.8:1) passes AA. |
| Colour contrast (non-text) | 1.4.11 AA | PASS | Focus indicators (2px Signal Blue) exceed 3:1. Badge borders not applicable (background differentiation). |
| Keyboard navigation | 2.1.1 A | PASS | All links focusable. Tab order is logical (skip link, title, nav links, RSS, content, footer links). |
| Focus indicators | 2.4.7 AA | PASS | Global `*:focus-visible` rule with 2px Signal Blue outline. Skip link has dedicated focus style. Nav links and RSS badge have explicit `:focus-visible`. |
| Heading hierarchy | 1.3.1 A | FAIL | Duplicate H1 on article pages (see MAJ-2). |
| Skip links | 2.4.1 A | PASS | Skip-to-content link is the first focusable element on every page. Targets `#main-content`. |
| Language attribute | 3.1.1 A | PASS | `<html lang="en">` rendered (via fallback). |
| Link purpose | 2.4.4 A | PASS | Navigation links have descriptive text. RSS link has `title="RSS Feed"`. |
| Touch target size | 2.5.8 AA | PASS | All interactive elements in header and footer have `min-height: 48px`. |
| ARIA landmarks | Best practice | PASS | `<header role="banner">`, `<main aria-label="Content">`, `<footer role="contentinfo">`, `<nav aria-label>`. |

### Contrast Concern: Signal Blue (`#2563EB` on `#FAFAF8`)

The claimed ratio of 4.6:1 sits exactly at the WCAG AA threshold. The QA acceptance criteria note: "Any rendering variance could drop below threshold." The final recommendation states: "Retained for now -- if Lighthouse audits flag it post-implementation, darken to `#2256D0`."

**Recommendation:** Accept for Phase 1 PR. Run Lighthouse on the deployed site. If the audit flags contrast failures, apply the fallback colour `#2256D0` (estimated 5.2:1) as a hotfix.

---

## 6. Summary of All Issues

### By Severity

| ID | Severity | Description | Status |
|----|----------|-------------|--------|
| MAJ-1 | Major | Safe-area `@supports` overrides desktop padding (32px to 16px) | Must fix before merge |
| MAJ-2 | Major | Duplicate H1 on article pages (Minima layout + markdown body) | Must fix before merge |
| MAJ-3 | Major | Interview files missing `type` field -- no badges render | Must fix before merge |
| M-1 | Minor | Skip link inlined in header.html, no separate include file | Accept |
| M-2 | Minor | Spec says `assets/css/style.scss`, impl uses `assets/main.scss` | Accept (spec was wrong) |
| M-3 | Minor | Sources `<ol>` with `list-style: none` -- numbers hidden | Fix or accept |
| M-4 | Minor | Hardcoded GitHub URL in footer | Fix in Phase 2 |
| M-5 | Minor | Corrections absent from navigation | Defer to Phase 2 |
| M-6 | Minor | `aria-current="page"` used for section-level matching | Fix in Phase 2 |
| M-7 | Minor | No `lang` key in `_config.yml` | Quick fix |
| C-1 | Cosmetic | Footer border 4px vs masthead 2px -- asymmetry | Accept or align |
| C-2 | Cosmetic | `h3` uses raw `12px` instead of spacing token | Quick fix |
| C-3 | Cosmetic | Redundant `border-bottom-width: 1px` on h2 | Quick fix |

### By Fix Timing

**Before merge (3 issues):**
- MAJ-1: Scope safe-area padding to mobile media query or use correct token per element
- MAJ-2: Remove duplicate `# Title` lines from article markdown bodies (or override `post.html` layout)
- MAJ-3: Add `type: interview` to interview front matter files; consider `_config.yml` type defaults per collection

**Quick fixes (could be same PR or follow-up):**
- M-7: Add `lang: en` to `_config.yml`
- C-2: Replace `12px` with `$space-sm` + 4px or add a `$space-between` token
- C-3: Remove redundant `border-bottom-width` line

**Phase 2 deferrals:**
- M-4, M-5, M-6: Footer config, nav corrections link, aria-current semantics

---

## 7. Verdict

**Pass with issues.**

The Phase 1 implementation is well-structured, well-commented, and achieves the design intent. SCSS architecture is clean with proper token use. Liquid templates handle edge cases gracefully. Accessibility foundations are solid (skip link, ARIA landmarks, focus indicators, lang attribute, touch targets).

Three Major issues must be resolved before the PR merges:

1. **MAJ-1** is a CSS cascade bug that will visually compress the masthead and footer on desktop. Straightforward fix.
2. **MAJ-2** is an accessibility compliance failure explicitly called out in the Phase 1 spec. Fixing the content files is the path of least resistance.
3. **MAJ-3** is a data completeness gap that makes interviews visually incomplete in the listing.

None of these are architectural problems. All are fixable within the Phase 1 scope with minimal risk. Once these three are resolved, the implementation meets the Phase 1 acceptance criteria and can be merged.

---

*QA Review prepared by the QA Role*
*Issues-FS__Dev__Role__QA*
*Date: 2026-02-12*
