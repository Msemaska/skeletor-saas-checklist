# Comparison Content

---

## What It Is

"[Your Product] vs [Competitor]" pages that capture users actively evaluating options.

## Page Types

| Type | Example | Target Keyword |
|---|---|---|
| 1v1 comparison | "[Product] vs [Competitor]" | "[product] vs [competitor]" |
| Alternative page | "[Competitor] Alternative" | "[competitor] alternative" |
| Category listicle | "Best [Category] Tools in [Year]" | "best [category] tools" |

Every major competitor gets a 1v1 comparison page. The top 3 competitors also get "alternative" pages.

## Comparison Page Structure

**Design decisions:**
- Title: "[Your Product] vs [Competitor]: [Year] Comparison" — keyword-first, under 60 characters
- Hook: 3 sentences. Who this comparison is for, what criteria you're comparing, and that you'll be honest about both products.

**Sections:**

| Section | Content |
|---|---|
| Overview | One paragraph per product — what it is, who it's for |
| Feature comparison table | Side-by-side table with checkmarks and "Partial" indicators |
| Pricing comparison | Plans and pricing side-by-side. Link to your pricing page. |
| Where [Your Product] wins | 3 specific advantages with evidence |
| Where [Competitor] wins | 2 specific areas where they're genuinely better |
| Verdict | Clear recommendation: who each product is best for |
| CTA | "Try [Your Product] free" or "See our pricing" |
| FAQ | 5 comparison-specific questions from People Also Ask |

## Content Rules

**Design decisions:**
- Be honest. If the competitor is better at something, say so.
- Use the competitor's current pricing and features. Check their website before publishing.
- Don't trash the competitor. Frame differences as "different strengths for different needs" not "they're bad, we're good."
- Include screenshots of both products where relevant — the reader wants to see what each looks like.
- Update comparison pages quarterly — competitor features and pricing change. Set a calendar reminder.
- Make the feature comparison table scannable and accurate.

## "Alternative" Page Structure

For "[Competitor] Alternative" keywords:

**Design decisions:**
- Title: "Best [Competitor] Alternative in [Year]"
- Hook: acknowledge what the competitor does well, then state why users look for alternatives (pricing, complexity, missing feature)
- 3 reasons to switch, each with a specific comparison point
- Migration section: how to switch from [Competitor] to [Your Product] (data export, setup steps)
- CTA: "Switch to [Your Product] — free trial"

## Category Listicle Structure

For "Best [Category] Tools" keywords:

**Design decisions:**
- Include 7 tools in the list
- Your product is included naturally — not at #1 (obvious bias), position it at #3
- Each tool: name, one-line description, best for, 3 key features, pricing, screenshot
- Be genuinely useful — provide accurate information for every tool listed
- Summary comparison table at the end

## Production Pipeline

1. Identify comparison keywords from keyword research (phase-14/03-keyword-research.md)
2. Research the competitor: current features, pricing, reviews (G2, Capterra, Reddit)
3. Claude drafts the comparison from your research
4. Human edit: verify every feature claim against the competitor's live site, add screenshots, add honest "where they win" section
5. Internal links: link to your features page, pricing page, and 2 related blog posts
6. Publish as MDX, distribute via social (thread format works well for comparisons)

## What to Test

- Feature comparison table is accurate against competitor's current site
- Competitor pricing is current
- "Where they win" section is genuinely honest
- Screenshots are current for both products
- FAQ generates FAQPage structured data
- Internal links to pricing page and features page are present
- Post is indexed and ranking for the target comparison keyword (check after 4 weeks)

## Edge Cases

- **Competitor changes their product after publishing** — update the comparison page within 2 weeks. Set Google Alerts for competitor brand names to catch announcements.
- **Competitor asks you to take down the comparison** — you're within your rights to compare publicly available products. Ensure all claims are factual and verifiable. Don't comply unless there's a genuine factual error.
- **Competitor publishes a comparison page about you** — respond by ensuring your comparison page is more thorough, more honest, and more up-to-date than theirs.
- **Reader comments that a claim is wrong** — verify immediately. If wrong, fix and thank the commenter publicly. If right, respond with the evidence.

## Implementation Checklist

- [ ] Major competitors identified (from phase-0/03-competitor-analysis.md)
- [ ] 1v1 comparison page created for each major competitor
- [ ] "Alternative" page created for top 3 competitors
- [ ] Category listicle created ("Best [Category] Tools")
- [ ] Feature comparison tables verified against live competitor sites
- [ ] Competitor pricing verified as current
- [ ] "Where they win" section included on every comparison page
- [ ] Quarterly update reminder set for all comparison pages
- [ ] Google Alerts configured for competitor brand names

## Common Mistakes

- Claiming your product is better at everything
- Outdated competitor pricing or features
- No "where they win" section
- Positioning your product at #1 in the category listicle
- Not updating comparison pages when competitors change
- Feature comparison table with only features you win
- No migration section on alternative pages
- Screenshots from 2 years ago
