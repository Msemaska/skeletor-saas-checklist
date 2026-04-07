# App UI Design System

---

## What It Is

The set of tokens, components, and patterns that ensure your entire app looks and behaves consistently. Built on Shadcn + Tailwind, this is the system that every screen in your product draws from.

## 2026 Architecture: Shadcn + Tailwind v4

### The Stack

- **Tailwind v4** — CSS-first config, `@theme` directive for design tokens, native CSS variables
- **Shadcn** — unstyled Radix primitives, copied into your codebase, fully customizable
- **class-variance-authority (CVA)** — variant management for component styles
- **tailwind-merge** — deduplicates Tailwind classes at runtime
- **clsx** — conditional class joining

### Component Organization (2026 Best Practice)

Shadcn components live in your codebase. Organize them in three tiers:

```
components/
├── ui/              # Raw Shadcn components (don't edit directly)
│   ├── button.tsx
│   ├── input.tsx
│   ├── dialog.tsx
│   └── ...
├── primitives/      # Lightly customized Shadcn components
│   ├── formField.tsx    (input + label + error message)
│   ├── dataTable.tsx    (table + sorting + pagination)
│   └── ...
├── common/          # Product-level compositions
│   ├── navbar.tsx
│   ├── sidebar.tsx
│   └── ...
└── [feature]/       # Feature-scoped compositions
    ├── dashboard/
    ├── settings/
    └── ...
```

**Rules:**
- `ui/` — Shadcn output. Update via `npx shadcn@latest add`. Avoid editing these directly.
- `primitives/` — your customized building blocks. Combine 2-3 Shadcn components into reusable patterns.
- `common/` — app-wide layouts and navigation.
- `[feature]/` — components scoped to a specific feature.

## Design Tokens in Tailwind v4

### The @theme Directive

Tailwind v4 replaces `tailwind.config.js` with CSS-native `@theme`. Tokens defined here automatically generate utility classes:

```css
/* app/globals.css */

@import "tailwindcss";

@theme {
  /* Colors — defined in OKLCH for perceptual uniformity */
  --color-background: oklch(0.98 0.005 250);
  --color-foreground: oklch(0.15 0.01 250);
  --color-primary: oklch(0.55 0.17 250);
  --color-primary-foreground: oklch(0.98 0.005 250);
  --color-secondary: oklch(0.93 0.02 250);
  --color-secondary-foreground: oklch(0.15 0.01 250);
  --color-muted: oklch(0.93 0.02 250);
  --color-muted-foreground: oklch(0.45 0.03 250);
  --color-accent: oklch(0.93 0.02 250);
  --color-accent-foreground: oklch(0.15 0.01 250);
  --color-destructive: oklch(0.55 0.2 25);
  --color-destructive-foreground: oklch(0.98 0.005 250);
  --color-border: oklch(0.88 0.02 250);
  --color-input: oklch(0.88 0.02 250);
  --color-ring: oklch(0.55 0.17 250);
  --color-card: oklch(0.98 0.005 250);
  --color-card-foreground: oklch(0.15 0.01 250);
  --color-popover: oklch(0.98 0.005 250);
  --color-popover-foreground: oklch(0.15 0.01 250);

  /* Semantic colors */
  --color-success: oklch(0.55 0.15 145);
  --color-warning: oklch(0.75 0.15 80);
  --color-info: oklch(0.60 0.15 240);

  /* Typography */
  --font-sans: var(--font-inter), ui-sans-serif, system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;

  /* Radius — controls the overall roundness of your UI */
  --radius-sm: 0.25rem;
  --radius-md: 0.375rem;
  --radius-lg: 0.5rem;
  --radius-xl: 0.75rem;
  --radius-2xl: 1rem;
  --radius-full: 9999px;

  /* Spacing — extend if needed */
  /* Tailwind v4 has built-in spacing, only add custom values */

  /* Shadows */
  --shadow-sm: 0 1px 2px oklch(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px oklch(0 0 0 / 0.07);
  --shadow-lg: 0 10px 15px oklch(0 0 0 / 0.1);

  /* Animation */
  --animate-accordion-down: accordion-down 0.2s ease-out;
  --animate-accordion-up: accordion-up 0.2s ease-out;
}
```

### Dark Mode

```css
.dark {
  --color-background: oklch(0.13 0.01 250);
  --color-foreground: oklch(0.93 0.005 250);
  --color-card: oklch(0.16 0.01 250);
  --color-card-foreground: oklch(0.93 0.005 250);
  --color-popover: oklch(0.16 0.01 250);
  --color-popover-foreground: oklch(0.93 0.005 250);
  --color-primary: oklch(0.65 0.17 250);
  --color-primary-foreground: oklch(0.13 0.01 250);
  --color-secondary: oklch(0.20 0.01 250);
  --color-secondary-foreground: oklch(0.93 0.005 250);
  --color-muted: oklch(0.20 0.01 250);
  --color-muted-foreground: oklch(0.60 0.02 250);
  --color-accent: oklch(0.20 0.01 250);
  --color-accent-foreground: oklch(0.93 0.005 250);
  --color-destructive: oklch(0.65 0.2 25);
  --color-destructive-foreground: oklch(0.93 0.005 250);
  --color-border: oklch(0.25 0.01 250);
  --color-input: oklch(0.25 0.01 250);
  --color-ring: oklch(0.65 0.17 250);

  --color-success: oklch(0.65 0.15 145);
  --color-warning: oklch(0.80 0.15 80);
  --color-info: oklch(0.70 0.15 240);
}
```

### Three Token Layers

Structure your tokens in three layers for scalability:

