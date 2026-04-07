# Server Provisioning & Security

---

## What It Is

Setting up and securing the server(s) that run your application. This covers choosing a server, hardening it against attacks, and automating the setup so it's repeatable.

## Server Provider

Use Hetzner for production servers. Best price-to-performance ratio for European and US-based deployments, straightforward VPS provisioning, and reliable infrastructure.

**Choosing a server size:**
- Start with the smallest server that runs your stack comfortably. For a typical SaaS (app + database + cache + reverse proxy), a 4 vCPU / 8GB RAM server handles thousands of concurrent users.
- Scale up (bigger server) before scaling out (more servers). Vertical scaling is simpler and has no orchestration overhead.
- Monitor resource usage after launch — scale when CPU or memory consistently exceeds 70% utilization.

## Operating System

Ubuntu 24.04 LTS. Long-term support, widest ecosystem of guides and packages, most Docker tutorials assume Ubuntu.

## Security Hardening

Every production server must be hardened before deploying any application.

**SSH access:**
- Disable root login via SSH — create a non-root user with sudo privileges
- Disable password authentication — SSH key only
- Change the default SSH port from 22 to a non-standard port (e.g., 2222) — reduces automated brute force noise
- Use SSH keys generated with Ed25519 (stronger and shorter than RSA)

**Firewall (UFW):**
- Default deny all incoming traffic
- Allow only: SSH (custom port), HTTP (80), HTTPS (443)
- No other ports exposed to the internet — all internal services communicate via Docker network
- Enable UFW before deploying any services

**Automatic security updates:**
- Enable unattended-upgrades for security patches — the server applies critical security updates automatically without manual intervention
- Reboot automatically if required by a kernel update (configure unattended-upgrades to allow automatic reboots during maintenance windows)

**fail2ban:**
- Install and configure fail2ban to block IPs that repeatedly fail SSH login attempts
- Default: ban after 5 failed attempts for 10 minutes
- This is supplementary to SSH key-only auth — belt and suspenders

## User Management

- One non-root user for SSH access and server administration
- Docker runs as root (required), but application containers run as non-root users (configured in the Dockerfile — covered in 01-containerization.md)
- For team access: each team member gets their own SSH key added to the server's authorized_keys. Remove keys when team members leave.

## Automated Server Setup

The initial server setup should be scripted so it's repeatable for new servers, staging environments, and disaster recovery.

**What the setup script does:**
1. Update system packages
2. Create non-root user with sudo
3. Configure SSH (disable root, disable password, change port)
4. Install and configure UFW
5. Install fail2ban
6. Enable unattended-upgrades
7. Install Docker and Docker Compose
8. Configure Docker to start on boot
9. Create the application directory structure
10. Pull and start the application stack

**Design decisions:**
- Use a bash script for simple setups (single server). Move to Ansible when managing more than 2 servers.
- Store the setup script in the repository so it's version controlled
- The script is idempotent — running it twice doesn't break anything
- Don't include secrets in the script — secrets are added manually or via a secrets manager after setup

## Directory Structure on Server

```
/opt/app/
  ├── docker-compose.prod.yml
  ├── .env
  ├── caddy/
  │   └── Caddyfile
  └── data/
      ├── postgres/    (named volume mount)
      ├── caddy/       (certificate storage)
      └── valkey/      (cache persistence)
```

Application code is in the Docker image, not on the server filesystem. The server only has the compose file, environment variables, Caddy config, and persistent data volumes.

## Edge Cases

- **Server becomes unresponsive** — if you can't SSH in, use Hetzner's web console to access the server and diagnose. This is the fallback when SSH is locked out.
- **SSH key lost** — if you lose your SSH key and can't access the server, Hetzner's rescue mode boots the server into a recovery environment where you can mount the disk and add a new SSH key.
- **Disk full** — Docker logs and images accumulate. Set up a cron job to run `docker system prune -f` weekly and configure Docker's log rotation (max-size and max-file in the Docker daemon config).
- **Server compromised** — if you suspect a compromise, take the server offline immediately (Hetzner's firewall can block all traffic). Provision a new server from the setup script, restore data from backups, and investigate the old server offline.
- **Need to scale to a second server** — provision a new server with the same setup script, move the database to its own server (or use a managed database), and use Docker Swarm to distribute the app containers across both servers. Caddy load balances between them.
- **Server provider outage** — Hetzner outages are rare but happen. Your disaster recovery plan (Phase 9 Observability alerting) should detect downtime. Recovery means provisioning at a different provider using the same setup script and restoring from backups.

## Implementation Checklist

- [ ] Server provisioned on Hetzner (appropriate size for the stack)
- [ ] Ubuntu 24.04 LTS installed
- [ ] Non-root user created with sudo privileges
- [ ] SSH key-only authentication (password auth disabled)
- [ ] SSH port changed from default 22
- [ ] UFW configured (deny all, allow SSH + HTTP + HTTPS only)
- [ ] fail2ban installed and configured
- [ ] Unattended-upgrades enabled for security patches
- [ ] Docker and Docker Compose installed
- [ ] Docker configured to start on boot
- [ ] Setup script stored in repository and tested
- [ ] Application directory structure created
- [ ] Docker log rotation configured

## Common Mistakes

- Running as root (compromised container gets root access to the host)
- Password authentication enabled on SSH (brute force attacks will eventually succeed)
- Default SSH port 22 (exponentially more brute force noise)
- No firewall (all ports exposed to the internet, internal services accessible publicly)
- No automatic security updates (known vulnerabilities sit unpatched for months)
- Manual server setup without a script (can't reproduce the environment, disaster recovery is guesswork)
- Secrets in the setup script or repository (leaked credentials)
- Docker logs filling the disk (no log rotation configured, server runs out of space)
- Scaling out before scaling up (adding servers adds orchestration complexity — make the single server bigger first)
