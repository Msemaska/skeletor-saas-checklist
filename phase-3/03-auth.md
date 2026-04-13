# Authentication System

---

## What It Is

The system that handles who your users are, how they prove it, and what they're allowed to do. Auth touches every layer of your app — frontend forms, API middleware, database policies, email flows, and session management.

## Architecture Overview

Every SaaS auth system has the same fundamental components regardless of provider:

```
┌─────────────────────────────────────────────────┐
│                   Frontend                       │
│  Sign Up ─ Sign In ─ OAuth ─ Sign Out           │
│  Password Reset ─ Password Change ─ Email Change │
│  2FA Challenge ─ Session Check                   │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│               Auth Middleware                     │
│  Route protection ─ Token validation             │
│  Session refresh ─ Public/private gate            │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│               Auth Backend                        │
│  User storage ─ Password hashing ─ Token mgmt    │
│  OAuth handshakes ─ Email verification           │
│  Session storage ─ 2FA secrets                   │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│               Database Layer                      │
│  Users table ─ Sessions table ─ Accounts table   │
│  RLS policies ─ Roles/permissions                │
└─────────────────────────────────────────────────┘
```

## Core Capabilities

### 1. Identity (who is the user?)

**Email + Password:**
- Sign up with email and password
- Password hashing (bcrypt/argon2 — never store plaintext)
- Email verification (send confirmation link before account is active)
- Password strength requirements (minimum length, complexity)

**Social Login / OAuth:**

Every provider requires: client ID, client secret, callback URL, and scope configuration. Your auth provider (Neon Auth or self-hosted Better Auth) handles the OAuth handshake — you configure the credentials and callback URLs, not the protocol itself.

**Google OAuth — mandatory for every SaaS:**
- The single most expected social login. If you only add one, make it Google.
- Requires a Google Cloud Console project with OAuth 2.0 credentials
- Request scopes: `email` and `profile` (gives you email, name, avatar). Do not request broader scopes unless you need them — each additional scope requires justification during Google's verification review.
- Google requires a consent screen with your app name, logo, privacy policy URL, and terms of service URL. Unverified apps show a scary warning to users — submit for verification before launch.
- Verification review can take 2-4 weeks. Start this process early, not the week before launch.
- Google provides a verified email — you can skip email verification for users who sign up via Google OAuth.
- Data returned: email (verified), full name, profile picture URL, locale

**Apple Sign-In — required if you have an iOS app, recommended otherwise:**
- Apple requires Sign in with Apple if your iOS app offers any other social login. Even if you're web-only, it's a strong trust signal.
- Requires an Apple Developer account ($99/year) and a Services ID configured in the Apple Developer portal
- Apple's unique feature: users can choose to hide their email. Apple generates a relay address (random@privaterelay.appleid.com) that forwards to their real email. Your app must handle these relay addresses as valid emails.
- Apple only sends the user's name on the FIRST sign-in. If you don't capture and store it on the first callback, you will never get it again. Your OAuth callback must handle this.
- Key rotation: Apple uses a private key (not a client secret) that must be refreshed periodically. Set a reminder.
- Data returned: email (real or relay), full name (first sign-in only), no profile picture

**GitHub OAuth — recommended for developer-facing products:**
- If your audience is developers, GitHub is expected. If not, skip it.
- Requires a GitHub OAuth App created in GitHub Developer settings
- Request scopes: `read:user` and `user:email` (gives you profile + email). The `user:email` scope is needed because GitHub allows private emails — without it you may not get their email at all.
- GitHub users can have multiple emails. The API returns a list — use the one marked as `primary` and `verified`.
- Data returned: email (may need separate API call), username, display name, avatar URL

**Facebook Login — optional, consider for consumer-facing products:**
- Relevant for B2C products targeting a general audience. Less relevant for developer tools or B2B.
- Requires a Facebook App created in Meta Developer portal
- Request permissions: `email` and `public_profile`
- Facebook's app review process is required before your app can request data from users who aren't app admins/testers. Submit for review before launch.
- Facebook email is not always available — some users sign up to Facebook with a phone number and have no email on file. Handle this gracefully (prompt them to provide an email after sign-in).
- Data returned: email (not guaranteed), name, profile picture URL

**Provider maintenance (ongoing):**
- OAuth callback URLs must be updated if your domain changes — a missed callback URL means broken sign-in
- Google and Facebook consent screens and app reviews can be revoked if your privacy policy changes or if you violate their terms
- Client secrets should be rotated periodically and stored securely in environment variables, never committed to code
- Monitor for provider API changes — OAuth providers occasionally deprecate endpoints or change token formats

