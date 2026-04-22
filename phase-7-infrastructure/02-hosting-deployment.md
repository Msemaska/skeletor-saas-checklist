# Hosting & Deployment

---

## What It Is

How your application is deployed, served, and scaled in production. Railway handles containers, networking, SSL, preview environments, and rollback — you bring Dockerfiles and a GitHub repo.

## Provider

Use Railway for hosting. It deploys Docker containers from your repo, handles SSL and networking, provides preview environments per PR, and supports instant rollback. Unlike fully managed platforms, Railway still accepts raw containers — you control the Dockerfile, the build process, and the runtime environment.

**AI interface:** Railway CLI (`railway`), Railway MCP server (`railway-mcp-server`).

## Architecture

```
GitHub Repository
  │ (push to main)
  ▼
Railway Build
  │ (build from Dockerfile or Nixpacks)
  ▼
Railway Network
  │ (private networking, public domains, SSL)
  ▼
Users
```

Every push to main triggers an automatic deployment. Railway builds the Docker image, deploys it, and routes traffic — no CI/CD pipeline for deployment needed (GitHub Actions handles tests and linting, Railway handles deployment).

## Deployment

**Design decisions:**
- Connect the GitHub repository to Railway — deployments trigger automatically on push to main
- Every push to main is a production deployment
- Railway builds from your Dockerfile by default — if no Dockerfile exists, it uses Nixpacks (auto-detected). Prefer explicit Dockerfiles for control.
- Environment variables are set in the Railway dashboard or via CLI, not in the repository
- Each service (app, worker) is a separate Railway service within the same project

## SSL & Networking

Railway handles both automatically:

- All traffic is served over HTTPS — SSL certificates are provisioned and renewed automatically
- HTTP is automatically redirected to HTTPS
- Private networking between services (app ↔ worker ↔ database) uses Railway's internal network — no ports exposed to the internet
- Static assets are served through Railway's edge — for global CDN, put Cloudflare in front

No reverse proxy needed. Railway handles HTTPS termination, routing, and internal networking.

## Custom Domain

**Design decisions:**
- Add your custom domain in the Railway dashboard or via CLI
- Point your domain's DNS to Railway using a CNAME record (Railway provides the value)
- Add both the root domain (yourdomain.com) and www subdomain — configure a redirect from www to root
- SSL is provisioned automatically once DNS resolves to Railway
- For global CDN performance, proxy through Cloudflare (set Cloudflare to proxy mode, CNAME points to Railway)

## Preview Environments

Every pull request gets its own preview environment — a unique URL running the PR's code with separate environment variables.

**Design decisions:**
- Enable PR environments in Railway project settings — every PR gets an isolated deployment
- Configure separate environment variables for preview environments (test API keys for Stripe, Resend, etc.)
- Preview environments connect to Neon database branches (covered in 03-database-hosting.md) — each PR gets its own database branch with auth data included (Neon Auth branches with the database)
- Share preview URLs with stakeholders for review before merging
- Preview environments are automatically torn down when the PR is closed

## Rollback

If a deployment breaks production, roll back instantly.

**Design decisions:**
- Railway retains all previous deployments — any previous deployment can be promoted to production
- Rollback is near-instant (re-routes traffic to the previous deployment)
- After rollback, fix the issue in a new PR and re-deploy normally
- Database migrations complicate rollbacks — if the new schema is incompatible with the old code, you need a forward fix (covered in 05-cicd-pipeline.md)

## Scaling

**Design decisions:**
- Railway supports horizontal scaling (multiple replicas) and vertical scaling (more CPU/RAM per instance)
- Start with a single instance. Scale vertically first — increase the instance size before adding replicas.
- When horizontal scaling is needed, Railway distributes traffic across replicas automatically
- Set resource limits to prevent runaway costs — configure max replicas and instance size caps
- Railway's usage-based pricing means you pay for what you use — no idle server costs during low traffic

## Regions

**Design decisions:**
- Deploy services to the region closest to your database. If your Neon instance is in eu-west, deploy Railway services to eu-west.
- Mismatched regions (app in us-east, database in eu-west) add ~100ms latency to every database query
- Railway supports multiple regions — choose based on your primary user base

## Spending Limits

**Design decisions:**
- Set a usage limit in the Railway dashboard to prevent unexpected bills from traffic spikes
- Monitor usage (CPU, RAM, bandwidth, build minutes) in the Railway dashboard
- Railway's Trial plan is sufficient for development. Pro plan for production.

## What to Test

- Deployment triggers automatically on push to main
- Preview environments are created for every PR with a unique URL
- Custom domain resolves correctly with HTTPS
- Rollback to a previous deployment works and serves the old version
- Environment variables are correctly scoped (preview vs production)
- Build completes in a reasonable time (under 5 minutes)
- Private networking between services works (app can reach worker, database)

## Edge Cases

- **Build fails on Railway but passes locally** — usually caused by: different Node.js version (pin in Dockerfile), missing environment variables (set in Railway dashboard), or case-sensitive file imports (Railway runs Linux, Windows is case-insensitive).
- **Preview environment uses production environment variables** — configure separate environment variable scopes in Railway. Preview environments should use test API keys.
- **Custom domain SSL not provisioning** — DNS is not pointing to Railway yet, or DNS propagation is still in progress. Verify DNS records and wait up to the TTL period.
- **Traffic spike causes unexpected bill** — set usage limits in the Railway dashboard. For DDoS-level traffic, put Cloudflare in front of Railway for edge-level protection.
- **Need to deploy a hotfix while a PR is in review** — push the hotfix directly to main. It deploys immediately. The PR will need to be rebased after.
- **Service-to-service communication fails** — verify services are in the same Railway project and using private networking URLs (internal DNS), not public URLs.

## Implementation Checklist

- [ ] GitHub repository connected to Railway
- [ ] Railway project created with app service (and worker service if needed)
- [ ] Dockerfile verified and building correctly on Railway
- [ ] Custom domain added and DNS configured (CNAME to Railway)
- [ ] Cloudflare proxy enabled for CDN and DDoS protection (if needed)
- [ ] SSL verified (automatic)
- [ ] Environment variables set in Railway (production and preview scopes)
- [ ] Preview environments enabled for PRs
- [ ] Service region matches Neon database region
- [ ] Usage limits configured
- [ ] Rollback tested (promote a previous deployment)
- [ ] Private networking verified between services

## Common Mistakes

- Services in a different region than the database (adds 100ms+ to every query)
- No environment variable scoping (preview environments use production Stripe keys — charges real customers during testing)
- No usage limits (traffic spike or bot attack causes a surprise bill)
- Using Nixpacks when you should be using an explicit Dockerfile (less control, harder to debug build issues)
- Not testing rollback before you need it (discover the process during a production incident)
- Case-sensitive imports that work on Windows but fail on Railway's Linux build
- Using public URLs for service-to-service communication (adds latency, bypasses private network)
