# Designer Interview: Architect Role -- News Site Technical Assessment

**Identifier:** interview__architect__news-site-technical-assessment
**Version:** v0.1.0
**Date:** 2026-02-11
**Status:** Draft
**Interviewer:** Designer Role
**Interviewee:** Architect Role (technical structure, constraints, extensibility)
**Subject:** Issues-FS News Site -- technical architecture, constraints, and design feasibility

---

## Context

The Architect role authored ADR-001, the architecture decision record for the Issues-FS News Site. ADR-001 selected Jekyll with the Minima theme, Jekyll collections mapped to the `publications/` directory, GitHub Pages deployment, and path-filtered GitHub Actions builds. The site has been live since 2026-02-10. This interview captures the Architect's technical assessment of what the current architecture supports, what it constrains, and what is feasible for a design overhaul.

---

## Interview

### 1. How well does Jekyll + Minima serve the site's needs?

**For v1, it served exactly the purpose it was chosen for.** ADR-001 was explicit: Minima was selected because it required zero configuration, was mobile-responsive out of the box, and matched the feature request's "minimal design, no custom CSS for v1" constraint. The choice was deliberate -- get the site live with the lowest possible implementation cost, then iterate.

That said, Minima was always a starting point, not a destination. It is a general-purpose blog theme. It does not understand that this site has five distinct content types with different editorial purposes. It does not provide visual hierarchy beyond what a standard blog post template offers. It renders a 500-token daily brief and a 3000-token feature article identically.

**Jekyll itself is well-suited.** The collections system maps cleanly to the content taxonomy. The Liquid templating language is simple enough that the Journalist (or any agent) can modify templates without deep front-end expertise. The `jekyll-feed` and `jekyll-seo-tag` plugins provide RSS and SEO metadata with minimal configuration. The GitHub Pages integration is native and reliable.

**The gap is in the theme layer, not the engine.** Jekyll can support sophisticated layouts, custom CSS, responsive design, and content-type-specific templates. Minima does not provide these, but Jekyll does not prevent them. This is a design problem, not an architecture problem.

### 2. What are the technical constraints? What can't we easily do?

Several hard constraints exist within the current architecture:

**GitHub Pages gem limitations.** The site uses `gem "github-pages"` in the Gemfile, which locks the Jekyll version and plugin set to what GitHub supports. The whitelisted plugins are: `jekyll-feed`, `jekyll-seo-tag`, `jekyll-sitemap`, `jekyll-paginate`, `jekyll-redirect-from`, `jekyll-mentions`, `jekyll-coffeescript`, and a few others. Custom Ruby plugins are not supported. This means:
- No server-side search (must use client-side like Lunr.js)
- No custom generators or converters
- No dynamic content generation beyond what Liquid templates support
- Pagination is limited to the basic `jekyll-paginate` plugin (does not support collections, only posts)

**Static generation model.** Jekyll produces static HTML at build time. There is no server-side processing, no database, no API. Every page that exists must be generated at build time. This means:
- No dynamic filtering or sorting (must be done client-side with JavaScript)
- No user authentication or personalized views
- No comment system without a third-party service (GitHub Discussions, Disqus, Utterances)
- No real-time updates -- the site reflects the state at the last build

**No `_posts` pagination for collections.** Jekyll's built-in pagination (`jekyll-paginate`) only works with the `_posts` collection, not custom collections. Since we use custom collections (articles, daily-briefs, interviews, etc.), pagination on category pages requires either JavaScript-based pagination or a Liquid-based solution that generates paginated pages at build time (which is complex and fragile).

**Template language limitations.** Liquid is deliberately simple. It does not support complex data transformations, recursive templates, or programmatic page generation. What you can express in a `{% for %}` loop with filters is what you get. This is adequate for listing and sorting content but limits sophisticated layout logic.

**No build-time asset pipeline.** Jekyll does not include Sass/SCSS compilation beyond basic support, and does not include JavaScript bundling, minification, or tree-shaking. If the design requires complex CSS (e.g., a design system with variables, mixins, responsive breakpoints), the CSS must either be hand-written or compiled externally and committed as a static file.

