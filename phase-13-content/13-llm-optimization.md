# LLM Optimization

---

## What It Is

How to get your product mentioned in AI-generated responses from ChatGPT, Claude, Perplexity, and Google AI Overviews. When someone asks "what's the best [category] tool?", your product should appear in the answer.

## How LLMs Find Products to Recommend

**Design decisions:**
- LLMs pull from publicly crawled web content. Your product appears in AI responses when it's mentioned frequently across authoritative, independent sources.
- Third-party mentions carry more weight than self-promotion.
- Structured, factual content (comparison tables, FAQ answers, feature lists) is easier for LLMs to parse and cite than narrative prose.
- Build 5+ interconnected pages per topic cluster (phase-14-content/03-keyword-research.md).

## Strategy

### 1. Structured Comparison Content

Already covered in detail in phase-14-content/06-comparison-content.md. The key decisions for LLM optimization:

**Design decisions:**
- Every comparison page includes a feature comparison table — LLMs parse tables better than paragraphs
- Include clear verdict statements: "[Product] is best for [use case]" — LLMs extract these as recommendations
- FAQ sections with direct-answer first sentences are frequently cited by LLMs as featured snippets and AI answers

### 2. Monitor LLM Mentions

Track whether and how LLMs mention your product.

**Design decisions:**
- Monthly monitoring: query ChatGPT, Claude, and Perplexity with your category's key questions
- Questions to ask each LLM: "What's the best [category] tool?", "What are the top [category] alternatives to [competitor]?", "[Your product] vs [competitor] — which is better?", "What tools do [your audience] use for [your use case]?"
- Record the responses in a spreadsheet: date, LLM, query, whether your product was mentioned, what was said, which sources were cited
- If your product isn't mentioned: identify which products ARE mentioned and why (usually more third-party references, more structured content, or older/more established domain)
- If your product is mentioned incorrectly: you can't directly correct LLM responses, but you can update your own content to be more accurate and prominent, which LLMs will pick up in future training/crawling

### 3. Encourage Third-Party Reviews

Third-party mentions are the strongest signal for LLM inclusion.

**Design decisions:**
- Submit your product to aggregator sites: G2, Capterra, AlternativeTo, StackShare, Product Hunt
- Encourage happy customers to leave reviews on G2 and Capterra — these sites are heavily cited by LLMs
- Write guest posts on industry blogs that mention your product in context
- Contribute to relevant GitHub awesome-lists (e.g., "awesome-[category]")
- Get listed on "alternatives to [competitor]" pages — these are frequently cited in LLM responses
- Post on Reddit in relevant subreddits when genuinely helpful — Reddit threads are a major LLM training source

**How to ask for reviews:**
- After a customer completes their first 3 months and has measurable results (same timing as case studies — phase-14-content/07-case-studies.md)
- Send a short email: "Would you mind leaving a quick review on G2? It takes 5 minutes and helps other [audience] find us."
- Include the direct link to the review form — don't make them navigate to it
- Don't offer incentives for reviews (violates most platform terms of service)

### 4. Content Structure for LLM Parsing

**Design decisions:**
- Use clear headings that match questions people ask ("How much does [product] cost?" not "Pricing Information")
- FAQ sections with the direct answer in the first sentence — LLMs extract this for AI-generated answers
- Feature comparison tables with clear column headers and checkmarks — more parseable than prose
- Schema markup (FAQPage, Product) helps search engines and LLMs understand content structure
- Keep product descriptions factual and specific — "Automates invoice reminders for freelancers, starting at $9/month" is more citable than "A powerful platform that transforms your workflow"

## What to Test

- Monthly LLM queries show your product mentioned (or improving over time)
- Comparison pages include feature tables and FAQ sections with direct-answer first sentences
- Product is listed on G2, Capterra, AlternativeTo, and Product Hunt
- At least 3 reviews on G2 or Capterra from real customers
- Relevant GitHub awesome-lists include your product
- Schema markup is present on comparison pages and FAQ sections

## Edge Cases

- **LLM mentions your product but with wrong information** — update your own content to be more prominent and accurate. LLMs will pick up corrections in future crawls. You cannot directly edit LLM training data.
- **Competitor is mentioned but you're not** — check why: do they have more third-party reviews? More comparison content? More backlinks from authoritative sources? Close the gap on whichever signal they have that you don't.
- **LLM traffic overtakes organic search** — continue optimizing for structured, authoritative, well-linked content. The same content that ranks in Google performs well in LLM citations.
- **Review platform changes their terms** — G2, Capterra, and others occasionally change review policies. Never incentivize reviews or post fake ones — platform bans are permanent and visible.

## Implementation Checklist

- [ ] Comparison pages include feature tables and FAQ with direct-answer first sentences
- [ ] Schema markup (FAQPage, Product) on comparison pages and FAQ sections
- [ ] Product submitted to: G2, Capterra, AlternativeTo, StackShare, Product Hunt
- [ ] Customer review request process established (email after 3 months with direct G2 link)
- [ ] Monthly LLM monitoring: query ChatGPT, Claude, Perplexity with category questions
- [ ] LLM mention tracking spreadsheet created (date, LLM, query, mentioned, source)
- [ ] Relevant GitHub awesome-lists identified and submitted to
- [ ] Guest post opportunities identified in industry blogs

## Common Mistakes

- Expecting LLM mentions immediately (takes months of building third-party presence)
- Only self-promoting on your own site (third-party mentions matter more)
- Narrative prose instead of structured content (tables, FAQs, and clear headings are more parseable)
- No monitoring (you don't know if you're mentioned or not)
- Incentivizing reviews (violates platform terms, leads to bans)
- Ignoring Reddit (one of the largest LLM training sources)
- Vague product descriptions ("powerful platform" — not citable)
- No schema markup on comparison and FAQ pages
