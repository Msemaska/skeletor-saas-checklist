# Unified Alerting Hub

---

## What It Is

A single channel where ALL operational alerts land — errors, downtime, payment failures, job failures, usage thresholds. The admin never needs to log into Sentry, UptimeRobot, Stripe, or any other dashboard to know something is wrong. Everything comes to one place.

## Channel

Use a dedicated Discord server. Create separate channels for different alert severities:

- `#alerts-critical` — downtime, error spikes, payment system failures. Requires immediate attention.
- `#alerts-warning` — degraded performance, usage thresholds hit, failed background jobs. Needs attention within hours.
- `#alerts-info` — new sign-ups, subscription changes, successful deployments. Informational, no action needed.

## What Feeds Into the Hub

| Source | Alert Type | Channel |
|---|---|---|
| Sentry | New error type, error spike (10x normal rate) | #alerts-critical |
| Sentry | Elevated error rate (2x normal) | #alerts-warning |
| UptimeRobot | Site down | #alerts-critical |
| UptimeRobot | Site recovered | #alerts-info |
| Stripe | Payment failed after all retries | #alerts-warning |
| Stripe | Dispute/chargeback received | #alerts-critical |
| Stripe | New subscription, cancellation | #alerts-info |
| Background jobs | Job failed after all retries | #alerts-warning |
| Background jobs | Critical job failed (billing sync, data migration) | #alerts-critical |
| Usage metering | Workspace hit 80% of limit | #alerts-info |
| Health check | Health endpoint returning 503 | #alerts-critical |
| Deployment | Successful deployment | #alerts-info |
| Deployment | Deployment failed | #alerts-critical |

## How to Connect Each Source

**Sentry:** Sentry has built-in Discord integration. Configure alert rules in Sentry to send notifications to the appropriate channel based on severity.

**UptimeRobot:** UptimeRobot supports webhook notifications. Configure webhooks to post to Discord when monitors go down or recover.

**Stripe:** Stripe doesn't have native Discord integration. Use a lightweight webhook relay: Stripe sends webhooks to your API, your webhook handler posts to Discord for events that need attention (payment failures, disputes, new subscriptions). This happens in the same webhook handler that processes billing events (phase-9/02-stripe-architecture.md).

**Background jobs:** Trigger.dev has webhook/notification support for failed jobs. Configure it to post to the alerting channel on final failure (after all retries exhausted).

**Deployment:** GitHub Actions can post to Discord at the end of the deployment workflow — success or failure.

## Alert Content

Every alert message must include enough context to act without opening the source dashboard:

- **What happened** — "Payment failed for workspace Acme Co" not just "Payment failed"
- **Severity** — critical, warning, or info
- **Link** — direct link to the relevant dashboard (Sentry issue, Stripe payment, UptimeRobot monitor) for deeper investigation
- **Timestamp** — when the event occurred

## Alert Fatigue Prevention

Too many alerts and the team ignores them all.

**Design decisions:**
- Critical channel should fire no more than 2 times per day on a normal day. If it fires more, thresholds are too sensitive.
- Info channel can be noisy — it's opt-in to read. Never put actionable alerts in the info channel.
- Sentry error alerts should be grouped by issue, not by occurrence. One new error type = one alert, not 100 alerts for 100 occurrences of the same error.
- UptimeRobot: set check interval to 5 minutes. Don't alert on a single failed check — wait for 2 consecutive failures (10 minutes of downtime) before alerting.
- Mute alerts during planned maintenance windows

## What to Test

- Each alert source successfully posts to the correct Discord channel
- Alert messages include sufficient context (what, severity, link, timestamp)
- Critical alerts are distinguishable from warnings and info at a glance
- Sentry groups errors by issue (not one alert per occurrence)
- UptimeRobot alerts only after 2 consecutive failures

## Edge Cases

- **Discord is down** — alerts are lost. This is acceptable for a secondary notification channel. The primary detection mechanism (Sentry dashboard, UptimeRobot dashboard) still works. Don't build a backup for the backup.
- **Alert flood during an incident** — a major outage causes hundreds of errors. Sentry's grouping prevents 100 separate alerts, but multiple sources may fire simultaneously (Sentry + UptimeRobot + health check). This is correct — during an incident, you want redundant signals.
- **Nobody is watching the channel** — set up mobile push notifications for the critical channel. Discord supports per-channel notification settings.
- **False positive from UptimeRobot** — a single network blip triggers an alert. The 2-consecutive-failures threshold prevents most false positives. If it still happens, investigate the monitor's location and check interval.

## Implementation Checklist

- [ ] Discord server created with three channels: critical, warning, info
- [ ] Sentry → Discord integration configured with severity-based routing
- [ ] UptimeRobot → Discord webhook configured (down + recovery)
- [ ] UptimeRobot alert threshold set to 2 consecutive failures
- [ ] Stripe webhook handler posts payment failures and disputes to warning/critical channel
- [ ] Stripe webhook handler posts new subscriptions and cancellations to info channel
- [ ] Background job runner posts final failures to warning channel
- [ ] GitHub Actions posts deployment success/failure to info/critical channel
- [ ] Mobile push notifications enabled for the critical channel
- [ ] Alert messages include: what happened, severity, source link, timestamp

## Common Mistakes

- Alerts spread across 5 different dashboards (nobody checks all of them regularly)
- Everything in one channel (critical alerts buried under info noise)
- No link to the source dashboard in the alert (team has to go hunting for context)
- Alert on every single error occurrence (100 alerts for the same bug — alert fatigue)
- UptimeRobot alerting on single check failures (false positives from network blips)
- No mobile notifications for critical alerts (team sees the alert 3 hours later when they check their laptop)
- Stripe alerts not integrated (payment failures discovered when a customer complains, not when they happen)
