# Favicon & App Icons

---

## What It Is

The small icons that represent your app across browsers, operating systems, bookmarks, home screens, and PWA installs. In 2026 the required set is smaller and clearer than older guides suggest.

## 2026 Minimal Required Set

You only need **6 files** for full coverage:

| File | Size | Purpose |
|---|---|---|
| `favicon.ico` | 32x32 (or multi-size 16+32) | Legacy browser fallback, placed in site root |
| `favicon.svg` | Scalable | Modern browsers, supports dark mode adaptation |
| `favicon-16x16.png` | 16x16 | Browser tab (some contexts) |
| `favicon-32x32.png` | 32x32 | Browser tab (standard) |
| `apple-touch-icon.png` | 180x180 | iOS home screen, Safari |
| `icon-192x192.png` | 192x192 | Android Chrome, PWA manifest |
| `icon-512x512.png` | 512x512 | PWA splash screen, Android install |

**Sizes you can skip in 2026:** 48x48, 96x96, 128x128, 144x144, 384x384. These were needed for older Android/Windows scenarios that are no longer relevant for standard web deployment.

## SVG Favicon with Dark Mode

The biggest 2026 upgrade: SVG favicons support `prefers-color-scheme`, meaning your favicon can adapt to dark mode automatically:

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32">
  <style>
    .icon { fill: #1a1a2e; }
    @media (prefers-color-scheme: dark) {
      .icon { fill: #e0e0e0; }
    }
  </style>
  <rect class="icon" width="32" height="32" rx="6" />
  <!-- your logomark paths here -->
</svg>
```

**Browser support:** Chrome, Firefox, Edge all support SVG favicons. Safari still requires the apple-touch-icon PNG fallback.

## PWA Maskable Icons

If your app is installable as a PWA, Android uses "maskable" icons that can be cropped into circles, rounded squares, or other shapes by the OS.

**Safe area rule:** keep your actual icon content within the center 80% of the canvas. For a 512x512 icon, that means a 410x410 safe area with 51px padding on each side.

Create maskable versions by adding extra padding/background around your logomark:

```json
// manifest.json icon entry
{
  "src": "/icons/icon-512x512-maskable.png",
  "sizes": "512x512",
  "type": "image/png",
  "purpose": "maskable"
}
```

You need both a regular and maskable version at 192 and 512:

```json
{
  "icons": [
    { "src": "/icons/icon-192x192.png", "sizes": "192x192", "type": "image/png", "purpose": "any" },
    { "src": "/icons/icon-192x192-maskable.png", "sizes": "192x192", "type": "image/png", "purpose": "maskable" },
    { "src": "/icons/icon-512x512.png", "sizes": "512x512", "type": "image/png", "purpose": "any" },
    { "src": "/icons/icon-512x512-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

## Implementation in Next.js

### File Placement

Next.js App Router supports file-based favicon configuration. Place these in your `app/` directory:

```
app/
├── favicon.ico          (auto-served at /favicon.ico)
├── icon.svg             (modern browsers)
├── icon.png             (fallback, 32x32)
├── apple-icon.png       (180x180, auto apple-touch-icon)
└── manifest.ts          (or manifest.json for PWA)
```

### Metadata API

```tsx
// app/layout.tsx
export const metadata = {
  icons: {
    icon: [
      { url: '/favicon.ico', sizes: '32x32' },
      { url: '/icon.svg', type: 'image/svg+xml' },
    ],
    apple: '/apple-icon.png',
  },
  manifest: '/manifest.json',
}
```

### manifest.json

```json
{
  "name": "Your App Name",
  "short_name": "AppName",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#your-primary-color",
  "icons": [
    { "src": "/icons/icon-192x192.png", "sizes": "192x192", "type": "image/png", "purpose": "any" },
    { "src": "/icons/icon-192x192-maskable.png", "sizes": "192x192", "type": "image/png", "purpose": "maskable" },
    { "src": "/icons/icon-512x512.png", "sizes": "512x512", "type": "image/png", "purpose": "any" },
    { "src": "/icons/icon-512x512-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

## Design Guidelines

- **Use your logomark, not the full lockup** — favicons are tiny, text is illegible at 16x16
- **Simplify for small sizes** — if your logomark has fine details, create a simplified version for 16x16 and 32x32
- **Test at actual size** — zoom into your browser tab. Is it recognizable?
- **Strong silhouette** — the icon should be identifiable by shape alone, even in grayscale
- **Contrast against both light and dark browser chrome** — test on Chrome (light), Chrome (dark), Safari, Firefox
- **No transparency for apple-touch-icon** — iOS renders transparent areas as black. Always use a solid background.

## Generation Workflow

1. Design your logomark as SVG (Claude can do this)
2. Create the dark-mode-aware `favicon.svg`
3. Export PNG at 180x180 for apple-touch-icon (with solid background, no transparency)
4. Export PNG at 192x192 and 512x512 for PWA (both regular and maskable)
5. Use **RealFaviconGenerator** to generate the `.ico` file from your 32x32 PNG
6. Validate with RealFaviconGenerator's checker tool

## Folder Structure

```
public/
├── favicon.ico
├── icons/
│   ├── icon-192x192.png
│   ├── icon-192x192-maskable.png
│   ├── icon-512x512.png
│   └── icon-512x512-maskable.png
app/
├── favicon.ico (symlink or copy)
├── icon.svg
├── apple-icon.png
└── manifest.json
```

## Deliverables

- [ ] Logomark simplified for small sizes (if needed)
- [ ] favicon.svg with dark mode support
- [ ] favicon.ico (32x32, or multi-size)
- [ ] apple-touch-icon (180x180, solid background)
- [ ] PWA icons at 192x192 and 512x512 (regular + maskable)
- [ ] manifest.json configured with all icons
- [ ] Next.js metadata configured
- [ ] Tested in Chrome, Safari, Firefox (light + dark mode)
- [ ] Tested on iOS home screen (add to home screen)
- [ ] Tested on Android home screen (install PWA)
- [ ] Validated with RealFaviconGenerator checker

## Common Mistakes

- Transparent background on apple-touch-icon (renders as black on iOS)
- Not testing at actual browser tab size (looks great at 512, unrecognizable at 16)
- Missing maskable icons (Android crops your icon ugly without them)
- Using the full lockup (text + icon) as a favicon (illegible)
- Forgetting favicon.ico (some legacy clients and tools still look for it)
- Not placing favicon.ico in the root (some crawlers only check the root path)
