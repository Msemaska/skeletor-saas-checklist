# Stripe Sandbox & Testing

---

## What It Is

How to set up and use Stripe's test mode to develop and verify billing flows without processing real payments. Every billing feature is built and tested in test mode before touching live credentials.

## Dual Credentials

Stripe provides separate API keys for test mode and live mode. Both exist on the same Stripe account.

**Design decisions:**
- Store test mode keys in local and staging environment variables, live mode keys in production only
- `STRIPE_SECRET_KEY` and `STRIPE_PUBLISHABLE_KEY` — the same variable names, different values per environment
- `STRIPE_WEBHOOK_SECRET` — separate webhook signing secrets for test and live mode
- Never use live keys in development or staging — test mode keys have no ability to charge real money

## Test Mode Setup

**Design decisions:**
- Create all Products and Prices in test mode first. Stripe maintains separate product catalogues for test and live mode — you'll recreate them in live mode when going to production (covered in 04-stripe-production.md).
- Configure a webhook endpoint in the Stripe test mode dashboard pointing to your staging URL. For local development, use Stripe CLI to forward events.
- Register only the webhook events you handle (listed in 02-stripe-architecture.md)

## Stripe CLI for Local Development

Stripe CLI forwards webhook events from Stripe's test mode to your local development server.

**Design decisions:**
- Use `stripe listen --forward-to localhost:3000/api/webhooks/stripe` during local development
- Stripe CLI provides a temporary webhook signing secret — use this in your local `.env`
- Trigger test events manually with `stripe trigger` commands to simulate specific scenarios without going through the UI
- Stripe CLI logs every event received and every response from your server — use this for debugging webhook handlers

## Test Card Numbers

Stripe provides test card numbers that simulate different scenarios:

| Card Number | Scenario |
|---|---|
| 4242 4242 4242 4242 | Successful payment |
| 4000 0000 0000 0002 | Card declined |
| 4000 0000 0000 3220 | 3D Secure authentication required |
| 4000 0000 0000 9995 | Insufficient funds |
| 4000 0025 0000 3155 | Requires authentication (SCA) |

Use these in your E2E tests (phase-2-testing/04-e2e-testing.md) and during manual testing.

## Testing Workflows

Test every billing flow end-to-end in test mode before going to production.

### Subscription Creation
1. User clicks Subscribe on the pricing page
2. App creates a Checkout Session with a test mode Price
3. User is redirected to Stripe Checkout
4. User enters test card number (4242...)
5. Stripe redirects to success URL
6. Webhook `checkout.session.completed` fires
7. Verify: workspace has stripe_subscription_id, plan is set, user can access paid features

### Upgrade
1. User on Starter plan clicks Upgrade to Pro
2. App updates the subscription via Stripe API
3. Webhook `customer.subscription.updated` fires
4. Verify: workspace plan is updated, proration charge created, new features accessible

### Downgrade
1. User on Pro plan clicks Downgrade to Starter
2. App updates the subscription via Stripe API
3. Verify: downgrade takes effect at end of billing period (not immediately), proration credit applied

### Cancellation
1. User clicks Cancel Subscription
2. App sets `cancel_at_period_end` on the subscription
3. Verify: user retains access until period end, plan changes to free tier after period ends, workspace shows "Cancels on [date]"

### Payment Failure
1. Create a subscription with the 4000 0000 0000 0002 (decline) card
2. Verify: `invoice.payment_failed` webhook fires, payment failed email sent, subscription moves to past_due
3. Simulate retries or update the card to a working one
4. Verify: subscription returns to active after successful retry

### Trial Expiry
1. Create a subscription with a trial period
2. Use Stripe's test clocks to advance time to 3 days before trial end
3. Verify: trial expiring email sent
4. Advance to trial end with no payment method
5. Verify: subscription cancelled, user loses paid access

## Stripe Test Clocks

Test clocks let you simulate the passage of time in test mode — advancing hours, days, or months without waiting. Essential for testing trial expiry, billing cycle resets, and dunning flows.

