# Blog System

---

## What It Is

The technical system that powers your public blog — content storage, rendering pipeline, SEO metadata, search, and scheduling. The blog is a primary SEO and authority-building asset. Every technical decision here has a direct impact on search ranking and content velocity.

## Architecture Decisions

**Content storage:** MDX files in `/content/blog/`. Each post is a single `.mdx` file with YAML frontmatter. Posts live in the git repo — publishing requires a deploy, but batch publishing (write 10 posts, set staggered `publishedAt` dates, deploy once) means one deploy covers weeks of content.

**MDX processor:** `next-mdx-remote` with App Router server components. Content is read from the filesystem at request time (not at build time), rendered server-side, and served as static HTML via ISR. No client-side MDX processing.

**Scheduling:** Set a future `publishedAt` date in frontmatter. ISR revalidates blog pages every hour — posts go live automatically at the next revalidation after their scheduled date, with no redeploy needed. Maximum delay: 59 minutes. Acceptable for a solo publishing workflow.

**Publishing pipeline:** Every time a post goes live, three indexing actions must fire — ping Google to re-fetch the sitemap, call the Google Indexing API with the post URL, and submit the URL to IndexNow. These actions have two triggers depending on how the post was published:

- **Deploy-time posts** (publishedAt in the past when deployed) — `instrumentation.ts` fires on server start and handles all three actions for the full sitemap. New posts are covered automatically.
- **ISR-scheduled posts** (publishedAt is a future date, goes live via revalidation) — `instrumentation.ts` has already run and won't re-fire. A Trigger.dev scheduled job runs hourly, compares the list of currently-live posts against a previously stored snapshot, and fires all three indexing actions for any newly-live post. The snapshot is stored in the database.

Never rely on manual Search Console submission — it hits rate limits fast and doesn't scale beyond a handful of URLs.

**Routing:** `app/(marketing)/blog/page.tsx` for the index, `app/(marketing)/blog/[slug]/page.tsx` for individual posts. Slugs are derived from the filename (`my-post-title.mdx` → `/blog/my-post-title`).

**Search:** Pagefind. Generates a static full-text search index at build time from the rendered HTML. Zero infra, zero runtime cost, works offline. Integrated as a client component with a search modal.

**Syntax highlighting:** Shiki with a VSCode Dark+ theme. Runs server-side — zero client JS, no flash of unstyled code.

**Table of contents:** Extracted from MDX headings at render time using `rehype-slug` and `rehype-autolink-headings`. Rendered as a sticky sidebar on desktop, collapsible on mobile.

**Reading time:** Calculated from word count using the `reading-time` package at render time. Displayed in the post header.

**Related posts:** Computed at render time by finding posts that share the most tags with the current post. Show 3 related posts maximum. If fewer than 3 share tags, fill with most recent posts.

**Images:** Stored in `/public/blog/[slug]/`. Use the Next.js `Image` component inside MDX via a custom `MDXImage` component that enforces `width`, `height`, and `alt`. Never use raw `<img>` tags — they break Core Web Vitals.

## Frontmatter Schema

Every post requires all of these fields. Missing fields cause a build error.

```
title: string           # Post title (50–60 chars for SEO)
description: string     # Meta description (150–160 chars)
publishedAt: string     # ISO date — future dates are scheduled (YYYY-MM-DD)
updatedAt: string       # Last updated date — used in schema markup and shown to readers
author: string          # Author display name
tags: string[]          # 2–5 tags — used for related posts and filtering
coverImage: string      # Path relative to /public (e.g. /blog/my-post/cover.jpg)
draft: boolean          # true = never shown, even after publishedAt passes
featured: boolean       # true = shown in featured section on blog index
```

A post is visible when: `draft === false` AND `publishedAt <= now`.

## Folder Structure

```
├── content/
│   └── blog/
│       ├── my-first-post.mdx
│       └── another-post.mdx
├── public/
│   └── blog/
│       ├── my-first-post/
│       │   └── cover.jpg
│       └── another-post/
│           └── cover.jpg
├── app/
│   └── (marketing)/
│       └── blog/
│           ├── page.tsx              # Blog index
│           └── [slug]/
│               └── page.tsx          # Individual post
├── components/
│   └── blog/
│       ├── PostCard.tsx              # Card used on index page
│       ├── PostHeader.tsx            # Title, date, author, reading time
│       ├── TableOfContents.tsx       # Sticky TOC sidebar
│       ├── RelatedPosts.tsx          # 3 related posts at bottom
│       ├── ShareButtons.tsx          # Twitter/X, LinkedIn, Facebook, Reddit, WhatsApp, copy link
│       ├── NewsletterCta.tsx         # Email capture block mid-post
│       ├── BlogSearch.tsx            # Pagefind search modal
│       └── mdx/
│           ├── MDXImage.tsx          # Wraps Next.js Image for use in MDX
│           ├── Callout.tsx           # Info/warning/tip callout blocks
│           └── CodeBlock.tsx         # Shiki-powered code block
└── lib/
    └── blog.ts                       # All blog utility functions (getPosts, getPost, etc.)
```

