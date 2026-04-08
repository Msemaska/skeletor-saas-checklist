# Redis / Valkey

---

## What It Is

An in-memory data store used for caching, rate limiting, session storage, and job queues when your product needs sub-millisecond reads or shared state across serverless-style workers. Use Valkey — it's the open-source fork of Redis after Redis changed its license in 2024. API-compatible, community-maintained, same performance.

## When You Need This

This is not part of the default stack. Add Valkey only when you hit a specific need that Postgres and application-level caching can't solve efficiently.

**You need Valkey when:**
- Running a public API with per-key rate limiting across distributed workers
- Using BullMQ or similar for job queues (Trigger.dev/Inngest are the default — Valkey is only needed if self-hosting the queue)
- Caching expensive computed values that are read thousands of times per second (leaderboards, real-time counters)
- Storing ephemeral shared state between multiple app instances (e.g., multiple workers coordinating)

**You don't need Valkey when:**
- Next.js built-in cache and TanStack Query handle your caching needs (covered in phase-3/05-cache.md)
- Rate limiting is only on auth endpoints at moderate volume (database-backed counters are sufficient)
- You're running a single app instance (no shared state needed)

## Architecture

```
App Container ──→ Valkey Container (port 6379, Docker network only)
Worker Container ──→ Valkey Container
```

Valkey runs as a container in the Docker Compose stack, accessible only on the Docker network. Never expose Valkey's port to the internet.

## Configuration

**Design decisions:**
- Use the official Valkey Alpine image, pinned to the major version
- Set a memory limit (`maxmemory`) appropriate to your server (256MB is a reasonable starting point for an MVP)
- Eviction policy: `allkeys-lru` (when memory is full, evict the least recently used key). This is correct for cache-style usage.
- Persistence: enable RDB snapshots for data that should survive a restart (job queue state). Disable persistence entirely if using Valkey purely as a cache (data can be regenerated).
- Require a password (set via environment variable) even though it's only accessible on the Docker network — defense in depth

## Data Patterns

**Caching:**
- Key format: `cache:{entity}:{id}` (e.g., `cache:workspace:abc123`)
- Always set a TTL (expiry) on cache keys — never let cache entries live forever
- Default TTL: 5 minutes for most cached data. Adjust per use case.
- The application checks the cache first, falls back to the database on miss, and populates the cache after the database read (cache-aside pattern)

**Rate limiting:**
- Key format: `ratelimit:{identifier}:{window}` (e.g., `ratelimit:ip:192.168.1.1:60`)
- Use atomic increment with TTL — increment the counter and set the expiry in a single operation to prevent race conditions
- Counter expires at the end of the window (e.g., 60 seconds for a per-minute limit)

**Job queues (if using BullMQ):**
- Valkey stores the queue, job data, and job state
- Persistence must be enabled — a Valkey restart without persistence loses all pending and in-progress jobs
- Monitor queue depth — if jobs are accumulating faster than workers process them, add more workers or investigate slow jobs

## Edge Cases

- **Valkey is down** — the application must handle Valkey being unavailable gracefully. For caching: fall back to the database (slower but functional). For rate limiting: fail open (allow the request) rather than fail closed (block all requests). For job queues: jobs can't be enqueued — show an error and retry.
- **Memory full** — with `allkeys-lru`, Valkey evicts old keys to make room for new ones. This is fine for caching but dangerous for job queues (pending jobs could be evicted). If using Valkey for both caching and queues, use separate Valkey instances or separate key namespaces with different eviction policies.
- **Data loss on restart** — without persistence, all data is lost on restart. For caches this is fine (data is regenerated from the database). For job queues, enable RDB persistence.
- **Connection leak** — the app opens connections to Valkey but doesn't close them. Use a connection pool with a maximum connection limit. Monitor active connections.
- **Key naming collision** — two different features use the same key format and overwrite each other's data. Use namespaced key prefixes (`cache:`, `ratelimit:`, `queue:`) to prevent collisions.

## Implementation Checklist

- [ ] Valkey container in Docker Compose (Alpine image, pinned version)
- [ ] Port not exposed to the internet (Docker network only)
- [ ] Password set via environment variable
- [ ] Memory limit configured (`maxmemory`)
- [ ] Eviction policy set to `allkeys-lru`
- [ ] Persistence configured appropriately (RDB for queues, disabled for pure cache)
- [ ] Key naming convention established (namespaced prefixes)
- [ ] TTL set on all cache keys (never indefinite)
- [ ] Application handles Valkey being unavailable (fallback to database for cache, fail open for rate limiting)
- [ ] Connection pool configured with maximum limit
- [ ] Data stored on a named Docker volume (if persistence is enabled)

## Common Mistakes

- Adding Valkey to the stack before it's needed (unnecessary complexity — Postgres and Next.js cache handle most cases)
- Exposing Valkey's port to the internet (no auth by default in older versions, remote code execution risk)
- No password set (even on Docker network, defense in depth)
- Cache keys without TTL (memory fills up with stale data that's never evicted)
- Using Valkey for both caching and job queues with the same eviction policy (LRU eviction drops pending jobs)
- No fallback when Valkey is unavailable (app crashes instead of falling back to database)
- No persistence for job queue data (Valkey restart loses all pending jobs)
- Storing large values (>1MB) in Valkey (it's designed for small, fast reads — large values degrade performance)
