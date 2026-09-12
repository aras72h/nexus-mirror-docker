# Nexus APT Proxy Setup Guide

Detailed instructions for configuring Nexus Repository Manager as an APT proxy for Ubuntu and Debian.

## Overview

Nexus supports APT repositories for Debian-based distributions. You'll need to create **separate proxy repositories** for each distribution:
- Ubuntu 24.04 (noble)
- Debian 12 (bookworm)

## Important: APT Proxy Limitations

**Note**: Nexus APT proxy requires you to specify a **single distribution** per repository. Unlike a full mirror, it only caches packages you actually request, making it more like a pull-through cache.

---

## Create Ubuntu 24.04 APT Proxies

Ubuntu requires **TWO separate proxy repositories** because main packages and security updates come from different servers.

### Repository 1: Ubuntu Main Packages

#### Step 1: Access Nexus Web UI

Open: `http://MIRROR_SERVER_IP:8081`

Login with admin credentials.

#### Step 2: Navigate to Repositories

1. Click the **gear icon (⚙️)** at the top (Settings)
2. Go to **Repository** → **Repositories** (in left sidebar)
3. Click **Create repository**

#### Step 3: Select apt (proxy)

Click on **apt (proxy)** from the recipe list.

#### Step 4: Configure Ubuntu Main Repository

**Repository Settings**:
- **Name**: `ubuntu-noble-proxy`
- **Online**: ✓ (checked)

**Proxy Section**:
- **Remote storage**: `http://archive.ubuntu.com/ubuntu/`
  - Or use a mirror closer to you: `http://us.archive.ubuntu.com/ubuntu/`
- **Distribution**: `noble` (Ubuntu 24.04 codename)
- **Flat**: ☐ (unchecked - Ubuntu uses standard repo structure)
- **Use certificates stored in truststore**: ✓ (checked)
- **Block outbound connections**: ☐ (unchecked)
- **Auto-block outbound connections**: ✓ (checked)
- **Maximum component age**: `-1` (cache forever)
- **Maximum metadata age**: `1440` minutes (1 day)

**Storage Section**:
- **Blob store**: `default`
- **Strict Content Type Validation**: ✓ (checked)

**Negative Cache Section**:
- **Cache responses for content not present**: ✓ (checked)
- **How long to cache**: `1440` minutes

**Cleanup**:
- Leave empty (cache forever)

**HTTP Section**:
- **Authentication**: Leave blank

#### Step 5: Save

Click **Create repository**.

---

### Repository 2: Ubuntu Security Updates

Security updates come from a **different server**, so we need a separate proxy.

#### Step 1: Create Second Repository

1. Settings (⚙️) → Repository → Repositories → Create repository
2. Select: `apt (proxy)`

#### Step 2: Configure Ubuntu Security Repository

**Repository Settings**:
- **Name**: `ubuntu-noble-security-proxy`
- **Online**: ✓ (checked)

**Proxy Section**:
- **Remote storage**: `http://security.ubuntu.com/ubuntu/`
  - **Important**: Different URL than main repository!
- **Distribution**: `noble-security` (Ubuntu 24.04 security)
- **Flat**: ☐ (unchecked)
- **Use certificates stored in truststore**: ✓ (checked)
- **Block outbound connections**: ☐ (unchecked)
- **Auto-block outbound connections**: ✓ (checked)
- **Maximum component age**: `-1` (cache forever)
- **Maximum metadata age**: `1440` minutes (1 day)

**Storage Section**:
- **Blob store**: `default`
- **Strict Content Type Validation**: ✓ (checked)

**Negative Cache Section**:
- **Cache responses for content not present**: ✓ (checked)
- **How long to cache**: `1440` minutes

**Cleanup**:
- Leave empty (cache forever)

**HTTP Section**:
- **Authentication**: Leave blank

#### Step 3: Save

Click **Create repository**.

---

## Create Debian 12 APT Proxies

Debian also requires **TWO separate proxy repositories** because main packages and security updates come from different servers.

### Repository 1: Debian Main Packages

#### Step 1: Create Main Repository

1. Settings (⚙️) → Repository → Repositories → Create repository
2. Select: `apt (proxy)`

#### Step 2: Configure Debian Main Repository

**Repository Settings**:
- **Name**: `debian-bookworm-proxy`
- **Online**: ✓ (checked)

**Proxy Section**:
- **Remote storage**: `http://deb.debian.org/debian/`
  - Or use a mirror: `http://ftp.us.debian.org/debian/`
