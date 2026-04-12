# Stripe Integration Architecture

---

## What It Is

How Stripe connects to your application — webhook handling, subscription lifecycle, data flow between Stripe and your database, and the state machine that governs billing state. This is the design doc. Testing is covered in 03-stripe-sandbox.md, going live is covered in 04-stripe-production.md.

**AI interface:** Stripe CLI (`stripe`), Stripe MCP server.

## Data Ownership

Stripe is the source of truth for billing. Your database stores only references and cached status.

**What lives in Stripe:**
- Customer records, payment methods, card details
- Subscription details, billing cycles, invoices
- Payment history, refunds, disputes
- Tax calculations, receipts
- Pricing and plan configuration (Products and Prices)

**What lives in your database (on the workspace record):**
- `stripe_customer_id` — links workspace to Stripe customer
- `stripe_subscription_id` — links to the current subscription
- `plan` — current plan name (synced from webhooks for quick access)

This is covered in phase-3/01-database-schema.md under Billing Tables. Never duplicate Stripe's data model locally.

## Webhook Architecture

Stripe communicates changes to your app via webhooks. This is the primary integration mechanism — your app doesn't poll Stripe, Stripe pushes events to you.

```
Stripe Event Occurs
  │
  ▼
Stripe sends POST to /api/webhooks/stripe
  │
  ▼
Your API Route
  │ 1. Verify webhook signature
  │ 2. Check event ID against deduplication table
  │ 3. Route to handler based on event type
  │ 4. Handler updates database + triggers side effects
  │ 5. Return 200 to Stripe
  ▼
Done
```

**Design decisions:**
- Webhook endpoint is an API route at `/api/webhooks/stripe`
- Every webhook request is verified using Stripe's webhook signing secret — reject requests with invalid signatures immediately (prevents spoofed webhooks)
- Every event is checked against the webhook deduplication table (covered in phase-3/01-database-schema.md) — if the event ID already exists, skip processing and return 200
- Return 200 to Stripe as quickly as possible — Stripe retries on non-200 responses. Do the heavy processing in the handler, but don't let it take more than 30 seconds.
- For long-running webhook processing, acknowledge with 200 immediately and queue the actual processing as a background job

## Webhook Events to Handle

| Event | What to Do |
|---|---|
| `customer.subscription.created` | Link subscription to workspace, set plan name |
| `customer.subscription.updated` | Update plan name if changed, handle upgrade/downgrade |
| `customer.subscription.deleted` | Mark workspace as unsubscribed, apply free tier limits |
| `invoice.paid` | Reset usage counters for new billing period, send payment receipt email |
| `invoice.payment_failed` | Send payment failed email, trigger dunning flow |
| `customer.subscription.trial_will_end` | Send trial expiring email (3 days before) |
| `checkout.session.completed` | Handle initial subscription creation after Stripe Checkout |

**Design decisions:**
- Handle only the events you need — don't register for every Stripe event type
- Each event has its own handler function in the service layer — no giant switch statement
- Every handler is idempotent — processing the same event twice produces the same result
- Unknown event types are logged and skipped (return 200), not rejected

## Subscription Lifecycle

The state machine for a subscription:

```
Trial → Active → Cancelled
                ↓
           Past Due → Cancelled
```

**States:**
- **Trialing** — free trial period, full access, no payment method required
- **Active** — paying customer, full access
- **Past Due** — payment failed, Stripe is retrying (dunning). User retains access during the retry window.
- **Cancelled** — subscription ended. This is a terminal state. Reactivation creates a new subscription record.

**Design decisions:**
- Trial period: defined on the Stripe Price (trial_period_days). Managed entirely by Stripe, not custom code.
- Free trial does not require a payment method upfront — reduces friction. The trial expiring email (3 days before) prompts them to add one.
- When a subscription is cancelled, the user retains access until the end of the current billing period (covered in phase-3/03-auth.md under Account Deletion — cancel at end of period, not immediately)
- Past Due: Stripe's Smart Retries handle payment retry logic. The user keeps access during the retry window. After all retries fail, the subscription moves to Cancelled.
- Cancelled subscriptions are end-of-lifecycle. A user who wants to resubscribe goes through the subscription flow again, creating a new subscription record.

## Stripe Checkout vs Custom Payment Form

Use Stripe Checkout (redirect to Stripe-hosted page) for the initial subscription. Don't build a custom payment form.

**Design decisions:**
- Stripe Checkout handles PCI compliance, payment method entry, and SCA (Strong Customer Authentication) for European cards
- The flow: user clicks "Subscribe" → your app creates a Checkout Session → user is redirected to Stripe Checkout → user pays → Stripe redirects back to your app with a success URL → your webhook receives `checkout.session.completed` → link the subscription to the workspace
- For plan changes (upgrade/downgrade), use Stripe's subscription update API — no Checkout redirect needed
- For adding/updating payment methods, use Stripe Customer Portal (covered in 04-stripe-production.md)

