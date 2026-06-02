# Account & Settings Pages

`[1]` Phase 5 — UI/UX, Item 9 — **Shadcn + React Hook Form + Neon Auth**

---

## What It Is

The account and settings surface is where users manage their identity, security, connected services, and personal preferences. It is visited infrequently but carries high trust weight — a poorly designed settings area signals a product that does not take user control seriously. This document covers the UI/UX of the settings surface only. The underlying auth mechanics (session model, password reset, OAuth linking, account deletion grace period) are defined in `phase-3-core-product/03-auth.md`. All forms follow `phase-3-core-product/02-validation.md`. Destructive confirmation dialogs follow `phase-5-ui-ux/05-overlays-feedback.md`.

---

## Settings Information Architecture & Navigation

Settings is a distinct section of the app, reached from the user avatar menu in the top-right of the app shell. It is not a sub-page buried inside the dashboard — it is a top-level route: `/settings`.

The left-nav pattern is standard and expected. On desktop a persistent left sidebar lists all settings sections. On mobile the sidebar collapses into a stacked list at the top of the page, each section expanding to reveal its content below.

**Desktop layout:**

```
┌─────────────────────────────────────────────────┐
│  ← Back to App                                  │
├──────────────────┬──────────────────────────────┤
│  Profile         │                              │
│  Password        │   [Selected section content] │
│  Email           │                              │
│  Sessions        │                              │
│  Connected       │                              │
│  Notifications   │                              │
│  ─────────────── │                              │
│  Danger Zone     │                              │
└──────────────────┴──────────────────────────────┘
```

**Mobile layout:**

```
┌──────────────────────┐
│  ← Back to App       │
├──────────────────────┤
│  Profile          ›  │
├──────────────────────┤
│  Password         ›  │
├──────────────────────┤
│  Email            ›  │
├──────────────────────┤
│  Sessions         ›  │
├──────────────────────┤
│  Connected        ›  │
├──────────────────────┤
│  Notifications    ›  │
├──────────────────────┤
│  Danger Zone      ›  │
└──────────────────────┘
```

Each nav item maps to a route: `/settings/profile`, `/settings/password`, `/settings/email`, etc. Navigating directly to a URL loads the correct section. The "Danger Zone" item is visually separated with a divider and uses a muted destructive colour to signal its nature without being alarmist.

For products with team/organisation features, personal settings and workspace/team settings are separated into two distinct navigation groups — never mixed together. Personal settings apply to the user's account. Team settings apply to the current workspace and are only visible to users with sufficient role. See `phase-3-core-product/13-team-rbac.md` for the role model. The personal vs team split is detailed in the Organisation & Team Settings section below.

---

## Profile

The profile section lets users update their display name and avatar. These are the lowest-risk fields in settings — no re-authentication required.

The form contains: avatar upload, first name, last name, and optionally a display name or bio field if the product surfaces them. It uses React Hook Form with a Zod schema. Validation is client-side first, server-side confirmed. On submit the save button enters a loading state. On success a "Changes saved" toast appears and the nav avatar updates immediately via optimistic update.

Avatar upload: clicking the avatar opens a file picker (accepts JPEG, PNG, WebP, max 5MB). The selected image is previewed immediately before upload. Upload happens on form save, not on file selection. The existing avatar is deleted from storage after a successful replacement.

Read-only fields (email, account creation date, user ID) are shown in a separate "Account Info" block below the form using static display values, not inputs. These exist to help users identify their account but cannot be changed here — email changes happen in their own section.

---

## Password Change

Password change is a separate section from profile. It requires the user's current password before accepting a new one — this re-authentication requirement is non-negotiable even for logged-in users. The form has three fields: current password, new password, confirm new password. All are validated with Zod (minimum 12 characters, no maximum, no arbitrary complexity rules). Confirm new password is validated client-side only.

On successful password change: show a success toast, invalidate all other active sessions (all sessions except the current one are revoked server-side), and show a "You have been signed out of other devices" informational message inline below the form. The mechanics of session invalidation are in `phase-3-core-product/03-auth.md`.

OAuth-only users (users who signed up via Google with no password set) see this section in a disabled state with a message: "You signed in with Google. Set a password to also enable email sign-in." A "Set Password" flow sends them a password-setup link via email — it is not the same as password reset.

