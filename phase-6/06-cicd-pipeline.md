# CI/CD Pipeline

---

## What It Is

How code goes from a pull request to running in production. This covers the build pipeline (test, build, push image), the deployment pipeline (pull image, run migrations, restart services), zero-downtime deployments, and rollback strategy.

## Pipeline Architecture

```
Developer pushes code
  │
  ▼
GitHub Actions (CI)
  │ 1. Run tests
  │ 2. Run linter
  │ 3. Build Docker image
  │ 4. Push image to registry
  ▼
GitHub Actions (CD) — on merge to main
  │ 5. SSH into production server
  │ 6. Pull new image
  │ 7. Run database migrations
  │ 8. Restart services with new image
  │ 9. Health check
  ▼
Production running new version
```

## CI: Build Pipeline

Runs on every pull request and every push to main.

**On pull requests:**
- Run the full test suite (unit, integration, component tests)
- Run the linter
- Build the Docker image (verify it builds successfully)
- Do NOT push the image to the registry or deploy — PRs are for validation only

**On merge to main:**
- Run the full test suite
- Build the Docker image
- Push the image to GitHub Container Registry tagged with the commit SHA
- Also tag the image as `latest`
- Trigger the deployment pipeline

**Design decisions:**
- Tests must pass before merge — branch protection rules on main require passing CI
- The build step uses the same Dockerfile as production — no separate "CI Dockerfile"
- Cache Docker layers in GitHub Actions to speed up builds (use `docker/build-push-action` with cache-to/cache-from)
- Fail fast — if tests fail, don't bother building the image

## CD: Deployment Pipeline

Runs after a successful build on main. Deploys to the production server.

**Deployment steps (in order):**
1. SSH into the production server (using a deployment SSH key stored as a GitHub Actions secret)
2. Pull the new image from the registry
3. Run database migrations (one-off container that runs migrations and exits)
4. Stop the old containers and start new ones with the new image
5. Wait for health checks to pass
6. If health checks fail, rollback (see below)

**Design decisions:**
- Deployment is automated — merging to main deploys to production. No manual "deploy" button.
- Migrations run before the new app starts — the database is updated first, then the app. This means migrations must be backward-compatible (the old code must work with the new schema during the transition).
- One deployment at a time — if a deployment is in progress and another merge happens, the second deployment queues until the first completes. Use GitHub Actions concurrency groups to enforce this.

## Zero-Downtime Deployments

The app should continue serving requests during deployment. Users should never see downtime for a code deployment.

**Design decisions:**
- Use Docker Compose's rolling update strategy: start the new container, wait for it to be healthy, then stop the old container
- Caddy continues routing to the old container until the new one passes its health check
- The health check endpoint verifies the app can connect to the database and respond to requests — not just that the process is running
- Graceful shutdown: when the old container receives a stop signal, it finishes processing in-flight requests (drain connections) before exiting. Set a stop timeout of 30 seconds.

## Rollback Strategy

If a deployment breaks production, rollback to the previous version immediately.

**Design decisions:**
- Every deployed image is tagged with its commit SHA in the registry — you can always pull a specific previous version
- Rollback is: SSH into the server, change the image tag in the compose file to the previous commit SHA, pull and restart
- This should be a single command or script — not a multi-step manual process
- After rollback, the bad commit is investigated and fixed in a new PR. Never re-deploy the same broken image.
- Database migrations complicate rollbacks — if the new schema is incompatible with the old code, you need a forward fix (new migration that reverts the schema change), not a rollback. This is why migrations must be backward-compatible.

## Staging Environment

**Design decisions:**
- A staging server mirrors production (same Docker setup, same compose file, different `.env`)
- Deploy to staging before production — staging runs the same CI/CD pipeline but triggers on a `staging` branch or a pre-release tag
- Staging uses a separate database with seed data, not a copy of production data
- Staging shares the same container registry — same images, different environment variables
- Test every deployment on staging first, verify it works, then merge to main for production deployment

## Environment Variables

- Production, staging, and local environments each have their own `.env` file
- `.env` files are never committed to the repository — they contain secrets
- `.env.example` is committed with all variable names (no values) so developers know what's needed
- On the server, `.env` is managed manually or via a secrets manager — it's not part of the deployment pipeline
- When adding a new environment variable, update `.env.example` in the same PR

## Edge Cases

- **Deployment fails mid-way** — if the new image fails to start (crash loop), the old containers are still running if using rolling updates. The deployment script should detect the health check failure and abort. If the old containers were already stopped, rollback immediately.
- **Migration fails** — migrations run before the app starts. If a migration fails, the deployment aborts before any new containers start. The old containers keep running with the old schema. Fix the migration and redeploy.
- **Two developers merge to main simultaneously** — the concurrency group ensures only one deployment runs at a time. The second deployment queues and runs after the first completes.
- **Server is unreachable during deployment** — SSH connection fails. The deployment pipeline fails and alerts the team. The old version is still running (nothing was changed). Investigate the server.
- **Need to rollback a migration** — write a new migration that reverts the schema change and deploy it as a forward fix. Never manually alter the production database.
- **Secrets rotated** — when rotating API keys or database credentials, update the `.env` file on the server and restart the affected services. No image rebuild needed.
- **Docker registry is down** — can't pull the new image. Deployment fails, old version keeps running. Retry when the registry is back.

## Implementation Checklist

### CI
- [ ] GitHub Actions workflow for pull requests (test + lint + build)
- [ ] GitHub Actions workflow for main branch (test + build + push + deploy)
- [ ] Branch protection on main (require passing CI)
- [ ] Docker layer caching in GitHub Actions
- [ ] Images tagged with commit SHA and `latest`

### CD
- [ ] Deployment SSH key stored as GitHub Actions secret
- [ ] Deployment script: pull image → run migrations → restart services → health check
- [ ] Concurrency group prevents simultaneous deployments
- [ ] Health check endpoint verifies database connectivity
- [ ] Graceful shutdown with 30-second drain timeout

### Rollback
- [ ] Rollback script/command that reverts to a specific commit SHA
- [ ] Previous images retained in registry for rollback
- [ ] Rollback tested at least once before launch

### Staging
- [ ] Staging server mirrors production setup
- [ ] Staging deploys before production
- [ ] Staging has separate database with seed data
- [ ] `.env.example` committed and kept up to date

## Common Mistakes

- Manual deployments via SSH (error-prone, not reproducible, no audit trail)
- No staging environment (deploying untested changes directly to production)
- Migrations that break backward compatibility (old code can't work with new schema during rolling update)
- No health check in the deployment pipeline (deploying a broken image and not detecting it)
- No rollback plan (scrambling to fix forward when rolling back would be faster)
- Deploying on Friday afternoon (if something breaks, you're debugging on the weekend)
- `.env` committed to the repository (secrets leaked to anyone with repo access)
- Simultaneous deployments overwriting each other (no concurrency control)
- Manual database changes outside of migrations (schema drift between environments)
