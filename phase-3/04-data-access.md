# Data Access Layer

---

## What It Is

How your frontend reads data and triggers mutations, how business logic is organized, and where the boundaries are between client, server, and database. This is the plumbing that every feature builds on top of.

## Reading Data

### Server Components (default for all reads)

Server Components fetch data on the server before the page renders. The user sees fully loaded content, no loading spinners, no client-side fetch waterfalls.

- All page-level data fetching happens in Server Components
- The Server Component creates a database client in user context (RLS enforced), queries the data, and passes it as props to child components
- No `useEffect` + `fetch` for initial page data — that pattern causes loading flashes and layout shifts
- No API route needed — the Server Component queries the database directly via the service layer

### Client-Side Reads (only when necessary)

Some reads must happen on the client — real-time updates, polling, infinite scroll, data that changes without a page navigation.

- Use TanStack Query for all client-side data fetching — it handles caching, deduplication, background refetching, and stale-while-revalidate
- Client-side reads call Server Actions that use the service layer — this keeps auth validation and business logic on the server, same as mutations
- Never duplicate a read that could be done in a Server Component — if the data is available at page load, pass it as props

### When to use which

| Scenario | Method |
|---|---|
| Page load data | Server Component |
| Data that changes in real-time | Client-side with TanStack Query (or Supabase Realtime subscriptions) |
| Infinite scroll / pagination | Client-side with TanStack Query |
| Data after a user interaction (filter, search) | Client-side with TanStack Query |
| Static reference data (plans, categories) | Server Component, cached |
| Dashboard with auto-refreshing metrics | Client-side with TanStack Query (polling interval) |

## Writing Data (Mutations)

### Server Actions (default for all mutations)

Server Actions are the standard way to handle mutations in Next.js App Router. They're server-side functions that can be called directly from client components.

- Every form submission goes through a Server Action
- Every button that creates, updates, or deletes data triggers a Server Action
- Server Actions run on the server — the client never sees the database query, the service key, or the business logic
- Server Actions have built-in CSRF protection
- Server Actions can revalidate cached data after a mutation (revalidatePath, revalidateTag)

**Every Server Action follows this sequence:**
1. Validate auth (get the user from the session, reject if not authenticated)
2. Validate input (parse with Zod, reject if invalid)
3. Execute business logic (via service function)
4. Revalidate cached data
5. Return result or redirect

Never skip steps 1 and 2. Never reorder them. Auth before input validation, input validation before business logic.

### API Routes (only for specific cases)

API routes are for situations where Server Actions don't work:

- **Webhook endpoints** — Stripe, Resend, and other services POST to a URL. This must be an API route, not a Server Action.
- **External API consumers** — if other services or apps call your API. Server Actions are not callable from outside your app.
- **File uploads** — handling multipart form data or streaming large files may require an API route depending on complexity.
- **Cron job targets** — scheduled tasks that hit a URL need an API route.

API routes follow the same sequence as Server Actions: validate auth → validate input → execute logic → return response. The only difference is the transport (HTTP request/response vs Server Action call/return).

### When to use which

| Scenario | Method |
|---|---|
| Form submission | Server Action |
| Button click mutation (delete, toggle, etc.) | Server Action |
| Webhook handler (Stripe, Resend) | API route |
| External API endpoint | API route |
| File upload | API route |
| Cron job target | API route |
| Everything else | Server Action |

Server Actions are the default. API routes are the exception.

## Service Layer

All database queries live in service functions. Not in Server Components, not in Server Actions, not in API routes. The service layer is the single place where data access happens.

**Why:**
- One place to find every query for a given table
- Reusable across Server Actions, API routes, and cron jobs
- Testable in isolation (mock the database client, test the logic)
- Business logic stays separate from request handling

**Structure:**

Services are organized by domain, one file per domain:

- `services/workspaces.ts` — createWorkspace, getWorkspace, updateWorkspace, deleteWorkspace
- `services/projects.ts` — createProject, getProjectsByWorkspace, updateProject, deleteProject
- `services/billing.ts` — syncSubscription, checkUsage, incrementUsage

**Rules:**
- Every service function receives the database client as a parameter (allows passing user context or system context depending on the caller)
- Every service function handles one operation — don't combine "get workspace and check if user is admin and update the plan" into one function
- Service functions return data or throw errors — they don't return HTTP responses, they don't redirect, they don't call `revalidatePath`
- The caller (Server Action or API route) decides what to do with the result — redirect, revalidate, return a response

