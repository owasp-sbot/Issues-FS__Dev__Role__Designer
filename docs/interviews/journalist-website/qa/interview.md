# Designer Interview: Journalist Website -- QA Perspective

- **Identifier:** interview__designer__journalist-website__qa
- **Version:** v0.1.0
- **Date:** 2026-02-11
- **Status:** Draft
- **Interviewer:** Designer Role
- **Interviewee:** QA Role (responding from user experience, adversarial testing, and quality perspective)
- **Subject:** Journalist news site at `https://owasp-sbot.github.io/Issues-FS__Dev__Role__Journalist/`

---

## Context

The Designer role is conducting interviews with team roles to gather perspectives on the current state of the Journalist website and inform design improvement decisions. This interview captures the QA role's perspective, focused on user experience defects, usability issues, accessibility concerns, and what breaks or confuses from a visitor's standpoint.

The QA assessment is based on adversarial review of: `_config.yml`, `index.md`, `Gemfile`, all category listing pages, the deploy workflow, all published content files, RSS feed templates, and the absence of any custom layouts, includes, CSS, or assets. The assessment considers the live site URL at `https://owasp-sbot.github.io/Issues-FS__Dev__Role__Journalist/`.

---

## Questions and Responses

### 1. First-time visitor experience -- what do they see?

A first-time visitor arriving at the site URL sees Minima's default `home` layout, which renders two things:

1. **Minima's built-in post list** (if it detects any `_posts`). Since this site uses collections, not posts, this section may be empty or absent -- it depends on whether Minima's `home` layout only iterates `site.posts` or also handles collections. This is an **untested ambiguity** that could produce a blank section at the top of the page.

2. **The custom "Recent Publications" section** from `index.md`, which concatenates all five collections and lists the 20 most recent items as H3 headings with date, collection type, and summary.

**What the visitor does NOT see:**
- Any explanation of what Issues-FS is
- Any explanation of who or what the "Journalist Agent" is
- Any visual branding or identity beyond the site title "Issues-FS News"
- Any visual cues distinguishing content types (an article looks identical to a correction)

**Verdict from a user advocacy perspective:** A first-time visitor with no prior context would see what appears to be a generic blog with dense, technical content. There is no onboarding. There is no "About" page. The phrase "agentic ecosystem" appears in the site description (`_config.yml`) but is never explained on any page. A visitor who arrives from a search engine result or a shared link would have no way to understand the context of what they are reading.

### 2. Is it clear what this site is and who produces content?

**No. This is a significant usability gap.**

- The site title is "Issues-FS News" and the description is "Journalism from inside the Issues-FS agentic ecosystem." Neither of these terms is self-explanatory to someone outside the project.
- Articles are bylined "By the Journalist Agent" or "By: Issues-FS Journalist" with no explanation that this is an AI agent role within an agentic software ecosystem.
- There is no "About" page, no sidebar description, no footer explanation.
- The `_config.yml` does not set `author`, `twitter`, `github_username`, or other Minima social/identity fields that would appear in the footer or sidebar.

**What a confused visitor might conclude:**
- "Issues-FS" could be a company, an open-source project, or an abbreviation they do not recognize
- "Journalist Agent" could be a pseudonym, a team name, or a bot
- The content is clearly technical and well-written, but its provenance is opaque

**Recommendation:** An "About" page is not optional -- it is a basic requirement for any publication site. It should explain: what Issues-FS is, what the agentic ecosystem is, what the Journalist role is, and who (Dinis Cruz / OWASP) is behind the project. Without this, the site fails the most basic "who is telling me this?" trust test that any reader applies to a news source.

### 3. How well does navigation work?

**Navigation is functional but minimal.**

Minima's default header renders links to all pages with a `title` in their front matter. Based on the current pages, the header navigation likely shows: "Issues-FS News" (home), "Articles", "Daily Briefs", "Interviews", "Investigations", "Corrections".

**What works:**
- Category pages exist and link correctly using `relative_url`
- Each category page lists its content in reverse chronological order
- Individual articles render with the `post` layout, which includes title, date, and content

**What does not work well:**
- **No breadcrumbs.** Once a visitor is reading an article, there is no way to see which category it belongs to or navigate back to the category page. The only navigation is the header links and the browser back button.
- **No "back to list" link** on individual article pages.
- **No previous/next navigation** between articles. The original feature request explicitly listed this as a requirement. It was not implemented.
- **Five category links in the header** is already crowding Minima's default navigation. On mobile, this becomes a hamburger menu with five items that all look identical -- no icons, no visual differentiation.
- **The index page's "Categories" section** at the bottom lists hardcoded links like `[Articles](/articles/)`. These paths do NOT include the base URL prefix (`/Issues-FS__Dev__Role__Journalist/`). If `relative_url` is not applied, **these links may be broken on the deployed site.** The articles and other listing links higher on the page use `{{ doc.url | relative_url }}` correctly, but the category links at the bottom of `index.md` are raw markdown links without the Liquid filter. **This is a potential broken link defect.**

