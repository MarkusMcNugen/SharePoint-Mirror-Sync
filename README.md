<div align="center">
  <img src="logo/logo.png" alt="SharePoint File Upload Action Logo" width="300">

  # SharePoint File Upload GitHub Action

  > 🚀 Automatically sync files from GitHub to SharePoint with intelligent change detection and Markdown-to-HTML conversion

  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
  [![Version](https://img.shields.io/badge/Version-1.0.0-blue)](https://github.com/AunalyticsManagedServices/sharepoint-file-upload-action)

  [![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?logo=github-actions&logoColor=white)](https://github.com/features/actions)
  [![Microsoft Graph](https://img.shields.io/badge/Microsoft_Graph-0078D4?logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/graph/)
  [![SharePoint](https://img.shields.io/badge/SharePoint-0078D4?logo=microsoft-sharepoint&logoColor=white)](https://www.microsoft.com/en-us/microsoft-365/sharepoint/)
</div>

## 📋 Quick Navigation

| Getting Started | Configuration | Features | Resources |
|:----------------|:--------------|:---------|:----------|
| [Overview](#-overview) | [Required Parameters](#required-parameters) | [Smart Sync](#smart-sync-with-content-hashing) | [Troubleshooting](#-troubleshooting) |
| [Quick Start](#-quick-start) | [Optional Parameters](#optional-parameters) | [Markdown Conversion](#markdown-conversion) | [Performance](#-performance) |
| [Usage Examples](#-usage-examples) | [Glob Patterns](#file-glob-patterns) | [Sync Deletion](#sync-deletion) | [Security](#-security) |
| | [Exclusion Patterns](#exclusion-patterns) | [Filename Sanitization](#filename-sanitization) | [Contributing](#-contributing) |

## 🎯 Overview

Seamlessly synchronize files from your GitHub repository to SharePoint document libraries. This action intelligently uploads only changed files, converts Markdown to SharePoint-friendly HTML, and maintains perfect sync between your repository and SharePoint.

### Why Use This Action?

| Benefit | Description |
|---------|-------------|
| 📁 **Automated Sync** | Keep SharePoint documentation current with your GitHub repository |
| ⚡ **Smart Uploads** | Only uploads new or modified files (typically skips 60-90% of files) |
| 📝 **Markdown Support** | Converts `.md` files to styled HTML with Mermaid diagram rendering |
| 🔄 **Mirror Sync** | Optional deletion of SharePoint files removed from repository |
| 📊 **Detailed Reports** | Clear statistics on uploads, skips, and failures |
| 🔒 **Enterprise Ready** | Supports GovCloud, Sites.Selected permissions, and large files |

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| **Smart Sync** | xxHash128 content comparison + upfront file & folder caching (80-95% fewer API calls, 4-6x faster) |
| **FileHash Backfill** | Automatically populates empty hashes without re-uploading files |
| **Markdown → HTML** | GitHub-flavored HTML with embedded Mermaid diagrams and rewritten links |
| **Large Files** | Chunked upload for files >4MB with automatic retry logic |
| **Folder Structure** | Maintains directory hierarchy from repository to SharePoint |
| **Special Characters** | Auto-sanitizes filenames for SharePoint compatibility |
| **Multi-Cloud** | Commercial, GovCloud, and sovereign cloud support |

## 🚀 Quick Start

### 1. Get SharePoint Credentials

Your SharePoint administrator provides:
- **Tenant ID** (Azure AD tenant identifier)
- **Client ID** (Application ID from app registration)
- **Client Secret** (App password/secret value)

**Permissions needed** (choose one):
- ✅ **Recommended**: `Sites.Selected` + grant access to specific sites ([setup guide](#sitesselected-setup))
- Alternative: `Sites.ReadWrite.All` (tenant-wide access)
- Optional: `Sites.Manage.All` (enables automatic FileHash column creation)

### 2. Add GitHub Secrets

**Settings** → **Security** → **Secrets and variables** → **Actions** → **New repository secret**

```
SHAREPOINT_TENANT_ID
SHAREPOINT_CLIENT_ID
SHAREPOINT_CLIENT_SECRET
```

### 3. Create Workflow

Create `.github/workflows/sharepoint-sync.yml`:

```yaml
name: Sync to SharePoint

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Upload to SharePoint
        uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
        with:
          file_path: "**/*"
          host_name: "yourcompany.sharepoint.com"
          site_name: "YourSite"
          upload_path: "Shared Documents/GitHub Sync"
          tenant_id: ${{ secrets.SHAREPOINT_TENANT_ID }}
          client_id: ${{ secrets.SHAREPOINT_CLIENT_ID }}
          client_secret: ${{ secrets.SHAREPOINT_CLIENT_SECRET }}
```

That's it! Push to `main` or click **Run workflow** to sync.

## ⚙️ Configuration

### Required Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `file_path` | Files to sync (glob patterns supported) | `"**/*"`, `"docs/**/*.md"` |
| `host_name` | SharePoint domain | `"company.sharepoint.com"` |
| `site_name` | SharePoint site name | `"TeamDocs"` |
| `upload_path` | Target folder path | `"Shared Documents/Reports"` |
| `tenant_id` | Azure AD Tenant ID | `${{ secrets.SHAREPOINT_TENANT_ID }}` |
| `client_id` | App Client ID | `${{ secrets.SHAREPOINT_CLIENT_ID }}` |
| `client_secret` | App Client Secret | `${{ secrets.SHAREPOINT_CLIENT_SECRET }}` |

<details>
<summary><strong>📖 Detailed Parameter Guide</strong></summary>

#### `file_path` - Which files to upload

Supports glob patterns for flexible file selection:
- `"docs/readme.md"` - Single specific file
- `"*.pdf"` - All PDFs in root directory
- `"**/*.md"` - All Markdown files in all subdirectories
- `"reports/**/*"` - Everything in reports folder

See [File Glob Patterns](#file-glob-patterns) for comprehensive pattern guide.

#### `host_name` - Your SharePoint domain

The domain part of your SharePoint URL (without `https://`):
- Commercial: `"company.sharepoint.com"`
- GovCloud: `"agency.sharepoint.us"`
- Custom: `"sharepoint.customdomain.com"`

**How to find**: Open SharePoint in browser, copy domain from URL bar.

#### `site_name` - SharePoint site name

The site collection name from your SharePoint URL:
- URL: `https://company.sharepoint.com/sites/TeamSite`
- Site name: `"TeamSite"`

**Note**: Case-sensitive in some configurations.

#### `upload_path` - Target folder in SharePoint

Format: `"DocumentLibrary/FolderPath/Subfolder"`

Common libraries:
- `"Shared Documents"` - Default library
- `"Documents"` - Alternate default
- `"Site Assets"` - Web resources

Examples:
- `"Shared Documents"` - Library root
- `"Documents/Reports"` - Reports subfolder
- `"Shared Documents/Q1/Financial"` - Multi-level path

**Folders are created automatically if they don't exist.**

#### `tenant_id`, `client_id`, `client_secret` - Azure credentials

**How to find**:
1. [Azure Portal](https://portal.azure.com) → **Azure Active Directory**
2. **App registrations** → Select your app
3. Copy **Tenant ID** and **Application (client) ID**
4. **Certificates & secrets** → Create secret → Copy **Value** immediately

⚠️ **Store all three in GitHub Secrets** - never commit to repository.

</details>

### Optional Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `file_path_recursive_match` | `false` | Enable recursive directory traversal |
| `max_retries` | `3` | Upload retry attempts (1-10) |
| `force_upload` | `false` | Skip change detection, upload all files |
| `convert_md_to_html` | `true` | Convert Markdown to HTML |
| `force_md_to_html_regeneration` | `false` | Force regenerate HTML from .md files (Mermaid/font changes) |
| `exclude_patterns` | `""` | Comma-separated exclusion patterns |
| `sync_delete` | `false` | Delete SharePoint files not in repository |
| `sync_delete_whatif` | `true` | Preview deletions without deleting |
| `max_upload_workers` | `4` | Concurrent upload workers (1-10) |
| `debug` | `false` | Enable general debug output |
| `debug_metadata` | `false` | Enable metadata-specific debug output |
| `login_endpoint` | `"login.microsoftonline.com"` | Azure AD endpoint |
| `graph_endpoint` | `"graph.microsoft.com"` | Microsoft Graph endpoint |

<details>
<summary><strong>📖 Detailed Optional Parameters</strong></summary>

#### `file_path_recursive_match` - Recursive traversal

Enable to include all subdirectories:

```yaml
# Without recursive (default):
file_path: "docs/*"        # Only files directly in docs/

# With recursive:
file_path: "docs/**/*"
file_path_recursive_match: true  # Includes all subdirectories
```

#### `max_retries` - Retry attempts

- **Default**: 3 attempts
- **When to adjust**:
  - Unstable networks: Increase to 5
  - Fast failure: Decrease to 1
- Uses exponential backoff (2s, 4s, 8s...)

```yaml
max_retries: 5    # More resilient
max_retries: 1    # Fail fast
```

#### `force_upload` - Force all uploads

- **Default**: `false` (smart sync - only changed files)
- **Use `true` when**:
  - First-time sync
  - Troubleshooting sync issues
  - Intentionally re-uploading everything

⚠️ **Warning**: Uploads ALL files regardless of changes (slower).

💡 **Tip**: Smart sync typically skips 60-90% of files.

#### `convert_md_to_html` - Markdown conversion

- **Default**: `true` (auto-convert `.md` files)
- **Output**: Styled HTML with GitHub formatting + Mermaid diagrams + rewritten internal links
- **Use `false`** when you want raw `.md` files

```yaml
convert_md_to_html: true   # README.md → README.html (styled with links rewritten)
convert_md_to_html: false  # README.md → README.md (as-is)
```

#### `force_md_to_html_regeneration` - Force HTML regeneration

- **Default**: `false` (smart sync - only convert changed .md files)
- **Requires**: `convert_md_to_html: true`
- **Use `true` when**:
  - Updated Mermaid diagram configuration
  - Changed Docker font packages
  - Modified Puppeteer settings
  - Need to rebuild all HTML without changing source .md files

**Behavior**:
- ✅ **Forces regeneration**: All `.md` files → `.html` (bypasses hash comparison)
- ✅ **Forces upload**: All regenerated `.html` files uploaded to SharePoint
- ✅ **Smart sync preserved**: Regular files (PDFs, images, etc.) still use smart sync
- ❌ **Does NOT affect**: Non-markdown files

**Difference from `force_upload`**:

| Setting | Markdown Files | Regular Files |
|---------|---------------|---------------|
| `force_upload: true` | ✅ Force convert & upload | ✅ Force upload |
| `force_md_to_html_regeneration: true` | ✅ Force convert & upload | ⚡ Smart sync (unchanged) |

💡 **Use Case**: After updating Mermaid config or Docker fonts, rebuild all documentation HTML without wasting bandwidth re-uploading unchanged binary files.

```yaml
# Rebuild all markdown HTML after Mermaid config changes
convert_md_to_html: true
force_md_to_html_regeneration: true   # Only affects .md → .html conversion

# Force EVERYTHING to upload (markdown + all other files)
force_upload: true
```

#### `exclude_patterns` - File exclusions

Comma-separated list to skip files:

```yaml
# Python projects
exclude_patterns: "*.pyc,__pycache__,.pytest_cache"

# Node.js projects
exclude_patterns: "node_modules,*.log,dist,build"

# Sensitive data
exclude_patterns: ".env,.env.local,secrets.json,*.key"
```

See [Exclusion Patterns](#exclusion-patterns) for detailed guide.

#### `sync_delete` - Mirror sync

- **Default**: `false` (safer)
- **Use `true`** to delete SharePoint files removed from repository
- ⚠️ **Always test with `sync_delete_whatif: true` first!**

```yaml
sync_delete: false  # No deletions (default)
sync_delete: true   # Enable deletion (test with whatif first!)
```

See [Sync Deletion](#sync-deletion) for complete guide.

#### `sync_delete_whatif` - Preview deletions

- **Default**: `true` (preview mode)
- **Requires**: `sync_delete: true`
- **Use `false`** only after reviewing WhatIf output

**Recommended workflow**:
1. Run with `whatif: true` (preview)
2. Review console output
3. Set `whatif: false` to actually delete

```yaml
# Step 1: Preview
sync_delete: true
sync_delete_whatif: true   # Shows what would be deleted

# Step 2: Execute (after review)
sync_delete: true
sync_delete_whatif: false  # Actually deletes
```

#### `max_upload_workers` - Concurrent uploads

- **Default**: 4 (Graph API concurrent request limit)
- **Range**: 1-10 (capped to respect API limits)
- **When to adjust**:
  - Increase to 6-8 for high-bandwidth environments
  - Decrease to 2-3 if experiencing throttling
  - Keep at 4 for most use cases

⚠️ **IMPORTANT UPCOMING CHANGE (September 30, 2025)**:
Microsoft will reduce per-app/per-user throttling limits to **HALF** the total per-tenant limit to prevent monopolization. This may impact high-volume upload scenarios. The default of 4 workers should remain safe, but monitor for increased 429 responses after September 2025.

```yaml
max_upload_workers: 4   # Default (recommended)
max_upload_workers: 8   # High-performance networks (risk throttling)
max_upload_workers: 2   # Conservative (low throttling risk)
```

#### `debug` - General debug output

- **Default**: `false`
- **Use `true`** to troubleshoot file processing decisions

**When enabled, shows**:
- File discovery and glob pattern results
- Individual file upload vs skip decisions
- Folder creation operations
- Hash comparison details
- Sync deletion path comparisons
- Relative path calculations
- Thread identifiers (`[Main]`, `[Upload-N]`, `[Convert-N]`)

**When to enable**:
- Troubleshooting why files are uploaded vs skipped
- Debugging sync deletion unexpected deletions
- Understanding base_path calculation
- Verifying exclusion pattern matches
- Tracing markdown to HTML conversion

```yaml
debug: true   # Enable general debug output
```

#### `debug_metadata` - Metadata debug output

- **Default**: `false`
- **Use `true`** only when debugging Graph API or SharePoint field issues

**When enabled, shows**:
- Graph API HTTP requests and responses
- SharePoint list item field enumeration
- Column existence checks
- FileHash column value inspection
- Internal column name mappings
- Graph API batch request/response details
- Rate limiting header analysis

⚠️ **Warning**: Produces EXTREMELY verbose output. Only enable for specific metadata troubleshooting.

```yaml
debug_metadata: true   # Enable metadata debug (very verbose!)
```

💡 **Tip**: Both `debug` and `debug_metadata` can be enabled simultaneously for maximum diagnostic detail.

#### `login_endpoint` / `graph_endpoint` - Cloud environments

**Default**: Commercial cloud (`login.microsoftonline.com`, `graph.microsoft.com`)

**Government/Sovereign clouds**:

```yaml
# US Government GCC High
login_endpoint: "login.microsoftonline.us"
graph_endpoint: "graph.microsoft.us"

# Germany
login_endpoint: "login.microsoftonline.de"
graph_endpoint: "graph.microsoft.de"

# China (21Vianet)
login_endpoint: "login.chinacloudapi.cn"
graph_endpoint: "microsoftgraph.chinacloudapi.cn"
```

⚠️ **Endpoints must match the same cloud environment.**

</details>

<details>
<summary><strong>🎯 File Glob Patterns Reference</strong></summary>

### Pattern Syntax

| Pattern | Matches | Examples |
|---------|---------|----------|
| `*.pdf` | PDFs in root | `report.pdf`, `guide.pdf` |
| `**/*.pdf` | PDFs anywhere | `docs/report.pdf`, `archive/2024/guide.pdf` |
| `docs/*` | Files directly in docs | `docs/readme.md`, `docs/config.json` |
| `docs/**/*` | All files in docs tree | `docs/api/spec.yaml`, `docs/guides/intro.md` |
| `**/*.{md,txt}` | Markdown and text files | `readme.md`, `notes.txt` |
| `**/*` | Everything | All files and folders |

### Pattern Rules

- `*` - Matches any characters in single directory
- `**` - Matches zero or more directories
- `{a,b}` - Matches either a or b
- `!pattern` - Excludes matching files
- Always use forward slashes `/` (even on Windows)

### Common Patterns

```yaml
# All Markdown documentation
file_path: "**/*.md"

# Specific folder only
file_path: "reports/*"

# Multiple file types
file_path: "**/*.{pdf,docx,xlsx}"

# Everything except tests
file_path: "**/*"
exclude_patterns: "*.test.*,__tests__"
```

</details>

<details>
<summary><strong>🚫 Exclusion Patterns Reference</strong></summary>

### How Exclusions Work

Provide comma-separated patterns in `exclude_patterns`:

```yaml
exclude_patterns: "*.log,*.tmp,__pycache__,node_modules"
```

### Pattern Types

| Type | Example | Excludes |
|------|---------|----------|
| File extension | `*.log` | All `.log` files |
| Exact name | `config.json` | Files named `config.json` |
| Directory | `__pycache__` | Any `__pycache__` directory |
| Wildcard | `temp*` | Files starting with `temp` |
| Multiple | `*.pyc,*.pyo` | Both `.pyc` and `.pyo` |

### Matching Rules

- **Basename**: `*.tmp` matches `file.tmp`
- **Path component**: `node_modules` excludes `src/node_modules/file.js`
- **Case**: Sensitive on Linux, insensitive on Windows
- **Extensions**: `log` auto-expands to `*.log`

### Common Examples

**Python projects:**
```yaml
exclude_patterns: "*.pyc,*.pyo,__pycache__,.pytest_cache,.mypy_cache"
```

**Node.js projects:**
```yaml
exclude_patterns: "node_modules,*.log,dist,build,.cache"
```

**Build artifacts:**
```yaml
exclude_patterns: "*.dll,*.exe,*.so,bin,obj"
```

**Sensitive data:**
```yaml
exclude_patterns: ".env,.env.local,secrets.json,*.key,*.pem"
```

**Temp/hidden files:**
```yaml
exclude_patterns: "*.tmp,*.bak,.DS_Store,Thumbs.db,.git"
```

</details>

## 📚 Usage Examples

<details>
<summary><strong>Example 1: Sync Entire Repository</strong></summary>

```yaml
- name: Checkout Repository
  uses: actions/checkout@v4

- name: Sync All Files
  uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    file_path: "**/*"
    file_path_recursive_match: true
    host_name: "company.sharepoint.com"
    site_name: "RepoDocumentation"
    upload_path: "Shared Documents/GitHub Mirror"
    tenant_id: ${{ secrets.SHAREPOINT_TENANT_ID }}
    client_id: ${{ secrets.SHAREPOINT_CLIENT_ID }}
    client_secret: ${{ secrets.SHAREPOINT_CLIENT_SECRET }}
```

</details>

<details>
<summary><strong>Example 2: Markdown Documentation Only</strong></summary>

```yaml
- name: Checkout Repository
  uses: actions/checkout@v4

- name: Sync Documentation
  uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    file_path: "**/*.md"
    file_path_recursive_match: true
    convert_md_to_html: true  # Converts to styled HTML with rewritten links
    host_name: "company.sharepoint.com"
    site_name: "KnowledgeBase"
    upload_path: "Documentation"
    tenant_id: ${{ secrets.SHAREPOINT_TENANT_ID }}
    client_id: ${{ secrets.SHAREPOINT_CLIENT_ID }}
    client_secret: ${{ secrets.SHAREPOINT_CLIENT_SECRET }}
```

</details>

<details>
<summary><strong>Example 3: Exclude Build Artifacts</strong></summary>

```yaml
- name: Checkout Repository
  uses: actions/checkout@v4

- name: Sync Clean Repository
  uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    file_path: "**/*"
    file_path_recursive_match: true
    exclude_patterns: "*.pyc,__pycache__,node_modules,*.log,.git,dist,build"
    host_name: "company.sharepoint.com"
    site_name: "DevDocs"
    upload_path: "Projects/MyApp"
    tenant_id: ${{ secrets.SHAREPOINT_TENANT_ID }}
    client_id: ${{ secrets.SHAREPOINT_CLIENT_ID }}
    client_secret: ${{ secrets.SHAREPOINT_CLIENT_SECRET }}
```

</details>

<details>
<summary><strong>Example 4: GovCloud Deployment</strong></summary>

```yaml
- name: Checkout Repository
  uses: actions/checkout@v4

- name: Sync to GovCloud
  uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    file_path: "compliance/**/*"
    host_name: "agency.sharepoint.us"
    site_name: "Compliance"
    upload_path: "FY2025"
    tenant_id: ${{ secrets.SHAREPOINT_TENANT_ID }}
    client_id: ${{ secrets.SHAREPOINT_CLIENT_ID }}
    client_secret: ${{ secrets.SHAREPOINT_CLIENT_SECRET }}
    login_endpoint: "login.microsoftonline.us"
    graph_endpoint: "graph.microsoft.us"
```

</details>

<details>
<summary><strong>Example 5: Mirror Sync with Cleanup</strong></summary>

```yaml
- name: Checkout Repository
  uses: actions/checkout@v4

- name: Full Sync with Deletion
  uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    file_path: "docs/**/*"
    file_path_recursive_match: true
    sync_delete: true              # Enable deletion
    sync_delete_whatif: true       # Preview first (safe)
    host_name: "company.sharepoint.com"
    site_name: "Documentation"
    upload_path: "Shared Documents/Docs"
    tenant_id: ${{ secrets.SHAREPOINT_TENANT_ID }}
    client_id: ${{ secrets.SHAREPOINT_CLIENT_ID }}
    client_secret: ${{ secrets.SHAREPOINT_CLIENT_SECRET }}
```

</details>

<details>
<summary><strong>Example 6: High-Performance Sync for Large Repositories</strong></summary>

```yaml
name: Sync Documentation
on: [push]

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Sync to SharePoint with Optimized Performance
        uses: AunalyticsManagedServices/sharepoint-file-upload-action@v5
        with:
          site_name: 'TeamSite'
          host_name: ${{ secrets.SHAREPOINT_HOST }}
          tenant_id: ${{ secrets.SHAREPOINT_TENANT_ID }}
          client_id: ${{ secrets.SHAREPOINT_CLIENT_ID }}
          client_secret: ${{ secrets.SHAREPOINT_CLIENT_SECRET }}
          upload_path: 'Documents/LargeDocumentation'
          file_path: 'docs/**/*'
          file_path_recursive_match: true
          force_upload: 'false'  # Enable smart sync + caching
```

**Expected Output:**
```
============================================================
[1/5] CONFIGURATION
============================================================
[✓] Smart sync mode: Enabled (skip unchanged files)
[✓] Markdown conversion: Enabled (with Mermaid diagrams)
[✓] Sync deletion: Disabled (no files will be removed)
[✓] Parallel processing: 4 workers

============================================================
[2/5] FILE DISCOVERY
============================================================
[*] Working directory: /github/workspace
[*] Pattern: docs/**/* (recursive)
[✓] Found 2,500 files to process

============================================================
[3/5] SHAREPOINT CONNECTION
============================================================
[*] Connecting to SharePoint...
[✓] Connected: Documents/LargeDocumentation
[*] Verifying FileHash column...
[✓] FileHash column available for hash-based comparison

============================================================
[4/5] BUILDING METADATA CACHE
============================================================
[*] Building SharePoint metadata cache for: Documents/LargeDocumentation

[CACHE] SharePoint Metadata Cache:
   - Total files cached:          2,500
   - Total folders cached:          150
   - Files with FileHash:       2,400/2,500
   - Files with list_item_id:   2,500/2,500

============================================================
[5/5] FILE PROCESSING
============================================================
[*] Uploading files...
...
============================================================
[✓] SYNC PROCESS COMPLETED
============================================================
[STATS] Sync Statistics:
   - Files skipped (unchanged):  2,400
   - Files updated:                100

[COMPARE] File Comparison Methods:
   - Compared by hash:           2,500 (100.0%)

[CACHE] Cache Performance:
   - Cache hits:                 2,400
   - Cache misses:                 100
   - Cache efficiency:            96.0% (API calls avoided)

[DATA] Transfer Summary:
   - Data uploaded:   12.5 MB
   - Data skipped:    237.8 MB (2,400 files not re-uploaded)
   - Sync efficiency: 95.0% (bandwidth saved by smart sync)
```

**Performance:** 2,500 files processed in ~2 minutes (vs ~15 minutes without caching)

</details>

<details>
<summary><strong>Example 7: Force Markdown HTML Regeneration (Mermaid/Font Changes)</strong></summary>

```yaml
- name: Checkout Repository
  uses: actions/checkout@v4

- name: Rebuild All Markdown HTML
  uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    file_path: "**/*.md"
    file_path_recursive_match: true
    convert_md_to_html: true
    force_md_to_html_regeneration: true  # Force regenerate ALL HTML
    host_name: "company.sharepoint.com"
    site_name: "Documentation"
    upload_path: "Shared Documents/Docs"
    tenant_id: ${{ secrets.SHAREPOINT_TENANT_ID }}
    client_id: ${{ secrets.SHAREPOINT_CLIENT_ID }}
    client_secret: ${{ secrets.SHAREPOINT_CLIENT_SECRET }}
```

**Use Cases:**
- ✅ Updated Mermaid diagram configuration (`mermaid-config.json`)
- ✅ Changed Docker font packages for better diagram rendering
- ✅ Modified Puppeteer settings (`puppeteer-config.json`)
- ✅ Need to rebuild all documentation HTML without touching source `.md` files

**What It Does:**
- ✅ **All .md files**: Regenerated as HTML and uploaded (bypasses hash check)
- ✅ **Regular files**: Smart sync preserved (only uploads if changed)
- ⚡ **Result**: Rebuilt documentation without wasting bandwidth on unchanged binaries

**Difference from `force_upload`:**

```yaml
# Option 1: Rebuild ONLY markdown HTML (efficient)
force_md_to_html_regeneration: true  # 50 MB markdown HTML regenerated
                                      # 500 MB images/PDFs use smart sync

# Option 2: Force upload EVERYTHING (wasteful)
force_upload: true                    # 550 MB total re-uploaded
                                      # Images/PDFs re-uploaded unnecessarily
```

</details>

## 🔧 Advanced Features

### Parallel Processing for Maximum Performance

The action uses **parallel processing by default** for maximum performance with all file operations executing concurrently.

**Performance Improvements:**
- ⚡ **4-6x faster** overall sync time for typical repositories
- 🚀 **5-10x faster** uploads for repos with 50+ files
- 📊 **10-20x fewer** API calls (batch metadata updates and caching)

<details>
<summary><strong>📊 Detailed Parallel Processing Information</strong></summary>

### What Runs in Parallel

| Operation | Workers | Description |
|-----------|---------|-------------|
| **File Uploads** | 4 (configurable) | Multiple files upload simultaneously |
| **Markdown Conversion** | 4 (internal) | Multiple Mermaid diagrams render concurrently |
| **Metadata Updates** | Batched (20/request) | Graph API batch endpoint reduces API calls |

### Auto-Configuration

The action automatically detects your system specifications and configures optimal worker counts:
- Upload workers: Default 4 (respects Graph API concurrent limit)
- Hash workers: Detects CPU count for optimal parallel hashing
- Markdown workers: Fixed at 4 (balances performance vs memory)

### Thread Safety

All parallel operations are fully thread-safe with:
- Sequential console output (no garbled messages)
- Atomic statistics updates (accurate counts)
- Proper rate limiting coordination
- Batch queue for efficient metadata updates

### Console Output

```
============================================================
[✓] SYSTEM CONFIGURATION
============================================================
CPU Cores Available:       8
Upload Workers:            4 (concurrent uploads)
Markdown Workers:          4 (parallel conversion)
Batch Metadata Updates:    Enabled (20 items/batch)
============================================================
```

### Configuration

```yaml
# Use defaults (recommended for most cases)
- uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    # ... other parameters

# Adjust for high-performance environments
- uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    max_upload_workers: 8      # Increase concurrent uploads
    # ... other parameters

# Conservative settings (minimize throttling risk)
- uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    max_upload_workers: 2      # Reduce concurrent requests
    # ... other parameters
```

### Backward Compatibility

Parallel processing is fully compatible with existing workflows:
- ✅ Same console output format
- ✅ Same statistics structure
- ✅ Same error handling and retry logic
- ✅ No configuration changes required

### Performance Benchmarks

| Scenario | Sequential (Old) | Parallel (New) | Improvement |
|----------|-----------------|----------------|-------------|
| 100 files, 50 MB | 250s (4 min) | 40-60s (1 min) | **4-6x faster** |
| 500 files, 200 MB | 1200s (20 min) | 180-300s (3-5 min) | **4-6x faster** |

</details>

### Smart Sync with Content Hashing

The action uses **xxHash128** for lightning-fast change detection:

**How it works:**
1. Calculates hash for local file (3-6 GB/s processing speed)
2. For converted markdown: Uses **source `.md` file hash** for comparison
3. Compares with `FileHash` column in SharePoint
4. Skips file if hash matches (unchanged)
5. Uploads only new or modified files

**Fallback:** Uses file size comparison if hash unavailable.

**Benefits:**
- ⚡ Typically skips 60-90% of files
- 💾 Saves bandwidth and time
- 🎯 More reliable than timestamps (especially in Docker/CI)
- 📊 Detailed statistics show comparison methods and hash operations
- 🔄 Automatic FileHash backfill for files with empty hashes

**Statistics Tracked:**
- **Comparison Methods**: Shows how many files were compared using hash vs size
- **FileHash Operations**: Tracks new hashes saved, hashes updated, hash matches, and backfills
- **Cache Performance**: Displays cache hits, misses, and efficiency percentage
- **Efficiency Metrics**: Displays bandwidth saved and skip rate

**Example Output:**
```
============================================================
[1/5] CONFIGURATION
============================================================
[✓] Smart sync mode: Enabled (skip unchanged files)
[✓] Markdown conversion: Enabled (with Mermaid diagrams)
[✓] Sync deletion: Disabled (no files will be removed)
[✓] Parallel processing: 4 workers

============================================================
[2/5] FILE DISCOVERY
============================================================
[*] Working directory: /github/workspace
[*] Pattern: **/* (recursive)
[✓] Found 953 files to process

============================================================
[3/5] SHAREPOINT CONNECTION
============================================================
[*] Connecting to SharePoint...
[✓] Connected: Shared Documents/Docs
[*] Verifying FileHash column...
[✓] FileHash column available for hash-based comparison

============================================================
[4/5] BUILDING METADATA CACHE
============================================================
[*] Building SharePoint metadata cache for: Shared Documents/Docs

[CACHE] SharePoint Metadata Cache:
   - Total files cached:            953
   - Total folders cached:          23
   - Files with FileHash:           682/953
   - Files with list_item_id:       953/953

============================================================
[5/5] FILE PROCESSING
============================================================
[*] Processing markdown files...
[✓] Verified or converted 12 markdown files

[*] Uploading files...

[*] Checking for orphaned files...
[✓] No orphaned files to delete

============================================================
[✓] SYNC PROCESS COMPLETED
============================================================
[STATS] Sync Statistics:
   - New files uploaded:          255
   - Files updated:               129
   - Files skipped (unchanged):   569
   - Total files processed:       953

[COMPARE] File Comparison Methods:
   - Compared by hash:            682 (71.7%)
   - Compared by size:            271 (28.3%)

[HASH] FileHash Column Statistics:
   - New hashes saved:            255
   - Hashes updated:              129
   - Hash matches (skipped):      569
   - Hashes backfilled:           45

[CACHE] Cache Performance:
   - Cache hits:                  698
   - Cache misses:                255
   - Cache efficiency:            73.2% (API calls avoided)

[DATA] Transfer Summary:
   - Data uploaded:   93.7 MB
   - Data skipped:    177.5 MB (569 files not re-uploaded)
   - Sync efficiency: 65.4% (bandwidth saved by smart sync)
============================================================
```

<details>
<summary><strong>🔄 FileHash Backfill (Automatic)</strong></summary>

Automatically populates empty FileHash values without re-uploading files.

### Problem Solved

- Files uploaded before FileHash column existed have empty hash values
- These files fall back to less reliable size comparison
- Traditional solution: Force re-upload entire repository (slow, wasteful)

### How Backfill Works

1. Detects files with empty FileHash during comparison
2. Confirms file is unchanged via size match
3. Calculates hash from local file content
4. Updates FileHash field directly (no file upload)
5. Future checks use hash-based comparison

### Benefits

- 📈 Gradual migration to 100% hash-based comparison
- 💾 **99.8% bandwidth savings** vs force upload approach
- ⚡ **90% time savings** (PATCH request vs full upload)
- 🔄 Always-on feature (no configuration needed)

### Statistics Tracked

- `hash_backfilled` - Files with hash populated (no upload)
- `hash_empty_found` - Files discovered with empty hash
- `hash_backfill_failed` - Failed backfill attempts
- `hash_column_unavailable` - Files checked when column doesn't exist

### Example Output

```
[HASH] FileHash Column Statistics:
   - New hashes saved:         1
   - Hash matches (skipped):   152  ← Increased from 107
   - Hashes backfilled:        45   ← NEW! Automatic backfill
   - Empty hash found:         0    ← Drops to 0 after first run
```

### Performance Impact

- One-time PATCH request per file with empty hash (~200ms each)
- Example: 200 files with empty hash = 40 seconds vs 6.7 minutes for re-upload
- **Time savings: 90%** | **Bandwidth savings: 99.8%**

</details>

### Markdown Conversion

Converts `.md` files to GitHub-flavored HTML with embedded styling:

**Supported Features:**
- ✅ Headers (H1-H6) with anchors
- ✅ Tables with styling
- ✅ Code blocks with syntax highlighting
- ✅ Task lists with checkboxes
- ✅ Blockquotes
- ✅ Links and images
- ✅ **Internal link rewriting** (relative links → SharePoint folder URLs)
- ✅ **Mermaid diagrams with automatic sanitization** (rendered as embedded SVG)

<details>
<summary><strong>🔗 Automatic Internal Link Rewriting</strong></summary>

**New in v4.2.0**: Automatically converts relative file links to SharePoint folder URLs for easy navigation.

### How It Works

When converting `.md` to `.html`, the action rewrites **all local file links** (not just .md files) to point to their parent folders in SharePoint:

**Before Conversion (in .md file):**
```markdown
[Installation Guide](../setup/install.md)
[Update Script](./scripts/Update-System.ps1)
[API Reference](api/reference.md#endpoints)
[Config File](./config.json)
```

**After Conversion (in .html file):**
```html
<a href="https://company.sharepoint.com/sites/Docs/Shared%20Documents/setup" title="View folder containing install.html">Installation Guide</a>
<a href="https://company.sharepoint.com/sites/Docs/Shared%20Documents/scripts" title="View folder containing Update-System.ps1">Update Script</a>
<a href="https://company.sharepoint.com/sites/Docs/Shared%20Documents/api" title="View folder containing reference.html#endpoints">API Reference</a>
<a href="https://company.sharepoint.com/sites/Docs/Shared%20Documents" title="View folder containing config.json">Config File</a>
```

### Link Behavior

**Links point to parent folders**, not files directly:
- ✅ Clicking opens the SharePoint folder containing the file
- ✅ Users see all related files in the folder (context)
- ✅ Hover tooltip shows which file the link refers to
- ✅ Works for **ALL file types** (.ps1, .py, .json, .md, images, etc.)

### What Gets Rewritten

| Link Type | Rewritten? | Example |
|-----------|------------|---------|
| Markdown files | ✅ Yes | `../README.md` → Parent folder URL |
| PowerShell scripts | ✅ Yes | `./Update.ps1` → Parent folder URL |
| Python scripts | ✅ Yes | `script.py` → Parent folder URL |
| Config files | ✅ Yes | `config.json` → Parent folder URL |
| Any file with extension | ✅ Yes | `file.ext` → Parent folder URL |
| Folders | ✅ Yes | `../docs/` → Folder URL |
| Anchor links | ✅ Yes | `#section` → Preserved in tooltip |
| External links | ❌ No | `https://example.com` → Unchanged |
| Image links | ❌ No | `![logo](logo.png)` → Unchanged |

### Benefits

- 📂 **Folder-based navigation** - Users see file context
- 🔗 **No broken links** from relative paths
- 📝 **Works for all file types** - Scripts, configs, markdown, etc.
- ℹ️ **Helpful tooltips** - Shows which file is in the folder
- ✅ **Automatic** - No manual link updates needed

### Notes

- Only applies to converted markdown (`.md` → `.html`)
- Raw markdown uploads (when `convert_md_to_html: false`) preserve original links
- SharePoint folder structure must match repository structure
- Links open folder views where users can find the referenced file

</details>

<details>
<summary><strong>🔧 Smart Mermaid Sanitization</strong></summary>

The action uses a **smart two-phase approach** to Mermaid diagram rendering:

1. **First attempt**: Try rendering the original diagram as-is (preserves perfect fidelity)
2. **Second attempt**: If syntax errors occur, automatically sanitize and retry
3. **Fallback**: If both attempts fail, show diagram as code block

This ensures valid diagrams render unchanged while problematic diagrams get automatic fixes.

### Sanitization Strategy

**Only sanitizes when needed:**
- ✅ Detects specific issues before applying fixes
- ✅ Avoids unnecessary transformations
- ✅ Prevents double-escaping already-sanitized entity codes
- ✅ Validates each rule before execution

### Before Sanitization (Would Fail)
````markdown
```mermaid
graph TD
    A[Server #1] --> B{Version |No| Skip}
    B -->|"Proceed; Deploy"| C[Deploy]
    C --> end
    D[Post Action<br/>(Restart)]
```
````

### After Sanitization (Will Render)

````markdown
```mermaid
graph TD
    A[Server &#35;1] --> B{Version &#124;No&#124; Skip}
    B -->|'Proceed&#59; Deploy'| C[Deploy]
    C --> End
    D[Post Action<br>(Restart)]
```
````

### What Gets Fixed

| Issue | Before | After | Why |
|-------|--------|-------|-----|
| Special chars in nodes | `[Server #1]` | `[Server &#35;1]` | `#` breaks parser |
| Pipes in diamonds | `{Version \|No\| Skip}` | `{Version &#124;No&#124; Skip}` | Unescaped pipes break syntax |
| Semicolons in labels | `\|"text; more"\|` | `\|'text&#59; more'\|` | Semicolons used as line breaks |
| Double quotes | `\|"text"\|` | `\|'text'\|` | Prevents syntax errors |
| Reserved words | `end` | `End` | Lowercase breaks flowcharts |
| XHTML tags | `<br/>` | `<br>` | Mermaid only supports `<br>` |

### Special Characters Escaped

All special characters are escaped using entity codes to prevent syntax errors:

| Character | Entity Code | Why Escaped |
|-----------|-------------|-------------|
| `&` | `&#38;` | Can break parser |
| `#` | `&#35;` | Mistaken for comments |
| `%` | `&#37;` | Reserved character |
| `\|` | `&#124;` | Used for node/edge delimiters |
| `;` | `&#59;` | Alternative line break syntax |
| `"` | `'` | Converted to single quote |

**Supported Node Shapes:**
- Square brackets `[text]` - Rectangles
- Parentheses `(text)`, `((text))` - Rounded rectangles
- Curly braces `{text}`, `{{text}}` - Diamond/rhombus nodes
- Trapezoids `[/text\]`, `[\text/]` - Trapezoid shapes

All shapes automatically sanitized for special characters. No manual fixes needed!

### Console Output

When sanitization is applied, you'll see which fixes were needed:

```
[*] Mermaid conversion failed, attempting with sanitization...
    File: docs/workflow.md
[SANITIZE] Applied fixes: special-chars-in-brackets, reserved-word-end, special-chars-in-edge-labels
[OK] Mermaid diagram converted successfully after sanitization
```

This helps identify which diagrams had issues and what was fixed.

</details>

<details>
<summary><strong>💡 Smart Sync for Converted Markdown</strong></summary>

Converted markdown files use **source `.md` file hash** for comparison to prevent unnecessary re-uploads.

**Why?**
- Mermaid CLI may generate SVGs with varying internal IDs between renders
- Content and visual appearance remain identical, but generated HTML hash changes
- Using source `.md` file hash ensures consistent comparison

**How It Works:**
1. **Calculate Source Hash**: Hash the original `.md` file before conversion
2. **Convert to HTML**: Create styled HTML with Mermaid diagrams
3. **Early Skip Check**: Query SharePoint to see if `.html` file exists with matching source hash
4. **Smart Decision**:
   - If source `.md` unchanged → Skip conversion entirely (95-98% time savings)
   - If source `.md` changed → Convert and upload
5. **Save Hash**: Store source `.md` hash in FileHash column for the `.html` file

**Result:**
- ✅ Markdown files with unchanged content won't re-upload
- ✅ Mermaid diagram variations don't trigger false positives
- ✅ FileHash column populated with deterministic source hash
- ✅ Perfect smart sync for documentation repositories
- ✅ Avoids expensive conversion (3-10s per file) when not needed

**Performance Impact:**
- First run: Convert all markdown files
- Subsequent runs: Only convert changed files (typically 5-10%)
- 100 unchanged .md files: ~10 seconds vs 5-16 minutes (sequential conversion)

**Note:** Regular files (non-markdown) still use hash-based comparison of actual file content for maximum accuracy.

</details>

<details>
<summary><strong>🐛 Debug Mode</strong></summary>

Enable detailed logging for troubleshooting and diagnostics.

### General Debug (`debug: true`)

- Shows file processing decisions
- Displays path calculations and folder operations
- Traces sync deletion comparisons
- Includes thread identifiers for parallel operations
- Logs hash comparison details

### Metadata Debug (`debug_metadata: true`)

- Shows Graph API HTTP requests/responses
- Displays SharePoint column inspection
- Tracks FileHash value lifecycle
- Logs rate limiting headers
- Shows batch update details

### Thread Identifiers (when `debug: true`)

- `[Main]` - Main orchestration thread
- `[Upload-N]` - Upload worker threads (N = worker number)
- `[Convert-N]` - Markdown conversion worker threads
- Helps track which thread produced each log line

### Configuration

```yaml
# General debug only (recommended for most troubleshooting)
- uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    # ... other parameters
    debug: true

# Full debug mode (maximum verbosity)
- uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    # ... other parameters
    debug: true
    debug_metadata: true  # Very verbose!
```

### Example Output with Debug

```
[Main] [?] Checking if file exists in SharePoint: docs/api/README.html
[Main] [#] Source .md file hash: a1b2c3d4... (will be used for .html file)
[Upload-1] [OK] File uploaded: assets/image.png
[Convert-2] [MD] Converting markdown to HTML: docs/guide.md
[Main] [=] File unchanged (hash match): docs/setup.html
[Main] [#] Backfilling empty FileHash for unchanged file: legacy/old.html
```

### Use Cases

- Troubleshooting unexpected uploads or skips
- Debugging sync deletion identifying wrong files
- Understanding folder structure preservation
- Verifying exclusion patterns work correctly
- Analyzing performance bottlenecks
- Investigating Graph API permission issues

</details>

### Sync Deletion

**Mirror sync** - automatically removes SharePoint files that no longer exist in your repository.

**Use Cases:**
- 🔄 Renamed files (old name removed)
- 🗑️ Deleted files (removed from SharePoint)
- 📁 Restructured folders (cleanup old locations)
- 📝 Obsolete documentation (auto-removal)

**Safety Features:**
1. **Explicit opt-in** (`sync_delete: false` by default)
2. **WhatIf mode** (preview deletions before executing)
3. **Scoped deletion** (only within `upload_path`)
4. **Statistics** (audit trail of deletions)

**Configuration:**

| Parameter | Default | Purpose |
|-----------|---------|---------|
| `sync_delete` | `false` | Enable deletion feature |
| `sync_delete_whatif` | `true` | Preview mode (safe default) |

**Recommended Workflow:**

**Step 1: Preview (Safe)**
```yaml
sync_delete: true
sync_delete_whatif: true  # Shows what would be deleted
```

**Console Output:**
```
============================================================
[1/5] CONFIGURATION
============================================================
[✓] Smart sync mode: Enabled (skip unchanged files)
[✓] Markdown conversion: Enabled (with Mermaid diagrams)
[!] Sync deletion: Enabled in WHATIF mode (preview only)
[✓] Parallel processing: 4 workers

...

============================================================
[5/5] FILE PROCESSING
============================================================
[*] Uploading files...

[*] Checking for orphaned files...
[!] Found 3 orphaned files (WhatIf mode - no actual deletions will occur)

File Deleted (WhatIf): old-readme.md
File Deleted (WhatIf): deprecated/guide.md
File Deleted (WhatIf): archive/notes.txt

[✓] WhatIf: Would delete 3 orphaned files from SharePoint
```

**Step 2: Execute (After Review)**
```yaml
sync_delete: true
sync_delete_whatif: false  # Actually deletes files
```

**Console Output:**
```
============================================================
[1/5] CONFIGURATION
============================================================
[✓] Smart sync mode: Enabled (skip unchanged files)
[✓] Markdown conversion: Enabled (with Mermaid diagrams)
[!] Sync deletion: Enabled (will delete orphaned files)
[✓] Parallel processing: 4 workers

...

============================================================
[5/5] FILE PROCESSING
============================================================
[*] Uploading files...

[*] Checking for orphaned files...
[!] Found 3 orphaned files to delete from SharePoint

File Deleted: old-readme.md
File Deleted: deprecated/guide.md
File Deleted: archive/notes.txt

[✓] Successfully deleted 3 orphaned files from SharePoint
```

⚠️ **Important:** Always test with WhatIf first. Deleted files may be in SharePoint recycle bin (depends on configuration).

💡 **Best Practice:** Use in scheduled workflows for automated cleanup.

#### 🔍 Troubleshooting Sync Deletion

If sync deletion is marking unexpected files for deletion, enable **debug mode** to see detailed comparison:

```yaml
- name: Sync with Debug Output
  uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    # ... your parameters
    sync_delete: true
    sync_delete_whatif: true
    debug: true  # Enable detailed debug logging
```

**Debug output shows:**
- Full list of SharePoint files found
- Full list of local files in sync set
- Path-by-path comparison results
- Whether items are files or folders
- Exact reason each file is orphaned or matched

This helps diagnose path mismatches, folder structure issues, or markdown conversion problems.

### Filename Sanitization

SharePoint restricts certain characters. The action auto-converts them:

| Invalid | Replacement |
|---------|-------------|
| `#`, `%`, `&` | Fullwidth Unicode equivalents |
| `:`, `<`, `>`, `?`, `\|` | Fullwidth Unicode equivalents |

Reserved names (CON, PRN, AUX, NUL, etc.) are prefixed with underscore.

**Example:** `file:name#test.md` → `file：name＃test.md`

<details>
<summary><strong>⚡ Performance Optimization: Metadata Caching</strong></summary>

### How It Works

The action automatically builds a comprehensive cache of all SharePoint file AND folder metadata in a single bulk operation, eliminating 80-95% of API calls.

**Traditional Approach:**
```
For each of 100 files (in 10 folders):
  - Check if folder exists (1 API call per folder) = 10 calls
  - Query SharePoint for file metadata (1 API call per file) = 100 calls
  - Compare with local file
  - Upload if changed

Total: 110+ API calls
```

**Optimized Approach:**
```
1. Build cache: Query all files and folders once (10-20 API calls)
2. For each of 100 files (in 10 folders):
   - Lookup folder from cache (instant, 0 API calls)
   - Lookup file metadata from cache (instant, 0 API calls)
   - Compare with local file
   - Upload if changed

Total: 10-20 API calls (80-95% reduction)
```

### Performance Gains

| Repository Size | API Calls (Old) | API Calls (New) | Time Saved | Speedup |
|----------------|-----------------|-----------------|------------|---------|
| 100 files (10 folders) | 110 calls | 20 calls | ~25 seconds | 4-6x faster |
| 500 files (25 folders) | 550 calls | 50 calls | ~2.5 minutes | 5-7x faster |
| 1000 files (50 folders) | 1100 calls | 100 calls | ~5 minutes | 6-10x faster |

### When Caching is Used

**Automatic activation:**
- ✅ Smart sync mode (default) - Caches file metadata for comparisons
- ✅ Sync deletion enabled - Caches file list for deletion comparison
- ❌ Force upload mode only - Cache not needed (no comparisons)

**Error handling:**
- Gracefully falls back to individual API queries if cache build fails
- No action required - works transparently

### Console Output

When cache is built successfully:
```
[*] Building SharePoint metadata cache for: Documents/Reports

[CACHE] SharePoint Metadata Cache:
   - Total files cached:            456
   - Total folders cached:          18
   - Files with FileHash:           398/456
   - Files with list_item_id:       456/456
```

During file processing (when debug enabled):
```
[CACHE HIT] Folder found in cache: docs/2024
[CACHE HIT] Found docs/2024/README.html in cache
[=] File unchanged (cached hash match): docs/2024/README.html
```

### Technical Details

**Graph API Query:**
```
GET /drives/{drive-id}/items/{folder-id}/children?$expand=listItem($expand=fields($select=FileHash,FileSizeDisplay,FileLeafRef))
```

**What's Cached:**
- **Files:** Drive item metadata (id, name, size), List item ID, FileHash column values
- **Folders:** Drive item IDs and folder names for instant folder existence checks

**Benefits:**
- Dramatically faster sync operations
- Better rate limiting compliance
- Lower throttling risk
- Scales better with large repositories
- Reused for both comparison and sync deletion

</details>

## 🔒 Security

### Best Practices

- ✅ **Never commit secrets** to repository
- ✅ **Use GitHub Secrets** for all credentials
- ✅ **Use Sites.Selected** for granular access control (recommended)
- ✅ **Limit permissions** to only required sites
- ✅ **Review app permissions** regularly
- ✅ **Use branch protection** to control workflow triggers
- ✅ **Rotate client secrets** before expiration

### Sites.Selected Setup

**Enhanced security** - grant app access to specific sites only (vs. tenant-wide access).

<details>
<summary><strong>🔐 Sites.Selected Configuration Guide</strong></summary>

#### 1. Configure App Registration

In Azure AD, grant **Sites.Selected** permission:
- API: Microsoft Graph
- Permission: Sites.Selected (Application)
- Type: Application permission

#### 2. Grant Site Access

**Method A: Graph API**
```bash
# Get site ID
GET https://graph.microsoft.com/v1.0/sites/{hostname}:/sites/{sitename}

# Grant app access (requires admin permission)
POST https://graph.microsoft.com/v1.0/sites/{site-id}/permissions
{
  "roles": ["write"],
  "grantedToIdentities": [{
    "application": {
      "id": "{client-id}",
      "displayName": "{app-name}"
    }
  }]
}
```

**Method B: PowerShell (PnP)**
```powershell
# Install PnP PowerShell
Install-Module -Name PnP.PowerShell

# Connect to site
Connect-PnPOnline -Url "https://{tenant}.sharepoint.com/sites/{sitename}" -Interactive

# Grant permissions
Grant-PnPAzureADAppSitePermission `
  -AppId "{client-id}" `
  -DisplayName "{app-name}" `
  -Permissions Write

# Verify
Get-PnPAzureADAppSitePermission
```

#### 3. Use in Workflow

No workflow changes needed! Works identically:

```yaml
- uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    site_name: "TeamSite"  # Must match granted site
    # ... other parameters (same as before)
```

**Benefits:**
- ✅ App can only access specified sites
- ✅ Compliance-friendly granular control
- ✅ Clear audit trail
- ✅ No workflow changes required

</details>

## 🐛 Troubleshooting

<details>
<summary><strong>Common Issues & Solutions</strong></summary>

### 1. Authentication Failed

**Error:** `(401) Unauthorized`

**Solutions:**
- ✅ Verify Client ID and Secret are correct
- ✅ Check app has required permissions:
  - `Sites.ReadWrite.All` OR `Sites.Selected`
  - `Sites.Manage.All` (optional, for FileHash column)
- ✅ For Sites.Selected: Verify app granted access to site
- ✅ Ensure Tenant ID matches SharePoint instance

### 2. File Not Found

**Error:** `No files matched pattern`

**Solutions:**
- ✅ Check glob pattern syntax
- ✅ Enable `file_path_recursive_match: true` for nested directories
- ✅ Add debug step: `run: ls -la` to verify files exist
- ✅ Check `exclude_patterns` isn't excluding your files

### 3. Upload Timeout

**Error:** `Operation timed out`

**Solutions:**
- ✅ Large files automatically retry (check logs)
- ✅ Consider splitting large uploads into multiple actions
- ✅ Increase `max_retries` for unstable networks
- ✅ Check network connectivity

### 4. Markdown Conversion Failed

**Error:** `Mermaid conversion failed`

**Solutions:**
- ✅ Verify Mermaid syntax (use [Mermaid Live Editor](https://mermaid.live))
- ✅ Simplify complex diagrams
- ✅ Check for unsupported Mermaid features
- ✅ Automatic sanitization handles most issues

### 5. Permission Denied (Column Creation)

**Error:** `Access denied creating FileHash column`

**Solutions:**
- ✅ This is expected if app lacks `Sites.Manage.All`
- ✅ Action automatically falls back to file size comparison
- ✅ Grant `Sites.Manage.All` for hash-based comparison (optional)

### 6. Throttling Issues (429 Errors)

**Error:** `Rate limited (429)`

**Solutions:**
- ✅ Decrease `max_upload_workers` to 2-3
- ✅ Increase retry delays
- ✅ Default of 4 workers should be safe for most cases
- ⚠️ Monitor for increased throttling after September 30, 2025 (Microsoft reducing limits)

### 7. Slow Performance Despite Caching

**Problem:** Sync still slow even with caching enabled

**Solutions:**

1. **Verify cache is being used**:
   - Check console for "[CACHE] SharePoint Metadata Cache" section
   - Should show total files cached and FileHash availability

2. **Check cache build failure**:
   - Look for "[!] Warning: Failed to build SharePoint cache"
   - Falls back to individual API queries (slower)
   - Possible causes: network timeout, permissions, very large folder

3. **Verify you're in smart sync mode**:
   ```yaml
   force_upload: 'false'  # Must be false for caching benefits
   ```

4. **Enable debug mode to see detailed cache usage**:
   ```yaml
   debug: 'true'
   ```
   Look for `[CACHE HIT]` messages showing cache lookups

5. **Large repositories (10,000+ files)**:
   - Cache building may take 30-60 seconds
   - Overall still much faster than individual queries
   - Consider splitting into multiple upload paths

### Debug Steps

Add before upload step to inspect files:

```yaml
- name: Debug - List Files
  run: |
    echo "Files matching pattern:"
    ls -la docs/**/*.md
```

Enable debug logging:
```yaml
- name: Upload with Debug
  uses: AunalyticsManagedServices/sharepoint-file-upload-action@v4
  with:
    # ... parameters
    debug: true
```

</details>

## 📊 Performance

### Optimization Tips

1. **Use specific glob patterns** - `"docs/*.md"` vs. `"**/*"`
2. **Enable smart sync** (default) - skips unchanged files
3. **Upload frequently** - minimizes changes per sync
4. **Organize files logically** - enables targeted patterns
5. **Schedule off-peak** - for large syncs

**Hashing Performance:** xxHash128 processes files at 3-6 GB/s (10-20x faster than SHA-256).

<details>
<summary><strong>Performance Benchmarks</strong></summary>

| Files | Size | Smart Sync | Force Upload |
|-------|------|------------|--------------|
| 10 | 50 MB | ~15 seconds | ~45 seconds |
| 100 | 500 MB | ~30 seconds | ~5 minutes |
| 1000 | 5 GB | ~2 minutes | ~30 minutes |

*Times vary based on network speed and file changes. Smart sync typically skips 60-90% of files.*

**File Size Handling:**
- Small files (<4MB): Direct upload (~10 files/second)
- Large files (≥4MB): Chunked upload (~50 MB/second)

**FileHash Backfill Performance:**
- Backfilling 200 empty hashes: ~40 seconds
- Force re-upload alternative: ~6.7 minutes
- **Time savings: 90%** | **Bandwidth savings: 99.8%**

</details>

## 🤝 Contributing

We welcome contributions! Here's how to get started:

### Development Setup

1. Fork the repository
2. Create feature branch: `git checkout -b feature/amazing-feature`
3. Make your changes
4. Test locally (see below)
5. Commit: `git commit -m 'Add amazing feature'`
6. Push: `git push origin feature/amazing-feature`
7. Open Pull Request

### Local Testing

```bash
# Build Docker image
docker build -t sharepoint-upload .

# Test with sample files
docker run sharepoint-upload \
  "test-site" \
  "test.sharepoint.com" \
  "tenant-id" \
  "client-id" \
  "client-secret" \
  "test-path" \
  "*.md"
```

## 📞 Support

- **🐛 Issues:** [GitHub Issues](https://github.com/AunalyticsManagedServices/sharepoint-file-upload-action/issues)
- **💬 Discussions:** [GitHub Discussions](https://github.com/AunalyticsManagedServices/sharepoint-file-upload-action/discussions)

## 🙏 Acknowledgments

Built with these excellent open-source projects:
- [Mistune](https://github.com/lepture/mistune) - Markdown parsing
- [Mermaid-CLI](https://github.com/mermaid-js/mermaid-cli) - Diagram rendering
- [xxHash](https://github.com/Cyan4973/xxHash) - Ultra-fast hashing
- [Requests](https://github.com/psf/requests) - HTTP client
- [MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-python) - Microsoft Authentication Library

---