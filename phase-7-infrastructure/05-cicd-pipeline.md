# CI/CD Pipeline

---

## What It Is

How code goes from a pull request to running in production. GitHub Actions handles CI (test, lint, build verification). Railway handles CD (build image, deploy, route traffic). This separation keeps CI fast and deployment simple.

## Pipeline Architecture

```
Developer pushes code
  │
  ▼
GitHub Actions (CI)
  │ 1. Run tests
  │ 2. Run linter
  │ 3. Type check
  ▼
Railway (CD) — on merge to main
  │ 4. Build Docker image from Dockerfile
  │ 5. Deploy new version
  │ 6. Health check
  │ 7. Route traffic to new version
  ▼
Production running new version
```

**AI interface:** GitHub CLI (`gh`), Railway CLI (`railway`), Railway MCP server.

## CI: Build Pipeline (GitHub Actions)

Runs on every pull request and every push to main.

**On pull requests:**
- Run the full test suite (unit, integration, component tests)
- Run the linter and type checker
- Do NOT deploy — PRs are for validation only
- Railway creates a preview environment automatically (covered in 02-hosting-deployment.md)

**On merge to main:**
- Run the full test suite (redundant with PR checks, but catches issues from merge conflicts)
- Railway auto-deploys from the main branch — no deployment step needed in GitHub Actions

**Design decisions:**
- Tests must pass before merge — branch protection rules on main require passing CI
- Fail fast — if tests fail, the PR can't merge
- Cache dependencies in GitHub Actions (`actions/cache` for node_modules) to speed up runs
- Keep CI under 5 minutes — slow CI discourages frequent commits

## CD: Deployment (Railway)

Railway handles deployment automatically. On every push to main, Railway:
1. Detects the code change
2. Builds a new Docker image from the Dockerfile
3. Runs the health check endpoint
4. Routes traffic to the new deployment
5. Keeps the previous deployment available for instant rollback

**Design decisions:**
- Deployment is automated — merging to main deploys to production. No manual "deploy" button.
- Database migrations must run before the new app starts. Use a Railway deploy command or a pre-deploy script in your Dockerfile. Migrations must be backward-compatible (the old code must work with the new schema during the transition).
- One deployment at a time — Railway queues deployments if multiple pushes happen in quick succession.

## Zero-Downtime Deployments

Railway performs rolling deployments by default — the new version is started and verified before the old version is stopped.

**Design decisions:**
- Railway starts the new container, waits for the health check to pass, then routes traffic to it
- The health check endpoint verifies the app can connect to the database and respond to requests — not just that the process is running
- Graceful shutdown: when the old container receives a stop signal, it finishes processing in-flight requests before exiting

## Rollback Strategy

If a deployment breaks production, rollback to the previous version immediately.

**Design decisions:**
- Railway retains previous deployments — rollback is instant via the dashboard or CLI (`railway rollback`)
- After rollback, the bad commit is investigated and fixed in a new PR. Never re-deploy the same broken code.
- Database migrations complicate rollbacks — if the new schema is incompatible with the old code, you need a forward fix (new migration that reverts the schema change), not a rollback. This is why migrations must be backward-compatible.

## Preview Environments

Railway creates a preview environment for every PR automatically (if enabled in project settings).

**Design decisions:**
- Each PR gets an isolated Railway environment with its own URL
- Preview environments connect to Neon database branches — each PR gets its own database branch (covered in 03-database-hosting.md)
- Preview environments use separate environment variables (test API keys)
- Share preview URLs with stakeholders for review before merging
- Preview environments are destroyed when the PR is closed

## Environment Variables

- Production and preview environments each have their own variables set in Railway
- Local development uses `.env` files
- `.env` files are never committed to the repository — they contain secrets
- `.env.example` is committed with all variable names (no values) so developers know what's needed
- When adding a new environment variable, update `.env.example` in the same PR and add it to Railway via dashboard or CLI

## Edge Cases

- **Deployment fails mid-way** — Railway detects the health check failure and keeps the old version running. The failed deployment is visible in the dashboard. Fix and redeploy.
- **Migration fails** — if migrations run as part of the deploy process and fail, the deployment fails. The old version keeps running with the old schema. Fix the migration and redeploy.
- **Two developers merge to main simultaneously** — Railway queues deployments. The second deployment runs after the first completes.
- **Need to rollback a migration** — write a new migration that reverts the schema change and deploy it as a forward fix. Never manually alter the production database.
- **Secrets rotated** — update the environment variable in Railway dashboard or CLI. Railway redeploys the service automatically when environment variables change.

## Security Considerations

- GitHub Actions secrets are not visible in logs, but a malicious or misconfigured workflow step can exfiltrate them. Only use trusted GitHub Actions (official or verified). Pin third-party actions to a specific commit SHA, not a tag (tags can be moved to point to malicious code).
- Branch protection on main is a security control, not just a workflow convenience. Without it, anyone with write access can push directly to main and trigger a production deployment — bypassing code review and CI checks.
- Treat the CI/CD configuration (`.github/workflows/`) as security-critical code — changes to workflow files should require the same review as changes to authentication or payment code.
- Never echo, print, or log environment variables or secrets in CI steps. A common mistake is adding debug output that accidentally includes credentials in the build log.
- Railway's API token (if used for CLI deployments) has production access. Store it as a GitHub Actions secret and treat it with the same care as database credentials.

## Implementation Checklist

### CI
- [ ] GitHub Actions workflow for pull requests (test + lint + type check)
- [ ] GitHub Actions workflow for main branch (test — Railway handles deployment)
- [ ] Branch protection on main (require passing CI)
- [ ] Dependency caching in GitHub Actions
- [ ] Third-party GitHub Actions pinned to commit SHA (not tag)

### CD
- [ ] Railway project connected to GitHub repository
- [ ] Auto-deploy from main branch enabled
- [ ] Database migration strategy defined (pre-deploy or startup script)
- [ ] Health check endpoint configured in Railway
- [ ] Preview environments enabled for PRs

### Rollback
- [ ] Rollback tested via Railway dashboard or CLI
- [ ] Previous deployments retained
- [ ] Team knows the rollback procedure

### Environment
- [ ] Environment variables set in Railway (production and preview scopes)
- [ ] `.env.example` committed and kept up to date
- [ ] Neon database branch strategy aligned with Railway preview environments

## Common Mistakes

- Manual deployments (error-prone, not reproducible, no audit trail)
- Migrations that break backward compatibility (old code can't work with new schema during rolling update)
- No health check in the deployment pipeline (deploying a broken image and not detecting it)
- No rollback plan (scrambling to fix forward when rolling back would be faster)
- Deploying on Friday afternoon (if something breaks, you're debugging on the weekend)
- `.env` committed to the repository (secrets leaked to anyone with repo access)
- Manual database changes outside of migrations (schema drift between environments)
- Not aligning Railway preview environments with Neon database branches (preview app hits production database)
