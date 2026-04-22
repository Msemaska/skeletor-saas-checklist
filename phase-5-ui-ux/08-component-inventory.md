# Component Inventory

---

## What It Is

A complete catalogue of every UI component a SaaS product needs, split into two levels: primitives (atomic building blocks from Shadcn — never build these yourself) and composites (assembled from primitives, built once per project). Reading this file tells you exactly what to install on day one and what to build during Phase 5. No guessing, no discovering components are missing mid-feature.

---

## Level 1: Always-Install Primitives

These Shadcn components are installed at project setup regardless of what the product does. If a component is not in this list, install it when you first need it rather than pre-emptively.

### Form & Input
| Component | Usage |
|---|---|
| Button | Every interactive action. Variants: default, destructive, outline, ghost, link. Sizes: sm, default, lg, icon. |
| Input | Single-line text entry |
| Textarea | Multi-line text entry |
| Label | Field labels — always paired with an input, never standalone |
| Select | Dropdown selection from a fixed list |
| Checkbox | Binary yes/no, or multi-select in a list |
| Radio Group | Single selection from a small visible set (≤6 options) |
| Switch | Toggle for settings that take immediate effect (no save button needed) |
| Slider | Numeric range input |
| Calendar | Date picker — use inside a Popover |
| Form | React Hook Form wrapper — use for all form construction |

### Layout & Structure
| Component | Usage |
|---|---|
| Card | Content container with optional header, content, footer sections |
| Separator | Visual divider between sections |
| Scroll Area | Constrained-height scrollable region |
| Tabs | Switch between views within the same page context |
| Accordion | Collapsible content sections (FAQ, settings groups) |
| Collapsible | Single collapsible section (simpler than accordion) |
| Resizable | Split pane layouts |

### Overlay & Feedback
| Component | Usage |
|---|---|
| Dialog | Modal for focused tasks, confirmations, and short forms |
| Alert Dialog | Destructive action confirmation only — not for general modals |
| Sheet | Slide-in panel from the right (detail views, filters, mobile nav) |
| Popover | Anchored floating panel (date pickers, rich tooltips, small forms) |
| Tooltip | Single-line hint on hover — for icon buttons and truncated text only |
| Dropdown Menu | Context menu anchored to a trigger (row actions, user menu) |
| Context Menu | Right-click menu |
| Toast (Sonner) | Transient feedback notifications |
| Alert | Inline status message (info, warning, error, success) |
| Progress | Linear progress bar for determinate operations |
| Skeleton | Loading placeholder — mirrors the shape of the content it replaces |

### Data & Display
| Component | Usage |
|---|---|
| Table | Structured tabular data |
| Badge | Status labels, tags, counts |
| Avatar | User profile image with fallback initials |
| Command | Command palette primitive (Cmd+K) |

### Navigation
| Component | Usage |
|---|---|
| Breadcrumb | Hierarchical location indicator |
| Pagination | Page navigation controls |
| Navigation Menu | Top-level marketing site navigation |

---

## Level 2: Composite Components

Built once per project from primitives. Composites are the domain-aware building blocks. Each entry states what it contains, what variants it has, and which component states it must handle.

### App Shell

**Sidebar**
The primary navigation for the app. Contains: app logo/wordmark, main nav links with icons, workspace switcher at the top, user menu at the bottom. Two states: expanded (icon + label) and collapsed (icon only). Collapses to a Sheet drawer on mobile. Active route is highlighted. Built on Shadcn's Sidebar primitive.

**TopBar**
Horizontal bar at the top of the main content area. Contains: page title or breadcrumb on the left, primary page action button(s) on the right (e.g. "New Project"), notification bell, and search trigger. Does not contain the user avatar — that lives in the sidebar.

**UserMenu**
Avatar + dropdown anchored to the bottom of the sidebar. Menu items: profile settings, workspace settings, keyboard shortcuts, sign out. Shows user name and email above the menu items. Displays the current plan badge.

**CommandPalette**
Triggered by Cmd+K (Mac) and Ctrl+K (Windows). Built on Shadcn Command. Contains: global search across entities, quick navigation to any page, recent pages. Closes on Escape or clicking outside.

**MobileNav**
Sheet drawer triggered by a hamburger menu in the TopBar on mobile. Contains the same nav links as the Sidebar. The Sidebar is hidden on mobile — MobileNav replaces it entirely.

**PageHeader**
Reusable section at the top of each page. Contains: page title (h1), optional subtitle, optional action buttons (right-aligned). Used on every main content page for consistency.

---

### Auth

**LoginForm**
Email + password fields, submit button, "Forgot password?" link below the password field, OAuth buttons above the form with a divider. Handles: invalid credentials error (form-level banner), loading state on submit, redirect after success.