## SEO Requirements

Blog SEO builds on the site-wide technical SEO foundation in `phase-6-marketing-site/02-technical-seo.md` (canonical URLs, sitemap config, robots.txt, OG tag conventions). The following decisions are blog-specific.

### Per-Post Metadata

Use `generateMetadata()` in `[slug]/page.tsx`. Every post gets:
- Unique `title` and `description` from frontmatter
- `canonical` URL (`https://yourdomain.com/blog/[slug]`)
- `og:title`, `og:description`, `og:image` (cover image), `og:type: article`
- `twitter:card: summary_large_image`
- `article:published_time`, `article:modified_time`, `article:author`

### Article Schema (JSON-LD)

Every post page must include an `Article` JSON-LD block. This is required — not optional — for Google AI Overviews and LLM citation. Include:

```
@type: BlogPosting
headline: post title
description: post description
datePublished: publishedAt
dateModified: updatedAt
author: { @type: Person, name: author }
image: coverImage (absolute URL)
url: canonical URL
publisher: { @type: Organization, name: product name, logo: ... }
```

### BreadcrumbList Schema

Include a `BreadcrumbList` JSON-LD block on every post: Home → Blog → Post Title. Improves navigation clarity in search results.

### Blog Index Metadata

The `/blog` index page gets static metadata: title "Blog — [Product Name]", description summarising the blog's topic focus. It also gets a `CollectionPage` JSON-LD schema listing the 10 most recent posts.

### Sitemap

Include all published blog posts in the sitemap generated by `next-sitemap`. Each post entry includes `lastmod` (from `updatedAt`), `changefreq: weekly`, and `priority: 0.7`. The blog index gets `priority: 0.8`.

### OG Images

Generated dynamically via `@vercel/og` at `app/(marketing)/blog/[slug]/opengraph-image.tsx`. Renders the post title, author, publication date, and brand styling. Each post gets a unique OG image — not a global default.

### Internal Linking

Every post must contain at least 3 internal links — to other blog posts or product pages. This is a content discipline rule, not a technical one, but enforce it in the publishing checklist.

### E-E-A-T Signals

- Author name displayed on every post
- `AuthorCard` component at the bottom of every post. Contains: avatar, full name, title/credentials, 2–3 sentence bio, links to Twitter/X and LinkedIn. Author data lives in a static `authors.ts` config file — not in frontmatter — so it stays consistent across all posts and only needs updating in one place. Multiple authors on a single post are supported (co-authored content). No dedicated author page — the card is the full author presence.
- `updatedAt` date shown on posts that have been refreshed — signals freshness to Google
- Original data, statistics, or first-hand experience in every post — not rehashed generic content

## Performance Requirements

- Blog pages must pass Core Web Vitals: LCP < 2.5s, CLS < 0.1, INP < 200ms
- No client-side JS on post pages except: TOC scroll behaviour, Pagefind search, share button interactions
- All images use Next.js `Image` with explicit `width` and `height` — eliminates CLS
- Cover images are preloaded (`priority` prop on the above-the-fold image)
- Shiki runs server-side — code blocks add zero client JS
- ISR revalidation: `revalidate: 3600` on all blog routes

## Security Considerations

- MDX can execute arbitrary JavaScript if custom components are misconfigured — never allow user-submitted MDX to be rendered. Blog content is author-only (committed to git), so this is not a risk in normal operation.
- OG image route is public and unauthenticated — rate limit it using Upstash Ratelimit directly on the route handler to prevent abuse. It is not an API route and does not go through the API middleware stack.

## What to Test

