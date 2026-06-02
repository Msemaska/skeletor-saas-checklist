# Team / Organization Management & RBAC

> `[1]` if B2B / multi-user — **Neon Auth (Better Auth Organization plugin)**

---

## What It Is

The system that defines organizational units (teams, companies, workspaces), controls who belongs to them, and determines what each member is allowed to do. In a B2B product, the organization is the billing and access boundary — every resource, every permission check, and every Stripe seat count flows through it. Without this system, you have a personal-use app. With it, you have a product a team can actually buy together.

---

## Org / Tenant Model & Data Isolation

Use the Better Auth Organization plugin. It adds `organization`, `member`, `invitation`, and `team` tables to your Neon Postgres database alongside the core auth schema. The plugin is the source of truth for membership and roles; your application schema extends from it.

Every tenant-scoped row in every feature table carries an `organization_id` foreign key. There are no exceptions. This is the single column that separates one customer's data from another's. Data isolation is enforced at the database level via RLS policies (see `phase-3-core-product/01-database-schema.md`) — the policy checks that the authenticated user is a member of the row's organization before allowing any operation.

The data flow is:

```
Organization
  │
  ├── Members (user ←→ organization, with role)
  │     owner / admin / member / [custom roles]
  │
  ├── Invitations (email, role, token, expiry)
  │
  ├── Teams (optional sub-groups within an org)
  │     └── Team Members
  │
  └── Feature Tables
        every row carries organization_id
```

Never query feature data without an explicit `WHERE organization_id = :currentOrgId`. This filter must live in the service layer (see `phase-3-core-product/04-data-access.md`), not in individual components or pages. The service functions receive the authenticated user's current organization context and apply it to every query automatically.

RLS is the safety net: even if the service layer has a bug, RLS policies at the database level will block cross-org reads. All RLS policies on tenant-scoped tables must check membership in the organization, not just user identity.

---

## Roles & Permissions

Three default roles cover 90% of B2B SaaS products:

| Role | What they can do |
|---|---|
| **owner** | Everything — billing, settings, member management, deleting the org. There is always exactly one owner per org. |
| **admin** | Invite and remove members, update member roles (up to admin — cannot set owner), manage org settings. Cannot access billing or delete the org. |
| **member** | Use the product. Cannot manage the org, members, or billing. |

Custom roles are supported by the Better Auth Organization plugin. Add them when a specific permission boundary is needed (e.g., a `billing_admin` role for a finance contact who should see invoices but not manage users). Do not add custom roles speculatively — every extra role is a combinatorial surface that needs testing and UI support.

Roles are per-organization, never global. A user who is an admin in one organization is a member (or nothing) in another. The Better Auth session context carries the user's current organization and their role in it — that role is the only thing server-side permission checks should trust.

Permission checks happen exclusively in the service layer, never trusted from the client. The role is read from the authenticated session on the server and checked before executing any operation. The UI may show or hide elements based on role (admin sees the "Members" settings tab), but hiding the button is UX only — the server action enforces the real restriction.

The check hierarchy:

```
Request arrives at Server Action / API route
  │
  ├── 1. Is the user authenticated? (session check — see phase-3-core-product/03-auth.md)
  ├── 2. Does the user belong to the target organization? (membership check)
  ├── 3. Does their role permit this operation? (role check)
  └── 4. Does RLS agree? (database-level enforcement — see phase-3-core-product/01-database-schema.md)
```

Never skip step 2. Checking role without first confirming membership allows a user with a valid session in org A to attempt role-bearing operations against org B.

---

## Member Invitations & Onboarding

Invitations are the only way members join an organization — there is no self-join for B2B products. An invite always carries a role. The inviting user must be an admin or owner.

**Invitation flow:**

1. Admin enters an email address and selects a role.
2. Better Auth creates an invitation record with a cryptographically random token and an expiry (48 hours default, configurable to 7 days — use 7 days to reduce friction for busy teams).
3. An invitation email is sent via your transactional email provider with a link containing the token.
4. Clicking the link:
   - If the recipient has an account: they are immediately added to the org with the specified role and redirected to the org dashboard.
   - If they do not have an account: they are taken through the sign-up flow; on completion, they are automatically added to the org. The token is preserved through the sign-up redirect (see `phase-3-core-product/03-auth.md` for redirect context handling).
