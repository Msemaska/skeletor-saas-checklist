# Validation Strategy

---

## What It Is

The system that ensures data entering your application is correct, safe, and consistent — at every boundary. Validation is not a single layer. It's three layers working together, each with a different purpose and a different failure mode.

## Three Layers

```
User Input
  │
  ▼
┌─────────────────────────────────┐
│  Layer 1: Frontend              │  ← Instant feedback, UX
│  (runs in the browser)          │     Catches typos and formatting
└───────────────┬─────────────────┘
                ▼
┌─────────────────────────────────┐
│  Layer 2: Backend               │  ← The real enforcement
│  (server actions / API routes)  │     Never trust the client
└───────────────┬─────────────────┘
                ▼
┌─────────────────────────────────┐
│  Layer 3: Database              │  ← Safety net
│  (check constraints, types)     │     Last line of defense
└─────────────────────────────────┘
```

All three are required. They are not alternatives.

### Layer 1: Frontend (UX, not security)

Validation in the browser is for the user's benefit — instant feedback, inline error messages, preventing form submission with obviously bad data.

**Design decisions:**
- Use Zod schemas with React Hook Form for all form validation
- Define the Zod schema first, derive the TypeScript type with `z.infer<>` — never duplicate types manually
- Show validation errors inline next to the field, not as a banner or toast
- Validate on blur (when the user leaves the field) for individual fields, validate all on submit
- Disable the submit button while the form is invalid or submitting
- Frontend validation is trivially bypassable — anyone can open DevTools and POST directly to your API. Never rely on it for security or data integrity.

### Layer 2: Backend (the real enforcement)

Every server action and API route must validate its input independently. This is the layer that actually protects your application.

