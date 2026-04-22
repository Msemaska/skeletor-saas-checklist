# Pre-Launch Security Audit

---

## What It Is

A structured verification that every security decision made during development was actually implemented correctly. This is not new work — it's a checklist walk-through before real users touch the product. Run this audit after all features are built and before the product goes live.

## Audit Process

Walk through each category below. For each item, verify the implementation matches the design decision from the referenced phase file. If something is missing or incorrect, fix it before launch.

## 1. Authentication

Reference: phase-3-core-product/03-auth.md

**Verify:**
- [ ] Sign-up, sign-in, password reset, OAuth, and 2FA flows all work end-to-end
- [ ] Duplicate email on sign-up returns the same response as a successful sign-up (no account enumeration)
- [ ] Password reset tokens expire after 1 hour
- [ ] Password reset invalidates all existing sessions
- [ ] Password change invalidates all other sessions (keeps current)
- [ ] Email change verifies the new email before swapping, notifies the old email
- [ ] Rate limiting is active on sign-in (5 attempts per email per 15 minutes)
- [ ] Rate limiting is active on password reset (3 requests per email per hour)
- [ ] Rate limiting is active on sign-up (5 attempts per IP per hour)
- [ ] 2FA challenge blocks session creation until TOTP is verified
- [ ] Session tokens are in HttpOnly, Secure, SameSite=Lax cookies (not localStorage)
- [ ] Absolute session expiry is configured (30 days max)
- [ ] Account deletion flow works end-to-end (Stripe cancellation, session destruction, data cleanup)

## 2. Authorization & Data Access

Reference: phase-3-core-product/03-auth.md (Route Protection), phase-3-core-product/01-database-schema.md (RLS)

**Verify:**
- [ ] Middleware redirects unauthenticated users on all protected routes
- [ ] Middleware redirects authenticated users away from sign-in/sign-up pages
- [ ] Every Server Action validates auth before any business logic
- [ ] Every API route validates auth before any business logic
- [ ] RLS is enabled on every table with user data
- [ ] Cross-workspace data isolation: user in workspace A cannot read workspace B's data (test this explicitly)
- [ ] Role-restricted operations enforce role checks (admin/owner only actions reject member/viewer)
- [ ] RLS UPDATE policies restrict to USER columns only (SYSTEM columns cannot be modified via user context)
- [ ] Soft-deleted records are invisible through RLS SELECT policies
- [ ] Owner/creator ID is always set server-side from the session, never accepted from client requests

## 3. Input Validation & Security

Reference: phase-3-core-product/02-validation.md (Input Security section)

**Verify:**
- [ ] Every Server Action and API route validates input with Zod before any business logic
- [ ] Zod schemas strip unknown fields (mass assignment protection)
- [ ] No `dangerouslySetInnerHTML` without DOMPurify sanitization
- [ ] All database queries use parameterized queries (no string concatenation of user input into SQL)
- [ ] File uploads validate file type by magic bytes (not just extension or Content-Type header)
- [ ] File uploads enforce size limits (10MB for images, 50MB for documents)
- [ ] Filenames are generated server-side (client-provided filenames are not used for storage paths)
- [ ] Request body size limits are configured per endpoint

## 4. Session & Cookie Security

Reference: phase-3-core-product/03-auth.md (Session Management)

**Verify:**
- [ ] All cookies are HttpOnly (not accessible via JavaScript)
- [ ] All cookies are Secure (only sent over HTTPS)
- [ ] All cookies are SameSite=Lax (CSRF protection)
- [ ] No sensitive data in localStorage (tokens, keys, session data)
- [ ] Session revocation works (sign out all devices)

## 5. API & Webhook Security

Reference: phase-8/02-stripe-architecture.md, phase-3-core-product/04-data-access.md

