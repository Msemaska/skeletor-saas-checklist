# Skeletor — SaaS Launch Checklist & Reference Architecture

This repo is a comprehensive, opinionated reference for building a SaaS business from idea to fully functioning automated business. It is NOT a codebase — it is a knowledge base of design decisions, guidelines, and checklists.

## Purpose

AI is non-deterministic in system design. Every conversation produces different architecture decisions, misses critical business systems, and makes arbitrary tech choices. This repo eliminates that by pre-making every decision. An AI reading these files should produce nearly identical implementations every time.

## What This Repo Is

- A phased checklist (`saas-launch-checklist.md`) covering every system a SaaS business needs
- Deep-dive reference files for each checklist item (in `phase-X/` folders)
- Opinionated design decisions, not tutorials or documentation
- A living document that gets updated as tools and best practices evolve

## What This Repo Is NOT

- A codebase or starter template
- A collection of options to choose from
- Documentation for a specific project

---

## Writing Rules

### User-First Design Bias

Never pinch pennies at the cost of user experience. Every design decision should favour the user — their trust, their time, their confidence in the product. If a choice saves you money or effort but makes the experience worse, it's the wrong choice.

This means: no surprise charges, no deleting user data to make your life easier, no dark patterns, no friction that exists to serve your metrics instead of the user. Build the product you'd want to use.

### Be Opinionated, Not Educational

Every section should state a decision, not present options. The goal is determinism — an AI reading this should know exactly what to do without making judgment calls.

**Bad:**
> There are two approaches to session management: JWT (stateless) and database sessions (stateful). Each has tradeoffs...

**Good:**
> Use hybrid sessions: JWT for short-lived access tokens, database-backed for refresh and revocation. Both Supabase Auth and Better Auth implement this by default.

### No Code

These files contain design decisions, rules, and architecture guidance expressed in words. No code examples, no snippets, no implementation details. The HOW is decided at coding time based on the current state of the tools and APIs. Code in reference docs goes stale fast and creates false confidence.

Exceptions:
- ASCII diagrams for architecture/flow visualization are fine
- File/folder structure examples are fine
- Configuration examples (like CSS variable names) are fine when they represent a design decision, not implementation

### No Alternatives Unless There's a Genuine Fork

Do not present options that create noise. If email + password + Google OAuth is the answer, don't mention magic links, passkeys, phone auth, or biometrics. Someone who needs those will know to add them.

Only present alternatives when the choice fundamentally changes the approach — like Supabase (managed) vs self-hosted, where the entire infrastructure path differs.

### Timeless Over Trendy

Recommend proven defaults that work for 90% of SaaS products. Skip niche tools, bleeding-edge patterns, and anything that requires explaining why it's better than the obvious choice. If something has been the standard for 3+ years and still works, it's the right recommendation.

### Minimal Context, Maximum Decision

Every section should be as short as possible while still capturing the decision and the reasoning behind it. If something can be said in one line, don't use five. Cut educational context — the reader (AI or human) doesn't need to understand the tradeoffs, it needs to know what to do.

**Bad:**
> OAuth tokens from providers expire independently of your app's session. When a user signs in with Google OAuth, your app creates its own session. Days later, their Google access token may have expired...

**Good:**
> If you only use OAuth for sign-in, provider token expiry is a non-issue — your session is independent. If you access provider APIs post-sign-in, store refresh tokens encrypted and handle expiry gracefully.

### Cover Edge Cases That Aren't Really Edge Cases

Every deep-dive file must include an Edge Cases section listing real-world scenarios that AI consistently misses. These aren't obscure — they're things like "what happens when a user clicks a password reset link after it expires" or "what if someone signs up with an email that already exists." If a real user will hit it within the first month of your product being live, it belongs in this section. Each entry gets a bold scenario name and a one-sentence solution based on the recommended architecture.

### Research Before Writing

When creating or updating deep-dive files, search the web for current best practices, tool comparisons, and industry standards. These files should reflect the current state of the ecosystem, not historical knowledge. Research informs the decisions — but don't cite sources in the files. The value is in the decisions themselves, not in where they came from.

