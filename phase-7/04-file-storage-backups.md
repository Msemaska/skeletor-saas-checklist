# File/Object Storage Backups

---

## What It Is

How user-uploaded files stored in Supabase Storage are backed up. Database backups don't include file storage — they're separate systems that need separate backup strategies.

## Assessment: Do You Need File Storage Backups?

Not all files are worth backing up. The answer depends on whether the files are replaceable.

**Files that are replaceable (no backup needed):**
- Profile pictures (users can re-upload)
- Generated thumbnails and processed images (regenerated from originals)
- Temporary uploads (cleaned up automatically)
- Marketing assets created by the team (stored in the repository)

**Files that are irreplaceable (backup needed):**
- Legal documents, contracts, signed agreements
- User-uploaded documents that are the primary copy (not just an attachment)
- Files required for regulatory compliance

**Design decision:** For most MVP SaaS products, user-uploaded files are replaceable — users can re-upload them. The database (which tracks what files exist and their metadata) is the critical backup target, not the files themselves. Skip file storage backups unless your product stores irreplaceable documents.

## If You Need File Storage Backups

For products that store irreplaceable files:

**Design decisions:**
- Back up to Cloudflare R2 as the secondary storage provider
- Run a daily sync job that copies new and modified files to the backup provider
- The sync job runs as a background job (covered in phase-3/10-background-jobs.md)
- Retain backups for 30 days — this matches the database backup strategy
- The backup is a one-way sync: primary → backup. Never read from the backup during normal operations.
- Test a restore from the backup provider at least quarterly

## Supabase Storage Resilience

Supabase Storage is built on top of S3-compatible object storage, which is inherently durable (designed for 99.999999999% durability). The risk of Supabase losing your files due to hardware failure is extremely low.

The real risks are:
- **Accidental deletion** — a bug in your cleanup job deletes files it shouldn't. The database tracks what files exist — if the database says a file should be there but it's not in storage, you know something went wrong.
- **Supabase account issue** — account suspension, billing issue, or provider outage. Having a backup on a separate provider protects against this.

## What to Test

- Backup sync job copies new files to the backup provider
- Files in the backup provider match files in the primary provider (spot check)
- Restore from backup provider works (file is accessible and intact)
- Orphan detection catches files missing from storage but referenced in the database

## Edge Cases

- **Sync job fails silently** — the backup job errors but nobody notices. Monitor the job's success/failure status (covered in phase-3/10-background-jobs.md). Alert if the backup hasn't completed in 24 hours.
- **File deleted from primary before backup runs** — the file is lost. For truly irreplaceable files, consider a soft-delete approach in storage (move to a "deleted" prefix, purge after 30 days) rather than immediate deletion.
- **Backup storage costs** — duplicating all files to a second provider doubles storage costs. For large file volumes, back up only the irreplaceable bucket, not the public assets bucket.
- **Restore requires updating file URLs** — files in the backup provider have different URLs. Restore by copying files back to the primary provider so existing URLs remain valid.

## Implementation Checklist

- [ ] Assessment: are files replaceable or irreplaceable?
- [ ] If replaceable: no backup needed — document this decision
- [ ] If irreplaceable: backup provider configured (Cloudflare R2)
- [ ] Daily sync job copying new/modified files to backup provider
- [ ] Sync job monitored for failures (alerts if backup hasn't run in 24 hours)
- [ ] Restore tested at least quarterly
- [ ] Orphan detection job running (files in database but missing from storage)

## Common Mistakes

- Backing up all files when most are replaceable (unnecessary cost and complexity)
- Assuming Supabase Storage backups are included in database backups (they're not — separate systems)
- No monitoring on the sync job (backup silently stops working, discovered when you need it)
- Immediate hard-delete of irreplaceable files (no recovery window — use soft-delete in storage)
- Never testing a restore from the backup provider (discover the backup is incomplete during an incident)
