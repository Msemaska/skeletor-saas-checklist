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
7. `[1]` Discounting strategy (when, how much, annual, launch, win-back, enterprise, Stripe coupons) — **Custom policy**
8. `[1]` User journey map (trigger, evaluation, sign-up, activation/aha moment, habit, expansion) — **Custom (per ICP)**

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

1. `[1]` Database schema — **Neon (Postgres)**
2. `[1]` Validation strategy (frontend + backend + database) — **Zod + React Hook Form + Postgres check constraints**
3. `[1]` Auth (sign up, sign in, password reset, social login, 2FA) — **Neon Auth (Better Auth)**
4. `[1]` Data access layer (reads, mutations, service pattern) — **Next.js Server Actions + Neon serverless driver**
5. `[1]` Cache layer strategy — **Next.js built-in cache + TanStack Query**
6. `[1]` File uploads & image processing (if applicable) — **Cloudflare R2 + Sharp**
7. `[1]` Notification system (in-app, email, internal alerts) — **Novu + Resend**
8. `[1]` Search (if applicable) — **Postgres pg_trgm / full-text search (Neon)**
9. `[1]` Credit / usage metering system (limits, tracking, alerts) — **Custom (Neon)**
10. `[1]` Background & scheduled jobs (async processing, cron tasks, cleanup) — **Trigger.dev + pg_cron**
11. `[1]` AI integration (if applicable) — **Claude API (text + vision) + Recraft (image generation)**
12. `[1]` Error handling & logging strategy — **Pino + Sentry + custom error types**
13. `[1]` Team / organization management & RBAC (orgs, roles, permissions, member invites, seats — if B2B / multi-user) — **Neon Auth (Better Auth Organization plugin)**
14. `[1]` Audit logging (append-only trail of user + admin actions for security & compliance) — **Custom (Neon append-only table)**

## Phase 4: Public / Customer API (if applicable)

1. `[1]` API foundation (versioning, rate limiting, key management) — **Next.js routing + Upstash Ratelimit + Unkey**
2. `[3]` API documentation & explorer — **Fumadocs + Scalar**
3. `[3]` Webhook system (event delivery to customers, idempotency) — **Svix + Custom**
4. `[3]` SDKs / client libraries — **openapi-typescript**
5. `[3]` MCP server (expose product capabilities to AI assistants and agents) — **MCP TypeScript SDK**
6. `[3]` CLI (developer-facing command-line interface for power users and automation) — **Commander.js**
7. `[3]` Native integrations (connect to tools users already work in — Zapier, Make, n8n) — **Zapier Developer Platform + Make SDK**

## Phase 5: UI/UX

1. `[1]` Marketing site layout (navbar, footer, responsive, auth pages) — **Next.js + Shadcn + Tailwind**
2. `[1]` App layout & navigation (sidebar, header bar, command palette, responsive) — **Next.js + Shadcn + Tailwind**
3. `[1]` Component states & error pages (loading, empty, error, populated, no permission, 404, 500) — **Shadcn**
4. `[1]` Data input & display patterns (forms, tables, sorting, filtering, pagination) — **React Hook Form + TanStack Table + Shadcn**
5. `[1]` Overlays & feedback (toasts, modals, dialogs, confirmation) — **Sonner + Shadcn Dialog**
6. `[1]` Onboarding (first-run wizard, tooltip tour, progressive disclosure) — **Custom (Next.js + Zustand) + Onborda**
7. `[1]` Localization & formatting (currency, timezone, date) — **Intl.NumberFormat + date-fns + date-fns-tz**
8. `[1]` Component inventory (primitive install list, composite components for app shell, auth, billing, data display, marketing) — **Shadcn + MagicUI**
9. `[1]` Account & settings pages (profile, change password, active sessions, connected accounts, danger zone) — **Shadcn + React Hook Form + Neon Auth**

## Phase 6: Marketing Site & Technical SEO

