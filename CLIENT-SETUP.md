# Client Setup Guide

Quick setup instructions for configuring clients to use the Nexus package mirror.

## NPM Configuration

### Windows

**Option 1: Global (Permanent)**
```cmd
npm config set registry http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

**Option 2: Per-Project**
Create `.npmrc` in project root:
```
registry=http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

**Verify**:
```cmd
npm config get registry
```

**Revert to Default**:
```cmd
npm config delete registry
```

---

## PNPM Configuration

### Windows

**Global**:
```cmd
pnpm config set registry http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

**Per-Project** (`.npmrc` in project root):
```
registry=http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

**Verify**:
```cmd
pnpm config get registry
```

---

## PIP Configuration

### Windows

**Global Configuration**

1. Create directory (if not exists):
```cmd
mkdir %APPDATA%\pip
```

2. Create/edit `%APPDATA%\pip\pip.ini`:
```ini
[global]
index-url = http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
trusted-host = MIRROR_SERVER_IP
```

**Per-Project** (`requirements.txt`):
```
--index-url http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
--trusted-host MIRROR_SERVER_IP

requests==2.31.0
flask==3.0.0
```

**Environment Variable**:
```cmd
set PIP_INDEX_URL=http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
set PIP_TRUSTED_HOST=MIRROR_SERVER_IP
pip install requests
```

**Verify**:
```cmd
pip config list
```

---

## UV Configuration

### Windows

**Environment Variables**:
```cmd
set UV_INDEX_URL=http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
uv pip install requests
```

**pyproject.toml** (per-project):
```toml
[[tool.uv.index]]
url = "http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple"
default = true
```

**Verify**:
```cmd
uv pip list
```

---

## Testing the Setup

### Test npm
```cmd
npm install express
```

### Test pnpm
```cmd
pnpm add express
```

### Test pip
```cmd
pip install requests
```

### Test uv
```cmd
uv pip install requests
```

First install fetches from internet and caches. Subsequent installs are instant from cache.

---

## Troubleshooting

### Cannot Connect to Mirror

1. **Test connectivity**:
```cmd
curl http://MIRROR_SERVER_IP:8081
```

2. **Check firewall** (allow port 8081)

3. **Verify Nexus is running**:
```cmd
# On mirror server
docker ps
```

### SSL/HTTPS Errors

For local mirrors without SSL, add to trusted hosts:
- npm: Already handled (http works)
- pip: `trusted-host = MIRROR_SERVER_IP` in pip.ini

### Slow First Install

This is normal. The mirror fetches from internet on first request. Subsequent installs will be fast.

---

## Reverting to Public Registries

### NPM
```cmd
npm config delete registry
```

### PNPM
```cmd
pnpm config delete registry
```

### PIP
Delete or rename `%APPDATA%\pip\pip.ini`

### UV
Remove environment variables or pyproject.toml configuration
