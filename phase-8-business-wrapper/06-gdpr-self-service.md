# GDPR Self-Service

---

## What It Is

The self-service mechanism that lets users exercise their GDPR/CCPA rights directly from within the product — specifically the right to access (download all their data) and the right to erasure (delete their account and personal data). These rights are the promises made in `phase-8-business-wrapper/05-legal-pages.md`; this file is the implementation of those promises.

**Priority: `[3]` — after launch.** Most products launch with manual, email-based handling and add the self-service UI later. That is a legitimate starting point. The obligation under GDPR exists from day one (respond within one month), but the mechanism for fulfilling it can be manual at first. Build self-service when your user volume makes manual handling impractical or when your privacy policy explicitly promises it.

The recommended tool stack is **custom: Next.js Server Actions + Trigger.dev background jobs + Cloudflare R2 + Resend**.

The admin-initiated equivalent — an operator exporting a user's data on their behalf — is covered in `phase-15-admin-operations/01-admin-dashboard.md`. This file is the user-initiated, self-service version only.

---

## Right to Access / Data Export

Users can request a full export of all personal data held about them. The export must be machine-readable (JSON is the correct format — it is structured, commonly used, and satisfies both GDPR Article 20 portability and CCPA access requirements). Do not provide CSV-only; it cannot represent nested data cleanly.

**What the export bundle contains:**

- Profile: name, email, account metadata
- All entities the user created within their workspace(s)
- Workspace memberships and roles
- Billing history: invoice dates, amounts, plan changes (the data, not payment instrument details)
- Consent records: when cookie consent was given and what was accepted
- Activity that is personal to them and is within the retention window

**What the export bundle never contains:**

- Password hashes or session tokens
- Other users' data (even in shared workspaces — each user receives only their own data; shared workspace content is included only in the form of their own contributions)
- Internal system fields (row IDs that have no meaning outside the database, internal flags)
- Data belonging to a separate user who happens to share a workspace

The export is generated as a background job (see `phase-3-core-product/10-background-jobs.md`). Generation is not synchronous — it can take seconds to minutes depending on data volume. The user triggers it from account settings, receives an acknowledgement immediately, and gets an email with a signed download link when the bundle is ready.

The bundle is written to a private Cloudflare R2 bucket (see `phase-7-infrastructure/04-object-storage.md`) under a path scoped to the user's ID. The download link is a short-lived signed URL with a 24-hour expiry. Once the link expires, the user must request a new export. The bundle itself is deleted from R2 after 48 hours regardless of whether it was downloaded, to avoid retaining a packaged copy of all user data indefinitely.

Limit export requests to one per user per 24-hour window to prevent abuse and runaway background job queues.

---

## Right to Erasure / Account Deletion

The deletion flow is user-initiated from the Danger Zone section of account settings (see `phase-5-ui-ux/09-account-settings.md`). Deletion must be as easy to initiate as signup was — no "email us to delete" friction, no dark patterns, no hidden flows.

The full mechanics of account deletion — the grace period, the soft-delete state, the cancellation of the Stripe subscription at period end, and the final hard-delete job — are defined in `phase-3-core-product/03-auth.md` under the Account Deletion section. Do not restate them here. The one addition specific to GDPR self-service: when the hard-delete job runs, it must also confirm that no pending export job exists for that user and either complete it or discard it before proceeding.

The grace period (typically 14–30 days) serves a dual purpose: it gives the user a window to reverse an accidental deletion, and it gives time for any pending export to complete and be delivered before the underlying data is destroyed.

---

## What Is Erased vs. What Is Legally Retained

Not all data can be deleted on request. GDPR Article 17 explicitly permits retention where a legal obligation requires it.

**Erased (hard-deleted):**

- Profile data: name, display name, avatar
- Authentication credentials (email is anonymised, not deleted, to prevent re-registration confusion — see Edge Cases)
- All content and entities the user created
- Session records and tokens
- Analytics and behaviour data tied to the user

**Retained (anonymised or untouched):**

- Financial/billing records: invoices, payment events, and subscription history must be retained for 7 years to satisfy tax and accounting obligations across most jurisdictions. Stripe retains its own copy regardless; your database copy is the source of truth for your accounting. Anonymise by removing the user's name and email from these records while preserving the amounts, dates, and plan identifiers.
- Audit logs: immutable records in the audit log (`phase-3-core-product/14-audit-logging.md`) are retained indefinitely. They are anonymised by replacing the user's name and email with a tombstone token (e.g. `[deleted-user:<hashed-id>]`) so the action trail remains legible without being attributable to a named individual.
- Legal hold data: if the user is subject to a legal hold (active dispute, fraud investigation, regulatory inquiry), erasure is suspended until the hold is lifted. This is rare but must be handled — see Edge Cases.

Anonymisation, not hard-deletion, is the correct approach for retained records. Truly anonymised data is outside GDPR's scope — but only if the anonymisation is genuine (no feasible re-identification path). Pseudonymisation (hashed IDs that can be reversed) is not anonymisation and does not satisfy the erasure obligation.