**Verify:**
- [ ] Stripe webhook signatures are verified on every webhook request
- [ ] Webhook deduplication table prevents duplicate event processing
- [ ] Every webhook handler is idempotent
- [ ] API routes return 401 for unauthenticated requests, 403 for unauthorized
- [ ] No internal error details (stack traces, database errors) reach the client
- [ ] Error responses use the consistent `{ success, error, fields }` shape

## 6. Security Headers

Reference: phase-7-infrastructure/05-security-headers.md

**Verify:**
- [ ] Strict-Transport-Security header present (max-age=31536000, includeSubDomains, preload)
- [ ] X-Content-Type-Options: nosniff header present
- [ ] X-Frame-Options: DENY header present
- [ ] Referrer-Policy: strict-origin-when-cross-origin header present
- [ ] Permissions-Policy disables unused browser features (camera, microphone, geolocation)
- [ ] Content-Security-Policy configured and not breaking third-party integrations
- [ ] CSP violations reported to Sentry

## 7. Secrets & Environment Variables

Reference: phase-7-infrastructure/07-local-development.md (Environment Variables)

**Verify:**
- [ ] No API keys, tokens, or secrets in the codebase (search the git history, not just the current code)
- [ ] All secrets are in environment variables, validated by Zod at startup
- [ ] `.env` files are in `.gitignore`
- [ ] Production environment variables are different from staging/development
- [ ] Stripe live keys are only in the production environment (not staging)
- [ ] Service role / admin database credentials are only used in system context (never exposed to the client)

## 8. Dependencies

Reference: phase-7-infrastructure/06-dependency-scanning.md

**Verify:**
- [ ] `npm audit` shows no critical or high severity vulnerabilities
- [ ] Dependabot is enabled and security update PRs are being reviewed
- [ ] No unused dependencies in package.json

## 9. Infrastructure

Reference: phase-7-infrastructure/03-server-provisioning.md

**Railway:**
- [ ] Environment variable scoping is correct (production keys not in preview deployments)
- [ ] Spending limit configured

**Self-hosted — port exposure audit:**
- [ ] Only ports 80 (HTTP), 443 (HTTPS), and SSH (custom port) are open to the internet
- [ ] PostgreSQL port (5432) is NOT exposed to the internet (Docker network only)
- [ ] Redis/Valkey port (6379) is NOT exposed to the internet (Docker network only)
- [ ] PgBouncer port is NOT exposed to the internet (Docker network only)
- [ ] MinIO/storage ports are NOT exposed to the internet (Docker network only)
- [ ] All internal services communicate via Docker network, not host-bound ports
- [ ] Run `nmap` or `ss -tlnp` against the server's public IP to verify only expected ports are open
- [ ] Firewall (UFW) rules reviewed — deny all incoming except SSH, HTTP, HTTPS
- [ ] Redis/Valkey requires a password even on Docker network (defense in depth)

## 10. security.txt

**Verify:**
- [ ] `/.well-known/security.txt` exists
- [ ] Includes contact email (hello@yourdomain.com), preferred language (English), acknowledgment policy
- [ ] Linked from the marketing site footer

## Penetration Testing

**Verify:**
- [ ] Every form tested with unexpected input (script tags, SQL fragments, extremely long strings, empty required fields)
- [ ] Every API route tested without authentication (curl the endpoints without cookies)
- [ ] Cross-workspace data access tested by manipulating IDs in the URL
- [ ] SYSTEM column modification tested (plan, credits) via the API by adding extra fields to request bodies
- [ ] File uploads tested with mismatched extensions and content types
- [ ] All the above are correctly rejected

## Edge Cases

- **Security fix needed after audit** — fix it before launch. Never launch with known security issues.
- **Audit reveals a design flaw, not just a missing implementation** — data exposure risks are launch blockers. Everything else can be addressed post-launch.
- **Third-party service has a known vulnerability** — update the SDK or add an additional check before launch.

## Common Mistakes

- Skipping the security audit
- Auditing only the happy path
- No security.txt
- Searching only current code for secrets instead of full git history
- Launching with known critical/high npm vulnerabilities
- Auditing once and never again
