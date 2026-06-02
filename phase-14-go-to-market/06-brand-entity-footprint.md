# Brand Entity Footprint (Entity Stacking)

---

## What It Is

The process of establishing a consistent, recognizable brand presence across high-authority platforms so that search engines and large language models treat your company as a real, trusted entity. Google and LLMs increasingly rely on brand mention signals — consistent references to the same company across reputable domains — to decide who ranks and who gets cited. Without an entity footprint, you're a domain. With one, you're a brand.

The SaaS-relevant version of this differs from the local-business version: NAP (name, address, phone) consistency matters far less when you have no storefront, but Web2 brand reservation, software-directory citations, schema markup, and press mentions matter more. Build this in parallel with the marketing site, not after launch.

## Brand Reference Sheet

**Design decisions:**
- Create a single canonical document (Notion page) that defines every immutable fact about the brand: exact name spelling, capitalization (e.g., "SiteGPT" not "Sitegpt"), tagline, one-sentence description, paragraph description, founder name and bio, founding year, category, logo files, brand colors, primary URL, contact email, support email, X handle, LinkedIn URL.
- Every listing, profile, and press release uses these exact strings. Variants ("SiteGPT" vs "Site GPT" vs "site-gpt") fragment the entity in Google's eyes.
- The sheet is the source of truth. When something changes — a new tagline, a new logo — update the sheet first, then propagate to every listing.
- Maintain a list of every place the brand is listed, with the last-verified date. Re-verify quarterly.

## Schema.org Organization Markup

**Design decisions:**
- Add JSON-LD `Organization` schema to the marketing site homepage and root layout. Include: name, alternateName (common misspellings), URL, logo, sameAs array (every official social profile and Web2 site), founder, foundingDate, contactPoint, description.
- The `sameAs` array is the single most important signal in this work. It tells Google which external profiles belong to this entity. Every Web2 profile and social account you create gets added here.
- Validate with Google's Rich Results Test after every change. A broken schema is worse than no schema.

## Social Profile Reservation

**Design decisions:**
- Reserve the exact brand handle on every major platform on day one, even if you have no plans to use the platform actively. Squatters and impersonators are a real problem post-launch.
- Mandatory: X, LinkedIn (company page), YouTube, Instagram, TikTok, Facebook (business page), Threads, Bluesky, Pinterest, Reddit (user account matching brand), GitHub (organization), npm (organization scope if applicable), Hacker News (user account).
- Use the same logo, banner, bio, and URL across all profiles. Bio length varies per platform but the core sentence stays identical.
- Add every profile URL to the schema.org `sameAs` array.
- Active posting matters for ranking on the platform itself, not for entity signals. Reservation alone is enough for the entity-stacking purpose. Decide separately which platforms warrant active investment based on where the ICP actually spends time.

## SaaS Directory Citations

**Design decisions:**
- Submit manually to legitimate, high-authority software directories. Skip Fiverr/Legiit citation gigs entirely — they place links on spammy directories that Google can identify, and the risk of negative association outweighs the link value.
- Required (every SaaS): G2, Capterra, GetApp, Software Advice, AlternativeTo, Product Hunt, SaaSHub, Slant, Crozdesk.
- Category-specific additions: Futurepedia and There's An AI For That for AI products, Awesome-* GitHub lists for developer tools, Indie Hackers Products for bootstrapped, BetaList for pre-launch.
- Use the brand reference sheet for every submission. Identical name, description, logo, URL.
- Track submission status, listing URL, and approval date in the brand reference sheet.
- Listings with reviews (G2, Capterra) need a review-collection plan separately — listings without reviews rank poorly within the directory. This is covered in Phase 16 (Growth & Retention) under social proof collection.

## Web2 Brand Reservation

**Design decisions:**
- Create branded mini-sites on free Web2 platforms to establish presence on high-authority domains. The goal is brand signal, not traffic.
- Required platforms: Medium, Dev.to, Hashnode, Substack, WordPress.com, Wix, Webflow, Ghost, Site123, Blogger, LiveJournal.
- Each mini-site is more than a stub: include the logo, the brand description (paragraph version from the reference sheet), 2-3 photos or product screenshots, links to the main site and primary social profiles.
- Cross-post 2-3 blog posts from the main blog to Medium, Dev.to, and Hashnode using their canonical-URL feature. Canonical URLs point back to the main site so Google attributes the ranking to the main domain.
- Add every Web2 profile URL to the schema.org `sameAs` array.
- Do not link-stuff. Each mini-site links to the main site once or twice, naturally placed.

## Press Releases

**Design decisions:**
- Distribute one press release at launch and additional releases for legitimately newsworthy events: funding rounds, major product launches, integrations with well-known brands, milestones (10k users, $1M ARR), notable hires.
- Use a paid distribution service for syndication. EIN Presswire and AB Newswire are the standard low-cost options that get pickup on hundreds of secondary news sites. Skip the "PR Newswire" tier unless you have a $5k+ budget and a genuinely major story.
- The press release does not need to be picked up by Forbes to do its job. The pickup on Yahoo Finance, MarketWatch, Benzinga, and dozens of local news syndicates is what generates the brand mentions across high-authority domains.
- Write the release for the syndicates, not for journalists. Lead with the news angle, include a quote from the founder, include boilerplate brand description from the reference sheet, include the canonical URL.
- Do not fabricate news. "We launched a beta" is not a press release. "We launched the first AI chatbot that does X" is, if it's actually true.

## LLM Citation Surface

