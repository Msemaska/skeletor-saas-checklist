# Component States & Error Pages

---

## What It Is

Every component that displays data has multiple states beyond the "happy path" of showing content. If you only design for the populated state, users see broken layouts, blank screens, and confusing errors. Every data-driven component must handle all of its possible states explicitly.

## The Six States

Every component that fetches or displays data must handle these six states. No exceptions.

```
Component Mounts
  │
  ├── Data loading → Loading State
  │
  ├── Data loaded, has items → Populated State (happy path)
  │
  ├── Data loaded, zero items, first time → First-Time Empty State
  │
  ├── Data loaded, zero items, filtered → Filtered Empty State
  │
  ├── Data loaded, user lacks permission → No Permission State
  │
  └── Data failed to load → Error State
```

### 1. Loading State

The component is fetching data. The user needs visual feedback that something is happening.

- Show a skeleton that matches the shape of the expected content — not a generic spinner. A table shows skeleton rows, a card grid shows skeleton cards, a form shows skeleton fields.
- Skeletons use a subtle pulse animation (built into Shadcn's Skeleton component)
- Never show a blank area while loading — the layout should feel stable with placeholders filling the space the content will occupy
- For server-rendered pages (Server Components), the loading state is handled by Next.js `loading.tsx` — the skeleton appears while the page loads
- For client-side data (TanStack Query), use the `isLoading` / `isPending` state to conditionally render the skeleton

### 2. Populated State (happy path)

Data loaded successfully and there are items to display. This is the normal state — the one you design first.

- No special rules beyond standard UI patterns
- This is the state shown in screenshots, demos, and marketing materials

### 3. First-Time Empty State

Data loaded successfully but there are zero items because the user hasn't created anything yet. This is different from a filtered empty state — the user isn't looking for something that doesn't exist, they haven't started yet.

- Show a clear message explaining what this section is for
- Include a single prominent CTA to create the first item ("Create your first project")
- Include a one-line description below the heading explaining what this feature does. No illustration — keep it minimal, the CTA is the focus.
- The empty state IS the onboarding for this feature — it teaches the user what to do next
- Never show a blank table with headers and no rows — that looks broken, not empty

### 4. Filtered Empty State

Data loaded successfully but the current filters/search returned zero results. The data exists, the user's query just didn't match anything.

- Show "No results match your filters" (or "No results for [search term]")
- Include a "Clear filters" or "Reset search" button that removes all filters and shows all items
- Do not show the first-time empty state CTA ("Create your first project") — the user isn't looking to create, they're looking to find
- If the user searched for something specific, echo their query back: "No results for 'quarterly report'"

### 5. No Permission State

Data exists but the user doesn't have permission to view it. This happens with role-based access — a viewer trying to access admin settings, or a user trying to access a workspace they're not a member of.

- Show a clear message: "You don't have permission to view this" — not a 404 (which implies it doesn't exist) and not a blank screen
- Suggest next steps: "Contact your workspace admin for access" or "Go back to dashboard"
- Never reveal what's behind the permission gate — don't show titles, counts, or previews of content they can't access
- For pages: redirect to dashboard or show an inline permission message depending on context
- For components within a page (e.g., a section that requires admin role): hide the section entirely or show a locked state with an explanation

### 6. Error State

Data failed to load. Something went wrong — network error, server error, timeout.

- Show "Something went wrong" with a retry button — not the raw error
- The error state replaces the content area, not the whole page (the sidebar, header, and navigation stay stable)
- Include a secondary link: "If this keeps happening, contact support"
- Log the error to Sentry (covered in phase-3-core-product/12-error-handling-logging.md)
- For partial errors (some data loaded, one section failed), show the error inline for the failed section only — don't blow up the entire page because one API call failed

## Error Pages

Full-page error states for when the entire page fails or doesn't exist. Architectural decisions (Sentry reporting, maintenance page hosting) are covered in phase-3-core-product/12-error-handling-logging.md. This section covers the UX design.

**404 (Not Found):**
- Custom branded page, not the default framework 404
- Friendly message: "This page doesn't exist" or "We can't find what you're looking for"
- Navigation back to the app (link to dashboard)
- Search bar or common links to help the user find what they were looking for

**500 (Server Error):**
- Custom branded page
- Friendly message: "Something went wrong on our end"
- Retry button (reloads the page)
- Support link

**Maintenance:**
- Clear message: what's happening, estimated return time (if known), link to status page
- Branded to match the marketing site — this is often the first impression during an outage

## Deciding Which State to Show

The decision logic for which state to render:

1. Is data still loading? → **Loading State**
2. Did the fetch fail? → **Error State**
3. Does the user have permission? → **No Permission State** (if not)
4. Is the result set empty because of active filters/search? → **Filtered Empty State**
5. Is the result set empty with no filters? → **First-Time Empty State**
6. Otherwise → **Populated State**

This order matters. Check loading first, then errors, then permissions, then empty states. If you check emptiness before permissions, you might show "Create your first project" to a user who can't create projects.

## Design Consistency

All states across the entire app should feel consistent:

- Every skeleton uses the same animation speed and colour
- Every empty state uses the same layout pattern (icon/illustration, heading, description, CTA)
- Every error state uses the same message format and retry pattern
- Every no-permission state uses the same messaging tone

Define these as reusable patterns (primitives in your component library), not as one-off designs per feature. A `<EmptyState>` component, a `<ErrorState>` component, a `<Skeleton>` variant per content type.

## What to Test

- Loading state shows content-shaped skeleton (not generic spinner)
- First-time empty state shows CTA to create first item
- Filtered empty state shows "clear filters" button (not the first-time CTA)
- No permission state shows access denied message (not 404)
- Error state shows retry button and recovers on successful retry
- Partial error (one section fails) shows inline error without affecting the rest of the page
- State decision order: loading → error → permission → filtered-empty → first-time-empty → populated
- 404 page is custom branded with navigation back to app
- 500 page auto-reports to Sentry

## Edge Cases

- **Loading takes too long** — if data hasn't loaded after 5 seconds, show a secondary message below the skeleton: "This is taking longer than expected..." Don't show this immediately — most loads complete in under 1 second and the message would flash.
- **Data loads empty then populates** — user creates their first item, navigates back to the list. The list should show the new item immediately (via cache revalidation), not briefly flash the empty state.
- **Partial data load** — a page has three data sections. Two load successfully, one fails. Show the two populated sections and an inline error state for the failed section. Don't fail the entire page.
- **Infinite scroll reaches the end** — show "You've reached the end" or simply stop showing the loading indicator. Don't show the empty state — the user isn't looking at an empty list, they've scrolled through all items.
- **User toggles a filter and results go from many to zero** — the transition should feel smooth. Don't show a loading skeleton between populated and filtered-empty if the data is cached — go directly from populated to filtered-empty.
- **Permission changes while user is on the page** — user's role is downgraded while they're viewing admin content. On next data fetch or navigation, the no-permission state should appear. Don't crash.
- **Error state retry succeeds** — user clicks retry, data loads successfully. Replace the error state with the populated state seamlessly. Don't navigate away or reload the page.
- **Mobile skeleton layout** — skeletons must match the mobile layout, not the desktop layout. If tables become cards on mobile, the skeleton should show card-shaped placeholders, not table row placeholders.

## Implementation Checklist

- [ ] Loading skeletons defined for every data-driven component (matching the content shape)
- [ ] Skeletons use consistent animation and colour across the app
- [ ] First-time empty state with CTA for every list/collection component
- [ ] Filtered empty state with "clear filters" for every filterable view
- [ ] No permission state with clear messaging and next steps
- [ ] Error state with retry button for every data-fetching component
- [ ] Partial error handling (one section fails, rest of page stays up)
- [ ] Reusable `<EmptyState>`, `<ErrorState>`, `<Skeleton>` components in the component library
- [ ] 404 custom page with navigation and search
- [ ] 500 custom page with retry and support link, auto-reports to Sentry
- [ ] Maintenance page hosted separately (not served by the app)
- [ ] State decision order implemented: loading → error → permission → filtered-empty → first-time-empty → populated

## Common Mistakes

- Only designing the populated state (every other state is an afterthought or a blank screen)
- Using a generic spinner instead of content-shaped skeletons (feels slower, layout shifts when content loads)
- Showing first-time empty state when filters are active (confusing — "Create your first project" when they have 50 projects but a filter is hiding them)
- Showing 404 for permission issues (the resource exists, the user just can't access it — different situations, different messages)
- Blank table with headers and no rows as the empty state (looks broken, not empty)
- Full-page error when only one section fails (destroys the user's context — they lose the rest of the page)
- Error state without a retry button (user has no way to recover without refreshing the browser)
- Inconsistent empty/error states across features (each developer invents their own — build reusable components)
- Checking empty state before permission state (shows "no items" to a user who shouldn't even see the page)
- Maintenance page served by the app itself (when the app is down, the maintenance page is also down)
