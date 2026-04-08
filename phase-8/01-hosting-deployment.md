# Hosting & Deployment

---

## Provider

Use Vercel for hosting. It handles deployment, CDN, SSL, preview environments, and rollback with zero infrastructure management.

## Architecture

```
GitHub Repository
  │ (push to main)
  ▼
Vercel Build
  │ (build Next.js app)
  ▼
Vercel Edge Network
  │ (CDN, SSL, routing)
  ▼
Users
```

Every push to main triggers an automatic deployment. No CI/CD pipeline to configure — Vercel IS the pipeline.

## Deployment

**Design decisions:**
- Connect the GitHub repository to Vercel — deployments trigger automatically on push to main
- Every push to main is a production deployment
- Every pull request gets a preview deployment with a unique URL — use this for QA, design review, and stakeholder demos
- Build settings: Next.js is auto-detected. No custom build configuration needed.
- Environment variables are set in the Vercel dashboard, not in the repository

## CDN & SSL

Vercel handles both automatically:

- All traffic is served through Vercel's Edge Network (global CDN)
- SSL certificates are provisioned and renewed automatically for your custom domain
- HTTP is automatically redirected to HTTPS
- Static assets are cached at the edge with content-based hashing (immutable, long TTL)
- Server-rendered pages are served from the nearest edge region

No configuration needed. This works out of the box when you connect a custom domain.

## Custom Domain

**Design decisions:**
- Add your custom domain in the Vercel dashboard
- Point your domain's DNS to Vercel using a CNAME record (Vercel provides the value)
- Add both the root domain (yourdomain.com) and www subdomain — Vercel redirects www to root
- SSL is provisioned automatically once DNS resolves to Vercel

## Preview Deployments (Staging)

Every pull request gets its own preview deployment — a unique URL running the PR's code against separate preview environment variables.

**Design decisions:**
- Use preview deployments as the staging environment — no need for a separate staging server
- Configure separate environment variables for previews (test API keys for Stripe, Resend, etc.)
- Share preview URLs with stakeholders for review before merging
- Preview deployments are automatically deleted when the PR is closed

## Rollback

If a deployment breaks production, roll back instantly.

**Design decisions:**
- Vercel retains all previous deployments — any previous deployment can be promoted to production with one click
- Rollback is instant (no rebuild) — it re-routes traffic to the previous deployment's build output
- After rollback, fix the issue in a new PR and re-deploy normally
- Database migrations complicate rollbacks — if the new schema is incompatible with the old code, you need a forward fix (same as self-hosted, covered in phase-7/06-cicd-pipeline.md)

## Build Optimization

**Design decisions:**
- Enable Vercel's build cache — subsequent builds only rebuild what changed
- Keep build times under 3 minutes. If builds take longer, investigate: large dependencies, unoptimized images bundled at build time, or excessive static page generation.
- Use `next/image` for all images — Vercel optimizes and caches them at the edge
- ISR (Incremental Static Regeneration) for marketing pages — they're static at build time and revalidated on a schedule

## Regions

**Design decisions:**
- Deploy serverless functions to the region closest to your database. If your Supabase instance is in eu-west, deploy Vercel functions to eu-west.
- Mismatched regions (functions in us-east, database in eu-west) add ~100ms latency to every database query
- Static assets and edge functions are served globally regardless of the function region

## Spending Limits

**Design decisions:**
- Set a spending limit in the Vercel dashboard to prevent unexpected bills from traffic spikes
- Monitor usage (bandwidth, function invocations, build minutes) in the Vercel dashboard
- Vercel's free tier (Hobby) is sufficient for development. Pro plan for production.

## What to Test

- Deployment triggers automatically on push to main
- Preview deployments are created for every PR with a unique URL
- Custom domain resolves correctly with HTTPS
- Rollback to a previous deployment works and serves the old version
- Environment variables are correctly set (test vs production keys)
- Build completes in under 3 minutes
- Static assets are served from CDN (check cache headers)

## Edge Cases

- **Build fails on Vercel but passes locally** — usually caused by: different Node.js version (pin in package.json engines), missing environment variables (set in Vercel dashboard), or case-sensitive file imports (Vercel runs Linux, Windows is case-insensitive).
- **Preview deployment uses production environment variables** — configure separate environment variable scopes in Vercel: Production, Preview, and Development. Preview deployments should use test API keys.
- **Custom domain SSL not provisioning** — DNS is not pointing to Vercel yet, or DNS propagation is still in progress. Verify DNS records and wait up to the TTL period.
- **Traffic spike causes unexpected bill** — set spending limits in the Vercel dashboard. For DDoS-level traffic, Vercel's edge network absorbs it, but function invocations still count. Consider Cloudflare in front of Vercel for additional DDoS protection.
- **Need to deploy a hotfix while a PR is in review** — push the hotfix directly to main. It deploys immediately. The PR will need to be rebased after.

## Implementation Checklist

- [ ] GitHub repository connected to Vercel
- [ ] Custom domain added and DNS configured
- [ ] SSL verified (automatic)
- [ ] Environment variables set in Vercel dashboard (Production, Preview, Development scopes)
- [ ] Preview deployments working for PRs
- [ ] Serverless function region matches database region
- [ ] Spending limit configured
- [ ] Build time under 3 minutes
- [ ] Rollback tested (promote a previous deployment)
- [ ] ISR configured for marketing pages

## Common Mistakes

- Serverless functions in a different region than the database (adds 100ms+ to every query)
- No environment variable scoping (preview deployments use production Stripe keys — charges real customers during testing)
- No spending limit (traffic spike or bot attack causes a surprise bill)
- Building the entire app on every push when ISR could handle marketing pages statically
- Not testing rollback before you need it (discover the process during a production incident)
- Case-sensitive imports that work on Windows but fail on Vercel's Linux build
