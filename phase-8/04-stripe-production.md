# Stripe Production Launch

---

## What It Is

The checklist for going from Stripe test mode to live mode — switching credentials, recreating products, configuring production-specific features, and verifying everything works with real money.

## Go-Live Checklist

Complete these in order. Each step depends on the previous one.

### 1. Recreate Products and Prices in Live Mode

Stripe's test mode and live mode have separate product catalogues. Products and Prices created in test mode don't exist in live mode.

**Design decisions:**
- Recreate every Product and Price in Stripe's live mode dashboard, matching the test mode configuration exactly
- Update the Price IDs in your production environment variables — live mode Price IDs are different from test mode
- Verify the pricing, currency, billing intervals, and trial periods match what you tested

### 2. Switch API Keys

- Replace `STRIPE_SECRET_KEY` and `STRIPE_PUBLISHABLE_KEY` in your production environment with live mode keys
- Generate a new webhook signing secret for the live mode webhook endpoint and update `STRIPE_WEBHOOK_SECRET`
- Live keys start with `sk_live_` and `pk_live_` — verify the prefix before deploying

### 3. Configure Live Webhook Endpoint

- Add a new webhook endpoint in Stripe's live mode dashboard pointing to your production URL (`yourdomain.com/api/webhooks/stripe`)
- Register the same event types you handle in test mode
- Verify the webhook signing secret is updated in your production environment variables
- Test the endpoint by triggering a test event from the Stripe dashboard — verify your server receives and processes it

### 4. Enable Stripe Tax

Stripe Tax calculates and collects the correct tax amount based on the customer's location.

**Design decisions:**
- Enable Stripe Tax on your Stripe account
- Configure tax settings: your business address (determines nexus), tax registration numbers for jurisdictions where you're registered
- Tax is calculated automatically on every invoice — no custom tax logic in your application
- Tax amounts are included in the invoice and payment receipt emails

### 5. Configure Dunning (Failed Payment Recovery)

Stripe's Smart Retries automatically retry failed payments on an optimised schedule.

**Design decisions:**
- Enable Smart Retries in Stripe's billing settings — Stripe determines the best retry schedule based on the failure reason
- Configure the dunning email sequence: Stripe can send payment failure emails, but use your own emails (via your transactional email system — 01-transactional-emails.md) for consistent branding
- Set the maximum retry period: 4 weeks. After all retries fail, the subscription is cancelled.
- During the retry period, the user retains access (subscription is `past_due`, not `cancelled`)

### 6. Configure Stripe Customer Portal

Stripe Customer Portal provides a hosted page where customers can manage their subscription, update payment methods, view invoices, and cancel.

**Design decisions:**
- Enable the Customer Portal in Stripe's settings
- Customise the portal: add your logo and brand colours
- Configure what customers can do in the portal: update payment method, view invoices, cancel subscription, switch plans
- Link to the portal from your app's billing settings page — create a portal session via Stripe API and redirect the user
- The portal handles PCI compliance for payment method updates — you don't need to build a payment form

### 7. Configure Invoice and Receipt Branding

- Add your logo and brand colours to Stripe's invoice settings
- Set your company name, address, and tax ID on invoices
- Disable Stripe's automatic receipt emails — your transactional email system sends receipts instead

**Design decision:** Disable Stripe's automatic receipt emails and send your own instead. This ensures consistent branding and lets you include additional context (link to dashboard, usage summary, etc.).

### 8. Verify Refund Flow

- Process a test refund on a real $1 payment to verify the refund flow works
- Verify the refund shows in the customer's invoice history
- Verify the workspace's plan is not affected by a partial refund (only a full cancellation changes the plan)

## Post-Launch Monitoring

**Design decisions:**
- Monitor Stripe webhook delivery in the Stripe dashboard — check for failed deliveries daily during the first week
- Monitor the revenue dashboard (Phase 9 Observability) for: MRR, new subscriptions, churn, failed payments
- Set up Stripe's built-in email alerts for: successful payments, failed payments, disputes/chargebacks
- Review Stripe's radar (fraud detection) weekly during the first month

## What to Test

- Live webhook endpoint receives and processes events correctly (test with Stripe dashboard's "Send test webhook" feature)
- Checkout Session with live Price ID creates a real subscription
- Payment with a real card succeeds and triggers the correct email
- Refund processes correctly and appears in invoice history
- Customer Portal opens and allows payment method update
- Invoice displays correct branding (logo, company info, tax)
- Dunning emails fire when payment fails (test with a card that will be declined)

## Edge Cases

- **First real payment fails** — if the very first live payment fails, verify: live API keys are correctly set, the Price ID in environment variables is the live mode version (not test mode), and the webhook endpoint is registered in live mode.
- **Test mode Price ID used in production** — the Checkout Session creation will fail because the Price doesn't exist in live mode. Verify all Price IDs in production environment variables start with `price_` and are from the live mode catalogue.
- **Webhook endpoint receives test mode events in production** — webhook signature verification will reject them (different signing secrets). But log these rejections for debugging.
- **Customer disputes a charge** — Stripe sends `charge.dispute.created`. This requires manual action: submit evidence to Stripe within the dispute deadline. The team must be notified immediately.
- **Tax configuration changes after launch** — adding a new tax jurisdiction requires updating Stripe Tax settings. Existing invoices are not retroactively updated — the change applies to future invoices only.

## Implementation Checklist

- [ ] Products and Prices recreated in Stripe live mode
- [ ] Live mode Price IDs updated in production environment variables
- [ ] Live API keys set in production environment variables (sk_live_, pk_live_)
- [ ] Live webhook endpoint registered with correct event types
- [ ] Live webhook signing secret set in production environment variables
- [ ] Stripe Tax enabled and configured (business address, tax registrations)
- [ ] Smart Retries enabled with 4-week retry period
- [ ] Stripe receipt emails disabled (using custom transactional emails instead)
- [ ] Customer Portal enabled and branded (logo, colours)
- [ ] Customer Portal linked from app billing settings
- [ ] Invoice branding configured (logo, company info, tax ID)
- [ ] Test payment processed with real card ($1, then refunded)
- [ ] Refund flow verified
- [ ] Stripe dashboard alerts configured (payments, failures, disputes)

## Common Mistakes

- Using test mode Price IDs in production environment variables (Checkout fails — Price doesn't exist in live mode)
- Forgetting to register the live webhook endpoint (webhooks work in test, fail silently in production)
- Keeping Stripe's automatic receipt emails enabled alongside your custom emails (customers get duplicate receipts)
- Not testing with a real payment before public launch (first customer hits an error)
- No monitoring of webhook deliveries during the first week (failed webhooks go unnoticed, subscriptions don't activate)
- Not configuring dunning (failed payments go straight to cancellation instead of retrying)
- No dispute/chargeback notification (dispute deadline passes, Stripe auto-resolves in customer's favour)
