# Revenue Metrics

---

## What It Is

The numbers that tell you if your business is healthy — monthly recurring revenue, churn rate, customer lifetime value, and acquisition cost. These are not vanity metrics (page views, total sign-ups). These are the metrics that determine whether the business survives.

## Key Metrics

| Metric | What It Measures | How to Calculate |
|---|---|---|
| MRR (Monthly Recurring Revenue) | Total monthly revenue from active subscriptions | Sum of all active subscription monthly values |
| ARR (Annual Recurring Revenue) | Annualised MRR | MRR × 12 |
| Churn Rate | Percentage of subscribers who cancel per month | Cancelled subscriptions ÷ total active subscriptions at start of month |
| LTV (Customer Lifetime Value) | Average total revenue from a customer | Average revenue per month ÷ monthly churn rate |
| CAC (Customer Acquisition Cost) | Cost to acquire one paying customer | Total marketing/sales spend ÷ new paying customers in the period |
| Trial-to-Paid Conversion | Percentage of free trials that become paid | Paid conversions ÷ total trials started in the same cohort |
| Net Revenue Retention | Revenue from existing customers over time | (MRR from existing customers this month ÷ MRR from those same customers last month) × 100 |

## Where the Data Comes From

**Stripe Dashboard** provides:
- MRR, ARR (calculated automatically)
- Subscription counts, new vs churned
- Revenue by plan
- Payment success/failure rates
- Invoice and refund history

**PostHog** provides:
- Trial-to-paid conversion (funnel: trial started → subscription created)
- Feature usage per plan (which features drive upgrades)
- User engagement before churn (do churned users show warning signs?)
- CAC (if you track marketing source via UTM parameters)

**Design decisions:**
- Stripe Dashboard is the primary revenue dashboard — don't build a custom one. Stripe calculates MRR, ARR, churn, and revenue charts automatically.
- PostHog supplements with product-level insights — funnels, feature usage, user behaviour before churn
- Review metrics weekly during the first 3 months, monthly after that
- Share metrics with the team — revenue transparency keeps everyone aligned on what matters

## Revenue Alerts

Feed critical revenue events into the unified alerting hub (01-unified-alerting.md):

**Design decisions:**
- Alert on: payment failures after all retries (revenue at risk), dispute/chargeback (revenue clawed back), MRR drop of 10%+ month-over-month (churn spike)
- Don't alert on: individual successful payments (too noisy), individual cancellations (normal business). These go to #alerts-info for visibility, not #alerts-critical.

## Churn Analysis

Understanding why users cancel is more valuable than knowing the churn number.

**Design decisions:**
- Cancellation survey (covered in Phase 16 Growth & Retention) captures the reason at the point of cancellation
- PostHog identifies behaviour patterns before churn: users who haven't logged in for 14 days, users whose usage dropped by 50%, users who downgraded
- These patterns inform re-engagement campaigns (win-back emails, covered in Phase 16)

## What to Test

- Stripe Dashboard shows correct MRR and subscription counts
- PostHog trial-to-paid funnel shows correct conversion rates
- Revenue alerts fire when payment fails after all retries
- Revenue alerts fire on disputes/chargebacks
- Cancellation events are tracked in PostHog with correct metadata

## Edge Cases

- **Annual subscriptions skew MRR** — an annual subscription of $1200 is $100 MRR, not $1200. Stripe calculates this correctly. Don't double-count annual payments.
- **Refund affects MRR** — a refund reduces the revenue attributed to that month. Stripe's MRR calculation accounts for this.
- **Free plan users in churn calculation** — free users who stop using the product are not "churn" in the revenue sense. Churn rate is calculated on paid subscribers only.
- **Trial conversions across months** — a user who starts a trial on January 28 and converts on February 2 appears in February's conversion count. Track conversion by trial start cohort, not by conversion date, for accurate trial-to-paid rates.
- **Multiple workspaces per user** — if one user has 3 paid workspaces, that's 3 subscriptions contributing to MRR. Revenue metrics are per-subscription, not per-user.

## Implementation Checklist

- [ ] Stripe Dashboard accessible and showing MRR, ARR, churn
- [ ] PostHog trial-to-paid funnel configured
- [ ] PostHog feature usage tracking per plan
- [ ] Revenue alerts integrated into unified alerting hub (payment failures, disputes)
- [ ] Individual cancellations and subscriptions posted to #alerts-info
- [ ] Weekly metric review scheduled for first 3 months
- [ ] Team has access to Stripe Dashboard and PostHog

## Common Mistakes

- Building a custom revenue dashboard instead of using Stripe's (Stripe already calculates everything correctly)
- Tracking total sign-ups as a success metric (sign-ups without activation are meaningless)
- Not tracking trial-to-paid conversion (you don't know if your trial is working)
- Including free users in churn calculations (inflates churn rate, misleading)
- Not reviewing metrics regularly (data exists but nobody looks at it — problems go undetected for months)
- Alerting on every cancellation (normal business, creates alert fatigue — track the trend, not individual events)
- No churn analysis (you know users are leaving but not why)
