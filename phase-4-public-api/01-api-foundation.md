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

**Middleware Stack:** Create a shared API middleware that runs on every `/api/v1/` route in this order: validate API key, check rate limit, check idempotency key, parse and validate request body, execute handler. Use a wrapper function that each route handler calls.

**Idempotency:** Any non-GET request with a side effect that costs money, sends a communication, or creates persistent state must support the `Idempotency-Key` header. The client generates a UUID and passes it with the request. The server stores the response against that key (keyed to the workspace + idempotency key) for 24 hours. If the same key is received again within that window, replay the stored response without re-executing the handler. This covers: credit-consuming operations (AI inference, file processing), resource creation endpoints, payment actions, and notification triggers. Return 422 if an idempotency key is reused with a different request body — that is a client bug. Store idempotency records in Postgres with a TTL cleanup job, not in Redis — the response must survive cache eviction.

## Contract Stability

The API is a contract. Once a field, endpoint, or behaviour is published, callers depend on it. Breaking it silently breaks their integration. The rules below eliminate ambiguity about what is safe to change and what requires a new version.

**Breaking vs. non-breaking changes:**

Non-breaking (safe to ship without a new version):
- Adding a new optional field to a response
- Adding a new optional request parameter
- Adding a new endpoint
- Adding a new value to an enum that the caller is not expected to exhaustively handle
- Expanding a field's accepted values (e.g. increasing a max length)

Breaking (requires a new version):
- Removing or renaming a field from a request or response
- Changing a field's type (string → integer, object → array)
- Making an optional field required
- Removing an endpoint
- Changing the meaning of an existing field
- Changing HTTP status codes for existing conditions
- Changing pagination behaviour or cursor format

The test: if a caller written against the current version would fail, error, or silently misbehave after your change — it is breaking.

**OpenAPI spec as source of truth:** Define the OpenAPI spec first, before writing the route handler. The spec is the contract. The implementation conforms to the spec, not the other way around. Store the spec in `docs/api/openapi.yaml`. When you change a route, update the spec first and treat the diff as the review target.

**Type generation:** Use `openapi-typescript` to generate TypeScript types from `openapi.yaml` as part of the build. These types are used by both the API route handlers (enforcing the response shape server-side) and the SDK (enforcing the request/response shape client-side). A spec drift that would break a caller becomes a TypeScript error at build time before it ships.

**Deprecation policy:** When a breaking change is needed, create a new version (`/api/v2/`) and keep the old version running. Announce the deprecation in the API changelog with a removal date — minimum 90 days notice. Add a `Deprecation` response header to all v1 responses once v2 is live, pointing to the migration guide. Remove v1 only after the deadline has passed and usage in your analytics has dropped to zero.

Never silently remove a version. A caller who misses your announcement and hits a 404 where a 200 used to be has a broken integration. Explicit headers and a changelog give them the best chance to migrate.

## Security Considerations

- API keys are the only auth mechanism for the public API — no session cookies, no OAuth tokens from end users
- Every API request is authenticated via API key in the Authorization header — reject requests without a valid key immediately
- API keys are tied to a workspace — all data returned through the API is scoped to that workspace's data via RLS
- Rate limiting is per API key, not per IP — prevents a single customer from overwhelming the system
- Rate limit responses include Retry-After header so clients know when to retry
- API keys can be revoked instantly — revocation takes effect on the next request
- API responses never include data from other workspaces — cross-tenant isolation is enforced at the database level
- Request body size limits configured to prevent abuse
- Idempotency keys are scoped to the workspace — a key from one workspace cannot replay a response from another

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
- Generated TypeScript types match the OpenAPI spec (build fails on drift)
- Adding a new optional response field does not break existing tests
- A renamed field in the spec causes a TypeScript error in the route handler
- Deprecated version returns `Deprecation` header pointing to migration guide
- Unknown fields in request body are stripped, not passed through
- Idempotent request returns identical response to the original
- Reusing an idempotency key with a different request body returns 422
- Idempotency key scoped to workspace — cannot replay across workspaces
- Expired idempotency key (>24h) is treated as a new request

## Edge Cases

- **API key valid but workspace deleted** — the key validates in Unkey but the workspace no longer exists in your database. Return 403 with a clear message that the workspace has been deactivated.
- **Rate limit race condition** — two requests arrive at the exact same millisecond when only 1 request remains in the window. Upstash handles this atomically via Redis — both requests will not succeed. One gets 429.
- **Cursor points to a deleted record** — the item the cursor references was deleted between requests. The cursor should still work — it represents a position in the sort order, not a specific record. Use the record's created_at timestamp as the cursor value so the query finds the next item after that position regardless of whether the cursor record still exists.
- **Very large response payload** — a workspace with thousands of items. The 50-item default page size prevents this. Never allow unbounded queries.
- **Concurrent duplicate requests with same idempotency key** — two requests arrive simultaneously before either has stored a response. Use a Postgres unique constraint on `(workspace_id, idempotency_key)` with `INSERT ... ON CONFLICT` to guarantee only one executes. The second gets the stored response once the first completes.
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
- [ ] Idempotency key support added to all non-GET endpoints with side effects
- [ ] Idempotency records stored in Postgres with `(workspace_id, idempotency_key)` unique constraint
- [ ] 24-hour TTL cleanup job configured for expired idempotency records
- [ ] 422 returned when idempotency key reused with different request body
- [ ] OpenAPI spec created at `docs/api/openapi.yaml` before any route is implemented
- [ ] `openapi-typescript` wired into build to generate types from spec
- [ ] Generated types used in route handlers and SDK
- [ ] `Deprecation` response header added to all routes when a new version supersedes them

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
- Storing idempotency records in Redis (eviction loses the record, causing a duplicate execution)
- Not scoping idempotency keys to the workspace (keys from different workspaces can collide)
- Applying idempotency to GET requests (unnecessary — they have no side effects)
- Not returning 422 on body mismatch (silently replaying wrong response confuses clients)
- Writing the implementation before the OpenAPI spec (the spec becomes documentation after the fact rather than a contract)
- Treating a field rename as non-breaking (callers reading the old name get undefined)
- Removing a deprecated version before usage has dropped to zero