---

## Email Change

Email change is a high-trust operation. It follows a two-step verification flow: the user enters their new email address, receives a verification link at the new address, and the change only takes effect after clicking that link. The old email address receives a notification that a change was requested, with a "This wasn't me" cancel link that revokes the pending change.

The form is a single field: new email address, with Zod email validation. On submit the button shows "Sending verification email…" then a success state: "Verification email sent to [new address]. Check your inbox." The current email is shown read-only above the form for reference.

Until the new address is verified, the pending change is surfaced with a dismissible banner: "Email change pending — verify [new@address.com] to complete." Clicking "Cancel change" in the banner or in the notification email revokes the pending request.

If the new address already belongs to another account, the form shows a field-level error: "That email address is already in use." No information about which account holds it is revealed.

---

## Active Sessions

The sessions section lists all currently active sessions for the user's account. Each session shows: device type (icon: desktop/mobile/tablet), browser and OS string, approximate location (city, country derived from IP at login time — not real-time), and time of last activity.

The current session is labelled "This device" and is listed first. All other sessions have a "Revoke" button. A "Sign out all other devices" button above the list revokes all sessions except the current one in a single action.

Revoking a session is optimistic — the row is removed from the UI immediately and the server call fires in the background. If the server call fails, the row reappears with an error toast. Revoking the current session is not possible from this list (the button is absent) — the user signs out via the avatar menu.

The session list is fetched fresh on each page load — no caching, since stale session data is a security risk. If the list is empty (only the current session exists), show a message: "No other active sessions."

---

## Connected Accounts

Connected accounts lists all OAuth providers linked to the user's account (e.g. Google, GitHub). Each row shows: provider logo, provider name, the connected email address, and an "Unlink" button.

Unlinking an account is only permitted if the user has at least one other login method remaining — either a password set or another OAuth provider linked. The Unlink button is disabled (with a tooltip: "You must have at least one login method") when it would leave the account with no way to sign back in.

Linking a new provider is done via an "Add [Provider]" button that initiates the OAuth flow. On return the new provider appears in the list with a success toast. The linking mechanics are in `phase-3-core-product/03-auth.md`.

If no providers are linked beyond the account's primary sign-in method, the section shows a prompt encouraging the user to connect an additional provider for easier future sign-in.

---

## Notification Preferences

Notification preferences let users control which communications they receive. This section surfaces the channels and categories defined in `phase-3-core-product/07-notification-system.md` as user-facing toggles.

Group notifications by category, not by channel. Categories come first (e.g. "Account security", "Product updates", "Team activity", "Billing alerts"). Under each category, show the available channels (Email, In-app) as toggles per row.

Certain notifications cannot be turned off — these are security and billing critical alerts (password changed, payment failed, account suspended). Show them as non-toggleable rows labelled "Required" with a tooltip explaining why.

Changes save immediately on toggle (auto-save, no submit button). Show a "Saved" indicator next to the toggled row for 2 seconds. If a save fails, revert the toggle to its previous state and show an error toast.

---

## Danger Zone

The Danger Zone is a visually distinct section at the bottom of the settings sidebar, separated by a divider. It contains a single action for personal settings: **Delete Account**. For team settings it may also include workspace deletion or ownership transfer — those are covered in the Organisation & Team Settings section.

The Delete Account button is styled with a destructive outline (not filled — filled red is reserved for the final confirmation). Clicking it opens a confirmation dialog following `phase-5-ui-ux/05-overlays-feedback.md` with these specifics:

- The dialog explains what will be permanently deleted (data, workspaces the user owns, access to shared resources)
- If the user has an active paid subscription, they are blocked from deletion and shown: "You have an active subscription. Cancel your subscription before deleting your account." Link to billing management.
- If the user owns a team workspace with other members, they are blocked and shown: "Transfer ownership or remove all members from your workspace before deleting your account."
- If neither condition blocks, the user must type their email address into a text input to confirm — the delete button remains disabled until the typed value matches exactly.

Account deletion triggers the GDPR self-service deletion flow in `phase-8-business-wrapper/06-gdpr-self-service.md`. Do not implement ad-hoc deletion logic here — link to that flow.

---