```css
/* Layer 1: Base tokens (raw values, never used in components directly) */
:root {
  --blue-500: oklch(0.55 0.17 250);
  --blue-600: oklch(0.47 0.17 250);
  --red-500: oklch(0.55 0.2 25);
}

/* Layer 2: Semantic tokens (purpose-driven, used by @theme) */
@theme {
  --color-primary: var(--blue-500);
  --color-destructive: var(--red-500);
}

/* Layer 3: Component tokens (if needed for complex components) */
.btn-primary {
  --btn-bg: var(--color-primary);
  --btn-hover: var(--blue-600);
}
```

This means changing your brand color updates every component that references `--color-primary`.

## Spacing System

Use Tailwind's built-in 4px grid. Don't invent custom spacing:

| Token | Value | Use |
|---|---|---|
| `1` | 4px | Tight gaps (icon to label) |
| `2` | 8px | Small gaps (between related items) |
| `3` | 12px | Medium-small gaps |
| `4` | 16px | Standard padding, gaps |
| `6` | 24px | Section padding, card padding |
| `8` | 32px | Large section gaps |
| `12` | 48px | Section separators |
| `16` | 64px | Page section spacing |
| `24` | 96px | Hero section spacing |

**Rules:**
- Card padding: `p-4` (mobile) / `p-6` (desktop)
- Page padding: `px-4` (mobile) / `px-6` (tablet) / `px-8` (desktop)
- Max content width: `max-w-7xl` (1280px) for marketing, `max-w-full` for app
- Stack spacing: `space-y-4` for form fields, `space-y-6` for sections

## Border Radius System

Pick a radius personality and stick with it:

| Vibe | Radius | Example |
|---|---|---|
| Sharp, professional | `rounded-sm` to `rounded-md` (2-6px) | Linear, Notion |
| Balanced, modern | `rounded-lg` (8px) | Most SaaS |
| Soft, friendly | `rounded-xl` to `rounded-2xl` (12-16px) | Consumer apps |
| Pill/full | `rounded-full` | Tags, badges, avatars |

**Consistency rule:** buttons, inputs, cards, modals should all share the same base radius. Don't mix `rounded-md` buttons with `rounded-xl` cards.

## Shadow System

Shadows create depth. Use sparingly:

| Level | Use | Tailwind |
|---|---|---|
| No shadow | Default state for most elements | (none) |
| `shadow-sm` | Cards, elevated surfaces | Subtle lift |
| `shadow-md` | Dropdowns, popovers | Clear elevation |
| `shadow-lg` | Modals, command palettes | Floating above content |

**2026 trend:** less shadow, more border. Many modern SaaS apps use subtle borders (`border border-border`) instead of shadows for cards. Shadows reserved for truly floating elements (dropdowns, modals).

## Animation Defaults

Shadcn uses `tailwindcss-animate`. Keep animations subtle and fast:

```css
@theme {
  --animate-in: enter 0.2s ease-out;
  --animate-out: exit 0.15s ease-in;
}
```

**Rules:**
- Duration: 150-300ms for UI transitions. Never longer.
- Easing: `ease-out` for entrances, `ease-in` for exits, `ease-in-out` for state changes.
- Reduce motion: always respect `prefers-reduced-motion`. Shadcn handles this by default.
- Don't animate layout (width, height) — animate opacity and transform only.

## Responsive Breakpoints

Tailwind's defaults, mobile-first:

| Prefix | Min Width | Target |
|---|---|---|
| *(none)* | 0px | Mobile (default) |
| `sm:` | 640px | Large phones, small tablets |
| `md:` | 768px | Tablets |
| `lg:` | 1024px | Laptops |
| `xl:` | 1280px | Desktops |
| `2xl:` | 1536px | Large screens |

**Rules:**
- Write mobile styles first, layer up with breakpoints
- App sidebar: hidden on mobile, visible at `lg:`
- Navigation: hamburger on mobile, full nav at `md:`
- Grid: 1 column mobile, 2 at `md:`, 3-4 at `lg:`

## The cn() Utility

Standard helper for merging Tailwind classes:

```tsx
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

Use this everywhere components accept `className` props.

## Dark Mode

- Use `next-themes` for theme management
- Three options: System (follows OS preference), Light, Dark. Default to System.
- Theme toggle in the user menu (not in settings — it should be quick to switch)
- Store preference in localStorage via next-themes (handled automatically)
- All colours are defined as CSS variables with light and dark variants (covered in phase-1/02-color-palette.md). The `.dark` class on the HTML element swaps the variable values.
- Test every component and page in both modes — dark mode is not an afterthought, it ships with the same quality as light mode
- Avoid pure black (#000) backgrounds in dark mode — use your darkest neutral (oklch 0.13-0.15) for a softer appearance

## Deliverables

- [ ] @theme tokens defined (colors, fonts, radius, shadows)
- [ ] Dark mode variables defined
- [ ] Shadcn installed and base components added
- [ ] Component folder structure set up (ui/, primitives/, common/, features/)
- [ ] cn() utility created
- [ ] Spacing system documented (which values to use where)
- [ ] Border radius personality chosen and consistent
- [ ] Animation defaults configured
- [ ] `globals.css` organized with @theme, dark mode, base styles
- [ ] Tested in light mode, dark mode, mobile, desktop

## Common Mistakes

- Editing `ui/` components directly (they get overwritten on Shadcn updates — customize in primitives/)
- Hardcoding colors instead of using tokens (`bg-blue-500` instead of `bg-primary`)
- Inconsistent radius (mixing rounded-md and rounded-xl randomly)
- Over-animating (every hover, every mount, every state change — exhausting)
- Not using the three-layer token system (changing brand color requires updating 47 files)
- Ignoring mobile-first (building desktop then "fixing" mobile is always harder)
- Using `@apply` in Tailwind v4 (deprecated pattern — use explicit CSS or utility classes)
- Building custom components when Shadcn already has one (always check first)
