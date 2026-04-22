# Unit & Integration Testing Patterns

---

## Unit Test Scope

- Utility functions
- Zod schemas
- Price calculations
- Data transformation functions
- Pure business logic that doesn't touch the database

## Integration Test Scope

- Service functions (real database queries, real RLS)
- Server Actions (auth check → validation → service call → revalidation)
- API routes (request parsing → auth → service call → response formatting)
- Webhook handlers (signature verification → event processing → database update)
- RLS policies (verify access control with different user roles)

## Testing Service Functions

Service functions are the core of your business logic. They query the database through the service layer and are tested against the real test database with RLS enforced.

**Design decisions:**
- Test with two database clients: one in user context (RLS enforced) and one in system context (RLS bypassed). This lets you set up test data in system context and verify access in user context.
- For each service function, test: successful operation, expected failures (not found, unauthorized, validation), and RLS enforcement (user in workspace A can't access workspace B's data)
- Create test data using factories at the start of each test, rollback at the end using transactions

## Testing Server Actions

Server Actions are tested as plain async functions — not through the browser. They're imported and called directly in the test.

**Design decisions:**
- Mock the auth session (provide a fake authenticated user) — you're testing the action's logic, not the auth system (auth is tested separately)
- Test the full sequence: auth check rejects unauthenticated calls, validation rejects invalid input, the service function is called with correct parameters, revalidation occurs after success, the correct response shape is returned (`{ success: true/false, data/error }`)
- For each action, test: success path, auth failure, validation failure, service error handling
- Don't mock the service function — let it hit the real test database. The Server Action test IS an integration test.

## Testing API Routes

API routes are tested by constructing a Request object and passing it to the route handler function.

**Design decisions:**
- Construct a real Request with headers, body, and params — don't use a special test client
- Verify: correct HTTP status codes, correct response body shape, correct headers
- For webhook routes: include the correct webhook signature in the test request. Test that invalid signatures are rejected.
- Test with both authenticated and unauthenticated requests

## Testing RLS Policies

RLS is the database-level security layer. Testing it verifies that users can only access data they're authorized to see.

**Design decisions:**
- For each table with RLS, test all four operations (SELECT, INSERT, UPDATE, DELETE) with different user roles
- Create test data in system context (bypass RLS), then query in user context and verify the results are correctly filtered
- Test cross-workspace isolation: user in workspace A queries for data, verify workspace B's data is not returned
- Test column-level UPDATE restrictions: verify that USER columns can be updated but SYSTEM columns cannot
- Test soft delete integration: verify that soft-deleted records (deleted_at IS NOT NULL) are invisible through RLS SELECT

## Testing External Service Integrations

External services (Stripe, Resend, AI providers) are mocked in unit tests and optionally tested against real test environments in integration tests.

**Design decisions:**
- **Stripe:** use Stripe's test mode API keys for integration tests that verify the full billing flow. Use mocked Stripe SDK for unit tests that verify your webhook handler logic. Test all webhook event types you handle.
- **Resend:** mock the send function. Verify the correct template was used, the correct recipient was set, and the correct variables were passed. Don't actually send emails in tests.
- **AI providers:** mock the API response. Verify the correct prompt was constructed, the response was parsed and validated with Zod, and errors are handled gracefully.
- **File storage:** mock upload and download functions. Verify signed URLs are generated correctly, file metadata is saved to the database, and cleanup logic works.

## Test Naming

Every test name describes the expected behavior in plain English.

**Pattern:** `it('returns null when user is not authenticated')`

- Start with the expected outcome: "returns", "throws", "creates", "blocks", "redirects"
- Include the condition: "when user is not authenticated", "when input is invalid", "when workspace is at limit"
- Don't describe implementation: not "calls the service function and checks the result" but "creates a project and returns it"

## What to Test (per test type)

**Unit tests — test in isolation:**
- Every Zod schema: valid input accepted, invalid input rejected with correct errors
- Every utility function: all input variations including edge cases
- Price calculations and proration logic: exact numeric results
- Data transformation functions: correct output shape and values

**Integration tests — test with real dependencies:**
- Every service function: CRUD operations, error cases, RLS enforcement
- Every Server Action: auth → validate → execute → revalidate → respond
- Every API route: status codes, response shapes, auth enforcement
- Every webhook handler: signature validation, event processing, idempotency
- RLS policies: cross-workspace isolation, role-based access, column restrictions, soft delete visibility

## Edge Cases

- **Test data leaks between tests** — use transaction rollback to isolate test data. If a test creates data and doesn't clean up, the next test sees unexpected data and may fail or pass incorrectly.
- **Mock not reset between tests** — if a mock's return value is set in one test and not reset, the next test gets the wrong mock response. Use `vi.restoreAllMocks()` in `afterEach`.
- **Testing the wrong client context** — a test passes because it used system context (bypassing RLS) when it should have used user context. Establish a convention: tests that verify RLS must use user context.
- **Server Action test doesn't verify the response shape** — the action returns `{ success: false, error: "..." }` but the test only checks that no exception was thrown. Always assert the response shape.
- **Webhook test doesn't verify idempotency** — send the same webhook event twice and verify the second one is a no-op (no duplicate data, no duplicate side effects).

## Implementation Checklist

- [ ] Unit tests for every Zod schema (valid + invalid inputs)
- [ ] Unit tests for every utility function
- [ ] Unit tests for price calculation / proration logic
- [ ] Integration tests for every service function (CRUD + errors + RLS)
- [ ] Integration tests for every Server Action (auth → validate → execute → respond)
- [ ] Integration tests for every API route (status codes + response shapes)
- [ ] Integration tests for every webhook handler (signature + processing + idempotency)
- [ ] RLS policy tests (cross-workspace isolation, role restrictions, column restrictions, soft delete)
- [ ] Two database client contexts available in tests (user context + system context)
- [ ] Transaction rollback between tests for isolation
- [ ] Mock reset in afterEach (vi.restoreAllMocks)
- [ ] Test names describe expected behavior in plain English

## Common Mistakes

- Mocking service functions in Server Action tests (you're testing a shell that calls a mock — no value)
- Testing RLS with system context (RLS is bypassed — you're testing nothing)
- No test for the unhappy path (only testing success, never testing what happens on failure)
- Tests that assert implementation details instead of behavior ("expect service.query to have been called with X" instead of "expect the returned data to match Y")
- Webhook tests that don't send the event twice (idempotency is untested)
- Test names that describe what the code does instead of what the outcome is
- No transaction rollback (test data accumulates, tests become order-dependent)
