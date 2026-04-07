# Onboarding

---

## What It Is

How new users go from signing up to getting value from your product. If a user signs up and lands on a blank dashboard with no guidance, they leave. Onboarding bridges the gap between "I have an account" and "I understand how to use this."

## Onboarding Layers

Onboarding is not a single feature — it's three layers that work together:

1. **First-run wizard** — a guided setup flow immediately after sign-up that collects essential information and gets the user to their first meaningful action
2. **Tooltip tour** — contextual hints that appear as the user explores the product for the first time, pointing out key features
3. **Progressive disclosure** — showing only what's relevant at each stage, revealing more features as the user becomes familiar

## First-Run Wizard

A stepped flow that appears immediately after the user signs up and verifies their email. The goal is to get the user to their "aha moment" — the first time they experience the product's value.

**Design decisions:**
- The wizard runs once per user, tracked by a `has_completed_onboarding` flag on the user's profile in the database (not localStorage — it must persist across devices)
- Each step represents one distinct action the user needs to take or one concept they need to understand. Steps are defined by what the user needs, not by an arbitrary number.
- Every step must earn its place — if a step can be skipped or deferred to settings, remove it from the wizard.
- Each step collects information the product needs to function or personalise the experience. Don't collect information you don't immediately use.

**Typical steps:**
1. **Workspace setup** — workspace name (auto-generates slug). If multi-tenant, this creates their first workspace. If single-tenant, this sets up their account.
2. **Profile basics** — display name, avatar (if not already provided via OAuth). Skip this step entirely if the user signed up with Google OAuth and you already have their name and photo.
3. **First action** — guide the user to create their first entity (project, document, whatever your product's core object is). This is the most important step — it transforms them from a passive account holder to an active user.

**What NOT to include in the wizard:**
- Billing/plan selection (they just signed up — let them use the product first)
- Notification preferences (defaults are fine, they'll adjust later)
- Integrations setup (too much complexity for day one)
- Feature explanations (that's what the tooltip tour is for)

**Behaviour:**
- The wizard is not skippable. If they can skip it entirely, they land on an empty dashboard with no context. Each step can have optional fields, but the flow itself must be completed.
- If the user closes the browser mid-wizard and comes back, resume where they left off (progress is saved server-side per step)
- After completing the wizard, redirect to the dashboard or the entity they just created — not back to a generic home screen
- The wizard uses the same form patterns as the rest of the app (covered in phase-4/04-data-input-display.md)

## Tooltip Tour

Contextual hints that appear the first time a user encounters a feature. Unlike the wizard (which is a dedicated flow), the tooltip tour is embedded in the actual product UI.

**Design decisions:**
- The tour triggers after the wizard is complete, on the first page the user lands on (typically the dashboard)
- Maximum 5 tooltips per tour. More than that and the user dismisses without reading.
- Each tooltip highlights one element on the page with a brief explanation (1-2 sentences max)
- Tooltips appear sequentially — the user clicks "Next" to advance, "Skip" to dismiss the rest
- Track tour completion per user in the database (`has_completed_tour` flag)
- The tour is re-triggerable from a "Help" or "?" menu in the app — users who dismissed too quickly can restart it

**Tooltip content rules:**
- Point out WHERE things are, not HOW they work in detail ("This is your sidebar — your projects, settings, and team are all accessible from here")
- Focus on navigation and discoverability — the features the user would take longest to find on their own
- Don't tooltip obvious things (the sign-out button, the search bar)
- Each tooltip should be useful standalone — if the user sees only one, it should still help

**What to highlight:**
- Sidebar navigation and what each section contains
- The primary create action (where to create a new entity)
- Settings location
- The notification bell / inbox
- The command palette shortcut (Cmd+K)

## Progressive Disclosure

Don't show everything at once. Reveal features as the user needs them.

**Design decisions:**
- New users see a simplified UI with fewer options. Advanced features appear after the user has completed basic actions.
- Use empty states as progressive disclosure — each empty state teaches the user about that feature when they first encounter it (covered in phase-4/03-component-states.md)
- Don't hide features permanently — progressive disclosure delays exposure, it doesn't gate it. A user should be able to find any feature if they look for it.
- Track feature discovery milestones in the database (first project created, first member invited, first integration connected). Use these to trigger contextual prompts.

**Examples of progressive disclosure:**
- Empty states introduce features when the user first encounters them — the empty state IS the progressive disclosure (covered in phase-4/03-component-states.md)
- Show contextual prompts within page content when a feature the user hasn't tried yet could help with what they're currently doing (e.g., a banner suggesting team invitations after creating their third project)
- Navigation items in the sidebar remain static — don't hide or show sidebar items based on user progress. Progressive disclosure happens within page content, not by altering navigation.

## Sample Data vs Empty Start

**Decision: empty start with guided first action.**

Don't pre-populate the user's workspace with sample data. Sample data creates confusion ("is this mine? can I delete it? what's real?"), adds cleanup burden, and teaches the user to use fake content instead of their own.

Instead, the first-run wizard guides the user to create their first real entity. The empty states (phase-4/03-component-states.md) handle what every screen looks like before the user has created content.

The one exception: if your product requires non-trivial setup before the user can create anything (templates, categories, configuration), provide sensible defaults that the user can customise — not sample data they have to delete.

## Measuring Onboarding Success

Track these events via product analytics (PostHog, covered in Phase 9 Observability):

- **Wizard completion rate** — percentage of sign-ups that complete the wizard
- **Time to first action** — duration between sign-up and creating the first entity
- **Day-1 retention** — percentage of users who return the day after signing up
- **Tour completion rate** — percentage of users who finish the tooltip tour vs dismissing it

## What to Test

- Wizard triggers on first sign-in after sign-up
- Wizard does not re-trigger after completion (has_completed_onboarding flag)
- OAuth user skips profile step if name/avatar already provided
- Invited user skips workspace creation step
- Wizard progress resumes after browser close and re-login
- Tooltip tour triggers after wizard completion on first dashboard visit
- Tooltip tour does not re-trigger after completion (has_completed_tour flag)
- Tour is re-triggerable from Help menu
- Tooltips work on mobile (bottom sheets, not hover-positioned)

## Edge Cases

- **User signs up via OAuth and already has name + avatar** — skip the profile step in the wizard. Don't ask for information you already have.
- **User is invited to an existing workspace** — they don't need the workspace creation step. Adapt the wizard to skip workspace setup and start with profile basics → first action within the existing workspace.
- **User completes wizard then signs out and signs back in** — the wizard does not reappear. The `has_completed_onboarding` flag persists.
- **User's first entity creation fails** — if the "create your first project" step fails (server error, validation error), show the error inline and let them retry. Don't advance past the step or skip it.
- **Multiple users joining the same workspace** — the first user (workspace creator) gets the full wizard. Subsequent users invited to the workspace get a shortened wizard (profile basics + tour of the existing workspace).
- **User dismisses the tooltip tour** — record the dismissal. Don't re-trigger it on next page load. The user can restart it from the Help menu if they want.
- **Product adds new features after the user completed onboarding** — the tooltip tour should not re-trigger for new features. Use a "What's new" changelog notification instead (covered in phase-3/07-notification-system.md).
- **User on mobile completes onboarding** — the wizard and tooltip tour must work on mobile. Tooltips on mobile appear as bottom sheets or cards rather than pointing at specific elements (touch targets are different from hover targets).

## Implementation Checklist

### First-Run Wizard
- [ ] `has_completed_onboarding` flag on user profile in database
- [ ] Wizard triggers immediately after sign-up / email verification
- [ ] Each step represents one distinct action or concept
- [ ] Progress saved server-side per step (resume on return)
- [ ] Wizard is not skippable (individual fields can be optional, flow cannot be skipped)
- [ ] OAuth users skip profile step if name/avatar already provided
- [ ] Invited users skip workspace creation step
- [ ] Redirects to dashboard or first entity after completion

### Tooltip Tour
- [ ] Tour triggers after wizard completion on first dashboard visit
- [ ] Maximum 5 tooltips
- [ ] Sequential with Next/Skip controls
- [ ] `has_completed_tour` flag in database
- [ ] Re-triggerable from Help menu
- [ ] Works on mobile (bottom sheets, not hover-positioned tooltips)

### Progressive Disclosure
- [ ] Simplified UI for new users (fewer options visible)
- [ ] Feature discovery milestones tracked in database
- [ ] Contextual prompts triggered by milestones
- [ ] Empty states serve as feature introductions

### Analytics
- [ ] Wizard completion rate tracked
- [ ] Time to first action tracked
- [ ] Tour completion vs dismissal rate tracked

## Common Mistakes

- Skippable wizard that leads to a blank dashboard (user has no context, leaves immediately)
- Steps that bundle multiple actions into one (each step should be one thing the user does or learns)
- Asking for information you don't immediately use (notification preferences, integrations — defer to settings)
- Pre-populating with sample data (creates confusion and cleanup burden)
- Tooltip tour with 10+ tooltips (user stops reading after 3, dismisses the rest)
- Tour that re-triggers every time the user logs in (tracked in localStorage instead of database, resets on new device)
- Tooltips explaining obvious UI elements (the big red "Delete" button doesn't need a tooltip)
- No way to restart the tooltip tour (user dismissed it too quickly and now can't find features)
- Treating onboarding as a one-time build (onboarding should evolve as the product evolves — measure and iterate)
- Same wizard for workspace creators and invited members (different contexts, different needs)
