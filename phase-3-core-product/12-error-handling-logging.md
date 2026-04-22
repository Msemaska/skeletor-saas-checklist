# Error Handling & Logging Strategy

---

## What It Is

How your application handles things going wrong and how it records what happened. This defines the unified approach to errors and logging across every layer — service functions, Server Actions, API routes, client components, and background jobs. Without a consistent strategy, every developer (and every AI) invents their own error handling, and the result is chaos.

## Error Categories

Every error falls into one of three categories. The category determines what the user sees, what gets logged, and whether the system retries.

**User errors** — the user did something wrong or hit a known limit:
- Validation failure (invalid input)
- Authorization failure (not allowed to do this)
- Limit reached (usage quota, seat limit)
- Resource not found (deleted, doesn't exist)
- Conflict (duplicate slug, already exists)

The user sees: a specific, helpful message explaining what happened and what to do. These are not logged as errors — they're normal application flow. Log at `info` level if at all.

**System errors** — something broke inside your application:
- Unhandled exception
- Database connection failure
- Failed migration or data corruption
- Out of memory, timeout

The user sees: "Something went wrong. Please try again." — never the raw error. These are logged at `error` level with full context and sent to Sentry.

**Third-party errors** — an external service failed:
- Stripe API down
- Resend email delivery failure
- AI provider rate limited or unavailable
- Storage provider timeout

The user sees: a message specific to what failed — "Payment processing is temporarily unavailable" not "Something went wrong." These are logged at `warn` level (expected to happen occasionally) and retried if the operation supports it.

## Custom Error Types

Define typed errors so your application can handle each category consistently.

**Design decisions:**
- Create a base application error with: message (user-facing), code (machine-readable), status (HTTP status code), and an optional fields object for field-level validation errors
- Extend for specific cases: NotFoundError (404), ValidationError (400), UnauthorizedError (401), ForbiddenError (403), ConflictError (409), LimitReachedError (429)
- Service functions throw these typed errors
- Server Actions and API routes catch them and translate to the appropriate response format
- Untyped errors (anything not extending the base application error) are treated as system errors — logged, sent to Sentry, and surfaced as generic "Something went wrong"

## Error Flow

```
Service Function
  │ throws typed error (NotFound, Validation, etc.)
  │ or throws unhandled error (system error)
  ▼
Server Action / API Route
  │ catches typed errors → returns structured error to client
  │ catches unhandled errors → logs to Sentry, returns generic error
  ▼
Client
  │ typed error → show specific message (inline for forms, toast for actions)
  │ generic error → show "Something went wrong" with retry option
```

**Design decisions:**
- Service functions never catch their own errors (unless retrying) — they throw and let the caller decide
- Server Actions return structured error objects, never throw to the client — the client checks the return value for success or error
- API routes return HTTP status codes with JSON error body
- The client never receives stack traces, database error messages, or internal details

## Error Responses

**From Server Actions:**
- Return an object with a consistent shape: either `{ success: true, data: ... }` or `{ success: false, error: "message", fields: { ... } }`
- For form submissions, the fields object contains per-field errors that React Hook Form can display inline
- The client checks `result.success` and handles accordingly — no try/catch on the client for Server Action calls

**From API Routes:**
- HTTP status code conveys the error category (400, 401, 403, 404, 409, 429, 500)
- JSON body: `{ error: "message", fields: { ... } }` — same shape as Server Action errors
- 500 errors never include internal details — just "Internal server error"

## Client-Side Error Handling

**Global error boundary:**
- A root error boundary catches any unhandled error in the React tree
- Shows a full-page "Something went wrong" with a retry button and a link to go home
- Logs the error to Sentry via the client-side Sentry SDK
- This is the last resort — most errors should be caught and handled before reaching the boundary

**Component-level error handling:**
- Forms: show field-level errors inline from the Server Action response
- Actions (buttons that trigger mutations): show a toast on error with the specific message
- Data fetching: show an inline error state in the component (not a full-page error) with a retry button

**Design decisions:**
- Never show a blank screen on error — always show the error boundary or an inline error state
- Toasts are for transient operation errors ("Failed to save") — not for persistent state errors
- Persistent errors (can't load data, page not found) use inline error states or dedicated error pages
- Every error the user sees should suggest what to do next — retry, go back, contact support

## Error Pages

- **404 and 500** — custom branded pages, not the default Next.js pages. Friendly messaging, navigation back to the app, and support link. 500 auto-reports to Sentry. Design details are covered in Phase 3 (UI/UX).
- **Maintenance** — static page (not served by the app) shown when the app is intentionally down for maintenance. Hosted separately or via the CDN.

## Logging

### Log Levels

| Level | When |
|---|---|
| `debug` | Development only — detailed tracing |
| `info` | Normal operations worth recording |
| `warn` | Something unexpected but handled |
| `error` | Something broke, needs attention |

**Design decisions:**
- Use Pino as the logging library — it's fast, structured (JSON output), and the standard for Node.js
- All logs are structured JSON — never unstructured `console.log` strings. Structured logs are searchable and filterable in log aggregation tools.
- `debug` level is disabled in production — only `info`, `warn`, and `error` reach log aggregation
- Never log sensitive data: passwords, tokens, API keys, full credit card numbers, personal data beyond what's needed for debugging

### What to Log

**Always log:**
- Service function calls that modify data (create, update, delete) — at `info` level with enough context to identify what changed, who changed it, and in which workspace
- Authentication events (sign in, sign out, password change, failed attempt) — at `info` level
- Third-party API calls (Stripe, Resend, AI providers) — at `info` level with: service, endpoint, response status, duration
- Errors — at `error` level with: error type, message, stack trace, request context (user ID, workspace ID, URL, request body minus sensitive fields)
- Background job execution — at `info` level with: job type, status (started, completed, failed), duration

**Never log:**
- Passwords or password hashes
- API keys, tokens, secrets
- Full request/response bodies from AI providers (may contain user data)
- Personal data beyond IDs (don't log user emails in every request — the user ID is enough to look it up)

### Log Context

Every log entry should include enough context to trace it back to a specific request:

- Request ID (generated per request, passed through all layers)
- User ID (from the session)
- Workspace ID (from the request context)
- Timestamp (automatic with Pino)

This allows filtering all logs for a specific user, workspace, or request when debugging an issue.

## Sentry Integration

Sentry captures errors that reach production — unhandled exceptions, unexpected failures, performance issues — across every layer of the stack. It must be wired to the server, client, and edge runtime. API-only Sentry is not sufficient: most user-facing bugs surface in the browser, not the server logs.

**AI interface:** Sentry CLI (`sentry-cli`).

### Setup

Next.js requires four Sentry config files. All four must exist for full coverage:

- `sentry.client.config.ts` — browser SDK init (replays, user context, frontend error capture)
- `sentry.server.config.ts` — Node.js SDK init (Server Actions, API routes, background jobs)
- `sentry.edge.config.ts` — Edge runtime SDK init (middleware)
- `instrumentation.ts` (Next.js) — imports the correct Sentry config based on runtime

Enable source maps in `next.config.ts` via the Sentry webpack plugin. Without source maps, Sentry shows minified stack traces that are undebuggable.

### What Gets Captured Where

| Layer | What Sentry Catches |
|---|---|
| API routes | Unhandled exceptions, system errors |
| Server Actions | Unhandled exceptions thrown past the action boundary |
| Middleware (Edge) | Unhandled exceptions in request middleware |
| React tree (client) | Unhandled exceptions caught by error boundaries |
| Background jobs | Manually captured errors via `Sentry.captureException()` |
| Streaming responses | Manually captured on stream error |

### User & Workspace Context

Call `Sentry.setUser({ id: userId, workspace_id: workspaceId })` immediately after session resolution — in the root server layout for server-side context, and in the auth provider/session hook for client-side context. This links every Sentry event to a specific user and workspace, making issues debuggable without asking the user to reproduce steps.

Clear the user context on sign-out: `Sentry.setUser(null)`.

### Error Boundaries & Frontend Coverage

The root error boundary (`app/error.tsx` and `app/global-error.tsx`) catches unhandled errors anywhere in the React tree and reports them to Sentry automatically via the Sentry error boundary integration. This is mandatory — without it, client-side crashes produce no Sentry events.

Add nested error boundaries around major feature sections (dashboard, editor, settings) so one section failing doesn't crash the entire app. Each nested boundary shows an inline "Something went wrong" state and reports to Sentry with the component context.

### What to Exclude

- Typed application errors (ValidationError, NotFoundError, UnauthorizedError, etc.) — expected application flow, not bugs
- Network errors the user caused (offline, slow connection interrupting a fetch)
- 4xx errors from API routes — these are user/client errors, not system failures

Filter these in Sentry's `beforeSend` callback to avoid noise burying real bugs.

### Sampling & Quotas

- Error events: 100% sample rate — capture every error
- Performance transactions: 10–20% sample rate in production — errors matter more than traces
- Configure Sentry alerts for error spikes — connects to Phase 11 Observability alerting

## Error-to-Notification Bridge

When an error affects a user's pending action (background job failed, export failed, processing failed), the user must be notified through the notification system (07-notification-system.md). The error handling layer logs and reports the error. The notification system tells the user. These are separate concerns:

- The error handler logs the failure, sends to Sentry, and triggers a notification event
- The notification system receives the event and delivers it through the appropriate channel (in-app, email)
- The user sees "Your export failed — please try again" in their notification inbox, not a raw error

This applies to any error that occurs outside the request/response cycle — background jobs, scheduled tasks, webhook processing. If the user triggered the action, they need to know it failed.

## Security Considerations

- No stack traces, database error messages, or internal details reach the client
- Typed application errors (validation, not found) are NOT sent to Sentry — only system errors
- Logs never contain: passwords, tokens, API keys, full credit card numbers, personal data beyond user IDs
- Error messages on auth endpoints don't reveal whether an account exists ("if account exists" language)
- Request context in logs (user ID, workspace ID) enables audit trails without exposing sensitive data

## What to Test

- Typed errors (NotFound, Validation, etc.) return specific user-facing messages
- Untyped/system errors return generic "Something went wrong" (no internal details)
- Server Actions return `{ success: false, error, fields }` shape on failure (never throw)
- API routes return correct HTTP status codes per error type
- Sentry receives system errors but NOT typed application errors
- Global error boundary catches unhandled errors and shows friendly page
- Form errors display inline per field
- Action errors display as toasts
- Logs include request ID, user ID, workspace ID on every entry
- Sensitive data (passwords, tokens, keys) is never logged

## Edge Cases

- **Error during streaming response** — if an error occurs mid-stream (AI generation, large data export), the client has already started receiving data. Send an error event in the stream and let the client display what was received plus the error message. Don't silently stop streaming.
- **Error in error handler** — if Sentry itself fails (network issue, SDK error), the error handling shouldn't crash the app. Wrap Sentry reporting in a try/catch and fall back to console.error. Losing an error report is better than crashing the user's request.
- **Error in background job** — background jobs don't have a user-facing response. Log the error, send to Sentry, and alert via the job monitoring system. The user finds out through a notification ("your export failed") not through an error screen.
- **Rate limited by Sentry** — high-traffic apps can hit Sentry's event limits. Configure sampling (send 100% of errors but only 10-20% of performance transactions) to stay within limits while capturing all actual errors.
- **Client-side error with no network** — the user is offline when an error occurs. The error boundary shows the error page but Sentry can't report it. The error is lost. This is acceptable — offline errors are typically network-related and self-resolving.
- **Same error flooding logs** — a bug causes the same error on every request. Without deduplication, logs fill up and Sentry burns through quota. Sentry deduplicates by fingerprint automatically. For logs, rate-limit identical error messages to prevent log flooding.
- **Sentry wired to API only, not the frontend** — client-side crashes produce no Sentry events without `sentry.client.config.ts` and the root error boundary. Most user-facing bugs will be invisible. All four config files must be present.
- **Third-party error vs system error ambiguity** — Stripe returns a 500. Is that a Stripe issue (third-party) or did you send a bad request (system)? Log as `warn`, include the full Stripe response, and let the retry mechanism handle it. If it persists after retries, escalate to `error`.

## Implementation Checklist

### Error Types
- [ ] Base application error type defined (message, code, status, fields)
- [ ] Typed errors for common cases (NotFound, Validation, Unauthorized, Forbidden, Conflict, LimitReached)
- [ ] Service functions throw typed errors
- [ ] Untyped errors treated as system errors

### Error Flow
- [ ] Server Actions return `{ success, data/error }` — never throw to client
- [ ] API routes return appropriate HTTP status codes with JSON error body
- [ ] No stack traces, database errors, or internal details reach the client
- [ ] Server Action error responses include field-level errors for forms

### Client Handling
- [ ] Global error boundary at root layout
- [ ] Error boundary shows friendly page with retry and home link
- [ ] Form errors displayed inline per field
- [ ] Action errors shown as toasts
- [ ] Data fetching errors show inline error state with retry button
- [ ] 404 and 500 custom error pages

### Logging
- [ ] Pino configured for structured JSON logging
- [ ] Log levels configured (debug off in production)
- [ ] Data-modifying operations logged at info level
- [ ] Authentication events logged at info level
- [ ] Third-party API calls logged with service, status, duration
- [ ] Request ID, user ID, workspace ID included in all log entries
- [ ] No sensitive data in logs (passwords, tokens, keys)

### Sentry
- [ ] `sentry.client.config.ts` created and initialized
- [ ] `sentry.server.config.ts` created and initialized
- [ ] `sentry.edge.config.ts` created and initialized
- [ ] `instrumentation.ts` imports correct Sentry config per runtime
- [ ] Source maps enabled in `next.config.ts` via Sentry webpack plugin
- [ ] `Sentry.setUser()` called after session resolution (server layout + client auth provider)
- [ ] `Sentry.setUser(null)` called on sign-out
- [ ] Root error boundary (`app/error.tsx` + `app/global-error.tsx`) reports to Sentry
- [ ] Nested error boundaries around major feature sections
- [ ] Typed application errors excluded via `beforeSend` filter (only system errors reported)
- [ ] 4xx API errors excluded from Sentry
- [ ] Performance sampling set to 10–20% in production
- [ ] Sentry alerts configured for error spikes

## Common Mistakes

- Returning raw database errors to the client (leaks schema details, confusing to users)
- Using `console.log` instead of a structured logger (unsearchable, no levels, no context)
- Sending expected application errors (validation, not found) to Sentry (creates noise, buries real bugs)
- No error boundary (unhandled error shows a blank white screen)
- Catching errors silently without logging or rethrowing (bugs disappear, data drifts)
- Inconsistent error response shapes (some actions return `{ error }`, others throw, others return `{ message }`)
- Logging sensitive data (passwords, tokens, full request bodies with PII)
- No request context in logs (an error log without user ID or workspace ID is nearly impossible to debug)
- Showing "Something went wrong" for user errors that have a specific cause (validation failure should say what's wrong, not generic error)
- Not handling errors in streaming responses (stream stops, user sees incomplete data with no explanation)
- Missing `sentry.edge.config.ts` (middleware errors are silently swallowed)
- No source maps (Sentry events show minified code, stack traces are useless)
- Not calling `Sentry.setUser()` (errors arrive with no user context, impossible to reproduce or prioritize)
- No `beforeSend` filter (typed application errors flood Sentry, burying real bugs in noise)
