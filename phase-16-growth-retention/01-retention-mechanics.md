# Retention Mechanics

---

## What It Is

Systems that reduce churn and bring back users who have left. Three parts: social proof collection to reinforce trust, win-back emails for churned users, and a cancellation flow that gives users a reason to stay.

## Social Proof Collection

Automated collection and display of user testimonials and reviews using Senja.

### Design Decisions

**Design decisions:**
- Use Senja for testimonial collection, management, and embedding
- Automated review request email sent via Senja 3 months after a user signs up (enough time to have formed an opinion)
- The email asks users to leave a testimonial on Senja's hosted form — one link, no friction
- Senja widget embedded on the marketing site landing page showing approved testimonials in a carousel
- Separately ask power users (users who have been active for 6+ months and use the product at least 4 days per week) to leave reviews on G2 and Capterra — manual email, not automated
- Approve testimonials manually before they appear on the site
- Display testimonials with the user's first name, company name, and avatar (if available)

## Win-Back Emails

A 3-email sequence sent to users who cancel their subscription, delivered via Resend.

### Design Decisions

**Design decisions:**
- Triggered when a subscription is cancelled (status changes to cancelled in the webhook handler)
- 3 emails total, spaced over 4 weeks:

| Email | Timing | Content |
|---|---|---|
| 1 | 1 week after cancellation | Remind them what they're missing — highlight the 2 features they used most (pulled from their usage data) |
| 2 | 2 weeks after cancellation | What's new — list 3 features or improvements shipped since they left |
| 3 | 4 weeks after cancellation | Final offer — 30% discount on their previous plan for 3 months, with a direct resubscribe link |

- If the user resubscribes at any point during the sequence, stop sending the remaining emails
- If the user unsubscribes from marketing emails, stop the win-back sequence immediately
- The discount in email 3 is a Stripe coupon applied automatically when they click the resubscribe link
- Track: open rate per email, click rate, resubscribe rate per email

## Cancellation Flow

A multi-step cancellation process that collects feedback and offers an alternative before confirming.

### Design Decisions

**Design decisions:**
- Cancellation starts from Settings > Billing > Cancel Subscription
- Step 1: Survey — 3 multiple-choice questions max:
  - "Why are you cancelling?" (options: too expensive, missing features, switching to competitor, not using it enough, other)
  - "How satisfied were you overall?" (1-5 scale)
  - "Would you recommend us?" (yes/no)
- Step 2: Downgrade offer — show the next cheaper plan with a comparison of what they keep vs. what they lose. Button: "Downgrade to [plan name] instead." If they're already on the cheapest paid plan, offer a 3-month pause instead of downgrade.
- Step 3: Confirmation — if they decline the downgrade/pause, confirm cancellation with a clear message: "Your subscription will remain active until [billing period end date]. You can resubscribe anytime from Settings > Billing."
- Store survey responses in the database (cancellation_surveys table: user_id, workspace_id, reason, satisfaction_score, would_recommend, cancelled_at)
- Review survey responses monthly to identify patterns

## What to Test

- Senja review request email sends exactly 3 months after signup
- Senja testimonial widget renders on the landing page with approved testimonials only
- Win-back email 1 sends 7 days after cancellation
- Win-back email 2 sends 14 days after cancellation
- Win-back email 3 sends 28 days after cancellation with a working discount link
- Win-back sequence stops when the user resubscribes
- Win-back sequence stops when the user unsubscribes from marketing emails
- Cancellation survey collects all 3 responses and stores them in the database
- Downgrade offer shows the correct cheaper plan with accurate feature comparison
- Users on the cheapest plan see a pause offer instead of a downgrade offer
- Cancellation confirmation shows the correct billing period end date
- Stripe coupon applies correctly when the user clicks the resubscribe link in email 3

## Edge Cases

- **User cancels and resubscribes on the same day** — stop the win-back sequence. Don't send email 1 a week later to an active subscriber.
- **User cancels during their trial** — don't send win-back emails. They never paid, so a discount offer makes no sense. Collect the survey only.
- **User has already received a win-back discount once** — don't offer the 30% discount again on a second cancellation. Send emails 1 and 2 only.
- **Survey response is "missing features" but user doesn't specify which** — the survey is multiple choice, not open-ended. Accept the response as-is. If patterns emerge in the data, follow up manually.
- **User is on the only available plan** — skip the downgrade step entirely. Show the pause offer, then confirmation.

## Implementation Checklist

- [ ] Senja account created and configured
- [ ] Automated review request email scheduled at 3 months post-signup
- [ ] Senja testimonial widget embedded on the landing page
- [ ] Win-back email sequence: 3 emails drafted (1 week, 2 weeks, 4 weeks)
- [ ] Win-back emails triggered by subscription cancellation webhook
- [ ] Win-back sequence stops on resubscribe or marketing email unsubscribe
- [ ] Stripe coupon created for the 30% discount (3 months duration)
- [ ] Resubscribe link in email 3 applies the coupon automatically
- [ ] Cancellation flow: survey step with 3 multiple-choice questions
- [ ] Cancellation flow: downgrade offer step with plan comparison
- [ ] Cancellation flow: pause offer for users on the cheapest plan
- [ ] Cancellation flow: confirmation step with billing period end date
- [ ] cancellation_surveys table created in the database
- [ ] Win-back email metrics tracked (open rate, click rate, resubscribe rate)
- [ ] Monthly review process for survey responses documented

## Common Mistakes

- Sending win-back emails to trial users who never paid (irrelevant offer, damages credibility)
- Sending the discount offer on every cancellation (users learn to cancel-and-return for a permanent discount)
- Making the cancellation flow too long (more than 3 steps feels manipulative — users will email support instead, which costs more)
- Not stopping the win-back sequence when the user resubscribes (they get a "come back" email while actively using the product)
- Skipping the downgrade offer (users who cancel for price reasons would have stayed on a cheaper plan)
- Not storing survey responses (collecting feedback you never read is pointless)
- Testimonial widget showing unapproved or placeholder content on the landing page
