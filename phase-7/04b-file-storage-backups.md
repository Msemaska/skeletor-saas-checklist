# File/Object Storage Backups

---

## What It Is

How user-uploaded files stored in Cloudflare R2 are backed up. Database backups don't include file storage — they're separate systems that need separate backup strategies.

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

## Cloudflare R2 Resilience

Cloudflare R2 is S3-compatible object storage with built-in durability (designed for 99.999999999% durability across multiple data centers). The risk of R2 losing your files due to hardware failure is extremely low.

The real risks are:
- **Accidental deletion** — a bug in your cleanup job deletes files it shouldn't. The database tracks what files exist — if the database says a file should be there but it's not in storage, you know something went wrong.
- **Cloudflare account issue** — account suspension, billing issue, or provider outage. Having a backup on a separate provider protects against this.

## If You Need File Storage Backups

For products that store irreplaceable files:

**Design decisions:**
- Back up to a second R2 bucket in a different Cloudflare account, or to a separate S3-compatible provider
- Use Cloudflare R2's event notifications to trigger backup copies on upload (or run a daily sync job as a background job — covered in phase-3/10-background-jobs.md)
- Retain backups for 30 days — this matches the database backup strategy
- The backup is a one-way sync: primary → backup. Never read from the backup during normal operations.
- Test a restore from the backup at least quarterly

**AI interface:** Cloudflare CLI (`wrangler`), Cloudflare MCP server.

## What to Test

- Backup sync job copies new files to the backup location
- Files in the backup match files in the primary bucket (spot check)
- Restore from backup works (file is accessible and intact)
- Orphan detection catches files missing from storage but referenced in the database

## Edge Cases

- **Sync job fails silently** — the backup job errors but nobody notices. Monitor the job's success/failure status (covered in phase-3/10-background-jobs.md). Alert if the backup hasn't completed in 24 hours.
- **File deleted from primary before backup runs** — the file is lost. For truly irreplaceable files, consider a soft-delete approach in storage (move to a "deleted" prefix, purge after 30 days) rather than immediate deletion.
- **Backup storage costs** — duplicating all files to a second provider doubles storage costs. For large file volumes, back up only the irreplaceable bucket, not the public assets bucket. R2 has no egress fees, so backup syncing is cost-effective.
- **Restore requires updating file URLs** — files in the backup have different URLs. Restore by copying files back to the primary bucket so existing URLs remain valid.

## Implementation Checklist

- [ ] Assessment: are files replaceable or irreplaceable?
- [ ] If replaceable: no backup needed — document this decision
- [ ] If irreplaceable: backup location configured (second R2 bucket or separate provider)
- [ ] Sync job copying new/modified files to backup
- [ ] Sync job monitored for failures (alerts if backup hasn't run in 24 hours)
- [ ] Restore tested at least quarterly
- [ ] Orphan detection job running (files in database but missing from storage)

## Common Mistakes

- Backing up all files when most are replaceable (unnecessary cost and complexity)
- Assuming R2 file backups are included in database backups (they're not — separate systems)
- No monitoring on the sync job (backup silently stops working, discovered when you need it)
- Immediate hard-delete of irreplaceable files (no recovery window — use soft-delete in storage)
- Never testing a restore from the backup (discover the backup is incomplete during an incident)