**Design decisions:**
- Re-validate with the same Zod schema used on the frontend. One schema, two places.
- Parse, don't validate: use `schema.parse(input)` which throws on invalid data, not `schema.safeParse()` in contexts where invalid data should halt execution
- Validate at the entry point of every server action and API route — before any business logic runs
- Return structured error responses (field-level errors that the frontend can map back to form fields)
- Validate webhook payloads from external services (Stripe, Resend) against expected shapes before processing
- Validate URL parameters and query strings — they're user input too
- Trim whitespace on string inputs before validation (a name that's all spaces should fail "required" checks)
- HTML sanitization is handled in the Input Security section below — do not sanitize at this layer blindly, see XSS defense for the correct approach

### Layer 3: Database (safety net)

Check constraints and type enforcement at the database level catch anything that slips through the application layer — bugs, direct database access, system context operations, migration scripts.

**Design decisions:**
- Add check constraints for rules that must never be violated: non-negative numbers, valid enum values, non-empty required strings, value ranges
- Use Postgres types to enforce structure: `uuid` for IDs, `timestamptz` for timestamps, `boolean` for booleans, `numeric` for money
- NOT NULL on columns that should always have a value
- UNIQUE constraints where duplicates would cause data corruption
- Foreign key constraints with appropriate ON DELETE behavior
- Keep constraints simple — complex business logic belongs in the application layer, not in SQL check constraints

## Schema as Single Source of Truth

Define each Zod schema once. Use it everywhere:

- **Form validation** — React Hook Form resolver uses the schema
- **Server action validation** — parse input against the same schema
- **API route validation** — parse request body against the same schema
- **Type inference** — `z.infer<typeof schema>` generates the TypeScript type

Never define a TypeScript type and a Zod schema separately for the same data. They will drift apart.

## What to Validate Where

| Check | Frontend | Backend | Database |
|---|---|---|---|
| Required fields | Yes | Yes | NOT NULL |
| String length limits | Yes | Yes | CHECK constraint |
| Email format | Yes | Yes | — |
| Number ranges (e.g., credits >= 0) | Yes | Yes | CHECK constraint |
| Enum values (status, role) | Yes | Yes | CHECK constraint |
| Unique values (email, slug) | No (check via API) | Yes (query + catch constraint error) | UNIQUE constraint |
| Foreign key existence | No | Yes (query) | FOREIGN KEY constraint |
| Auth (is user signed in) | No | Yes | RLS policy |
| Authorization (can user do this) | No | Yes | RLS policy |
| XSS / HTML sanitization | No | Yes | — |
| File type / size | Yes (UX) | Yes (enforce) | — |
| CSRF | No | Yes (token) | — |

**The pattern:** frontend catches the obvious stuff for UX, backend enforces everything for security, database constrains the data model for integrity.

## Input Security

Every piece of user input is a potential attack vector. These are the attacks that happen through form fields, URL parameters, API payloads, and file uploads — and the validation-layer defenses against them.

### XSS (Cross-Site Scripting)

An attacker injects a script into a field (e.g., name: `<script>steal(cookies)</script>`), which then executes in another user's browser when that data is rendered.

**Where it happens:** any user-provided text that gets rendered in the UI — names, comments, messages, descriptions, workspace names, even filenames.

**Defense:**
- **React handles most of it** — React escapes all content rendered via JSX by default. A string containing `<script>` is rendered as literal text, not executed. This covers the majority of XSS vectors.
- **The danger is `dangerouslySetInnerHTML`** — if you ever render raw HTML (rich text editors, markdown rendering, email templates), you must sanitize it first. Use a library like DOMPurify to strip dangerous tags and attributes before rendering.
- **Sanitize on output, not on input** — store the original text as-is, sanitize when rendering. This preserves the original data and lets you improve sanitization rules later without re-processing stored data.
- **Content Security Policy (CSP) headers** — configured at the infrastructure level (Phase 6), CSP blocks inline scripts even if XSS sanitization fails. It's a defense-in-depth layer.
- **Never construct HTML from user input server-side** — if you're building HTML strings (for emails, PDFs, etc.), use a templating engine that auto-escapes, not string concatenation.

### SQL Injection

An attacker crafts input that alters a SQL query (e.g., `'; DROP TABLE users; --`).

**Defense:**
- **Use parameterized queries exclusively** — every database client and ORM (Supabase client, Drizzle, Prisma) uses parameterized queries by default. As long as you use the client's query builder and never construct SQL strings from user input, SQL injection is not possible.
- **Never concatenate user input into SQL strings** — not in application code, not in database functions, not in migration scripts that accept parameters.
- **RLS policies are safe** — they reference the authenticated user's session function, not user input.
- **This is a solved problem** — if you're using modern tooling correctly, SQL injection requires actively doing something wrong (string concatenation). The rule is simple: never build SQL from strings.

### CSRF (Cross-Site Request Forgery)

An attacker tricks a signed-in user's browser into making a request to your API (e.g., a hidden form on a malicious site that POSTs to your /api/delete-account endpoint).

**Defense:**
- **SameSite cookies** — session cookies set with `SameSite=Lax` (our auth recommendation) prevent cross-origin requests from including the cookie. This blocks most CSRF attacks.
- **Server Actions are CSRF-protected by default** — Next.js Server Actions include automatic CSRF protection via origin checking.
- **For API routes** — verify the `Origin` or `Referer` header matches your domain on state-changing requests (POST, PUT, DELETE).
- **For forms that don't use Server Actions** — include a CSRF token (generated server-side, validated on submission). Most CSRF libraries handle this automatically.

### Mass Assignment / Over-Posting / JSON Injection

An attacker sends extra or unexpected fields in a request, hoping the backend blindly saves them (e.g., adding `plan: "enterprise"` or `role: "admin"` to a profile update).

**Defense:**
- **Zod parsing strips unknown fields** — this is the primary defense. Define exactly which fields are accepted, parse the input, use only the parsed output. Extra fields are silently discarded.
- **Never spread raw request body into database queries** — always parse through Zod first, then use the parsed output.
- **Column ownership classification** — even if extra fields somehow reach the database, RLS UPDATE policies should only allow USER columns to be modified. SYSTEM columns like `plan`, `credits`, `stripe_customer_id` are never writable through user context.
- **For jsonb columns** — validate the structure with Zod before storing. Don't accept arbitrary JSON from the client into a jsonb field.

### Path Traversal / File Upload Attacks

An attacker manipulates filenames or paths to access files outside the intended directory (e.g., filename: `../../../etc/passwd`), or uploads a file with a malicious payload disguised as an image.

**Defense:**
- **Generate filenames server-side** — never use the client-provided filename for storage. Generate a UUID or hash-based filename and store the original name as metadata.
- **Validate file type on backend** — check both the MIME type AND the file's magic bytes (first few bytes that identify the file type). Don't trust the client-reported Content-Type header alone.
- **Enforce file size limits** — at the application level and at the storage/infrastructure level.
- **Store uploads outside the application directory** — use object storage (Supabase Storage, S3, R2), never the local filesystem.

### Rate Limiting on All Input Endpoints

Not just auth endpoints — any endpoint that accepts user input can be abused.

**Defense:**
- **Auth endpoints** — covered in detail in the auth doc (5 attempts per email per 15 min, etc.)
- **Form submissions** — limit how frequently a user can submit any form (e.g., max 10 submissions per minute per user). Prevents spam and abuse.
- **File uploads** — limit upload frequency and total storage per workspace/user.
- **API endpoints** — if you have a public or semi-public API, rate limit per API key or per user.
- **Search endpoints** — expensive queries can be used for denial-of-service. Limit search frequency and result set sizes.

### Input Length as a Security Concern

Extremely long inputs can cause performance issues, memory exhaustion, or denial-of-service even if they're otherwise "valid."

**Defense:**
- **Every text field has a max length** — enforced at all three layers (frontend, Zod, database CHECK constraint)
- **Request body size limit** — configure your web server / hosting to reject oversized request bodies (Next.js defaults to 1MB for API routes — adjust per endpoint if needed)
- **jsonb fields** — validate maximum size before storing. An unrestricted jsonb field can accept megabytes of data.

## Common Field Validations

These are the standard fields that appear in almost every SaaS. The rules below apply across all three layers unless noted otherwise.

### Email

- Required, trimmed, lowercased before validation
- Must be a valid email format
- Max 255 characters
- Unique per user (enforced by auth provider + UNIQUE constraint)
- Allow `+` aliases (user+tag@gmail.com) — these are valid emails and useful for testing
- **Frontend:** inline format check, show error on blur
- **Backend:** required string, trimmed, lowercased, validated as email format, max 255 chars
- **Database:** text, NOT NULL, UNIQUE constraint (handled by auth provider's users table)

### Password

- Required, minimum 8 characters, max 128 characters (cap to prevent abuse, but let password managers generate long ones)
- At least one uppercase letter, one lowercase letter, one digit
- Show requirements alongside the input as the user types — never only after submission
- Password reset and password change must enforce the same rules as sign-up
- **Frontend:** real-time strength indicator showing which requirements are met/unmet
- **Backend:** required string, min 8, max 128, regex for complexity. Hash immediately after validation — never log, store, or pass around plaintext.
- **Database:** not applicable — the auth provider stores the hash. Never store plaintext or validate password rules at database level.

### First Name / Last Name

- Required, trimmed, 1-50 characters after trimming
- Allow Unicode letters (accents, non-Latin scripts) — not just ASCII a-z
- Allow internal spaces, hyphens, and apostrophes (Mary Jane, O'Brien, Mary-Jane)
- No numbers, no other special characters
- **Frontend:** trim on blur, show error if empty or too long
- **Backend:** required string, trimmed, min 1, max 50, regex allowing Unicode letters, spaces, hyphens, apostrophes
- **Database:** text, NOT NULL, CHECK that trimmed length is between 1 and 50

### Full Name (derived)

- Auto-generated from first_name + last_name on the backend — never accept full_name directly from the user
- If your product uses a single "display name" field instead of first/last, apply the same rules as first name but with a 100-character limit
- **Frontend:** not a user input (derived)
- **Backend:** construct server-side from validated first_name and last_name
- **Database:** text, NOT NULL, CHECK that trimmed length is between 1 and 100

### Workspace Name

- Required, trimmed, 1-50 characters after trimming
- Allow letters, numbers, spaces, hyphens, ampersands (e.g., "Acme & Co", "My Project 2")
- No pure whitespace
- **Frontend:** trim on blur, show error if empty or too long
- **Backend:** required string, trimmed, min 1, max 50
- **Database:** text, NOT NULL, CHECK that trimmed length is between 1 and 50

### Workspace Slug

- Auto-generated from workspace name on creation (lowercase, replace spaces with hyphens, strip special characters)
- 1-50 characters, lowercase letters, numbers, and hyphens only
- Must not start or end with a hyphen
- Must be unique across all workspaces
- Reserve common slugs that would conflict with routes: admin, api, app, auth, billing, dashboard, settings, etc.
- **Frontend:** show the generated slug as a preview ("your URL will be app.com/acme-co"), allow manual editing
- **Backend:** required string, trimmed, lowercased, min 1, max 50, regex for lowercase alphanumeric with hyphens, checked against reserved slugs list
- **Database:** text, NOT NULL, UNIQUE, CHECK that format matches lowercase alphanumeric with hyphens and length is max 50

### Phone Number (if collected)

- Optional in most SaaS products
- Store in E.164 format (+14155551234) — strip all formatting before storage
- Validate format but do not validate whether the number actually exists (that requires SMS verification)
- **Frontend:** use an international phone input component that handles country codes and formatting
- **Backend:** optional string, regex for E.164 format (+ followed by 1-15 digits, first digit non-zero)
- **Database:** text, nullable, CHECK for E.164 format

### URL Fields (website, avatar_url, links)

- Optional, trimmed
- Must start with https:// — reject http://
- Max 2000 characters
- **Frontend:** trim on blur, show error if doesn't start with https://
- **Backend:** optional string, trimmed, validated as URL format, must start with https://, max 2000 chars
- **Database:** text, nullable, CHECK that length is max 2000. The https check is better handled at the application level since database regex for URLs is fragile.

### Currency / Money Fields

- Store as integer cents (not decimal dollars) — $19.99 is stored as 1999
- Never use float or double precision
- Must be non-negative (or explicitly allow negative for refunds/credits)
- **Frontend:** display formatted with currency symbol and decimal point, but send as integer cents
- **Backend:** required integer, non-negative
- **Database:** integer, NOT NULL, CHECK that value is >= 0 (use numeric if you need sub-cent precision for tax calculations)

### Date / Time Fields

- Store all timestamps in UTC as timestamptz
- Accept input as ISO 8601 strings from the frontend
- Convert to local time for display in the frontend only — never store local times
- For date-only fields (birth date, due date), use the date type, not timestamptz
- **Frontend:** use a date/time picker component, send ISO 8601 string
- **Backend:** validate as ISO 8601 datetime string, coerce to date object
- **Database:** timestamptz for timestamps, date for date-only fields

### Boolean Fields

- Default must be explicit — never rely on "undefined means false"
- Name with is_ or has_ prefix for clarity
- **Frontend:** checkbox or toggle component, always has a value (true/false)
- **Backend:** boolean with explicit default (false), coerce from string "true"/"false" if coming from form data
- **Database:** boolean, NOT NULL, DEFAULT false

### Enum / Status Fields

- Use a defined set of string values, not integer codes
- Values must be human-readable: 'active', 'suspended', 'cancelled' — not 1, 2, 3
- **Frontend:** dropdown or radio buttons populated from the enum values
- **Backend:** validated against the exact set of allowed values — compile-time type safety
- **Database:** text, NOT NULL, CHECK against the allowed values list, or Postgres enum type

## Webhook Payload Validation

External webhooks (Stripe, Resend, etc.) are untrusted input, same as user forms.

- Verify the webhook signature before parsing the payload (prevents spoofed webhooks)
- Parse the payload against a Zod schema matching the expected event shape
- Handle unknown event types gracefully (log and skip, don't crash)
- Validate that referenced resources exist in your database before acting on them (e.g., a Stripe webhook referencing a customer_id that doesn't match any workspace)

## Error Response Format

When backend validation fails, return structured errors that the frontend can map to specific form fields:

- Return HTTP 400 for all validation errors (not 500 — that's for server errors, not bad input)
- Include field-level errors so the frontend can highlight the specific field
- Never expose internal error details (database constraint names, stack traces) to the client

## What to Test

- Every Zod schema rejects invalid input (wrong type, missing required, exceeding length)
- Every Zod schema accepts valid input including edge cases (unicode names, + in email, max length strings)
- Server Actions return structured field-level errors on validation failure
- Whitespace-only strings fail required validation
- File upload type validation catches mismatched MIME types vs magic bytes
- XSS payloads in text fields are rendered as text, not executed
- Mass assignment: extra fields in request body are stripped by Zod parsing

## Edge Cases

- **Whitespace-only strings** — a field containing only spaces should fail "required" validation. Trim all string inputs before validating.
- **Unicode edge cases** — a name field accepting "letters only" must handle Unicode letters (accents, non-Latin scripts), not just ASCII a-z.
- **Extremely long input** — a user pastes 10MB of text into a text field. Frontend should enforce max length, backend should enforce max length, and the database should have a check constraint. All three catch it at different stages.
- **Duplicate submission** — user double-clicks submit. Disable the button on first click. Backend should handle idempotency (don't create two records from the same submission).
- **Stale form data** — user opens a form, walks away for an hour, submits. The data they loaded may be outdated. Use optimistic locking (check `updated_at` hasn't changed) for edit forms.
- **Array/object injection** — API route expects a string, attacker sends an array or object. Zod's `.parse()` catches this automatically — but only if you actually validate the input.
- **Numeric string coercion** — form sends "42" as a string, backend expects a number. Use Zod's `.coerce.number()` for form inputs that come as strings over HTTP.
- **File upload validation** — check file type (MIME type AND extension — MIME can be spoofed), file size, and dimensions (for images) both on the frontend (for UX) and backend (for enforcement). Never trust the client's reported MIME type alone.
- **Locale-dependent validation** — phone numbers, postal codes, currency formats vary by country. Use locale-aware validation libraries when validating international input, not hardcoded regex.

## Implementation Checklist

### Validation Layers
- [ ] Zod schemas defined for every form and API input
- [ ] TypeScript types derived from Zod schemas via `z.infer<>` (no manual type duplication)
- [ ] React Hook Form configured with Zod resolver for all forms
- [ ] Inline error messages displayed per field on the frontend
- [ ] Every server action validates input with Zod before any business logic
- [ ] Every API route validates request body / params with Zod
- [ ] Webhook payloads validated (signature check + Zod schema)
- [ ] URL parameters and query strings validated
- [ ] String inputs trimmed before validation
- [ ] Database check constraints on: non-negative numbers, enum values, required strings, value ranges
- [ ] NOT NULL on all required columns
- [ ] UNIQUE constraints where duplicates would corrupt data
- [ ] Error responses return structured field-level errors (HTTP 400)
- [ ] Submit buttons disabled during submission (prevent double-submit)

### Input Security
- [ ] No use of `dangerouslySetInnerHTML` without DOMPurify sanitization
- [ ] All database queries use parameterized queries (no string concatenation of user input into SQL)
- [ ] SameSite=Lax on session cookies (CSRF protection)
- [ ] CSRF protection on any forms not using Server Actions
- [ ] All Zod schemas strip unknown fields (prevents mass assignment)
- [ ] RLS UPDATE policies restrict to USER columns only (prevents over-posting at DB level)
- [ ] File uploads: filenames generated server-side, file type validated by magic bytes, size limits enforced
- [ ] Request body size limits configured per endpoint
- [ ] Rate limiting on form submission endpoints (not just auth)
- [ ] No user input concatenated into HTML strings server-side (use templating with auto-escaping)

## Common Mistakes

- Validating only on the frontend (bypassed by anyone with DevTools or curl)
- Validating only on the backend (user gets no feedback until after a round trip — bad UX)
- Defining Zod schema and TypeScript type separately (they drift apart, bugs follow)
- Not validating webhook payloads (trusting Stripe's payload structure blindly — what if it's not actually from Stripe?)
- Using `.safeParse()` everywhere and forgetting to check `.success` (validation silently passes with invalid data)
- Not trimming whitespace (spaces-only string passes "required" check)
- Trusting client-reported file MIME types (trivially spoofed)
- Returning raw database errors to the client (leaks schema details, confusing to users)
- No check constraints in the database (application bugs can write invalid data with no safety net)
- Validating only the request body and ignoring URL params / query strings (they're user input too)
- Using `dangerouslySetInnerHTML` without sanitization (XSS)
- Concatenating user input into SQL strings (SQL injection — use parameterized queries)
- Spreading raw request body into database queries without Zod parsing (mass assignment)
- Using client-provided filenames for storage paths (path traversal)
- No rate limiting on non-auth endpoints (abuse, spam, denial-of-service)
