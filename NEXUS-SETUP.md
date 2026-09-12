# Nexus PyPI Proxy Setup Guide

Detailed instructions for configuring Nexus Repository Manager as a PyPI proxy.

## Initial Setup

### 1. Start Nexus

```cmd
cd f:\dockers\package-mirrors
docker-compose up -d nexus
```

Wait 2-3 minutes for Nexus to initialize.

### 2. Get Initial Admin Password

```cmd
docker exec nexus cat /nexus-data/admin.password
```

Copy the password shown.

### 3. Access Web UI

Open browser: `http://MIRROR_SERVER_IP:8081`

### 4. Login

- **Username**: `admin`
- **Password**: (from step 2)

### 5. Complete Setup Wizard

1. **Change Password**: Set a new admin password (save it in `.env` file)
2. **Configure Anonymous Access**: 
   - **Enable** anonymous access (recommended for pull-through cache)
   - Allows clients to pull packages without authentication
3. **Complete**: Click finish

---

## Create PyPI Proxy Repository

### Step-by-Step

1. **Navigate**: Click gear icon (⚙️) → Repository → Repositories

2. **Create Repository**: Click "Create repository"

3. **Select Recipe**: Choose `pypi (proxy)`

4. **Configure Settings**:

   **Repository Settings**:
   - **Name**: `pypi-proxy` (unique identifier)
   - **Online**: ✓ (checked - accepts incoming requests)
   - **Sonatype Nexus Firewall**: Leave default (if shown)

   **Proxy Section**:
   - **Remote storage**: `https://pypi.org`
   - **Use certificates stored in the Nexus Repository truststore**: ✓ (checked)
   - **Keep encoded characters**: ☐ (unchecked - not needed for PyPI)
   - **Block outbound connections**: ☐ (unchecked)
   - **Auto-block outbound connections**: ✓ (checked - auto-block if unreachable)
   - **Maximum component age**: `-1` (never re-check - cache forever)
     - *This does NOT delete packages, it just controls re-checking for updates*
     - *PyPI packages are immutable (version 1.0.0 never changes), so `-1` is perfect*
   - **Maximum metadata age**: `1440` minutes (1 day - for discovering new versions)

   **Storage Section**:
   - **Blob store**: `default`
   - **Strict Content Type Validation**: ✓ (checked)

   **Routing Rule**:
   - Leave as "None" (or default)

   **Negative Cache Section**:
   - **Cache responses for content not present**: ✓ (checked)
   - **How long to cache the fact that a file was not found**: `1440` minutes (1 day)

   **Cleanup**:
   - **Leave empty** (no cleanup policies = cache forever)
   - *Cleanup policies actually delete cached packages - we don't want that*

   **HTTP Section**:
   - **Authentication**: Leave blank (no authentication needed for PyPI.org)

5. **Save**: Scroll down and click "Create repository"

---

## Verify Configuration

### Test Repository Access

1. In Nexus UI: Repository → Repositories → `pypi-proxy`
2. Click "Copy URL"
3. URL should be: `http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/`

### Test with curl

```cmd
curl http://MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple/
```

Should return HTML page listing packages.

---

## Configure Cleanup Policies (Optional)

Automatically remove old cached packages after 6 months.

### Create Cleanup Policy

1. **Navigate**: Settings (⚙️) → Repository → Cleanup Policies

2. **Create Policy**: Click "Create Cleanup Policy"

3. **Configure**:
   - **Name**: `pypi-6month-retention`
   - **Format**: `pypi`
   - **Criteria**:
     - **Last downloaded**: `180` days
   - **Preview**: Click to see what would be deleted

4. **Save**: Click "Create"

### Apply Policy to Repository

1. **Navigate**: Repository → Repositories → `pypi-proxy`

2. **Edit**: Click repository, then scroll to bottom

3. **Cleanup Policies**: Select `pypi-6month-retention`

4. **Save**

### Schedule Cleanup Task

1. **Navigate**: System → Tasks

2. **Create Task**: Click "Create task"

3. **Select Type**: `Admin - Compact blob store`

4. **Configure**:
   - **Name**: `Cleanup PyPI Cache`
   - **Blob store**: `default`
   - **Schedule**: Daily at 2:00 AM

5. **Save**

---

## Additional PyPI Repositories (Optional)

### Create Hosted Repository (Private Packages)

For hosting your own Python packages:

1. Create repository → `pypi (hosted)`
2. Name: `pypi-hosted`
3. Save

### Create Group Repository (Combine Multiple)

To combine proxy and hosted:

1. Create repository → `pypi (group)`
2. Name: `pypi-all`
3. **Member repositories**:
   - Add: `pypi-hosted`
   - Add: `pypi-proxy`
4. Save

Then configure pip to use: `http://MIRROR_SERVER_IP:8081/repository/pypi-all/simple`

---

## Security Configuration

### Change Admin Password

