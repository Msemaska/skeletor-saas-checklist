# Foundational Copy Bank

---

## What It Is

A single document containing every piece of marketing copy your product needs — written once, reused everywhere. When Product Hunt asks for a tagline, when G2 asks for a description, when Twitter needs a bio, when an investor asks "what do you do?" — you open this document and copy-paste. No rewriting from scratch each time.

## Where to Store It

One markdown file in the repository: `docs/copy-bank.md`. Version-controlled, searchable, always up to date.

## What the Copy Bank Contains

### 1. One-Liner (under 15 words)

The single sentence that explains what your product does. Derived from phase-0/01-value-proposition.md.

Used on: Twitter/X bio, LinkedIn tagline, email signature, Product Hunt tagline, App Store subtitle, Slack community intro.

**Rules:**
- Under 15 words
- Outcome-focused, not feature-focused
- A stranger with zero context understands it in 5 seconds
- No jargon, no buzzwords

### 2. Taglines by Character Limit

Different platforms have different character limits. Pre-write the tagline at each common limit so you don't have to think about it when filling out a form.

| Length | Use Case |
|---|---|
| 60 characters | Product Hunt tagline |
| 100 characters | App store subtitle, directory listings |
| 160 characters | Meta description, Google result preview |
| 280 characters | Tweet / X post |

Each version is a self-contained description — not a truncated version of a longer one.

### 3. Short Description (3 sentences)

A brief paragraph explaining: what the product does, who it's for, and what outcome it delivers.

Used on: Product Hunt description, directory listings (BetaList, SaaSHub, AlternativeTo), LinkedIn company description, GitHub org bio, app store descriptions.

**Rules:**
- Start with the problem, not the solution
- Name the audience: "for [who]"
- End with the outcome: "so you can [result]"
- Under 50 words

### 4. Full Description (3 paragraphs)

An expanded version covering: the problem, the solution, key features, and the differentiator.

Used on: Product Hunt "About" section, G2/Capterra listings, landing page hero + subheadline, investor one-pager, press kit.

**Rules:**
- First paragraph: problem + solution (what you do and why it matters)
- Second paragraph: key features (3 most important capabilities)
- Third paragraph: differentiator (what makes you different from alternatives)
- Under 150 words total
- Write for someone evaluating 5 products in 10 minutes — they're scanning, not reading

### 5. Boilerplate (press/formal)

A factual, third-person description used in press releases, partner pages, and formal contexts.

Used on: press releases, "About Us" sections of partner/integration pages, conference speaker bios, award submissions.

**Rules:**
- Third person: "[Product Name] is a..." not "We are..."
- Factual tone — no marketing superlatives ("revolutionary", "game-changing")
- Include: what the product is, when it was founded, who it's for
- 3 sentences

### 6. Elevator Pitch (30 seconds spoken)

The verbal version — what you say when someone asks "what do you do?" at a conference or in a meeting.

Used for: networking, investor conversations, podcast introductions, video intros.

**Rules:**
- 75 words when spoken aloud (30 seconds)
- Conversational tone — this is spoken, not read
- Structure: "You know how [problem]? We built [product] that [solution]. So instead of [old way], you can [new way]."
- Practice saying it out loud — if it sounds awkward spoken, rewrite it

### 7. Feature Summaries

One-line descriptions of each major feature. These get reused in: feature comparison tables, directory listings, app store feature lists, social media posts.

**Format per feature:**
- Feature name
- One-sentence description (what it does for the user, not how it works)
- One-sentence differentiator (why yours is better than the alternative)

### 8. FAQ Answers

Pre-written answers to the questions every platform and every user asks.

**Standard questions to pre-answer:**
- What is [product]?
- Who is it for?
- How much does it cost? / Is there a free plan?
- How is it different from [competitor]?
- Is my data secure?
- Can I cancel anytime?
- Do you offer refunds?
- How do I get support?
- What integrations do you support?
- Is there an API?

Each answer: 3 sentences, direct, no fluff. These get copy-pasted into: Product Hunt Q&A, G2 vendor responses, support knowledge base, marketing site FAQ, App Store review responses.

### 9. Product Hunt First Comment

The maker comment posted immediately when you launch on Product Hunt.

**Structure:**
- Personal intro (who you are, 1 sentence)
- Origin story (why you built this, 3 sentences — the problem you experienced)
- What the product does (2 sentences)
- What makes it different (1 sentence)
- Current status (beta, launched, user count if meaningful)
- Ask for feedback, not votes ("I'd love to hear what you think" not "please upvote")

Write this in advance. Don't improvise on launch day.

### 10. Social Media Bios

Pre-written bios for each platform:

| Platform | Format |
|---|---|
| Twitter/X | One-liner + link (160 chars) |
| LinkedIn (company) | Short description (200 chars) + full description (2000 chars) |
| GitHub | One-liner + link |
| YouTube | Short description + link |
| Product Hunt | Tagline (60 chars) |
| Discord server | One-liner description |

## Maintaining the Copy Bank

**Design decisions:**
- Update the copy bank whenever the product positioning changes (new feature, pivot, new audience)
- The copy bank is the source of truth — if the Twitter bio says something different from the Product Hunt tagline, one of them is wrong
- Review quarterly: is the messaging still accurate? Do the feature summaries reflect the current product?
- When adding a new feature, add its feature summary to the copy bank before announcing it anywhere

## What to Test

- Every tagline variant reads clearly to someone who has never seen the product
- The 60-character tagline fits within Product Hunt's limit
- The short description answers: what, who, and why in under 50 words
- FAQ answers are accurate and match current pricing, features, and policies
- Social media bios are consistent across platforms (same core message, adapted to length)
- The elevator pitch sounds natural when spoken aloud (not like reading marketing copy)

## Edge Cases

- **Product positioning changes** — update the copy bank first, then propagate to all platforms. The copy bank is always updated before any individual platform.
- **Different audiences need different messaging** — if you serve both developers and marketers, create audience-specific variants of the short description and feature summaries. Label them clearly in the copy bank.
- **Platform changes its character limits** — Product Hunt, Twitter, and app stores occasionally change limits. Verify limits before a launch, not during.
- **Competitor mentioned in your copy** — comparison content and "how is it different from X" FAQ answers reference competitors by name. Keep these factual and honest. Update when competitors change their product.

## Implementation Checklist

- [ ] copy-bank.md created in docs/ directory
- [ ] One-liner written (under 15 words)
- [ ] Taglines written at 60, 100, 160, and 280 character limits
- [ ] Short description written (under 50 words)
- [ ] Full description written (under 150 words, 3 paragraphs)
- [ ] Boilerplate written (third person, factual, 3 sentences)
- [ ] Elevator pitch written and tested spoken aloud (30 seconds)
- [ ] Feature summaries written for every major feature
- [ ] FAQ answers written for all standard questions
- [ ] Product Hunt first comment drafted
- [ ] Social media bios written for all platforms
- [ ] Copy bank stored in the repository (docs/copy-bank.md)
- [ ] Quarterly review reminder set

## Common Mistakes

- No copy bank — rewriting the product description from scratch every time a new platform asks for it (inconsistent messaging, wasted time)
- Copy bank stored outside the repository where nobody can find it
- Tagline that's clever but unclear ("Reimagine your workflow paradigm" — nobody knows what this means)
- Feature descriptions that explain how it works instead of what it does for the user
- FAQ answers that dodge the pricing question (users want a straight answer)
- Different messaging on every platform (Twitter says one thing, Product Hunt says another — confuses users who see both)
- Never updating the copy bank after product changes (copy drifts from reality)
- Product Hunt first comment that asks for upvotes instead of feedback
