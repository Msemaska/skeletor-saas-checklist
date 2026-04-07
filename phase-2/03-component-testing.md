# Component Testing Patterns

---

## Testing Library Approach

Use React Testing Library with Vitest. Test components the way a user interacts with them — by visible text, labels, roles, and actions. Not by CSS classes, component internals, or implementation details.

**Design decisions:**
- Query elements by role, label, or text — never by test ID unless there's no accessible alternative. If you can't find an element by role or label, the component has an accessibility problem.
- Fire events the way a user would: click buttons, type in inputs, submit forms. Don't call component methods directly.
- Assert on what the user sees: visible text, rendered elements, navigation changes. Don't assert on component state or props.
- Mock Next.js navigation hooks (useRouter, usePathname, useSearchParams) — these don't exist outside the browser. Create a shared mock in tests/mocks/.

## What to Test in Components

**For every component that accepts data:**
- Renders correctly with valid data (populated state)
- Renders the loading state (skeleton)
- Renders the empty state (first-time empty, filtered empty)
- Renders the error state (with retry button)
- Renders the no-permission state (if applicable)

**For every form component:**
- Submit button is disabled when form is invalid
- Submit button is disabled during submission (loading state)
- Field-level errors appear below the correct field on invalid input
- Successful submission shows the correct feedback (toast, redirect, or inline confirmation)
- Failed submission shows the error message without losing the user's input

**For every auth-gated component:**
- Renders correctly for each role (owner sees delete button, member doesn't)
- Hidden elements are not in the DOM (not just visually hidden — fully absent)
- This is a UI test only — server-side enforcement is tested in integration tests

**For every interactive component:**
- Click/tap triggers the expected action
- Keyboard interaction works (Enter, Escape, Tab)
- Loading indicators appear during async operations
- Optimistic updates show the expected state immediately

## Mocking in Component Tests

**Mock the Server Actions** — component tests verify the UI, not the business logic. When a form submits, the Server Action should be mocked to return a success or error response. The Server Action's actual logic is tested in integration tests.

**Mock the data fetching** — components that use TanStack Query should have the query pre-populated with test data via QueryClient. Don't let components make real API calls in tests.

**Don't mock Shadcn components** — they're part of your UI. If a Dialog or Dropdown doesn't work in tests, that's a real bug.

## Snapshot Testing

Don't use snapshot testing. Test behavior instead.

## Testing Forms

Forms are the most common component pattern. Test the full interaction flow.

**Test pattern:**
1. Render the form with initial data (or empty for create forms)
2. Fill in fields by their labels
3. Submit the form
4. Assert: the mocked Server Action was called with the correct data
5. Assert: success feedback appears (toast, redirect)
6. For errors: assert field-level error messages appear below the correct fields

**Specific form tests:**
- Required field left empty → shows "required" error
- Field with invalid format (email, URL) → shows format error
- Form submitted with valid data → Server Action called, success feedback shown
- Form submitted, Server Action returns error → error displayed, form data preserved
- Unsaved changes → navigation warning appears

## Testing Tables

**Test pattern:**
1. Render the table with test data via mocked TanStack Query
2. Verify rows render with correct data
3. Test sort: click column header, verify order changes
4. Test pagination: verify "Showing X-Y of Z", click next page
5. Test row action: click three-dot menu, verify actions appear
6. Test bulk select: check items, verify bulk action bar appears with correct count

**Specific table tests:**
- Empty table shows empty state component (not table headers with zero rows)
- Select all selects current page only
- Destructive action shows confirmation dialog
- Mobile viewport renders cards instead of table rows

## Testing Overlays

**Modals:**
- Opens when trigger is clicked
- Closes on overlay click, Escape key, and cancel button
- Focus is trapped inside the modal
- Confirmation dialog does NOT close on overlay click

**Toasts:**
- Appears after action completes
- Contains the correct message
- Dismiss button works

**Banners:**
- Appears when condition is met
- Dismiss button hides it
- Reappears on next render if condition persists

## Edge Cases

- **Component renders differently on mobile** — use a viewport resize utility to test mobile rendering. Verify tables convert to cards, sidebar collapses, etc.
- **Component depends on browser APIs** — localStorage, window.matchMedia, ResizeObserver. Mock these in the test setup file (tests/setup.ts).
- **Async component updates** — use React Testing Library's `waitFor` or `findBy` queries for elements that appear after async operations. Don't use arbitrary `setTimeout` in tests.
- **Component renders inside a provider** — wrap components in necessary providers (QueryClientProvider, ThemeProvider) using a custom render function in tests/setup.ts.
- **Form with auto-save** — test that the debounced save fires after the correct delay. Use `vi.useFakeTimers()` to control time advancement.

## Implementation Checklist

- [ ] React Testing Library configured with Vitest
- [ ] Custom render function wrapping components in necessary providers
- [ ] Next.js navigation hooks mocked (useRouter, usePathname, useSearchParams)
- [ ] Browser APIs mocked in test setup (localStorage, matchMedia, ResizeObserver)
- [ ] All six component states tested for every data-driven component
- [ ] All form components tested (submit, validation errors, success/error feedback)
- [ ] Auth-gated components tested per role (elements present/absent)
- [ ] Interactive components tested for click, keyboard, and loading states
- [ ] Tables tested for sort, pagination, row actions, bulk select, mobile card layout
- [ ] Overlays tested for open/close behavior and focus management
- [ ] No snapshot tests

## Common Mistakes

- Querying by test ID instead of accessible role/label (hides accessibility bugs)
- Asserting on component state or props instead of visible output (tests break on refactors that don't change behavior)
- Mocking Shadcn/Radix components (they're part of the UI — mock them and you're not testing the real component)
- Snapshot testing (captures HTML structure, not behavior — breaks on every visual change)
- Not wrapping components in providers (component crashes in test but works in the app)
- Not testing the error state (only testing the happy path — the error state is untested in production)
- Using `setTimeout` instead of `waitFor` for async assertions (flaky, timing-dependent)
- Testing implementation details (asserting that a specific function was called instead of asserting the visible result)
