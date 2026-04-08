# Content Operations

---

## What It Is

How content production is planned, scheduled, and automated. The editorial calendar determines what gets published when, and the repurposing workflow ensures every piece of content is distributed across all channels with minimal manual effort.

## Editorial Calendar

### Tool

Use Notion. One database table tracking all content across all types.

### Calendar Structure

Every content item in the calendar has:

| Field | Content |
|---|---|
| Title | The content piece title |
| Type | Blog post, tweet thread, comparison page, case study, changelog, video, carousel, pSEO |
| Target keyword | From keyword research (phase-14/03-keyword-research.md) |
| Status | Idea → Outlined → Drafted → Edited → Scheduled → Published |
| Publish date | Scheduled date |
| Topic cluster | Which pillar this supports |
| Author | Who is responsible |
| Distribution | Which channels it will be distributed to |

### Planning Cadence

**Design decisions:**
- Plan 4 weeks of content at the start of each month
- Weekly review every Monday: what's publishing this week, what's in draft, what's behind schedule
- Content priorities come from keyword research scoring — highest-priority keywords get content first
- Balance content types: every week should have 1 blog post, 5 tweets, and 1 social graphic minimum
- Don't plan more than 4 weeks ahead — priorities shift as you learn what performs

### Status Workflow

```
Idea → Outlined → Drafted → Edited → Scheduled → Published
```

- **Idea:** keyword identified, content type decided, added to calendar
- **Outlined:** Claude generated outline, human approved
- **Drafted:** Claude generated first draft
- **Edited:** human edited, stats added, links added, visuals added
- **Scheduled:** MDX committed or social post queued in Postiz
- **Published:** live on the site or posted on social media

Every content piece moves through every status. No jumping from Idea to Published.

## Content Repurposing Workflow

One piece of content becomes 5 distribution pieces.

### Blog Post → Everything Else

Every blog post automatically generates:

| Output | How | Tool | When |
|---|---|---|---|
| Tweet thread | Claude extracts 5 key points from the post | Claude + Postiz | Same day as publish |
| LinkedIn post | Claude writes a 200-word personal take on the post's topic | Claude + Postiz | Same day as publish |
| Newsletter section | Post headline + 2-sentence summary + "Read more" link | Resend | Next weekly newsletter |
| Carousel | Claude extracts 5 points, one per slide | Claude + @vercel/og | Within 3 days |
| Memes | 3 memes from memethropic.com API related to the post topic | memethropic.com API + Postiz | Staggered across the week |

### Repurposing Process

**Design decisions:**
- Repurposing happens immediately after publishing the blog post — it's step 8 of the blog production pipeline (phase-14/04-blog-posts.md)
- Claude handles the repurposing — provide the full blog post text and ask for each output format
- Human reviews each repurposed piece before scheduling (Claude's thread may miss the best insight, or the LinkedIn tone may be off)
- Social posts are scheduled via Postiz to go out over the following week — don't dump all distribution on the same day
- The blog post is the source. If the repurposed content contradicts the blog post, the blog post is correct.

### Distribution Schedule

After publishing a blog post:

| Day | Distribution |
|---|---|
| Day 0 (publish day) | Tweet thread + LinkedIn post |
| Day 1 | Meme #1 related to the topic |
| Day 2 | Carousel post |
| Day 3 | Meme #2 |
| Day 5 | Meme #3 |
| Day 7 | Newsletter includes the post |

This spreads content across the week instead of front-loading everything on publish day.

### Evergreen Recycling

**Design decisions:**
- Blog posts older than 3 months can be re-distributed with fresh social posts
- Claude rewrites the social angle (different hook, different takeaway from the same post)
- Schedule 1 recycled post per week via Postiz alongside new content
- Never recycle content that's outdated — only evergreen posts that are still accurate

## Automation Opportunities

**Design decisions:**
- Blog publish → auto-schedule social posts: Postiz can be triggered to queue pre-written social posts when a blog post goes live (via webhook or manual batch scheduling)
- Changelog publish → auto-notify in-app + auto-tweet: the notification system (phase-3/07-notification-system.md) handles in-app, Postiz handles the tweet
- Weekly newsletter → auto-include latest blog posts: the newsletter template pulls the latest published posts from the blog index
- Meme generation → memethropic.com API called with the blog post topic, 3 memes generated and queued in Postiz

## What to Test

- Editorial calendar in Notion has all required fields
- Every content piece moves through every status (no skipping from Idea to Published)
- Blog posts generate all 5 repurposed outputs (thread, LinkedIn, newsletter, carousel, memes)
- Distribution is spread across the week (not all on day 0)
- Evergreen recycling produces 1 post per week from older content
- Newsletter includes the latest blog post automatically

## Edge Cases

- **Blog post underperforms — should you still repurpose?** — yes. The social distribution may reach a different audience than organic search. A post that doesn't rank can still drive engagement on Twitter.
- **Repurposed content outperforms the original** — the tweet thread goes viral but the blog post has low traffic. Add a prominent link to the blog post in the thread and pin the thread. Use the social engagement to drive blog traffic.
- **Editorial calendar falls behind** — if you're 2 weeks behind, don't try to catch up by publishing 4 posts in one week. Adjust the calendar to the current pace and maintain consistency from here.
- **Content type isn't performing** — if carousels consistently get low engagement, stop producing them and double down on what works (threads, single tweets, video). Review performance monthly and adjust the content mix.

## Implementation Checklist

- [ ] Notion editorial calendar created with all required fields
- [ ] Monthly planning session scheduled (first Monday of each month)
- [ ] Weekly review scheduled (every Monday)
- [ ] Weekly minimum defined: 1 blog post, 5 tweets, 1 social graphic
- [ ] Repurposing workflow documented (blog → thread, LinkedIn, newsletter, carousel, memes)
- [ ] Distribution schedule documented (day 0 through day 7)
- [ ] Claude prompts created for each repurposing format
- [ ] Postiz configured for scheduled posting across all platforms
- [ ] Evergreen recycling: 1 recycled post per week from content older than 3 months
- [ ] Monthly performance review scheduled (what's working, what to cut)

## Common Mistakes

- No editorial calendar (publishing randomly without a plan)
- Publishing without distributing (blog post goes live, no social posts, no newsletter mention)
- All distribution on publish day (5 social posts on Monday, silence for 6 days)
- No repurposing workflow (every social post is written from scratch instead of derived from blog posts)
- Planning too far ahead (3 months of content planned, priorities changed after 2 weeks)
- Never recycling evergreen content (a great post from 6 months ago gets zero ongoing distribution)
- Skipping the human review on repurposed content (Claude's thread misses the best insight or sounds generic)