---

## File Structure

```
├── saas-launch-checklist.md          # Master checklist — all phases, all items
├── CLAUDE.md                         # This file — repo rules
├── phase-0/                          # Strategy & Positioning
│   ├── 01-value-proposition.md
│   ├── 02-ideal-customer-profile.md
│   ├── 03-competitor-analysis.md
│   ├── 04-pricing-model.md
│   ├── 05-domain.md
│   └── 06-business-name.md
├── phase-1/                          # Brand Foundation
│   ├── 01-logo.md
│   ├── 02-color-palette.md
│   ├── 03-typography.md
│   ├── 04-favicon-and-app-icons.md
│   └── 05-design-system.md
├── phase-2/                          # Testing
│   ├── 01-test-infrastructure.md
│   ├── 02-unit-integration-testing.md
│   ├── 03-component-testing.md
│   ├── 04-e2e-testing.md
│   └── 05-performance-cross-browser.md
├── phase-3/                          # Core Product
│   ├── 01-database-schema.md
│   ├── 02-validation.md
│   ├── 03-auth.md
│   ├── 04-data-access.md
│   ├── 05-cache.md
│   └── ... (remaining items 06-12)
├── phase-4/                          # UI/UX
├── phase-5/                          # Marketing Site
├── phase-6/                          # Self-Hosted Infrastructure
└── ...                               # Phases 7-20
```

### Naming Conventions

- Folders: `phase-X/` matching the phase number in the checklist
- Files: `{NN}-kebab-case.md` — number prefix matches the item number in the checklist (e.g., `01-database-schema.md`, `03-auth.md`, `05-cache.md`)
- One file per checklist item (or per logical group if items are tightly coupled, like auth covering sign-in + OAuth + 2FA + sessions)

---

## Deep-Dive File Structure

Every deep-dive file follows this pattern:

```
# Title

---

## What It Is
One paragraph. What this system/component is and why it matters.

## [Topic Sections]
Each section covers one aspect of the system.
Each section states design decisions inline — not in a separate section.
Sections should be ordered by importance/dependency.

## Providers (if applicable)
Comparison table of recommended tools.
Only include providers that are genuine options — not exhaustive lists.

## Deliverables / Implementation Checklist
Checkbox list of everything that needs to be done.
Grouped by logical category if the list is long.

## What to Test
Specifies what must be tested for this system. Lives next to the feature, not in a separate testing phase.
Every edge case in the Edge Cases section must have a corresponding test.
Every error in Common Mistakes must be verified as not-happening via a test.
AI must not skip these when implementing the feature.

## Edge Cases
Scenarios that happen in every SaaS but get missed unless explicitly handled.
Each entry: bold scenario name + one-sentence solution.
These are not theoretical — they are things real users will hit.
If an AI implements this system without handling these, it shipped a broken product.

## Common Mistakes
Bullet list of things that go wrong. Practical, not theoretical.
```

### What NOT to include in deep-dive files

- Code examples or snippets
- Step-by-step tutorials
- Historical context ("in 2024, people used to...")
- Comparisons with tools we've decided against
- "You could also..." suggestions
- Theoretical edge cases that don't affect 90% of SaaS products (but DO include common real-world scenarios that users will actually hit — these go in the Edge Cases section)

---

## Checklist Conventions

The master checklist (`saas-launch-checklist.md`) uses these markers:

- `[1]` = Need whilst developing
- `[2]` = Need before launch
- `[3]` = Need after launch

Format: `- \`[X]\` Item description — **Recommended Tool**`

Phases marked with "(if applicable)" or "(if B2B)" are conditional — they may not apply to every project.

---

## Updating the Repo

- When adding a new checklist item, add it to the correct phase in `saas-launch-checklist.md` with the appropriate `[1/2/3]` marker and tool recommendation
- When creating a deep-dive file, follow the file structure above and place it in the correct `phase-X/` folder
- When a tool or best practice changes, update both the checklist AND the relevant deep-dive file
- When removing something, remove it from both places
- Keep phase numbering consistent — if phases are reordered, renumber everything
