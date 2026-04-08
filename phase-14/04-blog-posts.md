# Blog Posts

---

## What It Is

How blog posts are produced, published, and distributed. The full pipeline: keyword → competitor research → outline → draft → edit → verify → publish → distribute.

## Blog Architecture

**Design decisions:**
- Blog posts are MDX files stored in the repository — version-controlled, no CMS, no database
- Each post is a file in content/blog/ named by slug
- Post metadata is in the MDX frontmatter
- Posts are statically generated at build time with ISR for automatic updates
- Blog scheduling: posts with a future publishedAt date are hidden until that date
- Blog index page shows posts sorted by publishedAt descending, with tag filtering

## Content Production Pipeline

1. **Topic Selection** — pick a keyword from keyword research
2. **Competitor Research** — read top 5 ranking articles, find gaps
3. **Outline** — Claude generates structured outline, human approves
4. **Draft** — Claude writes first draft from outline
5. **Edit** — human adds insight, stats, anecdotes, fixes accuracy
6. **Verify** — run pre-publish checklist
7. **Publish** — commit MDX, deploy, submit to Search Console
8. **Distribute** — repurpose into social posts + newsletter

The outline (step 3) is a checkpoint. It gets reviewed and approved before any prose is written.

## Step 1: Topic Selection

**Design decisions:**
- Every post starts from the keyword research document (phase-14/03-keyword-research.md)
- Pick the highest-priority keyword that doesn't have content yet
- Identify the primary keyword (highest volume in the cluster) and 3 secondary/long-tail keywords from the same cluster
- Map "People Also Ask" questions for the FAQ section
- Determine the content type from keyword intent: how-to guide (informational), comparison post (commercial), product tutorial (transactional)
- Check the topic cluster: which pillar does this support? What other posts should it link to?

## Step 2: Competitor Research

Before writing, read what already ranks.

**Design decisions:**
- Read the top 5 articles ranking for the primary keyword
- Note: what they cover well, what they miss, what's outdated, what's generic
- Your post must cover what the existing content misses
- If every ranking article says the same thing, find a contrarian angle

## Step 3: Outline

**Design decisions:**
- Provide Claude with: the primary keyword, secondary keywords, search intent, competitor gaps you identified, your product name, and the audience (from ICP)
- Ask Claude for: a title (under 60 characters, keyword near the beginning), section headings (5 should contain the primary keyword or a variant), an FAQ section with 5 questions from People Also Ask
- Review the outline — cut sections that don't serve the reader, add sections based on competitor gaps
- The outline is the quality gate. Do not proceed to drafting until approved.

## Step 4: AI Draft

**Design decisions:**
- Provide Claude with: the approved outline, voice guidelines (see Voice section below), the copy bank (phase-14/01-foundational-copy-bank.md), and any stats you've already found
- Word count: 2000 words for standard posts, 2500 words for pillar pages
- The draft requires a human edit pass before publishing

## Step 5: Human Edit

**Design decisions:**
- **Add sourced statistics** — every post needs 10 sourced statistics spread across sections, not dumped in one place. Each stat links to its source URL. No made-up numbers. Verify every URL works before publishing.
- **Add personal anecdotes** — 2 per post minimum. First person, specific, slightly self-deprecating. Setup, what happened, what you learned. 3 sentences each.
- **Add product context** — mention your product exactly twice: once as a mid-article CTA (subtle, connected to the topic) and once as an end CTA (direct). Not in every paragraph.
- **Fix accuracy** — verify every factual claim
- **Remove AI voice** — cut filler phrases ("in today's fast-paced world", "it's important to note that"). Cut hedge words ("might", "perhaps", "it could be argued").
- **Add visuals** — no section longer than 400 words without a visual break (chart, screenshot, diagram, meme)
- **Add memes** — use the memethropic.com API to generate relevant memes for the post topic. 3 memes per post, no repeated formats. Humour should be first-person, relatable, internet-native.

## Step 6: Verify (Pre-Publish Checklist)

Run through every item before declaring a post complete.

