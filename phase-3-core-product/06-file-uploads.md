# File Uploads & Image Processing

---

## What It Is

The system that handles users uploading files to your product, storing them securely, processing them (resizing, thumbnails, format conversion), and serving them back. This is one of the most common features in SaaS products — profile pictures, document attachments, media uploads — and one of the most commonly implemented badly.

## Upload Architecture

Files go directly from the browser to object storage using signed upload URLs. They never pass through your application server.

```
Browser
  │
  │ 1. Request signed upload URL
  ▼
Server Action
  │ 2. Validate auth + input
  │ 3. Generate signed upload URL
  │ 4. Return URL to client
  ▼
Browser
  │ 5. Upload file directly to storage using signed URL
  │ 6. Show progress bar during upload
  ▼
Object Storage
  │ 7. File stored
  ▼
Server Action (or webhook)
  │ 8. Create database record linking file to entity
  │ 9. Trigger image processing if needed
  ▼
Done
```

**Why signed URLs instead of uploading through the server:**
- Files never hit your application server — no memory pressure, no timeout risk on large files
- Upload goes directly to object storage, which is built for handling large binary data
- Signed URLs are time-limited (expire after 1-2 hours) — they can't be reused or shared
- The server only generates the URL after validating auth and permissions — unauthorized uploads are impossible

## Storage Configuration

### Bucket Strategy

Separate buckets by access pattern, not by content type:

**Public bucket** — for files that anyone can access via direct URL:
- Company logos, published blog images, public avatars
- No authentication required to view
- Served via CDN for fast global delivery
- Upload still requires authentication (public read, private write)

**Private bucket** — for files that require authentication to access:
- User documents, attachments, internal files
- Access via signed download URLs (time-limited, typically 1 hour)
- Never expose the raw storage URL — always generate a signed URL server-side after checking permissions

### Path Structure

File paths must encode ownership for access control:

- **Multi-tenant:** `{workspace_id}/{entity_type}/{entity_id}/{filename}`
- **Single-tenant:** `{user_id}/{entity_type}/{entity_id}/{filename}`

Examples:
- `ws_abc123/avatars/user_def456/avatar.webp`
- `ws_abc123/attachments/project_ghi789/document.pdf`

**Rules:**
- Never use the client-provided filename for storage — generate a unique filename server-side (UUID or content hash) and store the original name as metadata in the database
- Include the workspace/user ID in the path so storage-level access policies can verify ownership
- Include the entity type and ID so you can find all files for a given entity

### Storage-Level Access Policies

- Upload: only authenticated users who are members of the workspace in the file path
- Download (private bucket): only authenticated users who are members of the workspace in the file path
- Download (public bucket): anyone with the URL
- Delete: only authenticated users who own the entity the file belongs to, or workspace admins
- All policies validate that the workspace_id in the file path matches the authenticated user's workspace membership

## Upload Flow — Design Decisions

### Before Upload (client-side validation)

Validate before the file leaves the browser — don't waste bandwidth on files that will be rejected:

- **File size** — enforce a maximum per upload: 10MB for images, 50MB for documents. Show the limit in the UI.
- **File type** — check the file extension against an allowlist. This is UX only — the real validation happens server-side.
- **Image dimensions** — for profile pictures or avatars, check minimum dimensions client-side so the user knows immediately if their image is too small.

### During Upload

- Show a progress bar for any file over ~500KB — use XMLHttpRequest (not fetch) as it supports upload progress events
- Allow cancellation mid-upload
- For resumable uploads on large files (>50MB), use the TUS protocol — it allows resuming from where it left off after network interruptions

### After Upload (server-side validation)

The server must validate the file independently of client-side checks:

- **File type by magic bytes** — read the first few bytes of the file to verify the actual file type. Don't trust the Content-Type header or file extension — both are trivially spoofed. (Covered in detail in 02-validation.md under Path Traversal / File Upload Attacks)
- **File size** — enforce the same limit server-side. The storage provider should also have a per-bucket size limit as a backstop.
- **Image dimensions** — if you require specific dimensions (avatars must be square, banners must be 16:9), validate after upload.

### Database Record

After a successful upload, create a database record linking the file to its entity. Track: the storage path, the original filename the user provided, the verified mime type and file size (from server-side validation), which entity this file belongs to (type and ID), who uploaded it, whether it's in a public or private bucket, and the tenant boundary (workspace or user ID).