**Design decisions:**
- Use test clocks for any flow that depends on time passing (trial expiry, invoice generation, dunning retries)
- Create a test clock, attach test customers to it, then advance the clock to trigger time-based events
- Test clocks are only available in Stripe test mode — they don't exist in live mode

## What to Test

### Checkout
- Card 4242 4242 4242 4242: checkout completes, subscription created, workspace plan updated
- Card 4000 0000 0000 0002 (generic decline): checkout shows error, no subscription created
- Card 4000 0000 0000 9995 (insufficient funds): checkout shows appropriate error, no subscription created
- Checkout abandoned (user clicks back): no subscription created, workspace plan unchanged
- Card 4000 0025 0000 3155 (3DS required): 3DS challenge appears and completes successfully
- Card 4000 0000 0000 3220 (3DS fails): authentication fails, no subscription created

### Plan Changes
- Monthly → Annual (same plan): annual price charged immediately, subscription period changes to annual starting at end of current monthly period — user does not lose remaining monthly days
- Monthly (smaller) → Annual (bigger): prorated credit applied for remaining monthly days, annual subscription starts immediately, charge matches upcoming invoice preview
- Annual → Annual upgrade: credit for remaining annual time applied, difference charged immediately, new plan active now
- Annual → Annual downgrade: no immediate charge, credit applied to next annual invoice, access continues until annual period ends
- Annual → Monthly: no immediate charge, monthly takes effect at next annual renewal
- Downgrade (same cadence): plan change takes effect at next billing period, not immediately
- Cancel subscription: access retained until period end, workspace shows cancellation date
- Proration preview displayed to user before confirmation matches the actual charge

### Payment Failure & Dunning
- Card 4000 0000 0000 0341 (attaches but charge fails): invoice.payment_failed webhook fires
- Payment failure: dunning email sent to workspace owner
- Payment failure: subscription status set to past_due
- Payment recovery: subscription returns to active after successful retry charge

### Trial
- Trial created with correct end date
- Trial expiring email sent 3 days before end (verified via test clock)
- Trial ends with payment method: subscription continues, first invoice charged
- Trial ends without payment method: subscription cancelled, paid access revoked

### Webhook Integrity
- Webhook deduplication prevents double-processing when same event fires twice
- All webhook handlers process test events without errors

## Edge Cases

- **Test mode webhook endpoint receives live mode events** — this can happen if the live webhook is misconfigured to point to staging. Webhook signature verification will reject them (different signing secret). But verify your webhook endpoints are pointed at the correct URLs.
- **Test clock advances past multiple billing cycles at once** — Stripe fires all the intermediate events (invoice created, invoice paid, etc.) in sequence. Your handlers must process them in order and be idempotent.
- **Stripe Checkout session expires** — Checkout sessions expire after 24 hours. If the user leaves Checkout without completing, the session expires and no subscription is created. Test this by creating a session and not completing it.
- **Multiple test users share the same email** — Stripe test mode allows this, but your app might not (unique email constraint). Use unique emails for each test scenario.

## Implementation Checklist

- [ ] Test mode API keys stored in local and staging environment variables
- [ ] Products and Prices created in Stripe test mode
- [ ] Stripe CLI installed and configured for local webhook forwarding
- [ ] Local webhook signing secret stored in `.env.local`
- [ ] Staging webhook endpoint configured in Stripe test mode dashboard
- [ ] All billing workflows tested end-to-end in test mode (subscribe, upgrade, downgrade, cancel, fail, trial)
- [ ] Test clocks used for time-dependent flows (trial expiry, billing cycle reset)
- [ ] E2E tests use test card numbers for billing flows
- [ ] Webhook handlers verified against all registered event types

## Common Mistakes

- Using live keys in development (accidentally charging real money during testing)
- Not testing payment failure (only testing the happy path — dunning flow is untested)
- Not testing trial expiry (discovering the trial flow is broken when real users' trials end)
- Manually waiting for time-based events instead of using test clocks (waiting 14 days for a trial to expire)
- Not using Stripe CLI for local webhook testing (testing webhooks only in staging, slow iteration)
- Testing only with the success card number (never testing declined, insufficient funds, or 3D Secure)
- Not testing the cancel flow (user cancels, subscription immediately revokes access instead of at period end)