**Which providers to launch with:**
- Minimum: Email + Password and Google OAuth
- Add Apple if you have or plan to have an iOS app
- Add GitHub if your audience is developers
- Add Facebook if you're B2C consumer-facing
- Do not add providers "just in case" — each one is a maintenance surface. Add them when there's demand.

### 2. Session Management (is the user still authenticated?)

Use a hybrid approach: JWT for short-lived access tokens, database-backed sessions for refresh and revocation. Neon Auth (and Better Auth if self-hosted) implements this by default.

- Short-lived access token (15-60 minutes) — auto-refreshes, no DB lookup needed
- Long-lived refresh token (7-14 days) — stored in database, enables revocation
- All tokens stored in HttpOnly, Secure, SameSite=Lax cookies
- Never store tokens in localStorage (XSS vulnerable)

### 3. Multi-Factor Authentication (2FA)

- TOTP only (authenticator app — Google Authenticator, Authy). No SMS — it's less secure and adds carrier/international complexity.
- Must be optional at launch, but the infrastructure should exist
- 2FA setup lives in security settings, not in onboarding. Don't force it on users during sign-up — let them opt in when they're ready.

**Sign-in flow with 2FA enabled:**
- User enters email + password → server validates credentials → if 2FA is enabled, server responds with a 2FA challenge instead of creating a session → user enters TOTP code → server validates code → session created. The session is never created until the 2FA challenge passes.

**Recovery codes:**
- Generate 8 codes, each 8 alphanumeric characters, displayed in a grid
- Each code is single-use — once used, it's burned
- Show recovery codes exactly once when 2FA is first enabled. Prompt the user to save them. Do not allow them to proceed until they confirm they've saved them (checkbox or "I've saved these" button).
- Allow users to regenerate recovery codes from security settings (invalidates all previous codes, requires re-authentication)
- If a user exhausts all recovery codes and loses their authenticator device, they are locked out. Customer support must have a manual identity verification process to disable 2FA in this case.

### 4. Password Reset

- User requests reset via email
- Server generates a time-limited token (1 hour max)
- Email contains a link with the token
- User sets new password, token is invalidated
- Never reveal whether the email exists ("If an account exists, we've sent a reset link")

### 5. Account Verification

- Send verification email on sign-up
- Link contains a signed token
- Account is unverified until clicked
- **Decision: unverified users CAN access the app**, but with a persistent banner prompting them to verify. Do not block access entirely — it creates friction and increases drop-off. If they haven't verified after 7 days, restrict access to read-only until they verify.
- Resend verification email option must be available (with rate limiting — max 3 resends per hour)

### 6. Authorization (what can the user do?)

**Role-based access control (RBAC):**
- Common roles: owner, admin, member, viewer
- Roles are per-workspace/org, not global (a user can be admin in one workspace, viewer in another)
- Enforce at: API layer (server actions/routes), database layer (RLS), and UI layer (conditional rendering)
- Middleware is NOT the place for role checks — it only gates authenticated vs unauthenticated

**Resource-level permissions:**
- Not every permission maps to a role. Sometimes access is per-resource: "this user can edit this specific project because they were invited to it."
- Model this with a membership/permissions table that links users to specific resources, not just to workspaces
- Check resource ownership or membership in Server Components and API routes, not middleware
- Example: a user with "member" role in a workspace may own some projects but not others — role alone isn't enough, you also check if they're the owner or a collaborator on that specific resource

**Row-Level Security (RLS):**
- Database-level enforcement — even if your API has a bug, users can't access other users' data
- Neon supports Postgres RLS natively — the application layer sets session context (e.g., `set_config('app.user_id', ...)` at the start of each request) and RLS policies reference `current_setting('app.user_id')`
- This is standard Postgres, no vendor-specific functions — works the same whether using Neon Auth or self-hosted Better Auth
- RLS policies should handle both role-based and resource-level checks (e.g., "user is a member of this workspace AND is assigned to this project")

### 7. Email Change

Changing an email address is a security-sensitive auth event, not a simple profile update.

**Design decisions:**
- Require re-authentication (enter current password or re-confirm OAuth) before initiating an email change
- Send a verification link to the NEW email address. The change does not take effect until the new email is verified.
- The old email remains active and functional until the new one is confirmed
- If the new email already belongs to another account, reject the change and inform the user (without revealing whose account it is)
- Send a notification to the OLD email informing them that an email change was requested ("if this wasn't you, contact support")
- Once the new email is verified: update the user record, invalidate all existing sessions (force re-login with new email), send confirmation to both old and new email

### 8. Password Change (while signed in)

Different from password reset (which is for forgotten passwords via email link). This is changing your password from settings while already authenticated.

