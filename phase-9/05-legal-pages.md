# Legal Pages

---

## What It Is

The legal documents your SaaS needs before launch — privacy policy, terms of service, cookie consent, and related pages. These protect you legally and are required for compliance (GDPR, CAN-SPAM, ePrivacy).

## Pages Required

| Page | Required By | Priority |
|---|---|---|
| Privacy Policy | GDPR, most app stores, Google OAuth | `[2]` |
| Terms of Service | Standard business practice | `[2]` |
| Cookie Consent Banner | ePrivacy Directive (EU), GDPR | `[2]` |
| Acceptable Use Policy | Standard for SaaS (limits misuse) | `[2]` |
| Refund / Cancellation Policy | Consumer protection laws, Stripe requirement | `[2]` |
| DPA (Data Processing Agreement) | GDPR for B2B (if processing customer data) | `[3]` |

## How to Generate

Use Claude to draft all legal pages. These are not boilerplate copy-pastes — they need to be tailored to your specific product, what data you collect, how you process it, and which services you use.

**Design decisions:**
- Draft each page with Claude, providing: your product name, what the product does, what personal data is collected, which third-party services process data (Supabase, Stripe, Resend, PostHog, Sentry, Novu), your business entity and jurisdiction, and your contact information
- Have a lawyer review the drafts before launch if handling sensitive data (health, finance, children's data). For a standard SaaS, Claude-generated drafts are a reasonable starting point.
- Update legal pages when you add new third-party services, change data collection practices, or expand to new jurisdictions
- Display a "Last updated" date on every legal page — this is required by GDPR

## Privacy Policy

What to include:
- What personal data you collect (name, email, usage data, payment info)
- Why you collect it (providing the service, billing, analytics, support)
- How long you retain it
- Who you share it with (list every third-party service by name: Supabase for database, Stripe for payments, Resend for emails, PostHog for analytics, Sentry for error tracking)
- User rights: access, correction, deletion, data portability, objection to processing
- How users exercise their rights (self-service in app settings + support contact)
- Cookie policy (linked from the privacy page to `/cookies`)
- Contact information for privacy inquiries

## Terms of Service

What to include:
- Description of the service and eligibility (age requirements, business use)
- Account responsibilities (keeping credentials secure, accurate information)
- Acceptable use (what users can and cannot do — reference the AUP)
- Intellectual property (who owns user content, who owns the product)
- Payment terms (billing cycle, refund policy — reference the refund page)
- Termination (how either party can terminate, what happens to data)
- Limitation of liability
- Dispute resolution
- Changes to terms (how users are notified)

## Cookie Consent Banner

**Design decisions:**
- Show a non-intrusive banner on first visit with: brief message, link to full cookie policy, Accept and Reject buttons
- Accept enables analytics and marketing cookies. Reject disables them. Essential cookies (auth, session) are always active regardless of consent.
- Store the consent preference in a cookie — don't re-show the banner on every page load
- If the user rejects, do not load PostHog, analytics scripts, or any non-essential tracking
- Consent must be freely given — no dark patterns (the Reject button must be as prominent as the Accept button)

## Acceptable Use Policy

What to include:
- Prohibited content (illegal content, spam, malware, harassment)
- Prohibited activities (scraping, reverse engineering, automated abuse, exceeding rate limits)
- Consequences of violation (warning, suspension, termination)
- Reporting mechanism (how users report violations)

## Refund / Cancellation Policy

**Design decisions:**
- Cancellation is self-service (button in billing settings, not "email us to cancel")
- Cancellation takes effect at the end of the current billing period (user already paid for that time)
- Refunds: offer a 14-day money-back guarantee for new subscribers (first payment only). Beyond 14 days, no refunds — the user can cancel to stop future charges.
- Prorated refunds for annual plans cancelled within 14 days
- Refunds are processed back to the original payment method via Stripe

## DPA (Data Processing Agreement)

Only needed for B2B products where you process data on behalf of your customers (you're a data processor, they're the data controller under GDPR).

**Design decisions:**
- Draft a standard DPA template with Claude
- Make it available as a downloadable PDF linked from the privacy page
- Include: scope of processing, data categories, security measures, sub-processor list, breach notification obligations, data deletion upon termination
- B2B customers may request modifications — be prepared to negotiate

## Where Legal Pages Live

**Design decisions:**
- All legal pages are static pages in the marketing site layout (top nav + footer, no sidebar)
- Routes: `/privacy`, `/terms`, `/cookies`, `/acceptable-use`, `/refund-policy`
- DPA is a downloadable PDF linked from the privacy page
- The cookie consent banner is a global component that appears on every page (marketing site and app) until the user makes a choice
- Link to all legal pages from the marketing site footer

## What to Test

- All legal pages are accessible without authentication
- Cookie consent banner appears on first visit
- Accepting cookies enables analytics tracking (PostHog loads)
- Rejecting cookies disables analytics tracking (PostHog does not load)
- Cookie preference persists across page loads (banner doesn't re-appear)
- All legal pages show "Last updated" date
- Links to legal pages work from the marketing site footer
- DPA PDF download works (if applicable)

## Edge Cases

- **User clears cookies and revisits** — cookie consent preference is lost, banner re-appears. This is correct behaviour.
- **Legal page needs updating after adding a new service** — when you integrate a new third-party service (new analytics, new email provider), update the privacy policy to list them and update the "Last updated" date.
- **User in a jurisdiction with specific requirements** — GDPR (EU), CCPA (California), PIPEDA (Canada) have different requirements. The privacy policy should cover the most restrictive requirements (GDPR) — this satisfies all others.
- **Cookie consent on the app (not just marketing site)** — the cookie banner should appear in the app too if you're running analytics there. Auth cookies don't require consent, but analytics cookies do.
- **User requests data deletion** — the process is covered in phase-3/03-auth.md under Account Deletion. The privacy policy must describe this right and how to exercise it.

## Implementation Checklist

- [ ] Privacy policy drafted (tailored to your product and third-party services)
- [ ] Terms of service drafted
- [ ] Cookie consent banner implemented (Accept/Reject, stores preference, blocks non-essential cookies on reject)
- [ ] Acceptable use policy drafted
- [ ] Refund/cancellation policy drafted (14-day money-back, cancel at end of period)
- [ ] DPA template drafted (if B2B)
- [ ] All legal pages live at correct routes (/privacy, /terms, etc.)
- [ ] "Last updated" date displayed on every legal page
- [ ] All legal pages linked from the marketing site footer
- [ ] Cookie consent appears on both marketing site and app
- [ ] Analytics blocked when cookies are rejected
- [ ] Lawyer review scheduled (if handling sensitive data)

## Common Mistakes

- Copy-pasting legal templates from another product (doesn't reflect your specific data practices)
- No "Last updated" date (GDPR requires it)
- Cookie consent with only an Accept button (dark pattern — violates GDPR)
- Loading analytics before cookie consent is given (GDPR violation)
- "Email us to cancel" instead of self-service cancellation (user-hostile, may violate consumer protection laws)
- Not updating legal pages when adding new third-party services (privacy policy becomes inaccurate)
- No refund policy (Stripe and many jurisdictions require one to be stated)
- Legal pages behind authentication (they must be publicly accessible)
