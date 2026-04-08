# Data Input & Display Patterns

---

## What It Is

How users enter data (forms) and view collections of data (tables, lists). These are the two most common UI patterns in any SaaS product — almost every feature is either a form or a table. Consistent patterns across the product reduce cognitive load and development time.

## Forms

### Form Layout

**Label position:** above the field, not inline or to the left. Labels above fields work on all screen sizes without responsive adjustments and are faster to scan vertically.

**Field sizing:** fields should be sized to hint at expected input length. A name field should be wider than a zip code field. Don't make every field full-width — it looks lazy and gives no visual cues about expected input.

**Field grouping:** related fields are grouped together with a section heading. A "Profile" form groups name fields together, contact fields together, preferences together. Sections are separated by spacing, not by dividers or borders.

**Field ordering:** most important or most frequently changed fields go first. Required fields before optional fields. Read-only/derived fields at the bottom or in a separate "info" section.

**Form width:** forms should not stretch to full content width. Cap form width at 640px — wider forms create an uncomfortable reading distance between label and field edge. Center the form in the content area or left-align it.

### When to Use Which Form Type

| Scenario | Pattern |
|---|---|
| Creating a new entity (3+ fields) | Full-page form |
| Editing a single field (name, status) | Inline editing (click to edit, Enter to save, Escape to cancel) |
| Editing multiple fields on an entity | Full-page form (navigate to edit page) |
| Quick creation (name only, details later) | Modal with minimal fields |
| Multi-step setup (onboarding, complex config) | Wizard (stepped form with progress indicator) |
| Settings/preferences | Full-page form with sections, auto-save per field |

### Submission Patterns

**Submit button:**
- Placed at the bottom of the form, right-aligned
- Disabled while form is invalid or submission is in progress
- Shows loading indicator during submission (spinner or "Saving..." text)
- For destructive actions (delete, cancel subscription), use a separate destructive-styled button with a confirmation dialog — never in the same visual row as the save button

**Success feedback:**
- For create: redirect to the new entity's page or back to the list with a success toast
- For edit: stay on the same page with a success toast ("Changes saved")
- For settings: auto-save on blur with a subtle "Saved" indicator next to the field — no submit button needed

**Error feedback:**
- Field-level errors displayed inline below each field (covered in phase-3/02-validation.md)
- Form-level errors (server rejection, network failure) displayed as a banner at the top of the form
- Scroll to the first error if errors are off-screen

### Auto-Save vs Manual Save

