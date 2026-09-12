# Package Mirror Setup (npm/pnpm + pip/uv)

Pull-through cache for npm/pnpm and pip/uv packages using Nexus Repository Manager.

## Services

- **Nexus Repository Manager**: `http://MIRROR_SERVER_IP:8081`
  - npm/pnpm proxy: `http://MIRROR_SERVER_IP:8081/repository/npm-proxy/`
  - PyPI proxy: `http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple`
  - APT Ubuntu 24.04 (main): `http://MIRROR_SERVER_IP:8081/repository/ubuntu-noble-proxy/`
  - APT Ubuntu 24.04 (security): `http://MIRROR_SERVER_IP:8081/repository/ubuntu-noble-security-proxy/`
  - APT Debian 12 (main): `http://MIRROR_SERVER_IP:8081/repository/debian-bookworm-proxy/`
  - APT Debian 12 (security): `http://MIRROR_SERVER_IP:8081/repository/debian-bookworm-security-proxy/`

## Quick Start

### 1. Start Services

```cmd
docker-compose up -d
```

### 2. Check Status

```cmd
docker-compose ps
docker-compose logs -f
```

### 3. Wait for Nexus to Initialize

- **Nexus**: First startup takes 2-3 minutes
- Check logs: `docker-compose logs -f nexus`
- Ready when you see: "Started Sonatype Nexus"

---

## Nexus Initial Setup

### Access Web UI
- URL: `http://MIRROR_SERVER_IP:4873`
- No login required for reading packages
- Login required for publishing private packages

### Configure npm to Use Mirror

**Option 1: Global Configuration (All Projects)**
```cmd
npm config set registry http://MIRROR_SERVER_IP:4873
```

**Option 2: Per-Project (.npmrc in project root)**
```
registry=http://MIRROR_SERVER_IP:4873
```

**Option 3: Environment Variable**
```cmd
set NPM_CONFIG_REGISTRY=http://MIRROR_SERVER_IP:4873
npm install
```

### Configure pnpm to Use Mirror

**Global Configuration**
```cmd
pnpm config set registry http://MIRROR_SERVER_IP:4873
```

**Per-Project (.npmrc in project root)**
```
registry=http://MIRROR_SERVER_IP:4873
```

### Test npm Cache

```cmd
npm install express
```

First install fetches from npmjs.org and caches locally. Subsequent installs are served from cache.

### Publishing Private Packages (Optional)

1. Create user:
```cmd
npm adduser --registry http://MIRROR_SERVER_IP:4873
```

2. Publish:
```cmd
npm publish --registry http://MIRROR_SERVER_IP:4873
```

---

## Nexus Setup (pip/uv)

### Initial Setup

1. **Access Web UI**: `http://MIRROR_SERVER_IP:8081`

2. **Get Initial Admin Password**:
```cmd
docker exec nexus cat /nexus-data/admin.password
```

3. **Login**:
   - Username: `admin`
   - Password: (from step 2)

4. **Complete Setup Wizard**:
   - Change admin password
   - Enable anonymous access (recommended for pull-through cache)
   - Keep default settings

### Create PyPI Proxy Repository

1. **Go to**: Settings (gear icon) → Repository → Repositories → Create repository

2. **Select**: `pypi (proxy)`

3. **Configure**:
   - **Name**: `pypi-proxy`
   - **Remote storage**: `https://pypi.org`
   - **Maximum component age**: `180` (days)
   - **Maximum metadata age**: `1440` (minutes)
   - **Negative cache enabled**: ✓
   - **Negative cache TTL**: `1440` (minutes)
   - **Blob store**: `default`

4. **Save**

### Configure pip to Use Mirror

**Option 1: Global Configuration**

Create/edit `%APPDATA%\pip\pip.ini`:
```ini
[global]
index-url = http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
trusted-host = MIRROR_SERVER_IP
```

**Option 2: Per-Project (requirements.txt)**
```
--index-url http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
--trusted-host MIRROR_SERVER_IP

requests
flask
```

**Option 3: Command Line**
```cmd
pip install --index-url http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple --trusted-host MIRROR_SERVER_IP requests
```

