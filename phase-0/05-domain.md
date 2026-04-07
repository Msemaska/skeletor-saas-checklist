# Domain Purchase

---

## What It Is

Your product's home on the internet. The domain affects brand perception, SEO, email deliverability, and trust.

## Process

### Step 1: Choose the Domain Name

**Rules:**
- Short as possible (under 15 characters ideal)
- Easy to spell and say out loud (the "radio test" — if you say it on a podcast, can people type it?)
- No hyphens
- No numbers (unless part of the brand, like 37signals)
- .com is still king for trust, but .io, .co, .app, .dev are acceptable for SaaS
- Avoid .net, .org, .info (look amateur for a SaaS product)

**If your ideal .com is taken:**
- Try [name]app.com, get[name].com, use[name].com, [name]hq.com
- Consider .io (tech audience), .co (general), .app (mobile-first), .dev (developer tools)
- Check if the taken .com is an active business or parked — parked domains can sometimes be bought

**Check before committing:**
- Is the name trademarked? (search USPTO, EUIPO)
- Are matching social media handles available?
- Does the name mean something offensive in another language?
- Is the name too similar to an established brand?

### Step 2: Purchase on Cloudflare Registrar

**Why Cloudflare:**
- At-cost pricing (no markup, no renewal price hikes)
- Free WHOIS privacy
- Free DNS hosting (fast, global)
- Easy integration with Cloudflare CDN, email routing, and security later

**Steps:**
1. Create Cloudflare account
2. Search for domain in Cloudflare Registrar
3. Purchase (WHOIS privacy enabled by default)
4. Domain is now managed in Cloudflare DNS

### Step 3: Configure DNS Basics

Immediately after purchase:
- Point nameservers to Cloudflare (automatic if bought there)
- Don't add any records yet — those come in later phases (Vercel, email, etc.)

### Step 4: Secure Related Domains (optional)

Consider buying:
- Common misspellings of your domain
- The .com if you bought a different TLD (redirect it later)
- Country-specific TLDs if targeting specific markets (.co.uk, .de, etc.)

Don't go overboard — 1-3 extra domains max.

## Deliverables

- [ ] Domain name chosen (passes radio test, short, spellable)
- [ ] Trademark conflicts checked
- [ ] Social media handle availability confirmed
- [ ] Domain purchased on Cloudflare Registrar
- [ ] WHOIS privacy confirmed enabled
- [ ] DNS managed by Cloudflare
