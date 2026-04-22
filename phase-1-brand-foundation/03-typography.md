# Typography

---

## What It Is

Your font choices and how you use them. Typography is the highest-leverage design decision for a SaaS — it affects perceived quality more than almost anything else.

## 2026 Trends

- **Inter is still king for SaaS UI** — purpose-built for screens, excellent at small sizes, massive language support. If in doubt, use Inter.
- **Serif headlines are back** — pairing a serif heading font with a sans-serif body creates visual hierarchy and makes your marketing pages feel more premium. Not for the app UI — for landing pages and blog posts.
- **Variable fonts** — single file, multiple weights. Better performance than loading 4 separate font files. Most modern Google Fonts are variable now.
- **Performance is non-negotiable** — font loading directly impacts Core Web Vitals (CLS, LCP). Load only the weights you use. Use `font-display: swap`. Subset if possible.
- **System font stacks as fallback** — always define a fallback stack so text renders instantly before web fonts load.

## Choosing Fonts

### For App UI (the product itself)

Pick ONE sans-serif font. Use weight and size for hierarchy, not multiple font families.

**Top picks for SaaS in 2026:**

| Font | Vibe | Notes |
|---|---|---|
| **Inter** | Clean, neutral, professional | The default choice. Hard to go wrong. Best for data-heavy UIs. |
| **Geist Sans** | Modern, technical, Vercel-style | Designed by Vercel. Sharp and contemporary. |
| **Plus Jakarta Sans** | Friendly, rounded, approachable | Geometric, slightly softer than Inter. Good for consumer SaaS. |
| **DM Sans** | Geometric, minimal | Clean geometry, works well at larger sizes. |
| **Satoshi** | Premium, distinctive | Not on Google Fonts (Fontshare). Stands out without being loud. |
| **General Sans** | Modern, neutral-warm | Similar to Satoshi but slightly friendlier. Fontshare. |

### For Marketing Site (landing page, blog)

Option A: **Same font as app** — consistent, simple, one less font to load.

Option B: **Serif heading + sans-serif body** — adds personality and premiumness.

**Serif fonts that pair well with the above sans-serifs:**

| Serif Font | Pairs With | Vibe |
|---|---|---|
| **Fraunces** | Inter, DM Sans | Playful, distinctive, variable |
| **Playfair Display** | Inter, Geist | Elegant, high contrast |
| **Lora** | Inter, Plus Jakarta | Warm, readable, classic |
| **Source Serif 4** | Geist, Inter | Professional, neutral serif |
| **Instrument Serif** | Geist, DM Sans | Modern, editorial |

### Proven Pairings

| Heading | Body | Vibe |
|---|---|---|
| Inter (Bold) | Inter (Regular) | Safe, professional, SaaS standard |
| Fraunces | Inter | Distinctive, premium |
| Instrument Serif | Geist Sans | Modern editorial |
| Plus Jakarta Sans (Bold) | Plus Jakarta Sans (Regular) | Friendly, consumer |
| Playfair Display | DM Sans | Elegant, authoritative |

## Type Scale

Use a consistent scale. Don't pick arbitrary font sizes. Tailwind's default scale works, but here's a recommended SaaS type scale:

| Token | Size | Line Height | Weight | Use |
|---|---|---|---|---|
| `text-xs` | 12px | 16px | 400 | Captions, labels, badges |
| `text-sm` | 14px | 20px | 400 | Secondary text, table cells, descriptions |
| `text-base` | 16px | 24px | 400 | Body text (default) |
| `text-lg` | 18px | 28px | 500 | Emphasized body, card titles |
| `text-xl` | 20px | 28px | 600 | Section headers |
| `text-2xl` | 24px | 32px | 600 | Page headers |
| `text-3xl` | 30px | 36px | 700 | Marketing subheadings |
| `text-4xl` | 36px | 40px | 700 | Marketing headings |
| `text-5xl` | 48px | 48px | 800 | Hero headlines |
| `text-6xl` | 60px | 60px | 800 | Marketing hero (desktop only) |

**Rules:**
- Body text: minimum 16px. Never smaller for primary content.
- Line height: 1.5x font size for body, tighter (1.1-1.2x) for headings.
- Letter spacing: slightly tighter for headings (`tracking-tight`), default for body.
- Max line width: 65-75 characters (`max-w-prose` in Tailwind = 65ch).

## Implementation in Next.js

### Google Fonts (recommended approach)

```tsx
// app/layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
})

export default function RootLayout({ children }) {
  return (
    <html className={inter.variable}>
      <body>{children}</body>
    </html>
  )
}
```

### Tailwind v4 Config

```css
@theme {
  --font-sans: var(--font-inter), ui-sans-serif, system-ui, sans-serif;
  --font-serif: 'Instrument Serif', ui-serif, Georgia, serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;
}
```

### If Using Two Fonts

```tsx
import { Inter } from 'next/font/google'
import { Instrument_Serif } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
})

const instrumentSerif = Instrument_Serif({
  subsets: ['latin'],
  weight: '400',
  display: 'swap',
  variable: '--font-serif',
})

export default function RootLayout({ children }) {
  return (
    <html className={`${inter.variable} ${instrumentSerif.variable}`}>
      <body>{children}</body>
    </html>
  )
}
```

## Performance

- **Load only weights you use** — if you only need 400, 500, 600, 700, don't load the full range
- **Use `next/font`** — auto-optimizes, self-hosts, eliminates layout shift
- **Variable fonts** — one file covers all weights, smaller total download
- **Subset** — `subsets: ['latin']` unless you need other character sets
- **`display: 'swap'`** — text renders immediately with fallback, swaps when font loads
- **Max 2 font families** — each additional font is another network request

## Deliverables

- [ ] Primary font chosen (app UI)
- [ ] Secondary font chosen (marketing headings, if using two fonts)
- [ ] Monospace font chosen (for code blocks, if applicable)
- [ ] Type scale defined (sizes, weights, line heights)
- [ ] Fonts loaded via `next/font` with proper subsets
- [ ] CSS variables defined in Tailwind v4 @theme
- [ ] Fallback font stack specified
- [ ] Performance verified (no CLS from font loading)
- [ ] Both fonts tested at all sizes on mobile and desktop

## Common Mistakes

- Loading too many font weights (400 and 700 cover 90% of needs)
- Body text smaller than 16px (unreadable on mobile)
- No fallback stack (flash of invisible text while font loads)
- Using a display font for body text (looks cool in the hero, illegible in paragraphs)
- Tight line-height on body text (1.5 minimum for readability)
- Not testing on real devices (fonts render differently across OS and browsers)
- Importing from Google Fonts CDN instead of `next/font` (extra DNS lookup, no self-hosting)
