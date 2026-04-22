# E2E Testing

---

## Tool

Use Playwright.

**AI interface:** Playwright CLI (`playwright`).

## What E2E Tests Cover

E2E tests cover critical user paths only.

**Design decisions:**
- Test against the production build of the app (`next build` + `next start`), not dev mode — dev mode has different behavior (no caching, different error handling, slower)
- Run against a dedicated test environment with its own database, seeded with test data
- Each test is independent — no test depends on another test having run first
- Tests are read as user stories: "user signs up, completes onboarding, creates their first project"

## Auth in E2E Tests

**Design decisions:**
- Create a global setup that signs in once, saves the authenticated session state (cookies) to a file, and reuses it across all tests that need an authenticated user
- Playwright's `storageState` feature handles this — the setup project runs first, saves the state, and other test projects load it
- For tests that need different roles (admin, member, viewer), create separate authenticated states per role in the setup
- For tests that specifically test the sign-in flow itself, don't reuse saved state — test the actual sign-in UI
- Test users are created in the test database seed, with known credentials

## Critical Paths to Test

These are the user workflows that must always work. If any of these break, the product is broken.

**Auth flows:**
- Sign up with email/password → verify email → land on onboarding wizard
- Sign in with email/password → land on dashboard
- Sign in with Google OAuth → land on dashboard (or onboarding if first sign-in)
- Password reset → receive email → click link → set new password → sign in with new password
- Sign out → redirected to sign-in page → cannot access dashboard

**Onboarding:**
- Complete onboarding wizard → land on dashboard with first entity created
- Invited user accepts invitation → shortened wizard → land in the existing workspace

**Core product:**
- Create entity → entity appears in list
- Edit entity → changes persist after page reload
- Delete entity → entity disappears from list, confirmation dialog works

**Billing:**
- Subscribe to a plan → access paid features
- Upgrade plan → new limits take effect
- Cancel subscription → access continues until end of billing period

**Workspace:**
- Invite member → member receives invitation → member accepts → member appears in workspace
- Remove member → member loses access

**Account:**
- Delete account → signed out, cannot sign in again

## Test Environment

**Design decisions:**
- The E2E test environment runs the production build locally or on a CI server
- Database is seeded with known test data (deterministic users, workspaces, entities)
- Stripe is in test mode with test API keys — E2E tests that involve billing use Stripe's test card numbers
- Email delivery is mocked (mock the email API — don't actually send emails during E2E tests)
- The test environment starts and stops as part of the CI pipeline — it's not a persistent server

## Test Structure

**Design decisions:**
- Group tests by user workflow, not by feature or page
- Each test file covers one complete workflow (e.g., `auth-signup.spec.ts`, `billing-subscribe.spec.ts`)
- Use page objects or helper functions for common interactions (filling forms, navigating, asserting toasts) — don't repeat the same selectors in every test
- Use accessible selectors: `getByRole`, `getByLabel`, `getByText` — not CSS selectors or test IDs

## Parallel Execution

Playwright runs tests in parallel by default. Each test gets its own browser context (isolated cookies, storage, state).

**Design decisions:**
- Run tests in parallel in CI for speed
- Each test creates its own test data (or uses pre-seeded data with unique identifiers) — no shared state between parallel tests
- If two tests create a workspace with the same slug, they'll conflict. Use unique names per test.

## When NOT to Write E2E Tests

- Don't E2E test every form validation rule (covered by component tests)
- Don't E2E test every API error response (covered by integration tests)
- Don't E2E test every component state (covered by component tests)
- Don't E2E test third-party service behavior (Stripe's checkout UI, OAuth provider's consent screen)
- Limit E2E tests to critical paths that span multiple pages and involve real navigation, auth, and data persistence

## Edge Cases

- **Flaky tests** — E2E tests are the most prone to flakiness. Common causes: timing issues (element not yet visible), network latency (API call slower than expected), shared test data (another parallel test modified the data). Use Playwright's auto-waiting, avoid arbitrary timeouts, and isolate test data.
- **Test passes locally, fails in CI** — usually caused by: different screen resolution (Playwright's default viewport differs), faster/slower hardware affecting timing, or missing environment variables. Pin the viewport size and ensure all env vars are set in CI.
- **OAuth testing** — don't test the OAuth provider's consent screen (it's third-party UI you don't control). Instead, test the OAuth callback by directly hitting your callback URL with a mock authorization code, or use the saved auth state approach.
- **Stripe checkout testing** — use Stripe's test mode and test card numbers (4242 4242 4242 4242). Test the redirect flow to Stripe Checkout and the return URL, not Stripe's UI itself.
- **Email link testing** — for flows that require clicking an email link (password reset, email verification), query Inbucket's API to retrieve the captured email, extract the verification/reset link from the email body, and follow it in the test. This tests the full flow including email content and link validity. Inbucket setup is covered in phase-2-testing/01-test-infrastructure.md.

## Implementation Checklist

- [ ] Playwright installed and configured
- [ ] Tests run against production build (next build + next start)
- [ ] Dedicated test database with seed data
- [ ] Auth setup project saves session state for reuse across tests
- [ ] Separate auth states for each role (owner, admin, member, viewer)
- [ ] All critical paths covered (auth, onboarding, CRUD, billing, workspace, account)
- [ ] Page objects or helpers for common interactions
- [ ] Accessible selectors used (getByRole, getByLabel, getByText)
- [ ] Tests run in parallel in CI
- [ ] Each test is independent (no order dependency)
- [ ] Stripe test mode with test card numbers for billing tests
- [ ] Email tokens extracted from database (not from actual emails)
- [ ] Viewport size pinned for consistent rendering

## Common Mistakes

- Testing in dev mode instead of production build (different behavior, false confidence)
- Signing in through the UI for every test (slow — use saved auth state)
- Testing every feature with E2E (too slow, too fragile — use unit/component tests for most things)
- Arbitrary sleep/timeout instead of Playwright's auto-waiting (flaky)
- Shared test data between parallel tests (conflicts, flaky failures)
- Testing third-party UI (OAuth consent screen, Stripe checkout form — you don't control these)
- CSS selectors instead of accessible selectors (tests break on styling changes, miss accessibility issues)
- No test isolation (test A creates data that test B accidentally depends on)