**Auto-save (save on blur/change):**
- Use for settings and preferences — the user changes a toggle or selects a dropdown, it saves immediately
- Show a subtle confirmation ("Saved" text or checkmark) next to the field that changed
- Debounce text inputs by 1 second before auto-saving (don't save every keystroke)
- If auto-save fails, show an inline error and keep the field in its edited state so the user doesn't lose their change

**Manual save (submit button):**
- Use for entity creation and multi-field editing — the user fills out several fields and submits as a batch
- Use when the user needs to review before committing (e.g., publishing, sending, billing changes)
- The submit button is the explicit "I'm done" signal

### Multi-Step Forms (Wizards)

For complex creation flows that would overwhelm as a single form (onboarding, configuration, checkout).

- Show a progress indicator at the top with step numbers and labels (e.g., "Step 2 of 4: Configure Settings")
- The user can go back to previous steps — don't lock completed steps
- Each step validates independently — don't let the user proceed to step 3 with errors in step 2
- Final step shows a summary/review before submission
- Save progress server-side after each step — if the user leaves and comes back, resume where they left off
- Maximum 5 steps — if you need more, your flow is too complex. Simplify or split into multiple flows.

### Inline Editing

For quick single-field changes without navigating away (renaming a project, changing a status).

- Click the value to enter edit mode — show a text input or selector replacing the static text
- Enter to save, Escape to cancel
- Show a loading indicator briefly while saving
- If save fails, revert to the previous value and show an error toast
- Double-click is not discoverable — use a visible edit icon (pencil) on hover, or make the text itself look clickable (underline or hover highlight)

## Tables

### Table vs Card Layout

| Scenario | Pattern |
|---|---|
| Data has many columns, comparison matters | Table |
| Data has few fields, visual richness matters (images, status badges) | Cards |
| Desktop view of data-heavy content | Table |
| Mobile view of the same content | Cards (tables converted to cards on mobile) |

Default to tables for data-heavy views. Use cards for visual content (media, profiles, portfolios).

### Table Features

**Sorting:**
- Single-column sort only
- Default sort: most recently created first (created_at descending)
- Click column header to sort — toggle between ascending and descending
- Show a sort indicator (arrow) on the active column
- Sort state lives in URL query params (shareable, bookmarkable)

**Pagination:**
- Use offset-based pagination with page numbers, not infinite scroll — pagination gives the user a sense of total size and lets them jump to a specific page
- 20 rows per page
- Show: "Showing 1-20 of 245 results" (use "~12,000 results" when the total is estimated on large tables)
- Page controls: Previous, page numbers (with ellipsis for gaps), Next
- First page load shows page 1 — current page lives in URL query params

**When to use infinite scroll instead:**
- Only for feeds, activity logs, or content streams where the user is browsing, not searching for a specific item
- Not for data tables — users need to know how many items exist and where they are in the list

**Row actions:**
- Each row has a actions dropdown (three-dot menu) on the far right with contextual actions (edit, duplicate, delete, etc.)
- The most common action (usually "open/view") is triggered by clicking anywhere on the row — the row itself is a link
- Destructive actions (delete) in the dropdown use a destructive text colour and trigger a confirmation dialog

**Bulk actions:**
- Checkbox column on the far left for multi-select
- "Select all" checkbox in the header selects all rows on the current page (not all rows in the database)
- When items are selected, a bulk action bar appears above the table: "X selected" with action buttons (delete, export, change status)
- Bulk delete triggers a single confirmation dialog: "Delete X items? This cannot be undone."

**Column visibility:**
- For tables with many columns, provide a column toggle menu (gear icon or "Columns" button) to show/hide columns
- Remember column visibility preferences in localStorage
- Always show: name/title column, primary identifier, actions column. These cannot be hidden.

### Empty and Loading States

Handled by phase-5/03-component-states.md — the table follows the same six-state pattern. A table-specific note: empty tables should never show table headers with zero rows — show the empty state component instead.

### Filtering

Handled by phase-3/08-search.md. The table UI includes filter controls above the table (dropdowns, toggles, search bar), but the filtering logic and architecture are defined in the search doc.

## Shared Patterns

### Confirmation Dialogs

For destructive or irreversible actions (delete, cancel, remove member):

- Modal dialog with a clear description of what will happen
- The entity name or count in the message: "Delete project 'Website Redesign'?" not just "Delete this project?"
- Two buttons: "Cancel" (secondary, left) and "Delete" (destructive, right)
- The destructive button label should match the action: "Delete", "Remove", "Cancel Subscription" — not a generic "Confirm"
- For high-impact actions, require the user to type the entity name to confirm (e.g., deleting a workspace)

### Optimistic Updates

For actions that feel instant (toggling status, favouriting, reordering):

- Update the UI immediately before the server confirms
- If the server rejects, revert the UI and show an error toast
- Covered in detail in phase-3/04-data-access.md

## What to Test

- Form submit button is disabled while invalid and during submission
- Field-level errors display inline below the field
- Scroll to first error when errors are off-screen
- Auto-save triggers on blur for settings (debounced by 1 second for text)
- Inline editing: Enter saves, Escape cancels, failed save reverts value
- Table sort toggles between ascending and descending on column header click
- Table pagination shows correct "Showing X-Y of Z" counts
- Bulk select "select all" selects current page only (not all rows)
- Destructive action confirmation dialog includes entity name
- Unsaved changes warning appears before navigating away from a dirty form

## Edge Cases

- **Form submission with stale data** — user opens an edit form, walks away, comes back and submits. The data may have changed. Use optimistic locking (check updated_at hasn't changed since the form was loaded). If it has, show "This record was updated by someone else. Refresh to see the latest version."
- **Long form with session expiry** — user fills out a long form, their session expires before they submit. The error handling layer (phase-3/12-error-handling-logging.md) catches the 401 and redirects to sign-in. After sign-in, the form data should be preserved if possible (warn before navigating away with unsaved changes).
- **Bulk action on items the user doesn't have permission for** — user selects 10 items, 3 of which they don't have permission to delete. Show a partial success result: "7 items deleted. 3 items could not be deleted (insufficient permissions)."
- **Table with thousands of rows** — pagination handles this, but the total count query can be slow on large tables. Use an estimated count for display ("~12,000 results") and exact count only when the total is small (<1000).
- **Inline edit on a field with validation** — the user edits a name inline and enters an invalid value. Show the validation error inline next to the field, keep the field in edit mode, don't revert. Let them fix it.
- **Sort/filter changes while data is loading** — if the user changes sort or filters while a previous request is in-flight, cancel the in-flight request and fire a new one. Don't show results for the old query arriving after the new query was issued.
- **Table row clicked while a modal is open** — if a row click navigates to a detail page, it should close any open modals first. Don't navigate behind an open modal.
- **Auto-save conflict** — user changes a setting on their laptop and phone simultaneously. The last write wins. Show the updated value on both devices after the next refresh.

## Implementation Checklist

### Forms
- [ ] Labels above fields, form width capped at 640px
- [ ] Fields sized to hint at expected input length
- [ ] Related fields grouped with section headings
- [ ] Submit button disabled during invalid state and submission
- [ ] Loading indicator on submit button during submission
- [ ] Field-level errors inline, form-level errors as banner
- [ ] Scroll to first error if off-screen
- [ ] Success feedback: redirect or toast depending on action type
- [ ] Auto-save for settings (debounced by 1 second for text inputs)
- [ ] Manual save (submit button) for entity creation and editing
- [ ] Inline editing for single-field quick changes (Enter to save, Escape to cancel)
- [ ] Unsaved changes warning before navigating away

### Multi-Step Forms
- [ ] Progress indicator with step labels
- [ ] Back navigation to previous steps
- [ ] Per-step validation (can't proceed with errors)
- [ ] Summary/review step before final submission
- [ ] Progress saved server-side (resume on return)
- [ ] Maximum 5 steps

### Tables
- [ ] Single-column sort with direction toggle and URL state
- [ ] Default sort: created_at descending
- [ ] Pagination with 20 rows per page, page number in URL
- [ ] "Showing X-Y of Z results" display
- [ ] Row click navigates to detail page
- [ ] Actions dropdown (three-dot menu) per row
- [ ] Destructive row actions trigger confirmation dialog
- [ ] Bulk select with checkbox column
- [ ] Select all selects current page only
- [ ] Bulk action bar with count and action buttons
- [ ] Column visibility toggle with preference in localStorage
- [ ] Tables convert to card layout on mobile

### Confirmation Dialogs
- [ ] Entity name or count included in confirmation message
- [ ] Destructive button label matches the action (not generic "Confirm")
- [ ] High-impact actions require typing entity name to confirm

## Common Mistakes

- Full-width forms (uncomfortable reading distance, looks undesigned)
- Submit button always enabled (user submits invalid form, gets errors they could have been prevented from seeing)
- No loading state on submit button (user clicks multiple times, creates duplicate records)
- Auto-save on forms that should be manual-save (user accidentally changes a billing setting by clicking the wrong dropdown)
- Manual save on settings that should auto-save (user changes a preference, forgets to click save, thinks it took effect)
- "Confirm" as the button label on destructive actions (confirm what? "Delete" is unambiguous)
- Select all selects all items in the database (user selects all, clicks delete, wipes 10,000 records when they meant to delete the current page)
- Infinite scroll on data tables (user can't tell how many items exist, can't jump to a specific page, can't share a link to page 5)
- Multi-column sort for MVP (unnecessary complexity — single-column sort covers 99% of use cases)
- Table headers with zero rows visible (looks broken — show the empty state component instead)
- Sort/filter state in component state instead of URL (can't share or bookmark filtered views)