**The flow:**

```
Client Component
  │ (calls Server Action)
  ▼
Server Action
  │ 1. Validate auth
  │ 2. Validate input (Zod)
  │ 3. Call service function
  │ 4. Revalidate cache
  │ 5. Return/redirect
  ▼
Service Function
  │ (receives db client)
  │ (executes query)
  │ (applies business logic)
  ▼
Database (RLS enforced)
```

## Error Handling

### In Service Functions
- Throw typed errors for expected failures (not found, already exists, permission denied, limit reached)
- Let unexpected errors (database connection failures, etc.) bubble up naturally
- Never catch and swallow errors silently

### In Server Actions
- Catch service errors and return structured error objects to the client
- For form submissions: return field-level errors that React Hook Form can display inline
- For redirects after success: use `redirect()` after the mutation
- Never return raw database errors or stack traces to the client

### In API Routes
- Return appropriate HTTP status codes: 200 (success), 201 (created), 400 (validation error), 401 (not authenticated), 403 (not authorized), 404 (not found), 500 (server error)
- Return JSON error bodies with a consistent shape: `{ error: "message", fields: { name: "too long" } }` — always include the fields object (empty if no field-level errors) so the client doesn't have to handle two different shapes
- Log unexpected errors server-side with full context (user ID, request body, stack trace)

## Data Revalidation

After a mutation, cached data must be refreshed so the UI reflects the change.

- **revalidatePath** — use when a specific page's data has changed (e.g., after creating a project, revalidate the projects list page)
- **revalidateTag** — use when a specific data fetch is tagged and needs refreshing regardless of which page it's on
- **router.refresh()** — client-side, re-runs all Server Components on the current page. Use sparingly.
- Revalidation happens in the Server Action after the service function succeeds, never before, never in the service function itself

## Optimistic Updates

For actions where the UI should feel instant (toggling a favourite, marking as read, reordering items, liking), update the UI immediately before the server confirms.

- Update the local state or TanStack Query cache immediately with the expected result
- Send the mutation to the server in the background
- If the server confirms success, the optimistic update stays
- If the server returns an error, roll back the UI to the previous state and show an error message
- TanStack Query has built-in support for optimistic updates with automatic rollback
- Only use optimistic updates for actions where the success rate is very high (>99%) and the consequence of a failed rollback is low. Do not use for payments, account deletion, or anything irreversible.

## Loading & Pending States

Every mutation must show a pending state to the user. No silent background operations.

- Use `useFormStatus` to detect when a Server Action form submission is in progress — disable the submit button and show a spinner/loading indicator
- Use `useActionState` for Server Actions that return data or errors — track pending, success, and error states
- For TanStack Query mutations, use the `isPending` state from `useMutation` to show loading indicators
- Never allow double-submission — disable the trigger (button, link) immediately when the action starts
- For destructive actions (delete, cancel), show a confirmation dialog BEFORE the pending state, not during

## Direct Database Calls from the Client

Some database providers offer client-side SDKs that let the browser query the database directly, with RLS enforcing access control.

**The default is: don't do it.** Route through Server Components for reads and Server Actions for writes.

**The one exception:** real-time subscriptions. Listening for live database changes (new messages, status updates, presence) requires a persistent client-side connection. This is the only case where a direct client-to-database connection is acceptable.

Everything else — reads, writes, mutations — goes through the server (Server Components, Server Actions, or API routes) via the service layer.

## Database Functions vs Application Code

Some logic can live in the database (triggers, functions) instead of application code.

**Put in the database:**
- `updated_at` auto-update trigger (must never be missed)
- Audit log triggers (must fire on every change regardless of which code path triggered it)
- Cascade soft deletes (when a workspace is soft-deleted, all its children should be too)
- Usage counter increments (atomic, avoids race conditions)

**Keep in application code:**
- Business rules that involve external services (send email, call Stripe API, queue background job)
- Complex conditional logic that's hard to read and debug in SQL
- Anything that needs to be tested with your application test suite

**The rule:** if it must happen on EVERY write to a table regardless of how the write happens (application, migration script, admin tool), put it in the database. If it only matters in the context of a specific user action, keep it in application code.

## Security Considerations

- Server Actions and API routes validate auth before any business logic — no exceptions
- Owner/creator IDs are always set server-side from the session, never accepted from client requests
- Service functions receive the database client as a parameter — the caller determines user vs system context
- Webhook endpoints validate signatures before processing — excluded from session-based auth
- Never return raw database errors to the client — structured error responses only