**SignupForm**
Name, email, password fields. Password shows strength indicator. Submit button. Link to sign in. OAuth buttons above the form. Handles: email already exists error, password too weak error, loading state.

**ForgotPasswordForm**
Email field only. Submit sends a reset email. After submit: replace form with a confirmation message ("Check your email") — do not navigate away. Handles: email not found (show the same confirmation message regardless — never confirm whether an email exists).

**ResetPasswordForm**
New password + confirm password fields. Validates token from URL before rendering — show an error state if token is invalid or expired. Handles: token expired (show link to request a new one), passwords don't match.

**OAuthButtons**
One button per provider (Google, GitHub — whichever are configured). Consistent icon + "Continue with [Provider]" label. Full-width. Stacked vertically if more than one provider.

---

### Data Display

**StatCard**
Single KPI metric. Contains: label, large number, optional trend indicator (up/down arrow + percentage change vs. previous period), optional sparkline. Four per row on desktop, two on tablet, one on mobile. Used at the top of dashboard pages.

**DataTable**
Wraps Shadcn Table with sorting, pagination, column visibility toggle, bulk select checkboxes, and a bulk action bar. Accepts columns config and data as props. Handles all six component states (loading skeleton, empty, error, populated, filtered-empty, no-permission). The most reused composite in the app — build it once with full features, never build a partial version.

**FilterBar**
Lives above the DataTable. Contains: search input on the left, filter dropdowns on the right (status, date range, category — whatever the table needs), "Clear filters" link that appears when any filter is active. Filter state lives in URL query params.

**EmptyState**
Shown when a table or list has no data. Contains: icon, heading, supporting text, optional primary CTA button. Two variants: truly empty (no records exist — CTA to create the first one) and filtered empty (records exist but none match the filters — CTA to clear filters). Never show table headers with zero rows.

**InlineError**
Displayed inside a content area when a data fetch fails. Contains: error icon, short message, retry button. Different from a Toast — this lives in the content area, not floating. Used when the primary content of a page fails to load.

---

### Billing

**PricingCard**
Used on the pricing page and in upgrade flows. Contains: plan name, price (monthly/annual toggle handled at the parent level), feature list with checkmarks, CTA button. Variants: standard and highlighted (recommended plan). The highlighted variant has a border accent and "Most popular" badge.

**PlanBadge**
Small badge showing the user's current plan. Shown in the UserMenu and on the billing settings page. Variants match the plan names (Free, Pro, Team, Enterprise). Colour-coded: free is muted, paid plans use the primary accent.

**UsageBar**
Progress bar showing consumption of a metered resource (API calls, credits, seats). Contains: resource label, current/max values, progress bar. Turns amber at 80% and red at 95%. Includes a tooltip showing the exact numbers on hover. Used on the dashboard and in the billing settings page.

**UpgradePrompt**
Inline banner shown when a user hits a plan limit. Contains: short message explaining the limit, upgrade CTA button. Appears inline in the content area where the limit was hit (e.g., inside a disabled feature section) — not as a modal unless the action they attempted is completely blocked.

**BillingHistoryTable**
Specialised DataTable for invoices. Columns: date, description, amount, status badge, download PDF link. No sorting, no filtering, no bulk actions — invoices are read-only.

---

### Feedback & Overlays

**ConfirmDialog**
Wraps Shadcn Alert Dialog. Always contains: title (what will happen), description (consequences, entity name or count), cancel button, confirm button with a label that matches the action ("Delete", "Remove", "Cancel Subscription"). The confirm button is always destructive-styled for destructive actions. For high-impact actions (deleting a workspace, irreversible data loss), adds a text-confirmation input requiring the user to type the entity name before the confirm button enables.

**LoadingOverlay**
Full-screen or section-scoped overlay with a spinner. Used for operations that block the entire UI (initial page load with auth check, full-page form submission). Not used for individual component loading states — those use Skeleton instead.

**NotificationBell**
Icon button in the TopBar. Shows an unread count badge. Opens a Popover or Sheet with a list of recent notifications. Each notification has an unread indicator, title, description, timestamp, and a link to the relevant resource. "Mark all as read" action at the top of the list.

---

### Marketing Site

**HeroSection**
Top section of the landing page. Contains: headline, subheadline, primary CTA button, secondary CTA link, hero image or product screenshot. No carousel, no video autoplay. One clear primary action.

**FeatureCard**
Icon, heading, and short description. Used in feature grids. Three or four per row on desktop. Icon uses the brand primary colour.

**TestimonialCard**
Quote text, author name, author title, company, optional avatar. Used in social proof sections. Never fabricate testimonials — only add this component when real quotes exist.

