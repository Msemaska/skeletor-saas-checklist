# Background & Scheduled Jobs

---

## What It Is

Work that doesn't happen in the request/response cycle. This covers two types: scheduled jobs (run on a timer — "every hour", "every night at 2am") and event-triggered jobs (run in response to something happening — "user uploaded a file", "Stripe webhook received"). Both are async work that runs outside the user's request.

## Architecture

```
Triggers
  ├── User action (via Server Action) ──┐
  ├── Webhook (via API route) ──────────┤
  └── Schedule (cron) ─────────────────┤
                                        ▼
                                  Job Queue
                                    │
                                    ▼
                              Job Execution
                                    │
                                    ├── Success → update database, notify user
                                    └── Failure → retry (with backoff), alert on final failure
```

## Provider

Use Trigger.dev for both scheduled and event-triggered jobs. Use pg_cron for pure database housekeeping.

**When pg_cron is sufficient:**
- Simple database-only scheduled tasks (resetting a counter, purging old records) can use pg_cron directly in Postgres without involving an external service
- pg_cron runs SQL on a schedule inside the database — no network hop, no external dependency
- Use pg_cron for pure database maintenance, Trigger.dev for everything that involves application logic

**Railway cron:**
- Railway supports cron scheduling via its native scheduler, which is sufficient for triggering jobs on a schedule
- For anything beyond simple scheduling (retries, monitoring, concurrency control, complex workflows), Trigger.dev remains the recommendation
- Railway cron is acceptable for triggering a Trigger.dev job on a schedule, but not for running complex job logic itself

## Common Scheduled Jobs

Every SaaS needs these recurring jobs. Define them upfront:

| Job | Schedule | What It Does |
|---|---|---|
| Temporary file cleanup | Daily at 2:00 AM UTC | Delete expired temporary uploads from storage and database |
| Soft-deleted record purge | Daily at 2:15 AM UTC | Hard-delete records where `deleted_at` is older than 30 days |
| Notification digest | Hourly | Batch pending notifications into digest emails (handled by notification system) |
| Usage counter reconciliation | Weekly (Sunday 4:00 AM UTC) | Count actual records and correct any drifted usage counters |
| Expired invitation cleanup | Daily at 2:30 AM UTC | Delete invitation records past their expiry date |
| Session cleanup | Daily at 2:45 AM UTC | Purge expired sessions older than 30 days |
| Orphaned file detection | Weekly (Sunday 5:00 AM UTC) | Find database records with no matching storage file and vice versa |

Schedule maintenance jobs during low-traffic hours (2:00-5:00 AM UTC). Stagger start times so they don't all run simultaneously and compete for resources.

## Common Event-Triggered Jobs

These run in response to a specific event, not on a schedule:

| Trigger | Job | Notes |
|---|---|---|
| Stripe `invoice.paid` webhook | Reset period-based usage counters to 0 | Must be idempotent (webhook may fire twice) |
| File uploaded | Image processing (resize, thumbnails, WebP conversion) | Covered in 06-file-uploads.md |
| User signs up | Send welcome email, create default workspace | Don't block the sign-up response |
| Stripe webhook received | Sync subscription status, update plan, reset usage | Must be idempotent (webhook may fire twice) |
| Workspace deleted | Cascade delete all workspace data and storage files | Too much work for a single request |
| User requests data export | Generate export file (CSV/JSON), notify when ready | Long-running, user gets notified on completion |
| Account deletion requested | Process deletion pipeline (cancel Stripe, anonymize data, purge files) | Multi-step, must complete all steps |

## Job Design Rules

**Every job must be idempotent:**
- Running the same job twice with the same input must produce the same result
- The usage counter reset job should set the counter to 0, not decrement by the current value
- The file cleanup job should check if the file still exists before trying to delete it
- This is critical because retries, duplicate webhooks, and queue replays will cause jobs to run more than once

**Every job must handle failure gracefully:**
- Retry with exponential backoff (1s, 2s, 4s, 8s, 16s) up to 5 retries
- After all retries exhausted, log the failure with full context (job type, input, error, attempt count) and alert via the observability stack
- Never silently swallow errors — a failed job that nobody knows about is worse than a crash

**Every job must be observable:**
- Log when the job starts, completes, and fails
- Track execution duration — a job that normally takes 2 seconds but suddenly takes 60 seconds is a sign of a problem
- Trigger.dev provides a dashboard for this out of the box

**Jobs run in system context:**
- Background jobs don't have a user session — they run with elevated access (system context)
- This means RLS is bypassed — the job must enforce its own access rules
- Never pass user-provided data directly into a job without validating it first (the validation happened in the Server Action that triggered the job, but the job should still verify the referenced resources exist)

## Scheduled Job Timing

**Avoid exact clock times for critical jobs:**
- Don't rely on a job running at exactly 2:00:00 AM — there's always jitter
- Design jobs to be safe to run at any time within a window
- Use idempotency to ensure running twice is harmless

**Job concurrency:**
- Prevent the same scheduled job from running concurrently with itself — if the 2:00 AM cleanup is still running at 2:05 AM, don't start a second instance
- Trigger.dev handles this with concurrency controls
- For pg_cron, use advisory locks to prevent overlap