Backup copies of deleted data that exist in database snapshots are put "beyond use" when live deletion occurs — they must not be restored for any purpose after the deletion date. They are naturally overwritten as backups rotate (typically within 30–90 days). Document this in your privacy policy.

---

## Identity Verification Before Export or Deletion

Both export and deletion are irreversible or sensitive enough to require confirmation that the person making the request is the authenticated account owner.

For export: the user must be signed in. No additional verification step is required because the export is delivered to the verified email on file — an attacker who doesn't control the email cannot intercept the download link.

For deletion: require the user to re-authenticate (confirm their password or re-complete their auth factor) immediately before confirming deletion. This prevents a session hijack from silently deleting an account. The re-authentication prompt should appear in the Danger Zone, not just on the final confirmation. Better Auth supports re-authentication flows; use it.

OAuth-only users (no password) must confirm deletion by clicking a confirmation link sent to their registered email address before the grace period begins.

---

## Async Generation and Delivery

The export background job follows the standard background job pattern from `phase-3-core-product/10-background-jobs.md` using Trigger.dev.

The job:
1. Queries all tables that contain the user's personal data
2. Assembles the JSON bundle
3. Uploads it to R2 under `exports/{userId}/{timestamp}.json`
4. Generates a signed URL valid for 24 hours
5. Sends a transactional email via Resend (see `phase-8-business-wrapper/01-transactional-emails.md`) containing the signed URL and an expiry notice
6. Marks the export request as completed in the database

The database keeps a record of export requests: user ID, requested-at timestamp, completed-at timestamp, expiry timestamp, and status. This serves as the audit record of every export that was requested and fulfilled.

If the job fails, it retries with standard back-off. After exhausting retries, it sends a failure notification email to the user and logs the error. Do not silently fail — the user is waiting.

---

## User-Facing Entry Point

Both actions live in the account settings page, Danger Zone section (`phase-5-ui-ux/09-account-settings.md`).

The export entry point:
- Button labelled "Download my data"
- On click: triggers the background job, shows an in-page confirmation ("We're preparing your export. You'll receive an email with a download link within a few minutes.")
- Disabled with a tooltip if an export was already requested in the last 24 hours

The deletion entry point:
- Button labelled "Delete my account"
- Opens a confirmation modal that clearly states: what will be deleted, the grace period duration, that the Stripe subscription will be cancelled, and that the action can be reversed during the grace period
- Requires re-authentication before the modal's final confirm button is active
- On confirm: initiates the grace period, sends a confirmation email, signs the user out

Neither action should be buried, require a support ticket, or involve dark-pattern friction (no "are you REALLY sure?" multi-step gauntlets beyond one re-auth and one confirmation).

---

## Security Considerations

**Export belongs only to the requesting user.** The Server Action that triggers the export job must extract the user ID from the server-side session — never from a request parameter. An attacker must not be able to pass another user's ID to trigger or download their export.

**Signed URLs are single-recipient.** The signed R2 URL is sent only to the verified email address on file. If a user forwards the email and the link is accessed by a third party before expiry, that third party can download the bundle. Mitigate with a 24-hour expiry window (short enough to reduce risk, long enough for practical use) and a warning in the email that the link should not be forwarded.

**Export scope in teams and organisations.** When a user belongs to a workspace with other members, the export must contain only that user's personal data — not the full workspace contents. Shared content (documents, projects, records created collaboratively) is tricky: include the user's own contributions but not content where other users are the primary data controller. If in doubt, include a reference (e.g. "member of workspace: Acme Corp") rather than the full content.

**Re-authentication prevents session hijack deletions.** A valid session cookie found by an attacker (XSS, leaked token) should not be sufficient to delete the account. The mandatory re-auth step before deletion is the critical guard.

**Rate limiting on export requests.** Without a limit, a malicious or confused user can flood the job queue. One export per user per 24 hours is sufficient.

**Deletion during legal hold.** If you have a legal obligation to retain the account (active dispute, fraud investigation), the deletion flow must detect this and surface a message: "Your account cannot be deleted at this time. Please contact support." Do not silently swallow the request.

---

## Deliverables / Implementation Checklist

### Export

- [ ] Export request Server Action: validates session, checks 24-hour rate limit, enqueues Trigger.dev job, returns immediate acknowledgement
- [ ] Trigger.dev export job: queries all personal data tables, assembles JSON bundle, uploads to R2, generates signed URL, sends Resend email
- [ ] R2 bucket configured: private, scoped paths per user, 48-hour lifecycle rule auto-deletes bundles
- [ ] Export request table in Neon: tracks request status, timestamps, expiry
- [ ] Transactional email: "Your data export is ready" with signed URL and 24-hour expiry notice
- [ ] Transactional email: "Your export failed" with instructions to retry
- [ ] Export disabled state in UI when a request was already made in the last 24 hours
- [ ] Export action logged in audit trail

