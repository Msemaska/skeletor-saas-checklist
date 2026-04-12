# DNS Management

---

## What It Is

How your domain name points to your server and how subdomains are configured. DNS is the first thing you set up (before Caddy can provision SSL certificates) and the last thing you want to debug in production.

## Provider

Use Cloudflare DNS. Free, fast global propagation, integrates with Cloudflare R2 (object storage) and Cloudflare CDN, and provides DDoS protection at the DNS level.

## Records

**Essential DNS records for a self-hosted SaaS:**

| Type | Name | Value | Purpose |
|---|---|---|---|
| A | @ | Server IP | Root domain → server |
| A | www | Server IP | www subdomain → server |
| A | app | Server IP | App subdomain → server (if app is on a subdomain) |
| A | staging | Staging server IP | Staging environment |
| CNAME | assets | R2 custom domain | Object storage CDN |
| MX | @ | Email provider records | Email delivery |
| TXT | @ | SPF record | Email authentication |
| TXT | _dmarc | DMARC record | Email policy |
| CNAME | resend._domainkey | Resend DKIM | Email signing |

**Design decisions:**
- Use A records pointing to the server IP for the primary domain and subdomains — not CNAME for the root domain (CNAME on root is technically invalid per RFC, though Cloudflare supports CNAME flattening)
- Keep DNS TTL low (300 seconds / 5 minutes) during initial setup so changes propagate quickly. Increase to 3600 seconds (1 hour) once stable.
- Email DNS records (SPF, DKIM, DMARC) are covered in Phase 12 (Email & Domain Setup) — list them here for completeness but configure them during that phase

## Cloudflare Proxy

Cloudflare can proxy traffic through its network (orange cloud icon) or just provide DNS resolution (grey cloud icon).

**Design decisions:**
- **Proxy ON (orange cloud)** for the primary domain and www — provides DDoS protection and hides your server's IP address
- **Proxy OFF (grey cloud)** for email records (MX, SPF, DKIM) — email traffic must go directly to the mail server, not through Cloudflare's HTTP proxy
- **Proxy ON** for the assets subdomain (Cloudflare CDN for object storage)
- When Cloudflare proxy is on, Caddy still handles SSL for the origin connection (Caddy ↔ app), while Cloudflare handles SSL for the edge connection (user ↔ Cloudflare). Set Cloudflare's SSL mode to "Full (Strict)" so both connections are encrypted.

## Domain Structure

**Design decisions:**
- `yourdomain.com` — marketing site
- `app.yourdomain.com` — the application (separated from marketing site at the DNS level)
- `assets.yourdomain.com` — object storage / CDN
- `staging.yourdomain.com` — staging environment
- No other subdomains unless needed — every subdomain is another SSL certificate, another DNS record, another thing to maintain

## Edge Cases

- **DNS not propagated yet** — after adding or changing a record, it takes up to the TTL for the change to propagate globally. During initial setup, use a low TTL (300 seconds). Use DNS checker tools to verify propagation before expecting Caddy to provision SSL.
- **Server IP changes** — if you move to a new server, update the A records. With Cloudflare proxy ON, you can change the server IP without users seeing downtime (Cloudflare continues serving cached content during the transition).
- **Subdomain for each customer** — if your SaaS gives each customer a subdomain (customer.yourdomain.com), use a wildcard DNS record (*.yourdomain.com → server IP) and Caddy's on-demand TLS. Each new customer subdomain gets a certificate automatically.
- **Domain transfer** — if you move registrars (e.g., from GoDaddy to Cloudflare Registrar), update nameservers and wait for propagation. No DNS record changes needed if Cloudflare was already the DNS provider.
- **DNS cache poisoning** — Cloudflare's DNS provides DNSSEC (DNS Security Extensions). Enable it for protection against DNS spoofing.

## Implementation Checklist

- [ ] Domain DNS managed by Cloudflare
- [ ] A record for root domain pointing to server IP
- [ ] A record for www pointing to server IP (or CNAME to root)
- [ ] A record for staging pointing to staging server IP
- [ ] CNAME for assets subdomain pointing to R2 custom domain
- [ ] Cloudflare proxy enabled for web-facing records (orange cloud)
- [ ] Cloudflare proxy disabled for email records (grey cloud)
- [ ] Cloudflare SSL mode set to Full (Strict)
- [ ] TTL set to 300 seconds during setup, 3600 once stable
- [ ] DNSSEC enabled
- [ ] Email DNS records prepared (configured in Phase 12)

## Common Mistakes

- Not setting up DNS before deploying Caddy (Caddy can't provision SSL if the domain doesn't resolve to the server)
- High TTL during initial setup (changes take hours to propagate, debugging is slow)
- Cloudflare proxy ON for email records (email delivery breaks)
- Cloudflare SSL mode set to "Flexible" instead of "Full (Strict)" (traffic between Cloudflare and your server is unencrypted — security vulnerability)
- Exposing the server's real IP (with Cloudflare proxy OFF, attackers can bypass Cloudflare's DDoS protection by targeting the IP directly)
- Too many subdomains (each one is a maintenance burden — use path-based routing when possible)
- Forgetting to update DNS when migrating servers (old server IP in DNS after the new server is live)
