# Database Backups & Disaster Recovery

---

## What It Is

How your database is backed up and how you recover from data loss. On Neon, backups are continuous and automatic via its branch-based architecture — but "automatic" doesn't mean "tested" or "sufficient."

## Neon Automated Backups

Neon provides continuous backups through its copy-on-write branching architecture. Every change is durably stored, and you can create a branch from any point in time within the retention window.

**What Neon handles:**
- Continuous WAL-based backup of every transaction (not just daily snapshots)
- Point-in-Time Recovery (PITR) — restore to any specific second within the retention window
- Branch creation from any point in time — create a new branch that reflects the database state at a specific moment
- Backups stored redundantly within Neon's infrastructure

**What Neon does NOT handle:**
- Testing that restores actually work
- External backups (if you want a copy outside Neon's infrastructure)
- Alerting you proactively about data anomalies
- Backing up Cloudflare R2 file storage (separate system — covered in 04-object-storage.md)

**AI interface:** Neon CLI (`neonctl`), Neon MCP server.

## Backup Verification

A backup you've never restored is not a backup. It's a hope.

**Design decisions:**
- Test a restore by creating a branch from a past timestamp at least monthly
- Verify the branch: record counts match expectations, RLS policies are intact, application functions correctly against the branch
- Document the restore process step-by-step so anyone on the team can execute it under pressure
- Set a calendar reminder for monthly restore tests

## Point-in-Time Recovery (PITR)

Neon's branching IS PITR — you create a branch at any point in your retention window. This matters when:

- A bug corrupts data at 14:23 — branch from 14:22, losing only one minute of data instead of hours
- An accidental bulk delete wipes a table — branch from just before the delete
- A bad migration destroys data — branch from before the migration ran

**Design decisions:**
- Know your plan's PITR retention window — Free plan retains 24 hours, Pro plan retains 7+ days
- Upgrade to a plan with longer retention if your product handles critical data
- Recovery creates a new branch — update the application's connection string to point to the recovered branch, or promote it to replace the original

## External Backups

For defense-in-depth (protecting against Neon itself being unavailable):

**Design decisions:**
- Run a weekly `pg_dump` from a Neon branch to an external location (Cloudflare R2 or local encrypted storage)
- This is insurance against provider-level incidents, not your primary recovery strategy
- Retain external backups for 30 days
- The external backup job runs as a scheduled background job (covered in phase-3/10-background-jobs.md)

## Disaster Recovery Plan

Document what to do when the database is lost or corrupted.

**The plan:**
1. Detect the issue (monitoring alerts, user reports, or health check failure)
2. Assess the scope (is it a single table, the entire database, or a configuration issue?)
3. Decide the recovery method: Neon branch from a specific timestamp (for data loss) or external backup restore (for provider-level issues)
4. Execute: create a Neon branch at the target timestamp, or restore from external backup to a new Neon project
5. Update the application's database connection string to point to the recovered branch/project
6. Verify the application works correctly against the restored data
7. Notify affected users if data was lost (transparency builds trust)
8. Post-mortem: what happened, how to prevent it, what to improve

**Design decisions:**
- Write the disaster recovery plan before you need it — not during the incident
- Store the plan in the repository (not in a Google Doc that nobody can find at 3am)
- The plan includes: who to contact, where credentials are stored, step-by-step commands for each recovery method
- Run a disaster recovery drill at least once before launch

## What to Test

- Create a branch from a specific timestamp — verify data is correct at that point
- Application works correctly against a restored branch
- External backup (`pg_dump`) produces a valid backup file
- External backup can be restored to a new Neon project
- Disaster recovery plan is documented and accessible
- At least one team member has executed a full restore

## Edge Cases

- **PITR retention window has passed** — the data loss happened 3 days ago but your plan only retains 24 hours. Upgrade your plan's retention window. External backups are the fallback for incidents outside the PITR window.
- **Connection string changes after restore** — pointing to a new Neon branch means a new connection string. Environment variables must be updated in Railway. Document this in the disaster recovery plan.
- **RLS policies missing after restore** — Neon branches include all schema, RLS policies, and extensions. Verify after every restore test regardless.
- **Partial data loss** — only one table is corrupted. Branch from a specific timestamp, then copy just the affected table's data back to the main branch using SQL. Neon branches make this straightforward since both branches are live simultaneously.
- **Neon outage** — external backups are your insurance. Restore from the latest `pg_dump` to a different Postgres provider if Neon is down for an extended period.

## Implementation Checklist

- [ ] Neon plan with sufficient PITR retention (Pro for 7+ days)
- [ ] Branch restore tested at least once (monthly recurring reminder set)
- [ ] Restore process documented step-by-step in the repository
- [ ] External backup job running weekly (`pg_dump` to R2 or encrypted storage)
- [ ] External backup restore tested at least once
- [ ] Disaster recovery plan written and stored in the repository
- [ ] Disaster recovery drill executed
- [ ] At least one team member has performed a full restore
- [ ] Monitoring detects database unreachability (health check — 09-health-check.md)

## Common Mistakes

- Assuming Neon backups work without ever testing a branch restore
- No documented restore process (figuring out the steps during a 3am incident)
- PITR retention window too short for your needs (24 hours on Free plan may not be enough)
- No external backup strategy (single point of failure on Neon)
- Disaster recovery plan in someone's head, not written down
- Never running a drill (the first time anyone executes the plan is during a real emergency)
- Not notifying users about data loss (they find out on their own and lose trust)
- Forgetting to update connection strings after restore (app still points to the corrupted branch)
