# Transactional Emails

---

## What It Is

System-generated emails triggered by user actions or system events. These are not marketing emails (covered in Phase 12 Email & Domain Setup) — they're operational emails that the user expects to receive because something happened in the app.

## Email Provider

Use Resend for transactional email delivery. Use React Email for building email templates.

**AI interface:** Resend MCP.

## Template System

All emails share a common layout — header, content area, footer. Design the layout once, reuse it for every email.

**Design decisions:**
- Build email templates with React Email — they're React components that render to HTML, testable in development, and consistent with your app's component-based approach
- One shared layout component: logo at the top, content area in the middle, footer with company info, unsubscribe link, and social links at the bottom
- All emails include: your product name and logo, a clear subject line, a primary CTA button where applicable, plain text fallback
- Test every email in React Email's preview mode, dark mode, and on mobile before deploying

**Design system alignment — mandatory:**
Every email must pull directly from the design system defined in phase-1-brand-foundation/05-design-system.md. Emails are a brand touchpoint — a user should immediately recognise the email as coming from your product.
- Colours: use the exact primary, surface, and text colour tokens from the design system. No ad-hoc hex values.
- Typography: use the same type scale and font choices. If the font isn't web-safe, use the fallback stack defined in the design system.
- Logo: use the logo file specified in phase-1-brand-foundation/01-logo.md. Use the version with a transparent background so it works on both light and dark email backgrounds.
- CTA button: use the primary button colour and border radius from the design system.
- Spacing: match the spacing scale. Emails should feel like they came from the same product, not a different tool.
- When the design system is updated, email templates must be updated to match.

## Email Catalogue

Every transactional email your SaaS needs, grouped by trigger:

### Auth Emails
| Email | Trigger | Priority |
|---|---|---|
| Welcome / verify email | User signs up | `[1]` |
| Password reset | User requests reset | `[1]` |
| Email change verification | User changes email | `[1]` |
| Password changed notification | User changes password | `[1]` |
| New device sign-in alert | Sign-in from unrecognised device | `[1]` |

### Workspace Emails
| Email | Trigger | Priority |
|---|---|---|
| Invitation to workspace | Admin invites a member | `[1]` |
| Invitation accepted | Invited user joins | `[1]` |

### Billing Emails
| Email | Trigger | Priority |
|---|---|---|
| Trial expiring (3 days) | 3 days before trial ends | `[2]` |
| Trial expiring (1 day) | 1 day before trial ends | `[2]` |
| Payment receipt / invoice | Successful payment | `[2]` |
| Payment failed | Payment attempt fails | `[2]` |
| Plan upgraded / downgraded | Subscription changes | `[2]` |

### Account Emails
| Email | Trigger | Priority |
|---|---|---|
| Account deletion confirmation | User requests deletion | `[2]` |
| Account deletion completed | Grace period ends, account deleted | `[2]` |

### Product Emails
| Email | Trigger | Priority |
|---|---|---|
| Feature announcement / changelog | New feature released | `[3]` |

## Email Content Specifications

Each email's required content. These are the minimum fields — do not omit any.

### Auth Emails

**Welcome / verify email**
- Subject: "Welcome to [Product] — verify your email"
- Body: one sentence welcome, tell them they need to verify before they can continue, verification link expires in 24 hours
- CTA: "Verify email"
- Tone: warm and brief — they just signed up, don't overwhelm them

**Password reset**
- Subject: "[Product] Reset your password"
- Body: someone requested a password reset for this account, link expires in 1 hour, if this wasn't them they can ignore it and their password won't change
- CTA: "Reset password"
- Tone: neutral and functional — no alarm, no blame

**Email change verification**
- Subject: "[Product] Confirm your new email address"
- Body: a request was made to change the account email to [new email address], old email stays active until this is confirmed, link expires in 24 hours
- CTA: "Confirm new email"
- Tone: informational

**Password changed notification**
- Subject: "[Product] Your password was changed"
- Body: password was changed at [timestamp], device/browser if available, if this wasn't them they should reset their password immediately
- CTA: "Secure my account" — links to password reset
- Tone: security alert, factual not alarming