**Design decisions:**
- Require the current password before accepting a new one (prevents someone who finds an unlocked device from changing the password)
- Validate the new password meets strength requirements
- On successful change: invalidate all OTHER sessions (keep the current one active so the user isn't kicked out)
- Send a notification email: "Your password was changed. If this wasn't you, reset your password immediately."
- Do not allow the new password to be the same as the current password

### 9. Session Lifecycle & Limits

**Lifecycle — what happens at each event:**

| Event | Action |
|---|---|
| Sign up | Create user, send verification email, create session |
| Sign in | Validate credentials, create session, set cookie |
| OAuth callback | Create/link account, create session, set cookie |
| Page load (server) | Validate session from cookie, inject user into request |
| API call | Validate session/token, reject if expired |
| Token expiry | Use refresh token to get new access token |
| Sign out | Destroy session, clear cookie |
| Password reset | Invalidate ALL sessions |
| Password change | Invalidate all OTHER sessions (keep current) |
| Email change confirmed | Invalidate ALL sessions |
| Account deletion | Destroy all sessions, delete user data |
| Suspicious activity | Invalidate all sessions, require re-auth |

**Concurrent session policy:**
- Allow multiple concurrent sessions by default (user on phone + laptop + tablet is normal)
- Set a maximum of 5-10 concurrent sessions per user — if a new session is created beyond the limit, destroy the oldest session
- Display active sessions in the user's security settings (device name, location, last active, IP)
- Allow the user to revoke individual sessions or "sign out all other devices"

**Session duration:**
- Durations defined in section 2 (access token: 15-60 min, refresh token: 7-14 days)
- Absolute session expiry: 30 days max regardless of activity (forces re-authentication periodically)
- For sensitive actions (changing password, changing email, deleting account): require re-authentication even if the session is valid

**Shared/team accounts:**
- Discourage shared credentials — design your RBAC so that every person has their own account with appropriate permissions
- If you detect the same credentials being used simultaneously from different IPs/locations, flag it but don't block it (could be legitimate — VPN, travel)

### 10. Account Linking / Identity Merging

When a user signs up with email+password and later tries to sign in with Google OAuth using the same email (or vice versa), you need a clear policy.

**Design decisions:**
- If a user already exists with that email and they attempt OAuth with the same email, **do not auto-create a second account**. This leads to duplicate accounts and split data.
- **Recommended approach:** detect the existing account, prompt the user to sign in with their original method first, then link the OAuth identity to their existing account from their settings page.
- Never auto-merge without verification — an attacker could create an OAuth account with someone else's email to hijack their account.
- Once linked, the user can sign in with either method and reaches the same account.
- Store linked identities in an accounts/identities table (most auth providers handle this automatically).
- Display linked identities in the user's profile settings so they can see and manage their sign-in methods.

### 11. Account Deletion

GDPR and user trust require a clear account deletion path. This is more complex than "delete the row."

**What happens when a user requests deletion:**
- Cancel their Stripe subscription at end of current billing period (the user already paid for that time — don't cut them off early)
- Destroy all active sessions across all devices
- Send a confirmation email acknowledging the deletion request
- Define a grace period (e.g., 30 days) where the account is deactivated but recoverable, or delete immediately — pick one and be explicit

**What to delete:**
- User profile data (name, email, avatar)
- Authentication records (password hash, OAuth tokens, 2FA secrets, sessions)
- User-generated content that is solely theirs (private projects, personal settings)

**What to anonymize (not delete):**
- Content in shared workspaces (other users still need it — replace the author name with "Deleted User")
- Billing history (you may be legally required to retain invoices for tax purposes)
- Audit logs (retain with anonymized user ID for compliance)

**What to handle externally:**
- Stripe customer record — cancel subscription, but Stripe retains billing history on their side
- Email marketing lists — unsubscribe from all marketing email lists (Resend)
- Analytics — PostHog/analytics data should be anonymized, not deleted (aggregate data is still valuable)

**Design decisions:**
- Account deletion must be self-service (a button in settings, not "email us to delete")
- Require re-authentication (password or OAuth) before processing deletion
- If the user is a workspace owner, require them to transfer ownership or delete the workspace first
- Log the deletion request and completion for compliance auditing

### 12. Rate Limiting on Auth Endpoints

Auth endpoints are the most attacked surface of any SaaS. Rate limiting is mandatory.

**Sign-in attempts:**
- Max 5 failed attempts per email per 15-minute window
- After 5 failures: lock that email for 15 minutes (not the IP — legitimate users behind a shared IP shouldn't be punished)
- After 15 failures across any email from the same IP: block the IP for 1 hour (distributed brute force)
- Successful sign-in resets the failure counter for that email

**Password reset:**
- Max 3 reset requests per email per hour (prevent email bombing)
- Max 10 reset requests per IP per hour (prevent using reset as a spam tool)
- Reset tokens expire after 1 hour regardless

**Sign-up:**
- Max 5 sign-up attempts per IP per hour (prevent mass account creation)

**OAuth:**
- OAuth flows are harder to brute force (redirect-based), but rate limit the callback endpoint to prevent token replay attacks

**Response behavior:**
- Never reveal rate limit details in the error message ("too many attempts, try again later" — don't say "you have 2 attempts remaining")
- Return the same response time for rate-limited and non-rate-limited requests (prevent timing attacks)

### 13. OAuth Token Refresh

If you only use OAuth for sign-in (not for accessing provider APIs after sign-in), this is a non-issue — your app's session is independent of the OAuth token. This is the case for most SaaS apps.

If you DO access provider APIs post-sign-in (e.g., Google Calendar integration): store the provider's refresh token encrypted in your database, use it to get new access tokens when they expire, and handle revocation gracefully (prompt the user to re-link if their refresh token is rejected).

### 14. Route Protection & Access Control

Auth enforcement happens at **four layers**. You need all four — they're not alternatives, they're defense in depth.

```
Request
  │
  ▼
┌─────────────────────────────────┐
│  Layer 1: Middleware             │  ← Redirect unauthenticated users
│  (runs before any page renders) │     before they hit your app
└───────────────┬─────────────────┘
                ▼
┌─────────────────────────────────┐
│  Layer 2: Server Component      │  ← Fetch user, pass to components
│  (runs on each page load)       │     404/redirect if unauthorized
└───────────────┬─────────────────┘
                ▼
┌─────────────────────────────────┐
│  Layer 3: API / Server Actions  │  ← Validate session on every mutation
│  (runs on every data request)   │     Never trust the client
└───────────────┬─────────────────┘
                ▼
┌─────────────────────────────────┐
│  Layer 4: Database (RLS)        │  ← Even if API has a bug,
│  (runs on every query)          │     users can't see others' data
└─────────────────────────────────┘
```

#### Layer 1: Middleware

Middleware intercepts every request before it hits your pages. Use it for broad auth gates — not fine-grained permissions.

**What it does:**
- Check if a session cookie exists
- Redirect unauthenticated users to the sign-in page
- Redirect authenticated users away from sign-in and sign-up pages
- Refresh expired tokens (handled by Better Auth's Next.js integration)
- Preserve the full redirect context so the user continues where they left off after sign-in

**What it should NOT do:**
- Database lookups (middleware runs at the edge, latency to DB is high)
- Fine-grained permission checks (too slow, do this in Server Components)
- Heavy computation

**Design decisions:**
- Maintain an explicit list of public routes (landing page, sign-in, sign-up, reset password, pricing, blog, webhook endpoints)
- Everything not on the public list requires auth by default (deny by default, allow by exception)
- Middleware matcher should exclude static files and assets to avoid unnecessary auth checks on images, fonts, etc.

**Redirect context — preserving the user's intent across the login wall:**

When an unauthenticated user is redirected to sign-in, their full intent must survive the interruption. Redirect them back to exactly what they were trying to do — not just the URL, but the action.

- Store the destination as a `redirectTo` query param on the sign-in URL: `/sign-in?redirectTo=/dashboard/projects`
- Include all query params in the redirectTo value — a user selecting a plan at `/pricing?plan=pro` should land back at `/pricing?plan=pro` after sign-in, with the plan pre-selected
- After successful sign-in or sign-up, read `redirectTo` and navigate there — never hardcode a post-auth destination
- If no `redirectTo` is present, fall back to the default dashboard
- Validate the `redirectTo` value before redirecting — only allow relative URLs that belong to your app (prevent open redirect attacks)
- This same mechanism handles cross-system flows:
  - User clicks "Choose Plan" → not logged in → login → returns to checkout with plan pre-selected
  - User clicks an invite link → not signed up → registers → lands in the correct workspace
  - User clicks a shared resource link → not logged in → login → sees the resource, not the generic dashboard
  - User's session expires mid-flow → re-authenticates → continues from where they were

#### Layer 2: Server Components

Middleware confirms a session exists. Server Components fetch the actual user and handle authorization (roles, permissions, ownership).

**Design decisions:**
- Create reusable auth helper functions: one for "require any authenticated user" and one for "require specific role(s)"
- These helpers should fetch the user, check the condition, and redirect if it fails — so each page is a one-liner call
- The helpers return the authenticated user object so the page can use it without re-fetching
- Fine-grained checks (e.g., "is this user a member of this workspace?", "does this user own this resource?") happen here, not in middleware

#### Layer 3: API Routes & Server Actions

Every mutation must re-validate auth independently. Never trust that middleware already checked — server actions and API routes can be called directly by anyone with the URL.

**Design decisions:**
- Every API route and server action starts by validating the session. No exceptions.
- Return 401 Unauthorized for unauthenticated requests, 403 Forbidden for authenticated but unauthorized
- Webhook endpoints (Stripe, Resend, etc.) use signature verification instead of session auth — they are explicitly excluded from session-based middleware
- Always set the owner/creator ID server-side from the authenticated session, never accept it from the client request body

#### Layer 4: Database (RLS)

The last line of defense. Even if your API code has a bug that skips the auth check, RLS prevents data leakage at the database level.

**Design decisions:**
- Every table with user data must have RLS enabled
- Default policy: deny all. Then add explicit allow policies per operation (SELECT, INSERT, UPDATE, DELETE)
- For single-user data: policy checks that the row's owner_id matches the authenticated user
- For workspace/team data: policy checks that the authenticated user is a member of the workspace the row belongs to
- For role-restricted operations (e.g., delete): policy additionally checks that the user's role in that workspace is admin or owner
- RLS is not optional. It is the safety net for every other layer.

#### Client-Side (UI only, never for security)

Client-side auth checks are for UX only — hiding buttons, showing/hiding navigation items, conditional rendering.

**Design decisions:**
- Expose the authenticated user and their role to client components via a hook or context
- Use this to conditionally render UI elements (hide delete button for non-admins, show upgrade prompts for free users)
- Never rely on this for actual security — all enforcement happens server-side
- If a user manipulates the client to show a hidden button, the server and RLS will still reject the action

#### Route Map Convention

Every project should maintain an explicit route map documenting which routes require what level of auth:

**Public (no auth required):**
- Landing page, sign-in, sign-up, password reset, pricing, blog, legal pages
- Webhook endpoints (verified by signature, not session)

**Authenticated (any signed-in user):**
- Dashboard, project pages, profile settings, billing management

**Role-restricted (specific roles only):**
- Workspace settings (admin/owner), member management (admin/owner), admin dashboard (owner)

**API rules:**
- All mutation endpoints (POST, PUT, DELETE) require auth
- Most read endpoints require auth (except explicitly public data)
- Webhook endpoints use signature verification, not session auth

## Providers

### Neon Auth (recommended)

**Best for:** all projects. Managed auth built on Better Auth, stores auth data directly in your Neon Postgres database.

**AI interface:** Neon CLI (`neonctl`), Neon MCP server.

| Aspect | Details |
|---|---|
| **Type** | Managed, built on Better Auth |
| **Cost** | Included with Neon |
| **Session strategy** | Database sessions (Better Auth default, configurable) |
| **OAuth providers** | 20+ built-in via Better Auth (Google, GitHub, Apple, Microsoft, Discord, etc.) |
| **2FA** | TOTP support via Better Auth |
| **Email verification** | Built-in |
| **RLS integration** | Standard Postgres RLS — application sets session context (e.g., `set_config('app.user_id', ...)`) and RLS policies reference `current_setting('app.user_id')` |
| **Next.js support** | First-class, App Router native via Better Auth's Next.js integration |
| **Pre-built UI** | None — you build your own forms |
| **Vendor lock-in** | Low — all data in your Neon Postgres database, auth layer is standard Better Auth schema |
| **Branch-compatible** | Yes — auth data branches with your database for preview environments |
| **TypeScript** | Full inference via Better Auth, no manual type augmentation |

**Strengths:**
- Managed service — no auth infrastructure to maintain
- Auth data lives in your database, not a separate service — query users, sessions, and accounts with standard SQL
- Branch-compatible — preview environments get their own auth state automatically
- Built on Better Auth — same APIs, same schema, same TypeScript inference
- Auth.js (NextAuth) team joined Better Auth in Sept 2025 — it's the recommended replacement for NextAuth

**Weaknesses:**
- No pre-built UI components (you build every form)
- Does not yet support custom Better Auth plugins — if you need plugins beyond what Neon Auth exposes, self-host Better Auth instead
- Newer managed offering, fewer tutorials than established providers

### Better Auth (self-hosted, open source)

**Best for:** projects that need custom Better Auth plugins that Neon Auth doesn't support yet.

| Aspect | Details |
|---|---|
| **Type** | Self-hosted, MIT licensed |
| **Cost** | Free forever |
| **Session strategy** | Database sessions (configurable) |
| **OAuth providers** | 20+ built-in (Google, GitHub, Apple, Microsoft, Discord, etc.) |
| **2FA** | Plugin (TOTP, recovery codes) |
| **Email verification** | Built-in |
| **RLS integration** | Standard Postgres RLS — same pattern as Neon Auth (application-set session context) |
| **Next.js support** | First-class, App Router native |
| **Pre-built UI** | None — you build your own forms |
| **Vendor lock-in** | None — all data in your database |
| **TypeScript** | Full inference, no manual type augmentation |

**Strengths:**
- Full plugin ecosystem (2FA, organizations, passkeys, custom plugins)
- ~100K weekly downloads by March 2026, fastest-growing auth library
- Full TypeScript inference (session types auto-inferred from config)
- All user data in your own database, zero vendor lock-in
- Works with any database (Postgres, MySQL, SQLite via Drizzle/Prisma adapters)

**Weaknesses:**
- You manage the auth infrastructure (deployment, email sending configuration)
- No managed dashboard (you build admin tooling yourself)

## Which Provider to Choose

- **Default** → Neon Auth. Managed, branch-compatible, data in your database, built on Better Auth. Use this unless you have a specific reason not to.
- **Need custom Better Auth plugins** → Better Auth self-hosted against Neon. Same database, same schema, full plugin access. Migrate to Neon Auth later when plugin support catches up.

## Security Checklist

- [ ] Passwords hashed with bcrypt or argon2 (never MD5/SHA)
- [ ] Rate limiting on sign-in endpoint (prevent brute force)
- [ ] Rate limiting on password reset (prevent email bombing)
- [ ] Session tokens in HttpOnly, Secure, SameSite cookies
- [ ] No JWTs in localStorage
- [ ] CSRF protection on auth forms
- [ ] Password reset tokens expire (1 hour max)
- [ ] Password reset invalidates all existing sessions
- [ ] Email verification required (or flagged as unverified)
- [ ] OAuth state parameter validated (prevent CSRF on OAuth)
- [ ] Account enumeration prevented ("if account exists" language on reset)
- [ ] Suspicious login detection (new device/location → email alert)
- [ ] Session revocation possible (sign out all devices)
- [ ] 2FA available (not required, but offered)
- [ ] Recovery codes generated when 2FA is enabled
- [ ] Refresh token rotation enabled (new token on each refresh)
- [ ] Absolute session expiry (even active sessions expire after X days)

## Implementation Checklist

### Core Auth
- [ ] Auth provider chosen and configured
- [ ] Email + password sign up/sign in working
- [ ] Google OAuth configured, consent screen submitted for verification
- [ ] Apple Sign-In configured (if iOS app exists or planned)
- [ ] GitHub OAuth configured (if developer audience)
- [ ] Facebook Login configured (if B2C consumer-facing)
- [ ] Apple relay email addresses handled correctly
- [ ] Apple first-sign-in name capture implemented (name only sent once)
- [ ] GitHub private email handled (use primary + verified email from API)
- [ ] Facebook missing email handled (prompt user to provide email if not returned)
- [ ] Email verification flow working (with banner for unverified users, resend option)
- [ ] Password reset flow working (via email link)
- [ ] Password change flow working (from settings, requires current password)
- [ ] Email change flow working (verify new email before swap, notify old email)
- [ ] 2FA setup flow (enable, verify, recovery codes)

### Sessions
- [ ] Session management working (create, validate, destroy)
- [ ] Sign out (single device + all devices)
- [ ] Active sessions visible in user security settings
- [ ] Concurrent session limit enforced (oldest session destroyed when limit exceeded)
- [ ] Absolute session expiry configured (30 days max)
- [ ] Re-authentication required for sensitive actions (password change, email change, account deletion)

### Route Protection
- [ ] Route protection middleware in place
- [ ] Server-side auth check in Server Components via reusable helpers
- [ ] API routes and server actions independently validate auth
- [ ] RLS policies on all user-data tables
- [ ] Client-side auth state available via hook/context (for UX only)
- [ ] Route map documented (public vs authenticated vs role-restricted)

### Account Management
- [ ] Account linking handled (same email across password + OAuth)
- [ ] Linked identities visible and manageable in user profile settings
- [ ] Account deletion flow working (self-service, with re-authentication)
- [ ] Deletion handles: Stripe cancellation, session destruction, data cleanup, shared content anonymization
- [ ] Deletion grace period defined and implemented (if applicable)
- [ ] Workspace ownership transfer required before owner can delete account

### Security
- [ ] Rate limiting on sign-in (per email + per IP)
- [ ] Rate limiting on password reset (per email + per IP)
- [ ] Rate limiting on sign-up (per IP)
- [ ] Auth-related emails using custom templates (Resend + React Email)
- [ ] Error messages don't leak user existence
- [ ] OAuth refresh tokens stored encrypted (if accessing provider APIs post-sign-in)
- [ ] All auth flows tested (unit + integration + E2E)

## What to Test

### Registration
- Creates a new user account with valid email and password
- Rejects registration if email already exists — returns identical response as success (no enumeration)
- Rejects registration if password fails strength requirements
- Sends verification email immediately after registration
- Creates account in unverified state — grants access with persistent banner, not a hard block
- Rate limits sign-up to 5 attempts per IP per hour

### Email Verification
- Marks account as verified when valid token is clicked
- Rejects expired verification token and shows resend option
- Rejects already-used verification token
- Rate limits resend to 3 per hour per email
- Restricts account to read-only access after 7 days unverified

### Sign In
- Creates a session for valid email and password
- Rejects wrong password without revealing whether the email exists
- Rejects non-existent email with the same response as wrong password
- Locks email after 5 failed attempts for 15 minutes
- Resets failure counter on successful sign in
- Blocks IP after 15 total failures across any email within 1 hour

### Password Reset
- Sends reset email for an existing account
- Returns same response for non-existent email — no enumeration
- Rejects expired reset token with clear message and retry option
- Rejects already-used reset token
- Invalidates ALL active sessions after successful password reset
- Rate limits to 3 reset requests per email per hour
- Invalidates previous reset tokens when a new one is issued

### Password Change (authenticated)
- Changes password successfully with correct current password
- Rejects change if current password is wrong
- Rejects if new password is identical to current password
- Invalidates all OTHER sessions — keeps current session active
- Sends notification email after password change

### Email Change
- Requires re-authentication before initiating change
- Sends verification link to new email — change does not apply until verified
- Does not apply change if new email belongs to another account
- Sends notification to OLD email when change is requested
- Invalidates ALL sessions after email change is confirmed

### OAuth
- Creates new account on first OAuth sign-in
- Signs in existing OAuth user without creating a duplicate account
- Does not auto-create second account when OAuth email matches an existing password account — prompts to link instead
- Rejects OAuth callback with mismatched state parameter
- Handles Apple relay email (privaterelay.appleid.com) as a valid email — does not reject or prompt for real email
- Stores Apple user name on first sign-in — absent on subsequent sign-ins (only sent once by Apple)
- Fetches verified primary email from GitHub emails endpoint when profile email is null
- Redirects Facebook user to complete-profile step when no email is returned

### 2FA
- Blocks session creation until TOTP code is verified after correct password
- Creates session after correct TOTP code is entered
- Rejects incorrect TOTP code
- Allows sign in with a valid recovery code when authenticator is unavailable
- Burns recovery code after single use — cannot be reused
- Generates exactly 8 recovery codes on 2FA setup
- Invalidates all previous recovery codes when regenerated

### Sessions
- Refreshes access token using refresh token before expiry
- Destroys session on sign out
- Destroys oldest session when the concurrent session limit is exceeded
- Signs out all other devices while keeping the current session active
- Enforces absolute 30-day session expiry regardless of activity
- Requires re-authentication for sensitive actions: password change, email change, account deletion
- Active sessions list is visible in user security settings

### Route Protection
- Redirects unauthenticated user to sign-in page
- Redirects signed-in user away from /sign-in and /sign-up
- Returns 401 for unauthenticated API and server action calls
- Returns 403 for authenticated but unauthorized API calls
- Rejects any request that passes a user ID in the request body — ID must come from session
- Server actions independently re-validate auth — middleware passing is not sufficient
- Webhook endpoints bypass session auth and use signature verification instead

### Cross-System Flows (redirect context)
- User clicks "Choose Plan" while logged out → after login, returns to pricing with the same plan pre-selected, not to the generic dashboard
- User visits a protected page while logged out → after login, lands on that exact page, not the dashboard
- User clicks an invite link while not signed up → after registration, lands in the correct workspace, not the homepage
- User clicks a shared resource link while logged out → after login, sees that resource directly
- User's session expires while navigating the app → after re-authentication, returns to the page they were on
- redirectTo param containing an external URL is rejected — only relative app URLs are followed
- If no redirectTo is present after login, user lands on the default dashboard

### RBAC
- Returns 403 when member attempts an owner-only action
- Returns 403 when viewer attempts a write action
- Enforces role per workspace — admin in one workspace does not grant admin in another
- RLS blocks query from returning rows belonging to a different workspace

### Account Deletion
- Cancels Stripe subscription on deletion
- Destroys all sessions across all devices on deletion
- Anonymizes shared workspace content — replaces author name with "Deleted User"
- Requires re-authentication before deletion proceeds
- Blocks deletion if user is workspace owner — requires ownership transfer first
- Reactivates account and cancels pending deletion on sign-in during grace period

### Invitations
- Invite sent to existing user adds them to workspace immediately — no sign-up flow
- Invite sent to new user auto-joins workspace after account creation
- Invite token expires after 7 days

## Edge Cases

- **Duplicate email on sign-up** — show the same "check your email to verify" message as a successful sign-up. Never reveal whether the email already exists.
- **Expired password reset link** — show a clear "this link has expired" message with a prominent "request new link" button. Never a generic error.
- **OAuth provider temporarily down** — catch the redirect failure, show "Sign in with Google is temporarily unavailable, please use email and password" with a retry option.
- **Browser back button after sign-out** — set `Cache-Control: no-store` on all authenticated pages so the browser doesn't serve cached versions after sign-out.
- **Multiple tabs with different auth states** — use a broadcast channel or storage event listener to sync sign-out across all open tabs. When one tab signs out, all tabs redirect to sign-in.
- **Email change while other sessions active** — all other sessions should pick up the new email on their next token refresh. Force a session refresh rather than letting stale user data persist.
- **Invite sent to existing user** — detect the email exists, add them to the workspace immediately, send a notification email ("you've been added to X"). No sign-up required.
- **Invite sent to non-existent user** — send an invite email with a sign-up link that auto-joins them to the workspace after account creation. Invite tokens should expire after 7 days.
- **OAuth access revoked externally** — user revokes your app in their Google/GitHub settings. On next session refresh, detect the broken OAuth link, keep the user signed in (your session is independent), but mark the OAuth identity as disconnected in their settings.
- **Concurrent password reset requests** — user clicks reset twice, gets two emails. Both tokens are valid, but using either one invalidates all other outstanding reset tokens.
- **Sign-in during account deletion grace period** — if you offer a grace period, signing in during it should reactivate the account, cancel the pending deletion, and notify the user that their account has been restored.
- **User changes email to one that already exists** — reject the change. Show "this email is already associated with another account" (this is one place where revealing existence is acceptable because it's behind an authenticated action).
- **Expired session on form submission** — user fills out a long form, session expires before they submit. Catch the 401, redirect to sign-in with the redirect URL preserved, and ideally preserve the form data (or at minimum warn them before navigating away).
- **OAuth callback with mismatched state parameter** — reject the request entirely. This indicates a CSRF attack or a stale/replayed OAuth flow. Redirect to sign-in with a generic error message.
- **User signs up with OAuth, later tries email+password with same email** — prompt them to sign in with their OAuth provider. Offer to set a password from their settings after signing in, so they can use both methods going forward.
- **Apple user hides their email** — your app receives a relay address like abc123@privaterelay.appleid.com. Treat it as a real email — it forwards to the user. Do not reject it, do not flag it as invalid, do not ask them for their "real" email.
- **Apple only sends the user's name once** — if your OAuth callback fails to store the name on the first sign-in, you will never receive it again. Ensure your callback persists the name before any redirect or error handling.
- **Facebook user has no email** — some Facebook users sign up with a phone number only. Your app receives no email from the OAuth flow. Redirect them to a "complete your profile" step that requires an email before they can continue.
- **GitHub user has private email** — the default profile endpoint may return null for email. Make a separate API call to the emails endpoint, filter for the one marked primary and verified.
- **Google consent screen not verified** — unverified apps show a "This app isn't verified" warning with a scary full-screen interstitial. Users will abandon sign-up. Submit for verification 2-4 weeks before launch.
- **OAuth callback URL mismatch after domain change** — you change your domain or add www but forget to update the callback URL in the provider's console. OAuth breaks silently. Update callback URLs in ALL providers whenever the domain changes.

## Common Mistakes

- Storing JWTs in localStorage (XSS attack vector)
- Not rate-limiting sign-in (brute force attacks)
- Revealing whether an email exists on login failure ("incorrect password" vs "user not found")
- Not invalidating sessions on password reset (old sessions stay active)
- Skipping email verification (fake accounts, spam)
- Building custom auth from scratch (unless you really know what you're doing)
- Not testing the full lifecycle (sign up → verify → sign in → reset → sign out → delete)
- Assuming OAuth "just works" (tokens expire, providers change APIs, callback URLs break)
- Not planning for account linking (user signs up with email, later tries Google with same email)
- Only enforcing auth at one layer (middleware alone is not enough — you need all four layers)
- Accepting user IDs from client requests instead of deriving them from the server-side session
- Forgetting to enable RLS on new tables (one missed table = full data exposure)
- Allowing email changes without verifying the new email first (account takeover risk)
- Not invalidating sessions after password change (old sessions on compromised devices stay active)
- Treating role-based access as sufficient when resource-level permissions are needed (a "member" who owns a project has different rights than a "member" who doesn't)