5. The invitation record is marked accepted and the token is invalidated.

Pending invitations are visible to admins and owners. They can cancel a pending invite before it is accepted. Re-inviting the same email to the same org cancels the old invite and issues a new one.

An invitation email address may already belong to a user account in another organization. This is fine — a user can belong to multiple organizations. The invitation adds them to a new org without affecting their existing memberships.

---

## Seats & Billing Alignment

Seats are the bridge between your membership list and your Stripe subscription. Every organization is a Stripe customer. The Stripe subscription for that organization uses a quantity-based price item where the quantity equals the number of active members.

When a member is added (invitation accepted), increment the Stripe subscription quantity by 1. When a member is removed or leaves, decrement by 1. Better Auth's Stripe plugin handles this automatically when configured with `customerType: "organization"` and per-seat pricing. Pending invitations do not count as seats — only accepted memberships do.

Seat synchronization must be idempotent. If a webhook fires twice, the seat count should not change twice. Always derive the correct quantity from the database (count of active members) and set it as an absolute value rather than incrementing/decrementing via webhook-triggered calculations.

The organization's current plan and seat limit live on the workspace/org record (synced from Stripe webhooks). Before accepting an invitation, check that adding the new member would not exceed the org's purchased seat count. If it would, reject the invitation acceptance with a clear message: "This organization has reached its member limit. Ask an admin to upgrade the plan." See `phase-8-business-wrapper/02-stripe-architecture.md` for full billing architecture.

---

## Switching Organizations

A user who belongs to multiple organizations can switch between them. The currently active organization is stored in the session (Better Auth tracks the active organization context). Switching organizations updates the session context — all subsequent requests operate against the new org's data.

The URL should reflect the current organization via a slug or ID in the path (e.g., `/org/acme-corp/dashboard`). This makes deep links work across orgs — if a user navigates directly to an org-specific URL, load that org's context rather than defaulting to the user's last-active org.

When switching, do not carry over state from the previous org. A filtered view in org A is irrelevant when the user switches to org B.

The org switcher UI belongs in account settings — see `phase-5-ui-ux/09-account-settings.md`.

---

## Ownership Transfer

Every organization has exactly one owner. Ownership can be transferred — but the transfer is a deliberate, protected action, not a casual admin operation.

Transfer rules:
- Only the current owner can initiate an ownership transfer.
- The target must be an existing admin in the organization (cannot transfer to a plain member in a single step — promote them first, then transfer).
- On transfer: the current owner's role is immediately downgraded to admin. The new owner's role is set to owner. No two owners can exist simultaneously.
- Send notification emails to both the outgoing and incoming owner confirming the change.
- Log the ownership transfer in the audit log (`phase-3-core-product/14-audit-logging.md`) — ownership changes are high-value security events.

An owner cannot leave the organization or delete their account until they either transfer ownership or delete the organization entirely. This is enforced at the server level — see the edge case for last-owner protection below.

---

## Security Considerations

**Cross-tenant data leakage** is the most severe risk in any multi-tenant system. A missing `organization_id` filter in a single service function can expose every customer's data. Defense: the service layer applies the org filter on every query, and RLS provides a second enforcement layer at the database. Never disable RLS, and never run user-context queries without the organization ID present.

**Privilege escalation** occurs when a member grants themselves or another user a higher role than they are authorized to set. Enforce that: members cannot change any role; admins can set roles up to admin but never owner; only the owner can promote another user to owner. These checks happen server-side on every role-change action, not based on what the client sends.

**Invitation token security**: Tokens are single-use and time-limited. An accepted or expired token must be invalidated immediately — checking expiry alone is not enough if the token can be accepted multiple times. Store accepted_at on the invitation record and reject any token where accepted_at is already set. Tokens must be sufficiently random (use the Better Auth-generated token, not a predictable value like an invitation ID).

