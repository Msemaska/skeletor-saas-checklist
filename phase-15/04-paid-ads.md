# Paid Ads

---

## What It Is

Paid advertising to accelerate user acquisition after organic channels are established. Ads are a post-launch, post-traction investment — don't spend money on ads before you have product-market fit and a converting landing page.

## When to Start

**Design decisions:**
- Start paid ads after: the product is live, the landing page converts organic traffic at 3%+ trial signup rate, and you have at least 1 month of organic data (from PostHog) to understand which messaging resonates
- If the landing page doesn't convert organic visitors, paid traffic won't convert either — fix the page first
- Set a monthly ad budget before running any campaigns. Start with $500/month and scale based on cost per acquisition.

## Platforms

### Google Ads (Search)

**Design decisions:**
- Start with search ads only (not display). Search ads target people actively looking for your category.
- Target branded keywords ("[your product name]") and comparison keywords ("[your product] vs [competitor]", "[competitor] alternative") — these have the highest purchase intent
- Target category keywords ("[category] tool", "[category] software") only after branded/comparison campaigns are profitable
- One ad group per keyword theme. Don't mix branded and category keywords in the same ad group.
- Landing pages: branded keywords → homepage, comparison keywords → comparison page, category keywords → features page. Match the landing page to the search intent.
- Set daily budget limits per campaign

### Meta Ads (Facebook/Instagram)

**Design decisions:**
- Use for retargeting first — show ads to people who visited your landing page but didn't sign up
- Install the Meta pixel on your marketing site to build the retargeting audience
- Retargeting ads: show a product screenshot + the primary benefit + CTA ("Start your free trial")
- Prospecting ads (cold audience): use lookalike audiences based on your existing users or waitlist subscribers
- Creative assets come from the visual asset library (phase-14/02-visual-asset-library.md) and ad creatives (phase-14/10-visual-content.md)

### Other Platforms

**Design decisions:**
- LinkedIn Ads: only for B2B products targeting specific job titles or industries.
- Reddit Ads: target specific subreddits relevant to your category. Reddit users are hostile to obvious ads — use conversational, authentic ad copy.
- Twitter/X Ads: optional. Use for promoted tweets of high-performing organic content.
- Don't advertise on every platform. Start with Google Ads search + Meta retargeting. Add other platforms only after these are profitable.

## Ad Creative Rules

**Design decisions:**
- All ad creatives come from the copy bank (headlines) and visual asset library (screenshots, mockups)
- 3 ad variants per campaign for A/B testing
- Ad copy leads with the outcome, not the feature ("Get paid 3x faster" not "Automated invoice reminders")
- Every ad has one CTA. Not two. One.
- Rotate creatives every 4 weeks — ad fatigue reduces performance over time

## Tracking & Attribution

**Design decisions:**
- UTM parameters on every ad URL: utm_source, utm_medium, utm_campaign, utm_content
- UTM convention: source=google/meta/linkedin, medium=cpc/retargeting, campaign=branded/comparison/category, content=variant-a/variant-b
- PostHog tracks which UTM sources convert to trials and paid users (phase-11/02-product-analytics.md)
- Calculate cost per acquisition (CPA) per campaign weekly: ad spend ÷ conversions
- Kill campaigns with a CPA above your target (LTV ÷ 3 is a reasonable maximum CPA — if LTV is $300, CPA should be under $100)

## What to Test

- Landing page converts organic traffic at 3%+ before running ads
- UTM parameters are present on all ad URLs
- Ad URLs land on the correct page (branded → homepage, comparison → comparison page)
- Meta pixel is installed and firing on the marketing site
- Retargeting audience is building (check Meta Ads Manager)
- PostHog tracks UTM sources and conversion events
- 3 ad variants per campaign
- Daily budget limits are set

## Edge Cases

- **Ads drive traffic but no conversions** — the landing page isn't converting, not the ad's fault. Review the landing page conversion rate for ad traffic specifically. If organic converts but paid doesn't, the ad is attracting the wrong audience — narrow targeting.
- **Competitor bids on your brand name** — bid on your own brand name (branded keywords). It's cheap and prevents competitors from capturing your traffic.
- **Ad account suspended** — Meta and Google suspend accounts for policy violations (misleading claims, prohibited content). Review ad policies before launching. Appeal suspensions immediately.
- **Budget spent too quickly** — set daily budget limits, not just monthly. If a campaign spends the full monthly budget in 3 days, the daily limit is too high.

## Implementation Checklist

- [ ] Landing page converts organic traffic at 3%+ trial signup rate
- [ ] Monthly ad budget defined ($500/month starting)
- [ ] Google Ads account created
- [ ] Google Ads branded keyword campaign set up
- [ ] Google Ads comparison keyword campaign set up
- [ ] Meta Ads account created
- [ ] Meta pixel installed on marketing site
- [ ] Meta retargeting campaign set up (visited landing page, didn't sign up)
- [ ] UTM parameter convention documented and applied to all ad URLs
- [ ] 3 ad variants per campaign
- [ ] CPA tracked weekly per campaign
- [ ] Creative rotation every 4 weeks

## Common Mistakes

- Running ads before the landing page converts organic traffic
- No UTM parameters (can't track which ads drive conversions)
- Same landing page for every ad (branded, comparison, and category keywords all go to the homepage)
- One ad variant per campaign (no A/B testing data)
- No daily budget limit (entire monthly budget spent in 3 days)
- Ad copy that describes features instead of outcomes
- Running ads on every platform simultaneously (spread too thin — start with Google + Meta retargeting)
- Never rotating creatives (ad fatigue after 4 weeks)
- Not tracking CPA per campaign (spending money without knowing the return)
