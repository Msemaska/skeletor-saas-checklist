# Referral Program

---

## What It Is

A referral system that rewards existing users for bringing in new paying customers. Tracked and managed through Rewardful, integrated with Stripe for automatic commission payouts.

## Rewardful Integration

### Design Decisions

**Design decisions:**
- Use Rewardful for all referral and affiliate tracking — it integrates directly with Stripe
- Rewardful handles: unique referral link generation, click tracking, conversion attribution, commission calculation, payout management
- Install the Rewardful JavaScript snippet on the marketing site and the signup page so referral cookies are tracked across the funnel
- Rewardful's cookie window is 60 days — if a referred user signs up within 60 days of clicking the referral link, the referrer gets credit

## Referral Reward Structure

### Design Decisions

**Design decisions:**
- Reward type: recurring commission (not one-time)
- Commission: 20% of the referred user's subscription payment for the first 12 months
- After 12 months, the commission stops — the referrer earned their reward, the referred user is retained on their own
- Minimum payout threshold: $50 — commissions accumulate until they hit $50, then Rewardful processes the payout
- Payout method: PayPal (configured by the referrer in their Rewardful dashboard)
- No commission on refunded payments — if a referred user gets a refund, the commission for that payment is reversed

## Referral Page

### Design Decisions

**Design decisions:**
- Located at Settings > Referral in the app
- Shows the user's unique referral link with a copy-to-clipboard button
- Referral link format: yourdomain.com/?via=[referral_code] (Rewardful's default format)
- The page displays 4 stats:

| Stat | Description |
|---|---|
| Referrals sent | Number of unique clicks on the referral link |
| Conversions | Number of referred users who signed up and subscribed to a paid plan |
| Pending commissions | Total commissions earned but not yet paid out |
| Commissions paid | Total commissions paid out to date |

- Stats are pulled from the Rewardful API — don't duplicate this data in your database
- Include a short explanation of the program: "Share your referral link. When someone signs up and subscribes, you earn 20% of their subscription for 12 months."
- Social sharing buttons: copy link, share on X, share via email (pre-filled subject and body)

## What to Test

- Rewardful JavaScript snippet loads on the marketing site and signup page
- Clicking a referral link sets the Rewardful cookie
- A user who signs up within 60 days of clicking a referral link is attributed to the referrer
- Commission is calculated at 20% of the referred user's subscription payment
- Commission stops after 12 months of payments
- Refunded payments reverse the associated commission
- Referral page in settings displays the correct unique link
- Copy-to-clipboard button copies the referral link
- Stats on the referral page match the data in the Rewardful dashboard
- Social sharing buttons generate correct pre-filled content
- Payouts process when the balance exceeds $50

## Edge Cases

- **Referred user upgrades their plan** — commission is 20% of whatever they pay each month. If they upgrade, the commission increases for the remaining months of the 12-month window.
- **Referred user cancels and resubscribes** — the 12-month clock does not reset. If 4 months passed before cancellation, the referrer earns commission on the remaining 8 months after resubscription.
- **User refers themselves (second account)** — Rewardful uses cookies and Stripe customer IDs. If the same person creates a second account using their own referral link, Rewardful will track it. Flag and reject self-referrals manually during payout review.
- **Referral link shared publicly (blog, forum)** — this is fine. Treat it like affiliate marketing. The 60-day cookie window and Stripe attribution handle the rest.
- **Two referrers claim the same user** — first-click attribution. The referrer whose link was clicked first (oldest cookie) gets the credit.

## Implementation Checklist

- [ ] Rewardful account created and connected to Stripe
- [ ] Rewardful JavaScript snippet installed on the marketing site
- [ ] Rewardful JavaScript snippet installed on the signup page
- [ ] Campaign created in Rewardful: 20% recurring commission, 12-month duration
- [ ] Cookie window set to 60 days
- [ ] Minimum payout threshold set to $50
- [ ] Referral page added to Settings > Referral
- [ ] Unique referral link displayed with copy-to-clipboard
- [ ] 4 stats displayed on the referral page (clicks, conversions, pending, paid)
- [ ] Stats pulled from Rewardful API
- [ ] Social sharing buttons: copy link, X, email
- [ ] Commission reversal on refunded payments confirmed working
- [ ] Self-referral detection added to payout review process

## Common Mistakes

- Building a custom referral tracking system instead of using Rewardful (unnecessary complexity for a post-launch feature)
- One-time referral rewards instead of recurring (less motivation to refer — the referrer wants ongoing income)
- No minimum payout threshold (processing $2 payouts is not worth the transaction fees)
- Resetting the 12-month commission window on cancellation/resubscription (referrers exploit the loop)
- Duplicating Rewardful data in your database (single source of truth — query the API)
- Forgetting to install the Rewardful snippet on the signup page (referral cookie is set on the marketing site but lost at signup)
- No self-referral detection (users create a second account to earn commission on themselves)
