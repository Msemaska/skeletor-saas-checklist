# Post-Launch Pages

---

## What It Is

A set of high-intent marketing pages built after launch, once the product has real positioning, real customers, and real competitors to compare against. These pages — comparison/alternative, use-case/solution, and demo/screenshots — serve buyers who are actively evaluating options, not just discovering the product. They differ from the core launch pages (phase-6-marketing-site/01-marketing-site-pages.md) because their value depends on live product context: you cannot write an honest comparison without knowing where you win, and you cannot write a credible use-case page without knowing who actually uses you and why.

**Scope boundary:** This file covers page structure, layout decisions, URL scheme, navigation placement, technical SEO implementation, and internal linking for these post-launch page types. It does not cover the content strategy, copy production, keyword targeting, or update workflow for comparison and alternative pages — those live in phase-13-content/06-comparison-content.md. Programmatic or templated variants of use-case and integration pages are deferred to phase-13-content/09-programmatic-seo.md.

---

## Comparison and Alternative Pages

### URL Structure

- Comparison pages: `/compare/[competitor-slug]` — one page per major competitor
- Alternative pages: `/compare/[competitor-slug]-alternative` — built for top 3 competitors only

Keep both under `/compare/` so the pattern is recognisable and the sitemap segment is easy to manage.

### Page Layout

Each comparison page reuses the full marketing site layout from phase-5-ui-ux/01-marketing-site-layout.md. The body is a single long-scroll page with a sticky in-page anchor nav linking to the main sections.

```
[Sticky header with product nav]
[Comparison page body]
  ├── Hero — headline, 2-line summary, primary CTA
  ├── In-page anchor nav (Overview | Features | Pricing | Verdict)
  ├── Overview — one paragraph per product
  ├── Feature comparison table — scrollable on mobile
  ├── Pricing comparison
  ├── Where [Your Product] wins
  ├── Where [Competitor] wins
  ├── Verdict + CTA
  └── FAQ
[Site footer]
```

The feature comparison table is the centrepiece. On mobile it collapses to a two-column card layout (your product vs competitor) rather than a wide horizontal table that requires scrolling.

### Technical SEO for Comparison Pages

Apply the full technical SEO foundation from phase-6-marketing-site/02-technical-seo.md with these additions:

- Title: `[Your Product] vs [Competitor]: [Year] Comparison` — keyword-first, under 60 characters
- Canonical: always self-referencing — no pagination, no parameter variants
- `FAQPage` JSON-LD on every comparison page (the FAQ section at the bottom earns rich results)
- OG image: dynamically generated with both product names in the image text — makes social shares immediately legible
- Do not noindex comparison pages. They are public, indexable, high-intent pages.

---

## Use-Case and Solution Pages

### When to Build Them

Build use-case pages when two conditions are both true: you have at least 3 real customers from a specific audience or workflow, and keyword research (phase-13-content/03-keyword-research.md) confirms people search for that audience or workflow. Don't build speculative use-case pages — thin pages that aren't grounded in real usage patterns hurt rather than help.

### URL Structure

`/use-cases/[slug]` — one page per audience segment or workflow. Examples: `/use-cases/freelancers`, `/use-cases/marketing-teams`, `/use-cases/invoice-automation`.

Keep slugs descriptive and stable. A URL change after indexing requires a 301 redirect and loses search equity in the interim.

### Page Layout

Each use-case page is a focused landing page, not a rehash of the features page. The audience is named in the headline and the entire page speaks to their specific workflow.

```
[Hero — headline names the audience, subheadline names the outcome]
[How it solves the specific workflow — 3 key features, scoped to this use case]
[Social proof — testimonials or logos from this audience specifically]
[Before/after or workflow diagram — what changes when they use the product]
[CTA — "Start free trial" or "See [use case] pricing"]
```

