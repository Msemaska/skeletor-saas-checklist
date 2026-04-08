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

Google Search Console reports Core Web Vitals from real user data, complementing the synthetic data from Vercel Speed Insights / Sentry Performance (phase-11/04-performance-monitoring.md).

**Design decisions:**
- Check the Core Web Vitals report monthly
- Pages flagged as "Poor" or "Needs Improvement" should be investigated and fixed — poor vitals can affect search rankings
- Cross-reference with your performance monitoring tool for more detail on what's slow

## What to Test

- Google Search Console is verified and showing data
- Sitemap is submitted and accepted (no errors)
- Marketing pages are indexed (appear in the "Indexed" pages report)
- App pages and auth pages are NOT indexed
- No critical crawl errors on launch pages
- Search performance data is appearing (may take 1-2 weeks after launch)

## Edge Cases

- **Pages not being indexed despite no errors** — new sites take time. Google may take 1-2 weeks to crawl and index all pages. If pages aren't indexed after 2 weeks, use the URL Inspection tool to request indexing manually.
- **Sitemap rejected** — the sitemap URL is wrong, the sitemap format is invalid, or the sitemap includes URLs that return 4xx/5xx. Verify the sitemap is accessible and all URLs in it return 200.
- **Spike in crawl errors after a deployment** — a route was removed or renamed without a redirect. Add 301 redirects for old URLs that are no longer valid.
- **Search Console shows queries you don't want to rank for** — your content is being interpreted differently than intended. Review the page content and meta tags to better align with your target keywords.

## Implementation Checklist

- [ ] Google Search Console verified (DNS verification)
- [ ] Sitemap submitted to Google Search Console
- [ ] Bing Webmaster Tools set up (import from Google Search Console)
- [ ] robots.txt verified: marketing pages allowed, app/auth/API pages blocked
- [ ] Index coverage checked (all marketing pages indexed)
- [ ] No critical crawl errors on launch pages
- [ ] Weekly review scheduled for first 3 months (index coverage, crawl errors, search performance)
- [ ] Core Web Vitals report reviewed monthly

## Common Mistakes

- Not submitting the sitemap (Google discovers pages slower, some may not be found)
- App pages being indexed (users find sign-in pages or dashboard URLs in search results — block with robots.txt)
- Not checking crawl errors (broken links and missing pages go undetected)
- Ignoring Search Console after initial setup (problems accumulate undetected)
- Expecting immediate results (SEO takes weeks to months — don't panic if rankings are low initially)
- Not setting up redirects when renaming or removing pages (old URLs return 404, losing any existing search equity)
