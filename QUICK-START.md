# Quick Start Guide - Nexus Only Setup

## 1. Start Nexus (5 minutes)

```cmd
cd f:\dockers\package-mirrors
docker-compose up -d
```

Wait 2-3 minutes for Nexus to initialize.

## 2. Get Admin Password

```cmd
docker exec nexus cat /nexus-data/admin.password
```

Copy the password shown.

## 3. Login to Nexus Web UI

- Open: `http://MIRROR_SERVER_IP:8081`
- Username: `admin`
- Password: (from step 2)
- Complete setup wizard:
  - Change admin password
  - **Enable anonymous access** (allows clients to download without authentication)

## 4. Create npm Proxy Repository

1. Settings (⚙️) → Repository → Repositories → Create repository
2. Select: `npm (proxy)`
3. Configure:
   - **Name**: `npm-proxy`
   - **Remote storage**: `https://registry.npmjs.org/`
   - **Maximum component age**: `-1` (cache forever)
   - **Maximum metadata age**: `1440` minutes
   - **Blob store**: `default`
4. Save

## 5. Create PyPI Proxy Repository

1. Settings (⚙️) → Repository → Repositories → Create repository
2. Select: `pypi (proxy)`
3. Configure:
   - **Name**: `pypi-proxy`
   - **Remote storage**: `https://pypi.org`
   - **Maximum component age**: `-1` (cache forever)
   - **Maximum metadata age**: `1440` minutes
   - **Blob store**: `default`
4. Save

## 5a. Create APT Proxy Repositories (Optional - for Ubuntu/Debian)

**Important**: Each distribution requires **TWO repositories** (main + security).

### For Ubuntu 24.04 (Create Both):

**Repository 1 - Main:**
1. Create repository → `apt (proxy)`
2. Configure:
   - **Name**: `ubuntu-noble-proxy`
   - **Remote storage**: `http://archive.ubuntu.com/ubuntu/`
   - **Distribution**: `noble`
   - **Maximum component age**: `-1`
3. Save

**Repository 2 - Security:**
1. Create repository → `apt (proxy)`
2. Configure:
   - **Name**: `ubuntu-noble-security-proxy`
   - **Remote storage**: `http://security.ubuntu.com/ubuntu/`
   - **Distribution**: `noble-security`
   - **Maximum component age**: `-1`
3. Save

---

### For Debian 12 (Create Both):

**Repository 1 - Main:**
1. Create repository → `apt (proxy)`
2. Configure:
   - **Name**: `debian-bookworm-proxy`
   - **Remote storage**: `http://deb.debian.org/debian/`
   - **Distribution**: `bookworm`
   - **Maximum component age**: `-1`
3. Save

**Repository 2 - Security:**
1. Create repository → `apt (proxy)`
2. Configure:
   - **Name**: `debian-bookworm-security-proxy`
   - **Remote storage**: `http://security.debian.org/debian-security/`
   - **Distribution**: `bookworm-security`
   - **Maximum component age**: `-1`
3. Save

---

See **NEXUS-APT-SETUP.md** for detailed instructions and client configuration.

## 6. Configure npm/pnpm (30 seconds)

```cmd
npm config set registry http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
pnpm config set registry http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

Test:
```cmd
npm install express
```

## 7. Configure pip (30 seconds)

Create `%APPDATA%\pip\pip.ini`:
```ini
[global]
index-url = http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
trusted-host = MIRROR_SERVER_IP
```

Test:
```cmd
python -m pip install requests
```

## 8. Configure uv (30 seconds)

```cmd
setx UV_INDEX_URL "http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple"
```

Close and reopen terminal, then test:
```cmd
uv pip install --system flask
```

## Done! 🎉

### Access URLs
- **Nexus Web UI**: http://MIRROR_SERVER_IP:8081
- **npm/pnpm**: http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
- **pip/uv**: http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple

### Next Steps
- Configure other machines (see CLIENT-SETUP.md)
- Set up backups (see README.md)
- Monitor disk usage in Nexus UI
