# Keyword Research

---

## What It Is

The research that identifies what your audience searches for, what language they use, and what questions they have. Informs blog topics, landing page copy, social media language, ad targeting, aggregator listings, FAQ content, and LLM optimization. Done once before launch, refined quarterly.

## Where to Store It

`docs/keyword-research.md` in the repository. Contains all keyword categories, topic clusters, scored keyword list, and content mapping. Version-controlled alongside the copy bank.

## Start with Customer Insights, Not Tools

**Design decisions:**
- Review your ICP (phase-0/02-ideal-customer-profile.md) — what language does your audience use to describe their problem?
- Review your competitor analysis (phase-0/03-competitor-analysis.md) — what terms do competitors rank for? What gaps exist?
- If you've talked to potential users (beta testers, waitlist signups), note the exact words they used to describe their problem.
- Focus on problems and outcomes, not features.

## Keyword Categories

### 1. Product Keywords

Terms that directly describe what your product is.

- "[category] software" / "[category] tool" / "[category] app"
- "[category] for [audience]"

Where they go: landing page headline, meta title, Product Hunt tagline, app store title, directory listings.

### 2. Problem Keywords

Terms people search when they have the problem before they know a solution exists.

- "how to [solve problem]"
- "best way to [workflow]"
- "why is [pain point] so hard"
- "[pain point] solution"

Where they go: blog post topics, FAQ content, landing page subheadlines.

### 3. Comparison Keywords

Terms people search when evaluating options.

- "[your product] vs [competitor]"
- "[competitor] alternative"
- "best [category] tools [year]"
- "[competitor] pricing"
- "switch from [competitor]"

Where they go: comparison blog posts, post-launch pages (phase-5/01-marketing-site-pages.md), FAQ answers, ad targeting.

### 4. Feature Keywords

Terms describing specific capabilities.

- "[specific feature]" (e.g., "automated invoice reminders")
- "[feature] for [audience]" (e.g., "time tracking for freelancers")
- "how to [feature-specific action]"

Where they go: features page sections, feature-specific blog posts, use case pages.

### 5. Audience Keywords

Terms your target audience uses to identify themselves.

- "[role] tools" / "[role] software" (e.g., "tools for startup founders")
- "[industry] software" (e.g., "software for marketing agencies")
- "best tools for [role/industry]"

Where they go: use case pages, social media bios and hashtags, ad audience targeting, Reddit/community posts.

## Research Process

### Step 1: Generate Seed Keywords

List 15 seed keywords from your value proposition, competitor sites, and ICP research.

### Step 2: Expand with Free Tools

For each seed keyword, expand into related terms:

- **Google Search autocomplete** — type the seed keyword, note every suggestion
- **Google "People Also Ask"** — the questions Google shows in search results.
- **Google "Related Searches"** — at the bottom of the results page
- **Claude** — ask: "What would someone search for if they wanted a tool that [your value proposition]?" and "What questions would someone have about [your category]?"
- **Reddit / community forums** — search for your category, note the exact words real users use.
- **Competitor websites** — look at their blog titles, features page headings, and meta descriptions.

### Step 3: Analyse Search Intent

| Intent | Signal | Content Type |
|---|---|---|
| Informational | "how to", "what is", "guide" | Blog post, tutorial |
| Commercial | "best", "vs", "alternative", "review" | Comparison page, listicle |
| Transactional | "[product] pricing", "sign up", "free trial" | Landing page, pricing page |
| Navigational | "[brand name]", "[product] login" | Homepage, sign-in page |

**Design decision:** Prioritise commercial intent keywords first, then transactional, then informational, then navigational.

### Step 4: Analyse the Actual SERP

**Design decisions:**
- Google the keyword. Look at the top 5 results.
- If the top results are from sites similar to yours in size and authority, you can compete.
- If the top results are from Wikipedia, Forbes, and established SaaS companies with 10 years of content, target a more specific long-tail variant instead.
- If the top results are old, thin, or poorly written, you can outrank them with better content regardless of domain authority.

### Step 5: Score and Prioritise

For each keyword, score on three dimensions:

| Dimension | Score 1 | Score 2 | Score 3 |
|---|---|---|---|
| Intent fit | Low intent (informational) | Medium (commercial) | High (transactional) |
| Competitive chance | Top results are dominant brands | Mixed results | Weak or outdated results |
| Conversion potential | Unlikely to sign up | May sign up eventually | Ready to sign up |

Keywords that score 3 on all three go to the top of your content priority list. Don't create content for low-scoring keywords until the high-scoring ones are covered.

### Step 6: Map to Content

Every high-priority keyword maps to a specific piece of content. No keyword should be unassigned, and no content should be created without a target keyword.

| Keyword Type | Content Type |
|---|---|
| Product keywords | Landing page, meta tags, directory listings |
| Problem keywords | Blog posts, FAQ page |
| Comparison keywords | Comparison pages, blog posts |
| Feature keywords | Features page sections, blog posts |
| Audience keywords | Use case pages, social media |

