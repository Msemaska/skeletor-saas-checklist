# Changelog / Release Notes

---

## What It Is

A running log of what's new in the product — new features, improvements, and fixes.

## Where It Lives

**Design decisions:**
- Changelog is a page on the marketing site at /changelog
- Each entry is an MDX file in content/changelog/ named by date (content/changelog/2026-04-08.mdx)
- Changelog index page shows entries reverse-chronologically (newest first)
- Entries are also delivered via the notification system (phase-3/07-notification-system.md) as a "What's new" in-app notification

## Entry Structure

Every changelog entry follows the same format:

| Element | Content |
|---|---|
| Date | Publication date |
| Title | The headline feature or theme of this release |
| Category tags | New Feature, Improvement, Fix |
| Summary | 1 sentence: what changed and why it matters to the user |
| Details | 3-5 bullet points: specific changes, each starting with a verb (Added, Improved, Fixed, Removed) |
| Visual | Screenshot or GIF of the most notable change |
| CTA | "Try it now" linking to the relevant feature page |

## Writing Rules

**Design decisions:**
- Write from the user's perspective, not the developer's. "You can now export reports as PDF" not "Implemented PDF export functionality in the reporting module."
- Start every bullet with an action verb: Added, Improved, Fixed, Removed, Updated
- Lead with the biggest change. If the entry has a new feature and 3 bug fixes, the new feature is the title and comes first.
- Include a visual for the headline change — a screenshot or 4-second GIF showing the feature in action
- Keep each entry under 200 words.
- Group related changes under a single entry. Don't publish 5 separate entries for 5 small fixes — batch them into one "Improvements & Fixes" entry.

## Publishing Cadence

**Design decisions:**
- Publish a changelog entry on every deployment that includes user-facing changes
- Internal refactors, dependency updates, and infrastructure changes don't get changelog entries — only changes the user can see or feel
- Batch small fixes into a single weekly entry if there's no major feature that week
- Never go more than 2 weeks without a changelog entry

## Distribution

**Design decisions:**
- In-app: "What's new" notification via the notification system when a changelog entry is published
- Social media: Claude repurposes the changelog entry into a tweet (before/after format or feature GIF). Schedule via Postiz.
- Newsletter: include the top 1-2 changelog highlights in the weekly newsletter
- Changelog page is linked from the app's help menu and from the marketing site footer

## What to Test

- Changelog page displays entries reverse-chronologically
- Each entry has: date, title, category tags, summary, details, visual
- In-app "What's new" notification fires on new changelog entry
- Changelog page is accessible from the app help menu and marketing footer
- Entry links to the correct feature page

## Edge Cases

- **Major feature launch** — a major feature gets a blog post AND a changelog entry. The blog post is the deep dive. The changelog entry is the summary with a link to the blog post.
- **Breaking change** — if a change affects existing user workflows, the changelog entry must explain: what changed, why, and what the user needs to do differently. Send this as an email notification, not just in-app.
- **Bug fix for a bug that was never publicly acknowledged** — still include it in the changelog. "Fixed: occasional timeout when exporting large reports." Users who experienced the bug will appreciate seeing it addressed.
- **Revert** — if you ship a feature and then revert it, add a changelog entry: "Temporarily reverted [feature] while we address [issue]. We'll re-release once resolved."

## Implementation Checklist

- [ ] /changelog page built with reverse-chronological entries
- [ ] Changelog MDX template created (date, title, tags, summary, details, visual, CTA)
- [ ] content/changelog/ directory created
- [ ] Changelog linked from app help menu and marketing footer
- [ ] In-app "What's new" notification triggers on new changelog entry
- [ ] Distribution workflow: tweet + newsletter highlight on every entry
- [ ] Publishing cadence: entry on every user-facing deployment, never more than 2 weeks between entries

## Common Mistakes

- Developer-speak instead of user-speak ("Refactored the auth middleware" — the user doesn't care)
- No visuals
- No changelog page at all
- Every tiny fix gets its own entry (batch small fixes weekly)
- No distribution
- Changelog goes silent for months
- Breaking changes without explanation (users' workflows break and they don't know why)
