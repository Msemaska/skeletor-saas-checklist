# Audit Logging

---

## What It Is

An append-only, immutable record of meaningful business events inside your application — who did what to which resource, when, and from where. This is not the same as operational logging (Pino/Sentry, covered in `phase-3-core-product/12-error-handling-logging.md`). Operational logs serve engineers debugging failures. The audit trail serves compliance, security investigations, customer trust, and admin accountability. The two systems have different consumers, different retention rules, different access controls, and must never be conflated.

In B2B SaaS, enterprise buyers require an audit trail before signing. A complete, customer-accessible audit log shortens procurement cycles and satisfies the security review questions that otherwise take months to answer.

## What Counts as an Auditable Event

Not every database write belongs in the audit log. Log events that answer the question "who did this, and should they have?" — actions with security, compliance, or accountability implications.

**Always audit:**
- Authentication — sign-in (success and failure), sign-out, password change, MFA enable/disable
- Authorization changes — role assignments, permission changes, workspace member invitations and removals
- Sensitive data access — data exports, bulk downloads, any access to PII in a compliance context
- Billing actions — subscription changes, plan upgrades/downgrades, manual billing overrides
- Admin operations — impersonation start/end, manual user role changes, account suspension/reinstatement
- Workspace/account lifecycle — workspace creation, deletion, rename, ownership transfer
- Configuration changes — feature flag toggles, integration connection/disconnection, API key creation/revocation

**Do not audit:**
- Read operations on ordinary data (viewing a project, loading a list) — these create noise without security value
- System events that have no actor (scheduled jobs, automated syncs) — these belong in operational logs, not the audit trail
- Validation failures, user input errors — these are application errors, not auditable events

The rule: if a security team, compliance auditor, or the affected user would need to know it happened, audit it. If only an engineer debugging a failure would care, it belongs in operational logs.

## The Event Schema

Every audit event in the `audit_events` table stores the same core fields:

- `id` — UUID, primary key
- `workspace_id` — which tenant the event belongs to (foreign key to workspaces; nullable for system-level events with no workspace context)
- `actor_id` — UUID of the user who performed the action (foreign key to profiles; nullable for automated system actions)
- `actor_type` — enum: `user`, `admin`, `system` — distinguishes human actions from automated ones
- `impersonated_by` — UUID of the admin whose session was used when `actor_type` is `admin` performing impersonation (see Impersonation section)
- `action` — string in `resource.verb` format: `member.invited`, `subscription.upgraded`, `api_key.revoked`, `user.impersonated`. This is the machine-readable event identifier.
- `target_type` — the entity type affected: `workspace`, `user`, `subscription`, `project`, `api_key`, etc.
- `target_id` — UUID of the specific record affected
- `before` — `jsonb`, the relevant fields before the change (null for create events)
- `after` — `jsonb`, the relevant fields after the change (null for delete events)
- `ip_address` — the IP the action came from (for user-facing actions)
- `user_agent` — the browser/client string (for user-facing actions)
- `created_at` — `timestamptz`, UTC, set by the database, not the application

The `before`/`after` fields are not full row dumps. Store only the fields that changed, and only the fields relevant to the security/compliance question the event answers. Avoid storing raw user content — audit logs record what changed structurally, not the full payload.

The `action` naming convention (`resource.verb`) makes the log human-readable and easily filterable. Define a finite set of valid action strings and treat them as an enum in code — don't let freeform strings creep in.

## Append-Only and Immutability

The `audit_events` table is append-only. No `UPDATE` or `DELETE` ever touches it from application code. This is a hard rule with no exceptions.

Enforce it at the database level, not just by convention:

- The application database role that handles normal read/write traffic has `INSERT` and `SELECT` on `audit_events`, but not `UPDATE` or `DELETE`
- A separate, restricted role (used only for admin tooling and direct database access) can read but still cannot delete
- Revoke all `UPDATE` and `DELETE` privileges on the table from every role except the database owner, and the database owner does not touch this table from running application code

This means audit events cannot be corrected if written incorrectly. The solution is to write a new compensating event rather than editing the old one. This is intentional — immutability is the point.