### 4. Any usability issues -- confusing labels, unclear hierarchy, dead ends?

**Several issues identified:**

**Issue 1: Inconsistent collection type display.** On the index page, the collection type is displayed via `{{ doc.collection | replace: "-", " " | capitalize }}`. This produces "Daily-briefs" (with the hyphen replaced by a space and only the first letter capitalized), resulting in "Daily briefs" -- but it would produce "Articles" not "Article". The inconsistency between singular and plural depends on the collection name definition in `_config.yml`. "Articles" and "Interviews" are plural; "corrections" is plural; all are rendered as the collection name, not a human-friendly label. This is a minor cosmetic issue but signals a lack of polish.

**Issue 2: Empty category pages with no guidance.** The Investigations and Corrections collections have no content (only `.gitkeep` files). A visitor navigating to `/investigations/` or `/corrections/` sees only the one-line description and then nothing. There is no "No content yet" message, no explanation, no redirect. This is a dead end.

**Issue 3: Feature-requests are invisible and referenced nowhere.** The `feature-requests` collection has `output: false` in `_config.yml`, which is correct (they should not be public pages). But there is also no category page for feature requests, and they are not included in the index page's `all_docs` concatenation. This is internally consistent but worth noting -- feature requests exist in the repo but are completely invisible on the site.

**Issue 4: Article title duplication.** Each article's markdown content starts with an H1 heading that repeats the `title` from the front matter. Minima's `post` layout already renders the front matter title as an H1. This means every article page has **two identical H1 headings** -- one from the layout and one from the article content. This is both a visual defect and an SEO/accessibility issue (multiple H1 elements on a page).

**Issue 5: Hardcoded dates in article bodies.** Articles include bylines like "**By the Journalist Agent** | 2026-02-11" in the markdown body, but Minima's `post` layout already renders the date from front matter. The result is duplicated date display.

### 5. Mobile performance (based on code/theme)?

**Baseline is acceptable. Specific concerns exist.**

Minima includes responsive CSS with breakpoints at 600px and 800px. The basic layout collapses to single-column on mobile and the navigation becomes a hamburger menu.

**Concerns:**

- **Long article titles on mobile.** Titles like "136 Commits, Zero Merges: The Issues-FS Ecosystem Is Building Fast but Not Landing" will wrap across multiple lines in the hamburger menu and in listing pages. No truncation or ellipsis handling exists.

- **Data tables are not responsive.** Articles contain tables with 4-6 columns of data (e.g., the submodule metrics table in the "136 Commits" article). Minima does not apply horizontal scrolling to tables. On a narrow viewport, these tables will overflow the content area and either be clipped or cause horizontal page scroll. This is a known Minima limitation.

- **Code blocks may overflow.** The feature request article includes YAML code blocks. Long lines in code blocks without wrapping will cause horizontal scroll on mobile.

- **Touch target size.** The H3 headings used for article links on listing pages have adequate tap targets due to their font size, but the "Categories" section at the bottom of the index page uses standard markdown links that may be too small for comfortable mobile tapping.

- **No `viewport` meta tag verification.** Minima should include `<meta name="viewport" content="width=device-width, initial-scale=1">` via the `head.html` include, but since no overrides exist, this depends entirely on the Minima version bundled with the `github-pages` gem. This should be verified on the live site.

### 6. Accessibility concerns (contrast, alt text, headings, keyboard nav)?

**Multiple accessibility concerns identified.**

**Heading hierarchy violations:**
- As noted in Issue 4, article pages have **duplicate H1 elements** (one from the Minima layout, one from the article content). This violates WCAG heading hierarchy requirements. Screen readers will announce two H1s, confusing the document structure.
- Within articles, the heading structure jumps from the layout's H1 to H2 sections, which is correct. But the index page renders article titles as H3 (`### [{{ doc.title }}]`) under an H2 ("## Recent Publications"), which is semantically correct but produces a flat, repetitive structure that screen readers must tediously navigate.

**Missing landmark context:**
- No `<main>` landmark annotation beyond what Minima provides by default. Minima's `default.html` layout wraps content in a `<main>` tag, which is good. But the absence of `<nav>` roles on the category list and `<article>` wrappers on individual items in the listing means the page has limited semantic structure for assistive technology.

**Color contrast:**
- Minima's default color scheme generally passes WCAG AA contrast requirements. However, the date/metadata text rendered by `{{ doc.date | date: "%B %-d, %Y" }}` in bold followed by a pipe and the collection name -- if Minima renders metadata in a lighter gray -- may fail contrast checks. This needs live testing.

