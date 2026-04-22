# Overlays & Feedback

---

## What It Is

How the app communicates back to the user — confirmations, warnings, errors, and information that appears over or alongside the main content. Every user action needs feedback. If the user clicks something and nothing visibly happens, they don't know if it worked.

## Feedback Principle

Every user action falls into one of these:

| Action Result | Feedback |
|---|---|
| Success (non-destructive) | Toast notification |
| Success (navigation) | Redirect only — the page change is the feedback |
| Success (destructive) | Toast confirming deletion/removal |
| Failure (validation) | Inline field errors |
| Failure (transient — save failed, API timeout) | Error toast with retry suggestion |
| Failure (persistent — connection lost, service down) | Banner at top of content area |
| In progress | Loading indicator on the trigger element |
| Requires confirmation | Confirmation dialog before proceeding |
| Requires input | Modal or sheet with form |

No action should result in zero feedback. If the user clicks a button, something must visibly change within 100ms — even if it's just a loading spinner appearing on the button.

## Toasts

Short, non-blocking messages that appear temporarily to confirm an action or report an error.

**Positioning:** bottom-right of the viewport. Fixed position, above the content, not inside it.

**Behaviour:**
- Auto-dismiss after 5 seconds for success messages
- Auto-dismiss after 8 seconds for error messages (user needs more time to read)
- Show a close button (X) on every toast so the user can dismiss early
- Toasts stack vertically — newest at the bottom, older toasts push up. Maximum 3 visible at once. If a 4th toast arrives, the oldest is dismissed.
- Toasts do not block interaction with the page — the user can continue working while a toast is visible

**Content:**
- One line for the message — keep it under 10 words. "Project created" not "Your project has been successfully created and is now available."
- Include an action link when relevant: "Project deleted. Undo" — the undo link is available for the duration of the toast
- Use the appropriate tone per type: success (neutral confirmation), error (what went wrong + what to do), warning (what to be aware of)

**Toast types:**
- **Success** — green accent or checkmark icon. "Changes saved", "Member invited", "Project deleted"
- **Error** — red accent or error icon. "Failed to save. Please try again.", "Could not send invitation."
- **Warning** — amber accent or warning icon. "You're approaching your usage limit."
- **Info** — neutral accent or info icon. "Your export is being prepared. We'll notify you when it's ready."

**When NOT to use toasts:**
- Validation errors — these go inline next to the field, not in a toast
- Persistent state information — toasts disappear. If the user needs to see the message longer than 8 seconds, use a banner or inline message.
- Critical errors that block the user — these need an error state or dialog, not a dismissible toast

## Modals (Dialogs)

Overlay windows that require the user's attention before they can continue.

**When to use a modal:**
- Quick creation forms (1-3 fields — name, description, etc.)
- Confirmation before destructive actions (delete, cancel, remove)
- Displaying detail that doesn't warrant a full page navigation (preview, quick view)
- Actions that must complete before the user continues (payment confirmation, terms acceptance)

**When NOT to use a modal:**
- Complex forms with many fields — use a full page instead
- Content the user needs to reference while working on the main page — use a sheet/drawer instead
- Displaying lists or tables — these need space, not a cramped overlay

**Sizing:**
- Small (400px width) — confirmations, simple prompts
- Medium (560px width) — quick creation forms, detail views
- Large (720px width) — complex previews, multi-field forms that don't warrant a full page
- Never full-screen modals on desktop — if the content needs full screen, it should be a page

**Behaviour:**
- Background is dimmed with a semi-transparent overlay — clicking the overlay closes the modal (except for confirmation dialogs where accidental dismissal would be bad)
- Pressing Escape closes the modal
- Focus is trapped inside the modal — tabbing cycles through modal elements, not the background page
- Scroll is locked on the background page while the modal is open
- The modal is centered vertically and horizontally in the viewport
- On mobile, modals take full width with padding and sit at the bottom of the screen (bottom sheet pattern)

**Opening and closing:**
- Modals open with a subtle scale-up animation (150ms)
- Modals close on: overlay click, Escape key, cancel button, or successful form submission
- After a form submission inside a modal, close the modal and show a success toast on the main page
- Never nest modals — if an action inside a modal needs confirmation, replace the modal content rather than opening a second modal on top

## Sheets (Drawers)