### Configure uv to Use Mirror

**Environment Variable**
```cmd
set UV_INDEX_URL=http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
uv pip install requests
```

**Or in pyproject.toml**
```toml
[[tool.uv.index]]
url = "http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple"
default = true
```

### Test pip Cache

```cmd
pip install requests
```

First install fetches from pypi.org and caches in Nexus. Subsequent installs are instant.

---

## Cache Management

### Cache Retention
- **Default**: 180 days (6 months)
- Both services configured to keep packages for 6 months
- Packages accessed within 6 months stay cached

### Verdaccio Cache Location
- Path: `./verdaccio/storage/data`
- View cache stats on web UI

### Nexus Cache Location
- Path: `./nexus/data/blobs`
- View cache stats: Settings → System → Blob Stores

### Clear Cache

**Verdaccio**: Delete storage directory (while stopped)
```cmd
docker-compose stop verdaccio
rmdir /s /q verdaccio\storage\data
docker-compose start verdaccio
```

**Nexus**: Use web UI
1. Repository → Select repository
2. Repair - Invalidate cache
3. Or: Settings → Tasks → Create task → "Invalidate cache"

---

## Monitoring

### Check Service Health

```cmd
# Verdaccio
curl http://MIRROR_SERVER_IP:4873

# Nexus
curl http://MIRROR_SERVER_IP:8081
```

### View Logs

```cmd
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f verdaccio
docker-compose logs -f nexus
```

### Disk Usage

```cmd
# Check storage size
dir verdaccio\storage
dir nexus\data
```

---

## Backup

### Verdaccio Backup
```cmd
docker-compose stop verdaccio
xcopy verdaccio\storage verdaccio-backup\ /E /I
docker-compose start verdaccio
```

### Nexus Backup
Use Nexus built-in backup task:
1. Settings → System → Tasks
2. Create task → "Admin - Export databases for backup"
3. Or backup entire `nexus/data` directory

---

## Troubleshooting

### Verdaccio Not Starting
- Check ports: `netstat -an | findstr 4873`
- Check logs: `docker-compose logs verdaccio`

### Nexus Not Starting
- Needs 2GB+ RAM
- Check logs: `docker-compose logs nexus`
- First startup takes 2-3 minutes

### npm/pip Not Using Cache
- Verify configuration: `npm config get registry`
- Test connectivity: `curl http://MIRROR_SERVER_IP:4873`
- Check firewall rules

### Packages Not Caching
- Verify uplink configuration in Verdaccio
- Check Nexus proxy repository settings
- Review retention policies

---

## Security Notes

### Production Recommendations

1. **Change default passwords**
2. **Enable HTTPS** (use reverse proxy like nginx/traefik)
3. **Configure authentication** for publishing
4. **Set up firewall rules**
5. **Regular backups**

### Nexus Security
- Change admin password immediately after first login
- Consider disabling anonymous access if security is critical
- Enable HTTPS in production

### Verdaccio Security
- Enable authentication for publishing
- Limit user registration (`max_users` in config)
- Use htpasswd for user management

---

## Advanced Configuration

### Verdaccio Plugins
Add plugins to `verdaccio/plugins/` directory and configure in `config.yaml`.

### Nexus Additional Repositories
Nexus can also proxy:
- Docker Hub
- Maven Central
- NuGet Gallery
- RubyGems
- And more...

---

## System Requirements

- **CPU**: 2+ cores recommended
- **RAM**: 3-4GB (1GB Verdaccio + 2-3GB Nexus)
- **Disk**: Depends on cache size
  - Estimate: 1-10GB for typical usage
  - 50GB+ for heavy usage

---

## Stopping Services

```cmd
# Stop all
docker-compose stop

# Stop specific service
docker-compose stop verdaccio
docker-compose stop nexus
```

## Removing Services

```cmd
# Stop and remove containers
docker-compose down

# Remove with volumes (WARNING: deletes cache)
docker-compose down -v
```

---

## Support & Documentation

- **Verdaccio**: https://verdaccio.org/docs/
- **Nexus**: https://help.sonatype.com/repomanager3