For environments requiring tamper-evidence beyond permission-based immutability (SOC 2 Type II, regulated industries), add hash chaining: each row stores a hash of the previous row's hash plus its own content. A broken chain proves tampering. This is opt-in complexity — start without it and add when a compliance requirement demands it.

## Where Audit Writes Happen

Audit events are written from the service layer (`phase-3-core-product/04-data-access.md`), not from components, hooks, or API route handlers. This is the same constraint as all database access.

For actions where the audit write and the underlying action must both succeed or both fail, perform both inside the same database transaction. If the membership removal succeeds but the audit write fails, the audit trail is incomplete and you have no record of the event. Wrap them in a transaction.

For actions where eventual consistency is acceptable (read-heavy operations, low-risk events), write the audit event after the primary action succeeds. If the audit write fails, log the failure to the operational log (Pino) and alert — do not silently swallow the miss.

Never write audit events from database triggers as the primary mechanism. Triggers fire outside the service layer, bypass the actor context (they have no knowledge of which user triggered the action), and are invisible to the application developer tracing a request. Write from service functions where actor identity is explicit and the intent is clear. Triggers can be used as a safety net on the most sensitive tables (subscription changes, role changes), but they cannot replace service-layer writes because they lack actor context.

## Actor Context and Impersonation

Every audit event must attribute the action to the real human who caused it. This requires actor context to be available in the service layer.

Pass the session's user ID into every service function that performs an auditable action. The service function is responsible for constructing the audit event with the correct actor. Do not derive the actor from the database row being modified — derive it from the authenticated session.

Impersonation (`phase-15-admin-operations/01-admin-dashboard.md`) requires special handling. When an admin impersonates a user, the application session technically belongs to the target user, but the real actor is the admin. Audit events written during an impersonation session must set:

- `actor_id` — the impersonated user's ID (whose account context is being used)
- `actor_type` — `admin`
- `impersonated_by` — the real admin's user ID

This makes it unambiguous in the audit trail that the action was performed by an admin acting as someone else, not by the user themselves. If impersonation is not attributed correctly, a user's audit log will show actions they did not take, which is both a trust and a legal liability problem.

The impersonation session must carry the real admin's ID alongside the impersonated user's context so the service layer always has both available.

## Retention and Access

Audit events are never deleted by application code. The default retention policy is indefinite — they are the legal record of what happened in your product.

GDPR creates a tension: it requires personal data not be stored longer than necessary. Resolve it by anonymising rather than deleting: if a user exercises their right to erasure (`phase-8-business-wrapper/06-gdpr-self-service.md`), nullify the `actor_id` in their audit events and replace their identifying information with a tombstone marker. The events remain for compliance purposes; the personal data is removed. The `created_at`, `action`, `target_type`, `target_id`, `before`, and `after` fields are retained — only the personal identifiers are cleared.