1. `[2]` Marketing site pages (landing, features, pricing, FAQ, about, branding, contact, waitlist, blog) — **Next.js + MagicUI + Shadcn + MDX**
2. `[2]` Technical SEO (meta tags, OG images, sitemap, robots.txt, canonical URLs, alt text) — **Next.js Metadata API + next-sitemap + @vercel/og**
3. `[2]` Blog system (MDX pipeline, scheduling, search, TOC, related posts, Article schema, OG images, Pagefind) — **next-mdx-remote + Shiki + Pagefind + @vercel/og**
4. `[3]` Post-launch pages (comparison, use case, demo/screenshots) — **Next.js**

## Phase 7: Infrastructure

1. `[1]` Containerization (Dockerfiles, docker-compose for local dev, registry) — **Docker + Docker Compose**
2. `[1]` Hosting & deployment (containers, CDN, SSL, preview deployments, rollback) — **Railway**
3. `[1]` Database hosting & backups (Postgres, connection pooling, branching, disaster recovery) — **Neon**
4. `[1]` Object storage & backups (buckets, CDN, file backup strategy) — **Cloudflare R2 + Cloudflare CDN**
5. `[1]` CI/CD pipeline (build, deploy, zero-downtime, rollback) — **GitHub Actions + Railway auto-deploy**
6. `[1]` Local development (dev environment, seeding, env vars, graceful shutdown) — **Docker Compose**
7. `[1]` DNS management (domain → app, subdomains) — **Cloudflare DNS**
8. `[1]` Redis / Valkey (cache, rate limiting, job queues — if needed) — **Railway add-on (Valkey)**
9. `[1]` Health check endpoint — **Next.js API route**
10. `[2]` Security headers (CSP, HSTS, X-Frame-Options, Permissions-Policy, Referrer-Policy) — **Next.js next.config headers**
11. `[2]` Dependency scanning — **GitHub Dependabot**

## Phase 8: Business Wrapper

1. `[1]` Transactional emails (templates, triggers, content for all system emails) — **Resend + React Email**
2. `[1]` Stripe integration architecture (webhooks, subscription lifecycle, data model) — **Stripe**
3. `[1]` Stripe sandbox & testing (test mode, test cards, webhook testing, flow verification) — **Stripe Test Mode + Stripe CLI**
4. `[2]` Stripe production launch (live keys, Tax, dunning, customer portal, branding) — **Stripe**
5. `[2]` Legal pages (privacy policy, terms, cookie consent, AUP, DPA, refund policy) — **Claude + Custom pages**
6. `[3]` GDPR self-service (user-initiated data export + account deletion — right to access & erasure) — **Custom (Next.js Server Actions + R2 export)**

## Phase 9: Customer Support

1. `[2]` Customer support platform (live chat, help desk, knowledge base, feedback widget) — **Tawk.to**

## Phase 10: Observability

1. `[2]` Unified alerting hub (all alerts → one Discord channel) — **Discord webhooks**
2. `[2]` Product analytics & feature flags (user behaviour, funnels, session replay) — **PostHog**
3. `[2]` Uptime monitoring & public status page — **UptimeRobot**
4. `[2]` Performance monitoring (Core Web Vitals, page speed) — **Sentry Performance**
5. `[2]` Revenue metrics (MRR, ARR, churn, LTV, CAC) — **Stripe Dashboard + PostHog**
6. `[2]` SEO monitoring (Search Console, sitemap submission, crawl errors, index coverage, rankings, Indexing API) — **Google Search Console + Google Indexing API**

## Phase 11: Email & Domain Setup

1. `[2]` Email infrastructure & DNS (mailbox, aliases, routing, subdomains, SPF/DKIM/DMARC) — **Cloudflare Email Routing + Google Workspace + Resend**
2. `[2]` Email deliverability (domain warming, spam testing, reputation monitoring) — **Resend + Mail Tester**

## Phase 12: Security Audit

1. `[2]` Pre-launch security audit (verification of all security decisions, penetration testing, security.txt) — **Manual**

## Phase 13: Content