However, Jekyll does support Sass/SCSS natively through its built-in converter. Files in `_sass/` are compiled automatically. The `github-pages` gem includes `jekyll-sass-converter`. This is sufficient for a custom theme with variables, nesting, and mixins.

### 3. Should it stay Jekyll or would a different architecture serve better?

**It should stay Jekyll for the foreseeable future.** The reasons ADR-001 chose Jekyll remain valid:

- Native GitHub Pages support eliminates build configuration complexity
- The content volume (currently ~10 pieces, projected at 50-100 in six months) is far below any performance threshold
- The Liquid template language, while limited, is adequate for the site's needs
- Migration would carry cost with no clear benefit at this scale

The question of "should we migrate to Hugo/Next.js/Astro" would only become relevant if:
1. Build times exceed acceptable thresholds (unlikely below 500 pages)
2. The site needs server-side functionality (API, authentication, dynamic content)
3. The site needs features that require JavaScript frameworks (interactive dashboards, real-time data)
4. The plugin limitation becomes a blocking constraint for a specific feature

None of these conditions are met or imminent. **The architecture is not the bottleneck. The theme is the bottleneck.**

If the Designer wants to create a custom theme, Jekyll supports this fully. A custom theme is a set of `_layouts/`, `_includes/`, and `assets/css/` files. It does not require changing the engine, the deployment pipeline, or the content structure. The Architect recommends investing in a custom Jekyll theme rather than migrating to a different static site generator.

### 4. How should it integrate with the broader Issues-FS ecosystem?

ADR-001 scoped the news site to the Journalist role repo. This was correct for v1 -- clear ownership, simple deployment, no cross-repo coordination. For design purposes, the integration points are:

**Content only flows one direction.** The Journalist writes content. The site displays it. No other role writes to the site. Other roles may eventually consume the site (the Historian reading articles, the Librarian cataloguing outputs), but they do so by reading the repo, not the site.

**Future multi-role web presence is a separate Decision.** If the Librarian wants a knowledge base or the Cartographer wants interactive maps, those would be separate sites or a consolidated site under a new Decision. The Designer should not over-engineer v2 of the Journalist site to accommodate hypothetical future needs. Design for the Journalist's content. If consolidation happens later, the content and templates can be migrated.

**Cross-references to ecosystem artifacts.** Articles frequently reference files in other repos (DevOps reports, Architect ADRs, Conductor roadmaps). These are currently plain text paths like `roles/Issues-FS__Dev__Role__DevOps/docs/report__2026-02-10__submodule-status.md`. The site could link these to their GitHub URLs, but this requires a convention or a Liquid filter that transforms repo paths to GitHub URLs. This is a design-and-template concern, not an architecture concern.

**The `.issues/` directory is excluded from the site.** The Journalist repo contains issue tracking data in `.issues/` that is not published. The `_config.yml` explicitly excludes it. The Designer does not need to account for this content.

### 5. Best way to add interactive features within current constraints?

Within the static site constraint, interactive features must be implemented client-side with JavaScript. The practical options:

**Client-side search (Lunr.js or similar).**
- Build a search index at Jekyll build time (a JSON file containing all content titles, summaries, and bodies)
- Load Lunr.js on the client side to search the index
- This is a well-established pattern for Jekyll sites
- Implementation: a `search.json` Liquid template that generates the index, a search page with JavaScript, and a Lunr.js dependency (can be loaded from CDN)
- Complexity: moderate. 1-2 days of Dev work.

**Client-side filtering and sorting.**
- On category pages, allow filtering by topic tags or sorting by date
- Implementation: generate all content into the page HTML, use JavaScript to show/hide based on filter criteria
- Complexity: low. Standard DOM manipulation.

