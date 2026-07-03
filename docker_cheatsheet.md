# Docker Cheatsheet

> Containers, images, volumes, networks, and Compose. On a VPS the priorities are: containers that restart themselves, ports that don't leak past the firewall, and disks that don't silently fill with logs.

---

## Table of Contents
- [Images](#images)
- [Containers](#containers)
- [Interacting with Containers](#interacting-with-containers)
- [Volumes](#volumes)
- [Networks](#networks)
- [Docker Compose](#docker-compose)
- [Cleanup & Disk Management](#cleanup--disk-management)
- [Dockerfile Reference](#dockerfile-reference)
- [Security on a VPS](#security-on-a-vps)
- [Tips & Gotchas](#tips--gotchas)

---

## IMAGES

```text
docker images                        → List local images
docker pull image:tag                → Download image from registry
docker build -t name:tag .           → Build image from Dockerfile
docker build --no-cache -t name:tag . → Build without cache
docker rmi image_id                  → Remove image
docker rmi $(docker images -q)       → Remove all images
docker tag old_name new_name:tag     → Rename/tag an image
docker image prune                   → Remove dangling images
docker image prune -a                → Remove ALL unused images
docker history image:tag             → Layer-by-layer size breakdown
```

---

## CONTAINERS

```text
docker ps                            → List running containers
docker ps -a                         → List all containers (inc. stopped)
docker run image                     → Run container (foreground)
docker run -d image                  → Run detached (background)
docker run -it image bash            → Run interactive shell
docker run --name myapp -d image     → Run with custom name
docker run -p 8080:80 image          → Map host:container ports (PUBLIC — see Security)
docker run -p 127.0.0.1:8080:80 image → Map to loopback only (front with nginx)
docker run -v /host/path:/container image → Bind mount
docker run --rm image                → Auto-remove on exit
docker run --env-file .env image     → Load env vars from file
docker run -d --restart unless-stopped image → Survive crashes AND reboots
docker run --memory 512m --cpus 1.5 image    → Resource caps
docker stop container_id             → Graceful stop (SIGTERM, then SIGKILL)
docker kill container_id             → Force stop
docker start container_id            → Start stopped container
docker restart container_id          → Restart container
docker rm container_id               → Remove stopped container
docker rm -f container_id            → Force remove running container
docker rm $(docker ps -aq)           → Remove all stopped containers
docker container prune               → Same, but cleaner (with confirm)
docker port container_id             → Show port mappings
```

```text
Restart policies: no (default) | on-failure[:N] | always | unless-stopped
💡 For long-lived agents: unless-stopped — like `always` but respects a
manual `docker stop` across daemon restarts.
```

---

## INTERACTING WITH CONTAINERS

```text
docker exec -it container_id bash    → Shell into running container
docker exec -it container_id sh      → Alpine images have no bash — use sh
docker exec container_id command     → Run command in container
docker logs container_id             → View logs
docker logs -f container_id          → Follow (tail) logs
docker logs --tail 100 container_id  → Last 100 lines
docker logs --since 1h container_id  → Last hour only
docker cp file.txt container_id:/path → Copy file into container
docker cp container_id:/path file.txt → Copy file out of container
docker inspect container_id          → Full container metadata (JSON)
docker stats                         → Live CPU/mem usage
docker stats --no-stream             → One-shot snapshot (script-friendly)
docker top container_id              → Running processes inside container
docker diff container_id             → Files changed since container start
```

### Targeted inspect (skip the JSON wall)

```bash
docker inspect -f '{{.State.Status}}' myapp
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' myapp
docker inspect -f '{{.LogPath}}' myapp        # Where the log file actually lives
```

---

## VOLUMES

```text
docker volume ls                     → List volumes
docker volume create my_vol          → Create named volume
docker volume inspect my_vol         → Volume details (inc. host path)
docker volume rm my_vol              → Remove volume
docker volume prune                  → Remove unused volumes
docker run -v my_vol:/app/data image → Attach named volume
docker run -v /host/path:/container:ro image → Bind mount, read-only
```

```text
Named volume (my_vol:/data)  → Docker-managed, survives container removal. Data.
Bind mount (/host/x:/data)   → Exact host path. Config files, dev source trees.
```

### Backup / restore a named volume

```bash
docker run --rm -v my_vol:/data -v "$(pwd)":/backup ubuntu \
  tar czf /backup/my_vol.tar.gz -C /data .

docker run --rm -v my_vol:/data -v "$(pwd)":/backup ubuntu \
  tar xzf /backup/my_vol.tar.gz -C /data
```

---

## NETWORKS

```text
docker network ls                    → List networks
docker network create my_net         → Create custom network
docker network inspect my_net        → Network details + attached containers
docker network rm my_net             → Remove network
docker run --network my_net image    → Run on custom network
docker network connect my_net container_id    → Connect container to network
docker network disconnect my_net container_id → Disconnect
```

```text
💡 On a CUSTOM network, containers resolve each other BY NAME
(http://myapp:8080). The default bridge network does NOT do name
resolution — if container DNS "doesn't work", this is why.
```

---

## DOCKER COMPOSE

```text
docker compose up                    → Start services (foreground)
docker compose up -d                 → Start detached
docker compose up -d --build         → Rebuild images, then start
docker compose down                  → Stop & remove containers
docker compose down -v               → Also remove volumes (⚠️ deletes data)
docker compose down --remove-orphans → Also remove containers no longer in the file
docker compose build                 → Build/rebuild images
docker compose ps                    → List compose services
docker compose logs -f               → Follow all service logs
docker compose logs -f service_name  → Follow specific service
docker compose exec service bash     → Shell into service
docker compose restart service       → Restart one service
docker compose pull                  → Pull latest images
docker compose config                → Validate + print resolved config
```

### VPS agent template (compose.yaml)

```yaml
services:
  agent:
    build: .
    restart: unless-stopped
    env_file: .env
    ports:
      - "127.0.0.1:8501:8501"    # Loopback only — nginx fronts it (see Security)
    volumes:
      - agent_data:/app/data
    healthcheck:
      test: ["CMD", "curl", "-fsS", "http://localhost:8501/health"]
      interval: 30s
      timeout: 5s
      retries: 3

volumes:
  agent_data:
```

```text
💡 Compose v2 is the `docker compose` subcommand (no hyphen, no
`version:` key needed). The standalone `docker-compose` binary is legacy.
```

---

## CLEANUP & DISK MANAGEMENT

```text
docker system df                     → Disk usage by images/containers/volumes
docker system prune                  → Remove all unused resources
docker system prune -a               → Also remove ALL unused images
docker system prune -a --volumes     → ⚠️ Also unused volumes — data loss if careless
docker builder prune                 → Clear build cache (often the hidden hog)
```

### Container log rotation (the classic full-disk cause)

Container stdout/stderr grows unbounded by default. Cap it globally:

```bash
# /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" }
}
```

```bash
sudo systemctl restart docker    # ⚠️ Restarts containers; applies to NEW containers only
```

---

## DOCKERFILE REFERENCE

```text
FROM ubuntu:22.04        → Base image
WORKDIR /app             → Set working directory
COPY . .                 → Copy files from host
RUN apt-get update       → Run command during build
ENV PORT=8080            → Set environment variable
EXPOSE 8080              → Document port (informational — -p publishes)
CMD ["python", "app.py"] → Default run command (overridable)
ENTRYPOINT ["python"]    → Fixed entrypoint (run args are appended)
ARG VERSION=1.0          → Build-time variable
USER appuser             → Drop root inside the container
HEALTHCHECK CMD curl -fsS http://localhost:8080/health || exit 1
```

### Python service — layer caching done right

```dockerfile
FROM python:3.11-slim
WORKDIR /app

# Dependencies FIRST — this layer only rebuilds when requirements change:
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Source LAST — code edits don't bust the pip layer:
COPY . .

RUN useradd -m appuser
USER appuser
CMD ["python", "app.py"]
```

```bash
# .dockerignore — keep junk out of the build context:
.git
.env
__pycache__/
*.pyc
venv/
```

---

## SECURITY ON A VPS

```text
⚠️ -p 8080:80 BYPASSES ufw — Docker writes its own iptables rules ahead
of the firewall. A "deny all" ufw box with a published port is still
publicly reachable on that port. (Details: VPS Hardening sheet.)
```

```text
Rules of thumb:
1. Publish to loopback: -p 127.0.0.1:8080:80 — then front with nginx
   (TLS + rate limit + allow/deny).
2. docker group membership == passwordless root. Grant it like sudo.
3. Never mount /var/run/docker.sock into a container you don't fully
   trust — that's host root by another name.
4. Run processes as USER appuser in the image, not root.
5. Pin image tags (python:3.11-slim), never :latest in anything persistent.
```

---

## TIPS & GOTCHAS

- **Firewall bypass is the big one** — see Security above. Audit with `sudo ss -tlnp` from the host: anything bound to `0.0.0.0` by docker-proxy is internet-facing regardless of ufw.
- **Disk full at 3am = container logs or build cache** — `docker system df` first, `docker inspect -f '{{.LogPath}}'` to find the offender, daemon.json rotation to fix it permanently, `docker builder prune` for the cache.
- **`docker compose down -v` deletes named volumes** — the `-v` that means "verbose" everywhere else means "destroy my data" here.
- **CMD vs ENTRYPOINT** — `docker run image args` REPLACES CMD but APPENDS to ENTRYPOINT. Combo pattern: ENTRYPOINT = the binary, CMD = default flags.
- **Pick ONE supervisor** — either `restart: unless-stopped` in compose OR a systemd unit managing `docker compose up`; both at once fight over restarts.
- **Layer order = build speed** — copy dependency manifests and install before copying source (see Dockerfile section). Backwards ordering reinstalls every dependency on every code change.
- **Container DNS "broken"** — you're on the default bridge. Create a custom network; name resolution only works there.
- **`:latest` is a moving target** — a redeploy can silently pull a different image than what you tested. Pin versions; pin digests (`image@sha256:...`) for anything critical.
- **Prefer COPY over ADD** — ADD's URL-fetching and auto-tar-extraction are surprise behaviors; COPY does exactly one thing.

---
*Last Updated: 2026-07*
