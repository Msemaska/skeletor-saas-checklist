# SaaS Launch Checklist

From idea to fully functioning, automated business.
Ordered from foundational to nice-to-have.

`[1]` = Need whilst developing
`[2]` = Need before launch
`[3]` = Need after launch

---

## Phase 0: Strategy & Positioning

1. `[1]` Value proposition / one-liner
2. `[1]` Ideal customer profile (ICP)
3. `[1]` Competitor analysis
4. `[1]` Pricing model (free trial, freemium, flat, tiered, usage-based)
5. `[1]` Domain purchased — **Cloudflare Registrar**
6. `[1]` Business name finalized

## Phase 1: Brand Foundation

1. `[1]` Logo (full, icon, wordmark variants) — **Claude + SVG/React**
2. `[1]` Color palette (primary, secondary, accent, semantic colors) — **Claude + Tailwind config**
3. `[1]` Typography (heading font, body font) — **Google Fonts**
4. `[1]` Favicon and app icons (all sizes) — **Claude + SVG → RealFaviconGenerator**
5. `[1]` App UI design system (components, spacing, tokens) — **Shadcn + Tailwind**

## Phase 2: Testing

1. `[1]` Test infrastructure & strategy (runner, test database, mocks, CI pipeline, coverage, fixtures) — **Vitest + GitHub Actions**
2. `[1]` Unit & integration testing patterns (service functions, Server Actions, RLS, external service mocks) — **Vitest**
3. `[1]` Component testing patterns (React Testing Library, auth-gated UI, forms, states) — **Vitest + React Testing Library**
4. `[2]` E2E testing (critical user paths, auth in E2E, test environment) — **Playwright**
5. `[2]` Performance & cross-browser testing (Lighthouse, load testing, browser matrix, mobile) — **Lighthouse CI + k6 + Playwright**

## Phase 3: Core Product

1. `[1]` Database schema — **Supabase (Postgres)**
2. `[1]` Validation strategy (frontend + backend + database) — **Zod + React Hook Form + Postgres check constraints**
3. `[1]` Auth (sign up, sign in, password reset, social login, 2FA) — **Supabase Auth**
4. `[1]` Data access layer (reads, mutations, service pattern) — **Next.js Server Actions + Supabase Client**
5. `[1]` Cache layer strategy — **Next.js built-in cache + TanStack Query**
6. `[1]` File uploads & image processing (if applicable) — **Supabase Storage + Sharp**
7. `[1]` Notification system (in-app, email, internal alerts) — **Novu + Resend + Supabase Realtime**
8. `[1]` Search (if applicable) — **Supabase pg_trgm / full-text search**
9. `[1]` Credit / usage metering system (limits, tracking, alerts) — **Custom (Supabase)**
10. `[1]` Background & scheduled jobs (async processing, cron tasks, cleanup) — **Trigger.dev + pg_cron**
11. `[1]` AI integration (if applicable) — **Claude API (text + vision) + Recraft (image generation)**
12. `[1]` Error handling & logging strategy — **Pino + Sentry + custom error types**

## Phase 4: UI/UX

1. `[1]` Marketing site layout (navbar, footer, responsive, auth pages) — **Next.js + Shadcn + Tailwind**
2. `[1]` App layout & navigation (sidebar, header bar, command palette, responsive) — **Next.js + Shadcn + Tailwind**
3. `[1]` Component states & error pages (loading, empty, error, populated, no permission, 404, 500) — **Shadcn**
4. `[1]` Data input & display patterns (forms, tables, sorting, filtering, pagination) — **React Hook Form + TanStack Table + Shadcn**
5. `[1]` Overlays & feedback (toasts, modals, dialogs, confirmation) — **Sonner + Shadcn Dialog**
6. `[1]` Onboarding (first-run wizard, tooltip tour, progressive disclosure) — **Custom (Next.js + Zustand) + Onborda**
7. `[1]` Localization & formatting (currency, timezone, date) — **Intl.NumberFormat + date-fns + date-fns-tz**

## Phase 5: Marketing Site & Technical SEO

1. `[2]` Marketing site pages (landing, features, pricing, FAQ, about, branding, contact, waitlist, blog) — **Next.js + MagicUI + Shadcn + MDX**
2. `[2]` Technical SEO (meta tags, OG images, sitemap, robots.txt, canonical URLs, alt text) — **Next.js Metadata API + next-sitemap + @vercel/og**
3. `[2]` Blog scheduling (publishedAt date, auto-publish via ISR) — **Next.js + MDX**
4. `[3]` Post-launch pages (comparison, use case, demo/screenshots) — **Next.js**

## Phase 6: Self-Hosted Infrastructure (if not using Vercel/managed platforms)

