# Designer Interview: Journalist Website -- Dev Perspective

- **Identifier:** interview__designer__journalist-website__dev
- **Version:** v0.1.0
- **Date:** 2026-02-11
- **Status:** Draft
- **Interviewer:** Designer Role
- **Interviewee:** Dev Role (responding from implementation and code quality perspective)
- **Subject:** Journalist news site at `https://owasp-sbot.github.io/Issues-FS__Dev__Role__Journalist/`

---

## Context

The Designer role is conducting interviews with team roles to gather perspectives on the current state of the Journalist website and inform design improvement decisions. This interview captures the Dev role's perspective, focused on implementation quality, code structure, quick wins, and technical feasibility of design changes.

The Dev's assessment is based on a review of: `_config.yml`, `index.md`, `Gemfile`, all category listing pages (`articles.md`, `briefs.md`, `interviews.md`, `investigations.md`, `corrections.md`), the deploy workflow (`.github/workflows/deploy-site.yml`), sample articles from `publications/_articles/`, daily briefs, interviews, feature requests, and the RSS feed templates.

---

## Questions and Responses

### 1. What is the current code quality? Is it well-structured for changes?

The site's code quality is **minimal but clean**. There is very little code to evaluate -- and that is both a strength and a limitation.

**Strengths:**
- `_config.yml` is well-organized with proper collection definitions, permalink patterns, and layout defaults. The five content collections (articles, daily-briefs, interviews, investigations, corrections) plus the non-output feature-requests collection are cleanly defined.
- The Liquid templating in `index.md` is correct: it concatenates all five collections, sorts by date in reverse, and limits to 20 items. This is the right approach for a unified feed.
- The deploy workflow at `.github/workflows/deploy-site.yml` is production-quality: proper path filtering (only triggers on content/config changes), correct permissions scoping, concurrency control, and uses the standard GitHub Actions Jekyll build chain (`actions/jekyll-build-pages@v1`).
- Front matter across all content files is consistent and rich -- `title`, `date`, `summary`, `author`, `slug`, `type`, `topics`, `sources`. This metadata is well-structured for future use even though most of it is not currently rendered.
- The `Gemfile` is minimal (`github-pages` gem only), which means zero dependency management burden.

**Weaknesses:**
- There are **zero customization files**: no `_layouts/`, no `_includes/`, no `_sass/`, no `assets/` directory, no custom CSS. The site is 100% default Minima theme. This means there is no existing customization layer to build on -- any design work starts from scratch.
- All five content types use the same `post` layout from Minima. There is no structural differentiation between a 3000-word feature article and a 100-word correction notice.
- The category listing pages (`articles.md`, `briefs.md`, etc.) are copy-paste duplicates of the same template with only the collection variable name changed. This is not DRY, but given Jekyll's limitations with includes and collection iteration, it is pragmatic.

**Verdict:** The codebase is clean but has no customization surface. It is a blank canvas. Changes will involve adding new files rather than modifying existing ones, which reduces risk of breaking things but means there is more work to do for any visual improvement.

### 2. What are the quick wins for visual improvement with minimal effort?

There are several high-impact, low-effort improvements available:

**Quick win 1: Custom CSS via Minima's built-in override mechanism.** Create `assets/css/style.scss` with:
```scss
---
---
@import "minima";
// Custom overrides here
```
This lets us override any Minima style without replacing the theme. We could immediately improve typography (line-height, max-width for readability, font-size adjustments) with 20-30 lines of CSS.

**Quick win 2: Add article metadata rendering.** The front matter includes `topics`, `type`, and `sources` fields that are never displayed. Overriding the `post` layout to render these would add visual richness and contextual information with minimal template work.

**Quick win 3: Visual badges for content types.** On the index page, `doc.collection` is displayed as plain text (e.g., "Articles"). Adding a CSS class per collection type and styling them as colored badges/labels would immediately help users distinguish between articles, briefs, interviews, etc.

**Quick win 4: Improve the index page heading hierarchy.** Currently, the index page renders each item as an `### h3` heading. Combined with Minima's `home` layout which already renders a post list, this is a missed opportunity. A custom `home` layout override could produce a proper card or summary layout.

**Quick win 5: Add a favicon and site title branding.** Currently uses Minima's default styling with no visual identity. A simple favicon and a customized header with "Issues-FS News" branding would improve professionalism immediately.

### 3. What would be hard to change?

**Hard: Previous/Next navigation between articles.** The original feature request (`2026-02-09__github-pages-news-site.md`) explicitly requested "Previous / Next" navigation. Jekyll collections do not natively support `previous` and `next` variables the way posts do. Implementing this requires either switching content to `_posts` (losing the multi-collection architecture) or writing custom Liquid logic that iterates the sorted collection to find adjacent items. This is non-trivial in Jekyll's template language.