- **Distribution**: `bookworm` (Debian 12 codename)
- **Flat**: ☐ (unchecked)
- **Use certificates stored in truststore**: ✓ (checked)
- **Block outbound connections**: ☐ (unchecked)
- **Auto-block outbound connections**: ✓ (checked)
- **Maximum component age**: `-1` (cache forever)
- **Maximum metadata age**: `1440` minutes (1 day)

**Storage Section**:
- **Blob store**: `default`
- **Strict Content Type Validation**: ✓ (checked)

**Negative Cache Section**:
- **Cache responses for content not present**: ✓ (checked)
- **How long to cache**: `1440` minutes

**Cleanup**:
- Leave empty (cache forever)

**HTTP Section**:
- **Authentication**: Leave blank

#### Step 3: Save

Click **Create repository**.

---

### Repository 2: Debian Security Updates

Security updates come from a **different server**, so we need a separate proxy.

#### Step 1: Create Security Repository

1. Settings (⚙️) → Repository → Repositories → Create repository
2. Select: `apt (proxy)`

#### Step 2: Configure Debian Security Repository

**Repository Settings**:
- **Name**: `debian-bookworm-security-proxy`
- **Online**: ✓ (checked)

**Proxy Section**:
- **Remote storage**: `http://security.debian.org/debian-security/`
  - **Important**: Different URL than main repository!
- **Distribution**: `bookworm-security` (Debian 12 security)
- **Flat**: ☐ (unchecked)
- **Use certificates stored in truststore**: ✓ (checked)
- **Block outbound connections**: ☐ (unchecked)
- **Auto-block outbound connections**: ✓ (checked)
- **Maximum component age**: `-1` (cache forever)
- **Maximum metadata age**: `1440` minutes (1 day)

**Storage Section**:
- **Blob store**: `default`
- **Strict Content Type Validation**: ✓ (checked)

**Negative Cache Section**:
- **Cache responses for content not present**: ✓ (checked)
- **How long to cache**: `1440` minutes

**Cleanup**:
- Leave empty (cache forever)

**HTTP Section**:
- **Authentication**: Leave blank

#### Step 3: Save

Click **Create repository**.

---

## Configure Ubuntu 24.04 Clients

### Using sources.list.d (Recommended for Ubuntu 24.04+)

Ubuntu 24.04 uses the new DEB822 format in `/etc/apt/sources.list.d/ubuntu.sources`.

#### Step 1: Backup original sources
```bash
sudo cp /etc/apt/sources.list.d/ubuntu.sources /etc/apt/sources.list.d/ubuntu.sources.backup
```

#### Step 2: Edit sources file
```bash
sudo nano /etc/apt/sources.list.d/ubuntu.sources
```

#### Step 3: Replace with Nexus proxies

**Original** (before):
```
Types: deb
URIs: http://archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

**Replace with** (after):
```
# Main, updates, and backports from Nexus
Types: deb
URIs: http://MIRROR_SERVER_IP:8081/repository/ubuntu-noble-proxy/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# Security updates from Nexus
Types: deb
URIs: http://MIRROR_SERVER_IP:8081/repository/ubuntu-noble-security-proxy/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

#### Step 4: Update package lists
```bash
sudo apt update
```

You should see both repositories fetching metadata successfully.

---

## Configure Debian 12 Clients

### Edit sources.list

#### Step 1: Backup original sources
```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
```

#### Step 2: Edit sources file
```bash
sudo nano /etc/apt/sources.list
```

#### Step 3: Replace URLs with Nexus proxies

**Original** (before):
```
deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware
deb http://deb.debian.org/debian/ bookworm-updates main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
```

**Replace with** (after):
```
# Main and updates from Nexus
deb http://MIRROR_SERVER_IP:8081/repository/debian-bookworm-proxy/ bookworm main contrib non-free non-free-firmware
deb http://MIRROR_SERVER_IP:8081/repository/debian-bookworm-proxy/ bookworm-updates main contrib non-free non-free-firmware

# Security updates from Nexus
deb http://MIRROR_SERVER_IP:8081/repository/debian-bookworm-security-proxy/ bookworm-security main contrib non-free non-free-firmware
```

#### Step 4: Update package lists
```bash
sudo apt update
```

You should see all three suites (bookworm, bookworm-updates, bookworm-security) fetching successfully.

---

## Test the Setup

### On Ubuntu 24.04
```bash
# Update package lists (first time will be slow)
sudo apt update

# Install a package (should be cached after first install)
sudo apt install curl

# Second install on another machine should be much faster
```

### On Debian 12
```bash
# Update package lists
sudo apt update

# Install a package
sudo apt install wget

# Reinstall to see cache effect
sudo apt install --reinstall wget
```

