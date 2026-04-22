# Local Development

---

## What It Is

The developer's local environment — how to run the full stack on your machine for development, how to seed the database with test data, how to manage environment variables across environments, and how to handle graceful shutdown.

## Local Stack

The local development environment mirrors production as closely as possible using Docker Compose.

```
docker-compose.yml (local)
  ├── app           (Next.js in dev mode — hot reload)
  ├── postgres      (same version as production)
  ├── pgbouncer     (same connection pooling as production)
  ├── valkey        (if used in production)
  └── caddy         (optional locally — can use localhost:3000 directly)
```

**Design decisions:**
- The app runs in dev mode on the host machine — `npm run dev` for fast hot reload. The database, cache, and other services run in containers.
- PostgreSQL version in the local compose file must match production exactly — schema features and behavior can differ between major versions
- Valkey/Redis in local compose only if used in production — don't add services locally that don't exist in production

## Database Seeding

Seed data populates the local database with enough data to develop and test against. This is not sample data for users (covered in phase-5-ui-ux/06-onboarding.md) — this is developer data.

**Design decisions:**
- Seed script lives in the repository (e.g., `scripts/seed.ts` or `scripts/seed.sql`)
- The seed script is idempotent — running it twice doesn't create duplicate data. Check if data exists before inserting.
- Seed data includes: at least one user per role (owner, admin, member, viewer), at least one workspace, enough entities to test pagination (25+ items), edge case data (long names, unicode characters, entities at usage limits)
- Seed data uses deterministic IDs (hardcoded UUIDs, not random) so developers can reference specific entities in tests and during debugging
- The seed script runs as part of `docker compose up` for a fresh environment, or manually via `npm run seed`
- Never seed with production data — use synthetic data that covers all roles, states, and edge cases

## Environment Variables

**Three environments, three `.env` files:**
- `.env.local` — local development (localhost URLs, test API keys, debug mode)
- `.env.staging` — staging server (staging URLs, test API keys, production-like config)
- `.env.production` — production server (production URLs, live API keys)

