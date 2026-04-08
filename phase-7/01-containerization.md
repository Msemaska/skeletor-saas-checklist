# Containerization

---

## What It Is

How your application is packaged and run as Docker containers. Every service (app, workers, database, cache, reverse proxy) runs in its own container, managed together via Docker Compose.

## Container Architecture

```
docker-compose.yml
  ├── app           (Next.js / your application)
  ├── worker        (background job processor — Trigger.dev worker or BullMQ)
  ├── postgres      (database)
  ├── valkey        (cache / rate limiting / job queues — if needed)
  ├── caddy         (reverse proxy — covered in 02-reverse-proxy.md)
  └── storage       (MinIO or mount to R2 — if self-hosting object storage)
```

## Dockerfile Design

**One Dockerfile per service type.** The app and worker often share the same Dockerfile with different entrypoint commands.

**Design decisions:**
- Use multi-stage builds — build stage installs dependencies and compiles, production stage copies only what's needed. This keeps the final image small and excludes dev dependencies.
- Base image: use the official Node.js Alpine image for the smallest footprint
- Pin the Node.js version to the major.minor (e.g., node:22-alpine), not `latest`
- Copy `package.json` and `package-lock.json` first, run `npm ci`, then copy the rest of the source. This ensures dependency installation is cached separately from code changes.
- Run the application as a non-root user inside the container — create a user in the Dockerfile and switch to it before the entrypoint
- Set `NODE_ENV=production` in the production Dockerfile
- Expose only the port your app listens on — don't expose database or internal service ports

## Docker Compose

**Design decisions:**
- One `docker-compose.yml` for local development, one `docker-compose.prod.yml` for production
- All services on a shared Docker network — services reference each other by container name (e.g., `postgres:5432`, `valkey:6379`)
- Use named volumes for persistent data (database, object storage) — never bind mounts for production data
- Health checks on every service — Compose should know when a service is ready, not just running
- Restart policy: `unless-stopped` for production services
- Environment variables loaded from `.env` file — never hardcoded in the compose file

## Container Registry

Store built images in a container registry so they can be pulled during deployment.

- Use GitHub Container Registry (GHCR) — it's free for public repos and included with GitHub plans for private repos
- Tag images with the git commit SHA for traceability — every deployed image maps to a specific commit
- Also tag the latest successful build as `latest` for convenience
- Clean up old images periodically — registries accumulate storage over time

## Orchestration

**Single server:** Docker Compose is sufficient. Run `docker compose up -d` on the server. This handles 90% of MVP SaaS products.

**Multiple servers:** Docker Swarm for multi-node deployments. Swarm is built into Docker, requires no additional tooling, and handles service distribution across nodes.

**Design decisions:**
- Start with a single server and Docker Compose. Scale to Swarm only when you've measured that a single server can't handle the load.
- For a single server: all services run on one machine, Caddy handles routing, no orchestration complexity
- For Swarm: services are distributed across nodes, Swarm handles placement and failover, Caddy sits in front as a load balancer

## Edge Cases

- **Container runs out of memory** — set memory limits on containers in the compose file. A Next.js app that leaks memory will eventually kill the server if unconstrained. Set a reasonable limit (512MB for the app) and let Docker restart the container if it exceeds it.
- **Database container restarts and loses data** — this only happens if you're not using named volumes. Named volumes persist across container restarts and removals. Verify your volume configuration before going to production.
- **Build cache grows unbounded** — Docker build cache accumulates layers over time. Run `docker system prune` periodically on the build machine and production server to reclaim disk space.
- **Container startup order** — the app starts before the database is ready. Use health checks and `depends_on` with `condition: service_healthy` to ensure dependencies are ready before dependent services start.
- **Secrets in Docker images** — never put API keys or secrets in the Dockerfile or image layers. Pass them as environment variables at runtime via the compose file or Docker secrets.
- **Image size bloat** — a 2GB image takes forever to push and pull. Multi-stage builds, Alpine base images, and `.dockerignore` (exclude node_modules, .git, test files) keep images under 200MB.

## Implementation Checklist

- [ ] Dockerfile for app (multi-stage, Alpine, non-root user, pinned Node version)
- [ ] Dockerfile for worker (or shared with app, different entrypoint)
- [ ] `.dockerignore` excludes node_modules, .git, test files, docs
- [ ] docker-compose.yml for local development (all services)
- [ ] docker-compose.prod.yml for production (or override file)
- [ ] Shared Docker network for inter-service communication
- [ ] Named volumes for persistent data (database, storage)
- [ ] Health checks on every service
- [ ] Restart policy `unless-stopped` on production services
- [ ] Environment variables in `.env` file, not hardcoded
- [ ] Images pushed to GitHub Container Registry tagged with commit SHA
- [ ] Old images cleaned up periodically

## Common Mistakes

- Using `latest` tag for Node.js base image (breaks when a new major version is released)
- Running containers as root (security risk — compromised container has root access to the host)
- Bind mounts for production database data (data loss if the path changes or is accidentally deleted)
- No health checks (Compose doesn't know if the service is actually working, just that the process is running)
- Hardcoding secrets in the Dockerfile or compose file (secrets end up in image layers or version control)
- Not using multi-stage builds (production image includes dev dependencies, build tools, and source maps)
- No `.dockerignore` (copies node_modules and .git into the build context — slow builds, huge images)
- Starting with Kubernetes for an MVP