**New device sign-in alert**
- Subject: "[Product] New sign-in to your account"
- Body: new sign-in at [timestamp] from [device/browser/location if available], if this wasn't them they should secure their account immediately
- CTA: "Secure my account" — links to active sessions page
- Tone: security alert, factual

### Workspace Emails

**Invitation to workspace**
- Subject: "[Inviter name] invited you to [Workspace] on [Product]"
- Body: [Name] has invited you to join [Workspace], what the workspace is for if available, invite expires in 7 days, if they don't have an account they'll be prompted to create one
- CTA: "Accept invitation"
- Tone: friendly and direct

**Invitation accepted**
- Subject: "[Member name] joined [Workspace]"
- Body: [Name] accepted the invitation and joined [Workspace]
- CTA: "View [Workspace]"
- Tone: brief, positive notification

### Billing Emails

**Trial expiring — 3 days**
- Subject: "Your [Product] trial ends in 3 days"
- Body: trial ends on [date], list of features they'll lose access to on free plan, plan price and billing options, link to add payment method
- CTA: "Continue with [Product]"
- Tone: helpful reminder, not pushy

**Trial expiring — 1 day**
- Subject: "Your [Product] trial ends tomorrow"
- Body: same as 3-day email but shorter, trial ends tomorrow [date]
- CTA: "Subscribe now"
- Tone: more urgent, still not alarming

**Payment receipt**
- Subject: "[Product] Receipt for [Month Year]"
- Body: payment of $[amount] received on [date], plan name, billing period (from–to), last 4 digits of card charged, invoice number
- CTA: "View invoice"
- Tone: clean, business-like — this is a record-keeping email

**Payment failed**
- Subject: "[Product] Payment failed — action required"
- Body: payment of $[amount] failed on [date], last 4 digits of the card on file, number of days before access is suspended, direct link to update payment method
- CTA: "Update payment method"
- Tone: matter-of-fact, not alarming — this happens to everyone. Do not use words like "urgent" or "immediately" — just state the facts and the deadline.

**Plan upgraded**
- Subject: "You're now on [Plan name]"
- Body: upgrade confirmed, new plan name, new billing amount and cadence, effective date, 2-3 bullet points of what they now have access to (higher limits, new features)
- CTA: "Explore [Product]"
- Tone: positive confirmation — make them feel good about the decision

**Plan downgraded**
- Subject: "[Product] Your plan has been updated"
- Body: change confirmed, new plan name, when it takes effect, what changes (limits that are lower, features no longer available), note that existing data is preserved
- CTA: "View your plan"
- Tone: neutral and informational — acknowledge the change without being negative or guilt-tripping

### Account Emails

**Account deletion confirmation**
- Subject: "[Product] Account deletion requested"
- Body: deletion request received, grace period length, exact date the account will be permanently deleted, clear option to cancel the deletion if they change their mind
- CTA: "Cancel deletion" — make this prominent, not buried
- Tone: serious but not alarming. Make reversal feel easy.

**Account deletion completed**
- Subject: "[Product] Your account has been deleted"
- Body: account permanently deleted on [date], list what was removed, note what was retained for legal purposes (billing history), contact support if this was a mistake
- No CTA — the account no longer exists
- Tone: factual and final. No upsell. No "we'll miss you."

### Product Emails

**Feature announcement / changelog**
- Subject: "[Product] [Feature name] is now available"
- Body: what it is in one sentence, one key benefit, how to access it
- CTA: "Try it now"
- Tone: excited but concise — one feature per email, never bundle announcements
- Must include unsubscribe link

## Email Content Rules

**Subject lines:**
- Under 50 characters
- Action-oriented: "Verify your email" not "Email verification required"
- Include the product name for context: "[ProductName] Your invoice for March"
- No ALL CAPS, no excessive punctuation, no emoji in subject lines (spam filters)

**Body:**
- One purpose per email — don't combine a password reset with a product update
- Primary CTA is a button, not a text link — large enough to tap on mobile
- Keep body text under 150 words — emails are scanned, not read
- Plain text fallback for every email — some email clients block HTML

