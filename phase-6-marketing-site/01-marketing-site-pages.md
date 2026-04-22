# Marketing Site Pages

---

## What It Is

The pages that make up your public-facing marketing site. Each page has a specific purpose in the user journey from discovery to sign-up. The layout and navigation for the marketing site are covered in phase-5-ui-ux/01-marketing-site-layout.md — this file covers what goes on each page.

## Pages to Build Before Launch

### Landing Page (/)

The most important page. A visitor decides in 5 seconds whether to stay or leave.

**Design decisions:**
- Hero section: one-liner headline (from phase-0-strategy-positioning/01-value-proposition.md), subheadline expanding on the mechanism, primary CTA button ("Get Started" or "Start Free Trial")
- Social proof section: customer logos, testimonials, or user count (if available — skip if you have none, don't fake it)
- Feature highlights: 3-4 key features with brief descriptions and visuals
- How it works: 3-step visual explanation of the user journey
- Pricing preview: link to the pricing page or an embedded summary
- Final CTA: repeat the primary CTA at the bottom of the page
- Built with MagicUI for animated, polished components

### Features Page (/features)

Detailed breakdown of what the product does.

**Design decisions:**
- One section per major feature with: heading, description, and a screenshot or visual
- Organize by use case or by user benefit, not by technical capability
- Each feature section has a CTA linking to sign-up
- Include comparison with manual alternatives ("Without [product]: 3 hours. With [product]: 3 minutes.")

### Pricing Page (/pricing)

Where the visitor decides whether to pay.

**Design decisions:**
- Plan comparison table with clear feature differentiation
- Monthly/annual toggle (annual shows the discount prominently)
- "Most Popular" badge on the recommended plan
- Each plan has a CTA button: "Start Free Trial" for paid plans, "Get Started" for free tier
- FAQ section below the plans answering common billing questions
- Enterprise "Contact Us" option (if applicable)
- Built with Shadcn components for the comparison table and toggles

### About Page (/about)

Who you are and why this product exists.

**Design decisions:**
- Brief founder/team story — why you built this, what problem you experienced
- Mission statement (one sentence)
- Team photos and bios (if comfortable sharing — not required for a solo founder)
- Link to careers page (if hiring) or "We're small and focused" if not

### Branding Page (/branding)

A living brand guide — logo downloads, colour swatches, typography, design system tokens.

**Design decisions:**
- Logo download section: all variants (logomark, logotype, lockup) in SVG and PNG
- Colour palette with hex/OKLCH values and copy-to-clipboard
- Typography with font names and usage guidelines
- Favicon and app icon downloads
- Usage guidelines (clear space, minimum size, what not to do)
- This page is useful for press, partners, and your own reference across projects

### Contact Page (/contact)

How to reach you.

**Design decisions:**
- Simple form: name, email, message. Submitted via Server Action to send an email to hello@
- Display the hello@ email address alongside the form (some users prefer to email directly)
- Expected response time: "We'll get back to you within 24 hours"
- No phone number (MVP SaaS doesn't need phone support)

### Blog Index (/blog)

List of all published posts. Entry point for organic search traffic and content discovery.

**Design decisions:**
- Grid of `PostCard` components sorted by `publishedAt` descending
- Featured post pinned at the top (the highest-quality or most recent post marked `featured: true`)
- Tag filter bar to browse posts by topic
- Search trigger (opens Pagefind modal)
- No pagination on the index — show all published posts. For blogs under 100 posts, infinite scroll or pagination adds complexity with no benefit
- Blog SEO, scheduling, search, and schema decisions are covered in `phase-6-marketing-site/03-blog-system.md`

### Waitlist / Early Access Page (/waitlist)

Pre-launch page to collect interested users before the product is ready.

**Design decisions:**
- Headline explaining what the product is and that it's coming soon
- Email collection form (name + email, stored in the database)
- Social proof: number of people on the waitlist (if above 100 — don't show "2 people waiting")
- Transition to the landing page once launched — redirect /waitlist to / with a "We're live!" banner

## Pages to Build After Launch

### Comparison Pages (/compare/competitor-name)

"Your Product vs Competitor X" — high-intent SEO pages.

**Design decisions:**
- One page per major competitor
- Feature comparison table (your product vs theirs)
- Honest — don't misrepresent competitor features. Highlight where you're better, acknowledge where they're stronger, explain why the tradeoff favours you.
- CTA: "Switch from [Competitor] to [Your Product]"

### Use Case Pages (/use-cases/use-case-name)

Pages targeting specific audiences or workflows.

**Design decisions:**
- One page per primary use case or audience segment
- Headline addresses the specific audience: "Project Management for Marketing Teams"
- Features highlighted are the ones most relevant to that use case
- Testimonials/case studies from that audience (when available)

### Demo / Screenshots Page (/demo)

Visual tour of the product for users who want to see it before signing up.

**Design decisions:**
- Annotated screenshots of key screens
- Product demo video (embedded from YouTube)
- Interactive demo (if feasible — a sandbox environment with sample data)
- CTA: "Try it yourself — Start Free Trial"

## What to Test

- Every page loads correctly on desktop, tablet, and mobile
- All CTAs link to the correct sign-up/pricing/contact page
- Contact form submits successfully and delivers email to hello@
- Waitlist form stores submissions in the database
- Monthly/annual pricing toggle shows correct prices
- All images have alt text
- Page load time under 3 seconds on mobile (Lighthouse)
- No broken links across all pages

## Edge Cases

- **No social proof yet** — don't fake testimonials or logos. Skip the social proof section until you have real ones. A landing page without testimonials is fine — a landing page with fake ones destroys trust.
- **Pricing changes after launch** — update the pricing page, but don't change existing subscribers' prices (Stripe handles this — covered in phase-9-business-wrapper/02-stripe-architecture.md).
- **Waitlist page still accessible after launch** — redirect /waitlist to / and show a banner: "We're live! Sign up now."
- **Contact form spam** — rate limit the contact form endpoint (5 submissions per IP per hour). Don't add CAPTCHA — it hurts conversion more than spam costs.

## Implementation Checklist

- [ ] Landing page with hero, features, how it works, pricing preview, CTAs
- [ ] Features page with detailed breakdown per feature
- [ ] Pricing page with plan comparison, monthly/annual toggle, FAQ
- [ ] About page with founder story and mission
- [ ] Branding page with logo downloads, colours, typography
- [ ] Contact page with form (submits to hello@)
- [ ] Blog index page with featured post, tag filter, and search trigger
- [ ] Waitlist page (pre-launch) with email collection
- [ ] All pages use marketing site layout (phase-5-ui-ux/01-marketing-site-layout.md)
- [ ] All pages responsive across desktop, tablet, mobile
- [ ] All CTAs link to correct destinations
- [ ] All images have alt text
- [ ] Page load under 3 seconds on mobile

## Common Mistakes

- Landing page that explains features before explaining the benefit (lead with the outcome, not the mechanism)
- Pricing page without a FAQ section (users have billing questions that prevent conversion)
- Fake testimonials or inflated user counts (destroys trust when discovered)
- Contact form with no expected response time (user submits and wonders if anyone will ever reply)
- No waitlist-to-launch redirect (users land on a "coming soon" page after the product is live)
- Comparison pages that misrepresent competitors (readers check — dishonesty backfires)
- Branding page with outdated assets (update when the brand evolves)