Features shown on a use-case page are a curated subset of the full features page — only the ones relevant to that audience. The features page (phase-6-marketing-site/01-marketing-site-pages.md) remains the authoritative list of everything the product does.

### Technical SEO for Use-Case Pages

- Title: `[Product] for [Audience/Use Case] — [Product Name]`
- Canonical: self-referencing
- No structured data type is mandatory, but `WebPage` schema is appropriate
- Each use-case page must have substantively different content from every other use-case page — overlapping copy triggers duplicate content treatment. If two audiences are nearly identical, merge them into one page.

---

## Demo and Screenshots Page

### When to Build It

Build this page before most post-launch pages — it serves visitors who want to see the product before committing to a trial. It should exist as soon as the product UI is stable enough that screenshots won't go stale within weeks.

### URL

`/demo` — single page. Do not create `/screenshots` and `/video` as separate pages; consolidate into one destination.

### Page Layout

```
[Hero — "See [Product] in action", primary CTA: "Start free trial"]
[Product video — embedded from YouTube, autoplay off, captions on]
[Annotated screenshot tour — 5-8 key screens, each with a caption]
[Interactive demo embed (if available) — sandbox with sample data]
[CTA — "Try it yourself — free trial, no card required"]
```

The video is the primary element. A 2-3 minute walkthrough of the core workflow converts better than a polished feature montage. Authenticity beats production value here.

Annotated screenshots use visual callouts (numbered overlays) pointing to the most important UI elements. Each screenshot has a one-line caption that names the screen and the action it enables — not "The dashboard" but "Project dashboard — see all active work at a glance."

An interactive demo (a sandboxed environment with sample data, not a marketing mock-up) is the highest-converting demo format when self-serve signup is the motion. If build cost is prohibitive at launch, ship the video + screenshots first and add the interactive demo later.

### Technical SEO for the Demo Page

- Title: `[Product] Demo — See it in Action`
- OG image: a polished product screenshot — the most visually compelling single screen
- The video embed uses a `VideoObject` JSON-LD block with `name`, `description`, `thumbnailUrl`, `uploadDate`, and `duration` — this enables video rich results in Google Search
- All screenshots have descriptive alt text naming the screen and the feature shown

---

## Navigation and Information Architecture

Post-launch pages slot into the marketing site nav without bloating it.

**Comparison pages** do not appear in the primary nav. They live in the site footer under a "Compare" column alongside legal pages. Visitors who need them will find them via search — they are SEO destinations, not browsing destinations.

**Use-case pages** appear in the primary nav as a dropdown under "Solutions" or "Use Cases" once there are 3 or more. Before that, link to them from the features page and relevant blog posts only. A single use-case page does not warrant a nav dropdown.

**Demo page** gets a nav link in the primary navigation alongside "Pricing." "See a demo" is a high-intent action and should be surfaced at the top level.

### Internal Linking

Every post-launch page must link to at minimum: the pricing page, the features page, and the signup CTA. Use-case pages link to the 1-2 most relevant comparison pages and vice versa. The demo page links to every use-case page in a "See it in context" section at the bottom. Blog posts in phase-13-content/04-blog-posts.md that mention a competitor link to the corresponding comparison page.

The comparison page footer links to the alternative page for the same competitor and to the 2 most closely related use-case pages — creating a topical cluster that builds authority on the competitive keyword space.

---

## Deliverables / Implementation Checklist

**Comparison and Alternative Pages**
- [ ] `/compare/[competitor-slug]` page created for each major competitor
- [ ] `/compare/[competitor-slug]-alternative` page created for top 3 competitors
- [ ] Sticky in-page anchor nav implemented
- [ ] Feature comparison table responsive (card layout on mobile)
- [ ] `FAQPage` JSON-LD added to each comparison page
- [ ] Dynamic OG image generated with both product names
- [ ] All comparison pages in sitemap (phase-6-marketing-site/02-technical-seo.md)
- [ ] Footer "Compare" column links added
- [ ] Internal links from comparison pages to pricing, features, use-case pages

