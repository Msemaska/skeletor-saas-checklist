# Skeletor

A comprehensive, opinionated reference architecture for building a SaaS business from idea to fully functioning automated business.

---

## What Is This

Skeletor is a knowledge base — not a codebase, not a starter template, not a boilerplate. It's a collection of design decisions, guidelines, and checklists that tell an AI exactly how to build every system a SaaS product needs.

The problem it solves: AI makes inconsistent decisions across conversations. One session builds auth with middleware-only protection. The next adds RLS but forgets rate limiting. The third skips email verification entirely. Every conversation starts from zero with no institutional knowledge.

Skeletor eliminates this. Every design decision is pre-made, documented, and opinionated. An AI reading these files should produce nearly identical implementations every time, regardless of which conversation it's in.

## How It Works

The project is structured as a phased checklist with deep-dive reference files:

- **`saas-launch-checklist.md`** — the master checklist of everything a SaaS business needs, organized in 21 phases from strategy to app store listings
- **`phase-X/` directories** — deep-dive files for each checklist item, containing the actual design decisions, edge cases, security considerations, and implementation checklists
- **`CLAUDE.md`** — the rules for how these files are written and maintained

## The Phases

| Phase | Name | When |
|---|---|---|
| 0 | Strategy & Positioning | Before everything |
| 1 | Brand Foundation | Before design |
| 2 | Testing | Before building (test infrastructure) |
| 3 | Core Product | The product itself |
| 4 | Public / Customer API | If applicable |
| 5 | UI/UX | Interface patterns |
| 6 | Marketing Site & Technical SEO | The public website |
| 7 | Self-Hosted Infrastructure | If not using Vercel |
| 8 | Managed Infrastructure | If using Vercel/Supabase |
| 9 | Business Wrapper | Billing, emails, legal |
| 10 | Customer Support | Help desk, knowledge base |
| 11 | Observability | Monitoring, analytics, alerting |
| 12 | Email & Domain Setup | Email infrastructure |
| 13 | Security Audit | Pre-launch verification |
| 14 | Content | Copy, visuals, blog, SEO content |
| 15 | Go-To-Market | Distribution, launch, ads |
| 16 | Admin & Operations | Internal tooling, runbooks |
| 17 | Growth & Retention | Post-launch optimization |
| 18 | Accessibility & i18n | Post-launch polish |
| 19 | Partnerships & Sales | If B2B |
| 20 | App Store & Marketplace | If applicable |

## How to Use It

### Starting a New Project

1. **Clone this repo** as the foundation for your new project. Don't modify the original — it's your reusable template.
2. **Work through Phase 0** — fill in your specific strategy: value proposition, ICP, competitors, pricing, domain, name.
3. **Create your product-specific docs** — `docs/copy-bank.md` and `docs/keyword-research.md` are outputs you create for each product.
4. **Work phase by phase** — each phase builds on the previous one. Don't skip ahead.

### Working with AI

When you start building a feature, give the AI the relevant deep-dive file as context:

- Building auth → provide `phase-3/03-auth.md`
- Setting up Stripe → provide `phase-9/02-stripe-architecture.md`
- Creating the blog → provide `phase-14/04-blog-posts.md`
- Deploying self-hosted → provide `phase-7/01-containerization.md` through `phase-7/10-redis-valkey.md`

The AI gets the architecture, design decisions, edge cases, what to test, security considerations, and common mistakes — all in one file. No improvisation needed.

For complex features that span multiple systems, provide multiple files:
- Auth + database schema + validation → three files covering all the decisions that interact

### Reviewing AI Output

After the AI builds something, use the deep-dive file to review:
- Does it handle all the edge cases listed?
- Does it follow the security considerations?
- Does the "What to Test" section have corresponding tests?
- Is it making any of the "Common Mistakes"?

## Human Task List

When starting a new SaaS project with Skeletor, the AI handles the code — but there are tasks only a human can do. Before or during development, the AI should generate a comprehensive to-do list of everything the human needs to handle.

### Accounts to Register

| Service | What to Do | When Needed |
|---|---|---|
| Cloudflare | Register domain, set up DNS | Phase 0 |
| Google Workspace | Set up email (hello@yourdomain.com) | Phase 12 |
| Supabase | Create project, get API keys | Phase 3 |
| Stripe | Create account, get test + live API keys | Phase 9 |
| Resend | Create account, add sending domains | Phase 9 + 12 |
| Sentry | Create project, get DSN | Phase 3 |
| PostHog | Create project, get API key | Phase 11 |
| UptimeRobot | Create account, add monitors | Phase 11 |
| Tawk.to | Create account, configure widget | Phase 10 |
| Google Search Console | Verify site ownership | Phase 11 |
| Google OAuth | Create OAuth app, get client ID/secret | Phase 3 |
| Apple Developer | If Apple Sign-In needed ($99/year) | Phase 3 |
| GitHub | Create org, configure Dependabot | Phase 8 |
| Vercel | Connect repo, configure domains | Phase 8 |
| Hetzner | Provision server (if self-hosted) | Phase 7 |
| Postiz | Self-host instance for social scheduling | Phase 15 |
| Rewardful | Create account for referral program | Phase 17 |
| Senja | Create account for testimonial collection | Phase 17 |
| Attio | Create CRM account (if B2B) | Phase 19 |

### Keys & Credentials to Collect