SOC 2 expects at least 12 months of accessible audit logs. Store at minimum the last 12 months in the hot table. Archive older records to cheaper storage (Neon's cold tier or Cloudflare R2 as JSON) rather than deleting them.

Access control on the audit table follows a strict hierarchy:

- **Workspace members** — can read audit events scoped to their own workspace (`workspace_id` matches). RLS enforces this automatically.
- **Workspace admins and owners** — same access as above. No elevated read access beyond workspace scope is granted to workspace-level roles.
- **System-level admins** — can read all audit events across all workspaces. This is done in system context, bypassing RLS (`phase-3-core-product/01-database-schema.md`).
- **No one** — can update or delete audit events.

The `audit_events` table has RLS enabled. The SELECT policy checks that the authenticated user is a member of the event's workspace. System context bypasses this for admin tooling.

## Customer-Facing Audit Log

Enterprise customers expect to see their workspace's audit trail in the product UI, not just have it available to your support team.

Expose a read-only audit log page in the workspace settings, accessible to workspace owners and admins. It shows:

- A chronological list of events, newest first
- Filters by actor, action category, and date range
- A human-readable label for each event (`action` values are translated to prose: `member.invited` → "Invited [name] as member")
- The actor's name, the timestamp in the viewer's timezone, and the target resource name where applicable

Do not expose `before`/`after` fields to all workspace members in the UI — restrict raw diffs to workspace owners and admins. Workspace members can see that an event occurred and who caused it, but not necessarily the full detail of what changed.

Pagination is mandatory. An active workspace can accumulate thousands of events. Default page size of 50, keyset-paginated on `created_at`. Never load the full table for display.

The customer-facing audit log is the workspace's self-service compliance artefact. B2B customers use it to satisfy their own internal audit requirements and to investigate activity in their account. Build it before launch — it is a trust signal, not a post-launch addition.

Team-level access control for who can see the audit log is governed by the RBAC model in `phase-3-core-product/13-team-rbac.md`.

## Security Considerations

**Immutability as a security control** — permission-based immutability (no `UPDATE`/`DELETE` granted to the application role) is the minimum. If the application role is ever compromised, an attacker could insert false events but cannot erase real ones. Document this guarantee clearly: the audit log is append-only, not cryptographically sealed by default. Add hash chaining when SOC 2 Type II or a specific compliance requirement demands it.

**No secrets or sensitive values in log fields** — `before`/`after` fields must never contain passwords, API key values, session tokens, payment card data, or encryption keys. Store field names and redacted indicators, not values. A `before` snapshot of a password change should record `{ "password_hash": "[REDACTED]" }`, not the hash itself. Define an explicit list of fields that are always redacted before serialisation.

**No PII bloat** — audit events should reference user IDs, not replicate PII. Do not store names, emails, or addresses in the `before`/`after` jsonb fields unless the change to that data is itself the auditable event (e.g., a user updating their own email). The actor's name and email for display in the UI are joined at read time from the profiles table, not stored in the audit event.

**Read access scoped by workspace** — a workspace member must never be able to read audit events from another workspace, even via direct API calls. RLS and server-side workspace scope checks both enforce this. Never expose an endpoint that fetches audit events by ID without also verifying the workspace matches the caller's membership.

**Admin audit trail integrity** — admin-level audit events (impersonation, billing overrides, manual role changes) are the most sensitive. The system-level admin role that can read cross-workspace events should require 2FA and have its own sign-in sessions monitored. Any access to the admin audit trail viewer should itself produce an operational log entry.

## Deliverables / Implementation Checklist

### Schema
- [ ] `audit_events` table created with all core fields (id, workspace_id, actor_id, actor_type, impersonated_by, action, target_type, target_id, before, after, ip_address, user_agent, created_at)
- [ ] `actor_type` enum defined (user, admin, system)
- [ ] Index on `(workspace_id, created_at DESC)` for workspace-scoped queries
- [ ] Index on `(actor_id, created_at DESC)` for per-user audit queries
- [ ] No `updated_at` column — audit events have no update lifecycle
- [ ] RLS enabled with SELECT policy scoped to workspace membership
- [ ] `UPDATE` and `DELETE` privileges revoked from the application database role

### Service Layer
- [ ] `createAuditEvent()` utility in the services layer (`services/audit.ts`)
- [ ] Actor context (user ID, actor type, impersonated_by) passed through service function signatures for all auditable operations
- [ ] Audit write co-transactional with the primary action for create/delete/permission-change events
- [ ] Impersonation session carries real admin ID so service layer always has both actor and impersonator available
- [ ] Sensitive fields redacted before serialising `before`/`after` (passwords, tokens, keys)
- [ ] Finite set of valid `action` strings defined as constants

### Customer-Facing UI
- [ ] Audit log page in workspace settings (owners and admins only)
- [ ] Keyset-paginated display (50 per page, newest first)
- [ ] Filters by actor, action category, date range
- [ ] Human-readable labels for all `action` values
- [ ] `before`/`after` detail restricted to owners and admins
- [ ] Timestamps displayed in viewer's local timezone

### Admin Tooling
- [ ] Admin audit trail view in admin dashboard (`phase-15-admin-operations/01-admin-dashboard.md`), searchable across all workspaces
- [ ] Impersonation start/end events written with `impersonated_by` populated
- [ ] Manual billing override events include reason field (stored in `after` as metadata)

### Retention
- [ ] GDPR erasure flow anonymises `actor_id` and PII fields rather than deleting rows (`phase-8-business-wrapper/06-gdpr-self-service.md`)
- [ ] Archive strategy defined for events older than 12 months (cold storage or Neon archive)

## What to Test

- Audit event is written when a member is invited, role is changed, or removed
- Audit event is written on subscription upgrade, downgrade, and cancellation
- Audit event is written on impersonation start and end, with `impersonated_by` set to the real admin's ID — not the target user's ID
- Audit event is written on API key creation and revocation
- Audit event is written on workspace creation and deletion
- No audit event records passwords, tokens, or API key values in `before`/`after`
- RLS prevents a user in workspace A from reading audit events belonging to workspace B
- `UPDATE` and `DELETE` on `audit_events` fail for the application database role
- The action that produces an audit event and the audit write are both rolled back together if the transaction fails
- Customer-facing audit log page is inaccessible to workspace members (visible only to owners and admins)
- Audit log page is paginated — does not load all records at once
- GDPR erasure nullifies `actor_id` on audit events without deleting the rows

## Edge Cases

- **Action succeeds but audit write fails** — for co-transactional writes, both roll back and the action is retried; for eventually-consistent writes, log the miss to the operational log (Pino) and alert so it can be investigated — a silent miss creates an incomplete trail.
- **Audit write succeeds but action fails** — within a transaction, both roll back together; outside a transaction, this cannot happen because the audit write follows the confirmed success of the primary action.
- **Admin impersonation attributed to wrong actor** — the impersonation session must carry the real admin's ID and the service layer must use `impersonated_by`, not `actor_id`, to identify the responsible human; events attributed to the impersonated user are a trust and legal liability problem.
- **Bulk operations producing thousands of events** — one API call that modifies 500 records writes 500 audit events in a single transaction; this is correct behaviour but stresses the table on write; batch audit inserts (multi-row `INSERT`) rather than looping individual inserts to keep transaction time acceptable.
- **User data deletion under GDPR vs audit retention** — never delete audit rows to satisfy erasure; instead, run an anonymisation pass that nullifies `actor_id`, `ip_address`, `user_agent`, and any PII present in `before`/`after` fields, then write a tombstone event recording when and why the anonymisation occurred.
- **Clock or timezone inconsistency** — `created_at` is set by the database server (`DEFAULT now()`), not the application; this ensures all timestamps are consistent UTC regardless of the server or timezone the application runs in; never accept a client-supplied timestamp for audit events.
- **Workspace deleted, audit events orphaned** — use `ON DELETE SET NULL` on the `workspace_id` foreign key, not `ON DELETE CASCADE`; deleting a workspace must not erase its audit history.
- **Actor deleted (user account closed)** — use `ON DELETE SET NULL` on the `actor_id` foreign key; audit events persist after the actor's account is removed, with `actor_id` nulled and the display layer showing "Deleted user".

## Common Mistakes

- Conflating audit events with operational logs — they are different systems with different consumers, retention rules, and access controls; do not write audit events to Pino or Sentry
- Writing audit events from database triggers instead of the service layer — triggers have no actor context; they cannot distinguish an admin action from a system migration
- Attributing impersonated actions to the target user instead of the admin — the audit trail must reflect the real human responsible
- Including passwords, tokens, or secrets in `before`/`after` snapshots
- Storing full row dumps in `before`/`after` — store only the changed fields; full dumps bloat the table and risk including sensitive values
- Deleting audit events to satisfy GDPR erasure requests — anonymise, do not delete
- No RLS on the audit table — one missing policy lets any authenticated user read the full cross-workspace audit history
- Using `ON DELETE CASCADE` on `workspace_id` — a workspace deletion must not erase its audit history
- Relying on application-layer convention to prevent writes, without revoking database privileges — convention breaks; privileges do not
- Freeform action strings instead of a defined constant set — makes the log unsearchable and the customer-facing UI impossible to translate consistently
- No index on `(workspace_id, created_at DESC)` — the most common query pattern scans the full table without it
- No pagination on the customer-facing audit log — an active workspace can have tens of thousands of events and will crash the page
- Omitting the customer-facing audit log UI until after launch — enterprise buyers ask for it before signing; it must be present at launch
