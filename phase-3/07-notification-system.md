# Notification System

---

## What It Is

The system that decides when something noteworthy happens, who needs to know about it, and how they should be told. This is not just the bell icon — it's the entire infrastructure for user-facing notifications, email delivery decisions, and internal system alerts.

## Architecture

Every notification flows through the same pipeline regardless of channel:

```
Event Occurs
  │
  ▼
Notification Service
  │ 1. What happened? (event type)
  │ 2. Who needs to know? (recipients)
  │ 3. Through which channel? (in-app, email, both)
  │ 4. Does the user want this? (preferences check)
  │ 5. Should this be batched? (digest check)
  ▼
Channel Delivery
  ├── In-App → real-time via websocket, stored in database
  ├── Email → queued via background job, sent via Resend
  └── Internal Alert → logged, sent to admin channel
```

One system, multiple channels. Never build in-app notifications and email notifications as separate systems — they're two outputs of the same pipeline.

## Event Types

Every notification starts with an event. Events fall into three categories:

**User-triggered events** — another user did something that affects you:
- Workspace invitations, membership changes
- Collaboration activity (your product's equivalent of comments, mentions, assignments)
- These are product-specific — define them based on your feature set

**System events** — the system needs to tell the user something:
- Trial expiring in 3 days / 1 day
- Payment failed
- Plan upgraded/downgraded
- Usage approaching limit (80% threshold)
- Account deletion confirmation
- Password/email change confirmation

**Internal events** — the system needs to tell the team/admins:
- New user signed up
- Subscription created/cancelled
- Payment failed after all retries
- These are supplemented by the observability stack (Sentry, UptimeRobot) for system-level alerts like error spikes and cron failures

## Channel Decision

For each event type, decide which channel(s) to use. This is a design decision made upfront, not at runtime:

| Event | In-App | Email | Notes |
|---|---|---|---|
| Workspace invitation | Yes | Yes | Email is the primary channel (user may not be logged in) |
| Comment on your project | Yes | Digest | Don't email every comment — batch into a digest |
| Task assigned to you | Yes | Yes | Time-sensitive, user needs to know |
| Team member joined | Yes | No | Informational, not urgent |
| Trial expiring (3 day) | Yes | Yes | Critical conversion moment |
| Trial expiring (1 day) | Yes | Yes | Last chance |
| Payment failed | Yes | Yes | Must be seen — revenue depends on it |
| Payment receipt | No | Yes | Email is the right channel for receipts |
| Plan change | Yes | Yes | Confirmation of their action |
| Usage at 80% | Yes | Yes | Prevent surprise limit hits |
| Password changed | No | Yes | Security notification, email only |
| Email changed | No | Yes (both old and new) | Security notification, email only |
| Welcome / verify email | No | Yes | Can't do in-app, they just signed up |

**Rules:**
- Security notifications (password change, email change, new device sign-in) are always email-only — the user may not be logged in, and email is the verified identity
- Receipts and invoices are always email-only — users expect these in their inbox for record-keeping
- High-frequency events (comments, activity) should use in-app primarily with email as a digest — never send individual emails for each one
- Critical business events (payment failed, trial expiring) use both channels — redundancy ensures the message is seen

## User Notification Preferences

Users must be able to control what they receive and through which channel. Never assume.

**Preference structure:**
- Each event category has a toggle per channel (in-app on/off, email on/off)
- Group related events into categories for the preferences UI (e.g., "Activity" covers comments and assignments, "Billing" covers payment and plan events)
- Some notifications cannot be disabled — security notifications (password change, email change) and legal-required notifications (terms of service updates) are always sent regardless of preferences
- Default all preferences to "on" at sign-up — let users opt out, don't make them opt in

**Preferences UI:**
- Lives in user settings under "Notifications"
- Simple table: rows are event categories, columns are channels (in-app, email)
- Each cell is a toggle
- Show which notifications are mandatory (grayed out, always on)

## Email Digests

For high-frequency events, batch notifications into a digest email instead of sending one email per event.

**Design decisions:**
- Digest window: 1 hour for activity notifications (comments, updates, assignments)
- If only 1 event in the window, send it as a normal individual email — don't wrap a single item in a digest template
- If the user has seen the in-app notification already (marked as read), skip the email entirely — they already know
- Digest emails group events by entity (e.g., "3 comments on Project X, 2 comments on Project Y"), not by event type
- Send digests on a schedule (every hour if there are pending notifications), not on a timer from the first event

## In-App Notifications

The bell icon / notification inbox in the app UI.

**Design decisions:**
- Notification inbox is a dropdown panel triggered by the bell icon in the navbar
- Bell icon shows an unread count badge (number, not just a dot)
- Each notification tracks: who it's for, what type of event, a human-readable title and body, a link to navigate to, read/unread status, and when it was created
- Clicking a notification marks it as read and navigates to the relevant page
- "Mark all as read" action available
- Notifications displayed in flat chronological order (newest first) — no grouping in the inbox. Grouping adds complexity without enough value for an MVP.
- Retain notifications for 90 days, then purge

**Real-time delivery:**
- Novu's inbox component handles real-time delivery out of the box — it maintains a websocket connection and updates the inbox when new notifications arrive
- On reconnect after a dropped connection, Novu fetches any missed notifications automatically
- If not using Novu's inbox component (custom UI), use Supabase Realtime (managed) or a raw websocket connection (self-hosted) to push notification events to connected clients. Only push the event ID over the websocket — the client fetches the full notification data from the database.

## Internal / System Alerts

Events that the team or admins need to know about, not end users.

**Design decisions:**
- Critical system alerts (error spikes, cron failures, payment failures after all retries) are handled by the observability stack (Sentry, UptimeRobot) — not by the notification system. Don't build a second alerting system.
- Business events (new sign-up, subscription change, churn) can be sent to a shared channel (Slack, Discord, email group) for team visibility
- These are fire-and-forget — no read/unread tracking, no preferences, no digest. Just log and notify.
- Use a simple webhook or background job to post to the team channel when these events occur

## Notification Service Provider

Use Novu as the notification infrastructure layer. It handles:
- Multi-channel routing (in-app, email, push, SMS from a single API)
- User preference management (built-in preference UI component)
- Digest engine (batching multiple notifications into one)
- In-app inbox component (drop-in React component for the bell icon + notification drawer)
- Template management (notification content templates per channel)
- Integration with Resend for email delivery

Novu sits between your events and your delivery channels. Your application triggers events through Novu's API, Novu checks preferences, applies digest rules, and delivers through the appropriate channel.

## Security Considerations

- Notifications are scoped to workspace — a user never receives notifications for a workspace they don't belong to
- Check membership at delivery time, not event time — user may have been removed between event and delivery
- Notification preferences bypass is limited to security and legal notifications only
- In-app notification data visible to a user must pass through RLS or equivalent permission check

## What to Test

- Notification is created in database when event fires
- In-app notification appears in real-time without page refresh
- Notification click marks as read and navigates to correct page
- Mark all as read clears the unread count
- User preferences are checked at delivery time (not event time)
- Mandatory notifications bypass user preferences
- Digest batches multiple events into one email (within 1-hour window)
- Already-read notifications are excluded from digest email
- Notification for a deleted entity handles "not found" gracefully
- User removed from workspace stops receiving notifications for that workspace

## Edge Cases

- **User is offline when notification fires** — in-app notification is stored in the database. When the user comes online, the inbox loads all unread notifications. The real-time channel only handles delivery of new notifications during an active session.
- **User has email notifications off but the notification is mandatory** — security and legal notifications bypass preferences. The preferences UI should make this clear (grayed out toggles with "Required" label).
- **Digest window has 50+ events** — cap the digest at 10 items with an "and X more" link to the in-app inbox. Don't send a 50-item email.
- **User reads the in-app notification before the digest email is sent** — skip the email. Check read status before sending the digest and exclude already-read notifications.
- **Real-time connection drops and reconnects** — on reconnect, query for notifications created since the last received timestamp. Don't re-fetch all notifications — just the gap.
- **Notification references a deleted entity** — user clicks a notification about a comment on a project that was deleted. The link should handle "not found" gracefully (redirect to dashboard with a toast "This item no longer exists"), not crash.
- **Same event triggers multiple notification types** — e.g., someone comments on your project AND assigns you in the same comment. Deduplicate into one notification: "X commented on Project Y and assigned you."
- **User is a member of multiple workspaces** — notifications must be scoped to the workspace. The notification inbox should show which workspace each notification belongs to, or filter by current workspace.
- **Notification sent to a user who has since been removed from the workspace** — check membership at delivery time, not just at event time. If the user is no longer a member, skip the notification.
- **Email bounces** — if Resend reports a bounce, mark the user's email as undeliverable and stop sending emails to them. Show an in-app banner prompting them to update their email address.

## Implementation Checklist

### Infrastructure
- [ ] Novu configured as the notification service
- [ ] Novu integrated with Resend for email delivery
- [ ] Event types defined for all user-triggered, system, and internal events
- [ ] Channel routing decided per event type (in-app, email, both)

### In-App
- [ ] Notification inbox UI (bell icon, dropdown, unread count badge)
- [ ] Notifications stored in database (recipient, type, title, body, link, read status, timestamp)
- [ ] Real-time delivery via Novu inbox component (or custom websocket if building custom UI)
- [ ] Click notification → mark as read + navigate to relevant page
- [ ] Mark all as read action
- [ ] Flat chronological display (newest first, no grouping)
- [ ] Reconnection handled (missed notifications fetched automatically)
- [ ] 90-day retention with scheduled purge

### Email
- [ ] Channel decision documented per event type
- [ ] Digest engine configured (1-hour window for activity events)
- [ ] Single-event digests send as normal emails (not wrapped in digest template)
- [ ] Already-read notifications excluded from digest
- [ ] Security notifications always sent regardless of preferences
- [ ] Bounce handling (mark email undeliverable, prompt user to update)

### Preferences
- [ ] Preferences UI in user settings (table of categories × channels)
- [ ] All preferences default to "on" at sign-up
- [ ] Mandatory notifications clearly marked (grayed out, non-toggleable)
- [ ] Preferences checked at delivery time (not at event time)

### Internal
- [ ] Business events posted to team channel (Slack/Discord)
- [ ] Critical system alerts handled by observability stack (not this system)

## Common Mistakes

- Building in-app and email notifications as two separate systems (they should be one pipeline with multiple outputs)
- Sending an individual email for every comment/activity event (notification fatigue — use digests)
- Not giving users control over notification preferences (users will mark your emails as spam)
- Defaulting all preferences to "off" (users never discover useful notifications)
- Sending the full data payload over websockets (just send the event, let the client fetch the data)
- Not checking preferences at delivery time (user changed preferences between event and delivery)
- Sending notifications to users who have been removed from the workspace
- No deduplication for related events (5 separate notifications for 5 comments in 2 minutes)
- Not handling email bounces (keeps sending to dead addresses, hurts sender reputation)
- Building a custom alerting system for internal events when Sentry/UptimeRobot already handles this
