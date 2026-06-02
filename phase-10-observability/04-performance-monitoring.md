# Performance Monitoring

---

## What It Is

Continuous tracking of how fast your app loads and responds. Performance issues are silent — users don't report "your app is slow," they just leave. Monitoring catches degradation before it becomes churn.

## Provider

Use Sentry Performance — same Sentry SDK already installed for error tracking, add performance monitoring by enabling tracing.

## Core Web Vitals

The metrics that matter for user experience:

| Metric | What It Measures | Target |
|---|---|---|
| LCP (Largest Contentful Paint) | How fast the main content loads | Under 2.5 seconds |
| FID / INP (Interaction to Next Paint) | How fast the app responds to user input | Under 200ms |
| CLS (Cumulative Layout Shift) | How much the layout shifts during load | Under 0.1 |

**Design decisions:**
- Monitor these on real user traffic, not just synthetic tests (Lighthouse). Real User Monitoring (RUM) captures actual device and network conditions.
- Sentry Performance collects RUM automatically via the client-side SDK.
- Track per-page — a slow dashboard page is a different problem than a slow landing page. Identify which pages are below target.

## Server-Side Performance

Beyond client-side vitals, track server-side response times:

**Design decisions:**
- Every Server Action and API route should respond within 200ms at the 95th percentile (excluding file uploads and AI calls which are inherently slower)
- Sentry Performance traces server-side operations automatically — it shows the duration of each request, database queries within the request, and external API calls
- Enable database query tracing — identify slow queries (>100ms) before they affect users
- Set performance budgets: alert when the 95th percentile response time exceeds 500ms on any endpoint (covered in unified alerting, 01-unified-alerting.md)

## Bundle Size Monitoring

JavaScript bundle size directly affects page load time.

**Design decisions:**
- Run Next.js bundle analyzer in CI on every PR that adds or updates dependencies
- Total first-load JavaScript must stay under 200KB (compressed) per page
- Any single chunk over 50KB should be lazy-loaded
- Track bundle size over time — a gradual increase indicates accumulating bloat

## What to Monitor vs What to Fix

Monitoring tells you there's a problem. The fix depends on the cause:

| Symptom | Likely Cause |
|---|---|
| High LCP | Large images not optimised, render-blocking resources, slow server response |
| High INP | Heavy JavaScript on the main thread, unoptimised React re-renders |
| High CLS | Images without dimensions, dynamically injected content, font loading shift |
| Slow API responses | Missing database index, N+1 query, unoptimised join |
| Large bundle | Unused dependencies, large library not lazy-loaded, missing tree shaking |

## What to Test

- Core Web Vitals meet targets on key pages (landing page, dashboard, data-heavy page)
- Server-side response times under 200ms at 95th percentile
- Bundle size under 200KB compressed per page
- Performance data is being collected (Sentry Performance dashboard shows data)
- Slow query detection is working (queries over 100ms are logged)

## Edge Cases

- **Performance degrades gradually** — no single PR causes a spike, but over 3 months LCP goes from 1.5s to 3.5s. Weekly review of performance dashboards catches gradual degradation. Set up alerts for sustained threshold breaches, not just spikes.
- **Performance differs by region** — users in Europe experience 2x latency because the server is in US. Ensure server region matches your primary user base (covered in phase-7-infrastructure/03-server-provisioning.md).
- **Performance tests pass in CI but fail for real users** — CI runs on fast machines with fast networks. RUM data from real users on slow devices and 3G connections tells the true story. Don't rely only on Lighthouse scores.
- **Third-party script kills performance** — an analytics or chat widget loads a 500KB script. Monitor third-party script impact and lazy-load non-critical scripts.

## Implementation Checklist

- [ ] Sentry Performance tracing enabled
- [ ] Core Web Vitals tracked on real user traffic (RUM)
- [ ] LCP under 2.5s, INP under 200ms, CLS under 0.1 on key pages
- [ ] Server-side response times tracked (95th percentile under 200ms)
- [ ] Database query tracing enabled (slow queries >100ms identified)
- [ ] Bundle analyzer run in CI on dependency changes
- [ ] Total first-load JS under 200KB compressed per page
- [ ] Performance dashboard reviewed weekly
- [ ] Alert configured for sustained performance threshold breaches

## Common Mistakes

- Only running Lighthouse in CI (synthetic tests don't reflect real user conditions)
- No server-side performance tracking (client loads fast but API calls take 3 seconds)
- Ignoring bundle size until it's 2MB (gradual bloat goes unnoticed without monitoring)
- Server in a different region than most users (adds 100ms+ to every request)
- Not tracking per-page performance (one slow page hides behind good averages)
- Third-party scripts loaded eagerly (chat widget, analytics, social embeds — lazy-load them)
- No weekly performance review (data accumulates, nobody looks at it)