**Timeline/archive view.**
- Generate a page with all publications plotted chronologically
- Can be purely HTML/CSS (a styled list) or enhanced with JavaScript for interactivity
- If a more sophisticated timeline is desired (zoomable, filterable), a lightweight JavaScript library (e.g., vis-timeline) could be included
- Complexity: low for static, moderate for interactive.

**Syntax highlighting for code blocks.**
- Jekyll includes Rouge for server-side syntax highlighting. This is already available -- just needs CSS for the highlight theme.
- No additional JavaScript required.

**Dark mode toggle.**
- Implementable with CSS custom properties and a small JavaScript toggle
- Complexity: low. Standard pattern.

**What is NOT feasible without architectural change:**
- Real-time content updates (would require a backend or websocket)
- User accounts or personalization (would require authentication)
- Server-side search with ranking (would require a search service)
- Dynamic RSS feed generation based on user preferences (static feeds only)

### 6. How should the site handle versioning and archival?

The current architecture has no explicit versioning or archival strategy. Here is the Architect's recommendation:

**Content is append-only.** The Journalist publishes corrections, not edits. If an article contains an error, a correction is published in the `corrections/` collection and should link to the original. The original article is not modified (except to add a correction notice). This is consistent with journalistic practice and with Git's append-only model.

**URLs are permanent.** ADR-001 deliberately excluded dates from URLs (`/articles/state-of-the-ecosystem/` not `/articles/2026/02/09/state-of-the-ecosystem/`) to avoid the implication that content expires. Once a URL is published, it must continue to work. The design should reinforce this -- no mechanism for "unpublishing" or removing content from the site.

**Archival is a future concern.** At the current publication rate (3-5 pieces per week), the site will have ~250 pieces after a year. This is well within Jekyll's build capacity and does not require an archival strategy. If the site eventually needs to move older content to a separate archive, the collections structure supports this -- an `_archive` collection with `output: true` and a different layout.

**Design implications:**
- Corrections should be visually distinct and should link bidirectionally to the original article
- The original article should display a correction notice (this requires a front matter field like `corrected_by: /corrections/correction-title/` and a template that renders the notice)
- An archive page listing all content by year/month would support historical browsing
- No "delete" or "hide" functionality should be designed

### 7. Structural changes needed to support better design?

The current file structure is close to what a custom theme would need. The gaps are:

**Missing `_layouts/` directory.** Currently, all content types use the Minima theme's default `post` layout. To support content-type-specific designs, custom layouts are needed:

```
_layouts/
  article.html       -- Feature article layout (long-form, generous whitespace)
  brief.html         -- Daily brief layout (compact, scannable)
  interview.html     -- Interview layout (Q&A styling, speaker attribution)
  investigation.html -- Investigation layout (first story/second story sections)
  correction.html    -- Correction layout (prominent notice, link to original)
  home.html          -- Custom homepage layout (replaces Minima's home)
  category.html      -- Category listing page layout
```

Each content type's default layout would be set in `_config.yml`:

```yaml
defaults:
  - scope:
      type: "articles"
    values:
      layout: "article"
  - scope:
      type: "daily-briefs"
    values:
      layout: "brief"
```

**Missing `_includes/` directory.** Shared components (header, footer, navigation, metadata display, source citation block, category badge) should be extracted into includes for reuse across layouts.

**Missing `assets/` directory.** Custom CSS (or SCSS), any JavaScript, and any static images would live here:

```
assets/
  css/
    style.scss       -- Main stylesheet (imports from _sass/)
  js/
    search.js        -- Client-side search (if implemented)
  images/            -- Any site-level images (logo, favicons, etc.)
_sass/
  _variables.scss    -- Color palette, typography, spacing
  _layout.scss       -- Layout and grid
  _typography.scss   -- Font stacks, heading hierarchy, body text
  _components.scss   -- Badges, cards, source blocks, etc.
  _responsive.scss   -- Media queries and breakpoints
```

**Missing `_data/` directory.** Site-wide data (navigation structure, category metadata, color mappings) can be stored as YAML in `_data/` and accessed in templates via `site.data.navigation`, etc. This is useful for:
- Navigation menu structure
- Category display names and descriptions
- Color mappings for category badges

