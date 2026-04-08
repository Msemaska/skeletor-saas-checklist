# Partnerships

---

## What It Is

A partnership program that lets other companies integrate with your product, co-market, and build on your API. Includes a public-facing partner page, an application process, onboarding documentation, and tracking of partner activity.

## Design Decisions

**Partner page:** A marketing page at `/partners` on the main site. Content includes: what the integration partnership looks like, benefits for partners (API access, co-marketing, listing in your integrations directory), example use cases, and an application form at the bottom. The page uses the same design system and components as the rest of the marketing site.

**Application form:** A React Hook Form with Zod validation. Fields: company name, website, contact email, integration description (what they want to build), estimated user overlap. Submissions go to a `partner_applications` table in Supabase with status `pending`. The team reviews applications in an internal admin view.

**Application workflow:** Submitted → Reviewed → Approved or Rejected. On approval, the partner receives an email with API credentials (a dedicated API key created in Unkey with a `partner_` prefix) and a link to the partner onboarding docs. On rejection, the partner receives an email with a brief reason.

**Partner onboarding docs:** A section within the Fumadocs site (from phase 18) at `/docs/partners/`. Content includes: API access setup, integration guide with step-by-step instructions for common integration patterns, co-branding guidelines (logo usage, naming conventions, approved copy), and a partner support contact.

**Co-marketing:** Joint blog posts published on your blog with a partner tag. Shared social media promotion — you post about the integration, the partner posts about the integration. Inclusion in each other's integration directories or marketplace listings. All co-marketing activities are agreed upon during the onboarding call, not ad-hoc.

**Tracking:** Track active partners and their API usage in PostHog. Create a PostHog dashboard showing: number of active partners, API calls per partner per month, integration adoption (how many of the partner's users connect to your product). Use the `partner_` API key prefix to filter partner traffic from regular API traffic in analytics.

**Partner tiers:** Keep it simple — one tier. Every approved partner gets the same API access, the same co-marketing opportunity, and the same onboarding support. Don't create bronze/silver/gold tiers until there are more than 20 active partners and clear differentiating criteria.

## What to Test

- Partner page renders correctly with all sections
- Application form validates required fields
- Form submission creates a record in `partner_applications` with status `pending`
- Approval flow sends email with API credentials and docs link
- Rejection flow sends email with reason
- Partner API key has the `partner_` prefix
- Partner docs section is accessible at `/docs/partners/`
- PostHog dashboard shows partner API usage filtered by key prefix
- Application form handles duplicate submissions (same email) gracefully

## Edge Cases

- **Partner applies multiple times** — check for existing applications with the same email. If a pending or approved application exists, show a message that they've already applied. If a rejected application exists, allow reapplication.
- **Partner's API key is compromised** — revoke the key in Unkey immediately. Issue a new key and notify the partner. The `partner_` prefix makes it easy to identify in logs.
- **Partner stops responding** — if a partner has not made any API calls in 90 days, flag them as inactive. Send a check-in email. If no response after 30 more days, revoke their API key and mark the partnership as dormant.
- **Partner violates co-branding guidelines** — have a clause in the partner agreement that allows you to request corrections. If they use your logo incorrectly or make misleading claims, email them with specific corrections needed.
- **Too many applications to review** — if the backlog exceeds 20 pending applications, the review process is too slow. Set a target of 5 business days from submission to decision.

## Implementation Checklist

- [ ] `/partners` marketing page built with application form
- [ ] `partner_applications` table created in Supabase (company_name, website, contact_email, description, status, created_at)
- [ ] Application form submits to a Server Action that inserts into the table
- [ ] Internal admin view to list, review, approve, and reject applications
- [ ] Approval email template with API credentials and docs link
- [ ] Rejection email template with reason field
- [ ] Partner API keys created in Unkey with `partner_` prefix on approval
- [ ] Partner onboarding docs section added to Fumadocs at `/docs/partners/`
- [ ] Co-branding guidelines documented (logo usage, naming, approved copy)
- [ ] PostHog dashboard created for partner API usage tracking
- [ ] Partner activity monitoring: flag inactive partners after 90 days of no API calls
- [ ] Partner agreement template covering API usage terms and co-branding rules

## Common Mistakes

- Building a complex multi-tier partner program before having any partners (over-engineering for zero demand)
- No application review process (partners self-approve and get API access without vetting)
- Partner docs are a separate system from the main API docs (two places to maintain, they drift apart)
- No tracking of partner API usage (no idea which partners are active and which have abandoned the integration)
- Co-marketing promises with no follow-through (partner publishes their side, you never promote the integration)
- No co-branding guidelines (partners use your logo wrong, make unauthorized claims, or misrepresent the integration)
- Manual API key creation outside of Unkey (no revocation, no usage tracking, no audit trail)
- Not setting a review SLA for applications (partners wait weeks and lose interest)