**Design decisions:**
- `.env.example` is committed to the repository with all variable names and comments explaining each one — no values, just names
- `.env.local`, `.env.staging`, `.env.production` are in `.gitignore` — never committed
- All environment variables are validated at application startup using Zod (covered in the application's `lib/env.ts`). If a required variable is missing or invalid, the app fails to start with a clear error message.
- Naming convention: `NEXT_PUBLIC_` prefix for client-safe variables, no prefix for server-only variables
- Secrets (API keys, database credentials) are never logged, never included in error reports, never sent to the client

**Common variables every SaaS needs:**
- Database connection URL
- Auth provider credentials
- Stripe API keys (test mode for local/staging, live for production)
- Email provider API key (Resend)
- Storage provider credentials (R2 or MinIO)
- Sentry DSN
- App URL (http://localhost:3000 for local, https://staging.example.com, https://example.com)
- Node environment (development, production)

## Graceful Shutdown

When a container receives a stop signal (during deployment, restart, or shutdown), it must finish in-flight work before exiting.

**Design decisions:**
- The app process listens for SIGTERM (the signal Docker sends when stopping a container)
- On SIGTERM: stop accepting new requests, finish processing in-flight requests, close database connections, then exit
- Docker's stop timeout: 30 seconds. If the app hasn't exited after 30 seconds, Docker sends SIGKILL (forced termination).
- Background job workers: on SIGTERM, finish the currently running job, then exit. Don't start new jobs after receiving the signal.
- Database connections: close the connection pool cleanly to avoid lingering connections on the PostgreSQL side

**Why this matters:**
- Without graceful shutdown, in-flight requests get dropped — users see errors mid-action
- Database connections that aren't closed cleanly consume PostgreSQL's limited connection slots until they time out
- Background jobs killed mid-execution may leave data in an inconsistent state (half-processed)

## Developer Onboarding

A new developer should be able to run the project with minimal setup.

**Design decisions:**
- The README documents: prerequisites (Node.js version, Docker), setup steps (clone, copy .env.example, docker compose up, npm install, npm run seed, npm run dev), and common commands (run tests, run linter, access database)
- Maximum 5 commands from clone to running app. If it takes more, simplify the setup.
- Include a `Makefile` or npm scripts for common operations: `npm run dev`, `npm run seed`, `npm run test`, `npm run db:reset` (drop and re-seed the database)

## Edge Cases

- **Database port conflict** — another service on the developer's machine is using port 5432. Map the container to a different host port (e.g., 5433:5432) in the local compose file. Document this in the README.
- **Docker not installed** — the README should list Docker as a prerequisite. Provide a link to installation instructions.
- **Seed script fails on schema change** — if a migration changes the schema but the seed script hasn't been updated, seeding fails. The seed script must be updated alongside schema changes in the same PR.
- **Environment variable missing** — Zod validation at startup catches this immediately with a clear error message listing which variable is missing or invalid. No debugging needed.
- **Developer's machine runs out of disk** — Docker images, volumes, and build cache accumulate. Document `docker system prune` as a periodic cleanup step.
- **Hot reload not working** — if running the app inside a container, file system events may not propagate correctly (especially on macOS/Windows). Running `npm run dev` on the host avoids this issue.
- **Different Node.js version** — pin the Node.js version in `.nvmrc` or `package.json` engines field. Developers use `nvm use` to match the project's version.

## Security Considerations

- `.env.local`, `.env.staging`, and `.env.production` must be in `.gitignore`. A single accidental commit puts secrets in the git history permanently — even deleting the file in a later commit doesn't remove it from history.
- Never use production credentials in local development. Local `.env.local` should use test API keys, a local database, and sandbox payment credentials. A developer's machine is not a secured environment — production secrets on a laptop are a breach waiting to happen.
- Seed data must be synthetic, never production data. Copying production data to local machines violates user privacy, may breach data protection regulations (GDPR, etc.), and exposes real user data on unsecured developer machines.
- The local PostgreSQL container listens on a host port (e.g., 5432 or 5433) for developer tooling access. On shared networks, this means the database is accessible to other machines. Use a strong password even locally, and bind to `127.0.0.1` only in the compose file (`ports: "127.0.0.1:5432:5432"`).

## Implementation Checklist

- [ ] docker-compose.yml for local development (database, cache, and supporting services)
- [ ] PostgreSQL version matches production
- [ ] Seed script in repository (idempotent, covers all roles and edge cases)
- [ ] Seed script uses deterministic IDs
- [ ] `npm run seed` command available
- [ ] `.env.example` committed with all variable names and comments
- [ ] `.env.local` / `.env.staging` / `.env.production` in `.gitignore`
- [ ] Zod validation of all environment variables at startup
- [ ] Graceful shutdown handling for SIGTERM (app and workers)
- [ ] Docker stop timeout set to 30 seconds
- [ ] README with setup instructions (5 commands or fewer)
- [ ] `.nvmrc` pinning Node.js version
- [ ] npm scripts for common operations (dev, seed, test, db:reset)

## Common Mistakes

- Local PostgreSQL version doesn't match production (schema features differ between major versions)
- Non-idempotent seed script (running it twice creates duplicate data, breaks unique constraints)
- Random UUIDs in seed data (can't reference specific entities during development or testing)
- `.env` files committed to the repository (secrets in version control history forever)
- No environment variable validation (app starts, seems to work, then crashes when it tries to use a missing variable)
- No graceful shutdown (in-flight requests dropped, database connections leak, background jobs corrupted)
- 15-step setup process in the README (new developer spends a day getting the app running)
- Running the full app in Docker for local development (hot reload is slow, debugging is harder)
- Production data used for local development (privacy violation, stale data, different edge cases than synthetic data)
