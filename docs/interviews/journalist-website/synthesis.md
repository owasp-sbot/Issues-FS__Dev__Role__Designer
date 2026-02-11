# Interview Synthesis: Journalist Website Redesign

**Identifier:** synthesis__journalist-website-redesign
**Version:** v0.1.0
**Date:** 2026-02-11
**Status:** Draft
**Author:** Designer Role
**Depends On:** interview__journalist__news-site-design, interview__architect__news-site-technical-assessment, interview__designer__journalist-website__dev, interview__designer__journalist-website__qa

---

## Overview

This synthesis cross-references findings from four role interviews conducted on 2026-02-11 as part of the Journalist website redesign discovery phase. The interviews captured perspectives from:

- **Journalist** -- site owner, daily content producer, primary workflow user
- **Architect** -- technical authority on Jekyll/GitHub Pages architecture, ADR-001 author
- **Dev** -- implementer, code quality assessor, feasibility evaluator
- **QA** -- adversarial tester, user advocate, accessibility assessor

The purpose of this synthesis is to identify consensus themes where all roles converge, tensions where priorities compete, and a prioritised list of design opportunities to inform the design proposal.

---

## Consensus Themes

These are findings where three or more roles independently identified the same concern, need, or recommendation. Consensus themes represent the strongest signals for design action.

### 1. Content types must be visually differentiated

**Strength of consensus: Unanimous (4/4 roles)**

Every role identified this as a critical gap. The site publishes five content types (daily briefs, feature articles, interviews, investigations, corrections) that serve fundamentally different editorial purposes, but all render identically through the Minima `post` layout.

- **Journalist:** "A 500-token daily brief and a 3000-token feature article with editorial assessment look the same. They should not."
- **Architect:** Provided a complete file structure for per-type layouts (`article.html`, `brief.html`, `interview.html`, etc.) with `_config.yml` defaults mapping each collection to its layout.
- **Dev:** Identified content-type badges as a "quick win" and noted that the `type` field in front matter exists but is never rendered.
- **QA:** Called this "the single largest user experience gap on the site" and detailed how each type should communicate differently.

**Designer assessment:** This is the highest-confidence design opportunity. All four perspectives converge. The data model (front matter metadata) already supports it. The architecture (Jekyll collections and layouts) enables it. The only missing element is design and implementation.

### 2. Content quality significantly exceeds presentation quality

**Strength of consensus: Strong (3/4 roles -- Journalist, Dev, QA)**

Three roles independently observed that the writing is substantive, well-sourced, long-form journalism presented in a generic blog template. The packaging does not match the product.

- **Journalist:** "The site needs to look like a publication, not a software project's documentation."
- **Dev:** "The content on this site is genuinely excellent... The default Minima styling does not do justice to the quality of the writing."
- **QA:** "The content quality significantly exceeds the presentation quality... the packaging does not reflect the product."

**Designer assessment:** This convergence validates the entire redesign effort. The gap between content quality and presentation quality is the core design problem. The solution is not decoration -- it is elevating the form to match the substance.

### 3. The architecture is sound; the theme is the bottleneck

**Strength of consensus: Strong (3/4 roles -- Architect, Dev, QA)**

The Jekyll engine, collections structure, permalink scheme, deployment pipeline, and front matter conventions are all well-designed. The visual limitations come entirely from using the Minima theme without customisation.

- **Architect:** "The gap is in the theme layer, not the engine... The architecture is not the bottleneck. The theme is the bottleneck."
- **Dev:** "The site is a blank canvas. Zero customization files exist."
- **QA:** (Implicitly validated through finding zero architectural defects while documenting numerous presentation defects)

**Designer assessment:** This is encouraging for design scope. The foundation does not need rebuilding. Design work is additive -- creating new `_layouts/`, `_includes/`, and `_sass/` files -- not replacing existing infrastructure. Risk is low; impact is high.

### 4. Rich front matter metadata is underutilised

**Strength of consensus: Strong (3/4 roles -- Journalist, Dev, QA)**

Every content file includes `topics`, `type`, `sources`, and `summary` fields in front matter. The site ignores most of this data.

- **Journalist:** "The `topics` field in front matter goes nowhere... Either the site should use it or I should stop writing it."
- **Dev:** "Rich front matter is underutilized. The data is there; it just needs to be rendered."
- **QA:** "The front matter contains rich metadata (type, topics, sources) that is never displayed."

**Designer assessment:** This is a design opportunity with zero content authoring cost -- the data already exists. Surfacing topics as tags, types as badges, sources as styled citations, and summaries as sub-headlines adds navigational value and visual richness without requiring the Journalist to change workflow.

### 5. An About page is essential

**Strength of consensus: Strong (3/4 roles -- Journalist, QA, Architect)**

The site provides no explanation of what Issues-FS is, who produces the content, or why an AI agent is publishing journalism.

