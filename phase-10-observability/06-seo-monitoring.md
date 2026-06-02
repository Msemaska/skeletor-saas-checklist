# SEO Monitoring

---

## What It Is

Ongoing monitoring of how your site performs in search engines — are pages being indexed, are they ranking, are there crawl errors, and how is organic traffic trending. This is the observability layer for your marketing site's search presence.

## Provider

Use Google Search Console. It's free, provides data directly from Google (not estimated), and is the authoritative source for how Google sees your site.

## Setup

**Design decisions:**
- Verify site ownership in Google Search Console using DNS verification (TXT record in Cloudflare DNS)
- Submit your sitemap URL to Search Console immediately after the marketing site goes live
- Set up Bing Webmaster Tools as well — Bing can import settings from Google Search Console directly, minimal additional effort

## Instant Indexing with Google Indexing API

Don't wait for Google to discover new pages — tell it directly. The Google Indexing API lets you notify Google the moment a page is published or updated, forcing it into the crawl queue immediately instead of waiting hours or days.

**Design decisions:**
- Set up a Google Cloud service account with Indexing API access, add it as an owner in Search Console
- Call the Indexing API from your CMS/publishing pipeline every time a page is created, updated, or deleted (blog posts, landing pages, programmatic SEO pages)
- Use the `URL_UPDATED` type for new and updated pages, `URL_DELETED` for removed pages
- Batch requests when publishing multiple pages at once (the API supports up to 200 URLs per batch)
- The API has a daily quota (default 200 requests/day) — sufficient for most SaaS publishing cadences. Request a quota increase if you're doing programmatic SEO at scale.
- This does not guarantee indexing — it guarantees Google will crawl the URL promptly. Pages still need to pass Google's quality checks.
- Combine with sitemap submission — the Indexing API handles real-time notifications, the sitemap is the persistent record of all pages

## Instant Indexing with IndexNow

IndexNow is an open protocol supported by Bing, Yandex, and other search engines. One submission notifies all participating engines simultaneously. Use it alongside the Google Indexing API — together they cover the full search engine landscape.

**Design decisions:**
- Generate a single random key (UUID) and store it as an environment variable (`INDEXNOW_KEY`)
- Host the key as a static file at `public/{key}.txt` — the file contents must be the key itself. This is how search engines verify ownership.
- Submit to `https://api.indexnow.org/indexnow` — this single endpoint fans out to all IndexNow-participating engines (Bing, Yandex, Seznam, Naver, and others)
- Submit all public URLs from your sitemap on every production deployment via `instrumentation.ts`
- `instrumentation.ts` runs once when the Next.js server starts. On every Railway deploy the container restarts, so `register()` fires automatically — no separate pipeline step needed
- On startup, fetch your sitemap XML, extract all page URLs, and submit them as a single batch to IndexNow (up to 10,000 URLs per request)
- Only submit URLs that are publicly indexable — the same set in your sitemap (marketing pages, blog posts). Never submit app pages, auth pages, or API routes.
- IndexNow has no meaningful quota — bulk submissions are expected and supported
- This does not guarantee indexing — it guarantees the engines will crawl the submitted URLs promptly

## What to Monitor

### Index Coverage

Verify that Google is indexing the pages you want indexed and not indexing pages you don't.