## Organisation & Team Settings

When the product has teams or workspaces, settings split into two groups:

**Personal settings** (`/settings/...`) — affect only the current user's account. Always visible. Nav label: "Account".

**Team settings** (`/settings/team/...` or `/[workspace]/settings/...`) — affect the current workspace. Only visible to users with Admin or Owner role (per `phase-3-core-product/13-team-rbac.md`). Nav label: "Workspace" or the workspace name.

The two groups appear as separate nav sections with a divider between them. Never mix personal and team settings in the same nav group — it creates confusion about what a change will affect.

Team settings includes: workspace name and logo, member management (invite, remove, change role), billing (if billing is workspace-scoped), SSO configuration (if applicable), and workspace danger zone (delete workspace, transfer ownership).

Member management in team settings must never expose other members' personal data beyond name, avatar, email address, and role. Phone numbers, secondary emails, session data, or any other personal fields from their profile are not surfaced to admins.

---

## Security Considerations

**Re-authentication for sensitive changes.** Password change requires the current password. Email change triggers out-of-band verification to the new address. Treat these as separate auth events — do not rely solely on the existing session being valid.

**Email change notification to old address.** The moment a change is requested, notify the old address. This is the user's only safety net if their account is compromised. Include a one-click revocation link that expires in 24 hours.

**Session revocation after password change.** Changing a password must invalidate all other sessions. Failure to do this means a compromised session stays alive after the user secures their account.

**Connected account unlink guard.** Server-side enforcement: never allow unlinking if it would leave the account with zero login methods. Client-side disabling is UX convenience — the server must also reject the request.

**Team settings data isolation.** Admin-level team settings pages must scope all queries to the current workspace. Verify server-side that the requesting user has the required role for the workspace — never rely on frontend role checks alone. A team admin must never be able to see data from a different workspace.

**Typed confirmation is not security.** Requiring the user to type their email or workspace name before deleting is friction that prevents accidents — it is not an authentication step. The actual authorisation check (is this user allowed to delete this resource?) must happen server-side regardless.

---

## Deliverables / Implementation Checklist

### Layout & Navigation
- [ ] `/settings` route with left-nav sidebar (desktop) and stacked nav (mobile)
- [ ] Active nav item highlighted with current route
- [ ] "Back to App" link at the top of the settings sidebar
- [ ] Personal settings and team settings in separate nav groups (if team features exist)
- [ ] Danger Zone item visually separated with divider and muted destructive colour

### Profile
- [ ] Display name fields with React Hook Form + Zod validation
- [ ] Avatar upload with client-side preview and file type/size validation
- [ ] Avatar replaces on save, old avatar deleted from storage
- [ ] Read-only account info block (email, joined date) below the form

### Password
- [ ] Current password + new password + confirm fields
- [ ] Zod: minimum 12 characters
- [ ] Success invalidates all other sessions
- [ ] "Signed out of other devices" message shown after change
- [ ] Disabled state for OAuth-only users with "Set Password" flow

### Email Change
- [ ] Single new-email field, submit triggers verification email
- [ ] Current email shown read-only above the form
- [ ] Pending change banner with cancel option
- [ ] Notification email sent to old address with revocation link
- [ ] Duplicate email shows field-level error (no account information leaked)

### Sessions
- [ ] Session list fetched fresh on load (no cache)
- [ ] Current session labelled "This device", listed first
- [ ] Per-session: device icon, browser/OS, location, last active timestamp
- [ ] "Revoke" button per non-current session (optimistic removal)
- [ ] "Sign out all other devices" bulk revoke button
- [ ] Empty state when no other sessions exist

### Connected Accounts
- [ ] List of linked OAuth providers with provider email
- [ ] Unlink button disabled when it would remove the last login method
- [ ] "Add [Provider]" button initiates OAuth link flow
- [ ] Success toast on link/unlink

### Notifications
- [ ] Categories as grouping, channels as per-row toggles
- [ ] Auto-save on toggle with "Saved" inline indicator
- [ ] Required (non-toggleable) notifications clearly marked
- [ ] Failed save reverts toggle and shows error toast

