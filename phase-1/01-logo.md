# Logo — Design Guidelines & Technical Requirements

---

## Logo Variants

Every SaaS needs these distinct versions:

### Primary Variants
- **Logomark** — icon/symbol only, no text
- **Logotype** — text only, stylized brand name
- **Lockup (horizontal)** — icon + text side by side
- **Lockup (vertical/stacked)** — icon above text

### Color Variants (for each of the above)
- **Full color** — primary brand colors
- **Full color on dark** — adjusted for dark backgrounds (may need lighter tones)
- **Monochrome black** — pure black (#000000)
- **Monochrome white** — pure white (#FFFFFF)
- **Grayscale** — for print contexts where color isn't available
- **Single-color brand** — primary brand color only, no secondary colors

That's **24 files minimum** (4 variants x 6 color treatments).

---

## File Formats

| Format | Use Case | Required |
|---|---|---|
| SVG | Web, app, components, scalable anywhere | Yes |
| PNG (transparent) | Social media, email signatures, docs | Yes |
| PNG (with background) | Platforms that don't support transparency | Yes |
| ICO | Favicon (legacy browsers) | Yes |
| WebP | Web (modern, smaller file size) | Optional |
| PDF | Print, press kits, partner pages | Optional |
| EPS | Print shops, merchandise vendors | Optional |

---

## Size Requirements

### Favicon Sizes
- 16x16 — browser tab (legacy)
- 32x32 — browser tab (standard)
- 48x48 — Windows taskbar
- 96x96 — Google TV
- 180x180 — Apple touch icon
- 192x192 — Android Chrome
- 512x512 — PWA / Android Chrome splash
- `favicon.ico` — multi-size ICO containing 16, 32, 48

### App Icons
- 1024x1024 — master icon (scale down from this)
- 512x512 — Google Play Store
- 180x180 — Apple Touch Icon
- 192x192 — Android homescreen
- 128x128 — Chrome Web Store

### Social Media Profile Images
| Platform | Size (px) | Shape | Notes |
|---|---|---|---|
| Twitter/X | 400x400 | Circle crop | Use logomark, not lockup |
| LinkedIn (company) | 300x300 | Square, rounded | Use logomark |
| GitHub | 500x500 | Circle crop | Use logomark |
| Product Hunt | 240x240 | Circle crop | Use logomark |
| Discord | 512x512 | Circle crop | Use logomark |
| Slack | 512x512 | Square, rounded | Use logomark |
| YouTube | 800x800 | Circle crop | Use logomark |
| Instagram | 320x320 | Circle crop | Use logomark |
| TikTok | 200x200 | Circle crop | Use logomark |
| Facebook | 170x170 | Circle crop | Use logomark |

### Social Media Banners / Covers
| Platform | Size (px) | Notes |
|---|---|---|
| Twitter/X | 1500x500 | Use horizontal lockup, left-aligned or centered |
| LinkedIn (company) | 1128x191 | Use horizontal lockup |
| YouTube | 2560x1440 | Safe area: 1546x423 center |
| Facebook | 820x312 | Displays at 820x312 on desktop, 640x360 on mobile |
| Product Hunt | 1270x220 | Gallery header image |

### Email
- **Email header logo** — max 600px wide, ~50-80px tall (horizontal lockup)
- **Email signature** — max 200px wide, ~40px tall (horizontal lockup)

---

## Design Guidelines

### Minimum Size
- Logomark should remain legible at **16x16px** (favicon use)
- Lockup should remain legible at **100px wide** minimum
- If the logomark has fine details, create a **simplified version** for sizes under 32x32

### Clear Space
- Maintain padding equal to the height of the logomark's "x-height" on all sides
- No other elements should enter this clear space zone
- This applies to all placements — web, print, social

### What Not To Do
- Don't stretch or distort the aspect ratio
- Don't rotate the logo
- Don't change the colors outside the defined variants
- Don't place the full-color version on a busy/clashing background
- Don't add drop shadows, bevels, or effects
- Don't rearrange the icon/text relationship in lockups

### Contrast Requirements
- Logo must meet **WCAG AA contrast ratio (4.5:1)** against its background
- This is why you need both light and dark variants — don't rely on one version everywhere
- Test on: white, black, brand-colored, and photographic backgrounds

### File Naming Convention
```
logo-lockup-horizontal-color.svg
logo-lockup-horizontal-white.svg
logo-lockup-horizontal-black.svg
logo-lockup-vertical-color.svg
logo-lockup-vertical-white.svg
logo-lockup-vertical-black.svg
logo-logomark-color.svg
logo-logomark-white.svg
logo-logomark-black.svg
logo-logotype-color.svg
logo-logotype-white.svg
logo-logotype-black.svg
```

PNG exports follow the same naming with size suffix:
```
logo-logomark-color-512.png
logo-logomark-color-192.png
logo-logomark-color-180.png
```

---

## Folder Structure

```
public/
├── brand/
│   ├── svg/
│   │   ��── logo-lockup-horizontal-color.svg
│   │   ├── logo-lockup-horizontal-white.svg
│   │   ├── logo-lockup-horizontal-black.svg
│   │   ├── logo-lockup-vertical-color.svg
│   │   ├─��� logo-lockup-vertical-white.svg
│   │   ├── logo-lockup-vertical-black.svg
│   │   ├── logo-logomark-color.svg
│   │   ├── logo-logomark-white.svg
│   │   ├── logo-logomark-black.svg
│   │   ├── logo-logotype-color.svg
│   │   ├── logo-logotype-white.svg
│   │   └── logo-logotype-black.svg
│   ├── png/
│   │   ├── logo-logomark-color-512.png
│   │   ├── logo-logomark-color-192.png
│   │   ���── logo-logomark-color-180.png
│   │   └── ...
│   ├── favicon/
│   │   ├── favicon.ico
│   │   ├── favicon-16x16.png
│   │   ├─��� favicon-32x32.png
│   │   ├── apple-touch-icon.png
│   │   ├── android-chrome-192x192.png
│   │   └── android-chrome-512x512.png
│   └── social/
│       ├── twitter-profile.png
│       ├── twitter-banner.png
│       ├── linkedin-profile.png
│       ├── linkedin-banner.png
│       ├── github-profile.png
│       ├── youtube-profile.png
│       ├── youtube-banner.png
│       └── producthunt-profile.png
```

---

## Implementation Checklist

- [ ] Logomark designed and works at 16x16
- [ ] Logotype designed
- [ ] Horizontal lockup designed
- [ ] Vertical lockup designed
- [ ] All 6 color variants created for each
- [ ] SVGs exported and optimized (SVGO)
- [ ] PNGs exported at all required sizes
- [ ] Favicons generated (RealFaviconGenerator)
- [ ] Social media profile images cropped and exported
- [ ] Social media banners designed and exported
- [ ] Email header/signature versions exported
- [ ] Clear space and minimum size documented
- [ ] All files named consistently and organized