Side panels that slide in from the right edge of the screen. Used for detail views and secondary forms that the user needs to reference alongside the main content.

**When to use a sheet:**
- Detail view of a list item (click a row, sheet opens with full details on the right)
- Secondary forms that relate to the main page context (editing properties of a selected item)
- Filters panel on mobile (slide-in filter options — filtering logic is covered in phase-3-core-product/08-search.md, the sheet is just the container)

**Sizing:**
- Width: 480px on desktop (approximately 1/3 of the screen)
- On mobile: full width (bottom-to-top sheet or full-screen slide-in)

**Behaviour:**
- Slides in from the right with a 200ms animation
- Background is dimmed — clicking the overlay closes the sheet
- Pressing Escape closes the sheet
- The main page content is still partially visible — this is the key difference from a modal (the user maintains context)
- The sheet scrolls independently of the main page
- Never nest sheets — one sheet at a time

## Confirmation Dialogs

A specific type of modal for confirming destructive or irreversible actions. Design decisions are covered in phase-5-ui-ux/04-data-input-display.md under "Confirmation Dialogs." Key rules repeated here for completeness:

- Entity name or count in the confirmation message
- Destructive button label matches the action ("Delete", "Remove", not "Confirm")
- High-impact actions require typing the entity name to confirm
- Clicking the overlay does NOT close a confirmation dialog — only the explicit Cancel or action button does. Accidental dismissal of a confirmation is dangerous.

## Banners

Persistent messages that sit at the top of the page or section. Unlike toasts, they don't auto-dismiss.

**When to use a banner:**
- Ongoing state that the user needs to be aware of: "Your trial expires in 3 days", "Email not verified", "Usage at 80%"
- System-wide announcements: "Scheduled maintenance on Saturday"
- Persistent errors that affect the whole page: "Unable to connect to the server"