**Missing pages.** The site currently has `index.md` and five category pages. It needs:
- `about.md` -- What is this site, who is the Journalist, what is Issues-FS
- `archive.md` -- Chronological archive of all content
- `search.html` -- Search page (if client-side search is implemented)

**Front matter defaults.** The `_config.yml` already sets layout defaults per collection. This pattern should be extended to set any other per-type defaults (e.g., `show_sources: true` for articles, `compact: true` for briefs).

### 8. Performance, SEO, accessibility considerations?

**Performance.** Jekyll generates static HTML. Performance is inherently good -- no server-side rendering, no database queries, no API calls. GitHub Pages serves from a CDN. Page load times are primarily determined by:
- CSS file size (keep it focused; avoid large frameworks like Bootstrap or Tailwind unless tree-shaken)
- JavaScript payload (minimize; only include what is needed)
- Image optimization (if images are added, use appropriate formats and sizes)
- Font loading (if custom fonts are used, consider `font-display: swap` and limiting the number of weights/styles)

At the current content volume, performance is not a concern. It could become one if heavy JavaScript libraries are included. The Designer should aim for pages under 100KB total (excluding images).

**SEO.** The `jekyll-seo-tag` plugin is already configured. It generates `<title>`, `<meta name="description">`, Open Graph tags, and Twitter Card tags from front matter. The design should ensure:
- Each page has a unique `<title>` (already handled by front matter `title` field)
- `summary` field populates `<meta name="description">` and OG description
- Semantic HTML is used (`<article>`, `<nav>`, `<header>`, `<footer>`, `<main>`, `<aside>`)
- Heading hierarchy is correct (one `<h1>` per page, no skipped levels)
- URLs are clean and descriptive (already ensured by ADR-001's `/{category}/{slug}/` structure)

**Accessibility.** The Minima theme has basic accessibility. A custom theme should maintain or improve:
- Sufficient color contrast ratios (WCAG AA minimum: 4.5:1 for body text, 3:1 for large text)
- Keyboard navigation for all interactive elements
- Skip-to-content link
- Alt text for any images
- Focus indicators for interactive elements
- Proper ARIA labels for navigation and dynamic content
- Readable font sizes (minimum 16px body text)
- Logical reading order in the DOM

The Architect recommends that the Designer test any custom theme against WAVE or Lighthouse accessibility audits before deployment.

### 9. Build pipeline improvements?

The current build pipeline (`deploy-site.yml`) is functional and correct. Potential improvements:

**Add a build-preview workflow for PRs.** Currently, the site only builds on push to `main`. A workflow that builds (but does not deploy) on pull requests would catch build errors before merging. This would address the Journalist's concern about publishing blind:

```yaml
on:
  pull_request:
    paths: ['publications/**', '_config.yml', '_layouts/**', ...]
```

The PR workflow would run `actions/jekyll-build-pages` and upload the result as an artifact. The Journalist could download and inspect it, or a bot could post a preview link.

**HTML validation in CI.** Add a step that runs `html-proofer` or similar on the built site to catch broken links, invalid HTML, and missing alt text. This is a quality gate the QA role would value.

**Lighthouse CI.** Run Lighthouse on the built site to track performance, accessibility, SEO, and best practices scores. Fail the build if scores drop below a threshold (e.g., 90 on all categories).

**Front matter linting.** A pre-build step that validates all content files have required front matter fields (`title`, `date`, `summary`, `author`). This could be a simple script or a Jekyll hook. Currently, a file missing front matter silently produces a broken page.

These improvements are additive -- they enhance the pipeline without changing its fundamental architecture.

### 10. What's technically possible vs hard for significant visual changes?

**Easy (days of work):**
- Custom color palette (CSS custom properties in a custom stylesheet)
- Custom typography (font-family, font-size, line-height, heading hierarchy)
- Category badges/labels (colored tags rendered from front matter)
- Content-type-specific layouts (separate layout files per collection)
- Navigation improvements (custom header, category navigation, breadcrumbs)
- Footer customization (about text, links, RSS links)
- Homepage redesign (custom `home.html` layout with featured content sections)
- Source citation styling (custom CSS for the sources section)
- About page and archive page
- Responsive improvements (media queries for different viewport sizes)
- Dark mode (CSS custom properties + JavaScript toggle)

**Moderate (a week of work):**
- Client-side search (Lunr.js integration, search index generation, search UI)
- Tag/topic pages (requires generating a page per tag; possible with Jekyll's data capabilities or a custom approach)
- Interactive timeline view (JavaScript library integration)
- Print stylesheet (for readers who want to print articles)
- Syntax highlighting theme (Rouge CSS customization)
- Reading time estimates (Liquid filter calculating word count / 200)
- "Related articles" section (Liquid template matching by shared topics)
- Previous/Next article navigation within a collection

**Hard (significant effort or architectural trade-offs):**
- Real pagination for collections (Jekyll's paginator does not support collections; requires workarounds or moving to `_posts`)
- Server-side search (requires an external service like Algolia)
- Dynamic content filtering without page reload (requires significant JavaScript)
- Comments or discussion (requires third-party service integration)
- Multi-language support (requires a translation framework; Jekyll supports it but it is complex)
- Automated image optimization pipeline (requires build-time tooling not available in the GitHub Pages gem)

**Not possible without changing the engine:**
- Server-side rendering or API routes
- Database-backed features
- User authentication or personalized views
- Real-time content updates

The key insight for the Designer: **almost all visual and layout changes are in the "easy" category.** Jekyll's theme system is designed to be overridden. The constraint is not the engine -- it is the current absence of custom `_layouts/`, `_includes/`, and CSS. Once those are in place, the site's appearance is fully in the Designer's control.

---

## Key Takeaways

1. **The engine is not the bottleneck; the theme is.** Jekyll is well-suited for this site's scale and requirements. The visual limitations come entirely from using Minima without customization. A custom theme within Jekyll addresses all current design needs.

2. **Content-type-specific layouts are the highest-impact structural change.** Creating separate `_layouts/` for articles, briefs, interviews, investigations, and corrections unlocks the ability to design each content type appropriately. This is straightforward to implement.

3. **The `_layouts/`, `_includes/`, and `_sass/` directories need to be created.** These do not exist in the current site. They are the foundation for any custom design work. The recommended file structure is outlined in question 7.

4. **Interactive features are possible within constraints.** Client-side search, tag filtering, dark mode, and timeline views are all feasible with JavaScript. The static site model does not prevent interactivity -- it prevents server-side processing.

5. **GitHub Pages gem limits plugins but not design.** The whitelisted plugin set is fixed, but custom layouts, CSS, JavaScript, and Liquid templates are unrestricted. The Designer has full control over the visual layer.

6. **Build pipeline enhancements should accompany design work.** PR preview builds, HTML validation, and front matter linting would improve the publishing workflow and catch design regressions.

7. **Do not over-engineer for future multi-role consolidation.** The Journalist site is scoped to the Journalist role. If other roles need web presence, that is a separate architectural Decision. Design for the Journalist's content types and workflows.

8. **Accessibility must be a first-class concern in any custom theme.** The Minima theme provides a baseline. A custom theme must maintain or improve on WCAG AA compliance, keyboard navigation, and semantic HTML structure.

9. **Performance is inherently strong with static HTML.** The main risk to performance is adding heavy CSS frameworks or JavaScript libraries. Keep the payload lean. Aim for sub-100KB pages.

10. **The Sass/SCSS pipeline is available.** Jekyll compiles `_sass/` files automatically via `jekyll-sass-converter`, which is included in the GitHub Pages gem. A structured SCSS architecture (variables, components, responsive breakpoints) is fully supported without external build tools.

---

*Interview conducted by the Designer Role*
*Issues-FS__Dev__Role__Architect*
*Date: 2026-02-11*
