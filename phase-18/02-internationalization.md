# Internationalization

---

## What It Is

Infrastructure for supporting multiple languages and right-to-left (RTL) layouts in the app. Starts with English only. Languages are added based on user demand, not preemptively.

## RTL Support

### Design Decisions

**Design decisions:**
- Enable Tailwind's `rtl:` variant in tailwind.config.ts by adding the `rtl` variant
- Set the `dir` attribute on the `<html>` element based on the active locale (dir="ltr" for English, dir="rtl" for Arabic/Hebrew)
- Use logical CSS properties everywhere — `ps-4` instead of `pl-4`, `me-2` instead of `mr-2`, `start` instead of `left`. Tailwind's logical properties handle LTR/RTL automatically.
- Test key pages in RTL mode: dashboard, settings, billing, onboarding, landing page
- Icons that indicate direction (arrows, chevrons) must flip in RTL mode — use `rtl:rotate-180` on directional icons
- Non-directional icons (close, search, settings) do not flip

## Translation Pipeline

### Design Decisions

**Design decisions:**
- Use next-intl for locale detection, string extraction, locale switching, and message formatting
- next-intl handles: pluralization, number formatting, date formatting, interpolation
- Default locale: en (English)
- Locale detection order: URL path prefix (/en/dashboard, /ar/dashboard) > user preference stored in database > browser Accept-Language header
- URL structure: /[locale]/[...path] — every route is prefixed with the locale code
- Locale switcher in the app footer and in settings — dropdown with available languages
- When a user selects a locale, store their preference in the user profile (preferred_locale column in the users table)

## Translation Files

### Design Decisions

**Design decisions:**
- Store translations as JSON files in the repository at /messages/[locale].json (next-intl's default convention)
- One JSON file per locale — not split by page or feature (keeps it simple for a small app)
- File structure: flat keys with dot-separated namespaces. Each key is a namespace-prefixed string (e.g., dashboard.title) mapped to the translated text. Interpolation uses curly braces around the variable name within the value string (e.g., the value for a welcome message includes the user's name as a placeholder).

- English is the source of truth — all other locale files must have the same keys as en.json
- Missing translations fall back to English — never show a raw translation key to the user

## Adding a New Language

### Design Decisions

**Design decisions:**
- Only add a language when 20+ users have requested it (tracked via a "request a language" form in settings)
- Translation process: extract all strings from en.json > translate using Claude > review translations with a native speaker > deploy
- Professional translator for high-stakes strings (legal pages, billing, error messages)
- Claude for lower-stakes strings (UI labels, tooltips, descriptions) — reviewed by a native speaker before merging
- New locale file is added to /messages/[locale].json and the locale is added to the supported locales list in next-intl config
- Test the new locale end-to-end: every page renders without missing translations, dates and numbers format correctly, RTL layout works (if applicable)

## What to Test

- Setting dir="rtl" on the html element mirrors the layout correctly on all key pages
- Directional icons flip in RTL mode
- Non-directional icons do not flip in RTL mode
- Logical CSS properties (ps, pe, ms, me, start, end) produce correct spacing in both LTR and RTL
- next-intl loads the correct locale file based on URL path prefix
- Locale switcher changes the language and URL prefix
- User locale preference is stored in the database and used on subsequent visits
- Interpolated strings render correctly (e.g., "Welcome, {name}" shows the actual name)
- Pluralization works correctly (e.g., "1 item" vs. "3 items")
- Dates and numbers format according to the active locale
- Missing translations fall back to English
- No raw translation keys are visible in the UI

## Edge Cases

- **User switches locale mid-session** — the page reloads with the new locale prefix. All client-side state (forms, modals) resets. Warn the user if they have unsaved changes.
- **Translation file has a missing key** — next-intl falls back to the default locale (English). Log the missing key in development mode so it gets caught before production.
- **RTL locale with LTR content (code snippets, URLs, email addresses)** — wrap LTR content in a `<bdi>` element. URLs and email addresses must remain LTR even in an RTL page.
- **Locale in URL doesn't match user's stored preference** — the URL takes priority. If a user with preferred_locale=ar visits /en/dashboard, they see English. The URL is the source of truth for the current session.
- **SEO with multiple locales** — add hreflang tags to the head of every page, pointing to all available locale versions. Use next-intl's built-in hreflang support.
- **Third-party content (Stripe billing portal, Senja widgets) doesn't support the user's locale** — fall back to English for third-party content. Don't attempt to translate content you don't control.

## Implementation Checklist

- [ ] next-intl installed and configured
- [ ] Default locale set to en
- [ ] /messages/en.json created with all UI strings extracted
- [ ] URL structure set to /[locale]/[...path]
- [ ] Locale detection: URL prefix > user preference > Accept-Language header
- [ ] Locale switcher added to app footer and settings
- [ ] User locale preference stored in the users table (preferred_locale column)
- [ ] Tailwind rtl: variant enabled in tailwind.config.ts
- [ ] Logical CSS properties used for all spacing and alignment (ps, pe, ms, me, start, end)
- [ ] dir attribute on html element set dynamically based on active locale
- [ ] Directional icons flip in RTL mode (rtl:rotate-180)
- [ ] Key pages tested in RTL mode: dashboard, settings, billing, onboarding, landing page
- [ ] Missing translation fallback to English confirmed working
- [ ] hreflang tags added to page heads
- [ ] "Request a language" form in settings in place
- [ ] Translation process documented: extract > translate > review > deploy

## Common Mistakes

- Pre-translating into 10 languages before anyone has asked (wasted effort — start with English, add on demand)
- Using physical CSS properties (pl, pr, ml, mr) instead of logical ones (ps, pe, ms, me) — breaks RTL layouts
- Hardcoding strings in components instead of using translation keys (makes future translation impossible without refactoring)
- Splitting translation files by page or feature (over-engineering for a small app — one file per locale is enough)
- Not testing RTL with actual RTL content (testing with English in RTL mode doesn't reveal real layout issues — test with Arabic or Hebrew text)
- Forgetting to flip directional icons in RTL mode (back arrow points the wrong way)
- No fallback for missing translations (user sees "dashboard.welcome" instead of "Welcome, John")
- Translating legal pages with Claude without professional review (legal text requires accuracy — use a professional translator)
