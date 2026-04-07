# Security Headers

---

## What It Is

HTTP response headers that instruct the browser to enforce security policies — a defense layer on top of your application-level security.

## Headers to Set

Configure these in Next.js `next.config` headers section. They apply to all responses.

### Strict-Transport-Security (HSTS)

Tells the browser to always use HTTPS for this domain — even if the user types `http://`.

**Value:** `max-age=31536000; includeSubDomains; preload`
- `max-age=31536000` — enforce HTTPS for 1 year
- `includeSubDomains` — apply to all subdomains
- `preload` — submit to the browser's HSTS preload list (browsers enforce HTTPS before the first visit)

### X-Content-Type-Options

Prevents the browser from MIME-sniffing the response content type.

**Value:** `nosniff`

### X-Frame-Options

Prevents your site from being embedded in an iframe on another domain.

**Value:** `DENY`

### Referrer-Policy

Controls how much referrer information is sent when navigating away from your site.

**Value:** `strict-origin-when-cross-origin`
- Same-origin navigations: full URL is sent as referrer
- Cross-origin navigations: only the origin (domain) is sent, not the full path
- Downgrade (HTTPS → HTTP): no referrer sent

### Permissions-Policy

Controls which browser features (camera, microphone, geolocation, etc.) the page can use. Restrict to only what you need.

**Value:** `camera=(), microphone=(), geolocation=(), interest-cohort=()`
- Disable camera, microphone, and geolocation
- Disable FLoC/interest-cohort

### Content-Security-Policy (CSP)

The most complex and most powerful security header. Controls which sources of content (scripts, styles, images, fonts, connections) the browser is allowed to load.

**Design decisions:**
- CSP is configured in Next.js `next.config`, not in the reverse proxy — it's application-specific and changes with your code
- Start with a restrictive policy and relax as needed (adding allowed sources when things break is safer than starting permissive)
- At minimum: restrict `script-src` to `self` and your trusted domains (Stripe, analytics), restrict `frame-ancestors` to `none` (same as X-Frame-Options DENY)
- Report CSP violations to your monitoring (Sentry can receive CSP reports) — this helps identify attacks and configuration issues
- CSP is the header most likely to break things. Test thoroughly after adding or changing it.

## Where to Configure

**In Next.js (next.config):**
- All headers listed above are set in the `headers` configuration
- CSP may need to be set differently for development vs production (development needs `unsafe-eval` for hot reload, production doesn't)
- Headers apply to all routes by default — you can scope them to specific paths if needed

**In Caddy (self-hosted only):**
- HSTS, X-Content-Type-Options, X-Frame-Options, Referrer-Policy, and Permissions-Policy can also be set in Caddy (covered in phase-6/02-reverse-proxy.md)
- If both Caddy and Next.js set the same header, the last one wins (usually Next.js since Caddy proxies to it)
- CSP is always set in Next.js, not Caddy — it's application-specific

## What to Test

- All security headers present in responses (check with browser DevTools → Network → Response Headers)
- HSTS header includes `includeSubDomains` and `preload`
- X-Frame-Options set to DENY (page cannot be loaded in an iframe on another domain)
- CSP blocks inline scripts (if configured — test that your app still works)
- CSP violations are reported to monitoring
- Headers are present on all routes, not just some

## Edge Cases

- **CSP breaks third-party scripts** — adding a strict CSP may block Stripe.js, analytics scripts, or font loading. Add the specific domains to the CSP allowlist. Test all third-party integrations after changing CSP.
- **CSP in development vs production** — development mode uses `eval()` for hot reload. CSP must allow `unsafe-eval` in development but not in production. Use environment-based CSP configuration.
- **Iframe embedding needed** — if your product has a widget that customers embed on their sites, X-Frame-Options DENY will block it. Use CSP's `frame-ancestors` with specific allowed domains instead of a blanket DENY.
- **Header duplication** — if both Caddy and Next.js set the same header, the browser receives duplicate headers. Some browsers use the first, some use the last. Configure headers in one place to avoid confusion.

## Implementation Checklist

- [ ] Strict-Transport-Security set (max-age=31536000, includeSubDomains, preload)
- [ ] X-Content-Type-Options set to nosniff
- [ ] X-Frame-Options set to DENY
- [ ] Referrer-Policy set to strict-origin-when-cross-origin
- [ ] Permissions-Policy disables unused browser features
- [ ] Content-Security-Policy configured (restrictive, with trusted domains allowed)
- [ ] CSP different for development vs production (unsafe-eval only in dev)
- [ ] CSP violations reported to monitoring (Sentry)
- [ ] All headers verified in browser DevTools
- [ ] All third-party integrations tested after CSP changes (Stripe, analytics, fonts)

## Common Mistakes

- No security headers at all (relying entirely on application-level security — browser-level protection is free and effective)
- CSP too permissive (`unsafe-inline`, `unsafe-eval` in production — defeats the purpose)
- CSP too restrictive without testing (breaks Stripe checkout, analytics, or font loading)
- Setting headers in Caddy AND Next.js (duplicate headers, unpredictable behavior)
- Forgetting to add `includeSubDomains` to HSTS (subdomains are still accessible via HTTP)
- No CSP violation reporting (attacks and misconfigurations go undetected)
- Same CSP for development and production (development breaks with strict CSP, or production runs with loose CSP)
