# App Layout & Navigation

---

## What It Is

How your product's interface is structured and how users navigate between features. This is separate from the marketing site — the app could be a web app, desktop app, mobile app, Chrome extension, or any other platform. The principles below apply universally; the implementation details change per platform.

## Core Navigation Elements

Every app, regardless of platform, needs these:

1. **Primary navigation** — persistent access to the main feature areas (5-7 items maximum at the top level — beyond that, the problem is information architecture, not navigation design)
2. **Workspace/account context** — which workspace or account the user is currently operating in (if multi-tenant)
3. **User menu** — profile, settings, sign out
4. **Search / command access** — a way to quickly find things and navigate without using the primary nav
5. **Breadcrumbs or back navigation** — where the user is in the hierarchy and how to go back
6. **Page-level actions** — contextual actions for the current page (create, filter, export)

## Web App Layout

For web-based SaaS products served in the browser.

```
┌──────────┬──────────────────────────────────┐
│          │  Header Bar (breadcrumbs, actions)│
│          ├──────────────────────────────────┤
│ Sidebar  │                                  │
│          │          Page Content             │
│          │                                  │
│          │                                  │
│          │                                  │
└──────────┴──────────────────────────────────┘
```

### Sidebar

- Fixed position, left side, full viewport height
- Width: 256px expanded, 64px collapsed (icons only)
- Collapse toggle button at the bottom of the sidebar
- Remember the user's collapse preference in localStorage
- When collapsed, show tooltips on hover for each icon to reveal the full label

**Sidebar sections (top to bottom):**
1. Workspace switcher (if multi-tenant) — shows current workspace name/logo, dropdown to switch. Searchable if the user belongs to 10+ workspaces.
2. Main navigation — primary feature pages. Maximum 5-7 items.
3. Secondary navigation — less frequent pages (Settings, Help)
4. User menu — avatar, name, sign out. Placed at the very bottom.

**Active state:**
- The nav item matching the current URL path is highlighted
- For nested routes, the parent item stays active
- Match by path prefix, not exact match

**Sidebar behaviour:**
- Sidebar is part of the app layout, rendered once — it does not re-render when navigating between pages
- Scroll position within the sidebar is preserved across navigation
- If the nav list is longer than the viewport, the sidebar scrolls independently — workspace switcher at the top and user menu at the bottom are fixed (always visible)

### Header Bar

- Sits above the page content, inside the content area (not above the sidebar)
- Contains: breadcrumbs on the left, page-level actions on the right
- Breadcrumbs are generated from the URL structure — each segment maps to a readable label. The last segment is the current page (not a link).
- The header bar is part of each page, not the layout — its content changes per page

### Content Area

- Fills the remaining space to the right of the sidebar
- Scrolls independently of the sidebar
- Let the content determine its own max width (forms are narrower, tables are wider)
- Consistent padding: 24px on desktop, 16px on mobile

### Web App Responsive Behaviour

| Breakpoint | Behaviour |
|---|---|
| Desktop (lg+) | Sidebar visible (expanded or collapsed per user preference) |
| Tablet (md) | Sidebar collapsed to icons by default, expandable on hover or click |
| Mobile (< md) | Sidebar hidden. Hamburger button in the header bar opens the sidebar as a drawer overlay. Drawer closes on navigation. |

- Content area takes full width on mobile (no sidebar space reserved)
- Tables switch to card layout on mobile (each row becomes a card)
- Multi-column form layouts stack to single column
- Header bar action buttons: show the primary action as a full button. If there are more than 2 actions, collapse the rest into a "more" dropdown on mobile.
- All styles are mobile-first
- Never hide critical functionality on mobile — rearrange, don't remove
- Test on real devices, not just browser resize

## Desktop App Considerations

If the app is built as a desktop application (Electron, Tauri):

- The sidebar pattern still applies — it's the standard for desktop productivity apps
- The window title bar may integrate with navigation (custom title bar with workspace name, traffic lights / window controls)
- Keyboard shortcuts are more important on desktop — users expect standard shortcuts (Cmd+N for new, Cmd+, for settings, etc.)
- Offline capability may be a factor — navigation should work even when disconnected
- No responsive concerns — the window has a minimum size, not a mobile breakpoint

## Mobile App Considerations

If the app is a native or hybrid mobile app:

- Replace the sidebar with a bottom tab bar for primary navigation (5 items maximum)
- Secondary navigation and settings move to a profile/settings screen accessible from the tab bar
- The workspace switcher moves to the top of the screen or into a sheet/modal
- Breadcrumbs are replaced by a back button and screen title in the top navigation bar
- Gesture-based navigation (swipe back) should be supported
- Tab bar is always visible — it's the persistent navigation element on mobile

## Loading States Between Navigation

When the user navigates between pages, the navigation stays stable. Only the content area shows a loading state.