---

## Important Notes

### Distribution Codenames

| Version | Codename |
|---------|----------|
| **Ubuntu 24.04 LTS** | noble |
| **Ubuntu 22.04 LTS** | jammy |
| **Ubuntu 20.04 LTS** | focal |
| **Debian 12** | bookworm |
| **Debian 11** | bullseye |
| **Debian 10** | buster |

### Why Separate Security Repositories?

Both Ubuntu and Debian use **different servers** for security updates:

| Distribution | Main Packages | Security Updates |
|--------------|---------------|------------------|
| **Ubuntu** | `archive.ubuntu.com/ubuntu` | `security.ubuntu.com/ubuntu` |
| **Debian** | `deb.debian.org/debian` | `security.debian.org/debian-security` |

**Why?** Security updates need:
- Faster propagation (no mirror lag)
- Higher reliability (dedicated infrastructure)
- Separate signing keys and policies

Nexus APT proxy can only point to **one remote URL** per repository, so you **must** create separate proxies for security updates.

### What Happens if You Don't?

If you try to use the same proxy for both main and security, you'll get:
```
Err:14 http://MIRROR_SERVER_IP:8081/repository/debian-bookworm-proxy bookworm-security Release
404 Not Found
E: The repository does not have a Release file.
```

This is **exactly what you encountered** - the fix is to create separate security proxies as documented above.

---

## Troubleshooting

### "Failed to fetch" errors

1. **Check Nexus is running**:
```bash
curl http://MIRROR_SERVER_IP:8081/repository/ubuntu-noble-proxy/
```

2. **Check repository is Online** in Nexus UI

3. **Verify anonymous access** is enabled (Settings → Security → Anonymous Access)

### GPG signature errors

Nexus proxies don't re-sign packages. Use the original GPG keys:
- Ubuntu: `/usr/share/keyrings/ubuntu-archive-keyring.gpg`
- Debian: `/usr/share/keyrings/debian-archive-keyring.gpg`

These are already installed on your systems.

### Slow first update

The first `apt update` on a new Nexus APT proxy downloads all package metadata. This can take 5-10 minutes. Subsequent updates are instant.

### "Repository doesn't support architecture" errors

Make sure your sources include the correct architectures. Ubuntu 24.04 and Debian 12 use `amd64` by default.

---

## Reverting to Public Repositories

### Ubuntu 24.04
```bash
sudo cp /etc/apt/sources.list.d/ubuntu.sources.backup /etc/apt/sources.list.d/ubuntu.sources
sudo apt update
```

### Debian 12
```bash
sudo cp /etc/apt/sources.list.backup /etc/apt/sources.list
sudo apt update
```

---

## Quick Reference

### Ubuntu 24.04 (Noble)

**Nexus Repository Names**:
- Main/Updates: `ubuntu-noble-proxy`
- Security: `ubuntu-noble-security-proxy` (**required**)

**Nexus Repository URLs**:
- Main: `http://MIRROR_SERVER_IP:8081/repository/ubuntu-noble-proxy/`
- Security: `http://MIRROR_SERVER_IP:8081/repository/ubuntu-noble-security-proxy/`

**Upstream URLs**:
- Main: `http://archive.ubuntu.com/ubuntu/` (Distribution: `noble`)
- Security: `http://security.ubuntu.com/ubuntu/` (Distribution: `noble-security`)

**Suites Handled**:
- Main proxy: `noble`, `noble-updates`, `noble-backports`
- Security proxy: `noble-security`

---

### Debian 12 (Bookworm)

**Nexus Repository Names**:
- Main/Updates: `debian-bookworm-proxy`
- Security: `debian-bookworm-security-proxy` (**required**)

**Nexus Repository URLs**:
- Main: `http://MIRROR_SERVER_IP:8081/repository/debian-bookworm-proxy/`
- Security: `http://MIRROR_SERVER_IP:8081/repository/debian-bookworm-security-proxy/`

**Upstream URLs**:
- Main: `http://deb.debian.org/debian/` (Distribution: `bookworm`)
- Security: `http://security.debian.org/debian-security/` (Distribution: `bookworm-security`)

**Suites Handled**:
- Main proxy: `bookworm`, `bookworm-updates`
- Security proxy: `bookworm-security`

---

## References

- **Nexus Documentation**: https://help.sonatype.com/repomanager3
- **APT Format**: https://help.sonatype.com/en/apt-repositories.html
- **Ubuntu Releases**: https://wiki.ubuntu.com/Releases
- **Debian Releases**: https://www.debian.org/releases/
