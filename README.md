# azrf — Azure Resource Finder

> **Find any Azure resource across all your subscriptions in seconds.**

A high-performance CLI tool for Azure engineers and FinOps teams. Search 50+ resource types by name, IP, hostname, tag, or secret — across every subscription you have access to — with sub-second results for most queries.

---

## Why azrf?

Azure Portal is slow and subscription-scoped. The CLI requires knowing which subscription a resource lives in. `azrf` searches **everywhere at once** — all subscriptions, all resource groups — and gives you a direct portal link to what you found.

```
$ azrf secret DB_PASSWORD

🔐 Searching for secret: DB_PASSWORD
🔍 Found 113 Key Vaults, checking in parallel...
⚙️  Using 20 workers

🔍 Progress: [113/113] 100.0% | Found: 1 | Elapsed: 2m31s

📋 RESULTS
==========
✅ Found 1 resource:

🔹 Resource #1:
   📝 Name: DB_PASSWORD
   🏷️  Type: Microsoft.KeyVault/vaults/secrets
   🔐 Key Vault: prod-secrets-eastus
   📁 Resource Group: production
   📍 Location: eastus
   🆔 Subscription: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   🔗 https://portal.azure.com/#view/...
```

---

## Features

- **⚡ Sub-second results** for most resource types via Azure Resource Graph
- **🔍 50+ resource types** — VMs, secrets, databases, networking, containers, and more
- **🔄 Smart fallback** — Resource Graph first, parallel Azure CLI for nested resources
- **🔐 Advanced secret search** — find by name across all vaults, or scan values
- **📊 Real-time progress** — live indicators with timing and found count
- **📁 Export** — JSON and CSV with one flag
- **🔗 Azure Portal links** — every result includes a direct deep link
- **🤖 Script-friendly** — clean JSON output mode for pipelines and FinOps automation
- **⚠️ Transparent coverage** — clearly reports when vaults or subscriptions are out of reach, so you always know what was actually searched

---

## Performance

| Search | Method | Time | Notes |
|--------|--------|------|-------|
| VM, storage, keyvault, etc. | Resource Graph | **< 1 second** | Instant |
| Secret by name (113 vaults) | Parallel SDK | **~2.5 min** | NumCPU×2 workers |
| Secret pattern in one vault | Key Vault SDK | **~1.5–2 sec** | No subprocess, cached token |
| Blob containers | ARM REST + Resource Graph | **< 2 sec** | Pure Go, no subprocess |
| SQL / Postgres / MySQL / MariaDB databases | Resource Graph | **< 1 second** | Instant |
| Subnets, Cosmos MongoDB | Resource Graph | **< 1 second** | Instant |

---

## Getting started

### 1. Prerequisites

- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) installed and signed in:
  ```bash
  az login
  ```
