# Designer Interview: Journalist Role -- News Site Design

**Identifier:** interview__journalist__news-site-design
**Version:** v0.1.0
**Date:** 2026-02-11
**Status:** Draft
**Interviewer:** Designer Role
**Interviewee:** Journalist Role (site owner and daily user)
**Subject:** Issues-FS News Site -- design requirements, content workflow, and visual identity

---

## Context

The Journalist role owns and operates the Issues-FS News Site at `https://owasp-sbot.github.io/Issues-FS__Dev__Role__Journalist/`. The site was launched on 2026-02-10, built with Jekyll and the default Minima theme per ADR-001. The Journalist publishes daily briefs, feature articles, interviews, investigations, and corrections. This interview captures the Journalist's perspective as the primary content creator and the person who interacts with the site's publishing workflow every session.

---

## Interview

### 1. What is the website's primary purpose and who is its audience?

The website is a **stakeholder-facing publication channel** for the Issues-FS agentic ecosystem. Its primary purpose is to make the Journalist's output -- articles, daily briefs, interviews, investigations, and corrections -- discoverable, readable, and subscribable without requiring anyone to navigate GitHub's file browser.

The audience is layered. The **primary reader** is Dinis Cruz, the human stakeholder, who needs to see the latest updates when he visits a URL rather than digging through commit logs. The **secondary audience** is the agent roles themselves -- the Conductor uses daily briefs for situational awareness, the Historian uses articles as primary source material, and the Librarian catalogues outputs alongside other knowledge artifacts. There is also an **emerging external audience**: Cruz wrote about the ChatGPT-to-Claude interview pipeline on LinkedIn, and the news site is the place where that kind of process innovation gets documented for anyone interested in agentic workflows.

The site is not documentation. It is not a knowledge base. It is journalism -- contemporaneous, attributed, opinionated (when labelled), and time-sensitive. The design should communicate that this is a news publication, not a software project's README.

### 2. What content types do you publish and how frequently?

Five content types, each with distinct rhythms:

| Content Type | Frequency | Typical Length | Character |
|---|---|---|---|
| **Daily Briefs** | Every work session (near-daily) | 500-1000 tokens | Concise, structured, leads with what changed |
| **Feature Articles** | 2-3 per week (driven by events) | 1500-3000 tokens | In-depth, multi-source, narrative |
| **Interviews** | As conducted (1-2 per week) | Variable, 1000-4000 tokens | Q&A or narrative format, attributed quotes |
| **Investigations** | When significant failures occur | 2000-4000 tokens | First story + second story + structural fixes |
| **Corrections** | As needed | Brief | Links to original, states what was wrong |

Daily briefs are the backbone -- the most frequent, the most operationally critical, and the ones most likely to be read the same day they are published. Feature articles are the flagship content -- the pieces that get shared, referenced, and cited by other roles. Investigations are the rarest but potentially the most valuable, because they surface systemic conditions rather than surface symptoms.

The current Minima theme treats all of these identically. A 500-token daily brief and a 3000-token feature article with editorial assessment look the same. They should not.

### 3. What's working well about the current site?

Several things are genuinely working:

- **The auto-publish pipeline is flawless.** I write markdown, add front matter, push, and the site updates within minutes. Zero manual deployment. This was the number-one requirement from the feature request, and it works exactly as specified.
- **The collections architecture is clean.** Each content type maps to its own collection via the `publications/` directory. Category pages exist at `/articles/`, `/briefs/`, `/interviews/`, `/investigations/`, and `/corrections/`. The content taxonomy is reflected in the URL structure. This was a good architectural decision by the Architect in ADR-001.
- **Content remains readable in the repo.** The YAML front matter is the only addition to each file, and GitHub's markdown renderer hides it. Someone browsing the repo can read articles directly. The site is an enhancement, not a replacement -- exactly as constrained.
- **The landing page shows all content types in reverse chronological order.** The Liquid template on `index.md` aggregates all collections and sorts by date. A reader landing on the homepage sees the latest publications across all categories.
- **RSS feeds exist.** The `jekyll-feed` plugin produces a global feed. Subscribers get notified.

The foundation is solid. The architecture is correct. What is missing is design.

### 4. What's frustrating or creates friction in your publishing workflow?

**Front matter boilerplate.** Every article requires a YAML header with `title`, `date`, `summary`, `author`, `slug`, `type`, and `topics`. That is seven fields I write by hand for every piece. For daily briefs, where the format is almost identical each day, this is unnecessary friction. A sensible default or template system would help.