- **Journalist:** "There is no page explaining what this site is, who the Journalist is, what the Issues-FS ecosystem is, or why an AI agent is publishing a news site."
- **QA:** Called this "a P0 user experience defect" and "a basic requirement for any publication site."
- **Architect:** Listed `about.md` among the missing pages needed.

**Designer assessment:** From a design perspective, the About page is not just content -- it is identity. It establishes the site's voice, context, and credibility. The design proposal will include layout and content structure for this page.

### 6. Override Minima; do not replace it

**Strength of consensus: Strong (3/4 roles -- Architect, Dev, QA)**

All technical roles agree that the correct approach is to override Minima through the standard mechanism (`_layouts/`, `_includes/`, `assets/css/style.scss`) rather than switching to a different theme or framework.

- **Architect:** "A custom theme within Jekyll addresses all current design needs."
- **Dev:** "Override Minima. Do not replace it. The standard Minima override path is well-documented, GitHub Pages-native, and avoids external dependencies."
- **QA:** (No dissent; testing plan assumes Minima override approach)

**Designer assessment:** This constrains the design approach productively. The design must work within Minima's structural conventions while overriding its visual presentation. This is appropriate for the site's scale and maturity.

### 7. Mobile and responsive issues need attention

**Strength of consensus: Moderate (2/4 roles with specific details -- Dev, QA)**

Both technical implementation roles identified specific responsive design concerns.

- **Dev:** Identified table overflow, content width, and article listing layout as mobile improvement areas.
- **QA:** Documented long title wrapping, data table overflow, code block overflow, and touch target sizing concerns.

**Designer assessment:** The articles contain data-heavy content (tables, code blocks, metric summaries) that breaks on narrow viewports. Responsive design must be a first-class concern in the proposal, not an afterthought.

### 8. No local preview workflow creates friction

**Strength of consensus: Moderate (3/4 roles -- Journalist, Dev, QA)**

There is no way to see how content or design changes will look before pushing to production.

- **Journalist:** "I cannot see what an article will look like on the site before pushing to `main`."
- **Dev:** "No local preview workflow exists... The only way to see changes is to push to `main` and wait."
- **QA:** "Design changes cannot be previewed without pushing to production."

**Designer assessment:** While this is primarily a DevOps concern, it affects design iteration directly. The design proposal should recommend a preview workflow as a prerequisite for efficient implementation.

---

## Tensions

These are areas where roles have competing priorities, different assessments of importance, or contradictory recommendations. Tensions are not problems to eliminate but trade-offs to navigate deliberately.

### Tension 1: About page vs. content differentiation as top priority

- **QA** positions the About page as the single most important change: "No amount of visual design improvement matters if the reader does not know what they are reading." This is framed as a trust and credibility issue.
- **Journalist** positions content-type differentiation as the highest-impact change: "This is the single highest-impact design change." This is framed as a usability and editorial identity issue.

**Designer resolution:** Both are correct within their own framing. The About page is a content and trust problem solvable with a single page. Content-type differentiation is a systemic design problem requiring layouts, CSS, and templates. The About page should ship first (lower effort, high trust value), but the design proposal should centre on content differentiation as the primary design challenge. These are not mutually exclusive -- they belong to different phases.

### Tension 2: Search now vs. search later

- **Journalist** wants search now: "The ecosystem is producing content rapidly... Within a month there will be 30-50 pieces. Client-side search would make this navigable."
- **Dev** recommends deferring search: "With fewer than 20 articles, manual browsing is sufficient." Recommends starting with reading time and table of contents instead.
- **Architect** classifies search as "moderate complexity" (1-2 days of Dev work).

**Designer resolution:** The Dev's pragmatism is correct for v2 launch. The Journalist's projection is correct for the medium term. The proposal should include search in the design specification but phase it for later implementation. The information architecture should accommodate search from day one (search page in navigation, search index generation) even if the UI ships later.

### Tension 3: Visual identity ambition vs. "earned, not imposed"

- **Journalist** wants the site to have a visual identity but cautions: "it should be earned rather than imposed." Requests a restrained palette with "one accent color."
- **Dev** advocates for the quickest path: "A single `assets/css/style.scss` file with 30-50 lines of overrides would dramatically improve typography." Focuses on incremental improvement.
- **Designer ROLE.md** defines the Designer as owning the full visual language: "colour palette, typography, spacing, component library, iconography, voice and tone."

**Designer resolution:** The Journalist's instinct is sound -- a publication earns its identity through the quality and consistency of its content, not through branding applied externally. However, "earned" does not mean "absent." The proposal will define a complete but restrained visual system: a limited colour palette with semantic meaning, a deliberate typographic hierarchy, and visual tokens that distinguish this site from generic Jekyll. The identity should feel inevitable rather than imposed -- as if the design emerged from the content rather than being applied to it.

### Tension 4: Scope boundary -- Journalist site vs. ecosystem web presence

- **Architect** explicitly warns: "Do not over-engineer v2 of the Journalist site to accommodate hypothetical future needs. Design for the Journalist's content."
- **Journalist** mentions an "emerging external audience" and the site's role as the ecosystem's public face.

