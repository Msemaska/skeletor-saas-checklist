# API Foundation

---

## What It Is

The public-facing REST API that external developers use to integrate with your product. Covers versioning, authentication, rate limiting, response formatting, and pagination. This is the contract your customers build on — once published, breaking changes require a new version.

## Design Decisions

**Versioning:** Path-based versioning using Next.js App Router folder structure. The version lives in the URL (`/api/v1/workspaces`), not in headers. Create `app/api/v1/` as the base folder. When a breaking change is needed, create `app/api/v2/` and keep v1 running until deprecated. Never version non-breaking changes — add new fields freely, only version when removing or renaming fields.

**Authentication:** API key passed in the `Authorization` header as a Bearer token (`Authorization: Bearer sk_live_abc123`). Use Unkey as the API key management service. Each API key is tied to a single workspace. Keys are created, revoked, and tracked in Unkey. Every API route validates the key by calling Unkey's `verifyKey` endpoint before processing the request. Invalid or revoked keys return 401. Keys have a prefix (`sk_live_` for production, `sk_test_` for test mode) so developers can distinguish them visually.

**Rate Limiting:** Upstash Ratelimit with a sliding window algorithm. Limit: 100 requests per minute per API key. When the limit is exceeded, return HTTP 429 with a `Retry-After` header indicating seconds until the window resets. Include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` headers on every response so developers can track their usage proactively.

**Response Format:** Every response is JSON with a consistent shape. Success responses contain a top-level `data` key holding the resource or array of resources. Error responses contain a top-level `error` key with `code` (a machine-readable string like VALIDATION_ERROR) and `message` (a human-readable description). Validation errors additionally include a `fields` key mapping each invalid field name to its error message. The `fields` key only appears on validation errors. HTTP status codes follow standard conventions: 200 for success, 201 for created, 400 for validation errors, 401 for auth failures, 403 for insufficient permissions, 404 for not found, 429 for rate limited, 500 for server errors.

**Pagination:** Cursor-based pagination using an opaque cursor string. Default page size is 50 items. Maximum page size is 100. Responses include a `data` array of resources and a `pagination` object containing `next_cursor` (the opaque cursor string for the next page) and `has_more` (boolean indicating whether more pages exist). The client passes the cursor as a query parameter to get the next page. No offset-based pagination — it performs poorly on large datasets and produces inconsistent results when data changes between requests.

**Request Validation:** Validate all incoming request bodies and query parameters with Zod schemas. Return 400 with field-level errors when validation fails. Strip unknown fields from request bodies — never pass unvalidated data to the database.

**Middleware Stack:** Create a shared API middleware that runs on every `/api/v1/` route in this order: validate API key, check rate limit, parse and validate request body, execute handler. Use a wrapper function that each route handler calls.

## Security Considerations

- API keys are the only auth mechanism for the public API — no session cookies, no OAuth tokens from end users
- Every API request is authenticated via API key in the Authorization header — reject requests without a valid key immediately
- API keys are tied to a workspace — all data returned through the API is scoped to that workspace's data via RLS
- Rate limiting is per API key, not per IP — prevents a single customer from overwhelming the system
- Rate limit responses include Retry-After header so clients know when to retry
- API keys can be revoked instantly — revocation takes effect on the next request
- API responses never include data from other workspaces — cross-tenant isolation is enforced at the database level
- Request body size limits configured to prevent abuse

## What to Test

- Valid API key returns 200 with correct data shape
- Missing API key returns 401
- Revoked API key returns 401
- Expired API key returns 401
- Rate limit exceeded returns 429 with Retry-After header
- Rate limit headers present on every response
- Invalid request body returns 400 with field-level errors
- Cursor pagination returns correct next page
- Passing no cursor returns the first page
- Passing an invalid cursor returns 400
- Response shape matches the documented format for both success and error cases
- API key can only access resources within its own workspace
- Unknown fields in request body are stripped, not passed through

## Edge Cases

- **API key valid but workspace deleted** — the key validates in Unkey but the workspace no longer exists in your database. Return 403 with a clear message that the workspace has been deactivated.
- **Rate limit race condition** — two requests arrive at the exact same millisecond when only 1 request remains in the window. Upstash handles this atomically via Redis — both requests will not succeed. One gets 429.
- **Cursor points to a deleted record** — the item the cursor references was deleted between requests. The cursor should still work — it represents a position in the sort order, not a specific record. Use the record's created_at timestamp as the cursor value so the query finds the next item after that position regardless of whether the cursor record still exists.
- **Very large response payload** — a workspace with thousands of items. The 50-item default page size prevents this. Never allow unbounded queries.
- **Clock skew on Retry-After** — the client's clock is wrong so the Retry-After value seems incorrect. Use relative seconds (e.g., `Retry-After: 32`) not absolute timestamps.

## Implementation Checklist

- [ ] Unkey account created and configured
- [ ] API key creation endpoint built (workspace settings page)
- [ ] API key revocation endpoint built
- [ ] API key list view in workspace settings showing active keys and last used date
- [ ] Upstash Ratelimit configured with 100 requests/minute sliding window
- [ ] API middleware created: auth validation, rate limiting, request parsing
- [ ] Consistent response format helper functions created (`apiSuccess`, `apiError`)
- [ ] Zod schemas defined for every API endpoint's request and response
- [ ] Cursor-based pagination utility function built
- [ ] Rate limit headers added to all responses
- [ ] 429 response includes Retry-After header
- [ ] API routes scoped under `app/api/v1/`
- [ ] All API routes use the shared middleware stack
- [ ] API key prefixes implemented (`sk_live_`, `sk_test_`)
- [ ] Workspace-scoped data access enforced (key can only access its own workspace)

## Common Mistakes

- Using header-based versioning (developers can't test in a browser, harder to route, easy to forget)
- Offset pagination on large tables (slow, inconsistent when data changes between pages)
- Inconsistent error response shapes across endpoints (client code can't handle errors generically)
- Not including rate limit headers on successful responses (developers can't anticipate limits)
- Returning Retry-After as an absolute timestamp instead of relative seconds
- Letting API keys access resources across workspaces (missing workspace scope check)
- Not validating request bodies (trusting client input, passing raw data to queries)
- Exposing the raw API key in your database (Unkey handles storage — never store the full key yourself)
- No test mode keys (developers test against production data)
- Rate limiting by IP instead of API key (shared IPs hit the limit, dedicated IPs bypass it)