**Use-Case and Solution Pages**
- [ ] `/use-cases/[slug]` created for each confirmed audience segment (min. 3 real customers)
- [ ] Each use-case page features a curated subset of relevant features only
- [ ] Testimonials/logos scoped to the specific audience on each page
- [ ] No duplicate copy across use-case pages (each is substantively distinct)
- [ ] "Solutions" nav dropdown added once 3+ use-case pages exist
- [ ] Internal links from features page to relevant use-case pages

**Demo and Screenshots Page**
- [ ] `/demo` page created
- [ ] Product walkthrough video embedded (YouTube, captions on, autoplay off)
- [ ] 5-8 annotated screenshots with descriptive captions
- [ ] `VideoObject` JSON-LD configured with full metadata
- [ ] All screenshot alt text descriptive and accurate
- [ ] "Demo" link added to primary navigation
- [ ] CTA on demo page links to free trial signup

---

## What to Test

- Comparison page feature table renders correctly on mobile (no horizontal overflow)
- `FAQPage` JSON-LD validates in Google's Rich Results Test on every comparison page
- `VideoObject` JSON-LD validates on the demo page
- All comparison pages are indexed (not accidentally noindexed)
- Use-case page titles are unique — no two pages share the same title or meta description
- Demo page video loads and plays correctly on mobile
- All annotated screenshots have alt text
- Footer "Compare" column links resolve correctly
- "Solutions" dropdown appears in nav when 3+ use-case pages exist
- Internal links from each post-launch page reach pricing, features, and signup

---

## Edge Cases

- **Naming a competitor on a public page** — only use the competitor's publicly-used brand name, never trademarks with ® or ™ unless you have written permission, and ensure every factual claim is verifiable against their live site. "Honest comparison" is the legal defence — fabricated or unverifiable claims are not.
- **Comparison page goes stale when competitor changes pricing** — set a quarterly calendar reminder to audit all comparison pages; phase-13-content/06-comparison-content.md covers the update workflow in full. A stale comparison page actively damages trust when a visitor cross-checks.
- **Thin use-case pages hurting SEO** — a use-case page that only rewrites the features page with a different headline is treated as duplicate or thin content by search engines. Each page must address a distinct workflow, show audience-specific social proof, and be longer than 600 words of substantive content.
- **Demo screenshots showing outdated UI** — every time a significant UI change ships, audit `/demo` and update affected screenshots within the same sprint. A demo page showing a UI that doesn't match the live product destroys trust at the highest-intent moment of the journey.
- **Competitor rebrands** — if a competitor changes their name, update the comparison and alternative pages immediately. A page targeting an old brand name loses its search relevance and may confuse visitors.
- **Use-case page URL changes after indexing** — add a 301 redirect from the old slug to the new one. Without it, the old URL returns 404 and accumulated link equity is lost.

---

## Common Mistakes

- Building comparison pages before launch when you don't yet know your real competitive advantages — the copy will be generic and quickly stale
- Using wide comparison tables on mobile without a responsive fallback (the page becomes unusable)
- Creating a dozen use-case pages at launch with no real customer evidence to back them — thin pages drag down the entire domain's search quality
- Putting comparison pages in the primary nav (they're SEO destinations, not browsing destinations — the footer is correct)
- Omitting the "Where they win" section on comparison pages — readers will check, and a comparison that only shows your product winning reads as dishonest
- Building `/screenshots` and `/video` as separate pages instead of consolidating into `/demo` — two thin pages rank worse than one authoritative one
- Interactive demo using fake or aspirational UI instead of the real product — it creates expectation mismatches that kill trial-to-paid conversion
- Forgetting `VideoObject` JSON-LD on the demo page — the video cannot appear as a rich result in Google Search without it
- Launching a use-case page for an audience the product doesn't yet serve well (premature positioning damages the relationship when they sign up and reality doesn't match)