**Behaviour:**
- Positioned at the top of the content area (below the header bar, above the page content)
- Dismissible with an X button — but reappears on next page load if the condition is still true (e.g., trial still expiring)
- Permanently dismissible for announcements (user clicks X, the announcement doesn't come back — store dismissal in localStorage or database)
- One banner visible at a time — if multiple conditions are active, show the highest priority one

**Priority order (highest first):**
1. Errors (connection failed, account suspended)
2. Warnings (trial expiring, usage limit approaching)
3. Info (announcements, new features)

## Inline Feedback

Feedback that appears within the page content, not as an overlay.

**Loading indicators:**
- Buttons: show a spinner inside the button and disable it during submission
- Page sections: show skeleton placeholders (covered in phase-5-ui-ux/03-component-states.md)
- Navigation: highlight the nav item immediately on click (covered in phase-5-ui-ux/02-app-layout-navigation.md)

**Inline success:**
- Auto-save confirmation: subtle "Saved" text or checkmark next to the field, fades after 2 seconds
- Status changes: the status badge updates immediately (optimistic update)

**Inline errors:**
- Form field errors: below the field, red text (covered in phase-3-core-product/02-validation.md)
- Section-level errors: inline error state with retry button (covered in phase-5-ui-ux/03-component-states.md)

## Z-Index Layering Order

All overlay elements follow a strict stacking order (lowest to highest):

1. **Page content** — base layer
2. **Sticky header bar** — above page content
3. **Banners** — above header bar
4. **Sheets** — above page content, beside sidebar
5. **Modal overlay + modal** — above everything except toasts
6. **Confirmation dialog** — same layer as modals (replaces modal content, never stacks)
7. **Toasts** — highest layer, always visible above all overlays

Define these as design tokens so every component uses the same z-index values. Never use arbitrary z-index numbers inline.

## Feedback Timing

How quickly feedback appears matters as much as what it shows:

| Timing | Perception | What to Show |
|---|---|---|
| 0-100ms | Instant | Update the UI immediately (optimistic update, button state change) |
| 100-300ms | Fast | Show loading indicator on the trigger element |
| 300ms-1s | Noticeable | Show loading indicator, user waits but it's acceptable |
| 1-5s | Slow | Show skeleton or progress indicator, keep the UI interactive |
| 5s+ | Too long | Show "This is taking longer than expected" message |

The goal: every action gets visible feedback within 100ms. Even if the actual operation takes 3 seconds, the button shows a spinner immediately.

## What to Test

- Toast appears within 100ms of action completion
- Toast auto-dismisses after 5 seconds (success) or 8 seconds (error)
- Maximum 3 toasts visible at once (oldest dismissed on overflow)
- Modal closes on overlay click, Escape key, and cancel button
- Confirmation dialog does NOT close on overlay click
- Focus is trapped inside open modal (tab cycles within modal only)
- Sheet slides in from right and background is dimmed
- Banner reappears on page load if condition is still true (after dismissal)
- Toasts render above all other overlays (z-index order correct)
- Rapid clicks are debounced (no duplicate toasts or submissions)

## Edge Cases

- **Toast appears behind a modal** — toasts should always render above modals in the z-index stack. If a form inside a modal succeeds, the modal closes and the toast appears on the main page.
- **Multiple rapid actions** — user clicks save 5 times quickly. Each click should be debounced or the button disabled after first click. Don't show 5 success toasts — show one.
- **Toast with undo after page navigation** — user deletes an item, sees "Deleted. Undo" toast, then navigates to another page. The undo action should still work if the toast is still visible. If the toast has dismissed, the undo opportunity is gone.
- **Modal open when browser back button is pressed** — closing the modal should be a history entry. Pressing back should close the modal, not navigate to the previous page. If the modal is closed, then back navigates normally.
- **Sheet open on mobile when device rotates** — the sheet should adapt to the new viewport size. If it was full-width in portrait, it might become a side panel in landscape.
- **Banner dismissed but condition returns** — user dismisses "trial expiring" banner, extends their trial, trial starts expiring again. The banner should reappear because the condition is new.
- **Error toast while user is typing** — an auto-save error toast appears while the user is typing in a different field. The toast should not steal focus or interrupt their typing. Toasts are non-blocking.
- **Confirmation dialog for bulk action on 1 item** — if the user selects 1 item and clicks bulk delete, the confirmation should say "Delete 1 item?" not "Delete 1 items?" Handle singular/plural.

## Implementation Checklist

### Toasts
- [ ] Toast component positioned bottom-right, fixed
- [ ] Auto-dismiss: 5 seconds for success, 8 seconds for error
- [ ] Close button on every toast
- [ ] Maximum 3 visible, oldest dismissed on overflow
- [ ] Four types styled: success, error, warning, info
- [ ] Undo action link available where applicable
- [ ] Toasts render above all other overlays (highest z-index)

### Modals
- [ ] Three sizes defined: small (400px), medium (560px), large (720px)
- [ ] Background dimmed with clickable overlay to close
- [ ] Escape key closes modal
- [ ] Focus trapped inside modal
- [ ] Scroll locked on background
- [ ] Bottom sheet pattern on mobile
- [ ] No nested modals — replace content instead
- [ ] Close modal + show toast on successful form submission

### Sheets
- [ ] Slide-in from right, 480px width on desktop
- [ ] Full width on mobile
- [ ] Background dimmed, overlay click closes
- [ ] Scrolls independently of main page
- [ ] One sheet at a time

### Banners
- [ ] Positioned at top of content area
- [ ] Dismissible with X button
- [ ] Reappears on next load if condition still true (except permanently dismissed announcements)
- [ ] Priority ordering: error > warning > info
- [ ] One banner visible at a time

### Feedback Timing
- [ ] Every action gets visible feedback within 100ms
- [ ] Buttons show spinner and disable during submission
- [ ] "Taking longer than expected" message after 5 seconds
- [ ] Rapid clicks debounced or button disabled after first click

## Common Mistakes

- No feedback after a successful action (user clicks save, nothing happens — did it work?)
- Toasts that block interaction (toasts should be non-blocking — the user should be able to continue working)
- Full-screen modals on desktop (if it needs full screen, it should be a page)
- Nested modals (modal opens a modal — confusing z-index, focus, and escape key behaviour)
- Confirmation dialog closes on overlay click (accidental dismissal of a destructive confirmation)
- Toast messages that are too long ("Your project has been successfully created and saved to the database" — just say "Project created")
- No loading state on buttons (user clicks multiple times because nothing happened, creates duplicate records)
- Banners that can't be dismissed (user sees the same warning on every page load forever)
- Error feedback in a toast when it should be inline (validation errors belong next to the field, not in a floating notification)
- Auto-dismiss on error toasts too fast (3 seconds isn't enough to read "Failed to save: connection timeout. Please check your network and try again")