**Design decisions:**
- LLMs cite brands that appear consistently across reputable sources. The above steps build this surface as a side effect.
- Add an `llms.txt` and `llms-full.txt` to the marketing site root with the brand description, primary use cases, and key URLs. This is an emerging convention for explicitly feeding context to LLM crawlers.
- Ensure the marketing site renders core brand information server-side (not behind JavaScript). LLM crawlers often don't execute JS, and content they can't read won't be cited.
- This overlaps with Phase 13 (Content) item 13 (LLM optimization). The entity footprint is the foundation; LLM optimization is the ongoing monitoring of how those signals translate into citations.

## Wikipedia and Wikidata

**Design decisions:**
- Do not create your own Wikipedia article. Editors will detect promotional intent and delete it, and a failed attempt is a permanent strike against the brand on the platform.
- Once the brand has earned independent press coverage (multiple unaffiliated articles in major publications), a Wikipedia article becomes viable. This is a Phase 16+ play, not a launch play.
- Wikidata is more permissive. Add a Wikidata entry for the company with structured properties (founded, founder, industry, website, social handles) once the company has any verifiable footprint. Wikidata feeds Google's Knowledge Graph directly.

## Google Knowledge Panel

**Design decisions:**
- The Knowledge Panel (the box that appears on the right of Google search for a brand name) is the goal of all of this work. It signals to users and to Google that the entity is recognized.
- You cannot directly request a Knowledge Panel. Google generates it when the entity signals are strong enough — typically: schema.org Organization markup, consistent presence across the platforms above, brand mentions across multiple authority domains, and a Wikidata entry.
- Once the panel appears, claim it via Google's entity verification flow. This lets you submit corrections and add a featured image.

## What to Test

- Schema.org Organization markup validates in Google's Rich Results Test
- The `sameAs` array in schema matches the actual list of active profiles (no broken links)
- Brand name appears identically on every profile (programmatically check via a small audit script)
- Every social profile uses the same logo dimensions and aspect ratio (visual consistency)
- Web2 mini-sites canonical URLs point to the main site
- Brand reference sheet is up to date and version-controlled
- `llms.txt` is served at the root and returns 200
- Marketing site renders core brand info without JavaScript (curl test)

## Edge Cases

- **Brand name is already taken on a major platform** — append a suffix that's still consistent across platforms (e.g., `@yourbrandhq`, `@getyourbrand`, `@yourbrandapp`). Whatever you choose, use it everywhere. Mixed suffixes ("yourbrand" on X, "getyourbrand" on Instagram) fragment the entity.
- **Company rebrand or name change** — the entity footprint is the most painful thing to migrate. Update the reference sheet first, then propagate. Keep old profiles alive for at least 12 months with a pinned post pointing to the new brand. Update `sameAs` array and schema.alternateName to include both old and new names during the transition.
- **Trademark dispute over the name** — resolve this before you build the footprint. Rebuilding after a forced rename is expensive and SEO-damaging.
- **Squatter holds your handle on a key platform** — most platforms have a trademark-based reclamation process. Filing requires a registered trademark, which is its own multi-month process.
- **Press release gets no organic pickup** — the syndication itself is the value. Do not pay for "guaranteed Forbes placement" services — they're either paid placements (which Google can detect and discount) or scams.
- **Web2 platform deletes inactive accounts** — log in to each at least once every 6 months. Set a calendar reminder. Some platforms purge after 90 days of inactivity.
- **Directory listings get outdated** — when the product changes meaningfully, update every directory listing. Outdated listings hurt conversion and signal that the brand is inactive.
- **Negative review on G2 or Capterra** — respond professionally on the platform. Do not attempt to remove it. Visible response handling is itself a trust signal.

## Implementation Checklist

- [ ] Brand reference sheet created in Notion with all canonical strings
- [ ] Schema.org Organization JSON-LD added to marketing site root layout
- [ ] Schema validates in Google's Rich Results Test
- [ ] Social handles reserved on all mandatory platforms (X, LinkedIn, YouTube, Instagram, TikTok, Facebook, Threads, Bluesky, Pinterest, Reddit, GitHub, HN)
- [ ] Each social profile filled out with brand logo, banner, bio, link
- [ ] Submitted to G2, Capterra, GetApp, Software Advice, AlternativeTo, Product Hunt, SaaSHub, Slant, Crozdesk
- [ ] Submitted to category-specific directories
- [ ] Web2 mini-sites created on Medium, Dev.to, Hashnode, Substack, WordPress.com, Wix, Webflow, Ghost, Site123, Blogger
- [ ] Each Web2 mini-site has logo, brand description, 2-3 images, link to main site
- [ ] Canonical URLs configured on cross-posted blog content
- [ ] `sameAs` array in schema.org markup includes every active profile URL
- [ ] `llms.txt` and `llms-full.txt` published at marketing site root
- [ ] Launch press release written and distributed via EIN Presswire or AB Newswire
- [ ] Wikidata entry created (once basic footprint exists)
- [ ] Quarterly re-verification reminder set in Notion
- [ ] Audit script in repo to check brand-name consistency across the `sameAs` array

## Common Mistakes

- Inconsistent brand name spelling/capitalization across profiles
- Skipping schema.org Organization markup — the single biggest entity signal you control
- Buying Fiverr/Legiit citation gigs that land on spammy directories
- Creating Web2 stubs with only a logo and a link — Google reads this as spam, not presence
- Submitting to directories without a review-collection plan, so the listing shows zero reviews
- Letting Web2 platforms purge inactive accounts after 90 days
- Trying to create your own Wikipedia article before the brand has earned independent coverage
- Fabricating press release news to game the syndication
- Stuffing the brand description with keywords instead of writing it for humans
- Forgetting to add new profiles to the `sameAs` array as they're created
- Treating this as a one-time setup instead of an ongoing maintenance task
- Doing this in Phase 14 but never updating it again after launch