### Deletion

- [ ] Deletion re-authentication flow (password confirm or email-link confirm for OAuth-only users)
- [ ] Deletion confirmation modal: lists what is deleted, grace period, subscription cancellation
- [ ] Grace period and hard-delete job wired up per `phase-3-core-product/03-auth.md`
- [ ] Hard-delete job checks for pending export and discards it before proceeding
- [ ] Anonymisation of retained records: billing records and audit log rows
- [ ] Legal hold detection: block deletion and surface support message if hold is active
- [ ] Confirmation email on deletion initiation (with grace period end date and cancellation link)
- [ ] Deletion action logged in audit trail

### UI

- [ ] "Download my data" button in account settings Danger Zone
- [ ] "Delete my account" button in account settings Danger Zone
- [ ] Both entry points follow the UI spec in `phase-5-ui-ux/09-account-settings.md`

---

## What to Test

- Export Server Action rejects unauthenticated requests
- Export Server Action uses session user ID — passing a different user ID in the request body has no effect
- Export job produces valid JSON containing all expected data categories
- Export job excludes password hashes, session tokens, and other users' data
- Export bundle is accessible via the signed URL before expiry
- Export bundle returns 403/404 after the signed URL expires
- R2 lifecycle rule deletes the bundle after 48 hours
- Second export request within 24 hours is rejected with a clear message
- Export job failure sends the failure email and logs the error
- Deletion re-authentication blocks submission without valid credentials
- Deletion confirmation modal clearly states the grace period and subscription impact
- Deletion during grace period can be successfully cancelled (per `phase-3-core-product/03-auth.md`)
- Hard-delete job anonymises billing records (name and email removed, amounts and dates retained)
- Hard-delete job anonymises audit log rows (user identifier replaced with tombstone token)
- Hard-delete job does not execute if a legal hold flag is active
- OAuth-only users receive and must click the email confirmation link before deletion begins
- Export request for a deleted account returns a graceful error (not a 500)
- Team workspace export contains only the requesting user's contributions, not other members' data

---

## Edge Cases

**Delete request while subscription is active.** Cancel the Stripe subscription at period end (not immediately) — the user already paid for the current period. The hard-delete job runs after both the grace period and the subscription end date have passed, whichever is later.

**Export requested, then account deleted before the download link is used.** The export bundle and its signed URL remain in R2 until the 48-hour bundle TTL expires. The user should still be able to download it during that window even though their account is in the grace period or deleted, because the signed URL is pre-authenticated at generation time.

**User in a team or organisation with shared data.** Include only the user's own profile, contributions, and membership records. Do not include other members' data or workspace content they did not author. If workspace content ownership is ambiguous (e.g. collaboratively edited documents), include a reference to the workspace rather than the full content.

**Legal hold blocking erasure.** Surface a message ("your account cannot be deleted at this time — contact support") and do not begin the grace period. Log the blocked attempt in the audit trail. Once the hold is lifted, allow the user to re-request deletion normally.

**Re-signup with the same email after erasure.** The email address is anonymised (e.g. replaced with a hashed tombstone) rather than fully deleted, so a uniqueness constraint prevents the original email from being silently reused by a new account without the user going through a fresh sign-up. The original account's data is not restored on re-signup.

**Export link forwarded to someone else.** The link is a signed URL with no additional identity check at download time — anyone with the link can download the bundle before expiry. Keep expiry at 24 hours and include a "do not forward this link" warning in the email. There is no way to make a signed URL user-specific without adding a separate auth layer.

**User requests export multiple times in quick succession.** Rate-limited to one export per 24-hour window. Return a clear message with the earliest time they can request again.

**Export of a very large account.** For accounts with unusually large data volumes, the Trigger.dev job must handle pagination internally when querying tables. Set a generous job timeout (10–15 minutes). If generation exceeds the timeout, send the failure email with a note to contact support for a manual export.

---

## Common Mistakes

- Delivering the export synchronously in the HTTP response — large exports will time out and leave the user with no data and no feedback
- Including other workspace members' data in a team account export
- Setting the signed URL expiry too long (days or weeks) — increases the window for interception
- Anonymising records by setting the email field to `null` — null is still personally identifiable if combined with other fields; replace with a tombstone token that cannot be reversed
- Starting the grace period before re-authentication is confirmed — a session hijack can begin the deletion timer silently
- Hard-deleting billing records — financial records carry a 7-year legal retention obligation; anonymise them instead
- Not logging export and deletion actions in the audit trail — these are the records you need if a user disputes what happened to their data
- Silently failing the export job without notifying the user — they are waiting for an email that never arrives
- Treating pseudonymisation as full anonymisation — hashed IDs that can be reversed do not satisfy the erasure obligation for retained records
- Building the self-service UI before the underlying deletion and anonymisation logic is solid — the UI is the easy part; test the data destruction thoroughly first