| Credential | From | Store In |
|---|---|---|
| Supabase URL + Anon Key + Service Role Key | Supabase dashboard | .env |
| Stripe Secret Key (test) | Stripe dashboard | .env.local |
| Stripe Secret Key (live) | Stripe dashboard | .env.production |
| Stripe Publishable Key (test + live) | Stripe dashboard | .env |
| Stripe Webhook Signing Secret (test + live) | Stripe dashboard | .env |
| Resend API Key | Resend dashboard | .env |
| Sentry DSN | Sentry project settings | .env |
| PostHog API Key | PostHog project settings | .env |
| Google OAuth Client ID + Secret | Google Cloud Console | .env |
| Apple Sign-In Key + Key ID + Team ID | Apple Developer portal | .env |

### Manual Setup Tasks

| Task | When | Notes |
|---|---|---|
| Design logo and brand assets | Phase 1 | Use Claude to generate SVG |
| Write value proposition and copy bank | Phase 0 + 14 | Fill in docs/copy-bank.md |
| Configure Stripe products and prices | Phase 9 | Create in test mode first |
| Set up DNS records (SPF, DKIM, DMARC) | Phase 12 | Follow phase-12/01-email-infrastructure.md |
| Submit Google OAuth consent screen for verification | Phase 3 | Takes 2-4 weeks — start early |
| Start domain warming | Phase 12 | 4 weeks before launch |
| Submit to BetaList and directories | Phase 15 | 3 weeks before launch |
| Write initial knowledge base articles | Phase 10 | Top 10 expected questions |
| Record product demo video | Phase 14 | After core features are built |
| Capture product screenshots | Phase 14 | After UI is stable |
| Draft Product Hunt listing and maker comment | Phase 15 | From copy bank |
| Set up social media accounts | Phase 15 | From copy bank bios |
| Create Discord server for alerting | Phase 11 | Three channels: critical, warning, info |
| Test backup restore | Phase 8 | At least once before launch |
| Run security audit | Phase 13 | After all features built, before launch |
| Test with real $1 payment on Stripe live mode | Phase 9 | Before public launch |

### The AI's Job vs The Human's Job

| AI Does | Human Does |
|---|---|
| Write all application code | Register for services and collect API keys |
| Implement auth, database, API, UI | Review and approve design decisions |
| Set up testing infrastructure | Run manual tests on real devices |
| Configure CI/CD pipelines | Set up DNS records and domain configuration |
| Write email templates | Verify email deliverability on real inboxes |
| Generate marketing copy drafts | Edit copy for voice, accuracy, and personal insight |
| Build blog infrastructure | Record demo videos and capture screenshots |
| Implement Stripe integration | Create Stripe products/prices and submit for verification |
| Write legal page drafts | Have a lawyer review (if handling sensitive data) |
| Set up monitoring dashboards | Respond to alerts and incidents |
| Generate content drafts | Add personal anecdotes, verify stats, edit for quality |

## Ethos

### Opinionated by Design

Every file makes decisions. It doesn't present options or explain tradeoffs — it states what to do. The goal is determinism: given the same file, two AI sessions produce the same architecture.

### User-First Always

When a decision involves a tradeoff between developer convenience and user experience, choose the user. No surprise charges, no deleting user data, no dark patterns, no friction that serves metrics instead of people.

### Proven Defaults

Recommend what works for 90% of SaaS products. Skip niche tools, bleeding-edge patterns, and anything that requires explaining why it's better than the obvious choice. If it's been the standard for 3 years and still works, it's the right recommendation.

### Security Is Not an Afterthought

Every system file includes security considerations. The security audit (Phase 13) verifies all of them. Security decisions live next to the feature, not in a separate document that nobody reads.

### Test What You Build

Every deep-dive file includes a "What to Test" section. Test requirements travel with the feature, not in a disconnected testing document. If the AI builds a feature without tests for everything in that section, it shipped incomplete work.

### No Context Bloat

Files contain decisions, not education. No explaining what JWT is. No comparing 5 tools and picking one. No "you could also do X." State the decision. Move on. The reader (AI or human) doesn't need to understand the tradeoffs — it needs to know what to do.

## Maintaining Skeletor

- When a tool or best practice changes, update the specific deep-dive file
- When you discover a gap (AI keeps getting something wrong), add it as an edge case or new section
- After every project, retrospective: what did AI get wrong that Skeletor should have prevented?
- Never let it go stale — if a decision in a project contradicts Skeletor, update Skeletor

## Tech Stack (Default)

| Layer | Choice |
|---|---|
| Framework | Next.js (App Router) |
| Runtime | Node.js |
| Package Manager | npm |
| Styling | Tailwind CSS |
| State | Zustand (client), TanStack Query (server) |
| Backend | Supabase (Auth, Database, Storage) |
| Forms | React Hook Form + Zod |
| UI | Shadcn (primitives), MagicUI (animated/decorative) |
| Payments | Stripe |
| Email | Resend + React Email |
| Analytics | PostHog |
| Error Tracking | Sentry |
| Uptime | UptimeRobot |
| Support | Tawk.to |
| Social Scheduling | Postiz (self-hosted) |
| Testing | Vitest + Playwright |
| CI/CD | GitHub Actions |
| Hosting (managed) | Vercel |
| Hosting (self-hosted) | Hetzner + Docker + Caddy |
| DNS + CDN | Cloudflare |
| Object Storage | Cloudflare R2 (self-hosted) / Supabase Storage (managed) |
