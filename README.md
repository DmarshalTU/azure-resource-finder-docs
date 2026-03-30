# azrf — Azure Resource Finder (user guide)

End-user documentation: **what you can search**, **flags**, **limits**, and **licensing**.  

## What it does

**azrf** searches Azure across subscriptions you can access: by IP, hostname, name, tags, ARM type, Key Vault secrets, and [dozens of resource-specific modes](#complete-list-of-search-types). It prefers **Azure Resource Graph** for speed and uses **Azure CLI / SDK** where graph queries are not enough.

## Before you run anything

1. Install [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli).
2. Sign in: `az login` (complete MFA if your tenant requires it).
3. Select a subscription: `az account set --subscription <id>` if you have several.
4. You need **Resource Graph** access plus permissions on the resources you query (Key Vault secret searches need vault/secret access).

## Command shape

```text
azrf [flags] <search_type> <search_value> [extra] [--export]
```

| Part | Meaning |
|------|--------|
| `search_type` | One of the [search types](#complete-list-of-search-types) (e.g. `ip`, `resource`, `secret`). |
| `search_value` | What to look for (IP string, name fragment, tag key, etc.). |
| `extra` | Only for modes that need two values: `tag`, `secret-in-vault`, or a numeric **limit** for `secret` / `secrets-contain`. |
| `--export` | Trailing only: same as `-export`. |

On Windows the binary may be `azrf.exe`.

### Flags

| Flag | Meaning |
|------|--------|
| `-json` | Print one JSON document on stdout (schemas for automation). |
| `-quiet` | No banner or stderr progress. |
| `-export` | After the search, write JSON and CSV in the current directory. |
| `-version` | Print embedded version / build metadata and exit. |
| `-h` / `--help` | Short built-in help (also lists the docs URL). |

### Subcommands

| Command | Purpose |
|---------|--------|
| `azrf help` | Same as `-h`. |
| `azrf license validate [KEY]` | Validate a Keygen license key (`KEYGEN_*` env). |
| `azrf update` | Check Keygen for a newer release and download the artifact (`KEYGEN_*`). |

---

## Secret search limits

For **`secret`** and **`secrets-contain`**, add an optional **third argument** (integer ≥ 0):

```text
azrf secret <secret_name> [limit]
azrf secrets-contain <word> [limit]
```

| Value | Effect |
|-------|--------|
| *(omitted)* | Default limit **1000** matches. |
| `0` | No cap (collect all matches the search can find). |
| `N` (`N` > 0) | Stop after **N** matching secrets (parallel vault scan). |

Examples: `azrf secret DB_PASSWORD 100` · `azrf secrets-contain password 50`

---

## Modes that need two values

| Mode | Syntax | Notes |
|------|--------|--------|
| **tag** | `azrf tag <key> <value>` | Exact tag key and value. |
| **secret-in-vault** | `azrf secret-in-vault <vault_name> <pattern>` | Search inside one vault. |

All other types use a single `search_value` (name/pattern/IP/host substring as applicable).

---

## Complete list of search types

Each row is: **`search_type`** — what `search_value` is (unless noted).

### Discovery & naming

| Type | You pass |
|------|----------|
| `ip` | Public IP address |
| `hostname` | FQDN (resolved then matched) |
| `resource` | Substring of resource **name** |
| `type` | Substring of ARM **resource type** |
| `tag` | Tag **key** (and `extra` = tag **value**) |

### Key Vault

| Type | You pass |
|------|----------|
| `secret` | Secret **name**; optional `[limit]` |
| `secret-in-vault` | Vault **name**; `extra` = secret **pattern** |
| `secrets-contain` | Word to find in secret **values**; optional `[limit]` |

### Compute & apps

`vm` · `vmss` · `vmimage` · `snapshot` · `availset` · `ppg` · `aks` · `aci` · `containerapp` · `sfcluster` · `webapp` · `functionapp` · `logicapp` · `apiapp`

### Data & messaging

`storage` · `registry` · `sqldb` · `sqlserver` · `postgresql` · `mysql` · `mariadb` · `mongodb` · `cosmosdb` · `redis` · `eventhub` · `servicebus` · `blobcontainer` · `fileshare` · `datalake` · `backupvault`

### Networking

`vnet` · `subnet` · `loadbalancer` · `nsg` · `routetable` · `appgateway` · `frontdoor` · `privateendpoint` · `expressroute` · `vpngateway` · `cdn` · `ddos` · `firewallpolicy`

### Platform & ops

`keyvault` · `appinsights` · `loganalytics` · `apim` · `automation` · `actiongroup` · `policy` · `identity`

For each of these, **`search_value`** is typically a **name or pattern** to match (behavior matches the implementation in `pkg/azrf`).

---

## Quick examples

```text
azrf ip 203.0.113.10
azrf resource my-prod-vm
azrf tag Environment Production
azrf secret qwerty 100
azrf -json -quiet resource my-app
```

---

## License and updates (Keygen)

Shipped builds may **require** a valid Keygen license before searches and use Keygen for **self-update**. Configure `KEYGEN_ACCOUNT_ID`, `KEYGEN_LICENSE_KEY`, and for `azrf update` also `KEYGEN_PRODUCT_ID`. Details: [Keygen documentation](https://keygen.sh/docs/) and `azrf -h` on your binary.

---

## Built-in help

```text
azrf -h
azrf help
```
