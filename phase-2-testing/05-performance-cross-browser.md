# Performance & Cross-Browser Testing

---

## Performance Testing

### Lighthouse

Automated performance audits for every key page.

**Design decisions:**
- Run Lighthouse CI in the GitHub Actions pipeline on every push to main
- Target score: 90+ on all key pages (landing page, dashboard, settings) for Performance, Accessibility, Best Practices, and SEO
- Key pages to test: landing page, sign-in page, dashboard, one data-heavy page (project list or table view)
- Fail the CI build if the Performance score drops below 90 on any key page
- Track scores over time — a gradual decline indicates accumulating performance debt

### Load Testing

Verify the app handles concurrent users without degrading.

**Design decisions:**
- Use k6 for load testing — scriptable, runs in CI, produces clear reports
- Test against a staging environment, not production — load tests would affect real users
- Baseline test: 100 concurrent users performing common actions (sign in, load dashboard, create entity, navigate between pages) for 5 minutes
- The app should respond within 200ms at the 95th percentile under this load. If it doesn't, investigate before launch.
- Run load tests before launch and before any major infrastructure change (adding a cache layer, changing database config)

### Database Performance

Catch slow queries before they affect users.

**Design decisions:**
- Enable `pg_stat_statements` in the database (covered in phase-3-core-product/01-database-schema.md for managed, phase-7-infrastructure/04-database-hosting.md for self-hosted)
- Review the 10 slowest queries weekly during development and before launch
- Every query that takes more than 100ms should be investigated — check for missing indexes, inefficient joins, or unnecessary data fetching
- Use `EXPLAIN ANALYZE` on suspicious queries to understand the query plan
- After adding indexes, verify the query plan actually uses them

### Bundle Size

Keep the JavaScript bundle small for fast page loads.

**Design decisions:**
- Monitor bundle size with Next.js's built-in bundle analyzer (`@next/bundle-analyzer`)
- Total first-load JavaScript should be under 200KB (compressed) for any page
- Investigate any single chunk larger than 50KB — lazy-load it
- Check bundle size before merging PRs that add new dependencies

## Cross-Browser Testing

### Browser Matrix

Test on these browsers — they cover 98%+ of users:

| Browser | Engine | Priority |
|---|---|---|
| Chrome (latest) | Chromium | Must pass |
| Firefox (latest) | Gecko | Must pass |
| Safari (latest) | WebKit | Must pass |
| Edge (latest) | Chromium | Covered by Chrome tests |

Playwright tests run against all three engines (Chromium, Firefox, WebKit) automatically.

**Design decisions:**
- Playwright E2E tests run against Chromium by default in development. In CI, run against all three engines.
- Manual testing is focused on Safari — it has the most rendering and API differences from Chromium. Test key flows in Safari before launch.
- Do not test on Internet Explorer — it's dead. Do not test on older browser versions — only current stable releases.

### Mobile Testing

| Device | Priority |
|---|---|
| iOS Safari (iPhone) | Must test — most common mobile browser |
| Android Chrome | Must test — most common Android browser |

**Design decisions:**
- Playwright can emulate mobile devices (viewport size, touch events, user agent). Use this for automated mobile testing in CI.
- Manual testing on a real iPhone and Android device before launch — emulators don't catch everything (touch target sizes, scroll behavior, Safari-specific bugs).
- Test at all Tailwind breakpoints: default (mobile), sm (640px), md (768px), lg (1024px), xl (1280px)
- Verify: sidebar collapses to drawer on mobile, tables convert to cards, forms stack to single column, hamburger menu works

### What to Test Across Browsers

Focus on things that differ between browsers:

- **CSS rendering** — flexbox/grid layouts, backdrop-filter (blur), sticky positioning, scroll behavior
- **Form behavior** — date pickers, file inputs, autofill styling
- **JavaScript APIs** — Intl.NumberFormat output differences, Clipboard API, ResizeObserver
- **Font rendering** — fonts render differently across OS/browser. Test that text doesn't overflow containers.
- **Touch interactions** — hover states don't exist on touch devices. Verify that features depending on hover (sidebar tooltips, dropdown on hover) have touch alternatives.
- **Safari-specific** — safe area insets (notch), overscroll behavior, PWA behavior, 100vh not matching actual viewport height

## What to Test

- Lighthouse score 90+ on landing page, sign-in, dashboard, and one data-heavy page
- Total first-load JS under 200KB compressed on every page
- 95th percentile response time under 200ms at 100 concurrent users
- Slowest database queries under 100ms
- All E2E critical paths pass on Chromium, Firefox, and WebKit
- Mobile layout correct at all Tailwind breakpoints
- Sidebar drawer works on mobile
- Tables convert to cards on mobile
- Forms stack to single column on mobile
- No hover-dependent features without touch alternatives

## Edge Cases

- **Lighthouse score drops after adding a feature** — a new dependency or a large component pushes the bundle over the threshold. Investigate: can the dependency be lazy-loaded? Can the component be code-split? Address before merging.
- **Load test reveals a bottleneck** — one API route is 10x slower than others under load. Usually a missing database index or an N+1 query. Fix the query, add the index, re-test.
- **Safari renders a layout differently** — Safari's flexbox and grid implementations have subtle differences. Test sticky headers, backdrop-filter, and CSS gap support specifically.
- **Mobile viewport height issue** — `100vh` on mobile Safari includes the address bar, causing content to overflow. Use `100dvh` (dynamic viewport height) instead.
- **Touch target too small** — buttons and links must be at least 44x44px for comfortable touch interaction. Check interactive elements on mobile.
- **Font loading causes layout shift** — if web fonts load after the initial render, text reflows and the layout shifts. Verify that `next/font` is configured correctly and CLS (Cumulative Layout Shift) is near zero.

## Implementation Checklist

### Performance
- [ ] Lighthouse CI configured in GitHub Actions
- [ ] Lighthouse score 90+ enforced on key pages
- [ ] k6 load test script written for common user flows
- [ ] Load test run against staging before launch (100 concurrent users, 5 minutes)
- [ ] 95th percentile response time under 200ms verified
- [ ] pg_stat_statements enabled, 10 slowest queries reviewed
- [ ] All queries under 100ms (or investigated and optimized)
- [ ] Bundle analyzer run, total first-load JS under 200KB compressed per page
- [ ] No single chunk larger than 50KB without justification

### Cross-Browser
- [ ] Playwright E2E tests run against Chromium, Firefox, and WebKit in CI
- [ ] Manual Safari testing on a real device before launch
- [ ] Manual testing on real iPhone (iOS Safari) and Android (Chrome) before launch
- [ ] All Tailwind breakpoints verified (mobile, sm, md, lg, xl)
- [ ] No hover-dependent features without touch alternatives
- [ ] 100dvh used instead of 100vh for mobile viewport height
- [ ] Touch targets at least 44x44px on mobile

## Common Mistakes

- No performance testing before launch (discover the dashboard takes 8 seconds to load after users are using it)
- Load testing against production (real users get slow responses during the test)
- Only testing on Chrome (Safari has the most differences — ship with Safari bugs undetected)
- Testing on emulators only (real devices have different performance, touch behavior, and rendering)
- Ignoring bundle size (adding a 500KB charting library for one dashboard widget — lazy-load it)
- Using 100vh on mobile (content hidden behind the address bar on Safari)
- Hover-dependent features with no touch alternative (tooltips on desktop, nothing on mobile)
- No Lighthouse in CI (performance degrades gradually, nobody notices until users complain)
- Testing old browser versions (wasting time on browsers nobody uses)
