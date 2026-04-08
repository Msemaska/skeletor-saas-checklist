# Visual Content

---

## What It Is

Ad creatives and social media graphics — the visual assets produced on an ongoing basis for campaigns, social posts, and promotions. Different from the visual asset library (phase-14/02-visual-asset-library.md) which is foundational assets created once. This covers the ongoing production of campaign-specific visuals.

## Ad Creatives

### What to Create

| Creative Type | Use Case | Size |
|---|---|---|
| Banner ad | Google Display, retargeting | 1200x628 |
| Social ad | Facebook/Meta, LinkedIn sponsored | 1200x628 |
| Story/Reel ad | Instagram, Facebook Stories | 1080x1920 |
| Square ad | Instagram feed, Facebook feed | 1080x1080 |

### Design Rules

**Design decisions:**
- Every ad includes: your product name or logo, a clear headline (under 10 words), a screenshot or visual of the product, a CTA button text ("Try Free", "See Pricing")
- Use your brand colours and typography from the design system (phase-1/05-design-system.md)
- Generate programmatically with @vercel/og — create React templates for each ad format, pass in different headlines and screenshots to produce variants
- Create 3 variants per campaign for A/B testing: same message, different visual. Or same visual, different headline.
- Headlines come from the copy bank (phase-14/01-foundational-copy-bank.md) — don't write new copy for every ad

### Production Pipeline

1. Define the campaign goal (sign-ups, feature awareness, retargeting)
2. Pull headline variants from the copy bank
3. Select product screenshot or GIF from the visual asset library
4. Claude generates 3 headline variants for the ad
5. Render ad creatives using @vercel/og templates at each required size
6. Upload to ad platform (Google Ads, Meta Ads)

## Social Media Graphics

### Types

| Type | Purpose | Format |
|---|---|---|
| Feature highlight card | Announce or promote a specific feature | 1200x675 (Twitter), 1080x1080 (Instagram) |
| Carousel post | Multi-slide educational content or product walkthrough | 1080x1080, 5 slides |
| Infographic | Data visualization, process explanation, comparison | 1080x1350 (portrait, Instagram) |
| Quote card | Customer testimonial or founder insight | 1080x1080 |
| Stat card | Single striking statistic with context | 1200x675 |

### Design Rules

**Design decisions:**
- Every graphic includes your product name or logo — identifiable without context
- Use the same visual language across all graphics: brand colours, consistent typography, same corner radius on cards
- Text on graphics must be readable at mobile size — minimum 16px font size
- Feature highlight cards pull from the copy bank's feature summaries
- Stat cards use stats from blog posts (the same sourced statistics)
- Generate programmatically with @vercel/og for template-based graphics. Claude creates the content, @vercel/og renders it.

### Carousel Post Structure

**Design decisions:**
- Slide 1: hook slide with a bold statement or question (this is the cover — it must stop the scroll)
- Slides 2-4: value slides with one point per slide
- Slide 5: CTA slide ("Follow for more" or "Try [product] free" with link in bio/comments)
- Each slide is self-contained — makes sense even if someone screenshots a single slide
- Consistent visual template across all slides (same background, typography, layout)
- Text per slide: 3 sentences maximum

### Production Pipeline

1. Identify the topic (from editorial calendar or blog post repurposing)
2. Claude generates the text content (headline, slide content, CTA)
3. Render using @vercel/og templates or Claude + SVG/React for more complex designs
4. Review: is the hook scroll-stopping? Is each slide readable on mobile?
5. Schedule via Postiz with appropriate caption and hashtags

## What to Test

- Ad creatives render correctly at all required sizes
- Text is readable at mobile size on all graphics
- Product name or logo is visible on every graphic
- Carousel slides are consistent in visual style
- Brand colours and typography match the design system
- 3 ad variants per campaign for A/B testing

## Edge Cases

- **Ad platform rejects the creative** — check platform-specific rules (text-to-image ratio for Meta, animation rules for Google Display). Meta flags images with more than 20% text.
- **Graphics look different on different devices** — test on a real phone, not just desktop. Dark mode on some platforms inverts or adjusts colours.
- **Carousel post engagement drops on later slides** — the hook slide isn't compelling enough. Rewrite slide 1 to be more specific and provocative.

## Implementation Checklist

- [ ] @vercel/og templates created for each ad format (banner, social, story, square)
- [ ] @vercel/og templates created for each social graphic type (feature card, carousel, stat card, quote card)
- [ ] Brand colours and typography configured in templates
- [ ] 3 ad variants per campaign as standard practice
- [ ] Carousel template: 5 slides with consistent visual style
- [ ] All graphics include product name or logo
- [ ] Graphics tested at mobile size for readability

## Common Mistakes

- Different visual style on every graphic (no brand consistency)
- Text too small to read on mobile
- No product name or logo on the graphic (unidentifiable when shared)
- Only one ad variant per campaign (no A/B testing data)
- Writing new copy for every ad instead of pulling from the copy bank
- Carousels with walls of text per slide (3 sentences max)
- Not testing on a real phone