1. `[1]` Containerization (Dockerfiles, docker-compose, registry, orchestration) — **Docker + Docker Compose**
2. `[1]` Reverse proxy & SSL (HTTPS termination, domain routing, load balancing) — **Caddy**
3. `[1]` Server provisioning & security (server, firewall, SSH, setup scripts) — **Hetzner + UFW**
4. `[1]` Database hosting (Postgres, connection pooling, replication, backups) — **Docker Postgres + PgBouncer**
5. `[1]` Object storage (buckets, CDN, backups) — **Cloudflare R2 + Cloudflare CDN**
6. `[1]` CI/CD pipeline (build, deploy, zero-downtime, rollback) — **GitHub Actions + SSH**
7. `[1]` Local development (dev environment, seeding, env vars, graceful shutdown) — **docker-compose**
8. `[2]` Server monitoring & log rotation — **Betterstack + logrotate**
9. `[1]` DNS management (domain → server, subdomains) — **Cloudflare DNS**
10. `[1]` Redis / Valkey (cache, rate limiting, job queues — if needed) — **Docker Valkey**

## Phase 7: Managed Infrastructure (if using Vercel/Supabase)

1. `[1]` Hosting & deployment (Vercel, CDN, SSL, preview deployments, rollback) — **Vercel**
2. `[1]` Health check endpoint — **Next.js API route**
3. `[2]` Database backups & disaster recovery — **Supabase (daily automatic + tested restores)**
4. `[2]` File/object storage backups — **Supabase Storage + backup strategy**
5. `[2]` Security headers (CSP, HSTS, X-Frame-Options, Permissions-Policy, Referrer-Policy) — **Next.js next.config headers**
6. `[2]` Dependency scanning — **GitHub Dependabot**

## Phase 8: Business Wrapper

1. `[1]` Transactional emails (templates, triggers, content for all system emails) — **Resend + React Email**
2. `[1]` Stripe integration architecture (webhooks, subscription lifecycle, data model) — **Stripe**
3. `[1]` Stripe sandbox & testing (test mode, test cards, webhook testing, flow verification) — **Stripe Test Mode + Stripe CLI**
4. `[2]` Stripe production launch (live keys, Tax, dunning, customer portal, branding) — **Stripe**
5. `[2]` Legal pages (privacy policy, terms, cookie consent, AUP, DPA, refund policy) — **Claude + Custom pages**

## Phase 9: Customer Support

1. `[2]` Customer support platform (live chat, help desk, knowledge base, feedback widget) — **Tawk.to**

## Phase 10: Observability

1. `[2]` Unified alerting hub (all alerts → one Discord/Slack channel) — **Discord/Slack webhooks**
2. `[2]` Product analytics & feature flags (user behaviour, funnels, session replay) — **PostHog**
3. `[2]` Uptime monitoring & public status page — **UptimeRobot**
4. `[2]` Performance monitoring (Core Web Vitals, page speed) — **Vercel Speed Insights (managed) / Sentry Performance (self-hosted)**
5. `[2]` Revenue metrics (MRR, ARR, churn, LTV, CAC) — **Stripe Dashboard + PostHog**
6. `[2]` SEO monitoring (Search Console, sitemap submission, crawl errors, index coverage, rankings) — **Google Search Console**

## Phase 11: Email & Domain Setup

1. `[2]` Email infrastructure & DNS (mailbox, aliases, routing, subdomains, SPF/DKIM/DMARC) — **Cloudflare Email Routing + Google Workspace + Resend**
2. `[2]` Email deliverability (domain warming, spam testing, reputation monitoring) — **Resend + Mail Tester**

## Phase 12: Security Audit

1. `[2]` Pre-launch security audit (verification of all security decisions, penetration testing, security.txt) — **Manual**

## Phase 13: Content

### Foundational Content
1. `[2]` Foundational copy bank (product descriptions, taglines, bios, FAQ answers, aggregator listings — one source for all platforms) — **Claude**
2. `[2]` Visual asset library (product screenshots, feature GIFs, mockups, social graphics, demo video) — **CleanShot X + Screen Studio + Claude**
3. `[2]` Keyword research (search terms, audience language, topic clusters — informs blog, social, ads, SEO, LLM optimization) — **Claude + Google Search Console**

### Text Content
4. `[3]` Blog posts (how-to guides, tutorials, thought leadership) — **MDX**
5. `[3]` Tweets / X posts (threads, takes, product updates) — **Claude + Postiz (self-hosted)**
6. `[3]` Comparison content ("X vs Y") — **MDX**
7. `[3]` Case studies from real customers — **MDX**
8. `[3]` Changelog / release notes — **MDX**

