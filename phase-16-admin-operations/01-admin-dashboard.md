# Admin Dashboard & Operations Tools

---

## What It Is

Internal tooling for managing users, troubleshooting issues, and handling billing exceptions. This is not user-facing — it's the tools you use to run the business day-to-day.

## Admin Dashboard

A protected section of the app accessible only to admin users.

### What It Shows

| Section | Content |
|---|---|
| User overview | Total users, new sign-ups today/this week, active users |
| Workspace list | Searchable list of all workspaces with: name, plan, member count, created date |
| User list | Searchable list of all users with: name, email, workspace(s), role, sign-up date, last active |
| Revenue summary | Current MRR, new subscriptions this month, cancellations this month (pulled from Stripe Dashboard — link, don't rebuild) |
| Recent errors | Link to Sentry dashboard for the latest errors |
| System health | Link to UptimeRobot and health check status |

### Design Decisions

**Design decisions:**
- The admin dashboard is a route group in the app: /app/admin/*
- Protected by role check — only users with the "admin" role at the system level (not workspace-level admin) can access it
- Built with the same component library as the rest of the app (Shadcn + Tailwind)
- Keep it simple — tables with search and filters. No custom charts or analytics. Link to Stripe Dashboard and PostHog for detailed analytics.
- The admin dashboard is a pre-launch item — build it before launch so you can manage users from day one

## User Impersonation

Sign in as a user to see exactly what they see — for debugging support issues.

### Design Decisions

**Design decisions:**
- Admin clicks "Impersonate" on a user row in the admin dashboard
- The app creates a session that acts as the target user — all queries run through their RLS context, the UI renders as they would see it
- A visible banner at the top of the page shows "You are impersonating [user name] — Exit" so you never forget you're impersonating
- Impersonation is logged in the audit trail: who impersonated whom, when, for how long
- Impersonation never reveals the user's password — it creates a temporary session using system context
- Only system-level admins can impersonate. Workspace-level admins cannot.
- Exit impersonation returns to the admin dashboard

## Manual Billing Overrides

Handle billing exceptions that the automated system doesn't cover.

### Design Decisions

**Design decisions:**
- Use the Stripe Dashboard directly for billing overrides — don't build custom billing admin tooling
- Common overrides: extend a trial (update trial_end on the subscription), apply a credit (create a credit note), issue a refund (process refund on the payment), change a plan manually (update subscription), cancel and re-create a subscription
- For each override, update both Stripe AND your database's plan/status fields via the webhook flow — don't update the database directly, let the webhook handle it
- Log every manual billing action in the audit trail with the reason

## User Data Export

Allow admin to export a user's data for GDPR data portability requests.

### Design Decisions

**Design decisions:**
- Build a Server Action that generates a JSON export of all data belonging to a specific user/workspace
- The export includes: profile data, workspace data, all entities they created, membership records
- The export excludes: password hashes, session tokens, internal system fields
- The export is generated on demand via the admin dashboard — click "Export Data" on a user row, download the JSON file
- This is the same data that would be exported if the user requests it from their own settings (GDPR data portability — phase-3-core-product/03-auth.md, Account Deletion section)
- Format: JSON (machine-readable, covers the GDPR requirement)

## Audit Trail

Every admin action is logged.

### Design Decisions

**Design decisions:**
- The audit trail is the same audit log table described in phase-3-core-product/01-database-schema.md
- Log: impersonation start/end, billing overrides (with reason), data exports, manual user actions (role changes, account suspensions)
- Audit logs are immutable — no editing or deleting
- Viewable from the admin dashboard: searchable by user, action type, and date range
- Retain audit logs indefinitely — they're your legal record of what happened

## What to Test

### Access Control
- Admin routes return 403 for authenticated non-admin users
- Admin routes redirect to sign-in for unauthenticated users
- Admin role is enforced server-side — cannot be bypassed via client manipulation

### User & Workspace Management
- User search returns matches by email, name, and ID
- User detail page shows correct plan, usage counts, and workspace memberships
- Workspace list shows correct member count, plan, and billing status
- Manually disabling a user blocks their subsequent sign-in

### Impersonation
- Impersonation creates a session scoped to the target user
- Impersonation banner is visible throughout the session
- Impersonated session has the target user's permissions — not admin permissions
- Impersonated user cannot access admin dashboard routes
- Impersonation start is logged in the audit trail with admin ID, target user ID, and timestamp
- Actions taken while impersonating are attributed to the target user in audit logs
- Exit impersonation destroys the impersonated session and returns to admin dashboard

### Data Export
- Export generates complete JSON including profile, workspace data, and billing history
- Export excludes password hashes, session tokens, and internal system fields
- Export action is logged in the audit trail

### Billing Overrides
- Manual subscription change via Stripe Dashboard triggers webhook and updates workspace plan in database
- Manual refund via Stripe Dashboard does not break subscription state

## Edge Cases

- **Admin impersonates a user and performs a destructive action** — impersonation is read-only (view only, no mutations). The banner must be impossible to miss.
- **User requests data export while their account is being deleted** — process the export first, then delete. The deletion grace period (phase-3-core-product/03-auth.md) gives time for this.
- **Billing override doesn't sync** — if an admin changes something in Stripe Dashboard but the webhook doesn't fire (endpoint down), the database and Stripe are out of sync. The health check and webhook monitoring should catch this. Manual fix: trigger a subscription sync from the admin dashboard.
- **Admin account is compromised** — admin accounts have system-level access. Enforce 2FA on all admin accounts. Log every admin action. If compromised, revoke the admin's sessions immediately and review the audit trail for unauthorized actions.

## Implementation Checklist

- [ ] Admin route group created (/app/admin/*)
- [ ] System-level admin role defined (separate from workspace admin)
- [ ] Admin dashboard: user list, workspace list, revenue summary links
- [ ] User impersonation: session creation, visible banner, exit button
- [ ] Impersonation logged in audit trail
- [ ] Data export: Server Action generating JSON for a user/workspace
- [ ] Audit trail viewable from admin dashboard (searchable by user, action, date)
- [ ] Stripe Dashboard used for billing overrides (not custom tooling)
- [ ] 2FA enforced on admin accounts

## Common Mistakes

- Building a custom analytics dashboard instead of linking to Stripe/PostHog/Sentry
- Impersonation without a visible banner (admin forgets they're impersonating, takes actions as the wrong user)
- Impersonation not logged (no audit trail of who accessed whose account)
- Manual database updates for billing instead of using Stripe Dashboard + webhooks
- Admin dashboard accessible to workspace-level admins (too broad — system-level only)
- No audit trail for admin actions (can't reconstruct what happened during an incident)
- Data export that includes password hashes or session tokens
