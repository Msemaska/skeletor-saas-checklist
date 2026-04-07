# Database Backups & Disaster Recovery

---

## What It Is

How your database is backed up and how you recover from data loss. On Supabase, daily backups are automatic — but "automatic" doesn't mean "tested" or "sufficient."

## Supabase Automated Backups

Supabase provides daily automated backups on all paid plans.

**What Supabase handles:**
- Daily backups of the full database (all schemas, tables, RLS policies)
- Point-in-Time Recovery (PITR) on Pro plan and above — recover to any specific second within the retention window
- Backups stored in a separate region from the database

**What Supabase does NOT handle:**
- Testing that restores actually work
- Alerting you if a backup fails
- Backing up Supabase Storage (file/object storage is separate — covered in 04-file-storage-backups.md)
- Custom retention beyond the plan's default

## Backup Verification

A backup you've never restored is not a backup. It's a hope.

**Design decisions:**
- Test a restore to a staging environment at least monthly
- Document the restore process step-by-step so anyone on the team can execute it under pressure (during an incident, not the time to figure out the process)
- The restore test verifies: data integrity (record counts match), RLS policies are intact, application functions correctly against the restored database
- Set a calendar reminder for monthly restore tests — this will be forgotten without a recurring reminder

## Point-in-Time Recovery (PITR)

PITR allows recovery to any specific second, not just the last daily backup. This matters when:

- A bug corrupts data at 14:23 — you can recover to 14:22, losing only one minute of data instead of up to 24 hours
- An accidental bulk delete wipes a table — recover to just before the delete
- A bad migration destroys data — recover to before the migration ran

**Design decisions:**
- Enable PITR if your Supabase plan supports it (Pro plan and above)
- Know the PITR retention window for your plan — this is how far back you can recover
- PITR recovery creates a new database instance — you restore to a new project and then migrate your application to point to it

## Disaster Recovery Plan

Document what to do when the database is lost or corrupted.

**The plan:**
1. Detect the issue (monitoring alerts, user reports, or health check failure)
2. Assess the scope (is it a single table, the entire database, or a configuration issue?)
3. Decide the recovery method: PITR (for recent data loss) or daily backup restore (for older issues)
4. Execute the restore to a new Supabase project
5. Update the application's database connection to point to the restored database
6. Verify the application works correctly against the restored database
7. Notify affected users if data was lost (transparency builds trust)
8. Post-mortem: what happened, how to prevent it, what to improve in the recovery process

**Design decisions:**
- Write the disaster recovery plan before you need it — not during the incident
- Store the plan in the repository (not in a Google Doc that nobody can find at 3am)
- The plan includes: who to contact, where credentials are stored, step-by-step commands for each recovery method
- Run a disaster recovery drill at least once before launch — execute the full plan against a staging environment

## What to Test

- Daily backups are running (verify in Supabase dashboard)
- Restore from backup produces a functional database (monthly test)
- Application works correctly against the restored database
- PITR recovery to a specific timestamp works (if enabled)
- Disaster recovery plan is documented and accessible
- At least one team member has executed a full restore

## Edge Cases

- **Backup succeeds but restore fails** — Supabase backups can be corrupted in rare cases. Monthly restore tests catch this before you need the backup for real.
- **PITR retention window has passed** — the data loss happened 3 days ago but PITR only retains 7 days. You have 4 days to notice and act. Monitoring and alerting help detect issues quickly.
- **Database connection string changes after restore** — restoring to a new Supabase project means a new connection string. The application's environment variables must be updated. Document this in the disaster recovery plan.
- **RLS policies missing after restore** — verify RLS policies are intact after every restore test. If they're missing, the restored database is a security risk.
- **Partial data loss** — only one table is corrupted. PITR restores the entire database, not individual tables. You may need to restore to a temporary database and copy just the affected table back.

## Implementation Checklist

- [ ] Supabase paid plan active (daily backups enabled)
- [ ] PITR enabled (if Pro plan or above)
- [ ] Restore tested at least once (monthly recurring reminder set)
- [ ] Restore process documented step-by-step in the repository
- [ ] Disaster recovery plan written and stored in the repository
- [ ] Disaster recovery drill executed against staging
- [ ] At least one team member has performed a full restore
- [ ] Monitoring detects database unreachability (health check — 02-health-check.md)

## Common Mistakes

- Assuming Supabase backups work without ever testing a restore (discover it's broken when you need it most)
- No documented restore process (figuring out the steps during a 3am incident)
- PITR not enabled (losing up to 24 hours of data on every incident instead of seconds)
- Disaster recovery plan in someone's head, not written down (that person is on vacation when the incident happens)
- Never running a drill (the first time anyone executes the plan is during a real emergency)
- Not notifying users about data loss (they find out on their own and lose trust)
