# Marketing Site Layout

---

## What It Is

The structure and navigation of your public-facing website — landing page, pricing, blog, about, contact, legal pages. This is always a website regardless of what platform your app runs on.

## Layout

```
┌─────────────────────────────────────────────┐
│  Navbar (logo, links, CTA)                  │
├─────────────────────────────────────────────┤
│                                             │
│              Page Content                   │
│           (full-width, sections)            │
│                                             │
├─────────────────────────────────────────────┤
│  Footer (links, legal, social)              │
└─────────────────────────────────────────────┘
```

## Route Structure

```
your-domain.com
  ├── /               Landing page
  ├── /pricing         Pricing
  ├── /features        Features
  ├── /blog            Blog
  ├── /about           About
  ├── /contact         Contact
  ├── /branding        Brand assets
  ├── /privacy         Privacy policy
  ├── /terms           Terms of service
  │
  ├── /sign-in         Auth pages (minimal layout — no nav, no footer)
  └── /sign-up
```

**Design decisions:**
- Marketing site uses its own layout — top navbar + footer
- Auth pages (sign-in, sign-up, reset password) use a minimal layout — centered card, no navigation, no footer
- The marketing site layout and the app layout are completely separate — different navigation, different structure, different route groups

## Navbar

- Fixed to the top of the viewport (sticky on scroll)
- Logo on the left — links to the landing page
- Navigation links right-aligned: Features, Pricing, Blog, About
- Primary CTA on the far right: "Get Started" or "Sign In" (depending on auth state)
- If the user is signed in, replace the CTA with "Go to App" or a user avatar menu
- Background adds a subtle backdrop blur on scroll to indicate it's floating over content
- Max content width matches the page content width (consistent gutters)

## Footer

- Full-width with a slightly different background (subtle contrast from page content)
- Columns: Product (features, pricing, changelog), Company (about, blog, contact), Legal (privacy, terms, cookies), Social (Twitter, GitHub, LinkedIn)
- Copyright line at the bottom
- The footer is the same on every marketing page

## Page Content

- Full-width sections with max content width constraint (1280px)
- Sections alternate between different background treatments for visual rhythm
- Consistent vertical spacing between sections
- Mobile: sections stack vertically, padding reduces

## Auth Layout

- Minimal — no navbar, no footer
- Centered card on the page with the logo above it
- Used for: sign-in, sign-up, password reset, email verification, 2FA challenge
- After successful sign-in, redirect to the app (or the page they were trying to access)
- If already signed in, redirect to the app — don't show the sign-in form to an authenticated user

## Responsive Behaviour

| Breakpoint | Behaviour |
|---|---|
| Desktop (lg+) | Full navbar with all links visible |
| Tablet (md) | Navbar condenses — fewer links visible, secondary links move to a "More" dropdown |
| Mobile (< md) | Hamburger menu replaces navbar links. Menu opens as a full-screen overlay. Logo + hamburger visible. |

- Footer columns stack on mobile (2 columns on tablet, single column on small mobile)
- Page sections reflow from multi-column to single column
- Hero sections reduce font size and padding
- All styles are mobile-first — write the mobile layout as default, add complexity with breakpoints going up
- Hero images and animations should be lazy loaded or reduced on mobile — slow landing page kills conversion

## What to Test

- Navbar CTA shows "Sign In" when unauthenticated and "Go to App" when authenticated
- Hamburger menu opens and closes correctly on mobile
- Footer is identical across all marketing pages
- Auth pages (sign-in, sign-up) use minimal layout (no navbar, no footer)
- Authenticated user accessing sign-in page is redirected to the app
- All marketing pages render correctly on mobile, tablet, and desktop breakpoints

## Edge Cases

- **Auth page accessed while already signed in** — redirect to the app. Don't show the sign-in form to an authenticated user.
- **Marketing page loads slowly on mobile** — hero images and animations should be lazy loaded or reduced on mobile. Performance budgets apply to marketing pages.
- **Deep link to the app while signed out** — middleware redirects to sign-in, preserving the intended URL. After sign-in, redirect back to the original deep link.
- **Navbar CTA flickers on load** — the auth state check (signed in vs not) may cause the CTA to flash between "Sign In" and "Go to App." Render the navbar server-side with the auth state already resolved to avoid this.
- **Blog post shared on social media** — OG tags must be set per page, not just globally. Each blog post needs its own title, description, and image for social sharing. Covered in Phase 6 (Marketing Site & SEO).

## Implementation Checklist

- [ ] Marketing layout with top navbar + footer (separate from app layout)
- [ ] Auth layout with centered card, no navigation
- [ ] Sticky navbar with logo, links, CTA
- [ ] Navbar CTA changes based on auth state (Sign In vs Go to App)
- [ ] Navbar rendered server-side to avoid auth state flicker
- [ ] Footer with consistent columns across all marketing pages
- [ ] Full-width sections with max content width constraint (1280px)
- [ ] Hamburger menu on mobile with full-screen overlay
- [ ] Footer columns stack on mobile
- [ ] Hero images lazy loaded on mobile
- [ ] Tested on real mobile devices

## Common Mistakes

- Sharing a layout between marketing site and app (they have different navigation, different structure — keep them separate)
- Auth pages showing marketing navigation (confusing — use the minimal auth layout)
- Navbar CTA not reflecting auth state (signed-in user sees "Sign In" instead of "Go to App")
- Different max content widths on different pages (inconsistent gutters, feels unpolished)
- No mobile hamburger menu (navbar links overflow or wrap on small screens)
- Heavy hero animations that destroy mobile performance (test on real devices, not just desktop Chrome)
- OG tags set globally instead of per page (every shared link shows the same generic preview)