- Published posts appear on the index and are accessible at `/blog/[slug]`
- Draft posts (`draft: true`) never appear regardless of `publishedAt`
- Future `publishedAt` posts are hidden until their date passes
- `generateMetadata()` returns correct title, description, and OG tags per post
- Article JSON-LD is present and valid on every post page (validate with Google's Rich Results Test)
- BreadcrumbList JSON-LD is present on every post page
- Sitemap includes all published posts with correct `lastmod`
- OG image renders correctly for each post
- Related posts show 3 posts sharing tags (or fall back to most recent)
- Reading time is calculated and displayed
- TOC renders and scrolls correctly on desktop
- Pagefind search returns relevant results
- All images use Next.js `Image` — no raw `<img>` tags in MDX output
- Blog index filters and sorts posts by `publishedAt` descending

## Edge Cases

- **Post with no matching related posts** — no posts share any tags. Fall back to the 3 most recent posts excluding the current one.
- **Two posts with the same slug** — filename collision in `/content/blog/`. The last one read wins, silently hiding the other. Add a build-time check that throws if duplicate slugs are detected.
- **Cover image missing from `/public`** — the post renders but the OG image and cover break. Validate that `coverImage` paths exist at build time and warn loudly.
- **Very long post title in OG image** — title overflows the OG image canvas. Cap title rendering at 2 lines with ellipsis in the `@vercel/og` template.
- **`updatedAt` before `publishedAt`** — editor error. Add a validation check and throw a build error if `updatedAt < publishedAt`.
- **Post with future `publishedAt` accessed directly by URL** — someone with the URL can access an unscheduled post. Return 404 for posts where `publishedAt > now`, even if the slug exists.
- **Pagefind index not rebuilt after new post** — Pagefind only indexes on build. A post deployed without a full build (if using ISR-only) won't appear in search. Acceptable tradeoff — search catches up on next deploy.
- **MDX compilation error in a single post** — one broken post shouldn't crash the entire blog. Catch MDX compilation errors per-file and log them, skipping the broken post rather than crashing the build.

## Implementation Checklist

### Content Pipeline
- [ ] `/content/blog/` folder created with frontmatter schema documented
- [ ] `lib/blog.ts` built with: `getAllPosts()`, `getPostBySlug()`, `getRelatedPosts()`, `getFeaturedPosts()`
- [ ] Frontmatter validation with Zod — missing or invalid fields throw at build time
- [ ] Duplicate slug detection at build time
- [ ] `next-mdx-remote` configured with custom MDX components
- [ ] Shiki configured for server-side syntax highlighting
- [ ] `rehype-slug` and `rehype-autolink-headings` added to MDX pipeline
- [ ] `reading-time` integrated into post rendering

### Routing & Pages
- [ ] `/blog` index page with ISR (`revalidate: 3600`)
- [ ] `/blog/[slug]` post page with ISR (`revalidate: 3600`)
- [ ] Future-dated posts return 404
- [ ] Draft posts return 404
- [ ] `generateStaticParams()` generates paths for all published posts

### SEO
- [ ] `generateMetadata()` per post with title, description, canonical, OG tags
- [ ] Article JSON-LD on every post page
- [ ] BreadcrumbList JSON-LD on every post page
- [ ] CollectionPage JSON-LD on blog index
- [ ] `@vercel/og` OG image route per post (`opengraph-image.tsx`)
- [ ] Blog posts included in `next-sitemap` with `lastmod` and `priority`
- [ ] Author bio section on every post
- [ ] `instrumentation.ts` pings Google sitemap, calls Indexing API, and submits to IndexNow on every production deploy
- [ ] Trigger.dev hourly job detects newly-live scheduled posts and fires all three indexing actions
- [ ] Post snapshot stored in database for the Trigger.dev job to diff against

### Components
- [ ] `PostCard` — used on index, featured section, related posts
- [ ] `PostHeader` — title, author, date, reading time, tags
- [ ] `TableOfContents` — sticky desktop, collapsible mobile
- [ ] `RelatedPosts` — 3 posts by tag matching
- [ ] `ShareButtons` — Twitter/X, LinkedIn, Facebook, Reddit, WhatsApp, copy link
- [ ] `NewsletterCta` — mid-post and end-of-post email capture
- [ ] `BlogSearch` — Pagefind modal triggered by search icon
- [ ] `MDXImage`, `Callout`, `CodeBlock` custom MDX components

### Search
- [ ] Pagefind installed and build script configured
- [ ] Pagefind index generated as part of Railway build step
- [ ] Search modal accessible from blog index and post pages

## Common Mistakes

- Using `<img>` tags in MDX instead of the custom `MDXImage` component (breaks CLS, skips optimisation)
- Missing Article JSON-LD (invisible to AI Overviews and LLM citation engines)
- Forgetting `updatedAt` in schema markup (Google uses it as a freshness signal — stale posts rank lower)
- Not returning 404 for future-dated posts accessed directly by URL (pre-published content leaks)
- Global OG image instead of per-post (every share looks identical, kills CTR)
- No canonical URL (cross-posting without canonicals creates duplicate content penalties)
- Pagefind not included in the Railway build command (search index never generates, search returns nothing)
- `revalidate` set too high (posts go live hours late) or too low (unnecessary revalidation load)
- No internal links in posts (misses the topical authority signal that cluster content builds)
- Rendering MDX on the client side (ships the full MDX bundle to the browser, kills performance)