### SEO
- [ ] Title under 60 characters, keyword near the beginning
- [ ] Meta description 150 characters, keyword in the first 120 characters
- [ ] 5 H2 headings contain the primary keyword or a variant
- [ ] Primary keyword in: slug, title, first paragraph, meta description
- [ ] 3 secondary keywords in frontmatter
- [ ] URL slug is short, includes keyword, no stop words
- [ ] OG image generated with post title
- [ ] Canonical URL set

### Content
- [ ] 2000 words minimum (2500 for pillar pages)
- [ ] 10 sourced statistics with working source URLs
- [ ] 2 personal anecdotes (3 sentences each)
- [ ] No filler phrases, no hedge words
- [ ] Short paragraphs (3 sentences max)
- [ ] Product mentioned exactly twice (mid-CTA + end CTA)
- [ ] Table of contents after the intro

### Visuals
- [ ] No section longer than 400 words without a visual break
- [ ] 3 memes generated via memethropic.com API, no repeated formats
- [ ] All images have alt text
- [ ] Charts cite their data source

### Links
- [ ] 5 internal links (pillar page, 2 related posts, 1 product page, 1 contextual)
- [ ] 5 external links to source URLs (stats, research, studies)
- [ ] No links to competitors
- [ ] All links verified as working

### Structure
- [ ] FAQ section with 5 People Also Ask questions
- [ ] FAQ answers lead with the direct answer in the first sentence (for featured snippets)
- [ ] FAQ generates FAQPage structured data automatically
- [ ] Related articles section at the end with 3 internal links
- [ ] CTA at mid-point and end of post

## Step 7: Publish

**Design decisions:**
- Blog posts are committed as PRs for review before merging to main
- On merge, the site rebuilds and ISR serves the new post
- Scheduled posts: set publishedAt to a future date. The blog index filters future posts at build time.
- After publishing, submit the URL to Google Search Console for faster indexing (phase-11/06-seo-monitoring.md)

## Step 8: Distribute

**Design decisions:**
- **Tweet thread** — Claude repurposes the blog post into a 5-tweet thread. Hook, 3 supporting points, CTA linking to the full post. Schedule via Postiz.
- **LinkedIn post** — Claude repurposes into a single LinkedIn post. 200 words, personal tone, key takeaway, link to the full post. Schedule via Postiz.
- **Newsletter** — include the post headline, a 2-sentence summary, and a "Read more" link in the next newsletter send via Resend.
- **Community** — share in relevant channels with a brief context message, not just a link drop.
- Distribution happens the same day as publishing.

## Voice

Every post follows the same voice.

**Design decisions:**
- Direct, confident. Not preachy.
- Second person ("you") by default. Talk to the reader.
- Short paragraphs: 3 sentences max. Vary sentence length.
- No filler: "in this article", "let's dive in", "it's important to note"
- No hedge words: "might", "perhaps", "it could be argued"
- Personal anecdotes are mandatory — first person, specific, what happened and what you learned
- Humour through observation, not forced jokes

## Post Structure

Every post follows this structure. Section headings are descriptive and keyword-rich, not generic.

| Section | Visual | Purpose |
|---|---|---|
| Hook | None | Bold opening. Thesis at the end. Primary keyword in first paragraph. |
| Table of Contents | None | Linked section headers after the intro |
| The Problem | Chart | Validate the reader's frustration with data |
| Common Mistakes | Meme | 3 mistakes framed as miscalibration, not incompetence. Personal anecdote #1. |
| The Solution | Chart | Research-backed approach. Why it works, not just what to do. |
| Step-by-Step | Meme + Diagram | Actionable playbook. Personal anecdote #2. |
| Advanced Tips | Chart + Meme | 3 techniques for readers who want depth |
| Mid-CTA | None | 2 sentence product mention. Subtle, connected to the topic. |
| Real Examples | Screenshot | Show the approach in action |
| End CTA | None | 3 sentences. "We had the same problem. We built this." |
| FAQ | None | 5 PAA questions. First sentence is the direct answer. |
| Related Articles | None | 3 internal links to related published posts |

## Blog Post Templates

