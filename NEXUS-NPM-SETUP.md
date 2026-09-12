# Nexus npm Proxy Setup Guide

Detailed instructions for configuring Nexus Repository Manager as an npm proxy.

## Create npm Proxy Repository

### Step 1: Access Nexus Web UI

Open: `http://MIRROR_SERVER_IP:8081`

Login with admin credentials.

### Step 2: Navigate to Repositories

1. Click the **gear icon (⚙️)** at the top (Settings)
2. Go to **Repository** → **Repositories** (in left sidebar)
3. Click **Create repository**

### Step 3: Select npm (proxy)

Click on **npm (proxy)** from the recipe list.

### Step 4: Configure Repository Settings

**Repository Settings**:
- **Name**: `npm-proxy` (unique identifier)
- **Online**: ✓ (checked - accepts incoming requests)

**Proxy Section**:
- **Remote storage**: `https://registry.npmjs.org/`
- **Use certificates stored in the Nexus Repository truststore**: ✓ (checked)
- **Keep encoded characters**: ☐ (unchecked - not needed for npm)
- **Block outbound connections**: ☐ (unchecked)
- **Auto-block outbound connections**: ✓ (checked - auto-block if unreachable)
- **Maximum component age**: `-1` (never re-check - cache forever)
  - *npm packages are immutable (lodash@4.17.21 never changes)*
  - *This does NOT delete packages, just controls re-checking*
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

**HTTP Section**:
- **Authentication**: Leave blank (no authentication needed for npmjs.org)

### Step 5: Save

Scroll down and click **Create repository**.

---

## Verify Configuration

### Test Repository Access

1. In Nexus UI: Repository → Repositories → `npm-proxy`
2. Click "Copy URL"
3. URL should be: `http://MIRROR_SERVER_IP:8081/repository/npm-proxy/`

### Test with curl

```cmd
curl http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

Should return JSON with repository info.

---

## Configure Clients

### npm Configuration

**Global (Permanent)**:
```cmd
npm config set registry http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

**Verify**:
```cmd
npm config get registry
```

**Test**:
```cmd
npm install express
```

### pnpm Configuration

**Global**:
```cmd
pnpm config set registry http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

**Verify**:
```cmd
pnpm config get registry
```

**Test**:
```cmd
pnpm add express
```

### yarn Configuration

**Global**:
```cmd
yarn config set registry http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

**Test**:
```cmd
yarn add express
```

---

## Per-Project Configuration

Create `.npmrc` in project root:

```
registry=http://MIRROR_SERVER_IP:8081/repository/npm-proxy/
```

This works for npm, pnpm, and yarn.

---

## Quick Reference: Configuration Values

| Field | Value |
|-------|-------|
| **Name** | `npm-proxy` |
| **Online** | ✓ Checked |
| **Remote storage** | `https://registry.npmjs.org/` |
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

---

## Important Notes

### Maximum Component Age = -1 (Cache Forever)

- **What it means**: Never re-check npmjs.org for updates to already-cached packages
- **Why `-1` is perfect**: npm packages are immutable (express@4.18.2 never changes)
- **Does NOT delete anything**: This only controls re-checking, not deletion
- **Cache forever**: Without cleanup policies, packages stay cached indefinitely

### Maximum Metadata Age = 1440 minutes

- Controls how often Nexus checks for NEW package versions
- Keeps package lists reasonably fresh
- Does not affect already-cached packages

### Cleanup Policies vs Maximum Component Age

- **Maximum Component Age**: How often to re-check remote (does NOT delete)
- **Cleanup Policies**: Actually deletes packages from disk
- **To cache forever**: Use `-1` for component age AND leave cleanup policies empty

---

## Publishing Private Packages (Optional)

### Create npm Hosted Repository

1. Settings (⚙️) → Repository → Repositories → Create repository
2. Select: `npm (hosted)`
3. Configure:
   - **Name**: `npm-hosted`
   - **Blob store**: `default`
4. Save

### Create npm Group Repository

Combine proxy and hosted:

1. Create repository → `npm (group)`
2. **Name**: `npm-all`
3. **Member repositories**:
   - Add: `npm-hosted`
   - Add: `npm-proxy`
4. Save

### Configure npm to Use Group

```cmd
npm config set registry http://MIRROR_SERVER_IP:8081/repository/npm-all/
```

### Publish to Hosted Repository

1. Create npm user in Nexus (Security → Users)
2. Configure npm authentication:
   ```cmd
   npm login --registry=http://MIRROR_SERVER_IP:8081/repository/npm-hosted/
   ```
3. Publish:
   ```cmd
   npm publish --registry=http://MIRROR_SERVER_IP:8081/repository/npm-hosted/
   ```

---

## Monitoring

### View Cached Packages

1. Navigate: Browse → Browse (folder icon)
2. Select: `npm-proxy`
3. View cached packages and sizes

### View Statistics

Settings (⚙️) → System → Blob Stores → `default`

Shows total size of cached content.

---

## Troubleshooting

### npm/pnpm Still Using Public Registry

Verify configuration:
```cmd
npm config get registry
pnpm config get registry
```

Should show: `http://MIRROR_SERVER_IP:8081/repository/npm-proxy/`

### 401 Unauthorized Errors

Enable anonymous access:
1. Settings (⚙️) → Security → Anonymous Access
2. Check: "Allow anonymous users to access the server"
3. Save

### Packages Not Caching

- Check Nexus logs: `docker-compose logs nexus`
- Verify repository is Online
- Check Maximum component age is set to `-1`
- Test connectivity: `curl http://MIRROR_SERVER_IP:8081/repository/npm-proxy/`

### Slow First Install

This is normal. Nexus fetches from npmjs.org on first request. Subsequent installs will be instant from cache.

---

## Reverting to Public Registry

### npm
```cmd
npm config delete registry
```

### pnpm
```cmd
pnpm config delete registry
```

### yarn
```cmd
yarn config delete registry
```

---

## References

- **Nexus Documentation**: https://help.sonatype.com/repomanager3
- **npm Format**: https://help.sonatype.com/en/npm-registry.html
