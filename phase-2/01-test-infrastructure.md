# Test Infrastructure & Strategy

---

## Test Runner

Use Vitest.

## Test Database

Tests must never run against the development or production database.

**Design decisions:**
- Use a dedicated test database instance — a separate Supabase local instance that is only used during test runs
- The test database is created and seeded before the test suite runs and destroyed after
- Each test file gets a clean database state — reset the database between test files (not between individual tests within a file — that's too slow)
- Use transactions for isolation within a test file: start a transaction before each test, rollback after. This is faster than re-seeding between every test.
- The test database uses the same schema, migrations, and RLS policies as production — never skip RLS in tests, that's the whole point of testing it

## Mock Strategy

External services are mocked. Internal services are not.

**What to mock:**
- Payment provider (Stripe) — use Stripe's test mode API keys for integration tests, mock the Stripe SDK for unit tests
- Email provider (Resend) — replaced by Inbucket in test environment (see Email Testing section below). Mock the send function only in unit tests that don't need to verify email content.
- AI provider (Claude API) — mock the API response, verify the correct prompt was sent
- Image generation (Recraft) — mock the API response
- File storage (R2/Supabase Storage) — mock the upload and download functions
- External webhooks — simulate incoming webhook payloads with test data

**What NOT to mock:**
- Your own service functions — test them against the real test database
- RLS policies — test them with actual database queries through different user contexts
- Zod schemas — test with real validation, not mocked results

**Design decisions:**
- Use `vi.mock()` for module-level mocks (replace an entire module with a mock implementation)
- Use `vi.spyOn()` when you want to observe calls while keeping the original implementation
- Mock at the boundary — mock the external SDK/client, not your internal wrapper around it. This ensures your wrapper code is also tested.
- Create mock factories in a `tests/mocks/` directory — reusable mock implementations for Stripe, Resend, etc. Don't re-define mocks in every test file.

## Email Testing

Use Inbucket as a local email capture server. In the test environment, the app sends emails to Inbucket's SMTP server instead of Resend. Inbucket captures every email and makes it queryable via API.

```
Production:  App → Resend API → Gmail/Outlook
Test:        App → Inbucket SMTP (Docker container) → captured, queryable via API
```

**Design decisions:**
- Inbucket runs as a Docker container in the test compose stack alongside the test database
- The app's email service checks the environment — in test, it sends via SMTP to Inbucket instead of calling the Resend API
- Resend is never involved in tests — Inbucket replaces it completely
- React Email templates still render the same way — the only thing that changes is where the rendered email is delivered
- E2E tests query Inbucket's API after triggering an action to verify: email arrived, correct recipient, correct subject, correct body content, correct links
- For auth flows (password reset, email verification): the test queries Inbucket for the email, extracts the verification/reset link from the email body, and follows it — testing the full flow including the actual email content
- For transactional emails (invoices, notifications): the test verifies the email was delivered with the correct template and data
- For marketing emails (newsletters, drip sequences): the test verifies delivery, content, and unsubscribe link functionality
- Inbucket automatically discards captured emails — no cleanup needed between tests

**What Inbucket tests that mocking doesn't:**
- Email template rendering is correct (not just that the send function was called)
- Links in the email body work (verification URLs, reset URLs, CTAs)
- Plain text fallback exists and is readable
- Subject line is correct (not just the parameters passed to the template)
- The full email pipeline works end-to-end (template → render → send → deliver)

## CI Pipeline

Tests run automatically on every pull request and every push to main.

**Design decisions:**
- GitHub Actions runs the test suite
- Tests must pass before a PR can be merged (branch protection rule on main)
- The CI workflow: install dependencies → start test database → run migrations → seed → run tests → report coverage
- Test timeout: 30 seconds per test. Any test that takes longer than 30 seconds is either testing too much or has a performance problem.
- Fail fast: if a test fails, continue running the remaining tests (don't abort) — developers need to see all failures, not just the first one

## Code Coverage

**Design decisions:**
- Use Vitest's built-in coverage with v8 provider
- Coverage threshold: 80% for statements, branches, functions, and lines. This is enforced in CI — the build fails if coverage drops below the threshold.
- Coverage is measured across: service functions, utility functions, Zod schemas, and hooks. Not across components (component coverage is handled by component tests and E2E tests, not line coverage).
- Target 80% coverage. Do not chase 100%.

## Test Data Factories

Reusable helpers that create test data with sensible defaults.

**Design decisions:**
- Create factory functions in `tests/factories/` — one file per entity (createTestUser, createTestWorkspace, createTestProject, etc.)
- Each factory creates a valid entity with default values that can be overridden per test
- Factories use deterministic IDs (hardcoded UUIDs) for commonly referenced entities, random UUIDs for one-off test data
- Factories create the entity in the test database and return the created record — they're not just object builders, they insert real data
- Use factories in every test instead of inline data creation — keeps tests consistent and reduces boilerplate

## File Organization

```
tests/
  ├── factories/           — test data factories (createTestUser, createTestWorkspace, etc.)
  ├── mocks/               — reusable mock implementations (stripe, storage, ai)
  ├── setup.ts             — global test setup (database connection, cleanup)
  ├── unit/                — unit tests (mirrors src/ structure)
  ├── integration/         — integration tests (service functions, API routes, webhooks)
  └── component/           — component tests (React Testing Library)
```

**Design decisions:**
- Test files are colocated in a `tests/` directory, not next to the source files. This keeps the source tree clean and makes it easy to run all tests of a specific type.
- File naming: `{feature}.test.ts` for unit/integration, `{component}.test.tsx` for component tests
- Test descriptions use the pattern: `it('returns null when user is not authenticated')` — describe the expected behavior, not the implementation

## Edge Cases

- **Test database port conflict** — the test database runs on a different port than the dev database (e.g., 5433 for test, 5432 for dev). Both can run simultaneously without conflict.
- **Flaky tests** — a test that passes sometimes and fails sometimes is worse than no test. Investigate and fix immediately — common causes are shared state between tests, timing dependencies, and network calls to unmocked services.
- **Slow test suite** — if the full suite takes more than 5 minutes, parallelize (Vitest runs tests in parallel by default) and check for tests that create excessive test data or don't use transaction rollback.
- **Mock drift** — your mock of Stripe returns a response shape that no longer matches Stripe's actual API. Periodically verify mocks against the real service (run integration tests against Stripe test mode, not just mocked Stripe).
- **Test passes locally, fails in CI** — usually caused by: different Node.js version, different database state, or timezone differences. Pin Node.js version, reset the database in CI, and use UTC everywhere.

## Implementation Checklist

- [ ] Vitest configured with TypeScript and Next.js path aliases
- [ ] Test database container configured (separate from dev database)
- [ ] Database reset between test files (transaction rollback within files)
- [ ] RLS policies active in test database (not bypassed)
- [ ] Mock factories created for: Stripe, AI provider, file storage
- [ ] Mocks stored in tests/mocks/ (reusable, not per-test)
- [ ] Test data factories created for all core entities (tests/factories/)
- [ ] GitHub Actions workflow runs tests on every PR and push to main
- [ ] Branch protection requires passing tests before merge
- [ ] Coverage threshold set to 80% and enforced in CI
- [ ] Test timeout set to 30 seconds per test
- [ ] tests/ directory structure created (factories, mocks, unit, integration, component)
- [ ] Inbucket Docker container in test compose stack
- [ ] App configured to send emails to Inbucket SMTP in test environment
- [ ] E2E tests query Inbucket API to verify email delivery, content, and links
- [ ] Auth flow tests extract verification/reset links from captured emails

## Common Mistakes

- Testing against the dev database (tests corrupt development data, dev changes break tests)
- Mocking your own service functions (you're testing the mock, not the code)
- Skipping RLS in tests (the production security layer is untested)
- No test data factories (every test creates data inline with slightly different shapes — inconsistent and verbose)
- Re-defining mocks in every test file (mock implementations drift apart, maintenance nightmare)
- Chasing 100% coverage (target 80%)
- Tests that depend on execution order (test B passes only if test A runs first — shared state)
- No CI pipeline (tests only run when developers remember to run them locally)
- Mocking at the wrong level (mocking your internal service function instead of the external SDK — internal code is untested)
- Mocking email sends instead of using Inbucket (verifies the send function was called but not that the email content, links, and rendering are correct)