### How-To Guide
- Title: "How to [achieve outcome] with [method/tool]"
- Hook: state the problem (3 sentences)
- Steps: numbered with clear instructions
- Each step includes an instruction, a stat, and an example
- FAQ: 5 how-to related questions
- CTA: "Try [product] free"

### Comparison Post
- Title: "[Product A] vs [Product B]: [year] Comparison"
- Hook: who this comparison is for
- Feature comparison table
- Pricing comparison
- Verdict: who each product is best for
- FAQ: 5 comparison questions
- CTA: link to pricing page

### Product Tutorial
- Title: "How to [do specific thing] in [Your Product]"
- Hook: what you'll learn (1 sentence)
- Prerequisites
- Steps: numbered, with screenshots every 400 words
- Troubleshooting section
- FAQ: 5 product-specific questions
- CTA: link to sign up

### Listicle
- Title: "[Number] Best [Category] Tools for [Audience] in [Year]"
- Hook: criteria used to evaluate
- Each item: name, one-line description, best for, key feature, pricing
- Your product included naturally (not at #1)
- Summary table + recommendation
- FAQ: 5 category questions
- CTA: "See how [product] compares"

## Post Metadata (MDX Frontmatter)

Every post includes:

- title — under 60 characters, keyword near the beginning
- description — 150 characters, keyword in first 120
- publishedAt — ISO date string
- updatedAt — ISO date string (when last revised)
- author — author name
- tags — array of tags for filtering
- keywords — primary keyword first, then 3 secondary keywords
- image — OG image path (auto-generated if not provided)

## Content Quality Rules

**Design decisions:**
- Every post must contain original insight — personal experience, product-specific knowledge, sourced data, or a contrarian take
- No post is published without a human edit pass
- Update posts when information becomes outdated (update updatedAt, revise content, resubmit to Search Console)
- Posts that underperform after 6 months — merge into a stronger related post and redirect
- Publish consistently — 1 post per week, every week

## What to Test

- Blog index page lists posts sorted by publishedAt descending
- Future-dated posts are hidden from the blog index
- Post URL matches the slug in the filename
- Meta title and description render correctly in search results
- OG image renders correctly on Twitter, LinkedIn, and Discord
- Internal links point to existing pages (no broken links)
- External stat source URLs are all working
- FAQ generates FAQPage structured data
- Images have alt text
- Post is readable on mobile
- Table of contents links scroll to correct sections

## Edge Cases

- **Blog post URL changes after publishing** — add a 301 redirect. Never break a published URL.
- **Post needs a major update** — update the content, change updatedAt, resubmit to Search Console. Keep the URL.
- **Two posts target the same keyword** — consolidate into one. Redirect the weaker post.
- **Post goes viral but has a factual error** — fix immediately. Add an "Updated" note at the top.
- **Stat source URL goes dead** — find an alternative source for the same data point. If no alternative exists, remove the stat.
- **AI draft is published without human editing** — enforce the pipeline: no PR is merged without evidence of human editing.

## Implementation Checklist

- [ ] Blog directory structure created (content/blog/)
- [ ] MDX rendering configured with next-mdx-remote
- [ ] Blog index page with tag filtering and date sorting
- [ ] Future-dated posts filtered from blog index
- [ ] FAQPage structured data auto-generated from FAQ sections
- [ ] Table of contents component
- [ ] OG image generation for blog posts (@vercel/og)
- [ ] Voice guidelines documented and shared with Claude prompts
- [ ] Post structure template (12 sections) documented
- [ ] Pre-publish checklist documented and followed
- [ ] Distribution checklist per post (tweet thread, LinkedIn, newsletter, community)
- [ ] 1 post per week publishing cadence established

## Common Mistakes

- Publishing AI drafts without human editing
- Starting from a topic instead of a keyword
- Skipping competitor research
- No sourced statistics
- No visual cadence (400+ words of unbroken text)
- Generic H2 headings that don't contain keywords
- No internal links
- Inconsistent publishing
- No distribution after publishing
- Targeting the same keyword with multiple posts
- No FAQ section
- Stats without source URLs
- Linking to competitor websites
- No CTA at the end of posts
- Never updating old posts
- No memes