## What to Test

- Server Actions validate auth before any business logic
- Server Actions validate input with Zod before any business logic
- Server Actions return structured error objects (never throw to client)
- Service functions throw typed errors for expected failures (NotFound, Validation, etc.)
- Revalidation occurs after successful mutations (stale data doesn't persist)
- Optimistic updates rollback correctly on server error
- API routes return correct HTTP status codes (400, 401, 403, 404, 500)
- Webhook endpoints validate signatures before processing
- TanStack Query cache is invalidated after Server Action mutations

## Edge Cases

- **Server Action called after session expires** — the auth check at the top of the Server Action catches this and returns an error. The client should handle the error by redirecting to sign-in, not showing a generic error message.
- **Concurrent mutations on the same record** — two users edit the same project simultaneously. Use optimistic locking: include `updated_at` in the update condition, reject if it's changed since the read. Surface a "this record was modified by someone else" message.
- **Server Action timeout** — long-running operations (file processing, bulk imports) can exceed the Server Action timeout. Offload to a background job and return immediately with a "processing" status.
- **Revalidation before mutation completes** — if you revalidate too early, the UI might show stale data. Always revalidate AFTER the service function returns successfully.
- **Client calls API route that should be a Server Action** — if a developer creates an API route for a form submission, they lose CSRF protection and have to handle it manually. Default to Server Actions unless one of the exception cases applies.
- **Service function called with wrong client context** — a service function that should run in user context (RLS enforced) is accidentally called with system context, bypassing all access control. Establish the convention that service functions receive the client as a parameter, and the caller is responsible for passing the correct context.
- **TanStack Query cache shows stale data after Server Action mutation** — Server Actions revalidate server-side cache, but TanStack Query has its own client-side cache. After a Server Action mutation, invalidate the relevant TanStack Query keys to ensure the client cache is refreshed too.
- **Webhook processes an event for a deleted workspace** — the webhook handler calls a service function referencing a workspace that was deleted between the event firing and processing. Service functions should handle "not found" gracefully rather than crashing.

## Implementation Checklist

### Architecture
- [ ] Server Components used for all page-level data fetching (no useEffect + fetch for initial data)
- [ ] TanStack Query configured for all client-side data fetching
- [ ] Server Actions used for all form submissions and mutations
- [ ] API routes created only for webhooks, external consumers, file uploads, and cron targets

### Service Layer
- [ ] Service functions organized by domain (one file per domain in services/)
- [ ] Every database query lives in a service function (not in components, actions, or routes)
- [ ] Service functions receive the database client as a parameter
- [ ] Service functions return data or throw errors (no HTTP responses, no redirects)

### Mutation Flow
- [ ] Every Server Action follows the sequence: auth → validate → execute → revalidate → return
- [ ] Every API route follows the same sequence
- [ ] Errors returned as structured objects with field-level detail
- [ ] Revalidation happens after successful mutation (revalidatePath or revalidateTag)
- [ ] TanStack Query cache invalidated after mutations where applicable

### UX
- [ ] Every mutation shows a pending/loading state (no silent background operations)
- [ ] Submit buttons disabled during submission
- [ ] Optimistic updates implemented for high-frequency, low-risk actions (toggles, favourites, reordering)
- [ ] Destructive actions show confirmation dialog before execution
- [ ] Optimistic rollback handled gracefully on server error

### Data Access Rules
- [ ] No direct database writes from the client (all mutations via Server Actions)
- [ ] Direct client reads limited to real-time subscriptions only
- [ ] Database triggers in place for updated_at, audit logs, and cascade operations
- [ ] System context only used in backend code (webhooks, cron jobs, admin operations)

## Common Mistakes

- Using `useEffect` + `fetch` for data that's available at page load (causes loading flash, use Server Components)
- Creating API routes for form submissions instead of Server Actions (loses CSRF protection)
- Putting database queries directly in Server Components instead of the service layer (scattered, untestable, unreusable)
- Forgetting to revalidate after a mutation (UI shows stale data)
- Revalidating before the mutation completes (UI briefly shows stale data then updates)
- Putting business logic in database triggers that depends on external services (email, Stripe — these belong in application code)
- Calling service functions with system context when user context was intended (bypasses RLS)
- Not handling "not found" in service functions (crashes instead of graceful error on deleted/missing records)
- Returning raw database errors from Server Actions (leaks schema details to the client)
- Mixing Server Action revalidation with TanStack Query without invalidating both caches
