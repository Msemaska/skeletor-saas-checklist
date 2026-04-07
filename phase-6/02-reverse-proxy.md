# Reverse Proxy & SSL

---

## What It Is

The service that sits in front of your application, handles HTTPS termination, routes traffic to the correct container, and provides automatic SSL certificates. Use Caddy.

## Architecture

```
Internet
  │
  ▼
Caddy (port 443/80)
  │ ├── yourdomain.com → app container (port 3000)
  │ ├── api.yourdomain.com → app container (port 3000)
  │ └── storage.yourdomain.com → MinIO container (port 9000)
  ▼
Docker Network (internal)
```

Caddy is the only container that exposes ports to the internet. All other containers are only accessible within the Docker network.

## SSL / HTTPS

Caddy provisions and renews SSL certificates automatically via Let's Encrypt. No configuration needed beyond pointing your domain to the server.

**Design decisions:**
- All traffic is HTTPS — Caddy automatically redirects HTTP to HTTPS
- Certificates are provisioned on first request and renewed automatically before expiry
- Certificate storage is on a named Docker volume so certificates persist across container restarts
- For wildcard subdomains (multi-tenant with customer subdomains), use Caddy's DNS challenge with your DNS provider's API credentials

## Domain Routing

**Design decisions:**
- Primary domain routes to the app container
- Subdomains route to specific services when needed (e.g., `storage.yourdomain.com` for object storage)
- Path-based routing within the app is handled by Next.js, not Caddy — Caddy just forwards everything for the domain to the app container
- API routes are served from the same domain as the app (no separate API subdomain) — Next.js handles `/api/*` routing internally

## Load Balancing

**Single server (default):** no load balancing needed. Caddy forwards directly to the single app container.

**Multiple servers:** Caddy can load balance across multiple app container instances using round-robin. Scale the app container replicas in Docker Compose or Swarm, and Caddy distributes traffic across them.

## Security Headers

Caddy should add security headers that protect against common attacks. These supplement the application-level headers set in Next.js.

**Headers to set in Caddy:**
- `Strict-Transport-Security` (HSTS) — enforces HTTPS for all future requests
- `X-Content-Type-Options: nosniff` — prevents MIME type sniffing
- `X-Frame-Options: DENY` — prevents clickjacking via iframes
- `Referrer-Policy: strict-origin-when-cross-origin` — controls referrer information sent with requests

CSP headers are set in the Next.js application config (covered in Phase 7 Infrastructure & Security), not in Caddy — CSP is application-specific and changes with your code.

## Configuration Reloading

Caddy supports zero-downtime configuration reloads via its admin API. When you change the Caddyfile, reload Caddy without restarting the container — existing connections are not dropped.

## Edge Cases

- **Let's Encrypt rate limits** — Let's Encrypt limits certificate issuance to 50 per week per domain. In development/staging, use Caddy's internal self-signed certificates to avoid hitting rate limits.
- **Certificate renewal fails** — Caddy retries automatically with exponential backoff. If renewal fails repeatedly (DNS misconfigured, port 80 blocked), Caddy logs the failure. Monitor Caddy logs for certificate errors.
- **Domain DNS not yet propagated** — Caddy can't provision a certificate if the domain doesn't resolve to the server yet. Point DNS first, then start Caddy.
- **Multiple services on same port** — all services are internal to the Docker network. Only Caddy binds to ports 80/443 on the host. Internal services use their own ports without conflict.
- **WebSocket connections** — Caddy proxies WebSocket connections automatically. No special configuration needed for real-time features.
- **Large file uploads** — increase Caddy's request body size limit for routes that accept file uploads. The default may be too small for large files.

## Implementation Checklist

- [ ] Caddy container in docker-compose with ports 80 and 443 exposed
- [ ] Caddy certificate storage on a named Docker volume
- [ ] Primary domain routed to app container
- [ ] HTTP → HTTPS redirect (automatic with Caddy)
- [ ] Security headers configured (HSTS, X-Content-Type-Options, X-Frame-Options, Referrer-Policy)
- [ ] Request body size limit increased for file upload routes
- [ ] Staging/dev uses internal certificates (not Let's Encrypt)
- [ ] Caddy logs monitored for certificate errors
- [ ] WebSocket proxying verified for real-time features

## Common Mistakes

- Exposing app container ports directly to the internet (bypasses SSL, security headers, and access control)
- Using self-signed certificates in production (browsers show security warnings, users leave)
- Not persisting Caddy's certificate storage volume (certificates re-issued on every container restart, hits rate limits)
- Setting CSP headers in Caddy instead of Next.js (CSP is application-specific, changes with code deployments)
- Using Nginx instead of Caddy
- Forgetting to increase request body size for file upload routes (large uploads fail silently)
- Testing with Let's Encrypt in development (burns through rate limits)