## Pricing Configuration

Plans and prices are defined in Stripe, not in your code.

**Design decisions:**
- Create Products in Stripe for each plan (e.g., Starter, Pro, Business)
- Create Prices on each Product for monthly and annual billing
- Store Stripe Price IDs in your environment variables — the app references these when creating Checkout Sessions
- When you change pricing, create new Prices in Stripe (don't modify existing ones — existing subscribers stay on their current price)
- Feature gates (which features each plan unlocks) are defined in your application, keyed by the plan name synced from Stripe

## Proration

When a user upgrades or downgrades mid-billing-cycle, the charge is prorated.

**Design decisions:**
- Use Stripe's built-in proration (proration_behavior: create_prorations)
- On upgrade: user is charged the prorated difference immediately for the rest of the current period
- On downgrade: credit is applied to the next invoice
- Show the prorated amount to the user before they confirm the plan change — use Stripe's upcoming invoice preview API

## What to Test

- Webhook signature verification rejects invalid signatures
- Webhook deduplication prevents duplicate processing of the same event
- Each webhook event type triggers the correct handler
- Subscription creation links the subscription to the correct workspace
- Plan change updates the plan name on the workspace
- Cancellation applies free tier limits after the billing period ends
- Invoice paid resets usage counters
- Payment failed sends the correct email
- Trial expiring sends the email 3 days before trial ends
- Checkout flow redirects correctly and handles the success callback
- Proration amount is calculated correctly on plan change
- Unknown event types are logged and skipped (not rejected)

## Edge Cases

- **Webhook arrives before Checkout success redirect** — Stripe sends the webhook immediately, but the user hasn't returned from Checkout yet. The webhook handler links the subscription to the workspace regardless of whether the user has been redirected. When they return, the app sees the subscription is already active.
- **User subscribes, immediately cancels** — the subscription is marked for cancellation at the end of the billing period. The user retains access until then. The workspace's plan doesn't change until the period ends.
- **Two webhooks for the same event arrive simultaneously** — the deduplication table's unique constraint on event ID prevents both from processing. One succeeds, one is skipped.
- **Stripe webhook endpoint is down during an event** — Stripe retries with exponential backoff for up to 3 days. When your endpoint comes back, it processes the queued events. Idempotent handlers ensure no duplicates.
- **User's card expires** — Stripe sends `invoice.payment_failed`. Your dunning flow starts. If the user updates their card before retries exhaust, Stripe retries with the new card automatically.
- **Free trial ends with no payment method** — subscription moves to `cancelled`. The user loses access to paid features and sees an upgrade prompt.
- **Price change for existing subscribers** — creating a new Price in Stripe doesn't affect existing subscribers (they stay on their current Price). Only new subscribers get the new price. To change existing subscribers' prices, use Stripe's subscription update API.
- **Dispute/chargeback** — handle `charge.dispute.created` webhook. Notify the team immediately (this needs manual resolution). Don't auto-refund or auto-cancel — disputes require evidence submission.

## Implementation Checklist

- [ ] Stripe account created
- [ ] Products and Prices configured in Stripe
- [ ] Stripe Price IDs stored in environment variables
- [ ] Webhook endpoint at /api/webhooks/stripe
- [ ] Webhook signing secret stored in environment variables
- [ ] Webhook signature verification on every request
- [ ] Webhook deduplication table active
- [ ] All webhook event handlers implemented (see event table)
- [ ] Every handler is idempotent
- [ ] Checkout Session creation for initial subscription
- [ ] Success and cancel URLs configured for Checkout
- [ ] Subscription update API for plan changes
- [ ] Proration configured (create_prorations)
- [ ] Plan name synced to workspace record on every subscription change
- [ ] Usage counters reset on invoice.paid

## Common Mistakes

- Duplicating Stripe's data model in your database (fighting sync issues forever)
- Not verifying webhook signatures (accepting spoofed webhooks)
- No webhook deduplication (processing the same event twice — duplicate emails, double credits)
- Building a custom payment form instead of using Stripe Checkout
- Modifying existing Prices instead of creating new ones (changes pricing for existing subscribers unexpectedly)
- Handling cancellation as immediate (user already paid for the current period — let them use it)
- Non-idempotent webhook handlers (retried webhooks cause duplicate side effects)
- Not returning 200 quickly enough (Stripe retries, you process the event multiple times)
- Feature gates in Stripe instead of your application (Stripe manages billing, your app manages access)