This record is the source of truth for what files exist. Never query storage directly — always go through the database.

## Image Processing

If your product handles image uploads, you almost always need processing — at minimum for thumbnails and format optimization.

### When to Process

Process on upload via background job. The upload completes immediately, a background job picks up the processing, and the processed versions are available within seconds. The UI shows a placeholder or the original until processing completes.

### What to Process

**Thumbnails:**
- Generate 3 sizes per image: thumbnail (150x150), medium (600x600), and original
- Maintain aspect ratio — crop to square for avatars, fit within bounds for content images
- Store each size as a separate file in the same path: `{path}/thumb.webp`, `{path}/medium.webp`, `{path}/original.webp`

**Format conversion:**
- Convert all uploaded images to WebP for serving — smaller file size, better quality, universally supported in 2026
- Keep the original file as-is for download/export purposes
- For transparency (logos, icons), use WebP with alpha channel or fall back to PNG

**Metadata stripping:**
- Strip EXIF data from all uploaded images before serving — EXIF can contain GPS coordinates, camera info, and other private metadata the user didn't intend to share
- Do this during processing, not on upload — the original file should retain EXIF for the user's own use

### Processing Pipeline

- Use Sharp for all image processing — it's the standard Node.js library, fast, and handles resize, format conversion, EXIF stripping, and quality optimization
- Processing runs as a background job (Trigger.dev / Inngest), not in the upload request
- After processing completes, update the database record with the paths to the processed versions
- If processing fails, log the error and keep the original — never delete the original if processing fails

## Serving Files

### Public Files
- Serve via the storage provider's public URL + CDN
- Cache headers are set by the storage provider — typically long-lived with content-based cache keys
- When a file is updated (e.g., user changes their avatar), upload to a new path rather than overwriting — this avoids CDN cache propagation delays

### Private Files
- Generate a signed download URL server-side after verifying the user's permissions
- Signed URLs expire after 1 hour — short enough to be secure, long enough for a browsing session
- Never cache signed URLs client-side beyond their expiry time
- For images in private buckets, you can generate signed URLs for each processed size (thumb, medium, original) and pass them all to the frontend

### Serving in Next.js
- Use `next/image` for all image rendering — it handles lazy loading, responsive sizes, and format optimization
- For images in public buckets, configure the storage domain in `next.config` as an allowed image domain
- For images in private buckets, use signed URLs as the `src` — `next/image` works with any URL

## Temporary vs Permanent Storage

Not all uploads are permanent. Some files are temporary (evaluation, processing, draft states).

- Flag temporary uploads with `is_temporary` in the database record
- Set an expiry (e.g., 7 days) for temporary files
- A scheduled cleanup job deletes expired temporary files from both storage and the database
- When a user confirms or publishes, clear the temporary flag — the file becomes permanent
- Files that are never confirmed get cleaned up automatically

## What to Test

### Signed URLs
- Signed upload URL is generated only for authenticated users
- Signed upload URL expires after 5 minutes — upload rejected after expiry
- Signed download URL expires after 15 minutes — access rejected after expiry
- Direct URL to private file returns 403 without a signed URL
- Public file is accessible via CDN URL without authentication

### Validation
- File type validation rejects files where extension and magic bytes do not match
- Images over 10MB are rejected
- Documents over 50MB are rejected
- Upload is blocked when it would push the workspace over its storage limit

### Storage
- Uploaded files are stored with server-generated UUIDs, not client-provided filenames
- Upload to public bucket makes file accessible via CDN URL
- Upload to private bucket requires signed URL for access

### Image Processing
- Processing generates 150x150 thumbnail
- Processing generates 600x600 medium variant
- Original file is preserved
- EXIF metadata is stripped from all served images

### Cleanup
- Temporary files are deleted by the scheduled cleanup job after expiry
- Entity deletion cascades to file deletion in storage
- Orphaned files (upload started, entity never created) are removed by scheduled cleanup job
- Failed upload does not create a file record in the database

## Edge Cases

