# Incident Runbook

---

## What It Is

A documented set of procedures for handling common incidents — what to do when things break, who to contact, and how to communicate with users. Written before incidents happen, not during them.

## Where It Lives

Store in the repository at `docs/runbook.md`. Not in Notion, not in a Google Doc — in the repo where it's version-controlled, searchable, and accessible when other tools might be down.

## Incident Severity Levels

| Level | Definition | Response Time | Example |
|---|---|---|---|
| Critical | Product is down or data is at risk | 15 minutes | Server unresponsive, database corrupted, security breach |
| High | Major feature is broken for all users | 1 hour | Auth broken, billing not processing, file uploads failing |
| Medium | Feature degraded or broken for some users | 4 hours | Slow performance, intermittent errors, one integration down |
| Low | Minor issue, workaround exists | 24 hours | UI glitch, non-critical email not sending, cosmetic bug |

## Incident Response Process

Every incident follows the same 5-step process:

### 1. Detect

**Design decisions:**
- Detection comes from: unified alerting hub (phase-10-observability/01-unified-alerting.md), user reports via Tawk.to, Sentry error spike, UptimeRobot downtime alert
- The person who detects the incident starts the response — don't wait for someone else to notice

### 2. Assess

**Design decisions:**
- Determine severity level using the table above
- Identify: what's broken, who's affected (all users, one workspace, one feature), when it started
- Check Sentry for error details, check server/database health, check recent deployments

### 3. Communicate

**Design decisions:**
- Critical and High: update the public status page (UptimeRobot) within 15 minutes. Post in the Discord #alerts-critical channel.
- Medium: no public communication unless it persists for more than 2 hours. Post in Discord #alerts-warning.
- Low: no public communication. Track internally.
- Status page updates use clear language: what's affected, what you're doing, estimated resolution time (if known)
- Update the status page every 30 minutes until resolved

### 4. Fix

**Design decisions:**
- If caused by a bad deployment: rollback immediately (phase-7-infrastructure/05-cicd-pipeline.md)
- If caused by a database issue: check backups, consider point-in-time recovery (phase-7-infrastructure/03b-database-backups.md)
- If caused by a third-party service: check the service's status page, implement workaround if possible, notify users that the issue is external
- If cause is unknown: add logging, reproduce the issue, fix and deploy
- Document every action taken during the fix (timestamp + what was done) — this becomes the post-mortem timeline

### 5. Post-Mortem

**Design decisions:**
- Write a post-mortem for every Critical and High incident within 48 hours
- Post-mortem includes: what happened, timeline of events, root cause, what was done to fix it, what will be done to prevent it from happening again
- Store post-mortems in the repository at `docs/post-mortems/YYYY-MM-DD-incident-title.md`
- Share the post-mortem with the team. For user-facing incidents, publish a simplified version on the status page.
- Post-mortems are blameless — focus on systems and processes, not people

## Common Incident Playbooks

### App is Down (Critical)

1. Check UptimeRobot — confirm the health check is failing
2. Check the hosting provider (Railway dashboard or server SSH)
3. If bad deployment: rollback to previous deployment
4. If server issue (self-hosted): check Docker containers, restart if needed
5. If database issue: check database connectivity, check disk space, check connection pool
6. Update status page: "We're investigating an issue affecting access to [Product]. We'll update every 30 minutes."
7. Once resolved: update status page, post in Discord, write post-mortem

### Auth is Broken (High)

1. Check Sentry for auth-related errors
2. Check Neon Auth status page and application auth logs
3. Check recent deployments — did middleware or auth config change?
4. If deployment-related: rollback
5. If provider-related: notify users, wait for provider resolution
6. Update status page if affecting all users

### Billing Not Processing (High)

1. Check Stripe Dashboard for failed webhooks
2. Check your webhook endpoint — is it responding? Are there errors in Sentry?
3. Check the webhook deduplication table — are events being processed?
4. If webhook endpoint is down: fix and replay missed events from Stripe Dashboard
5. If Stripe is down: check Stripe's status page, wait for resolution
6. Notify affected users if payments failed

### Database Performance Degraded (Medium)

1. Check slow query log (pg_stat_statements)
2. Check connection pool usage (PgBouncer stats)
3. Check disk space and CPU usage
4. Identify the slow query — add missing index or optimize
5. If disk space: clean up temporary data, expand disk
6. If connection pool exhausted: check for connection leaks in the application

### Third-Party Service Down (Medium)

1. Check the service's status page
2. Determine impact: which features are affected?
3. If the app handles the outage gracefully (phase-3-core-product/12-error-handling-logging.md): no action needed, feature degrades automatically
4. If the app crashes: deploy a hotfix to handle the outage gracefully
5. Notify users only if the impact is user-visible and persists for more than 2 hours

## What to Test

- Runbook is stored in the repository and accessible
- Severity levels are defined and understood
- Status page can be updated within 15 minutes (practice updating it)
- Rollback procedure works (test before you need it)
- Post-mortem template exists and is accessible
- Every team member knows where the runbook is

## Edge Cases

- **Incident happens while the runbook author is unavailable** — the runbook must be self-contained. Any team member should be able to follow it without additional context.
- **Multiple incidents at the same time** — prioritise by severity. Critical over High, High over Medium. If two Critical incidents happen simultaneously, fix the one affecting more users first.
- **Incident is caused by a security breach** — follow the incident process AND the security audit procedure (phase-12-security-audit/01-security-audit.md). Take the affected system offline immediately. Do not communicate publicly until you understand the scope.
- **User data was lost** — check backups. Restore from the most recent backup. Notify affected users with transparency about what happened and what was lost.

## Implementation Checklist

- [ ] Runbook stored in repository at docs/runbook.md
- [ ] Severity levels defined (Critical, High, Medium, Low)
- [ ] Response time targets per severity level
- [ ] 5-step incident process documented (Detect, Assess, Communicate, Fix, Post-Mortem)
- [ ] Playbooks written for: app down, auth broken, billing not processing, database degraded, third-party down
- [ ] Post-mortem template created at docs/post-mortems/
- [ ] Status page update process practiced (can you update it in 15 minutes?)
- [ ] Rollback procedure tested
- [ ] Every team member knows where the runbook is

## Common Mistakes

- No runbook (figuring out the process during a 3 AM incident)
- Runbook in a Google Doc nobody can find (put it in the repo)
- No severity levels (every incident gets the same panicked response)
- No public communication during extended outages (users think you don't know or don't care)
- No post-mortem (same incident happens again because nobody analysed the root cause)
- Post-mortems that blame people instead of systems
- Never testing the rollback procedure (first rollback attempt is during a real incident)
- Runbook that only one person understands (bus factor of 1)