### Visual Content
9. `[3]` Ad creatives (banners, copy variants) — **Claude + @vercel/og**
10. `[3]` Social media graphics (carousel posts, infographics) — **Claude + @vercel/og**

### Video Content
11. `[3]` Product tutorials and feature updates — **Screen Studio + YouTube**
12. `[3]` Podcast guest appearances (if relevant) — **Manual outreach**

### Content Operations
13. `[3]` Editorial calendar and publishing cadence — **Notion**
14. `[3]` Content repurposing workflow (blog post → social posts → newsletter) — **Claude + Postiz (self-hosted) + Resend**

### LLM Optimization
15. `[3]` Structured comparison content ("X vs Y") — **MDX**
16. `[3]` Monitor LLM mentions (query ChatGPT/Claude/Perplexity for your category) — **Manual**
17. `[3]` Encourage third-party reviews on aggregator sites — **Manual**

## Phase 14: Go-To-Market

1. `[2]` Social media accounts created and branded (Twitter/X, LinkedIn, GitHub, YouTube, Discord) — **Manual**
2. `[2]` Social media scheduling & automation — **Postiz (self-hosted)**
3. `[2]` Backlinks & listings (Product Hunt, HN, Indie Hackers, Reddit, directories, aggregators) — **Manual**
4. `[2]` Email marketing (newsletter signup, welcome drip, launch announcement) — **Resend**
5. `[3]` Paid ads — platforms, budgets, targeting, landing pages — **Google Ads + Meta Ads**
6. `[3]` Ongoing social media cadence — **Postiz (self-hosted)**
7. `[3]` Post-launch listings (niche directories, guest posts, awesome-lists) — **Manual**

## Phase 15: Admin & Operations

1. `[1]` Internal admin dashboard — **Custom (Next.js + Shadcn)**
2. `[3]` User impersonation (for support) — **Supabase service role**
3. `[3]` Manual billing overrides — **Stripe Dashboard**
4. `[3]` User export / data portability — **Custom (Next.js API route)**
5. `[3]` Runbook for common incidents — **Notion**

## Phase 16: Growth & Retention

1. `[3]` Social proof collection system — **Senja**
2. `[3]` Testimonials / case studies pipeline — **Senja**
3. `[3]` Referral / affiliate program — **Rewardful**
4. `[3]` Win-back emails for churned users — **Resend**
5. `[3]` Cancellation survey — **Custom (React Hook Form + Zod)**
6. `[3]` Downgrade-instead-of-cancel offer — **Custom (Stripe + modal)**
7. `[3]` NPS / satisfaction surveys — **PostHog Surveys**

## Phase 17: Accessibility & Internationalization (post-launch)

1. `[3]` WCAG compliance audit — **axe DevTools**
2. `[3]` Screen reader support — **Manual testing (NVDA)**
3. `[3]` Color contrast validation — **axe DevTools**
4. `[3]` RTL support (if applicable) — **Tailwind CSS (rtl: variant)**
5. `[3]` Translation pipeline (if multi-language) — **next-intl**

## Phase 18: API & Integrations (if applicable)

1. `[1]` API versioning strategy (/v1/, header-based, etc.) — **Next.js routing**
2. `[1]` API rate limiting / usage tracking — **Upstash Ratelimit**
3. `[1]` API key management — **Unkey**
4. `[3]` API documentation — **Fumadocs**
5. `[3]` OpenAPI spec / interactive API explorer — **Scalar**
6. `[3]` Webhook system (event delivery to customers) — **Svix**
7. `[3]` Webhook idempotency handling — **Custom (Upstash Redis TTL)**
8. `[3]` SDKs / client libraries — **Custom / openapi-typescript**

## Phase 19: Partnerships & Sales (if B2B)

1. `[3]` Partnership page ("integrate with us") — **Next.js**
2. `[3]` Partner onboarding docs — **Fumadocs / MDX**
3. `[3]` CRM for tracking leads — **Attio**
4. `[3]` Enterprise pricing / annual deal flow — **Custom + Stripe**
5. `[3]` Contract / proposal templates — **Claude + Custom**
6. `[3]` Sales demo environment (seed data, guided walkthrough) — **Supabase seed + Custom**
7. `[3]` Co-marketing with integration partners — **Manual**

## Phase 20: App Store & Marketplace Listings (if applicable)

1. `[3]` Chrome Web Store listing — **Chrome Web Store**
2. `[3]` Slack App Directory listing — **Slack API**
3. `[3]` Shopify App Store listing — **Shopify Partners**
4. `[3]` Browser extension packaging — **Plasmo**
5. `[3]` Marketplace assets (screenshots, descriptions, categories) — **Claude + @vercel/og**
6. `[3]` Review/approval process timelines — **Manual tracking**