**PricingSection**
Wraps PricingCards with a monthly/annual toggle above them. Annual toggle shows the discount percentage. Toggle state is local — no URL param needed.

**FAQAccordion**
Shadcn Accordion with question/answer pairs. Groups questions by category if there are more than 8 questions total.

**CTABanner**
Full-width section with a headline, short supporting text, and one or two buttons. Used at the bottom of marketing pages. Variants: light background and dark/brand-colour background.

**LogoGrid**
Row of company logos for social proof ("Trusted by teams at..."). Greyscale by default, colour on hover. Only add when real logos are approved for use.

---

## Implementation Checklist

### Day One (project setup)
- [ ] All always-install primitives added via `npx shadcn@latest add`
- [ ] Sonner installed and wired into the root layout
- [ ] App shell composites built: Sidebar, TopBar, UserMenu, CommandPalette, MobileNav
- [ ] PageHeader built and used on every app page

### Auth phase
- [ ] LoginForm, SignupForm, ForgotPasswordForm, ResetPasswordForm built
- [ ] OAuthButtons built and reused across all auth forms
- [ ] All auth forms handle loading, error, and success states

### Core app phase
- [ ] DataTable built once with full features (sort, paginate, bulk select, column toggle, all six states)
- [ ] FilterBar built and wired to URL query params
- [ ] EmptyState built with both variants (truly empty, filtered empty)
- [ ] StatCard built and used for all KPI metrics
- [ ] ConfirmDialog built and used for all destructive actions
- [ ] InlineError built and used wherever a fetch can fail

### Billing phase
- [ ] PricingCard built with highlighted variant
- [ ] PlanBadge added to UserMenu
- [ ] UsageBar built with amber/red thresholds
- [ ] UpgradePrompt built and placed at feature limits (not as a modal)
- [ ] BillingHistoryTable built

### Marketing site phase
- [ ] HeroSection, FeatureCard, PricingSection, FAQAccordion, CTABanner built
- [ ] TestimonialCard built only when real quotes exist
- [ ] LogoGrid built only when logo usage is approved

---

## Edge Cases

- **DataTable with server-side data and optimistic updates** — a bulk delete that removes rows optimistically while the request is in-flight can cause the pagination count to desync. Update the total count optimistically alongside the row removal, and revert both if the request fails.
- **UsageBar at exactly 100%** — the bar fills completely but the user has not been blocked yet. Show the red state and the UpgradePrompt immediately at 100%, before the next action fails.
- **ConfirmDialog closed mid-submission** — user triggers a delete, the dialog opens, they click confirm, and then close the dialog before the request completes. The delete request is still in flight. Disable the close button while the request is in progress.
- **CommandPalette opened from within a Dialog** — focus trap in the Dialog conflicts with the CommandPalette. The CommandPalette must render in a portal above the Dialog's z-index and manage its own focus trap independently.
- **EmptyState in a filtered DataTable** — showing "No results" without a "Clear filters" CTA when filters are active makes the table look broken. Always distinguish between truly empty and filtered empty.
- **PlanBadge for a cancelled subscription** — the subscription exists but is in a cancelled/past_due state. Show a "Cancelled" or "Past due" badge variant — not the plan name — so the user knows their access is at risk.
- **OAuthButtons when provider is down** — the OAuth redirect fails silently. Handle the OAuth error callback URL and show a form-level error on the auth page: "Google sign-in is temporarily unavailable. Sign in with email instead."
- **StatCard with no historical data for trend** — the product just launched and there is no previous period to compare against. Hide the trend indicator entirely rather than showing 0% or an error.

---

## Common Mistakes

- Building a one-off table for a specific feature instead of using the shared DataTable (creates inconsistency and duplicated logic)
- Using Dialog for destructive confirmations instead of Alert Dialog (different semantics — Alert Dialog is the correct component and has the right accessibility role)
- Building multiple empty state designs ad-hoc instead of the single EmptyState composite (visual inconsistency across the app)
- Putting the UserMenu in the TopBar instead of the Sidebar (the TopBar gets crowded; the Sidebar footer is the convention)
- Using Switch for settings that require a save button (Switch implies immediate effect — if saving is needed, use a Checkbox instead)
- Using Tooltip on full text labels (Tooltips are for icon-only buttons and truncated text — adding them to readable labels creates noise)
- Showing UpgradePrompt as a blocking modal (interrupts the user's flow; inline prompts at the point of the limit are less disruptive and convert better)
- Installing every Shadcn component at project start (bloats the project; only install what's in the always-install list and add others on demand)
- Building a custom confirmation dialog instead of reusing ConfirmDialog (inconsistent copy, missing entity name, wrong button labels)
- Skipping the skeleton loading state on DataTable (table appearing empty then suddenly populated is jarring — skeleton transitions are smoother)