**No alt text concerns (currently):**
- The site contains zero images. No `<img>` tags are generated. This means there are no alt text violations -- but it also means the site is entirely text-based with no visual interest, which is itself an accessibility concern from a cognitive load perspective (walls of text with no visual breaks).

**Keyboard navigation:**
- Minima's default keyboard navigation (Tab through links, Enter to activate) should work. The main concern is the number of tab stops on the index page: the header links (5-6 items), then 20 article title links, then 5 category links at the bottom. That is 30+ tab stops with no skip-to-content link. Minima may or may not include a skip link depending on version.

**Language attribute:**
- The `_config.yml` does not set `lang`. Minima's default layout should include `<html lang="en">` but this depends on the theme version. If the language attribute is missing, screen readers may default to the wrong language pronunciation.

### 7. How well do content types differentiate visually?

**They do not differentiate at all. This is a significant user experience failure.**

Every content type -- feature article, daily brief, interview questionnaire, correction notice -- renders with the identical `post` layout. There is no visual distinction between:
- A 3000-word investigative feature article
- A 200-word daily brief
- An interview questionnaire sent to another role
- A correction to a previous article

On the index page, the collection type is shown as plain text (e.g., "Articles", "Daily briefs") but there are no visual badges, icons, colors, or layout differences.

**Why this matters:**
- A daily brief should look scannable and short. A feature article should feel substantial. An interview should have a Q&A visual structure. A correction should be visually flagged as an amendment.
- The `type` field in front matter (`feature_article`, `daily_brief`, etc.) exists but is never rendered or used for styling. The `topics` arrays exist but are never displayed.
- The site has five distinct content collections with five distinct purposes. A reader scanning the index page has no visual shortcut to distinguish them. They must read the text label.

**Assessment:** This is the single largest user experience gap on the site. Content type differentiation is a fundamental requirement for a multi-format publication, and currently it does not exist at all.

### 8. QA plan if significant design changes are made?

If the Designer role produces a design specification and the Dev role implements it, QA would execute the following test plan:

**Phase 1: Build verification**
- Verify `jekyll build` completes without errors or warnings on all pages
- Verify no Liquid template errors in build output
- Verify all collections still render correctly after layout changes
- Verify the deploy workflow still succeeds on push to `main`

**Phase 2: Cross-browser testing**
- Test on Chrome, Firefox, Safari (desktop)
- Test on Chrome Mobile, Safari iOS (mobile)
- Verify layout does not break at standard breakpoints (320px, 375px, 768px, 1024px, 1440px)

**Phase 3: Content type verification**
- Verify each content type (article, brief, interview, investigation, correction) renders with its intended design treatment
- Verify edge cases: empty collections, single-item collections, very long titles, articles with tables, articles with code blocks

**Phase 4: Navigation and link integrity**
- Verify all header navigation links resolve correctly
- Verify all category page links resolve correctly
- Verify the category links at the bottom of `index.md` work with the base URL
- Verify RSS feeds still validate after layout changes
- Run `html-proofer` or equivalent link checker against the built site

**Phase 5: Accessibility validation**
- Run axe-core or WAVE on representative pages
- Verify heading hierarchy (exactly one H1 per page)
- Verify color contrast meets WCAG AA
- Test keyboard-only navigation
- Verify skip-to-content link presence and function

**Phase 6: Performance**
- Verify page load time has not regressed significantly
- Verify no large unoptimized assets were introduced
- Check that custom CSS is minified in the build output

**Ongoing regression:** After initial validation, any content push should be spot-checked to verify new articles render correctly with the new design.

### 9. From user advocacy: what is the #1 thing that should change?

**Add an "About" page that explains what this site is, who produces it, and why it exists.**

This is not a design question. It is a trust question.

A publication with no "About" page, no editorial statement, no explanation of its authors, and no organizational context fails the most basic credibility test. Every visitor -- human or AI -- encountering this site for the first time will ask: "Who wrote this? What is Issues-FS? What is a 'Journalist Agent'?"

Currently, the site provides no answers to any of these questions. The site description in `_config.yml` says "Journalism from inside the Issues-FS agentic ecosystem" -- a phrase that is meaningful only to someone already inside the project.

The "About" page should include:
- What Issues-FS is (one paragraph)
- What the agentic ecosystem is and how it works (one paragraph)
- What the Journalist role is and how content is produced (one paragraph)
- Who is behind the project (Dinis Cruz, OWASP SBOT)
- Link to the GitHub repository for transparency

This is a P0 user experience defect. The site is currently publishing content to the public internet with no contextual information about its provenance. No amount of visual design improvement matters if the reader does not know what they are reading or who produced it.

### 10. Any broken links, missing pages, content quality issues in the source?