1. **Navigate**: Settings (⚙️) → Security → Users → admin
2. **Change Password**: Set strong password
3. **Update** `.env` file with new password

### Create Read-Only User (Optional)

1. **Navigate**: Security → Users → Create user
2. **Configure**:
   - **ID**: `pypi-reader`
   - **Password**: (set password)
   - **Status**: Active
   - **Roles**: `nx-anonymous`
3. **Save**

### Disable Anonymous Access (Optional)

If you want authentication:

1. **Navigate**: Security → Anonymous Access
2. **Uncheck**: "Allow anonymous users to access the server"
3. **Save**

Then configure pip with credentials:
```ini
[global]
index-url = http://pypi-reader:password@MIRROR_SERVER_IP:8081/repository/pypi-proxy/simple
```

---

## Monitoring and Maintenance

### View Cache Statistics

1. **Navigate**: Browse → Browse (folder icon)
2. Select: `pypi-proxy`
3. View cached packages and sizes

### View System Information

1. **Navigate**: System → Support → System Information
2. View: Memory, disk usage, thread pools

### Export Logs

1. **Navigate**: System → Support → Log Viewer
2. **Download**: Export logs if needed

---

## Backup Configuration

### Manual Backup

```cmd
docker-compose stop nexus
xcopy nexus\data nexus-backup\ /E /I
docker-compose start nexus
```

### Automated Backup Task

1. **Navigate**: System → Tasks → Create task
2. **Select**: `Admin - Export databases for backup`
3. **Configure**:
   - **Location**: `/nexus-data/backup`
   - **Schedule**: Daily at 3:00 AM
4. **Save**

---

## Troubleshooting

### Cannot Access Web UI

```cmd
# Check if running
docker ps | findstr nexus

# Check logs
docker-compose logs nexus

# Restart
docker-compose restart nexus
```

### Port Already in Use

Edit `docker-compose.yml`:
```yaml
ports:
  - "8082:8081"  # Change external port
```

### Out of Memory

Increase memory in `docker-compose.yml`:
```yaml
environment:
  - INSTALL4J_ADD_VM_PARAMS=-Xms2048m -Xmx4096m -XX:MaxDirectMemorySize=4096m
```

### Slow Performance

1. Check disk space: `dir nexus\data`
2. Check memory usage in Nexus UI
3. Consider increasing VM parameters
4. Enable cleanup policies

---

## Advanced Features

### Enable HTTPS

Use reverse proxy (nginx/traefik) for SSL termination.

### LDAP Integration

Navigate: Security → LDAP → Create connection

### Webhooks

Navigate: System → Capabilities → Create capability → Webhook

### Custom Blob Stores

Navigate: Repository → Blob Stores → Create blob store

---

## References

- **Nexus Documentation**: https://help.sonatype.com/repomanager3
- **PyPI Format**: https://help.sonatype.com/repomanager3/nexus-repository-administration/formats/pypi-repositories


---

## Quick Reference: Configuration Values

For easy copy-paste when setting up PyPI proxy:

| Field | Value |
|-------|-------|
| **Name** | `pypi-proxy` |
| **Online** | ✓ Checked |
| **Remote storage** | `https://pypi.org` |
| **Use truststore certificates** | ✓ Checked |
| **Keep encoded characters** | ☐ Unchecked |
| **Block outbound connections** | ☐ Unchecked |
| **Auto-block outbound** | ✓ Checked |
| **Maximum component age** | `-1` (cache forever) |
| **Maximum metadata age** | `1440` minutes (1 day) |
| **Blob store** | `default` |
| **Strict Content Type Validation** | ✓ Checked |
| **Negative cache enabled** | ✓ Checked |
| **Negative cache TTL** | `1440` minutes |
| **Authentication** | (leave blank) |

### Important Notes:

- **Maximum component age = -1 (RECOMMENDED)**
  - **What it means**: Never re-check PyPI.org for updates to already-cached packages
  - **Why `-1` is perfect**: PyPI packages are immutable (requests==2.31.0 never changes)
  - **Does NOT delete anything**: This only controls re-checking, not deletion
  - **Cache forever**: Without cleanup policies, packages stay cached indefinitely

- **Maximum metadata age = 1440 minutes (1 day)**
  - Controls how often Nexus checks for NEW package versions
  - Keeps metadata (package lists) reasonably fresh
  - Does not affect already-cached packages

- **Cleanup Policies vs Maximum Component Age**:
  - **Maximum Component Age**: How often to re-check remote (does NOT delete)
  - **Cleanup Policies**: Actually deletes packages from disk
  - **To cache forever**: Use `-1` for component age AND leave cleanup policies empty

- **Keep encoded characters**: Only enable for AWS S3, Cloudflare CDN, or Azure Blob Storage. Leave unchecked for PyPI.org.

- **Auto-block**: Automatically blocks the repository if PyPI.org becomes unreachable, preventing slow timeouts.
