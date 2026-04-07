# Visual Asset Library

---

## What It Is

A centralised collection of every visual asset your product needs for marketing — screenshots, feature GIFs, mockups, social media graphics, and demo video. Like the copy bank, you create these once and reuse them across every platform. When Product Hunt needs a gallery, when a blog post needs a screenshot, when a tweet needs a visual — you pull from the library.

## Where to Store It

`public/marketing/` in the repository for web-accessible assets, and a `docs/visual-assets/` directory for source files and the asset catalogue. The catalogue is a markdown file listing every asset with its filename, dimensions, and where it's used.

## What the Library Contains

### 1. Product Screenshots

Annotated screenshots of the key screens in your product. These are the most reused visual assets — they appear everywhere.

**What to capture:**
- Dashboard (the first screen users see after sign-in)
- The core feature in use (the "money shot" — the product doing the thing it's built for)
- Settings/workspace page (shows customisation and professionalism)
- Mobile view of the dashboard (proves responsive design)

**Design decisions:**
- Capture at 2x resolution (Retina) for crisp display on all devices
- Clean up the screenshot: use realistic but polished data, remove personal information, use a professional-looking workspace name
- Add subtle browser chrome (window frame, URL bar) to give context — tools like BrandBird or CleanShot X add this automatically
- Light mode and dark mode variants of the top 3 screenshots
- Export as PNG at 1200px width minimum (works for blog posts, directories, and social media)

### 2. Feature GIFs

Short animated captures showing a feature in action. More engaging than static screenshots, used in tweets, Product Hunt galleries, and landing pages.

**What to capture:**
- The core workflow (4 second loop showing the main user action)
- A "wow" moment (the most impressive interaction — drag-and-drop, instant preview, AI generation)
- Onboarding flow (the first-run experience)

**Design decisions:**
- Duration: 4 seconds per GIF. Longer than that and the file size becomes unusable.
- Dimensions: 800px wide (balances quality and file size)
- Use Screen Studio for recording — it adds smooth cursor animations and zoom effects
- Export as GIF for universal compatibility, and as MP4 for web pages (smaller file size, better quality)
- No audio — GIFs are silent, and social media auto-plays without sound

### 3. Product Mockups

Screenshots placed into device frames (laptop, phone, tablet) for a polished, contextual look.

**What to create:**
- Laptop mockup with the dashboard (for landing page hero, pitch decks)
- Phone mockup with the mobile view (proves mobile support)
- Multi-device composition (laptop + phone side by side — for the "works everywhere" message)

**Design decisions:**
- Use clean, modern device frames — no dated bezels or unrealistic angles
- Background: your brand colour or a subtle gradient. Not a stock photo of a desk.
- Generate with Claude + SVG/React or use BrandBird
- Export at 1200x630 for social sharing and 2400px wide for landing pages

### 4. Social Media Graphics

Ready-made visuals sized for each platform's requirements.

**What to create:**
- **Announcement graphic** — "We just launched!" with product name, tagline, and screenshot. Used for launch day posts across all platforms.
- **Feature highlight cards** — one per major feature. Product name, feature name, one-line benefit, small screenshot or icon. Used for ongoing social content.
- **Quote/testimonial cards** — customer quote with attribution. Used when you have social proof.
- **Comparison graphics** — "Us vs Them" visual comparison. Used for competitive positioning posts.

**Sizes per platform:**

| Platform | Post Size | Notes |
|---|---|---|
| Twitter/X | 1200x675 | Single image or up to 4 in a grid |
| LinkedIn | 1200x627 | Single image posts |
| Instagram | 1080x1080 | Square for feed |
| Product Hunt | 1270x760 | Gallery images (up to 8) |
| OG / social share | 1200x630 | Default when link is shared |

**Design decisions:**
- Use your brand colours, typography, and logo consistently across all graphics
- Generate with @vercel/og for template-based graphics, or Claude for SVG/React compositions
- Every graphic includes your product name or logo — it should be identifiable without context

### 5. Demo Video

A short video showing the product in action. The single most impactful marketing asset.

**Design decisions:**
- Duration: 75 seconds. Show the core value, not every feature.
- Structure: problem (5 seconds) → solution intro (5 seconds) → product walkthrough (55 seconds) → CTA (10 seconds)
- Record with Screen Studio — it adds professional cursor animations, zoom, and transitions
- No voiceover — use text overlays and captions.
- Background music: subtle, upbeat, non-distracting. Royalty-free.
- Host on YouTube (free, embeddable, SEO value)
- Embed on the landing page, features page, and Product Hunt gallery

### 6. OG Image Template

The preview image that appears when someone shares a link to your site.

**Design decisions:**
- Already covered in phase-5/02-technical-seo.md — the OG image template is built with @vercel/og
- The visual asset library should include the default OG image as a reference asset
- Blog posts get custom OG images with the post title — these are generated dynamically

## Asset Naming Convention

Every file follows a consistent naming pattern:

`{type}-{subject}-{variant}.{format}`

Examples:
- `screenshot-dashboard-light.png`
- `screenshot-dashboard-dark.png`
- `gif-drag-drop-workflow.gif`
- `mockup-laptop-dashboard.png`
- `social-launch-announcement-twitter.png`
- `social-feature-search-linkedin.png`
- `video-demo-product.mp4`

## Maintaining the Library

**Design decisions:**
- Update screenshots and GIFs whenever the UI changes significantly (redesign, major feature launch)
- Don't update for every minor tweak — only when the current assets no longer represent the product accurately
- Keep a changelog in the asset catalogue noting when assets were last updated
- Social media graphics are created as needed for campaigns — the library holds the templates and the evergreen assets

## What to Test

- All screenshots are current (match the live product UI)
- Screenshots use realistic, polished data (no "test123" or Lorem ipsum)
- GIFs loop smoothly and are under 4 seconds
- Mockups use clean device frames without dated bezels
- Social graphics are correctly sized for each platform
- Demo video plays on YouTube and embeds correctly on the landing page
- OG image displays correctly when sharing links on Twitter, LinkedIn, and Discord
- All assets are 2x resolution (Retina-ready)

## Edge Cases

- **Product UI changes after assets are created** — if the dashboard gets a redesign, all dashboard screenshots and GIFs are now stale. Schedule an asset refresh alongside any major UI change. Don't ship the redesign and forget the marketing assets.
- **Dark mode variants needed mid-campaign** — if you only created light mode screenshots and someone asks for dark mode, you have to recapture everything. Create both variants from the start for the top 3 screens.
- **Platform changes image size requirements** — Product Hunt, Twitter, or LinkedIn occasionally change their recommended sizes. Verify sizes before a launch, not during.
- **Demo video shows a feature that was removed** — re-record the relevant section. A demo that shows a feature that doesn't exist anymore destroys trust.

## Implementation Checklist

- [ ] Product screenshots captured (dashboard, core feature, settings, mobile view)
- [ ] Screenshots at 2x resolution, 1200px width minimum
- [ ] Light mode and dark mode variants for top 3 screens
- [ ] Feature GIFs recorded (core workflow, wow moment, onboarding)
- [ ] GIFs under 4 seconds, 800px wide
- [ ] Laptop and phone mockups created
- [ ] Social media graphics created (announcement, feature highlights)
- [ ] Graphics correctly sized per platform (Twitter 1200x675, LinkedIn 1200x627, Instagram 1080x1080, Product Hunt 1270x760)
- [ ] Demo video recorded (75 seconds, hosted on YouTube)
- [ ] Demo video embedded on landing page and Product Hunt gallery
- [ ] Asset naming convention applied consistently
- [ ] Asset catalogue created listing every asset with filename, dimensions, and usage
- [ ] Assets stored in public/marketing/ and docs/visual-assets/

## Common Mistakes

- Using raw, uncleaned screenshots (test data, personal info, unprofessional workspace names visible)
- Only light mode screenshots (dark mode users feel excluded, dark mode screenshots look better on social media)
- GIFs that are 15 seconds long (too large, too slow to load, nobody watches the whole thing)
- No demo video (the highest-impact marketing asset and the most commonly skipped)
- Demo video that's 5 minutes long (nobody watches past 75 seconds — show the core value only)
- Screenshots with outdated UI (the live product looks different from the marketing — users feel deceived)
- No consistent naming convention (finding the right asset takes 10 minutes of scrolling through "Screenshot 2026-04-07 at 14.32.png")
- Creating assets for one platform only (Product Hunt gallery images that don't work on Twitter because the dimensions are wrong)
