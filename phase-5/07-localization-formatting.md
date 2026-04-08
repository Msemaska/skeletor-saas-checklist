# Localization & Formatting

---

## What It Is

How dates, times, numbers, and currency are displayed to the user based on their locale. This is display formatting only — not translation or multi-language support (that's Phase 18 post-launch). Every piece of data that represents a date, time, number, or currency must be formatted consistently across the entire product.

## Where Locale Comes From

The user's timezone and locale are determined in this order:

1. **Timezone:** stored on the user's profile as a setting. Default to the browser's timezone on sign-up. The user can change it in their profile settings.
2. **Locale for number/currency formatting:** derived from the browser's `navigator.language`. No user setting needed — the browser already knows the user's preferred locale.

All data is stored in UTC in the database. All formatting happens in the frontend at render time. Never store formatted strings in the database.

## Date Formatting

**Display format:** DD MMM YYYY (e.g., "09 Mar 2026"). Use the short month name, not a number — "09/03/2026" is ambiguous (March 9th in the UK, September 3rd in the US). The short month name eliminates ambiguity across all locales.

**Rules:**
- Always display dates in the user's timezone, converted from UTC at render time
- Use date-fns for all date formatting and manipulation
- Use date-fns-tz for timezone conversions
- Never format dates server-side — always send UTC timestamps to the frontend and format there
- Date-only fields (due dates, birth dates) are stored as `date` type in the database and displayed without timezone conversion (a due date of March 9th is March 9th everywhere)

## Time Formatting

**Display format:** 24-hour (e.g., "14:30"). No AM/PM.

**Rules:**
- Times are always displayed in the user's timezone
- Include seconds only when precision matters (logs, audit trails). For user-facing features, hours and minutes are sufficient.
- For events happening today, show just the time: "14:30"
- For events on a different day, show date and time: "09 Mar 2026, 14:30"

## Relative Time

For recent events, use relative time instead of absolute timestamps. It's more natural and immediately understandable.

**Rules:**
- Use relative time for events within the last 7 days: "just now", "3 minutes ago", "2 hours ago", "yesterday", "3 days ago"
- Beyond 7 days, switch to absolute date: "09 Mar 2026"
- Show the absolute timestamp on hover (tooltip) for all relative times — the user can always see the exact time if they need it
- Use date-fns `formatDistanceToNow` for relative formatting
- Update relative times live if the component stays mounted (a message showing "just now" should update to "1 minute ago" without a page refresh). Update every 60 seconds.

## Currency Formatting

**Rules:**
- Use `Intl.NumberFormat` with the user's browser locale for all currency display
- Always include the currency code or symbol — never display a bare number for money
- Store all currency values as integer cents in the database (covered in phase-3/02-validation.md). Convert to display format at render time.
- Use a single currency per product (defined as a system or workspace setting). Multi-currency is a post-launch concern.

**Display examples:**
- US locale: $19.99
- UK locale: £19.99
- EU locale: €19,99
- The same integer (1999 cents) displays differently per locale. `Intl.NumberFormat` handles this automatically.

## Number Formatting

**Rules:**
- Use `Intl.NumberFormat` for all number display — it handles thousands separators and decimal points per locale (1,234.56 in US, 1.234,56 in Germany)
- For counts and metrics (project count, message count), format with locale-appropriate thousands separator
- For percentages, use `Intl.NumberFormat` with `style: 'percent'`
- For large numbers, abbreviate: 1.2K, 45.3K, 1.2M. Use `Intl.NumberFormat` with `notation: 'compact'`

## Where Formatting Happens

All formatting is a frontend concern. The backend sends raw data, the frontend formats for display.

| Data | Backend Sends | Frontend Displays |
|---|---|---|
| Timestamps | UTC ISO 8601 string | Formatted in user's timezone |
| Dates (date-only) | ISO date string (YYYY-MM-DD) | DD MMM YYYY (no timezone conversion) |
| Money | Integer cents | Locale-formatted with currency symbol |
| Numbers | Raw number | Locale-formatted with separators |
| Relative time | UTC timestamp | "3 hours ago" with absolute tooltip |

**Never:**
- Format dates or numbers server-side (the server doesn't know the user's locale or timezone)
- Store pre-formatted strings in the database ("$19.99" as text — store 1999 as integer)
- Use `toLocaleDateString()` without explicit format options (output varies unpredictably across browsers)

## What to Test

- Dates display as DD MMM YYYY in user's timezone
- Times display in 24-hour format
- Relative time shows "3 hours ago" for recent events and absolute date beyond 7 days
- Absolute timestamp appears on hover for all relative times
- Currency displays with correct symbol and locale-appropriate separators
- Date-only fields do not shift timezone (due date stays the same across timezones)
- Changing timezone in settings updates displayed dates immediately

## Edge Cases

- **User changes timezone in settings** — all displayed dates and times should update immediately without a page refresh. Store timezone in a context/store that triggers re-renders.
- **Event spans midnight in the user's timezone** — "yesterday at 23:45" could be "today at 00:45" in a different timezone. Always convert to the user's timezone before determining "today" vs "yesterday."
- **Relative time on old content** — a notification from 6 months ago should show the absolute date, not "183 days ago." The 7-day threshold handles this.
- **Currency display in different locales** — workspace is billed in USD but a user's browser locale is German. Pass the currency code ("USD") to `Intl.NumberFormat` along with the browser locale — it handles the symbol and formatting correctly per locale ($19.99 in US, 19,99 $ in French). Don't manually map currencies to symbols.
- **Server-rendered pages** — Server Components don't have access to the browser locale or timezone. For server-rendered content, either send timestamps as raw UTC and format client-side (via a client component), or use the user's timezone from their profile setting if available.
- **Timezone abbreviations** — "EST" vs "EDT" vs "America/New_York." Always store and reference IANA timezone identifiers (America/New_York), never abbreviations (EST is ambiguous — multiple timezones share it).
- **Date-only fields with timezone confusion** — a due date of "March 9th" should not shift to "March 8th" when viewed from a different timezone. Date-only fields use the `date` type and are displayed without timezone conversion.

## Implementation Checklist

- [ ] User timezone stored in profile settings, defaulting to browser timezone on sign-up
- [ ] All timestamps stored as UTC in the database
- [ ] All date/time formatting happens in the frontend, never server-side
- [ ] date-fns used for date formatting and manipulation
- [ ] date-fns-tz used for timezone conversions
- [ ] Dates displayed as DD MMM YYYY (e.g., "09 Mar 2026")
- [ ] Times displayed in 24-hour format (e.g., "14:30")
- [ ] Relative time for events within 7 days, absolute date beyond
- [ ] Absolute timestamp shown on hover for all relative times
- [ ] Relative times update live every 60 seconds on mounted components
- [ ] Currency formatted with Intl.NumberFormat using browser locale
- [ ] Currency stored as integer cents, formatted at display time
- [ ] Numbers formatted with locale-appropriate separators via Intl.NumberFormat
- [ ] Large numbers abbreviated with compact notation (1.2K, 45.3K)
- [ ] IANA timezone identifiers used (never abbreviations like EST)
- [ ] Date-only fields displayed without timezone conversion

## Common Mistakes

- Formatting dates server-side (server doesn't know the user's timezone — dates will be wrong)
- Storing formatted strings in the database ("$19.99" instead of 1999 cents — can't reformat for different locales)
- Using MM/DD/YYYY or DD/MM/YYYY (ambiguous across locales — use DD MMM YYYY with month name)
- Using 12-hour time with AM/PM (inconsistent internationally — 24-hour is universal)
- Not showing absolute time on hover for relative timestamps (user needs to know the exact time sometimes)
- Relative time that says "183 days ago" instead of switching to absolute date (meaningless past a few days)
- Timezone abbreviations instead of IANA identifiers (EST is ambiguous, America/New_York is not)
- Applying timezone conversion to date-only fields (due date shifts to a different day in a different timezone)
- Using `toLocaleDateString()` without options (output varies across browsers and environments)
- Not updating relative times on mounted components ("just now" stays forever until page refresh)