**Hard: Cross-collection navigation with proper ordering.** The index page already does this with `concat` and `sort`, but replicating the same unified chronological view across the entire site (e.g., a global archive page, unified pagination) requires repeated Liquid logic that is verbose and fragile.

**Hard: Search functionality.** The feature request lists client-side search (Lunr.js) as a future extension. Implementing this requires: generating a JSON search index at build time, including the Lunr.js library, and building a search UI. It is feasible but is a full feature, not a quick win.

**Hard: Fundamentally restructuring the URL scheme.** The current permalink patterns (`/articles/:title/`, `/briefs/:title/`) are baked into `_config.yml` and any existing external links or RSS subscribers depend on them. Changing these would break bookmarks and feed entries.

### 4. How easy is it to add custom layouts, components, or styling?

**Easy, because there is nothing to conflict with.**

Minima supports a well-documented override mechanism:
- To override a layout: create `_layouts/post.html` (or `home.html`, `page.html`) and Jekyll will use it instead of Minima's built-in version.
- To override an include: create `_includes/header.html`, `_includes/footer.html`, etc.
- To add custom SCSS: create `_sass/` directory with partial files and import them from `assets/css/style.scss`.

The risk is low because there are currently zero overrides. We are adding, not modifying. The main consideration is that once we start overriding Minima layouts, we take ownership of those templates -- future Minima updates will not automatically apply to overridden files. This is an acceptable tradeoff for a site that needs design differentiation.

**Recommendation:** Start by overriding only `_layouts/post.html` and `_layouts/home.html`, and add custom CSS via the SCSS override. Leave `_layouts/page.html` and the default header/footer includes alone initially.

### 5. Best approach for custom CSS -- override Minima, replace it, or different theme?

**Override Minima. Do not replace it.**

Rationale:
- The `github-pages` gem bundles Minima natively. Replacing it requires either using a `remote_theme` directive (which adds a build dependency on an external repo) or vendoring an entire theme into the repo (which adds maintenance burden).
- Minima provides a solid baseline: responsive grid, sensible typography defaults, dark mode support (in Minima 3.x), syntax highlighting, and SEO tags via `jekyll-seo-tag`.
- The feature request explicitly stated "Default Jekyll theme (minima) or equivalent is fine" and "No custom CSS needed for v1." We are now in v1+ territory where design improvements are warranted, but the foundation should remain.
- The override path (`assets/css/style.scss` importing Minima then adding custom rules) is the standard, documented, lowest-risk approach.

**What I would not do:** Switch to a different theme. Hugo was already considered and rejected in ADR-001. Switching to a different Jekyll theme (e.g., Just the Docs, Minimal Mistakes) would require restructuring all templates and front matter. The cost is not justified when Minima overrides can achieve what we need.

### 6. Can we add client-side interactivity (search, filters)?

**Yes, with constraints.**

GitHub Pages serves static files only -- no server-side processing. All interactivity must be client-side JavaScript.

**Feasible additions:**
- **Client-side search (Lunr.js or Pagefind):** Generate a JSON index at build time, load it in the browser, provide instant search. Pagefind is particularly good for static sites -- it generates a search index during build with no configuration.
- **Category/topic filtering on listing pages:** Add JavaScript that shows/hides articles based on `data-` attributes rendered from front matter topics. The front matter already includes `topics` arrays, so the data is available.
- **Reading time estimates:** Calculate from word count in the template and display on listings.
- **Table of contents for long articles:** Auto-generate from heading structure with JavaScript.

**Constraints:**
- No external API calls from the deployed site (CORS, reliability).
- JavaScript bundles must be small -- GitHub Pages has no CDN optimization.
- All interactivity must degrade gracefully for no-JS visitors (accessibility requirement).

**My recommendation:** Start with reading time and table of contents (simple, high value). Defer search until the content volume justifies it -- with fewer than 20 articles, manual browsing is sufficient.

### 7. What tools/frameworks would make design changes easier?

**For the current setup (Jekyll + GitHub Pages):**
- **Minima source code** as reference: Clone the Minima gem source to understand which includes and layouts to override. The layouts are well-documented.
- **Jekyll's Liquid template language** is the primary tool. It is limited compared to Jinja2 or Nunjucks but adequate for what we need.
- **SCSS** (not plain CSS): Minima uses SCSS and the build pipeline supports it natively. Writing SCSS allows variables, nesting, and mixins.

**For local development and preview:**
- **Docker with `jekyll/jekyll` image** for local builds without installing Ruby. A `docker-compose.yml` with `jekyll serve --livereload` would give instant preview.
- Alternatively, a simple `Makefile` or `scripts/serve-local.sh` that runs `bundle exec jekyll serve`.

**Not recommended:**
- Tailwind CSS or utility-first frameworks -- they require a build step that conflicts with GitHub Pages' standard Jekyll build.
- React/Vue/Svelte components -- overkill for a content site and would break the "just push markdown" workflow.