**Design decisions:**
- Check the "Pages" report in Search Console weekly for the first month, monthly after that
- All marketing pages should show as "Indexed" — if they're not, investigate (blocked by robots.txt, noindex tag, crawl error)
- Auth pages (sign-in, sign-up), app pages (/app/*), and API routes should NOT be indexed — verify they're excluded by robots.txt or noindex meta tag
- A growing count of indexed pages indicates Google is discovering your content. A declining count indicates a problem.

### Crawl Errors

Pages that Google tried to crawl but couldn't.

**Design decisions:**
- Check the "Pages" report for errors weekly during the first month
- Common errors: 404 (page doesn't exist — fix the link or add a redirect), 5xx (server error during crawl — check server logs), redirect loops
- Fix all crawl errors on launch-critical pages immediately. Errors on post-launch pages (comparison, use case) can wait.

### Search Performance

How your pages appear in search results and how users interact with them.

**Design decisions:**
- Monitor: impressions (how often your pages appear in results), clicks (how often users click), average position (where you rank), click-through rate (clicks ÷ impressions)
- Track by page and by query — which pages get traffic and for which search terms
- Review weekly for the first 3 months to understand baseline performance
- A page with high impressions but low CTR has a weak title/description — improve the meta tags
- A page with good position but no impressions is ranking for queries nobody searches — reconsider the target keywords

### Core Web Vitals

Google Search Console reports Core Web Vitals from real user data, complementing the synthetic data from Sentry Performance (phase-10-observability/04-performance-monitoring.md).

**Design decisions:**
- Check the Core Web Vitals report monthly
- Pages flagged as "Poor" or "Needs Improvement" should be investigated and fixed — poor vitals can affect search rankings
- Cross-reference with your performance monitoring tool for more detail on what's slow

## What to Test

- Google Search Console is verified and showing data
- Sitemap is submitted and accepted (no errors)
- Indexing API notifies Google when a page is published, updated, or deleted
- Marketing pages are indexed (appear in the "Indexed" pages report)
- App pages and auth pages are NOT indexed
- No critical crawl errors on launch pages
- Search performance data is appearing (may take 1-2 weeks after launch)
- IndexNow key file is accessible at `https://yourdomain.com/{key}.txt` and returns the key as plain text
- `instrumentation.ts` submits all sitemap URLs to IndexNow on production server start
- IndexNow submission does not fire in development (`NODE_ENV !== 'production'`)
- Only public sitemap URLs are submitted — no app, auth, or API routes

## Edge Cases

- **Pages not being indexed despite no errors** — new sites take time. Google may take 1-2 weeks to crawl and index all pages. If pages aren't indexed after 2 weeks, use the URL Inspection tool to request indexing manually.
- **Sitemap rejected** — the sitemap URL is wrong, the sitemap format is invalid, or the sitemap includes URLs that return 4xx/5xx. Verify the sitemap is accessible and all URLs in it return 200.
- **Spike in crawl errors after a deployment** — a route was removed or renamed without a redirect. Add 301 redirects for old URLs that are no longer valid.
- **Search Console shows queries you don't want to rank for** — your content is being interpreted differently than intended. Review the page content and meta tags to better align with your target keywords.
- **Indexing API quota exceeded** — you're publishing more pages than the daily quota allows. Request a quota increase via Google Cloud Console, or prioritize high-value pages and let the sitemap handle the rest.
- **Indexing API call succeeds but page isn't indexed** — the API only guarantees a crawl, not indexing. Check that the page returns 200, has no noindex tag, and meets Google's quality standards.
- **IndexNow key file returns 404** — the key filename in `public/` doesn't match the key value in the environment variable. They must be identical. Verify the file is named `{INDEXNOW_KEY}.txt` and contains the key as its only content.
- **IndexNow submission fires in development** — the `register()` function in `instrumentation.ts` must gate on `process.env.NODE_ENV === 'production'`. Submitting localhost URLs to IndexNow is a no-op but wastes a network call and pollutes logs.
- **Sitemap not yet built when `instrumentation.ts` runs** — the sitemap is generated at build time before the server starts, so it will always be present when `register()` fires. If you're using a dynamic sitemap route instead of a static file, fetch it from the live URL rather than the filesystem.
- **IndexNow submission includes private pages** — parse your sitemap (which already excludes app/auth/API routes) rather than crawling your own routes. The sitemap is the source of truth for what's public.

## Implementation Checklist

- [ ] Google Search Console verified (DNS verification)
- [ ] Sitemap submitted to Google Search Console
- [ ] Bing Webmaster Tools set up (import from Google Search Console)
- [ ] robots.txt verified: marketing pages allowed, app/auth/API pages blocked
- [ ] Index coverage checked (all marketing pages indexed)
- [ ] No critical crawl errors on launch pages
- [ ] Google Indexing API set up (service account created, added as Search Console owner)
- [ ] Indexing API integrated into publishing pipeline (new/updated/deleted pages trigger API calls)
- [ ] `INDEXNOW_KEY` environment variable generated (UUID) and set in Railway production environment
- [ ] Key file created at `public/{INDEXNOW_KEY}.txt` containing the key as plain text
- [ ] `instrumentation.ts` configured to submit all sitemap URLs to IndexNow on every production server start
- [ ] Key file verified accessible at the production domain
- [ ] Weekly review scheduled for first 3 months (index coverage, crawl errors, search performance)
- [ ] Core Web Vitals report reviewed monthly

## Common Mistakes

- Not submitting the sitemap (Google discovers pages slower, some may not be found)
- App pages being indexed (users find sign-in pages or dashboard URLs in search results — block with robots.txt)
- Not checking crawl errors (broken links and missing pages go undetected)
- Ignoring Search Console after initial setup (problems accumulate undetected)
- Expecting immediate results (SEO takes weeks to months — don't panic if rankings are low initially)
- Not setting up redirects when renaming or removing pages (old URLs return 404, losing any existing search equity)
- Submitting URLs to IndexNow manually instead of automating via `instrumentation.ts` (creates a step that gets skipped)
- Using a hardcoded key string instead of an environment variable (key gets committed to the repo)
- Submitting to a specific engine's IndexNow endpoint instead of `api.indexnow.org` (misses all other participating engines)
