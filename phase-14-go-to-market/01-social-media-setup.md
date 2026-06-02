# Social Media Setup & Automation

---

## What It Is

Setting up social media accounts, branding them consistently, connecting them to the scheduling tool, and establishing the ongoing posting cadence. This is the distribution infrastructure — content comes from Phase 14, this phase handles where and when it goes out.

## Accounts to Create

| Platform | Type | Priority |
|---|---|---|
| Twitter/X | Personal founder account + product account | Must have |
| LinkedIn | Company page | Must have |
| GitHub | Organization | Must have |
| YouTube | Channel | Must have |
| Discord | Server (if community) | Must have |
| Instagram | Business account | Optional |
| TikTok | Business account | Optional |
| Facebook | Page | Optional |

**Design decisions:**
- Create the "must have" accounts before launch. Optional accounts can wait until you have content specifically for those platforms.
- Post from the founder's personal account primarily, repost to the product account.
- Create a GitHub org even if your product isn't open source.

## Branding Each Account

Every account uses assets from the copy bank (phase-13-content/01-foundational-copy-bank.md) and visual asset library (phase-13-content/02-visual-asset-library.md):

**Design decisions:**
- Profile image: logomark on all accounts (consistent, recognisable at small size)
- Banner/cover image: product name + tagline + screenshot mockup (sized per platform — dimensions in phase-1-brand-foundation/01-logo.md)
- Bio: platform-specific version from the copy bank's social media bios section
- Website link: landing page URL on all accounts
- All accounts use the same handle/username where possible

## Scheduling Tool

Use Postiz (self-hosted).

**Design decisions:**
- Connect all active social media accounts to Postiz
- Postiz handles: scheduling posts across platforms, queue management, optimal posting times, and analytics
- Batch-create content weekly (Monday), schedule for the week
- The content comes from Phase 14 (tweets, social graphics, repurposed blog posts). Postiz is only the distribution tool, not the content creation tool.

## Posting Cadence

**Design decisions:**
- **Twitter/X:** 1 post per day minimum, 1 thread per week
- **LinkedIn:** 3 posts per week
- **YouTube:** 1 video per month (tutorials, feature updates)
- **Discord:** active engagement when community members post, 1 announcement per feature launch
- **GitHub:** keep the README and org profile up to date, no posting cadence needed
- Consistency matters more than volume. Post every day rather than batching 5 posts in one day then going silent.

## Launch Day Plan

**Design decisions:**
- Pre-schedule all launch day posts in Postiz the day before launch
- Twitter/X: launch announcement thread (7 tweets), pin it to profile
- LinkedIn: launch announcement post with product screenshot
- Discord: announcement in #announcements channel
- GitHub: update org README with launch status
- Coordinate timing: all posts go out within 1 hour of launch
- Engage with every reply and comment for the first 4 hours after launch

## What to Test

- All accounts are created with consistent branding (same handle, same profile image, same bio messaging)
- Postiz is connected to all active accounts
- Scheduled posts go out at the correct time on each platform
- Launch day posts are pre-scheduled and ready
- Bio links point to the correct landing page URL

## Edge Cases

- **Handle not available on a platform** — use the closest variant (add "app", "hq", or "io" suffix). Document the actual handle per platform in the copy bank.
- **Account gets suspended** — don't use automation that violates platform terms (mass following, auto-DMs). Postiz for scheduling is fine — it's the same as using the platform's native scheduler.
- **Founder leaves the company** — if the founder's personal account is the primary distribution channel, establish the product account early and start building its following before transitioning.

## Implementation Checklist

- [ ] Twitter/X account created (founder personal + product)
- [ ] LinkedIn company page created
- [ ] GitHub organization created
- [ ] YouTube channel created
- [ ] Discord server created (if community model)
- [ ] Profile images set to logomark on all accounts
- [ ] Banners set with product name + tagline + screenshot
- [ ] Bios set from copy bank per platform
- [ ] Website links set to landing page URL
- [ ] Postiz installed (self-hosted) and connected to all accounts
- [ ] Posting cadence defined and documented
- [ ] Launch day posts pre-scheduled in Postiz
- [ ] First 4 hours post-launch: active engagement planned

## Common Mistakes

- Inconsistent branding across platforms (different logo, different bio, different handle)
- Product account only, no founder account (people follow people)
- Creating accounts on every platform but posting on none
- No scheduling tool (posting manually is unsustainable)
- Posting 10 times on launch day then going silent for 2 weeks
- Not engaging with replies (posting into the void)
- Identical content on every platform (each platform has different norms and formats)
