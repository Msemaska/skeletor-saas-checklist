# Pricing Model

---

## What It Is

How you charge for your product. This decision affects everything — feature gating, billing integration complexity, marketing messaging, and customer expectations.

## Pricing Models

### Free Trial
- Full access for a limited time (7, 14, or 30 days)
- Requires payment method upfront (higher conversion) or not (higher signups)
- Best when: the product's value is obvious once you use it, but takes time to realize

### Freemium
- Free tier with limited features/usage forever
- Paid tiers unlock more
- Best when: the free tier is genuinely useful and creates word-of-mouth
- Risk: most users never convert (typical conversion 2-5%)

### Flat Rate
- One price, one plan, everything included
- Best when: simplicity is your brand and the product is uniform
- Examples: Basecamp, Hey.com

### Tiered
- 2-4 plans at different price points (e.g., Starter, Pro, Business)
- Each tier unlocks more features, seats, or limits
- Best when: you serve different customer segments with different needs
- Most common SaaS model

### Usage-Based (Credits / Top-Up)
- Pay for what you use via a credit balance that gets consumed per action
- Users purchase credits in advance (top-up) or enable auto-top-up when balance runs low
- Best when: value scales directly with usage and customers want full control over spend
- Risk: lower predictable revenue, requires clear balance visibility so users don't run out mid-task
- Examples: Twilio, AWS

### Hybrid (Subscription + Top-Up)
- Base subscription includes a monthly usage allowance (e.g., "Pro plan: 1,000 credits/month")
- If the allowance runs out before the period ends, the user can purchase additional credits (top-up) to continue using the product immediately
- Auto-top-up option: user pre-authorizes automatic credit purchases when balance drops below a threshold — no interruption to their workflow
- Best when: you want predictable recurring revenue but value also scales with heavy usage. Users get a baseline included in their plan, power users pay more without being forced to a higher tier.
- This is the Anthropic model — subscription gives you a monthly allowance, credits top up when you exceed it
- Examples: Anthropic (Claude), OpenAI (ChatGPT Plus + usage), most AI/API companies

## Process

### Step 1: Choose Your Primary Model

| Factor | Points To |
|---|---|
| Product needs time to show value | Free trial |
| Product has viral/sharing potential | Freemium |
| Product is simple, one-size-fits-all | Flat rate |
| Different customer segments exist | Tiered |
| Value directly correlates to usage | Usage-based (credits) |
| Predictable base + heavy users pay more | Hybrid (subscription + top-up) |

### Step 2: Define Your Tiers (if tiered)

**Typical SaaS tier structure:**

| Tier | Purpose | Price Range |
|---|---|---|
| Free / Hobby | Lead generation, try before buy | $0 |
| Starter / Pro | Individual users, small teams | $9-29/mo |
| Team / Business | Growing teams, more seats/limits | $49-99/mo |
| Enterprise | Custom, sales-led | Custom pricing |

**What to gate per tier:**
- Number of seats / team members
- Usage limits (projects, API calls, storage)
- Features (advanced analytics, integrations, priority support)
- Support level (community → email → priority → dedicated)
- SLA / uptime guarantee

### Step 3: Set the Actual Price

**Research:**
- What do competitors charge?
- What's 10% of the value your product creates? (value-based pricing)
- What does your ICP consider "impulse buy" vs "needs approval"?

**Anchoring rules:**
- Most popular plan should be in the middle
- Highlight the plan you want people to pick ("Most Popular" badge)
- Annual pricing: always 20% discount (equivalent to ~2.4 months free). Do not offer less — it's the standard user expectation and a meaningful incentive. For all other discount types (launch, win-back, enterprise, trial drip), see phase-0-strategy-positioning/07-discounting-strategy.md
- Round numbers feel more approachable ($29 vs $31)

**Psychology:**
- $0 → $1 is the hardest conversion. Free tier to paid is the biggest jump.
- $29/mo feels like a personal decision. $99/mo feels like a business expense.
- $499+/mo typically needs sales involvement.

### Step 4: Define the Billing Details

Decisions you need to make now (they affect Stripe integration):

- Monthly only, or monthly + annual?
- Annual discount percentage?
- Free trial length? (7, 14, 30 days)
- Credit card required for trial?
- What happens when trial ends? (downgrade to free vs lock out)
- What happens on failed payment? (grace period length)
- Proration: payment always happens at time of decision. Monthly→Annual same plan: charge annual now, annual starts after current month ends. Monthly→Annual upgrade: full proration, starts immediately. See phase-8-business-wrapper/02-stripe-architecture.md for all four cases.
- Refund policy? (no refunds, prorated, 30-day money-back)
- Per-seat pricing or per-account?
- Usage overage handling? (hard limit until top-up, auto-top-up, throttle)
- If credits/top-up: credit pack sizes and prices? (e.g., $10 for 500 credits, $20 for 1,200)
- Auto-top-up available? (pre-authorized purchase when balance drops below threshold)
- Do unused credits roll over to next period or expire?
- Minimum top-up amount?

### Step 5: Plan the Pricing Page

Essential elements:
- Plan comparison table with clear feature differentiation
- Toggle for monthly / annual pricing
- "Most Popular" highlight on recommended plan
- FAQ section below plans (billing questions)
- Enterprise "Contact Us" option
- Money-back guarantee if applicable

## Deliverables

- [ ] Pricing model chosen (trial, freemium, tiered, usage, hybrid)
- [ ] Tiers defined with names, prices, and feature gates
- [ ] Annual vs monthly pricing decided
- [ ] Free trial or free tier parameters set
- [ ] Billing edge cases decided (failed payments, proration, refunds)
- [ ] If usage-based or hybrid: credit pack sizes, auto-top-up rules, and rollover policy decided (rollover cap: 2× monthly allowance; rollover lost on cancellation — see phase-3-core-product/09-credit-usage-system.md)
- [ ] Competitor pricing benchmarked
- [ ] Pricing page wireframe/content drafted

## Common Mistakes

- Pricing too low (you can always lower prices, raising them is harder)
- Too many tiers (3-4 max including free)
- Gating the wrong features (don't gate core value, gate scale/convenience)
- Not having annual pricing (you're leaving 15-20% revenue on the table)
- Copying competitor pricing without understanding your own value
- Overthinking it (pick something reasonable, you can change it later — early customers understand)
