# Health Check Endpoint

---

## What It Is

A dedicated API route that reports whether the application is functioning correctly. Used by uptime monitoring (Phase 11 Observability), deployment pipelines, and load balancers to determine if the app is healthy.

## Endpoint

`GET /api/health` — returns a 200 status with a JSON body when the app is healthy, non-200 when it's not.

## What to Check

The health check verifies that the critical dependencies are reachable, not just that the process is running.

**Design decisions:**
- Check database connectivity — execute a lightweight query to verify the database is reachable and responding
- Check auth service connectivity — verify the auth provider is reachable
- Return 200 if all checks pass, 503 (Service Unavailable) if any check fails
- Include which check failed in the response body so monitoring tools can report the specific issue
- The endpoint is public (no auth required) — monitoring services need to call it without credentials
- Response time should be under 500ms — the health check itself should not be a performance bottleneck. If the database query takes more than 500ms, that's a health issue worth flagging.

## What NOT to Check

- Don't check non-critical services (email provider, AI provider, analytics). These can be down without the app being "unhealthy" — features degrade gracefully.
- Don't run expensive queries or computations in the health check. A simple connectivity test is sufficient.
- Don't check third-party APIs that have their own downtime and would cause false alerts.

## Response Format

**Healthy (200):** JSON body with `status` set to "healthy" and a `timestamp` field.

**Unhealthy (503):** JSON body with `status` set to "unhealthy", a `checks` object identifying which check failed, and a `timestamp` field.

Keep the response minimal. The monitoring tool only needs to know: is it up, and if not, what's down.

## Caching

The health check must never be cached. Set `Cache-Control: no-store` on the response. A cached "healthy" response defeats the purpose — monitoring would report the app as healthy when it's actually down.

## What to Test

- Endpoint returns 200 when database is reachable
- Endpoint returns 503 when database is unreachable
- Response includes the correct status and timestamp
- Response is not cached (Cache-Control: no-store header present)
- Response time is under 500ms

## Edge Cases

- **Database is slow but reachable** — the health check query takes 2 seconds instead of 10ms. Set a timeout on the health check query (500ms). If it doesn't respond in time, report unhealthy — a 2-second database response time is effectively down for users.
- **Health check itself crashes** — wrap the entire check in a try/catch. If anything unexpected happens, return 503 with a generic error. Never let the health check endpoint return a 500 with a stack trace.
- **Monitoring tool calls health check too frequently** — the health check runs a database query on every call. At once per minute (standard monitoring interval), this is negligible. At once per second (misconfigured), it adds unnecessary database load. The monitoring interval is configured in the monitoring tool, not in the health check.

## Implementation Checklist

- [ ] `/api/health` route created
- [ ] Database connectivity check included
- [ ] Returns 200 on success, 503 on failure
- [ ] Failed check identified in response body
- [ ] No auth required on the endpoint
- [ ] Cache-Control: no-store header set
- [ ] Query timeout set to 500ms
- [ ] Wrapped in try/catch (never returns 500 with stack trace)
- [ ] Connected to uptime monitoring (Phase 11 Observability)

## Common Mistakes

- Health check only verifies the process is running (returns 200 without checking dependencies — the app is "up" but can't reach the database)
- Checking non-critical services (AI provider is down, health check reports unhealthy, team gets paged for a non-critical issue)
- No timeout on the database query (slow database keeps the health check hanging, monitoring reports timeout instead of unhealthy)
- Cached health check response (monitoring sees a stale "healthy" response while the app is actually down)
- Health check behind auth (monitoring tool can't call it without credentials)