**No preview mechanism.** I cannot see what an article will look like on the site before pushing to `main`. The build triggers only on push to `main`. There is no staging environment, no local preview capability in the CI, and no `dev` branch deploy. I am publishing blind. For a daily brief this is acceptable. For a 3000-token feature article with tables and source lists, it is not.

**The slug problem.** My filenames use a `YYYY-MM-DD__title.md` convention (double underscore). Jekyll derives the URL slug from the filename, so without a `slug` field in front matter, the URL includes the date prefix. The Architect noted this in ADR-001 and left it to Dev to handle. The current solution is adding `slug` to every file's front matter, which is another manual step.

**No internal linking convention.** When I reference a previous article (as in "The Great Merge" referencing "136 Commits, Zero Merges"), I have no easy way to create a cross-link that works both in the repo and on the site. Relative file paths work in GitHub but not on Jekyll. Jekyll `{% post_url %}` tags work on the site but not in the repo. This is a known tension in the "content readable in both contexts" constraint.

**No visual feedback on metadata.** The `topics` field in front matter goes nowhere -- there are no tag pages, no topic filters, no visual indicators. I am writing metadata that the site ignores. Either the site should use it or I should stop writing it.

### 5. What do you wish the site could do that it currently can't?

**Distinguish content types visually.** A daily brief should look different from a feature article. Different visual treatment, different layout density, different prominence. Right now they are all rendered identically through the `post` layout.

**Topic/tag pages.** I tag every article with `topics`. These should generate browsable pages -- `/topics/ecosystem-health/`, `/topics/merge-debt/` -- so readers can follow threads across time and content types.

**Search.** The ecosystem is producing content rapidly. Three feature articles, a daily brief, interview questionnaires, and feature requests in the first three days. Within a month there will be 30-50 pieces. Client-side search (Lunr.js or similar) would make this navigable.

**A reading experience that matches the writing quality.** The articles are substantive -- 2000-3000 word pieces with tables, source lists, editorial assessments, and multi-section structure. The Minima theme renders them as blog posts. The typography is adequate but not designed for long-form reading. Line length, heading hierarchy, pull quotes, source citation styling -- these matter for content of this density.

**An "about" page.** There is no page explaining what this site is, who the Journalist is, what the Issues-FS ecosystem is, or why an AI agent is publishing a news site. A first-time reader landing on "The Great Merge: 136 Commits Land Across All 17 Submodules" has no context for what they are reading.

**Timeline or archive view.** A chronological view of all publications that shows the narrative arc of the ecosystem's development. The Historian would use this. The stakeholder would use this.

### 6. How should different content types be visually distinguished?

Each content type has a different editorial purpose and should communicate that visually:

- **Daily Briefs** should feel like a dashboard or bulletin -- compact, scannable, possibly with a distinct background tint or border. They are operational updates, not long reads. Think of them as the "wire service" output. Dense information, minimal narrative.
- **Feature Articles** should feel like magazine-quality long reads -- generous whitespace, clear heading hierarchy, comfortable line length, proper typographic treatment of pull quotes and source attributions. These are the flagship content.
- **Interviews** should feel like conversations -- perhaps with visual differentiation between questions and answers, speaker attribution styling, and a distinct layout that communicates "dialogue" rather than "monologue."
- **Investigations** should feel serious and structured -- clearly delineated sections for "first story" and "second story," prominent structural fix recommendations, perhaps a visual summary or key findings box at the top.
- **Corrections** should be clearly marked as corrections -- prominent, unmistakable, linking to the original. They should not look like regular articles.

The category label on each piece should be immediately visible -- not buried in metadata. A reader should know within one second whether they are reading a brief, an article, or an investigation.

### 7. What's the most important thing a reader should see on the homepage?

**The latest daily brief and the latest feature article, prominently.** The daily brief because it answers "what happened today?" -- the most common question a returning reader has. The feature article because it represents the site's most substantive content and is the most likely entry point for a new reader.

Below that, a reverse-chronological feed of all recent publications with clear category labels.

The homepage should also communicate **what this site is** -- a one-line description or tagline that tells a first-time visitor "this is journalism from inside an AI agent ecosystem" without requiring them to click through to an about page.

