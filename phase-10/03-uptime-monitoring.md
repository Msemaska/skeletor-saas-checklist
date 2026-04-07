# Uptime Monitoring & Public Status Page

---

## Provider

Use UptimeRobot for both uptime monitoring and the public status page.

## Uptime Monitoring

### What to Monitor

| Monitor | URL | Check Interval | Alert After |
|---|---|---|---|
| App health check | yourdomain.com/api/health | 5 minutes | 2 consecutive failures |
| Marketing site | yourdomain.com | 5 minutes | 2 consecutive failures |
| API (if public) | api.yourdomain.com/health | 5 minutes | 2 consecutive failures |

**Design decisions:**
- Monitor the health check endpoint (phase-7/02-health-check.md), not just the homepage — the health check verifies database connectivity, not just that the server responds
- Check interval: 5 minutes. More frequent checks (1 minute) are unnecessary for an MVP and may count against UptimeRobot's free tier limits.
- Alert after 2 consecutive failures (10 minutes) — a single failure could be a network blip. Two consecutive failures indicate real downtime.
- Monitor from multiple geographic locations — UptimeRobot does this by default

### Alert Routing

UptimeRobot alerts feed into the unified alerting hub (01-unified-alerting.md):

- **Site down** → #alerts-critical (Discord webhook)
- **Site recovered** → #alerts-info (Discord webhook)
- **Email notification** as backup — in case Discord is also down

## Public Status Page

A public page where customers can check if the service is up without contacting support.

**Design decisions:**
- Use UptimeRobot's built-in status page — no custom development needed
- Host at `status.yourdomain.com` (CNAME to UptimeRobot's status page URL)
- Show monitors for: App, Marketing Site, API (if public)
- Show uptime percentage for the last 90 days
- Include a "Subscribe to updates" option so customers can get email notifications when status changes
- Link to the status page from the marketing site footer and from any error pages (500 page, maintenance page)

**What NOT to include on the status page:**
- Internal services (database, cache, background jobs) — customers don't need to see infrastructure details. They care about "is the app working," not "is the database up."
- Granular component breakdowns — keep it simple. "App" is sufficient. Don't list 15 microservices.

## Incident Communication

When downtime occurs, communicate proactively:

**Design decisions:**
- Post an incident on the status page within 15 minutes of confirmed downtime
- Incident updates every 30 minutes until resolved
- Use clear language: what's affected, what you're doing about it, estimated time to resolution (if known)
- After resolution: post a summary of what happened and what you'll do to prevent it

## What to Test

- Health check monitor is configured and receiving successful checks
- Alert fires when the health check returns 503 (simulate by temporarily breaking the health endpoint)
- Alert posts to the correct Discord channel
- Status page is accessible at status.yourdomain.com
- Status page shows current uptime and monitor status
- "Subscribe to updates" works on the status page

## Edge Cases

- **Health check is up but the app is broken** — the health check only verifies database connectivity. If the app has a bug that breaks all pages but the health endpoint still returns 200, monitoring won't detect it. The health check should verify core functionality, not just process liveness (covered in phase-7/02-health-check.md).
- **UptimeRobot itself is down** — rare, but possible. UptimeRobot's status is independent of your app. If UptimeRobot is down, you won't get alerts. This is an accepted risk — you'd notice from user reports or Sentry error spikes.
- **Status page shows "up" but users report issues** — regional issues, intermittent bugs, or slow performance won't trigger downtime alerts. The status page shows uptime (is the server responding), not performance (is it fast) or correctness (is it bug-free).
- **Scheduled maintenance** — before planned downtime, create a scheduled maintenance event on the status page. This prevents customers from seeing an unexpected "down" status and panicking.

## Implementation Checklist

- [ ] UptimeRobot account created
- [ ] Health check monitor configured (5-minute interval, 2 consecutive failures)
- [ ] Marketing site monitor configured
- [ ] API monitor configured (if public API exists)
- [ ] Alerts routed to unified alerting hub (Discord webhook)
- [ ] Email notification configured as backup
- [ ] Public status page enabled at status.yourdomain.com
- [ ] Status page linked from marketing site footer
- [ ] Status page linked from 500 and maintenance error pages
- [ ] "Subscribe to updates" enabled on status page

## Common Mistakes

- Monitoring the homepage instead of the health check endpoint (homepage can load while the database is down — health check catches this)
- Alerting on single check failures (false positives from network blips)
- No public status page (customers contact support to ask "is it down?")
- Status page shows internal infrastructure details (customers don't need to see database status)
- No incident communication during downtime (customers assume you don't know or don't care)
- Status page not linked from error pages (user hits a 500 error and has no way to check if it's a known issue)