1. `[2]` Foundational copy bank (product descriptions, taglines, bios, FAQ answers, aggregator listings — one source for all platforms) — **Claude**
2. `[2]` Visual asset library (product screenshots, feature GIFs, mockups, social graphics, demo video) — **CleanShot X + Screen Studio + Claude**
3. `[2]` Keyword research (search terms, audience language, topic clusters — informs blog, social, ads, SEO, LLM optimization) — **Claude + Google Search Console**
4. `[3]` Blog posts (how-to guides, tutorials, thought leadership) — **MDX**
5. `[3]` Tweets / X posts (threads, takes, product updates) — **Claude + Postiz (self-hosted)**
6. `[3]` Comparison content ("X vs Y") — **MDX**
7. `[3]` Case studies from real customers — **MDX**
8. `[3]` Changelog / release notes — **MDX**
9. `[3]` Programmatic SEO pages (template-based pages at scale — location, integration, use case variants) — **Next.js**
10. `[3]` Visual content (ad creatives, social graphics, carousel posts, infographics) — **Claude + @vercel/og**
11. `[3]` Video content (product tutorials, feature updates, podcast guest appearances) — **Screen Studio + YouTube**
12. `[3]` Content operations (editorial calendar, repurposing workflow, distribution schedule) — **Notion + Claude + Postiz (self-hosted) + Resend**
13. `[3]` LLM optimization (monitoring mentions, third-party reviews, structured content for AI citation) — **Manual + G2 + Capterra**
14. `[3]` Free utility tools (mini-tools tangential to the product as SEO/lead-gen assets — sitemap crawlers, validators, generators) — **Next.js + Server Actions + Upstash Ratelimit**

## Phase 14: Go-To-Market

1. `[2]` Social media setup & automation (accounts, branding, scheduling, ongoing cadence) — **Postiz (self-hosted)**
2. `[2]` Launch distribution (Product Hunt, HN, Indie Hackers, Reddit, directories, aggregators) — **Manual**
3. `[2]` Email marketing (newsletter signup, welcome drip, launch announcement) — **Resend**
4. `[3]` Paid ads (platforms, budgets, targeting, landing pages) — **Google Ads + Meta Ads**
5. `[3]` Post-launch distribution (niche directories, guest posts, awesome-lists) — **Manual**
6. `[2]` Brand entity footprint (schema.org Organization, social/Web2 brand reservation, SaaS directory citations, press releases, llms.txt) — **Manual + EIN Presswire**

## Phase 15: Admin & Operations

1. `[2]` Admin dashboard & operations tools (dashboard, impersonation, billing overrides, data export) — **Custom (Next.js + Shadcn) + Stripe Dashboard**
2. `[3]` Incident runbook (what to do when things break) — **Notion**

## Phase 16: Growth & Retention

1. `[3]` Retention mechanics (social proof collection, win-back emails, cancellation flow with survey + downgrade offer) — **Senja + Resend + Custom**
2. `[3]` Referral / affiliate program — **Rewardful**
3. `[3]` NPS / satisfaction surveys — **PostHog Surveys**

## Phase 17: Accessibility & Internationalization (post-launch)

1. `[3]` Accessibility audit (WCAG compliance, screen reader, color contrast) — **axe DevTools + Manual testing (NVDA)**
2. `[3]` Internationalization (RTL support, translation pipeline) — **Tailwind CSS + next-intl**

## Phase 18: Partnerships & Sales (if B2B)

1. `[3]` Partnerships (partner page, onboarding docs, co-marketing) — **Next.js + Fumadocs**
2. `[3]` Sales operations (CRM, enterprise pricing, contracts, demo environment) — **Attio + Stripe + Claude**

## Phase 19: App Store & Marketplace Listings (if applicable)

1. `[3]` App store & marketplace listings (Chrome, Slack, Shopify, packaging, assets, review timelines) — **Plasmo + Claude + @vercel/og**
2. `[3]` IDE extensions (VS Code, JetBrains — surface product inside the developer's editor) — **VS Code Extension API + JetBrains Plugin SDK**