- Show a skeleton matching the general shape of the page content (content-shaped placeholders, not a generic spinner)
- Navigation items give instant feedback — highlight the clicked item immediately, don't wait for the page to load
- For web apps, use the framework's loading conventions (Next.js `loading.tsx`)

## Command Palette

A standard power-user feature in 2026. Provides keyboard-driven access to navigation and actions.

- Triggered by Cmd+K (Mac) / Ctrl+K (Windows)
- Combines search (find entities) with navigation (go to page) and quick actions (create new, toggle setting)
- Navigation items and actions are hardcoded static entries, search results come from the search infrastructure (covered in phase-3-core-product/08-search.md)
- Group results by type: Pages, Actions, Search Results
- Keyboard navigable (arrow keys to select, Enter to activate, Escape to close)
- Available from any page — it's a global overlay, not page-specific
- On mobile, the command palette can be triggered via a search icon in the header bar instead of a keyboard shortcut

## URL Design (Web Apps)

- URLs are human-readable and bookmarkable
- Entity pages use slugs when available (human-readable, shareable). Fall back to UUID for entities without a name field.
- Settings pages use descriptive paths: `/app/settings/billing`, `/app/settings/members`
- Filter and sort state lives in URL query parameters (shareable, bookmarkable)

## What to Test

- Sidebar active state matches current URL path (prefix matching)
- Sidebar collapse preference persists across page navigations and page reloads
- Collapsed sidebar shows tooltips on hover
- Mobile drawer opens on hamburger click and closes on navigation
- Breadcrumbs generate correctly from URL segments
- Command palette opens on Cmd+K / Ctrl+K
- Command palette closes on Escape and on navigation
- Content area shows loading skeleton during page transitions (sidebar stays stable)
- Deep link while signed out redirects to sign-in, then back to the deep link after auth

## Edge Cases

- **Sidebar longer than viewport** — sidebar scrolls independently with workspace switcher and user menu fixed at top and bottom.
- **User navigates via browser back button** — nav active state must update based on the URL, not just on click events. URL is the source of truth.
- **Workspace switcher with many workspaces** — searchable dropdown, not just a list.
- **Sidebar collapse on page reload** — read localStorage preference before rendering to avoid a flash of the wrong state.
- **Mobile drawer open during navigation** — drawer closes when the user taps a nav item. Don't leave it open.
- **Content area has no data yet** — show the appropriate component state (empty state with CTA). Covered in phase-5-ui-ux/03-component-states.md.
- **User resizes browser window across breakpoints** — sidebar should respond dynamically. If the user shrinks below md, sidebar collapses. If they expand past md, sidebar returns to their preference (not always expanded).
- **Command palette open when navigating** — close the palette on navigation. Don't leave it floating over the new page.

## Implementation Checklist

### Web App Layout
- [ ] Sidebar with workspace switcher, main nav (5-7 items max), secondary nav, user menu
- [ ] Sidebar collapse toggle with preference persisted in localStorage
- [ ] Tooltips on hover when sidebar is collapsed
- [ ] Active nav state derived from current URL path (prefix matching)
- [ ] Header bar with breadcrumbs (from URL) and page-level actions
- [ ] Content area scrolls independently of sidebar
- [ ] Sidebar becomes drawer overlay on mobile
- [ ] Content-shaped loading skeletons during page transitions
- [ ] Nav items highlight immediately on click (before page loads)

### Command Palette
- [ ] Cmd+K / Ctrl+K triggers global palette
- [ ] Combines navigation, actions, and search results
- [ ] Keyboard navigable
- [ ] Closes on navigation and Escape

### Responsive (Web)
- [ ] Mobile-first styles throughout
- [ ] Tables convert to card layout on mobile
- [ ] Multi-column forms stack to single column on mobile
- [ ] Header bar actions collapse to dropdown at 2+ items on mobile
- [ ] No critical functionality hidden on any breakpoint
- [ ] Tested on real mobile devices

## Common Mistakes

- Marking entire layouts or pages as `'use client'` — only the specific component that needs event handlers, hooks, or browser APIs should be a client component. Page content that can be server-rendered should stay a Server Component.
- Using exact URL matching for active nav state (nested routes don't highlight the parent)
- Not persisting sidebar collapse preference (resets on every navigation)
- Sidebar re-renders on page navigation (should be part of the layout, rendered once)
- Mobile drawer stays open after navigation
- More than 7 items in the sidebar top level (information architecture problem — group or remove)
- No command palette (power users expect Cmd+K in 2026)
- No tooltips on collapsed sidebar icons (user sees icons with no context)
- No loading state between page navigations (user clicks nav item, nothing happens for 500ms)
- Hiding features on mobile instead of rearranging them
- Breadcrumbs hardcoded per page instead of generated from URL structure