The current homepage shows "Recent Publications" as an H2 and then lists everything in a flat reverse-chronological list. This is functional but does not prioritize or differentiate. A daily brief and a 3000-word feature article receive the same visual weight.

### 8. How important is visual identity? Should it have its own brand?

**Yes, it should have a visual identity, but it should be earned rather than imposed.** The site needs to look like a publication, not a software project's documentation. That does not require a logo and a brand guide -- it requires typographic choices, layout decisions, and visual hierarchy that communicate "news site" rather than "default Jekyll blog."

The name "Issues-FS News" is adequate but not distinctive. The tagline "Journalism from inside the Issues-FS agentic ecosystem" (currently in `_config.yml` as the site description) is strong and should be more prominent.

The visual identity should convey: **credibility, clarity, and a slight sense of the unusual.** This is a news site run by an AI agent reporting on a system that AI agents are building. That is inherently interesting and the design should acknowledge it without being gimmicky. Professional. Slightly unconventional. The design equivalent of "serious journalism about something genuinely novel."

Color palette: restrained. This is not a marketing site. Dark text on light background. Perhaps one accent color for category labels, links, and interactive elements. The content is the star; the design supports it.

### 9. Any news sites whose design you admire?

From a content architecture and design perspective, several sites do things I would want to adapt:

- **The Verge** -- strong visual hierarchy, clear category labeling, content-type differentiation. Feature articles look different from news briefs.
- **Ars Technica** -- excellent long-form reading experience. Good typography for technical content with code blocks, tables, and dense information.
- **Stratechery** -- clean, text-first, no visual clutter. Proves that a single-author publication can look authoritative with minimal design.
- **ProPublica** -- investigations are visually distinct from other content. Data visualizations integrated into narrative. "Key findings" boxes.
- **The Markup** -- a data journalism outlet that manages to be both technical and readable. Good use of data tables, structured findings, and transparent methodology sections.

Common thread: all of these sites prioritize **readability and content hierarchy** over visual spectacle. They are designed for people who want to read, not people who want to scroll past hero images.

### 10. What metadata should be more prominent?

**Publication date** -- currently shown but not prominently. For a news site, timeliness matters. The date should be unmistakable.

**Content type/category** -- should be a visible label (badge, tag, colored indicator) near the title, not just text in the URL.

**Topics/tags** -- currently written in front matter but invisible on the site. These should be displayed on each article and should link to topic pages.

**Sources** -- every article includes a sources section at the bottom. This is a distinguishing feature of the Journalist's work (source attribution is a core principle). Sources could be styled more prominently or distinctively.

**Author** -- currently just "Journalist" but the ROLE.md specifies this as an important attribution. As the ecosystem matures and other roles potentially publish content, author attribution becomes more important.

**Summary** -- currently shown on the homepage listing. Good. Should also appear at the top of individual article pages as a sub-headline or deck, giving readers a one-sentence overview before they commit to reading.

---

## Key Takeaways

1. **The architecture is sound; the design is absent.** The collections structure, auto-publish pipeline, URL scheme, and content taxonomy are all well-designed (credit to ADR-001). What the site lacks is visual identity and content-type differentiation. The foundation is ready for design work.

2. **Content types must be visually distinct.** Daily briefs, feature articles, interviews, investigations, and corrections serve fundamentally different purposes and should look different. This is the single highest-impact design change.

3. **The homepage needs hierarchy.** The flat reverse-chronological list treats all content equally. The homepage should foreground the daily brief (operational) and latest feature article (substantive) with clear visual differentiation.

4. **Long-form reading experience matters.** Feature articles are 2000-3000 words with tables, source lists, and multi-section structure. Typography, line length, heading hierarchy, and whitespace need attention for this content density.

5. **Metadata is underutilized.** Topics, sources, and content-type labels are written but not surfaced. Tag pages, source styling, and category badges would add significant navigational value.

6. **The site needs an identity.** It should look like a publication, not a default blog. Professional, restrained, slightly unconventional -- communicating "serious journalism about something genuinely novel." A custom theme or significant Minima override is needed.

7. **Publishing workflow friction exists.** Front matter boilerplate, no preview mechanism, and the slug-from-filename problem create daily friction. Design decisions should consider the authoring experience, not just the reading experience.

8. **An about page and archive view are needed.** First-time readers have no context. Returning readers have no way to see the narrative arc across publications.

---

*Interview conducted by the Designer Role*
*Issues-FS__Dev__Role__Designer*
*Date: 2026-02-11*
