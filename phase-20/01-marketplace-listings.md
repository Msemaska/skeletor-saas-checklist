# Marketplace & App Store Listings

---

## What It Is

Listings for your product on third-party marketplaces and app stores — Chrome Web Store, Slack App Directory, and Shopify App Store. Each listing acts as a distribution channel that puts your product in front of users who are already searching for solutions in that ecosystem. Includes browser extension packaging for cross-browser builds.

## Design Decisions

**Browser extension framework:** Plasmo. It handles cross-browser extension builds from a single codebase — Chrome, Firefox, Edge, and Safari. Write the extension once, Plasmo compiles it to the correct manifest format for each browser. Plasmo supports React and TypeScript natively, so the extension uses the same tech stack as the main product.

**Chrome Web Store listing:** The primary browser extension distribution channel. The listing includes: extension name, short description (132 characters max), detailed description (up to 16,384 characters), 5 screenshots at 1280x800, a 440x280 promotional tile image, a 128x128 icon, and a link to the privacy policy. Pull the description from the copy bank (phase 2). Pull screenshots from the visual asset library (phase 1). Do not create custom assets per store — reuse what already exists and crop/resize to meet the store's requirements.

**Slack App Directory listing:** Submit the Slack app (built in a previous phase if applicable) to the Slack App Directory. The listing requires: app name, short description, detailed description, 3 screenshots, app icon (512x512), redirect URLs, required scopes justification, and a privacy policy URL. Slack reviews the app for compliance with their guidelines before listing.

**Shopify App Store listing:** Only if the product has a Shopify integration. Register as a Shopify Partner, create the app listing with: app name, description, pricing, screenshots (1600x900), app icon, and a demo store URL. Shopify reviews the app for quality, performance, and compliance.

**Asset reuse:** All screenshots, descriptions, and icons come from the existing visual asset library and copy bank. Resize and crop to meet each store's dimension requirements. Do not commission separate creative for each marketplace — it's wasteful and creates inconsistency. Maintain a mapping document that lists each store's requirements and which existing assets map to them.

**Privacy policy:** Every marketplace requires a privacy policy URL. Use the same privacy policy page hosted on your main site (`/privacy`). The policy must cover what data the extension or app collects, how it's stored, and how users can delete their data.

**Review and approval timelines:**
- Chrome Web Store: 3 days for initial review. Updates review faster (usually same day).
- Slack App Directory: 2 weeks. Slack manually reviews scopes, permissions, and user experience.
- Shopify App Store: 4 weeks. Shopify reviews code quality, performance, and merchant experience.

Submit listings early — don't wait until launch day. Submit 4 weeks before planned launch to account for the slowest review cycle (Shopify).

**Listing updates:** When the product changes (new features, UI changes, pricing changes), update all marketplace listings. Update screenshots if the UI changed. Update the description if features were added or removed. Update the feature list. Set a calendar reminder to review listings every quarter even if no major changes occurred.

## What to Test

- Plasmo builds produce valid extension packages for Chrome, Firefox, and Edge
- Chrome extension installs and functions correctly from the packaged build
- All screenshots meet each store's dimension requirements
- Description text fits within each store's character limits
- Privacy policy URL is accessible and covers extension/app data usage
- Listing assets (icon, screenshots, promotional images) render correctly at required sizes
- Extension manifest includes correct permissions (no over-requesting)

## Edge Cases

- **Chrome Web Store rejects the listing** — common reasons: overly broad permissions, misleading description, or missing privacy policy. Fix the flagged issue and resubmit. Don't request permissions the extension doesn't actively use.
- **Slack rejects the app** — usually scope-related. Slack wants the minimum scopes necessary. Justify every scope in the submission form. Remove any scope that isn't strictly required for the core functionality.
- **Shopify rejects the app** — often performance or UX issues. Shopify expects the app to load within 3 seconds and follow their design guidelines. Fix flagged issues and resubmit.
- **Store listing shows outdated screenshots** — the product UI changed but the listing wasn't updated. The quarterly review reminder catches this. If a major UI overhaul ships, update listings immediately.
- **Extension update breaks for existing users** — Plasmo handles extension versioning. Chrome auto-updates extensions. Test every update against the previous version's stored data to ensure backward compatibility.
- **Multiple browser stores with different manifest versions** — Chrome uses Manifest V3, Firefox supports both V2 and V3. Plasmo handles this — it generates the correct manifest version per target browser. Don't manage manifests manually.

## Implementation Checklist

- [ ] Plasmo installed and configured in the project
- [ ] Extension builds successfully for Chrome, Firefox, and Edge
- [ ] Chrome Web Store developer account created ($5 one-time fee)
- [ ] Chrome listing assets prepared: 5 screenshots (1280x800), promotional tile (440x280), icon (128x128)
- [ ] Chrome listing description written (short: 132 chars, detailed: from copy bank)
- [ ] Chrome listing submitted with privacy policy link
- [ ] Slack App Directory submission prepared: screenshots, icon (512x512), description, scope justifications
- [ ] Slack app submitted for review
- [ ] Shopify Partners account created (if applicable)
- [ ] Shopify listing assets prepared: screenshots (1600x900), icon, demo store URL
- [ ] Shopify app submitted for review (if applicable)
- [ ] Asset mapping document created: which existing assets map to each store's requirements
- [ ] Quarterly calendar reminder set to review and update all listings
- [ ] All listings link to the privacy policy at `/privacy`
- [ ] Extension permissions audited — no over-requested scopes

## Common Mistakes

- Submitting listings on launch day and discovering the review takes 4 weeks (submit 4 weeks early)
- Creating custom screenshots and copy for each store instead of reusing from the asset library (inconsistent branding, wasted effort)
- Requesting every possible permission "just in case" (stores reject for overly broad permissions)
- Never updating listings after initial submission (screenshots show a UI from 6 months ago)
- Missing privacy policy (every store requires it — instant rejection)
- Managing separate extension codebases per browser instead of using Plasmo (3x maintenance burden)
- Not testing the packaged extension before submission (works in development, fails as a packaged build)
- Ignoring store-specific guidelines and design requirements (Shopify has strict UX expectations — read their docs)
- Waiting for all stores to approve before launching anywhere (launch on Chrome first, Slack and Shopify follow when approved)