**Designer resolution:** The Architect is correct. The design proposal scopes to the Journalist's five content types and workflows. However, the design tokens (colour palette, typography scale, spacing system) should be documented as a nascent design system that could be adopted by future ecosystem web properties. This is not over-engineering -- it is the natural output of principled design work.

### Tension 5: Complexity assessment of interactive features

- **Architect** classifies dark mode, tag filtering, and timeline views as "easy" or "moderate."
- **Dev** classifies previous/next navigation, cross-collection pagination, and search as "hard."
- **QA** flags that every interactive feature must degrade gracefully and be accessible.

**Designer resolution:** The gap is between architectural feasibility and implementation effort. The Architect assesses what Jekyll can support; the Dev assesses how much work it takes to build. The proposal will include interactive features with explicit complexity ratings from both perspectives and will sequence them accordingly.

---

## Prioritised Design Opportunities

Based on consensus strength, tension resolution, and the Designer's assessment of impact-to-effort ratio, the following design opportunities are ranked for the proposal.

### Priority 1 (Must-have for v2)

| Opportunity | Consensus | Impact | Effort |
|---|---|---|---|
| Content-type-specific layouts | Unanimous | Transformative | Medium |
| Custom colour palette and typography | Strong | High | Low |
| Homepage redesign with content hierarchy | Strong | High | Medium |
| About page design | Strong | High | Low |
| Category badges and metadata rendering | Strong | High | Low |
| Source citation styling | Moderate | Medium | Low |
| Responsive table and code block handling | Moderate | Medium | Low |

### Priority 2 (Should-have for v2)

| Opportunity | Consensus | Impact | Effort |
|---|---|---|---|
| Tag/topic pages | Journalist + Dev | Medium | Medium |
| Archive/timeline page | Journalist + Architect | Medium | Medium |
| Dark mode toggle | Architect + Dev | Medium | Low |
| Navigation improvements (breadcrumbs, back-to-list) | QA + Journalist | Medium | Low |
| Empty state handling for collections | QA | Low | Low |

### Priority 3 (Plan for, implement later)

| Opportunity | Consensus | Impact | Effort |
|---|---|---|---|
| Client-side search (Lunr.js) | Journalist + Architect | High (future) | Medium |
| Previous/next article navigation | Journalist + QA | Medium | High |
| Reading time estimates | Dev | Low | Low |
| Table of contents for long articles | Dev | Medium | Low |
| Print stylesheet | Architect | Low | Low |
| Local preview workflow (Makefile) | Dev + QA | Enabling | Low |

---

## Cross-Cutting Observations

### The site has a voice; it needs a face

The Journalist's writing has a distinctive voice -- analytical, sourced, editorially self-aware, and structurally rigorous. The design should amplify this voice, not compete with it. The visual treatment should be restrained enough that the words remain the primary experience, but intentional enough that a reader recognises they are on a news site, not a documentation page.

### The metadata architecture is a design asset

The front matter schema (`type`, `topics`, `sources`, `summary`, `author`, `date`, `slug`) is unusually rich for a Jekyll site. This metadata was written by the Journalist for editorial purposes, but it is also a design asset. Every field is a potential visual element: topics become tag links, types become colour-coded badges, sources become styled citation blocks, summaries become sub-headlines. The design proposal should treat the front matter schema as a design input, not just a data layer.

### The "dual readability" constraint shapes every decision

A hard constraint from ADR-001 is that content must be readable both on the Jekyll site and in the GitHub repository's markdown renderer. This means: no Liquid-heavy templates in content files, no custom shortcodes that break markdown rendering, no design treatments that require HTML in the content body. The design must live in layouts and CSS, not in the content itself. This is a healthy constraint that enforces a clean separation between content and presentation.

### Accessibility is a design requirement, not a testing afterthought

QA identified multiple accessibility concerns: duplicate H1 headings, missing skip-to-content links, potential colour contrast issues, insufficient semantic landmarks. The design proposal must address these as first-class design decisions, not as a QA checklist to satisfy after implementation. Accessible design is good design -- it is not a separate concern.

---

## Recommendation

The four interviews paint a consistent picture: a well-architected, content-rich news site that lacks design intentionality. The foundation is solid. The content is strong. The metadata is rich. What is missing is the design layer -- the layouts, typography, colour, hierarchy, and interactive features that would transform this from "a Jekyll blog with good articles" into "a publication."

The Designer recommends proceeding with a comprehensive design proposal covering all seven priority areas, sequenced into phases for incremental delivery. The proposal should be specific enough that the Dev role can implement against it and the QA role can validate against it -- not abstract principles but concrete specifications with colour values, type scales, layout mockups, and component definitions.

The goal is not to make the site look different. The goal is to make the site work better -- to communicate what it is, to differentiate what it contains, to respect the reader's time and attention, and to give the Journalist's substantive work the presentation it deserves.

---

*Synthesis prepared by the Designer Role*
*Issues-FS__Dev__Role__Designer*
*Date: 2026-02-11*
