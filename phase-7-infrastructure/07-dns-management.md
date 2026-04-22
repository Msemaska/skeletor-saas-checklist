# DNS Management

---

## What It Is

How your domain name points to your application and how subdomains are configured. DNS is the first thing you set up and the last thing you want to debug in production.

## Provider

Use Cloudflare DNS. Free, fast global propagation, integrates with Cloudflare R2 (object storage) and Cloudflare CDN, and provides DDoS protection at the DNS level.

**AI interface:** Cloudflare CLI (`wrangler`), Cloudflare MCP server.

## Records

**Essential DNS records for a Railway-hosted SaaS:**

| Type | Name | Value | Purpose |
|---|---|---|---|
| CNAME | @ | Railway CNAME target | Root domain → Railway app |
| CNAME | www | Railway CNAME target | www subdomain → Railway |
| CNAME | app | Railway CNAME target | App subdomain (if app is on a subdomain) |
| CNAME | assets | R2 custom domain | Object storage CDN |
| MX | @ | Email provider records | Email delivery |
| TXT | @ | SPF record | Email authentication |
| TXT | _dmarc | DMARC record | Email policy |
| CNAME | resend._domainkey | Resend DKIM | Email signing |

**Design decisions:**
- Use CNAME records pointing to Railway's provided target for the primary domain and subdomains. Cloudflare supports CNAME flattening for the root domain.
- Keep DNS TTL low (300 seconds / 5 minutes) during initial setup so changes propagate quickly. Increase to 3600 seconds (1 hour) once stable.
- Email DNS records (SPF, DKIM, DMARC) are covered in Phase 11 (Email & Domain Setup) — list them here for completeness but configure them during that phase

## Cloudflare Proxy

Cloudflare can proxy traffic through its network (orange cloud icon) or just provide DNS resolution (grey cloud icon).

**Design decisions:**
- **Proxy ON (orange cloud)** for the primary domain and www — provides DDoS protection and CDN caching
- **Proxy OFF (grey cloud)** for email records (MX, SPF, DKIM) — email traffic must go directly to the mail server, not through Cloudflare's HTTP proxy
- **Proxy ON** for the assets subdomain (Cloudflare CDN for object storage)
- When Cloudflare proxy is on, Railway handles SSL for the origin connection, while Cloudflare handles SSL for the edge connection (user ↔ Cloudflare). Set Cloudflare's SSL mode to "Full (Strict)" so both connections are encrypted.

## Domain Structure

**Design decisions:**
- `yourdomain.com` — marketing site
- `app.yourdomain.com` — the application (separated from marketing site at the DNS level)
- `assets.yourdomain.com` — object storage / CDN
- `staging.yourdomain.com` — staging environment
- No other subdomains unless needed — every subdomain is another SSL certificate, another DNS record, another thing to maintain

## Edge Cases

- **DNS not propagated yet** — after adding or changing a record, it takes up to the TTL for the change to propagate globally. During initial setup, use a low TTL (300 seconds). Use DNS checker tools to verify propagation before expecting Railway to provision SSL.
- **Railway CNAME target changes** — rare, but if Railway changes their CNAME target, update the DNS records. With Cloudflare proxy ON, the transition is seamless.
- **Subdomain for each customer** — if your SaaS gives each customer a subdomain (customer.yourdomain.com), use a wildcard DNS record (*.yourdomain.com → Railway CNAME) and Railway's custom domain support. Configure wildcard certificate handling in your app.
- **Domain transfer** — if you move registrars (e.g., from GoDaddy to Cloudflare Registrar), update nameservers and wait for propagation. No DNS record changes needed if Cloudflare was already the DNS provider.
- **DNS cache poisoning** — Cloudflare's DNS provides DNSSEC (DNS Security Extensions). Enable it for protection against DNS spoofing.

## Security Considerations

- Enable DNSSEC on Cloudflare to protect against DNS cache poisoning and spoofing attacks. Without DNSSEC, an attacker can redirect your domain's traffic to their server by poisoning a DNS resolver's cache.
- Cloudflare proxy (orange cloud) adds DDoS protection and CDN caching. With Railway, the origin is already managed infrastructure, but Cloudflare proxy adds an additional layer of protection and performance.
- Set Cloudflare SSL mode to "Full (Strict)" — never "Flexible". Flexible mode means traffic between Cloudflare and your server is unencrypted, which allows man-in-the-middle attacks on the origin connection. Full (Strict) ensures both the edge connection (user → Cloudflare) and the origin connection (Cloudflare → server) are encrypted and certificate-validated.
- Cloudflare account security is critical — whoever controls the DNS can redirect all traffic, intercept SSL, and impersonate the application. Enable two-factor authentication on the Cloudflare account, use a strong password, and limit account access to essential team members only.
- Wildcard DNS records (`*.yourdomain.com`) create a subdomain takeover risk if unused subdomains resolve to decommissioned infrastructure. Only use wildcard records if your application actively manages all subdomains (e.g., multi-tenant customer subdomains with on-demand TLS).
- Domain registration security: enable registrar lock (transfer lock) to prevent unauthorized domain transfers. A stolen domain is a total compromise — the attacker controls all traffic, email, and SSL certificates.

## Implementation Checklist

- [ ] Domain DNS managed by Cloudflare
- [ ] CNAME for root domain pointing to Railway target
- [ ] CNAME for www pointing to Railway target (or redirect to root)
- [ ] Preview environments use Railway-generated URLs (no DNS needed)
- [ ] CNAME for assets subdomain pointing to R2 custom domain
- [ ] Cloudflare proxy enabled for web-facing records (orange cloud)
- [ ] Cloudflare proxy disabled for email records (grey cloud)
- [ ] Cloudflare SSL mode set to Full (Strict)
- [ ] TTL set to 300 seconds during setup, 3600 once stable
- [ ] DNSSEC enabled
- [ ] Email DNS records prepared (configured in Phase 11)

## Common Mistakes

- Not setting up DNS before adding a custom domain in Railway (Railway can't provision SSL if the domain doesn't resolve)
- High TTL during initial setup (changes take hours to propagate, debugging is slow)
- Cloudflare proxy ON for email records (email delivery breaks)
- Cloudflare SSL mode set to "Flexible" instead of "Full (Strict)" (traffic between Cloudflare and your server is unencrypted — security vulnerability)
- Not using Cloudflare proxy when it would add useful DDoS and CDN benefits
- Too many subdomains (each one is a maintenance burden — use path-based routing when possible)
- Forgetting to update DNS when migrating servers (old server IP in DNS after the new server is live)
