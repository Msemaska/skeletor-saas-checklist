# Email Marketing

---

## What It Is

Marketing emails sent to prospects and users to drive sign-ups, engagement, and retention. Different from transactional emails (phase-9/01-transactional-emails.md) which are system-triggered. Marketing emails are intentionally sent to promote, nurture, or re-engage.

## Provider

Use Resend for all email — transactional and marketing. One provider, one sending infrastructure, one set of templates.

## Email Types

### Newsletter Signup

**Design decisions:**
- Newsletter signup form on the marketing site: email field + submit button in the footer and on the blog page
- Double opt-in: user enters email → confirmation email sent → user clicks confirm → subscribed. Required for GDPR compliance and list quality.
- Store subscriber list in your database (subscribers table with email, subscribed_at, unsubscribed_at)
- Subscribers who sign up for the product are automatically added to the newsletter list (with the option to unsubscribe)

### Welcome Drip Sequence

A series of emails sent automatically after someone subscribes to the newsletter or signs up for the product.

**Design decisions:**
- 5 emails over 14 days
- Each email delivers value first, mentions the product second

| Email | Day | Content |
|---|---|---|
| 1 | Day 0 | Welcome + the single most valuable insight about the problem your product solves |
| 2 | Day 2 | A how-to tip related to your category (actionable, no product pitch) |
| 3 | Day 5 | Your product's origin story — why you built it, the problem you experienced |
| 4 | Day 9 | A case study or customer result (phase-14/07-case-studies.md) |
| 5 | Day 14 | Direct CTA — "Ready to try [product]?" with a free trial link |

**Design decisions:**
- Emails 1-4 are value-first with a subtle product mention at the end (not a pitch)
- Email 5 is the only direct sales email
- If the user signs up for the product during the drip, stop the sequence — don't keep sending nurture emails to an active user
- Subject lines are specific, not generic ("The one invoice mistake that costs freelancers $2,400/year" not "Welcome to our newsletter")

### Launch Announcement Email

**Design decisions:**
- Sent to the entire waitlist on launch day
- Subject: "[Product Name] is live — [one-line value proposition]"
- Body: 3 sentences on what launched, a screenshot or GIF, and a CTA button to sign up
- Send at 9 AM in the majority timezone of your waitlist
- One email, not a sequence — don't spam the waitlist

### Weekly Newsletter

**Design decisions:**
- Sent weekly on Tuesday
- Content: latest blog post headline + 2-sentence summary + "Read more" link, 2 changelog highlights, 1 curated industry insight or tip
- Keep it under 300 words
- From address: hello@vibes.yourdomain.com (marketing subdomain — phase-12/01-email-infrastructure.md)
- Unsubscribe link in every email (mandatory)
- Track open rate and click rate — review weekly

## Sending Rules

**Design decisions:**
- Never send more than 2 marketing emails per week to any subscriber (drip + newsletter combined)
- If a drip email and the weekly newsletter fall on the same day, delay the newsletter by 1 day for that subscriber
- Respect unsubscribe immediately — remove from all marketing lists within 24 hours
- Don't send marketing emails to users who have rejected analytics/marketing cookies (phase-9/05-legal-pages.md)
- All marketing emails include: unsubscribe link, company address, link to notification preferences

## What to Test

- Newsletter signup form submits successfully and triggers confirmation email
- Double opt-in flow works (submit → confirm email → subscribed)
- Welcome drip sequence sends 5 emails over 14 days at correct intervals
- Drip sequence stops if user signs up for the product
- Launch announcement email delivers to the full waitlist
- Weekly newsletter renders correctly on desktop, mobile, and dark mode
- Unsubscribe link works and removes from all marketing lists
- No subscriber receives more than 2 marketing emails per week
- Open rate and click rate are tracked

## Edge Cases

- **User signs up during the drip sequence** — stop the drip. They're now an active user, not a prospect. Transactional emails (welcome, onboarding) take over.
- **User unsubscribes then re-subscribes** — allow re-subscription. Reset the drip sequence from email 1.
- **Drip email bounces** — mark the email as undeliverable. Don't retry marketing emails to bounced addresses — it hurts sender reputation.
- **Newsletter content is the same as the latest blog post** — the newsletter is a summary with a link, not a copy of the full post. Keep it to 2 sentences + link.
- **Waitlist has 10 subscribers** — still send the launch announcement. Don't wait for a "big enough" list.

## Implementation Checklist

- [ ] Newsletter signup form on marketing site (footer + blog page)
- [ ] Double opt-in flow implemented (confirmation email)
- [ ] Subscribers table in database (email, subscribed_at, unsubscribed_at)
- [ ] Welcome drip sequence: 5 emails drafted and scheduled (Day 0, 2, 5, 9, 14)
- [ ] Drip sequence stops when user signs up for the product
- [ ] Launch announcement email drafted
- [ ] Weekly newsletter template created
- [ ] Weekly send day chosen (Tuesday)
- [ ] Max 2 marketing emails per week per subscriber enforced
- [ ] Unsubscribe removes from all marketing lists within 24 hours
- [ ] All marketing emails include: unsubscribe link, company address, notification preferences link
- [ ] Open rate and click rate tracking configured

## Common Mistakes

- Single opt-in (no confirmation email — lower list quality, GDPR risk)
- Drip sequence that's all sales pitches (value first, pitch last)
- Continuing the drip after the user signed up for the product
- Newsletter that's a copy of the blog post (send a summary + link, not the full post)
- More than 2 marketing emails per week (subscriber fatigue → unsubscribe or spam report)
- No unsubscribe link (CAN-SPAM violation)
- Generic subject lines ("Newsletter #47" — nobody opens this)
- Not tracking open and click rates (no data on what's working)