**Sender:**
- From name: your product name (not a person's name for system emails)
- From address: hello@mail.yourdomain.com for all transactional emails (no noreply@ — users can reply to any email and it reaches the inbox)
- Consistent From address across all transactional emails

**Footer:**
- Company name and address (required by CAN-SPAM)
- Unsubscribe link for marketing/product emails (not required for pure transactional like password reset, but include on trial expiring, feature announcements, etc.)
- Link to notification preferences in the app

## Email Delivery

The decision of when to send email vs in-app notification is made in the notification system (phase-3-core-product/07-notification-system.md). This file covers the email content and design, not the delivery logic.

**Design decisions:**
- Emails are sent via background jobs (phase-3-core-product/10-background-jobs.md), not synchronously in the request. If Resend is slow or down, the user's action shouldn't fail.
- Every email send is logged: recipient, email type, timestamp, delivery status
- Failed sends are retried by the background job retry logic

## Dark Mode

Email clients increasingly support dark mode. Email templates must look correct in both light and dark mode.

**Design decisions:**
- Use React Email's dark mode support — define both light and dark colour variants
- Test every email in dark mode (Apple Mail, Gmail, Outlook all render dark mode differently)
- Don't rely on background colours for structure — use borders or spacing instead, as background colours may be overridden by dark mode
- Logo: use a version with transparent background that works on both light and dark backgrounds

## What to Test

- Every email renders correctly in React Email preview (desktop and mobile)
- Every email renders correctly in dark mode
- Every email has a plain text fallback
- Every email trigger fires the correct email with the correct recipient and content
- Subject lines are under 50 characters
- CTA buttons are clickable and link to the correct page
- Unsubscribe link works on emails that have one
- Failed email sends are retried
- Password reset / email verification links contain valid tokens and expire correctly

## Edge Cases

- **User has email notifications disabled** — transactional emails (password reset, payment receipt, security alerts) are mandatory and bypass notification preferences. The notification system handles this distinction.
- **Email bounces** — Resend reports bounces. Mark the user's email as undeliverable and show an in-app banner prompting them to update their email address (covered in phase-3-core-product/07-notification-system.md).
- **User signs up and immediately requests password reset** — they receive two emails in quick succession (welcome + reset). Both should arrive and be clearly distinguishable by subject line.
- **Trial expiring email for a user who already upgraded** — check subscription status before sending. If the user upgraded between the trigger and the send, skip the email.
- **Invoice email with $0 amount** — if proration or credits result in a $0 invoice, still send the receipt (the user should know their billing cycle renewed) but adjust the copy: "No payment due this period."
- **Email client strips HTML** — the plain text fallback should be readable and include the CTA as a plain URL. Don't just strip HTML tags — write a proper plain text version.

## Implementation Checklist

- [ ] React Email layout component (header, content, footer)
- [ ] Brand styling applied (colours, typography, logo)
- [ ] All `[1]` emails implemented (auth + workspace)
- [ ] All `[2]` emails implemented (billing + account)
- [ ] Subject lines under 50 characters for all emails
- [ ] Plain text fallback for every email
- [ ] Dark mode tested for every email
- [ ] Mobile rendering tested for every email
- [ ] Sender address configured (hello@mail.yourdomain.com for all transactional emails)
- [ ] Footer includes company info, unsubscribe link, notification preferences link
- [ ] Email sends handled by background jobs (not synchronous)
- [ ] Email delivery logging (recipient, type, timestamp, status)
- [ ] CAN-SPAM compliance (company address in footer, unsubscribe on applicable emails)

## Common Mistakes

- Sending emails synchronously in the request (Resend latency blocks the user's action)
- No plain text fallback (some email clients block HTML entirely)
- Not testing dark mode (logo disappears, text becomes invisible, buttons lose contrast)
- Using noreply@ instead of hello@ (users can't reply, feels impersonal and cold)
- Trial expiring email sent after user already upgraded (bad UX, erodes trust)
- Subject lines over 50 characters (truncated on mobile, looks unprofessional)
- No unsubscribe link on product/feature emails (CAN-SPAM violation, users mark as spam instead)
- Sending password reset and welcome email with identical subject lines (user opens the wrong one)
- Email templates not matching the app's branding (feels like it came from a different product)