### 8. How should we handle responsive design?

**Minima already handles responsive design at a basic level.** It includes a responsive navigation hamburger menu, fluid layout, and reasonable breakpoints. The default is functional on mobile.

**What needs improvement:**
- **Reading width:** Minima's default max-width for content is acceptable on desktop but could be more generous. Articles with tables and code blocks benefit from wider content areas.
- **Article listing cards:** The current H3-heading-based listing is not mobile-optimized. A card-based layout with proper spacing would improve touch targets.
- **Tables:** The articles contain several data tables (e.g., the DevOps report metrics). Minima does not apply horizontal scrolling to tables on narrow screens. Custom CSS should add `overflow-x: auto` on table containers.
- **Font sizes:** Minima's defaults are generally fine, but heading sizes may need adjustment for mobile readability.

**Approach:** Use Minima's existing breakpoints (`$on-palm: 600px`, `$on-laptop: 800px`) and extend them with custom media queries only where needed. Do not create a parallel responsive system.

### 9. What testing/preview workflow for design changes?

**Current state: No local preview workflow exists.** There is no `Gemfile.lock`, no `Makefile`, no `docker-compose.yml`, and no documentation for local development. The only way to see changes is to push to `main` and wait for GitHub Actions to deploy (2-3 minutes).

**Recommended workflow:**

1. **Local development:** Add a `Makefile` or `scripts/serve.sh` that runs `bundle install && bundle exec jekyll serve --livereload`. This gives instant preview at `localhost:4000`.

2. **Branch preview:** Create a branch, push it, and use the GitHub Actions workflow to deploy a preview. However, the current deploy workflow only triggers on `main`, so this would require either modifying the workflow or using a separate preview workflow.

3. **PR review:** For significant design changes, open a PR with screenshots or a Netlify Deploy Preview (though this adds an external dependency).

4. **Automated validation:** Add a build step to the CI pipeline that runs `jekyll build` and checks for broken links, missing front matter, and HTML validity. The `html-proofer` gem is standard for this.

**What I would implement immediately:** A `Makefile` with `serve` and `build` targets, and a note in the README about local development prerequisites (Ruby, Bundler, or Docker).

### 10. If you could change one thing right now, what would it be?

**Add `assets/css/style.scss` with typography and layout overrides.**

This single file would deliver the highest visual impact for the lowest effort. Specifically:

```scss
---
---
@import "minima";

.post-content {
  max-width: 42em;
  font-size: 1.05rem;
  line-height: 1.7;
}

.post-meta {
  display: flex;
  gap: 0.5em;
  flex-wrap: wrap;
}

// Content type badges
.collection-badge {
  display: inline-block;
  padding: 0.15em 0.5em;
  border-radius: 3px;
  font-size: 0.8em;
  font-weight: 600;
  text-transform: uppercase;
}
```

The content on this site is genuinely excellent -- the articles are well-researched, well-written, and substantive. The default Minima styling does not do justice to the quality of the writing. Better typography and spacing would immediately make the site feel more professional and more readable, which is the highest-value improvement for the least implementation risk.

The content deserves better than the default theme's generic presentation. As a Dev, I would want the implementation quality of the site to match the editorial quality of the content it hosts.

---

## Key Takeaways

1. **The site is a blank canvas.** Zero customization files exist. All design work is additive, not modification -- which reduces risk but means everything must be built from scratch.

2. **Quick wins are available.** A single `assets/css/style.scss` file with 30-50 lines of overrides would dramatically improve typography and readability. Custom CSS via Minima's override mechanism is the lowest-risk, highest-impact starting point.

3. **Rich front matter is underutilized.** The content already includes `topics`, `type`, and `sources` metadata that could power visual differentiation, filtering, and richer article displays. The data is there; it just needs to be rendered.

4. **Override Minima, do not replace it.** The standard Minima override path (`_layouts/`, `_includes/`, `assets/css/style.scss`) is well-documented, GitHub Pages-native, and avoids external dependencies.

5. **Content quality exceeds presentation quality.** The articles are substantive, well-sourced, long-form journalism. The default Minima styling presents them identically to a generic blog post. The design should elevate the content, not just wrap it.

6. **A local development workflow is missing.** There is no way to preview changes without pushing to `main`. Adding a `Makefile` or serve script is a prerequisite for efficient design iteration.

7. **Some features are inherently hard in Jekyll.** Previous/next navigation across collections, client-side search, and unified cross-collection pagination are non-trivial. These should be planned carefully rather than attempted as quick wins.

8. **The deploy pipeline is solid.** The GitHub Actions workflow is well-configured with proper path filtering and permissions. It does not need changes for design work to land.

---

*Interview conducted by the Designer Role*
*Responses provided from the Dev Role perspective*
*Date: 2026-02-11*