**Last-owner protection**: If the sole owner removes themselves, the organization becomes ungovernable — nobody can manage billing, transfer ownership, or delete the organization. Block any action (leave, deletion, role downgrade) that would result in an organization with zero owners. The error message should prompt the user to transfer ownership first.

**RBAC enforcement location**: Roles read from the client are never trusted. The role is always resolved from the database using the authenticated user's session, against the specific organization being acted upon. A user claiming to be an admin in their request cannot elevate their actual role.

---

## Deliverables / Implementation Checklist

### Organization Setup
- [ ] Better Auth Organization plugin configured (self-hosted if Neon Auth does not yet expose the plugin; otherwise use Neon Auth)
- [ ] `organization`, `member`, `invitation`, `team` tables present in the database (created by Better Auth plugin schema)
- [ ] All feature tables carry `organization_id` with foreign key reference and a composite index on `(organization_id, [primary filter column])`
- [ ] RLS policies on all feature tables enforce organization membership check before SELECT, INSERT, UPDATE, DELETE

### Roles & Permissions
- [ ] Three default roles defined: owner, admin, member
- [ ] Custom roles defined (if required by product — add only when there is a concrete need)
- [ ] Server-side permission check utilities created in the service layer: `requireOrgMember`, `requireOrgAdmin`, `requireOrgOwner`
- [ ] Every Server Action and API route that touches org-scoped data calls the appropriate check before executing
- [ ] UI hides restricted controls for lower-privilege roles (UX only — server enforcement is the source of truth)

### Invitations
- [ ] Invite flow working: admin enters email + role → invitation record created → email sent
- [ ] Invitation expiry set to 7 days
- [ ] Existing-account path: click link → immediately joined, redirected to org dashboard
- [ ] New-account path: click link → sign-up flow → auto-join on completion → redirected to org dashboard
- [ ] Pending invitations list visible to admins/owners with cancel option
- [ ] Re-invitation cancels old pending invite and issues a fresh one
- [ ] Invitation tokens are single-use (accepted_at check enforced)

### Seats & Billing
- [ ] Organization is the Stripe customer entity (customerType: "organization")
- [ ] Stripe subscription uses per-seat quantity pricing
- [ ] Seat count increments when invitation is accepted
- [ ] Seat count decrements when member is removed or leaves
- [ ] Invitation acceptance blocked if seat count would exceed plan limit — clear error message shown
- [ ] Seat count synchronization is idempotent (absolute set, not delta)

### Org Switching
- [ ] Active organization stored in session context
- [ ] URL structure includes org slug/ID for all org-scoped pages
- [ ] Direct org-specific URLs load the correct org context regardless of last-active org
- [ ] Org switcher available in the account settings UI

### Ownership Transfer
- [ ] Transfer only initiatable by current owner
- [ ] Target must be an existing admin before transfer
- [ ] On transfer: outgoing owner downgraded to admin, incoming user set to owner atomically
- [ ] Notification emails sent to both parties
- [ ] Transfer logged in audit log
- [ ] Owner cannot leave or delete account without transferring ownership first — server enforces this

---

## What to Test

### Isolation
- A user in org A cannot read, write, or delete any row scoped to org B — even if they supply org B's ID directly in the request
- RLS blocks cross-org access at the database level independently of the service layer
- Switching to a different org loads only that org's data — no bleed from previous org context

### Roles
- A member cannot invite users, change roles, or access org settings
- An admin can invite, remove members, and update roles up to admin — but not owner
- An admin cannot access billing pages or delete the organization
- An owner can do everything including billing and org deletion
- Role checks are enforced server-side — a forged client request claiming a higher role is rejected

### Invitations
- Inviting an existing account email immediately adds them to the org on acceptance
- Inviting an unknown email creates a pending invitation and sends the email
- Clicking an invite link as a new user: completing sign-up joins the correct org
- Expired invitation token is rejected with a clear message and a prompt to request a new invite
- An already-accepted invitation token is rejected on re-use
- Cancelling a pending invite invalidates the token immediately
- Re-inviting the same email cancels the previous invite and creates a fresh one

