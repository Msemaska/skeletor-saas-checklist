# Accessibility Audit

---

## What It Is

A structured audit of the application to ensure it meets WCAG 2.1 AA compliance. Covers automated scanning, keyboard navigation testing, screen reader testing, and color contrast verification across all key pages.

## Automated Audit

### Design Decisions

**Design decisions:**
- Use axe DevTools browser extension for automated accessibility scanning
- Run the audit on every key page: landing page, sign-in, sign-up, dashboard, settings, billing, onboarding
- Fix all "critical" and "serious" severity issues before considering the audit complete
- "Moderate" and "minor" issues are logged and fixed in the next development cycle
- Re-run the audit after every major UI change (new page, layout redesign, component overhaul)
- Add @axe-core/react in development mode to catch accessibility violations during development — it logs warnings to the browser console

## Keyboard Navigation

### Design Decisions

**Design decisions:**
- Every interactive element must be reachable via Tab key in a logical order (left-to-right, top-to-bottom)
- Every interactive element must be activatable via Enter or Space
- Dropdown menus and modals must be navigable with arrow keys
- Escape closes any open modal, dropdown, or popover
- Focus trap inside modals — Tab does not move focus outside the modal while it's open
- Focus returns to the trigger element when a modal or dropdown closes
- Visible focus indicator on every focusable element — a custom 2px solid ring in the primary color
- Skip-to-content link as the first focusable element on every page (visually hidden until focused)
- Test keyboard navigation on these flows: sign in, create a new entity, navigate between dashboard sections, open and close settings, complete billing

## Screen Reader Testing

### Design Decisions

**Design decisions:**
- Test with NVDA on Chrome (Windows)
- Test 4 key flows:
  1. Sign in (enter email, enter password, submit, verify landing on dashboard)
  2. Create a new entity (navigate to create form, fill fields, submit, verify confirmation)
  3. Navigate the dashboard (read headings, interact with tables, use sidebar navigation)
  4. Update settings (navigate to settings, change a value, save)
- Every image has alt text. Decorative images use alt="" (empty alt, not missing alt)
- Every form input has an associated visible label
- Every button has a descriptive label (not just an icon — add aria-label for icon-only buttons)
- Dynamic content updates use aria-live regions (toast notifications, form validation errors, loading states)
- Headings follow a logical hierarchy — one h1 per page, h2 for sections, h3 for subsections. No skipped levels.
- ARIA landmarks used: main, nav, banner, contentinfo, complementary for sidebar

## Color Contrast

### Design Decisions

**Design decisions:**
- Every text/background combination meets WCAG AA minimum contrast ratio: 4.5:1 for normal text, 3:1 for large text (18px+ or 14px+ bold)
- Non-text elements (icons, borders, focus indicators) meet 3:1 contrast ratio against their background
- Check contrast for all component states: default, hover, focus, active, disabled
- Disabled elements are exempt from contrast requirements but must still be distinguishable from enabled elements
- Use the axe DevTools contrast checker
- Don't rely on color alone to convey information — always pair color with text, icons, or patterns (e.g., error states use red + an error icon + error text)

## What to Test

- axe DevTools reports 0 critical and 0 serious issues on every key page
- Every interactive element is reachable and activatable via keyboard
- Tab order matches the visual reading order on every page
- Focus is trapped inside open modals and returns to the trigger on close
- Skip-to-content link is present and functional on every page
- NVDA can read and navigate all 4 key flows without getting stuck
- Every image has appropriate alt text (descriptive or empty for decorative)
- Every form input has a label
- Every icon-only button has an aria-label
- Toast notifications and validation errors are announced by screen readers (aria-live)
- Heading hierarchy is logical (no skipped levels, one h1 per page)
- All text meets 4.5:1 contrast ratio (3:1 for large text)
- Non-text elements meet 3:1 contrast ratio
- Color is never the sole indicator of meaning

## Edge Cases

- **Dynamic content loaded after page load (infinite scroll, lazy-loaded sections)** — axe DevTools only scans the current DOM. Re-run the scan after triggering dynamic content loads. Use aria-live="polite" to announce new content to screen readers.
- **Third-party embedded widgets (Senja testimonials, Stripe billing portal)** — you can't control the accessibility of third-party embeds. Document known issues and provide alternative paths where possible (e.g., link to the Stripe portal instead of embedding it if the embed fails accessibility checks).
- **Dark mode** — run the full contrast audit on both light and dark themes. A color combination that passes in light mode may fail in dark mode.
- **Custom components that override Shadcn defaults** — Shadcn components are accessible by default. If you override styles or structure, re-test the modified component with axe and keyboard navigation.
- **PDF or downloadable documents** — these are outside the scope of this audit. If you serve PDFs, ensure they are tagged for accessibility (separate effort).

## Implementation Checklist

- [ ] axe DevTools extension installed
- [ ] @axe-core/react added to dev dependencies and configured to log violations in development
- [ ] axe audit run on: landing page, sign-in, sign-up, dashboard, settings, billing, onboarding
- [ ] All critical and serious axe issues fixed
- [ ] Moderate and minor issues logged for next cycle
- [ ] Keyboard navigation tested on all key flows
- [ ] Skip-to-content link added to every page
- [ ] Focus trap implemented in all modals
- [ ] Focus returns to trigger element on modal/dropdown close
- [ ] Visible focus indicator on all focusable elements
- [ ] NVDA screen reader testing completed on 4 key flows
- [ ] All images have appropriate alt text
- [ ] All form inputs have associated labels
- [ ] All icon-only buttons have aria-labels
- [ ] aria-live regions added to toast notifications and validation errors
- [ ] Heading hierarchy verified (one h1, no skipped levels)
- [ ] Color contrast verified: 4.5:1 for text, 3:1 for non-text elements
- [ ] Both light and dark themes audited for contrast
- [ ] No information conveyed by color alone
- [ ] Process defined to re-run audit after major UI changes

## Common Mistakes

- Running axe once and considering the audit done (accessibility is ongoing, not a one-time check)
- Fixing automated issues but never testing with a keyboard or screen reader (axe catches 35% of accessibility issues — the rest require manual testing)
- Missing alt text on images or using the filename as alt text ("IMG_4392.png" is not helpful)
- Icon-only buttons with no aria-label (screen readers announce "button" with no context)
- Modals that don't trap focus (Tab key moves behind the modal, confusing screen reader users)
- Focus not returning to the trigger element when a modal closes (user loses their place on the page)
- Skipping heading levels (h1 followed by h3 — screen reader users navigate by headings)
- Passing contrast in light mode but failing in dark mode
- Using color alone to indicate errors (red border with no error text or icon)
- Removing the default browser focus outline without replacing it with a custom one