- A valid **azrf license key** (see [Licensing](#licensing))
- If you have multiple subscriptions:
  ```bash
  az account set --subscription <id>
  ```

### 2. Download

Binaries are distributed to licensed customers. After purchase you receive a license key and download instructions for your platform:

| Platform | Binary |
|----------|--------|
| macOS Apple Silicon | `azrf-darwin-arm64` |
| macOS Intel | `azrf-darwin-amd64` |
| Linux x64 | `azrf-linux-amd64` |
| Windows x64 | `azrf-windows-amd64.exe` |

**macOS only** — run this once after download:
```bash
xattr -d com.apple.quarantine azrf-darwin-arm64
chmod +x azrf-darwin-arm64
```

### 3. Activate your machine

```bash
export KEYGEN_LICENSE_KEY="your-license-key"
azrf license activate
```

Output:
```
✅ Machine activated (id: abc-123...)
   Activation saved to ~/.azrf/activation.json
```

You only need to do this **once per machine**.

### 4. Run a search

```bash
azrf resource my-vm
azrf ip 203.0.113.10
azrf secret DB_PASSWORD
```

---

## All search types

### By name or attribute

```bash
azrf resource <name>           # any resource whose name contains the string
azrf ip <address>              # resources tied to a public IP
azrf hostname <fqdn>           # App Services, Function Apps, Static Web Apps
azrf tag <key> <value>         # exact tag key + value match
azrf type <arm-type>           # ARM resource type substring
```

### Key Vault secrets

```bash
azrf secret <name>             # find secret by name across all vaults (default: up to 1000)
azrf secret <name> 100         # stop after 100 matches
azrf secret <name> 0           # no limit — return everything
azrf secret-in-vault <vault> <pattern>   # search inside one specific vault (fast)
azrf secrets-contain <word>    # secrets whose name contains a word, across all vaults
azrf secrets-contain <word> 50 # same, capped at 50
```

### Compute

`vm` `vmss` `vmimage` `snapshot` `availset` `ppg` `aks` `aci` `containerapp` `sfcluster`

### Applications

`webapp` `functionapp` `logicapp` `apiapp`

### Data & storage

`storage` `blobcontainer` `fileshare` `datalake` `sqldb` `sqlserver` `postgresql` `mysql` `mariadb` `mongodb` `cosmosdb` `redis` `backupvault`

### Messaging

`eventhub` `servicebus`

### Containers & registries

`registry`

### Networking

`vnet` `subnet` `loadbalancer` `nsg` `routetable` `appgateway` `frontdoor` `privateendpoint` `expressroute` `vpngateway` `cdn` `ddos` `firewallpolicy`

### Platform & operations

`keyvault` `appinsights` `loganalytics` `apim` `automation` `actiongroup` `policy` `identity`

---

## Flags

| Flag | Effect |
|------|--------|
| `-json` | Print one JSON document on stdout — use with `-quiet` for clean piping |
| `-quiet` | No banner or progress on stderr |
| `-export` | Write results to `.json` and `.csv` in the current directory |
| `-version` | Print version and exit |
| `-h` / `--help` | Show built-in help |

---

## Script / automation mode

```bash
azrf -json -quiet resource my-app | jq '.resources[].name'
```

JSON schema (`azure-resource-finder.results.v1`):

```json
{
  "schema": "azure-resource-finder.results.v1",
  "cliVersion": "1.0.0",
  "searchType": "resource",
  "searchValue": "my-app",
  "durationMs": 312,
  "count": 3,
  "resources": [...]
}
```

---

## azrf-agent — pipeline / FinOps engine

`azrf-agent` is a separate binary included in every release alongside the interactive CLI. It is designed for CI/CD pipelines, GitOps workflows, and FinOps automation engines where you need clean JSON, reliable exit codes, and no human-facing output.

| | `azrf` | `azrf-agent` |
|---|---|---|
| Output | Human-readable (emoji, progress) | Always JSON on stdout |
| Stderr | Banner + progress indicators | Silent (errors only) |
| Exit codes | 0 = found, 1 = not found | 0 = found, 1 = not found, **2 = error** |
| Flags | `-json`, `-quiet`, `-export` | None needed |

```bash
# Fail a pipeline step if a secret doesn't exist
if ! azrf-agent secret DB_PASSWORD > result.json; then
  echo "Secret not found — aborting deploy"
  exit 1
fi

# Extract subscription from results
SUB=$(jq -r '.resources[0].subscriptionId' result.json)

# Enforce a tag policy across all VMs
azrf-agent tag Environment "" | jq -r '.resources[].name'
```

Output envelope:

```json
{
  "schema": "azure-resource-finder.results.v1",
  "agentVersion": "1.0.0",
  "searchType": "secret",
  "searchValue": "DB_PASSWORD",
  "durationMs": 2937,
  "workers": 20,
  "count": 1,
  "resources": [...]
}
```

Binaries follow the same naming convention as the CLI:

| Platform | Binary |
|----------|--------|
| macOS Apple Silicon | `azrf-agent-darwin-arm64` |
| macOS Intel | `azrf-agent-darwin-amd64` |
| Linux x64 | `azrf-agent-linux-amd64` |
| Windows x64 | `azrf-agent-windows-amd64.exe` |

---

## Export results

```bash
azrf -export resource my-app
# writes: azure_resources_resource_my-app_20260101_120000.json
#         azure_resources_resource_my-app_20260101_120000.csv
```

---

## Licensing

azrf is a **commercial product** distributed under a proprietary license. Each license key is tied to a specific machine via hardware fingerprinting.

| Command | Purpose |
|---------|---------|
| `azrf license activate` | Register this machine — **required on first install** |
| `azrf license validate` | Check if your license key is currently valid |
| `azrf license deactivate` | Unregister this machine before moving to another device |

`KEYGEN_LICENSE_KEY` must be set in your environment for all searches on release builds:

```bash
# bash / zsh
export KEYGEN_LICENSE_KEY="your-license-key"

# PowerShell
$env:KEYGEN_LICENSE_KEY = "your-license-key"
```

---

## Self-update

```bash
azrf update
```

Checks for a newer release and downloads the new binary next to the current one. Follow the printed `mv` command to install it.

---

## Troubleshooting

| Error | Fix |
|-------|-----|
| `failed to get access token` | Run `az login` and complete MFA if required |
| `KEYGEN_LICENSE_KEY is required` | Set the env var: `export KEYGEN_LICENSE_KEY="..."` |
| `this machine is not activated` | Run `azrf license activate` |
| `license not valid (EXPIRED)` | Renew your license — contact your vendor |
| `license not valid (SUSPENDED)` | Contact your vendor |
| `X vault(s) were inaccessible` | Those vaults have network ACLs or you lack permissions — results may be incomplete |
| No results, some subscriptions missing | Run `az account list` and check you have Reader on all subscriptions |

---

## Architecture

### Two-tier search strategy
1. **Azure Resource Graph** — fast, single API call, covers the entire tenant in milliseconds
2. **Native SDK / ARM REST fallback** — for nested resources not fully indexed by Resource Graph (Key Vault secrets via data-plane SDK; blob containers and file shares via direct ARM REST). No `az` subprocess is ever spawned.

### Parallel processing
- Worker count: `NumCPU × 2` — no artificial cap, uses every core available
- Separate worker pools per search type
- Atomic progress counters, early termination on limit

### License enforcement
- Machine-locked via hardware fingerprint — one activation per device

---

© 2026 azrf. All rights reserved. Unauthorized redistribution prohibited.