### Danger Zone
- [ ] Delete Account button (destructive outline style)
- [ ] Pre-flight checks: active subscription → block with message, owns workspace with members → block with message
- [ ] Typed email confirmation required before delete button enables
- [ ] Deletion triggers GDPR self-service flow (`phase-8-business-wrapper/06-gdpr-self-service.md`)
- [ ] Team settings danger zone: workspace deletion with ownership transfer option

---

## What to Test

- Profile: saving with an invalid avatar file type is rejected client-side before upload
- Profile: avatar preview shows immediately on file selection before save
- Password: current password field rejects wrong value with a field-level error
- Password: successful change invalidates all sessions except the current one
- Password section: OAuth-only user sees the disabled state, not the form
- Email change: verification email sent to new address, not current address
- Email change: old address receives notification with working revocation link
- Email change: entering an address that belongs to another account shows field-level error only
- Email change: cancelling the pending change via banner removes the banner and cancels server-side
- Sessions: revoking a session removes it from the list immediately (optimistic)
- Sessions: revoking current session is not possible from this page
- Sessions: "Sign out all other devices" removes all rows except "This device"
- Connected accounts: Unlink button is disabled when only one login method remains
- Connected accounts: server rejects unlink request that would leave zero login methods
- Notifications: toggle auto-saves, "Saved" indicator appears and fades
- Notifications: required notifications cannot be toggled (disabled with tooltip)
- Danger zone: delete button is blocked when active subscription exists
- Danger zone: delete button is blocked when user owns a workspace with members
- Danger zone: delete button remains disabled until typed email matches exactly
- Team settings: admin cannot view or access settings for a workspace they are not a member of

---

## Edge Cases

- **Email change to an address already in use** — show a field-level error ("That email address is already in use") and do not reveal which account holds the address. Never expose account existence through error messages.
- **Revoking your own current session** — the current session's Revoke button must not be rendered. If an API call somehow reaches the server with the current session token, reject it and return an error rather than signing the user out unexpectedly.
- **Unlinking the only login method** — block at both UI (disabled button with tooltip) and API (server returns 400). If the user has no password and tries to unlink their only OAuth provider, show a prompt to set a password first.
- **Changing password invalidates other sessions** — intended behaviour, but inform the user. Show "You have been signed out of other devices" inline below the success message. Do not show this as an error — it is a security feature.
- **Deleting account while subscription is active** — block deletion and surface a clear path: a link to the billing management page to cancel the subscription first. Do not silently cancel the subscription on their behalf.
- **OAuth-only user navigating to the password change page** — the password change form is replaced with a disabled state explaining the situation. Do not show an empty form that fails silently.
- **Email verification link expired** — if the user clicks the email change verification link after it has expired (24 hours), show an expired-link page with a button to request a new verification email. Do not silently fail.
- **Team member viewing team settings without admin role** — server returns 403, the settings page shows a "You don't have permission to view this page" empty state. Do not redirect to the personal settings root — it is confusing.
- **User with pending email change attempts another email change** — cancel the first pending change server-side and issue a new verification for the latest address. Never have two pending email changes simultaneously for one account.
- **Session list shows unrecognised device** — if the user does not recognise a session, they should be able to revoke it immediately and optionally change their password. Add a "Revoke and report suspicious activity" option alongside the regular revoke, which also flags the event for logging.

---

## Common Mistakes

- Mixing personal and team settings in the same nav group — users don't know if changing a setting affects their account or the whole workspace.
- No "Back to App" link in settings — users end up using the browser back button or getting lost.
- Password change that doesn't invalidate other sessions — the most common security gap in settings implementations.
- Email change that takes effect immediately without verification — allows anyone with temporary access to the session to silently reroute account emails.
- Showing a typed-email confirmation on the delete button without blocking active subscriptions first — users complete the flow only to see a cryptic billing error at the last step.
- Toggling notifications with a manual save button — users change a toggle and think it took effect, but have to scroll down to find a save button they might miss.
- Not showing the current email address above the email change form — users don't know what address is currently on file, especially if they have a pending change.
- Unlink button enabled for the last login method — account becomes inaccessible; requires support intervention to recover.
- Session list cached on the client — user sees stale sessions that have already been revoked, or active sessions added since the page loaded.
- Team settings admin checks enforced only on the frontend — any user who knows the settings URL can reach the page without a server-side role check.
