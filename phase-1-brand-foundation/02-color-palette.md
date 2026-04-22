# Color Palette

---

## What It Is

A structured set of colors that defines your brand and UI. Not just "pick some nice colors" — it's a system of tokens that ensures consistency across your entire product, marketing site, emails, and social assets.

## 2026 Trends

- **OKLCH over HSL** — Shadcn and Tailwind v4 now recommend OKLCH color space. It's perceptually uniform (meaning lightness steps look even to human eyes), generates better color scales, and avoids the "muddy middle" problem HSL has when interpolating between colors.
- **Warm neutrals** — Pantone's 2026 Color of the Year is Cloud Dancer (a warm off-white). Pure #FFFFFF interfaces are giving way to subtly warm or cool neutrals that reduce glare and feel more refined.
- **Bold accent, restrained base** — the "dopamine aesthetic" is in: saturated, high-contrast accents (electric purple, vivid coral, neon green) paired with muted neutral foundations. The contrast grabs attention; the neutrals keep it usable.
- **Dark mode is mandatory** — not a nice-to-have. Users expect it. Design your palette for both from day one.

## The System

You need exactly these color categories:

### 1. Brand Colors
- **Primary** — your main brand color. Used for CTAs, links, active states, primary buttons.
- **Secondary** — complementary to primary. Used for secondary actions, tags, highlights.

### 2. Neutral Scale
- A full scale from near-white to near-black (10-12 steps)
- Used for: backgrounds, borders, text, cards, dividers
- Pick a base: Zinc, Slate, Neutral, Stone (Shadcn offers these out of the box)
- **Tip:** avoid pure gray. A slight warm (Stone) or cool (Slate) undertone looks more polished.

### 3. Semantic Colors
- **Success** — green. Confirmations, positive states, completed actions.
- **Warning** — amber/yellow. Caution states, approaching limits.
- **Error/Destructive** — red. Errors, deletions, failed states.
- **Info** — blue. Informational banners, tooltips, neutral alerts.

### 4. Surface Colors
- **Background** — page background
- **Card** — elevated surface
- **Popover** — dropdown/modal surface
- **Muted** — subtle background (empty states, disabled areas)

## Building the Palette

### Step 1: Pick Your Primary Color

Start with one color that represents your brand's personality:

| Vibe | Color Direction |
|---|---|
| Trust, professional, corporate | Blue (most common SaaS choice for a reason) |
| Creative, bold, premium | Purple / violet |
| Growth, health, money | Green |
| Energy, urgency, passion | Red / coral (use carefully — aggressive) |
| Warmth, friendly, approachable | Orange / amber |
| Modern, minimal, tech | Monochrome with a single accent |

### Step 2: Generate Your Scale in OKLCH

For each color, generate a 10-step scale (50-950) using OKLCH:

```css
/* Example: Primary blue in OKLCH */
--primary-50:  oklch(0.97 0.01 250);
--primary-100: oklch(0.93 0.02 250);
--primary-200: oklch(0.87 0.04 250);
--primary-300: oklch(0.77 0.08 250);
--primary-400: oklch(0.65 0.13 250);
--primary-500: oklch(0.55 0.17 250);  /* base */
--primary-600: oklch(0.47 0.17 250);
--primary-700: oklch(0.39 0.15 250);
--primary-800: oklch(0.32 0.12 250);
--primary-900: oklch(0.25 0.08 250);
--primary-950: oklch(0.18 0.05 250);
```

Keep the hue (last number) constant. Adjust lightness (first) and chroma (second) for each step. The middle value (500) is your base — lighter values go up, darker go down.

### Step 3: Define Shadcn Theme Variables

Shadcn uses a specific set of CSS variables. In Tailwind v4, define them with `@theme`:

```css
@theme {
  --color-background: oklch(0.98 0.005 250);
  --color-foreground: oklch(0.15 0.01 250);
  --color-card: oklch(0.98 0.005 250);
  --color-card-foreground: oklch(0.15 0.01 250);
  --color-popover: oklch(0.98 0.005 250);
  --color-popover-foreground: oklch(0.15 0.01 250);
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
}
```

### Step 4: Dark Mode Variants

Every variable needs a dark mode counterpart. The pattern: swap lightness values (light backgrounds become dark, dark text becomes light):

```css
.dark {
  --color-background: oklch(0.15 0.01 250);
  --color-foreground: oklch(0.93 0.005 250);
  --color-card: oklch(0.18 0.01 250);
  --color-primary: oklch(0.65 0.17 250);  /* slightly lighter primary for dark bg */
  --color-destructive: oklch(0.65 0.2 25);
  /* ... etc */
}
```

**Dark mode tips:**
- Don't just invert. Dark backgrounds should be slightly lighter than pure black (~oklch(0.15-0.18)).
- Reduce chroma slightly on dark backgrounds (saturated colors glow too much).
- Increase primary lightness by ~10% so it pops against dark surfaces.
- Borders should be subtle (low contrast) in dark mode.

### Step 5: Contrast Check

Every text/background combination must pass WCAG AA:

| Combination | Required Ratio |
|---|---|
| Body text on background | 4.5:1 minimum |
| Large text (18px+) on background | 3:1 minimum |
| UI elements (icons, borders) | 3:1 minimum |
| Primary button text on primary bg | 4.5:1 minimum |

Test with: browser DevTools contrast checker, or axe DevTools.

## Tools

- **Shadcn Themes** — https://ui.shadcn.com/themes (generate a full theme visually)
- **OKLCH Color Picker** — https://oklch.com (explore OKLCH space)
- **Tailwind Colors** — https://ui.shadcn.com/colors (all Tailwind colors in every format)
- **Realtime Colors** — https://www.realtimecolors.com (preview palette on a real page layout)

## Deliverables

- [ ] Primary brand color chosen
- [ ] Secondary color chosen
- [ ] Neutral scale selected (Zinc, Slate, Stone, or custom)
- [ ] Semantic colors defined (success, warning, error, info)
- [ ] All Shadcn CSS variables defined in OKLCH
- [ ] Dark mode variants for all variables
- [ ] Contrast ratios verified (WCAG AA minimum)
- [ ] Colors added to Tailwind config via @theme
- [ ] Palette documented (color swatches with names and values)

## Common Mistakes

- Too many colors (5-7 total is enough: primary, secondary, neutral scale, semantics)
- Not testing on real UI (a color that looks great in a swatch can look wrong on a button)
- Ignoring dark mode until later (retrofitting dark mode is painful)
- Using pure black (#000) for text (too harsh — use your darkest neutral, like 950)
- Using pure white (#FFF) for backgrounds (a subtle off-white feels more premium)
- Not checking contrast (your beautiful light-gray-on-white text is invisible to 15% of users)
- HSL instead of OKLCH (HSL scales look uneven — OKLCH gives perceptually uniform steps)
