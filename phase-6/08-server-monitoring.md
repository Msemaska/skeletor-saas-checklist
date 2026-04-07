# Server Monitoring & Log Rotation

---

## What It Is

How you know your server is healthy and how you prevent logs and temporary files from filling the disk. Application-level monitoring (errors, analytics, uptime) is covered in Phase 9 Observability. This covers server-level infrastructure monitoring.

## What to Monitor

| Metric | Warning Threshold | Critical Threshold |
|---|---|---|
| CPU usage | 70% sustained for 5 minutes | 90% sustained for 5 minutes |
| Memory usage | 80% | 95% |
| Disk usage | 80% | 90% |
| Disk I/O | Sustained high latency | Disk unresponsive |
| Container status | Container restarted | Container in crash loop |

**Design decisions:**
- Use Betterstack for server monitoring — it provides server metrics, log aggregation, and alerting in one platform (aligns with Phase 9 Observability)
- Install the Betterstack agent on the server — it collects CPU, memory, disk, and network metrics automatically
- Alerts go to: email for warnings, Slack/Discord + email for critical
- Check interval: every 60 seconds

## Container Monitoring

Beyond server-level metrics, monitor the health of individual containers:

- **Container restarts** — a container that restarts frequently has a bug (crash loop) or a resource issue (out of memory)
- **Container resource usage** — which container is consuming the most CPU/memory? Is it the app, the database, or the worker?
- **Health check status** — containers that fail health checks are not serving traffic correctly even if they're running

Docker's built-in health checks and restart counts are available via the Docker API. Betterstack or a simple monitoring script can poll these.

## Log Rotation

Docker containers generate logs. Without rotation, logs fill the disk.

**Design decisions:**
- Configure Docker daemon-level log rotation — this applies to all containers automatically
- Max log file size: 10MB per container
- Max log files retained: 3 per container (30MB total per container)
- This is configured in `/etc/docker/daemon.json` on the server — set it once during server provisioning

**Application logs vs container logs:**
- Container logs (stdout/stderr) are rotated by Docker's log driver
- Application logs (Pino structured JSON) should be shipped to a log aggregation service (Betterstack Logs), not written to disk. Write to stdout, let Docker capture it, and ship to Betterstack via the agent.
- Never write application logs to files inside the container — containers are ephemeral, logs disappear when the container is replaced

## Disk Cleanup

Beyond log rotation, other things accumulate on the server:

- **Docker images** — old images from previous deployments. Run `docker image prune -f` weekly via cron to remove unused images.
- **Docker build cache** — layers from previous builds. Run `docker builder prune -f` weekly.
- **Docker volumes** — orphaned volumes from removed containers. Run `docker volume prune -f` cautiously — only when you've verified no important data is in unnamed volumes.
- **System logs** — `/var/log` grows over time. The server's logrotate handles system logs by default, but verify it's configured.

**Design decisions:**
- Schedule a weekly cleanup cron job on the server: `docker image prune -f && docker builder prune -f` (cleans images and build cache without touching volumes)

## Alerting

**Design decisions:**
- Warnings (70% CPU, 80% disk) notify via email — these are not emergencies but need attention within hours
- Critical alerts (90% CPU, 90% disk, container crash loop) notify via Slack/Discord AND email — these need immediate attention
- All alerts include: what's wrong, which server, current metric value, threshold that was crossed
- Acknowledge alerts to stop repeat notifications — don't send the same alert every 60 seconds

## Edge Cases

- **Alert fatigue** — too many non-critical alerts and the team starts ignoring them. Set thresholds that only fire when action is actually needed. 70% CPU for 5 minutes is real. 50% CPU for 30 seconds is noise.
- **Disk fills up overnight** — a bug causes excessive logging or a backup fills the disk while the team is asleep. The 90% disk critical alert fires, team sees it in the morning. Having log rotation and Docker cleanup prevents most cases.
- **Monitoring agent itself crashes** — if the Betterstack agent crashes, the server appears offline in the dashboard. Betterstack detects "agent stopped reporting" and alerts as a separate condition.
- **Server time drift** — if the server's clock drifts, log timestamps become unreliable and cron jobs run at wrong times. Enable NTP (Network Time Protocol) — Ubuntu's `systemd-timesyncd` handles this by default.

## Implementation Checklist

- [ ] Betterstack agent installed on the server
- [ ] CPU, memory, and disk monitoring configured with thresholds
- [ ] Container health check monitoring configured
- [ ] Alert channels set up (email for warnings, Slack/Discord + email for critical)
- [ ] Docker log rotation configured in daemon.json (10MB max, 3 files retained)
- [ ] Application logs written to stdout (shipped to Betterstack Logs via agent)
- [ ] Weekly cleanup cron job for Docker images and build cache
- [ ] System logrotate verified as configured
- [ ] NTP enabled for accurate server time

## Common Mistakes

- No server monitoring (server runs out of disk at 3am, nobody knows until users report errors)
- Docker logs filling the disk (no log rotation configured — a chatty container fills 100GB in a week)
- Writing application logs to files inside containers (logs lost on container replacement, disk fills up)
- Alert thresholds too sensitive (50% CPU triggers an alert, team ignores all alerts)
- No disk cleanup job (old Docker images accumulate indefinitely)
- Using `docker volume prune` carelessly (accidentally deletes a database volume that wasn't named properly)
- Monitoring only the app, not the server (app is healthy but the server is running out of memory because of a different process)
