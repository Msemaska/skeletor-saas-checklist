# Email Infrastructure & DNS

---

## What It Is

The setup that allows your product to send and receive email from your own domain — transactional emails, marketing emails, and inbound email (support, contact). This covers the mailbox, aliases, routing, sending subdomains, and DNS authentication records.

## Architecture

**Sending:**

| Email Type | Subdomain | Provider | Destination |
|---|---|---|---|
| Transactional | mail.yourdomain.com | Resend | User inbox |
| Marketing | vibes.yourdomain.com | Resend | User inbox |

**Receiving:**

hello@yourdomain.com routes through Cloudflare Email Routing to the single real mailbox. Users can reply to any email.

## One Real Mailbox

For an MVP, maintain a single email inbox that receives all inbound email. Don't set up separate mailboxes for support, billing, and sales — one person reads them all anyway.

**Design decisions:**
- One Google Workspace mailbox
- hello@ routes to this one mailbox — users can reply to any email and it reaches the inbox
- Cloudflare Email Routing handles the forwarding — it's free and configured in the Cloudflare DNS dashboard
- When you outgrow one mailbox (dedicated support team, ticketing system), add more mailboxes then. Not now.

## Email Address

One address for everything: `hello@yourdomain.com`

**Design decisions:**
- `hello@` is the From address on all emails — transactional and marketing. No noreply@.
- `hello@` is the contact address on the marketing site, in-app support link, and invoice sender
- Users can reply to any email (password reset, invoice, welcome) and it reaches the real mailbox
- Configured in Cloudflare Email Routing, forwarding to the single real mailbox
- Add more specific aliases (support@, billing@) later when you have a team that needs to route emails to different people

## Two Sending Subdomains

Separate transactional and marketing email onto different subdomains. This protects transactional deliverability — if marketing emails get spam complaints, it doesn't affect password reset emails.

| Subdomain | Purpose | Provider |
|---|---|---|
| mail.yourdomain.com | Transactional emails (password reset, verification, invoices) | Resend |
| vibes.yourdomain.com | Marketing emails (newsletters, drip sequences, announcements) | Resend |

**Design decisions:**
- Both subdomains use Resend as the sending provider — one account, two domains configured
- Transactional emails send from `hello@mail.yourdomain.com`
- Marketing emails send from `hello@vibes.yourdomain.com`
- Each subdomain has its own SPF, DKIM, and DMARC records — they build independent sender reputations

## DNS Records

Each sending subdomain needs three DNS records for email authentication:

### SPF (Sender Policy Framework)

- One TXT record per sending subdomain
- The record specifies Resend's sending servers as authorised senders
- Resend provides the exact SPF record value during domain setup

### DKIM (DomainKeys Identified Mail)

- CNAME record(s) per sending subdomain
- Resend provides the DKIM record values during domain setup
- DKIM keys are managed by Resend — no manual key rotation needed

### DMARC (Domain-based Message Authentication, Reporting & Conformance)

- One TXT record on `_dmarc.yourdomain.com`
- Start with `p=none` (monitor mode) — this lets you receive reports without rejecting any email
- After 4 weeks of clean reports, upgrade to `p=quarantine` (suspicious emails go to spam)
- After 8 weeks of clean reports on `p=quarantine`, move to `p=reject` (unauthenticated emails are rejected entirely)

**Design decisions:**
- Configure DMARC on the root domain (applies to all subdomains)
- Start with `p=none` to monitor before enforcing
- DMARC reports go to your real mailbox

## Root Domain Protection

Even though you send from subdomains, protect the root domain too:

- Add an SPF record on the root domain that doesn't authorise any senders — this prevents spammers from spoofing your root domain
- DMARC on the root domain (configured above) applies to the root and all subdomains

## Setup Order

1. Configure Cloudflare Email Routing for inbound aliases
2. Verify the real mailbox receives forwarded emails
3. Add mail.yourdomain.com as a domain in Resend
4. Add vibes.yourdomain.com as a domain in Resend
5. Add the SPF, DKIM records for each subdomain in Cloudflare DNS
6. Add the DMARC record on the root domain
7. Verify all records in Resend's domain verification tool
8. Send a test email from each subdomain and verify delivery

## What to Test

- Emails sent from mail.yourdomain.com arrive in the recipient's inbox (not spam)
- Emails sent from vibes.yourdomain.com arrive in the recipient's inbox (not spam)
- Replies to transactional emails reach the real mailbox
- hello@ forwards to the real mailbox
- SPF, DKIM, and DMARC records pass verification in Resend's dashboard
- DMARC is set to `p=none` initially (monitoring mode)

## Edge Cases

- **Emails landing in spam** — check SPF, DKIM, and DMARC records are correctly configured. Use Mail Tester (covered in 02-email-deliverability.md) to diagnose. The most common cause is a missing or incorrect DKIM record.
- **Cloudflare Email Routing not forwarding** — verify the routing rules are active in the Cloudflare dashboard. Verify the destination email address has been confirmed (Cloudflare sends a verification email to the destination).
- **User replies to a transactional email** — since all emails come from hello@, replies go directly to the real mailbox. No reply-to header gymnastics needed.
- **DMARC set to reject too early** — if you set `p=reject` before verifying all senders are properly authenticated, legitimate emails get rejected. Start with `p=none`, review reports, then tighten.
- **New service needs to send email** — if you add a new service that sends email from your domain (e.g., a new transactional email provider), you need to update the SPF record to include their servers. Forgetting this causes SPF failures and deliverability issues.

## Implementation Checklist

- [ ] One real mailbox set up (Google Workspace)
- [ ] Cloudflare Email Routing configured for hello@
- [ ] All aliases verified (test emails forwarded to real mailbox)
- [ ] mail.yourdomain.com added as domain in Resend (transactional)
- [ ] vibes.yourdomain.com added as domain in Resend (marketing)
- [ ] SPF records added for both subdomains in Cloudflare DNS
- [ ] DKIM records added for both subdomains in Cloudflare DNS
- [ ] DMARC record added on root domain (p=none initially)
- [ ] SPF record on root domain (no authorised senders — anti-spoofing)
- [ ] All records verified in Resend's domain verification tool
- [ ] Test email sent from each subdomain and delivered to inbox (not spam)
- [ ] All emails send from hello@ (no noreply@)

## Common Mistakes

- Separate mailboxes for every alias (unnecessary cost and complexity for an MVP — one mailbox with routing)
- Sending transactional and marketing from the same subdomain (marketing spam complaints tank transactional deliverability)
- Missing DKIM record (emails fail authentication, land in spam)
- DMARC set to p=reject immediately (blocks legitimate email before you've verified the DNS is correct)
- Using noreply@ when hello@ works for everything (users can't reply, feels cold)
- No SPF on the root domain (spammers can spoof your root domain)
- Forgetting to verify the destination email in Cloudflare Email Routing (routing silently fails)
