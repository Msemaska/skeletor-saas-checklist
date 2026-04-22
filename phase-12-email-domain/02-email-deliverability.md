# Email Deliverability

---

## What It Is

Ensuring your emails reach the recipient's inbox, not their spam folder. This file covers domain warming, spam testing, and ongoing reputation monitoring.

## Domain Warming

Gradually building sender trust by increasing email volume over time on a new domain.

**Design decisions:**
- Use Resend's shared IP (their default). Warming is handled by Resend's shared reputation — no action needed on your end.
- Start domain warming 4 weeks before launch — send real transactional emails (welcome, verification) to your test accounts and early beta users during this period
- Don't send bulk marketing emails from a brand new domain on day one — start with transactional only, add marketing after 2 weeks of clean sending

## Spam Testing

Before launch, test every email for spam score and rendering.

**Design decisions:**
- Use Mail Tester (mail-tester.com) to check the spam score of every email template before launch
- Target score: 9/10 on Mail Tester. Below 8/10 indicates a problem that will affect delivery.
- Test each email by sending it to the Mail Tester address and reviewing the report
- Mail Tester checks: SPF alignment, DKIM signature, DMARC compliance, content quality, blacklist status, HTML quality

**What Mail Tester flags:**
- Missing or misconfigured SPF/DKIM/DMARC → fix DNS records
- Spammy content (ALL CAPS, excessive exclamation marks, misleading subject lines) → fix email copy
- Missing unsubscribe link on marketing emails → add unsubscribe
- Sending domain on a blacklist → investigate and request removal
- HTML quality issues (missing alt text on images, broken links) → fix email template

## Ongoing Reputation Monitoring

After launch, your sender reputation needs ongoing attention.

**Design decisions:**
- Monitor bounce rate — keep it under 2%. High bounce rates indicate you're sending to invalid addresses. Remove bounced addresses immediately (covered in phase-3-core-product/07-notification-system.md).
- Monitor spam complaint rate — keep it under 0.1% (1 complaint per 1000 emails). Higher rates trigger deliverability issues with major providers.
- Resend provides delivery analytics — check weekly for the first month, monthly after that
- Google Postmaster Tools: register your sending domain to see how Gmail classifies your emails (spam rate, authentication results, domain reputation). This is the most important monitoring tool for deliverability.

## Email Content Best Practices for Deliverability

Beyond DNS and reputation, the content of the email affects whether it reaches the inbox.

**Design decisions:**
- Every email has a plain text version (not just HTML) — emails with only HTML are more likely to be flagged
- Every marketing email has a visible, functioning unsubscribe link
- Subject lines: no ALL CAPS, no excessive punctuation, no misleading language
- From name is consistent across all emails (your product name) — changing the From name frequently looks suspicious
- Include your company address in the footer (CAN-SPAM requirement and deliverability signal)
- Don't use URL shorteners in emails (bit.ly, etc.) — spam filters flag shortened URLs
- Don't send image-only emails — include text content alongside images

## What to Test

- Every email template scores 9/10 on Mail Tester
- SPF, DKIM, and DMARC pass on every sent email (check email headers)
- Emails from mail.yourdomain.com land in inbox (not spam) on Gmail, Outlook, and Apple Mail
- Emails from vibes.yourdomain.com land in inbox on Gmail, Outlook, and Apple Mail
- Bounce rate is under 2% after first week of sending
- Google Postmaster Tools shows "Good" domain reputation

## Edge Cases

- **Emails land in spam despite correct DNS** — check email content for spam triggers (aggressive sales language, too many links, image-heavy with little text). Run through Mail Tester for specific recommendations.
- **Sending domain ends up on a blacklist** — check your domain against major blacklists (MXToolbox). If listed, investigate the cause (usually a spam complaint spike or compromised account), fix the issue, and request removal from the blacklist.
- **Gmail tabs — emails in Promotions instead of Primary** — marketing emails naturally land in the Promotions tab. Transactional emails should land in Primary. Separating subdomains (mail. vs vibes.) helps Gmail categorise correctly.
- **User marks your transactional email as spam** — this is rare but damages reputation. Ensure transactional emails are clearly expected (sent in response to a user action) and recognisable (consistent branding, clear subject line).
- **Volume spike triggers throttling** — if you suddenly send 10x your normal volume (product launch, feature announcement to all users), email providers may throttle delivery. For large sends, use Resend's batch sending which handles throttling automatically.

## Implementation Checklist

- [ ] Domain warming started 4 weeks before launch (transactional emails to test accounts)
- [ ] Transactional sending only for the first 2 weeks (no bulk marketing)
- [ ] Every email template tested with Mail Tester (score 9/10)
- [ ] Google Postmaster Tools configured for sending domain
- [ ] Bounce rate monitored (target: under 2%)
- [ ] Spam complaint rate monitored (target: under 0.1%)
- [ ] Resend delivery analytics reviewed weekly for first month
- [ ] All emails have plain text fallback
- [ ] All marketing emails have unsubscribe link
- [ ] No URL shorteners in email content
- [ ] Company address in email footer

## Common Mistakes

- Sending bulk marketing emails from a brand new domain on day one (no reputation, emails land in spam)
- Not testing with Mail Tester before launch (discover deliverability issues when real users don't receive emails)
- Ignoring bounce rate (sending to invalid addresses tanks reputation)
- No Google Postmaster Tools (flying blind on how Gmail sees your emails)
- URL shorteners in emails (spam filters flag them)
- Image-only emails with no text content (spam filters can't read the image, flag as suspicious)
- Inconsistent From name across emails (looks like spoofing to spam filters)
- Not starting domain warming before launch (first email to real users has zero reputation)
