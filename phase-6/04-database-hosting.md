# Database Hosting

---

## What It Is

Running and managing your own PostgreSQL instance instead of using a managed service like Supabase. This covers the database container, connection pooling, backup strategy, and disaster recovery.

## Architecture

```
App Container
  │
  ▼
PgBouncer (connection pooler)
  │
  ▼
PostgreSQL (database)
  │
  ▼
Named Volume (persistent data)
```

The app never connects to PostgreSQL directly in production. PgBouncer sits between them, managing a pool of connections.

## PostgreSQL Container

**Design decisions:**
- Use the official PostgreSQL Alpine image, pinned to the major version (e.g., postgres:16-alpine)
- Data stored on a named Docker volume — never a bind mount for production data
- Listen only on the Docker network — never expose PostgreSQL's port (5432) to the internet
- Set `max_connections` at the PostgreSQL level to a conservative number (100). PgBouncer handles the connection multiplexing.
- Enable `pg_stat_statements` for query performance monitoring
- Enable `pg_trgm` for fuzzy search (covered in phase-3/08-search.md)
- Set timezone to UTC

## Connection Pooling (PgBouncer)

PgBouncer maintains a pool of connections to PostgreSQL and multiplexes application requests across them.

**Design decisions:**
- PgBouncer runs as a separate container in the Docker Compose stack
- Pool mode: transaction pooling — connections are returned to the pool after each transaction. This is the most efficient mode for web applications.
- `default_pool_size`: 20 (number of connections PgBouncer maintains to PostgreSQL per database/user pair)
- `max_client_conn`: 200 (maximum simultaneous client connections PgBouncer accepts)
- The app connects to PgBouncer's port, not PostgreSQL's port directly
- PgBouncer is transparent to the application — the connection string just points to PgBouncer instead of PostgreSQL

## Backup Strategy

Backups are the single most important operational concern. Without tested backups, a disk failure means total data loss.

**Design decisions:**
- **Daily automated backups** using `pg_dump` via a cron job (or a dedicated backup container)
- Backups stored offsite — not on the same server as the database. Push to Cloudflare R2, AWS S3, or another storage provider.
- Retain: daily backups for 7 days, weekly backups for 4 weeks, monthly backups for 12 months
- Backup file naming: `backup-YYYY-MM-DD-HHmmss.sql.gz` (compressed with gzip)
- **Test restores regularly** — a backup you've never restored is not a backup. Test a restore to a local or staging environment at least monthly.
- For databases with high write volume or where point-in-time recovery matters, also enable WAL archiving (continuous archiving of write-ahead logs). This allows recovery to any specific point in time, not just the last daily backup.

**What to back up:**
- The full database (all schemas, all tables, roles, and extensions)
- Do NOT back up only specific tables — partial backups create restoration headaches with foreign key dependencies

## Migrations

Schema changes are applied via migration files, same as with a managed database. The only difference is how they're executed.

**Design decisions:**
- Migrations run as part of the deployment process — after pulling the new image, before starting the new containers
- Use a one-off container or an init script that runs migrations and exits before the app starts accepting traffic
- Never run migrations manually by SSH-ing into the server and pasting SQL — migrations must be version-controlled and reproducible
- Migration strategy (always roll forward, never edit deployed migrations) is covered in phase-3/01-database-schema.md

## Monitoring

- Monitor disk usage — databases grow. Set an alert when disk usage exceeds 80%.
- Monitor connection count — if PgBouncer's `max_client_conn` is consistently near the limit, increase it or investigate connection leaks
- Monitor query performance — enable `pg_stat_statements` and review the slowest queries weekly
- Monitor replication lag (if using replication) — high lag means the replica is falling behind and reads may be stale

## Edge Cases

- **Database container crashes and restarts** — named volumes persist. PostgreSQL recovers using its write-ahead log (WAL). Data is safe as long as the volume is intact.
- **Disk fills up** — PostgreSQL stops accepting writes and may crash. Monitor disk usage and set alerts at 80%. The biggest disk consumers are WAL files, pg_stat_statements data, and uncommitted temporary files.
- **Connection leak** — the app opens connections but never closes them. PgBouncer mitigates this by timing out idle connections (`server_idle_timeout`), but the root cause must be fixed in the application. Monitor connection counts.
- **Backup fails silently** — the backup cron job errors but nobody notices. Send the backup job's success/failure status to the monitoring system. If a backup hasn't completed in 24 hours, alert.
- **Restore to a different PostgreSQL version** — `pg_dump` output is forward-compatible but not always backward-compatible. Always restore to the same major version or newer.
- **Need to upgrade PostgreSQL major version** — dump the database on the old version, provision a new container with the new version, restore the dump. This requires downtime. Schedule during a maintenance window.
- **Corruption** — rare with modern PostgreSQL, but possible after a crash without proper shutdown. Enable `data_checksums` at database initialization to detect corruption early. Graceful shutdown handling (covered in 07-local-development.md) prevents most corruption.

## Implementation Checklist

- [ ] PostgreSQL container with pinned major version (Alpine image)
- [ ] Data on named Docker volume (not bind mount)
- [ ] PostgreSQL port not exposed to the internet (Docker network only)
- [ ] `max_connections` set conservatively (100)
- [ ] `pg_stat_statements` enabled
- [ ] `pg_trgm` enabled
- [ ] PgBouncer container configured with transaction pooling
- [ ] App connects to PgBouncer, not PostgreSQL directly
- [ ] Daily automated backups via `pg_dump` (compressed, stored offsite)
- [ ] Backup retention: 7 daily, 4 weekly, 12 monthly
- [ ] Backup restore tested at least monthly
- [ ] Disk usage alerts at 80%
- [ ] Connection count monitoring
- [ ] Migrations run as part of deployment (automated, not manual)

## Common Mistakes

- Exposing PostgreSQL port to the internet (direct database access from anywhere)
- No connection pooling (app opens 200 direct connections, PostgreSQL runs out of memory)
- Backups on the same server as the database (disk failure loses both the database and the backups)
- Never testing backup restores (discover the backup is corrupted when you actually need it)
- Bind mounts instead of named volumes (accidentally delete the host directory, lose all data)
- Running migrations manually via SSH (not reproducible, not tracked, can't be rolled back)
- No disk usage monitoring (database grows until the disk fills, PostgreSQL crashes)
- Upgrading PostgreSQL in-place (major version upgrades require dump and restore, not just pulling a new image)