### Seats
- Accepting an invitation when the org is at its seat limit is blocked — not silently allowed
- Stripe seat quantity increments exactly once per accepted invitation
- Removing a member decrements the Stripe seat quantity
- Seat sync is idempotent — processing the same event twice does not double-count

### Ownership Transfer
- Non-owners cannot initiate a transfer
- Transfer to a non-admin is rejected
- After transfer, the previous owner has admin role, the new owner has owner role — no concurrent owners
- Owner blocked from leaving or deleting account until ownership is transferred

### Org Switching
- Navigating to an org-specific URL while authenticated sets the correct org context
- Switching orgs clears state from the previous org
- A user with no membership in an org cannot access its pages — receives 403

---

## Edge Cases

- **Inviting an email that already has an account in another org** — allow it; a user can belong to multiple organizations. The invitation adds a new membership record without affecting existing ones.

- **Last owner tries to leave the organization** — block the action at the server level and return a clear error: "You are the only owner. Transfer ownership to another admin before leaving." Do not rely on client-side UI alone.

- **Last owner tries to delete their account** — same protection as above; account deletion must check for sole-owner status across all orgs and block if any unresolved sole-owner situation exists. See `phase-3-core-product/03-auth.md` for account deletion flow.

- **Removing a member who created shared resources** — do not cascade-delete their content. Set the `created_by` foreign key to `NULL` (using `ON DELETE SET NULL`) and display the author as "Removed Member" in the UI. The data belongs to the organization, not the individual.

- **Downgrading seats below current member count** — never silently remove members to fit a lower plan. Instead, block the downgrade and tell the admin exactly how many members need to be removed first, or redirect them to the member management page. The user decides who leaves; the app never auto-removes anyone.

- **A user belonging to multiple orgs** — every request to an org-scoped resource must explicitly identify which organization is in scope. Derive it from the URL or the active session org context, not from "the user's org" (which is ambiguous). A missing org context should return a 400, not silently pick one.

- **Pending invite to an email that later changes** — if the invited user changes their email after receiving the invite but before accepting, the token is still tied to the original email address. The link still works because the token resolves the invitation record directly, not by matching the current user's email. If there is no account at all, the sign-up created from the invite link should prompt for the email the invite was sent to, and validate it matches the invitation record.

- **Invitation sent to the org's existing member** — detect and reject at invite-creation time with a clear message ("this user is already a member") rather than creating a duplicate membership on acceptance.

- **Admin tries to set a member's role to owner** — reject server-side. Only the current owner can create a new owner. The admin role-change endpoint must have an explicit ceiling.

- **Organization deleted while members have active sessions** — on org deletion, any active session holding that organization as the current context should be invalidated or redirected on the next request. Do not allow continued access to a deleted org's data.

---

## Common Mistakes

- Checking org membership in the UI but not in the service layer — the server action is the only enforcement that matters
- Accepting `organization_id` from the request body instead of deriving it from the session context — a user can forge any org ID they want in a POST body
- Forgetting `organization_id` on a new feature table — one missed table means all members of all orgs can see that data, or none can
- Using global roles instead of per-org roles — admin in one org grants nothing in another; never query permissions without the org context
- Setting up invitations without a seat check — a user can invite beyond the plan limit if the check is missing, creating unbilled members
- Treating pending invitations as occupied seats before acceptance — this over-charges customers on plans where they are exploring adding users
- Not invalidating accepted invite tokens immediately — a replayed invite link creates a second membership record for the same user
- Allowing ownership transfer to a non-admin in a single step — promotes a low-trust member to the highest role with no intermediate step
- No last-owner guard on account deletion — orphaned organizations with billing relationships but no owner create a permanent support burden
- Seat sync implemented as delta (increment/decrement) rather than absolute set — retried webhooks double-count, causing seat count drift
- Not logging ownership transfers and role changes in the audit log — these are the events you will need most when something goes wrong
- Role checks that only inspect the role value without first confirming the user is actually a member of the org in question
