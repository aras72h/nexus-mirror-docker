# Local Package Mirror

A self-hosted, offline-capable pull-through cache for npm, PyPI, and APT packages using [Nexus Repository Manager](https://www.sonatype.com/products/sonatype-nexus-repository).

Designed for environments with unreliable internet access. Once a package is fetched, it is served from local storage indefinitely — no internet required on subsequent installs.

## What's Included

| Format | Tool(s) | Upstream |
|--------|---------|----------|
| npm | npm, pnpm, yarn | registry.npmjs.org |
| Python | pip, uv | pypi.org |
| APT | apt, apt-get | Ubuntu 24.04 + Debian 12 |

All formats are served through a single **Nexus Repository Manager** instance.

## Requirements

- Docker + Docker Compose v2
- 2–3 GB RAM available for Nexus
- 10–50 GB disk space (depends on cache size)

## Getting Started

Copy the environment file and set your server's IP:

```bash
cp .env.example .env
# Edit .env and set SERVER_IP to your machine's LAN IP
```

Start Nexus:

```bash
docker compose up -d
```

Then follow **[QUICK-START.md](QUICK-START.md)** to create repositories and configure your first client in ~10 minutes.

## Documentation

| File | What it covers |
|------|---------------|
| [QUICK-START.md](QUICK-START.md) | End-to-end setup from zero to working mirror |
| [NEXUS-NPM-SETUP.md](NEXUS-NPM-SETUP.md) | Detailed npm/pnpm proxy configuration in Nexus |
| [NEXUS-SETUP.md](NEXUS-SETUP.md) | Detailed PyPI proxy configuration in Nexus |
| [NEXUS-APT-SETUP.md](NEXUS-APT-SETUP.md) | APT proxy setup for Ubuntu 24.04 and Debian 12 |
| [CLIENT-SETUP.md](CLIENT-SETUP.md) | Configuring npm, pip, uv on client machines |
| [COMPARISON.md](COMPARISON.md) | Why Nexus over Verdaccio, pypiserver, apt-cacher-ng |

## Service URLs

Replace `MIRROR_SERVER_IP` with your server's LAN IP (set in `.env`).

| Service | URL |
|---------|-----|
| Nexus Web UI | `http://MIRROR_SERVER_IP:8081` |
| npm / pnpm | `http://MIRROR_SERVER_IP:8081/repository/npm-proxy/` |
| pip / uv | `http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple` |
| APT Ubuntu 24.04 (main) | `http://MIRROR_SERVER_IP:8081/repository/ubuntu-noble-proxy/` |
| APT Ubuntu 24.04 (security) | `http://MIRROR_SERVER_IP:8081/repository/ubuntu-noble-security-proxy/` |
| APT Debian 12 (main) | `http://MIRROR_SERVER_IP:8081/repository/debian-bookworm-proxy/` |
| APT Debian 12 (security) | `http://MIRROR_SERVER_IP:8081/repository/debian-bookworm-security-proxy/` |

## Quick Client Config

### npm / pnpm

```bash
npm config set registry http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
pnpm config set registry http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

### pip

```ini
# %APPDATA%\pip\pip.ini  (Windows)
# ~/.config/pip/pip.conf  (Linux/macOS)
[global]
index-url = http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
trusted-host = MIRROR_SERVER_IP
```

### uv

```bash
# Set once, applies to all uv commands
export UV_INDEX_URL=http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
```

See [CLIENT-SETUP.md](CLIENT-SETUP.md) for per-project and Windows-specific instructions.

## Managing the Service

### Stop and start

```bash
docker compose stop
docker compose start
```

### Restart

```bash
docker compose restart
```

### Remove containers (keeps cached data)

```bash
docker compose down
```

### Remove containers and all cached data

```bash
docker compose down -v
```

> ⚠️ `down -v` deletes the entire Nexus blob store. All cached packages will need to be re-fetched from the internet.

---

## Clearing the Cache

### Invalidate a single repository's cache

1. Nexus Web UI → Repository → Repositories
2. Select the repository (e.g. `npm-proxy`)
3. Click **Invalidate cache**

### Invalidate via scheduled task

1. Settings (⚙️) → System → Tasks → Create task
2. Select: `Invalidate cached data`
3. Configure the target repository and schedule
4. Save and run

### Rebuild metadata only (without deleting blobs)

1. Settings (⚙️) → System → Tasks → Create task
2. Select: `Repair - Rebuild repository browse`
3. Run manually as needed

---

## License

MIT
