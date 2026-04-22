# Object Storage

---

## What It Is

Where user-uploaded files, images, and static assets are stored and served from. Instead of storing files on the server's local disk (which doesn't scale and is lost if the server dies), files go to a dedicated object storage service with CDN delivery.

## Provider

Use Cloudflare R2 for object storage. S3-compatible API, no egress fees, and integrates with Cloudflare's CDN for fast global delivery.

**AI interface:** Cloudflare CLI (`wrangler`), Cloudflare MCP server.

## Architecture

```
Upload Flow:
  App → signed URL → Client → R2 (direct upload)

Serving Flow (public):
  Client → Cloudflare CDN → R2

Serving Flow (private):
  Client → App (generates signed URL) → R2
```

The upload and serving architecture is the same as described in phase-3-core-product/06-file-uploads.md. This doc covers the infrastructure-level decisions.

## Bucket Configuration

**Design decisions:**
- Two buckets: one public (CDN-served, no auth to read), one private (signed URLs required)
- Bucket naming: `{app-name}-public`, `{app-name}-private`
- Connect a custom subdomain for the public bucket: `assets.yourdomain.com` — this serves files through Cloudflare's CDN with your domain
- CORS configuration: allow uploads from your app's domain only
- No lifecycle rules for the public bucket (files are permanent unless explicitly deleted)
- Lifecycle rule on the private bucket: automatically delete objects older than 90 days if they haven't been marked as permanent (handled by the application's cleanup job)

## CDN

Cloudflare CDN sits in front of the public bucket automatically when you connect a custom domain.

**Design decisions:**
- Cache public assets with long TTLs (1 year) — files are immutable (new version = new path, covered in phase-3-core-product/06-file-uploads.md)
- No caching on signed URLs (private bucket) — each signed URL is unique and short-lived
- Enable Cloudflare's image optimization (Polish, WebP conversion) for additional compression on served images

## Backups

**Design decisions:**
- Public bucket: no backup needed. These are derived assets (uploaded by users, processed by the app). If lost, users can re-upload. The database tracks what files exist — the files themselves are replaceable.
- Private bucket: back up to a second provider (e.g., R2 → Backblaze B2 or another R2 bucket in a different region) if the files contain irreplaceable data (legal documents, contracts).
- For most MVP SaaS products, the database is the critical backup target, not file storage. Files are important but not as catastrophic to lose as the database.

## Alternative: MinIO (fully self-hosted)

If you need file storage entirely on your own servers (regulatory requirements, data sovereignty):

- MinIO is an S3-compatible object storage server that runs as a Docker container
- Runs on your server, stores files on a Docker volume or mounted disk
- Same S3 API as R2 — the application code doesn't change
- Requires more operational overhead: disk monitoring, backups, and no built-in CDN (put Cloudflare or another CDN in front)
- Use MinIO only when you can't use a managed service. R2 is simpler and cheaper for most cases.

## Edge Cases

- **R2 is temporarily unavailable** — uploads fail. The application should catch the error and show "Upload failed, please try again." Don't let a storage outage crash the app.
- **Public bucket URL guessing** — files in the public bucket are accessible by URL. Don't store sensitive files in the public bucket. Use UUIDs in file paths so URLs are unguessable.
- **CORS blocks uploads** — if the CORS configuration doesn't include your app's domain, browser-based uploads fail silently. Test uploads from the browser, not just from the server.
- **Files accumulate without cleanup** — temporary uploads, orphaned files, and failed uploads take up space. The application's cleanup job (phase-3-core-product/10-background-jobs.md) handles this, but verify it's actually running and deleting files.
- **Migration from one storage provider to another** — if the application uses the S3 API (which R2, MinIO, and S3 all support), migration is changing credentials and endpoint URL. No code changes needed.

## Security Considerations

- Private bucket access is controlled entirely by signed URLs. Signed URLs must have short expiry times (15 minutes for downloads, 5 minutes for uploads). A leaked signed URL grants temporary access — short expiry limits the window.
- R2 API credentials (access key ID and secret) grant full read/write access to all buckets. Store them in the `.env` file on the server, never in the repository. Rotate credentials periodically and immediately if a breach is suspected.
- Set hard upload size limits at the infrastructure level (R2 or MinIO configuration) as a backstop — application-level validation (covered in phase-3-core-product/06-file-uploads.md) handles the fine-grained rules, but the infrastructure must reject oversized payloads before they consume resources.
- If using MinIO (fully self-hosted), the MinIO admin console must not be exposed to the internet. Run it on the Docker network only and access it via SSH tunnel if needed.
- Cloudflare R2 encrypts data at rest by default. If using MinIO, enable server-side encryption and ensure the underlying disk is also encrypted.

## Implementation Checklist

- [ ] R2 account set up with two buckets (public and private)
- [ ] Custom subdomain connected for public bucket (assets.yourdomain.com)
- [ ] CORS configured to allow uploads from app domain only
- [ ] Public bucket served via Cloudflare CDN with long cache TTLs
- [ ] Signed URLs used for all private bucket access
- [ ] Backup strategy defined for private bucket (if containing irreplaceable data)
- [ ] Application cleanup job verified — temporary and orphaned files are being deleted
- [ ] Upload and download tested from the browser (not just server-side)

## Common Mistakes

- Storing files on the server's local disk (not scalable, lost on server failure, no CDN)
- Sensitive files in the public bucket (anyone who guesses the URL can access them)
- No CORS configuration (uploads fail from the browser with no clear error)
- Using AWS S3 instead of R2
- No cleanup job for temporary uploads (storage costs grow indefinitely)
- Backing up file storage instead of prioritizing database backups (the database is more critical — files are often replaceable)
