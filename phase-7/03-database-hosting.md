# Database Hosting

---

## What It Is

Your managed PostgreSQL database. Neon provides serverless Postgres with branching, connection pooling, autoscaling, and automatic backups — no database containers to manage.

**AI interface:** Neon CLI (`neonctl`), Neon MCP server.

## Provider

Use Neon for all database hosting. Serverless Postgres with built-in connection pooling, database branching for preview environments, and point-in-time recovery. Data stays in standard Postgres — no proprietary extensions or lock-in beyond the hosting layer.

## Architecture

```
App (Railway)
  │
  ▼
Neon Connection Pooler (built-in)
  │
  ▼
Neon Postgres (serverless)
  │
  ▼
Neon Storage (automatic, durable)
```

The app connects to Neon's pooled connection endpoint. No separate PgBouncer or connection pooler to manage.

## Connection Pooling

Neon provides built-in connection pooling via its proxy layer.

**Design decisions:**
- Use the pooled connection string (contains `-pooler` in the hostname) for all application connections
- The pooled endpoint uses transaction-mode pooling — connections are returned to the pool after each transaction
- Use the direct (non-pooled) connection string only for migrations and schema changes that require session-level features
- The Neon serverless driver (`@neondatabase/serverless`) is optimized for serverless environments — use it for all database access from the application

## Database Branching

Neon's killer feature for development workflows. A branch is a copy-on-write clone of your database at a specific point in time.

**Design decisions:**
- **Production:** the `main` branch — all production traffic hits this branch
- **Preview environments:** each Railway PR environment connects to its own Neon branch — created automatically or via CI. Neon Auth data branches with the database, so preview environments have working auth.
- **Development:** developers can create personal branches from `main` for local development
- Branches are cheap (copy-on-write, no full data duplication) — create liberally, delete when done
- Align branch lifecycle with PR lifecycle — create a branch when a PR is opened, delete it when the PR is merged or closed

## Extensions

**Design decisions:**
- Enable `pg_stat_statements` for query performance monitoring
- Enable `pg_trgm` for fuzzy search (covered in phase-3/08-search.md)
- Neon supports most standard Postgres extensions — enable them via SQL (`CREATE EXTENSION`)
- Extensions are per-branch — enable them in migrations so they're consistent across all branches

## Migrations

Schema changes are applied via migration files, same as with any Postgres host.

**Design decisions:**
- Migrations run as part of the deployment process — before the new app version starts accepting traffic
- Use the direct (non-pooled) connection string for migrations (some migrations need session-level features)
- Never run migrations manually — migrations must be version-controlled and reproducible
- Migration strategy (always roll forward, never edit deployed migrations) is covered in phase-3/01-database-schema.md
- Test migrations on a Neon branch first — branch from production, apply the migration, verify it works, then apply to production

## Backups & Disaster Recovery

Covered in detail in phase-7/03b-database-backups.md. Summary:
- Neon provides continuous backups via its copy-on-write architecture
- Point-in-time recovery by creating a branch at any timestamp within the retention window
- External `pg_dump` backups for defense-in-depth (weekly to Cloudflare R2)

## Monitoring

- Monitor query performance via `pg_stat_statements` — review slowest queries weekly
- Monitor connection count — if approaching the pooler limit, investigate connection leaks or scale the plan
- Neon dashboard shows compute usage, storage, and connection metrics
- Set up alerts for database unreachability via the health check endpoint (covered in 09-health-check.md)

## Scaling

**Design decisions:**
- Neon autoscales compute based on load — no manual scaling needed for most workloads
- Configure autoscaling bounds (min and max compute units) to control costs
- For high-traffic applications, Neon's read replicas can offload read queries — but start without replicas and add them only when measured latency or query volume demands it

## Edge Cases

- **Database branch out of sync** — a long-running PR's branch falls behind production. Reset the branch from the current production state before final testing.
- **Connection limit reached** — Neon's pooler has limits per plan. If the app creates too many connections, requests fail. Use the pooled endpoint and ensure the app doesn't hold connections open unnecessarily.
- **Migration fails on production but passed on branch** — the branch was created before recent production data changes. Always branch from the latest production state when testing migrations.
- **Neon outage** — external backups (weekly `pg_dump` to R2) are the insurance policy. Restore to a different Postgres provider if needed.
- **Autoscaling causes cold starts** — Neon's serverless compute can scale to zero. For production, set a minimum compute size to avoid cold start latency on the first request after idle.
- **Branch accumulation** — old branches from closed PRs accumulate. Automate branch cleanup in CI (delete the Neon branch when the PR is closed) or run periodic cleanup.

## Security Considerations

- The application should connect with the default role provided by Neon. For additional isolation, create a limited-privilege role for the application that only has permissions on application tables.
- Connection strings contain credentials. Never log them, include them in error reports, or expose them to the client. Store them in Railway environment variables and `.env` for local development.
- Neon encrypts data at rest and in transit. The connection uses SSL by default — never disable SSL in the connection string.
- Neon branches inherit the parent's data, including user data. Be aware that preview branches contain production data — if this is a concern, use seed data instead of branching from production.
- External backup files (`pg_dump`) contain the entire database. Encrypt before storing offsite.

## Implementation Checklist

- [ ] Neon project created in the correct region (matching Railway deployment region)
- [ ] Pooled connection string used for all application connections
- [ ] Direct connection string used for migrations only
- [ ] `@neondatabase/serverless` driver installed and configured
- [ ] `pg_stat_statements` enabled
- [ ] `pg_trgm` enabled
- [ ] Neon Auth configured (covered in phase-3/03-auth.md)
- [ ] Database branching strategy aligned with Railway preview environments
- [ ] Branch cleanup automated (delete branch when PR closes)
- [ ] Autoscaling bounds configured (min compute to avoid cold starts in production)
- [ ] External backup strategy defined (weekly `pg_dump` to R2)
- [ ] Backup restore tested at least once

## Common Mistakes

- Using the direct connection string for application traffic (bypasses connection pooling, hits connection limits)
- Not setting a minimum compute size in production (cold starts on first request after idle)
- Branches accumulating from old PRs (storage costs and clutter)
- Running migrations against the pooled endpoint (session-level features don't work in transaction-mode pooling)
- Not testing migrations on a branch first (applying untested schema changes to production)
- Ignoring autoscaling costs (set max compute bounds to prevent surprise bills)
- Branching from production for previews when the data is sensitive (preview branches contain real user data)
