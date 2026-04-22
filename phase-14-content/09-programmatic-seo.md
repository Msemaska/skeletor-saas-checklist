# Programmatic SEO Pages

---

## What It Is

Template-based pages generated at scale from structured data. Instead of manually writing 200 landing pages, you create one template and populate it with data variations — capturing long-tail search traffic that manual content creation can't reach.

## When to Use pSEO

**Design decisions:**
- Use pSEO when your product has a natural dimension that creates many variations: integrations, locations, industries, use cases, templates, tools
- Each variation must target a distinct keyword that real people search for
- Every generated page must provide genuine value — not just the same content with a word swapped
- pSEO is a post-launch strategy. Build it after the core marketing site and initial blog content are established.

## Page Types for SaaS

| Type | Template Pattern | Example |
|---|---|---|
| Integration pages | "[Your Product] + [Integration]" | "InvoiceTool + Slack Integration" |
| Alternative pages | "[Your Product] for [Use Case]" | "InvoiceTool for Freelancers" |
| Location pages | "[Category] in [City/Region]" | "Invoice Software in London" |
| Template gallery | "[Template Name] Template" | "Freelance Invoice Template" |
| Tool pages | "Free [Tool] Calculator/Generator" | "Free Invoice Generator" |
| Industry pages | "[Category] for [Industry]" | "Invoice Software for Restaurants" |

## Architecture

**Design decisions:**
- Pages are generated at build time from a data source (JSON file, database table, or API)
- One Next.js dynamic route handles all variations: /integrations/[slug], /for/[use-case], /in/[location]
- Each page renders the same template with different data injected
- Every generated page has unique: title, meta description, H1, and body content. Pages that differ only in the swapped word are thin content and will be penalised.
- Sitemap includes all generated pages
- noindex pages that have no search volume or no unique value (don't index 10,000 location pages if only 50 cities have search volume)

## Template Design

Every pSEO page must feel like a real page, not a mail merge.

**Design decisions:**
- The template has dynamic sections that change per variation AND static sections that provide consistent value
- Dynamic content per variation: H1 title, intro paragraph, 3 specific benefits or features relevant to this variation, relevant stats or data points, FAQ section with variation-specific questions
- Static content across all pages: product overview, feature comparison, pricing summary, CTA, testimonials
- Each page has at least 500 words of unique content per variation. Swapping one word in a sentence is not unique content.

## Data Source

**Design decisions:**
- Store the data for all variations in a JSON file in the repository
- Each data entry includes: slug, title, description, specific benefits, FAQ questions and answers, relevant keywords, and any variation-specific data points
- Claude generates the variation-specific content from a prompt template + the data entry. Human reviews a sample before bulk generation.
- New variations are added by adding a row to the data source — no code changes needed

## Content Quality Rules

**Design decisions:**
- Every generated page must pass the "would I share this?" test. If the page is just your homepage with "for Freelancers" appended, it's thin content.
- Each variation needs genuine, specific content: what's different about this use case, what features matter most for this audience, what problems are unique to this context
- Review 10 random pages after generation to verify quality. If they feel generic, the template needs more variation-specific content.
- Monitor Search Console for indexing issues. If Google is not indexing your pSEO pages, they're too thin.

## Integration Pages (Detailed Example)

The most common pSEO pattern for SaaS:

**Data per integration:**
- Integration name and logo
- What the integration does (3 sentences, specific to the pairing)
- 3 benefits of using your product with this integration
- Setup steps (how to connect)
- FAQ: 3 questions specific to this integration
- Related integrations

**Template:**
- H1: "[Your Product] + [Integration Name] Integration"
- Intro: what you can do when the two products work together
- Benefits section: 3 specific benefits
- How it works: setup steps with screenshots
- FAQ with structured data
- CTA: "Connect [Integration] to [Your Product]"
- Related integrations grid

## What to Test

- Every generated page has a unique title, description, and H1
- No two pages have identical body content
- Sitemap includes all generated pages
- Pages with no search volume are noindexed
- Each page has at least 500 words of unique content
- FAQ generates FAQPage structured data
- Template renders correctly with edge case data (long names, special characters, missing optional fields)
- Pages load within 3 seconds (generated pages shouldn't be slower than hand-crafted pages)

## Edge Cases

- **Data source has missing fields** — the template must handle optional fields gracefully. Don't show an empty section — hide it if the data isn't available.
- **Too many pages, low quality** — generating 5000 pages with thin content hurts your entire site's SEO. Start with 50 high-quality variations. Add more only after the first batch is indexed and ranking.
- **Integration partner changes their product** — update the data entry and regenerate. Set a quarterly review for integration pages.
- **Google deindexes pSEO pages** — the content is too thin or too similar across pages. Add more unique content per variation, reduce the number of pages, or noindex the weakest ones.
- **Cannibalisation with blog posts** — if you have a blog post "Best Invoice Tools for Freelancers" and a pSEO page "Invoice Software for Freelancers", they compete for the same keyword. The pSEO page should target the more transactional variant, the blog post targets the informational variant.

## Implementation Checklist

- [ ] pSEO page type chosen (integrations, use cases, locations, templates, tools, industries)
- [ ] Data source created with structured entries for each variation
- [ ] Dynamic route template built in Next.js
- [ ] Claude prompt template created for generating variation-specific content
- [ ] 10 sample pages reviewed for quality before bulk generation
- [ ] Each page has at least 500 words of unique content
- [ ] Unique title, description, H1 per page
- [ ] FAQ section with structured data per page
- [ ] Sitemap includes all generated pages
- [ ] Low-value pages noindexed
- [ ] Search Console monitored for indexing of pSEO pages
- [ ] Start with 50 pages, expand after indexing is confirmed

## Common Mistakes

- Generating thousands of thin pages (same content with one word swapped)
- No unique content per variation
- Starting with pSEO before having a strong domain
- Not monitoring indexing
- Cannibalising existing blog content (pSEO page and blog post targeting the same keyword)
- No FAQ structured data (missing featured snippet opportunities)
- Template doesn't handle missing data gracefully (empty sections, broken layouts)
- Indexing every variation including low-volume ones (noindex pages nobody searches for)
