# Transactional Emails

---

## What It Is

System-generated emails triggered by user actions or system events. These are not marketing emails (covered in Phase 12 Email & Domain Setup) — they're operational emails that the user expects to receive because something happened in the app.

## Email Provider

Use Resend for transactional email delivery. Use React Email for building email templates.

## Template System

All emails share a common layout — header, content area, footer. Design the layout once, reuse it for every email.

**Design decisions:**
- Build email templates with React Email — they're React components that render to HTML, testable in development, and consistent with your app's component-based approach
- One shared layout component: logo at the top, content area in the middle, footer with company info, unsubscribe link, and social links at the bottom
- Email styling uses the same brand colors and typography as the app — users should recognise the email as coming from your product
- All emails include: your product name and logo, a clear subject line, a primary CTA button where applicable, plain text fallback
- Test every email in React Email's preview mode, dark mode, and on mobile before deploying

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

The decision of when to send email vs in-app notification is made in the notification system (phase-3/07-notification-system.md). This file covers the email content and design, not the delivery logic.

**Design decisions:**
- Emails are sent via background jobs (phase-3/10-background-jobs.md), not synchronously in the request. If Resend is slow or down, the user's action shouldn't fail.
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
- **Email bounces** — Resend reports bounces. Mark the user's email as undeliverable and show an in-app banner prompting them to update their email address (covered in phase-3/07-notification-system.md).
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