**Potential broken links:**

- **Category links on the index page.** The "Categories" section at the bottom of `index.md` uses raw markdown links: `[Articles](/articles/)`, `[Daily Briefs](/briefs/)`, etc. These do NOT use Jekyll's `{{ "/articles/" | relative_url }}` filter. On the deployed site at `https://owasp-sbot.github.io/Issues-FS__Dev__Role__Journalist/`, the base URL is `/Issues-FS__Dev__Role__Journalist/`. Raw links like `/articles/` will resolve to `https://owasp-sbot.github.io/articles/` -- which does not exist. **These are likely broken on the live site.** This is a P1 defect.

- **Internal cross-references in articles.** The "State of the Ecosystem" article references file paths like `/home/user/Issues-FS__Dev/roles/Issues-FS__Dev__Role__DevOps/docs/report__2026-02-10__submodule-status.md` -- absolute filesystem paths from the build environment. These are not clickable links and would make no sense to a reader. They should be relative links to the GitHub repository.

**Empty collections with no handling:**
- Investigations: only `.gitkeep` -- the `/investigations/` page renders with a description but no content. No "Coming soon" or empty state.
- Corrections: same situation at `/corrections/`.

**Content quality observations:**
- The three articles are of genuinely high quality: well-researched, properly sourced, structured with clear sections, and editorially separated from factual reporting. The interview questionnaire is thorough and methodical. The daily brief follows a consistent format.
- Front matter is consistently structured across all content types with proper date formatting, summaries, and topic tags.
- One article (`2026-02-10__submodule-ecosystem-status.md`) lists "Six" detached HEAD submodules in the prose but the table above it says "4" -- minor factual inconsistency in the content.

**Missing content that the site structure implies should exist:**
- The interviews collection has two items from 2026-02-09 and two from 2026-02-10, but the 2026-02-10 items are in a subdirectory (`publications/_interviews/2026-02-10/`). Jekyll may or may not discover these depending on the collection configuration. The `_config.yml` defines the interviews collection but does not specify any glob pattern or special directory handling. If Jekyll does not recurse into subdirectories within a collection, **these two interviews may not appear on the site**. This is a P2 defect that needs verification.

**RSS feed correctness:**
- The per-category RSS feeds (`articles/feed.xml`, `briefs/feed.xml`, etc.) use `{{ site.url }}` for the channel link. The `_config.yml` sets `url` to `https://owasp-sbot.github.io/Issues-FS__Dev__Role__Journalist/` (with the repo path). The feed items use `{{ doc.url | absolute_url }}` which should correctly combine `url` and `baseurl`. However, `baseurl` is not explicitly set in `_config.yml`. If GitHub Pages infers it differently from the `url` setting, feed links could be incorrect. This needs live verification.

---

## Key Takeaways

1. **The site lacks identity and context.** No "About" page, no author explanation, no organizational branding. A first-time visitor cannot determine what the site is, who produces the content, or why they should trust it. This is the #1 user experience defect.

2. **Content types are visually indistinguishable.** Five collection types with five distinct purposes all render identically. A 3000-word feature article looks the same as a correction notice. The front matter contains rich metadata (type, topics, sources) that is never displayed.

3. **Likely broken links on the live site.** The category links at the bottom of `index.md` use raw markdown paths without Jekyll's `relative_url` filter. On a site deployed to a subdirectory path, these will resolve to the wrong URLs. This is a testable, fixable defect.

4. **Duplicate H1 headings on every article page.** The Minima layout renders the front matter title as H1, and the article content starts with its own H1. This is both a visual redundancy and an accessibility violation.

5. **Empty collection pages are dead ends.** Investigations and Corrections pages show a description and then nothing. No empty state handling, no "content coming soon" message.

6. **Interviews in subdirectories may not render.** Two interview files are stored in `publications/_interviews/2026-02-10/` rather than directly in `publications/_interviews/`. Jekyll collection behavior with nested subdirectories needs verification.

7. **Tables and code blocks will break on mobile.** The articles contain data tables and YAML code blocks that will overflow narrow viewports. Minima does not handle this natively.

8. **The content quality significantly exceeds the presentation quality.** The articles are well-written, well-sourced journalism. The site presents them as a generic, unstyled blog. From a QA perspective, this is a quality mismatch -- the packaging does not reflect the product.

9. **No local development or testing workflow exists.** Design changes cannot be previewed without pushing to production. This makes iterative design work slow and risky.

10. **A comprehensive QA pass is needed before and after design changes.** The existing site has never been systematically tested for link integrity, accessibility, cross-browser rendering, or mobile behavior. A baseline QA pass should precede any design work so that regressions can be measured.

---

*Interview conducted by the Designer Role*
*Responses provided from the QA Role perspective*
*Date: 2026-02-11*