## Topic Clusters

**Design decisions:**
- Define 3 topic clusters before launch. Start with 1, prioritise the one with the highest impact potential.
- Each cluster has one pillar page targeting a high-volume keyword and 10 supporting articles targeting long-tail keywords.
- The pillar page links to all supporting articles. Every supporting article links back to the pillar.
- Supporting articles also interlink with each other where relevant.

**Cluster template:**
- **Pillar:** "Complete Guide to [Your Category]" (2000 words, comprehensive)
- **Supporting:** "How to [specific workflow]", "[Your Product] vs [Competitor]", "Best [category] tools for [audience]", "[Feature] explained", "Common [category] mistakes", "[Category] for [specific industry]", "[Category] pricing comparison", "[Specific use case] tutorial", "FAQ: [Category] questions answered", "[Category] trends in [year]"

## Post-Launch: Google Search Console Gap Analysis

**Design decisions:**
- After 4 weeks of being live, review Search Console's Performance report
- Look for: pages with high impressions but low click-through rate (title/description needs improvement), keywords ranking positions 11-30 (close to page 1 — create or improve content to push higher), keywords you rank for that you didn't target (opportunity — create dedicated content)
- This is covered in phase-10/06-seo-monitoring.md but the insights feed back into keyword research

## How Keywords Feed Other Phases

| Phase | How Keywords Are Used |
|---|---|
| Phase 5 (Marketing Site) | Landing page headline, features page copy, meta titles, FAQ page |
| Phase 13 (Text Content) | Blog post topics, article titles, subheadings |
| Phase 13 (Social Content) | Hashtags, post topics, thread ideas |
| Phase 13 (LLM Optimization) | FAQ content, comparison content, structured data |
| Phase 14 (Go-To-Market — ads) | Ad targeting keywords, ad copy |
| Phase 14 (Go-To-Market — listings) | Directory category selection, listing descriptions |
| Phase 10 (Observability — SEO) | Ranking tracking, performance monitoring |

## What to Test

- Seed keywords cover all 5 categories (product, problem, comparison, feature, audience)
- Every keyword has intent, score, and priority assigned
- 3 topic clusters defined with pillar and 10 supporting articles mapped
- Every high-priority keyword is mapped to a specific piece of content
- Landing page and features page copy includes the top product keywords
- Keywords are stored in docs/keyword-research.md

## Edge Cases

- **Your category doesn't have a standard name** — target adjacent category keywords your audience currently searches for while also creating content that defines your new category.
- **Competitor dominates all comparison keywords** — focus on long-tail variants: "[competitor] alternative for [specific use case]" rather than the broad "[competitor] alternative."
- **Keywords change after product pivot** — re-run the full research process. The keyword list is a living document.
- **Zero search volume keywords** — create content for them if they match your ICP (niche B2B terms, emerging categories).
- **Your content ranks for unintended keywords** — Google Search Console shows this. If the unintended keyword has commercial intent, create dedicated content to capture it properly.

## Implementation Checklist

- [ ] Customer insights gathered (ICP language, competitor terms, real user words)
- [ ] 15 seed keywords identified
- [ ] Seed keywords expanded using Google autocomplete, People Also Ask, Claude, Reddit
- [ ] Competitor keyword analysis completed (blog titles, features pages, meta descriptions)
- [ ] Keywords categorised (product, problem, comparison, feature, audience)
- [ ] Search intent assigned to every keyword (informational, commercial, transactional, navigational)
- [ ] SERP analysis completed for top 20 keywords (is competing realistic?)
- [ ] Keywords scored (intent fit, competitive chance, conversion potential)
- [ ] 3 topic clusters defined (pillar + 10 supporting articles each)
- [ ] Every high-priority keyword mapped to a content piece
- [ ] keyword-research.md created in docs/ directory
- [ ] Landing page and features page copy reviewed against top product keywords
- [ ] Quarterly review reminder set
- [ ] Post-launch: Search Console gap analysis after 4 weeks live

## Common Mistakes

- Chasing high-volume keywords over high-intent keywords (10,000 monthly searches that bring unqualified traffic vs 200 searches from people ready to buy)
- Targeting keywords where top results are dominant brands you can't outcompete (check the actual SERP, not just the difficulty score)
- Writing separate posts for nearly identical keywords ("best scheduling tool for startups" and "top scheduling software for small companies" — you're competing with yourself. Cluster them.)
- All content targeting top-of-funnel informational keywords (traffic but no conversions — balance with commercial and transactional keywords)
- Keyword research done once and never updated (search behaviour changes, competitors emerge, your product evolves)
- Keywords identified but never mapped to content (research without execution)
- Ignoring problem keywords (missing users who don't know your category exists yet)
- Storing keyword research in someone's head instead of a documented file
- Not analysing the actual SERP (trusting difficulty scores that don't reflect reality)
- Creating content without a target keyword (writing what feels interesting instead of what the audience searches for)