- **Upload interrupted by network failure** — for large files, use resumable uploads (TUS protocol). For small files, show an error and let the user retry. Never leave partial uploads in storage — the storage provider handles cleanup for failed uploads.
- **User uploads a file then deletes the entity it belongs to** — cascade delete the file records. Use a background job to delete the actual files from storage (don't block the entity deletion on storage API calls).
- **Signed upload URL expires before upload completes** — the upload fails. The client should request a new signed URL and retry. Signed upload URLs should be long enough for the expected upload time (2 hours is standard).
- **User uploads a valid image extension but the file is actually a different type** — magic byte validation catches this server-side. Reject with a clear error: "The file doesn't match its extension."
- **Storage is full or provider is down** — handle storage API errors gracefully. Show "Upload failed, please try again later." Never show the raw storage error to the user.
- **Multiple users upload to the same entity simultaneously** — each upload gets a unique filename (UUID-based), so there are no collisions. The database records track all files per entity.
- **User uploads an extremely high resolution image** — processing (resize, thumbnail generation) handles this, but Sharp needs enough memory to process the image. Set a maximum dimension limit (e.g., 10000x10000px) and reject images beyond it before processing.
- **HEIC/HEIF uploads from iPhones** — iPhones take photos in HEIC format by default. Sharp supports HEIC conversion to WebP. Accept HEIC in your file type allowlist and convert during processing.
- **Animated GIFs** — converting to WebP preserves animation. If you generate thumbnails from GIFs, extract the first frame as a still thumbnail, not an animated one (saves bandwidth).
- **SVG uploads** — SVGs can contain embedded JavaScript (XSS vector). If you accept SVG uploads, sanitize them by stripping script tags and event handlers, or convert them to PNG/WebP during processing. Never serve user-uploaded SVGs raw.
- **File referenced in database but missing from storage** — storage deletion succeeded but database update failed, or vice versa. The cleanup job should detect orphaned database records (no file in storage) and orphaned storage files (no database record) and reconcile both.

## Implementation Checklist

### Storage
- [ ] Public and private buckets created with appropriate access policies
- [ ] Path structure includes workspace/user ID for access control
- [ ] Storage-level access policies enforce workspace membership
- [ ] Per-bucket file size limits configured

### Upload Flow
- [ ] Signed upload URLs generated server-side after auth + validation
- [ ] Files uploaded directly from browser to storage (not through app server)
- [ ] Client-side validation (file type, size, dimensions) before upload starts
- [ ] Server-side validation (magic bytes, size) after upload completes
- [ ] Progress bar shown for uploads over ~500KB
- [ ] Upload cancellation supported
- [ ] Filenames generated server-side (UUID), original name stored as metadata

### Image Processing
- [ ] Background job processes images after upload (not blocking the upload)
- [ ] Thumbnails generated at 3 sizes (150x150, 600x600, original)
- [ ] All images converted to WebP for serving
- [ ] EXIF metadata stripped from served images
- [ ] Original file preserved for download/export
- [ ] HEIC/HEIF accepted and converted
- [ ] SVG uploads sanitized or converted to raster format
- [ ] Maximum dimension limit enforced before processing

### Serving
- [ ] Public files served via CDN with long cache headers
- [ ] Private files served via signed download URLs (1 hour expiry)
- [ ] New path on update (avoid CDN cache staleness)
- [ ] `next/image` used for all image rendering

### Cleanup
- [ ] Temporary uploads flagged with expiry date
- [ ] Scheduled cleanup job deletes expired temporary files
- [ ] Entity deletion cascades to file deletion (via background job)
- [ ] Orphan detection for mismatched database records and storage files

## Common Mistakes

- Uploading files through the application server (memory pressure, timeout on large files, unnecessary bandwidth)
- Trusting client-reported file types (always verify with magic bytes server-side)
- Using the client-provided filename for storage (path traversal risk, naming collisions)
- Overwriting files instead of uploading to a new path (CDN cache serves stale content)
- Processing images in the upload request (blocks the response, times out on large images)
- Not stripping EXIF metadata (leaks user GPS coordinates and device info)
- Serving user-uploaded SVGs without sanitization (XSS via embedded scripts)
- Storing file metadata only in storage and not in the database (can't query, can't enforce access control, can't track ownership)
- Deleting files from storage synchronously during entity deletion (blocks the UI, fails silently if storage API is slow)
- Not handling HEIC uploads (iPhone users get a confusing error)
- No cleanup for temporary uploads (storage fills up with abandoned files)