**Timezone:**
- All schedules in UTC — never local time. Daylight saving time shifts will cause jobs to skip or double-run.

## pg_cron vs Trigger.dev Decision

| Use Case | Tool |
|---|---|
| Pure SQL operations (reset counter, delete expired rows, update timestamps) | pg_cron |
| Anything involving application logic (send email, call API, process file) | Trigger.dev |
| Anything that needs retry, monitoring, or concurrency control | Trigger.dev |
| Anything that runs longer than a few seconds | Trigger.dev |

pg_cron is for database housekeeping. Trigger.dev is for everything else.

## Security Considerations

- Background jobs run in system context (elevated access) — they must enforce their own access rules
- Job inputs should verify that referenced resources exist and belong to the expected workspace before processing
- System context credentials (service keys, database admin) are never logged in job output
- Webhook-triggered jobs validate the webhook signature before queuing

## What to Test

- Job executes successfully and updates the database
- Job is idempotent (running twice with the same input produces the same result)
- Job retries on transient failure with exponential backoff
- Job alerts after all retries are exhausted
- Scheduled jobs run at the correct times (UTC)
- Scheduled jobs don't run concurrently with themselves
- Job handles deleted resources gracefully (checks existence before processing)
- Webhook deduplication prevents duplicate job execution

## Edge Cases

- **Job references a deleted resource** — a file processing job is triggered, but by the time it runs, the user deleted the file. The job should check if the resource still exists before processing and exit gracefully if it doesn't.
- **Webhook triggers a job that's already running** — Stripe sends a duplicate webhook. The job queue should deduplicate by event ID (covered in webhook deduplication in 01-database-schema.md). If deduplication happens before the job is queued, the duplicate never runs.
- **Scheduled job fails repeatedly** — after 5 retries, the job stops retrying but the schedule keeps triggering new instances. Each new instance is independent — the failed one doesn't block future runs. Alert on repeated failures so someone investigates.
- **Job takes longer than expected** — an image processing job normally takes 2 seconds but a user uploads a 50MB image and it takes 5 minutes. Trigger.dev handles long-running jobs without timeout limits. Set an upper bound timeout (30 minutes) as a safety net to kill truly stuck jobs.
- **Database migration while jobs are running** — a migration changes a table structure while a background job is querying that table. Schedule migrations during maintenance windows and pause job execution during deploys, or design jobs to handle schema changes gracefully.
- **Job needs to notify the user on completion** — use the notification system (07-notification-system.md) to send an in-app notification when the job finishes. The job triggers a notification event, the notification system handles delivery.
- **Staggered bulk operations** — a cleanup job needs to delete 10,000 records. Don't delete all at once — process in batches (100 per batch) with a short delay between batches to avoid locking the table and blocking user queries.

## Implementation Checklist

### Infrastructure
- [ ] Trigger.dev configured and connected to the application
- [ ] pg_cron enabled for database-level scheduled tasks
- [ ] All scheduled jobs defined with cron expressions and staggered timing
- [ ] All event-triggered jobs defined with their trigger sources
- [ ] Job dashboard accessible for monitoring and debugging

### Job Design
- [ ] Every job is idempotent (safe to run twice with the same input)
- [ ] Every job retries with exponential backoff (5 retries default)
- [ ] Every job logs start, completion, and failure with context
- [ ] Failed jobs alert via observability stack after retries exhausted
- [ ] Jobs run in system context with explicit resource existence checks
- [ ] Bulk operations process in batches, not all at once

### Scheduled Jobs
- [ ] All schedules in UTC
- [ ] Maintenance jobs staggered between 2:00-5:00 AM UTC
- [ ] Concurrency protection (no duplicate concurrent runs of the same job)
- [ ] pg_cron used for pure SQL tasks (counter reset, record purge)
- [ ] Trigger.dev used for application logic tasks

### Common Jobs Implemented
- [ ] Temporary file cleanup (daily)
- [ ] Soft-deleted record purge (daily, 30-day threshold)
- [ ] Expired invitation cleanup (daily)
- [ ] Session cleanup (daily)
- [ ] Usage counter reconciliation (weekly)
- [ ] Orphaned file detection (weekly)

## Common Mistakes

- Running slow operations in the Server Action response (blocks the user, risks timeout)
- Non-idempotent jobs (running twice produces duplicate emails, double charges, corrupted data)
- No retry mechanism (external API is down, job fails permanently instead of retrying)
- Silent failures (job fails, nobody knows, data drifts)
- Using a basic cron scheduler for complex job logic instead of Trigger.dev (no retry, no monitoring, no concurrency control)
- All maintenance jobs scheduled at the same time (resource contention, slow queries for users)
- Deleting 10,000 records in a single query (locks the table, blocks user requests)
- Jobs running in user context instead of system context (RLS blocks the operation)
- Not checking if the resource still exists before processing (race condition with user deletions)
- Scheduled jobs using local time instead of UTC (daylight saving causes skips and double-runs)
- No concurrency control on scheduled jobs (long-running job overlaps with the next scheduled run)
