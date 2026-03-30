# Azure Resource Finder

🚀 **Lightning-fast Azure resource discovery across multiple subscriptions**

A high-performance Go tool for quickly finding Azure resources by name, IP address, hostname, tags, or resource type. Features parallel processing, intelligent fallback mechanisms, and comprehensive coverage of 50+ Azure resource types.

**User guide (all search types, flags, limits):** [docs/README.md](docs/README.md)

## ✨ Features

- **⚡ Lightning Fast**: Parallel processing with dynamic worker pools
- **🔍 Comprehensive Search**: 50+ Azure resource types supported
- **🔄 Smart Fallback**: Resource Graph + Azure CLI fallback for nested resources
- **📊 Real-time Progress**: Visual progress indicators with timing
- **📁 Export Options**: JSON and CSV export capabilities
- **🎯 Early Termination**: Configurable limits for optimal performance
- **🛡️ Error Handling**: Graceful handling of permissions and API limits
- **🔗 Azure Portal Links**: Direct links to resources in Azure Portal
- **🔐 Advanced Secret Search**: Search in specific vaults or across all vaults
- **📈 Performance Optimized**: Connection pooling and memory management

## 🚀 Performance

- **Secrets Search**: 113 Key Vaults in ~2.5 minutes using 28 workers
- **Resource Graph**: Sub-second queries for most resource types
- **Parallel Processing**: Dynamic worker count based on CPU cores (capped at 50)
- **Optimized CLI Calls**: Direct Azure SDK + efficient CLI fallbacks
- **Connection Pooling**: Shared HTTP client with connection reuse
- **Memory Optimization**: Pre-allocated slices with capacity hints
- **Early Termination**: Configurable limits for faster results

## 📋 Supported Resource Types

### Core Resources
- Virtual Machines, VM Scale Sets, VM Images
- Storage Accounts, Blob Containers, File Shares
- SQL Servers/Databases, PostgreSQL, MySQL, MariaDB
- Key Vaults, Secrets, Managed Identities
- Container Registries, AKS Clusters, Container Apps

### Secret Search Features
- **Specific Vault Search**: `secret-in-vault <vault> <pattern>` - Fast search in known vault
- **Cross-Vault Search**: `secrets-contain <word> [limit]` - Search all vaults for pattern
- **Global Secret Search**: `secret <name> [limit]` - Find specific secret across vaults

### Networking
- Virtual Networks, Subnets, Route Tables
- Load Balancers, Application Gateways
- Network Security Groups, Firewall Policies
- ExpressRoute Circuits, VPN Gateways
- Front Doors, CDN Profiles

### Data & Analytics
- Cosmos DB, Redis Cache, Event Hubs
- Log Analytics Workspaces, Application Insights
- Data Lake Storage, Backup Vaults

### Application Services
- Web Apps, Function Apps, API Apps
- Logic Apps, Container Instances
- Service Bus, API Management
- Service Fabric Clusters

## 🛠️ Installation

### Prerequisites
- Go 1.23+ 
- Azure CLI installed and authenticated
- Azure Resource Graph permissions

### Quick Start
```bash
# Clone the repository
git clone <repository>
cd azure-resource-finder

# Build for your platform
make build

# Or build manually (short CLI name)
go build -trimpath -o azrf ./cmd/azure-resource-finder
```

### Cross-Platform Builds
```bash
# Build for all platforms
make build-all

# Build for specific platform
make build-windows  # Windows
make build-linux    # Linux
make build-macos    # macOS (Intel + ARM)

# Create distribution packages
make dist
```

### Windows Users
```powershell
# Use PowerShell build script (recommended for Windows)
.\build.ps1

# Or use Makefile (requires Unix tools)
make build-all
```

### Using Makefile
```bash
make help          # Show all available commands
make test          # Run tests
make fmt           # Format code
make lint          # Lint code
make clean         # Clean build artifacts
make version       # Show version information
```

## 📖 Usage

### Basic Search
```bash
# Search for secrets
./azrf.exe secret my-secret-name

# Search by IP address
./azrf.exe ip 4.242.81.168

# Search by hostname
./azrf.exe hostname myapp.azurewebsites.net

# Search by resource name
./azrf.exe resource my-vm-name

# Search by tag
./azrf.exe tag Environment Production
```

### Advanced Secret Search
```bash
# Search secrets in specific Key Vault (fast)
./azrf.exe secret-in-vault my-keyvault-name my-secret-pattern

# Search all secrets containing word across all vaults
./azrf.exe secrets-contain sql 50

# Search with limit
./azrf.exe secret my-secret 100
```

### Advanced Search
```bash
# Search for specific resource types
./azrf.exe vmss agent-vmss-core
./azrf.exe frontdoor apilandingpage-cdn-prod
./azrf.exe blobcontainer qa-automation-blob

# Export results
./azrf.exe secret my-secret --export

# Limit secret search (positional limit)
./azrf.exe secret my-secret 100
```

## 🔧 Configuration

### Environment Variables
- `AZURE_CLI_DISABLE_CONNECTION_VERIFICATION`: Disable connection verification
- `AZURE_CLI_DISABLE_TELEMETRY`: Disable telemetry

### Command Line Flags
- `-json`: Print one JSON document on stdout (for scripts and FinOps pipelines; use with `-quiet` for clean stdout)
- `-quiet`: Suppress banner and stderr progress
- `-export`: Write JSON and CSV files after the search (trailing `--export` is still accepted)
- `-version`: Print embedded version / build metadata
- Secret limits: `secret <name> [limit]` — use `0` for no limit (default: 1000)

### Keygen (licensing, distribution, updates)
Shipped binaries are **licensed products**: searches call Keygen to validate `KEYGEN_LICENSE_KEY` before running (see [Keygen docs](https://keygen.sh/docs/)).

| Env / command | Purpose |
|---------------|---------|
| `KEYGEN_ACCOUNT_ID`, `KEYGEN_LICENSE_KEY` | Required on release builds for every search |
| `KEYGEN_PRODUCT_ID` | Required for `azrf update` (artifact download) |
| `AZRF_REQUIRE_LICENSE=1` | When building from source, turn on the same enforcement as release binaries |
| `azrf license validate [KEY]` | Call Keygen [validate-key](https://keygen.sh/docs/api/licenses/) |
| `azrf update` | Resolve [upgrade](https://keygen.sh/docs/api/releases/) vs embedded semver, download matching `azrf-*` artifact |

CI publishes GitHub **and** Keygen when repository secrets `KEYGEN_TOKEN`, `KEYGEN_ACCOUNT_ID`, and `KEYGEN_PRODUCT_ID` are set. Run `make dist` then `bash scripts/release-keygen.sh` locally with `VERSION_SEMVER` (no `v`). Binaries stay small via `-s -w` and `-trimpath`.

## 🆕 New Features

### Azure Portal Links
All search results now include direct links to Azure Portal:
- **One-click access** to any resource in Azure Portal
- **No manual navigation** required
- **Consistent format** across all resource types

### Advanced Secret Search
- **`secret-in-vault <vault> <pattern>`**: Fast search in specific Key Vault
- **`secrets-contain <word> [limit]`**: Search all vaults for pattern
- **Progress indicators** and **performance optimization**

## 📚 Go package (FinOps / automation)

Import the engine from your own services:

```go
import "github.com/denistu/azure-resource-finder/pkg/azrf"

client := azrf.NewClient(azrf.WithVerbose(false))
resources, err := client.RunSearch(ctx, "resource", "my-vm", "", 0)
// RunSearch covers all CLI search kinds; use -json on the binary for subprocess integration.
```

Set the module path in `go.mod` to your fork if you rename the repository.

## 🏗️ Architecture

### Two-Tier Search Strategy
1. **Resource Graph First**: Fast, broad searches using Azure Resource Graph
2. **CLI Fallback**: Parallel Azure CLI calls for nested resources (secrets, containers, databases)

### Parallel Processing
- Dynamic worker count: `runtime.NumCPU() * 2`
- Worker pools for Key Vaults, Storage Accounts, SQL Servers
- Progress tracking with atomic counters

### Error Handling
- Graceful permission error handling
- API rate limiting considerations
- Fallback mechanisms for failed queries

## 📊 Performance Benchmarks

| Resource Type | Search Method | Time | Workers | Notes |
|---------------|---------------|------|---------|-------|
| Key Vault Secrets (Global) | Parallel CLI | ~2.5 minutes | 28 | Across all vaults |
| Key Vault Secrets (Specific) | Direct CLI | ~15 seconds | N/A | Single vault search |
| Storage Accounts | Resource Graph | <1 second | N/A | Instant results |
| Virtual Machines | Resource Graph | <1 second | N/A | Instant results |
| Blob Containers | Parallel CLI | ~30 seconds | 28 | Cross-storage search |
| Secret Pattern Search | Optimized CLI | ~15 seconds | N/A | 535 secrets filtered |

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Ensure all tests pass
5. Submit a pull request

## 📦 Releases

### Download Pre-built Binaries
Binaries are distributed via **Keygen** to licensed customers. After purchase you receive a license key — use it to download the binary for your platform and activate your machine.

**macOS:** after download run `xattr -d com.apple.quarantine azrf-darwin-arm64 && chmod +x azrf-darwin-arm64` once to allow execution.

### Automated Builds
This project uses GitHub Actions for automated builds and releases. When you push a tag starting with `v`, it automatically:
1. Builds **azrf** for all supported platforms (`-trimpath`, stripped)
2. Creates a GitHub release and uploads binaries + checksums
3. Optionally publishes the same artifacts to **Keygen** (if `KEYGEN_*` secrets are configured)

```bash
# Create a new release
git tag v1.0.1
git push origin v1.0.1
```

## 📄 License

MIT License - see LICENSE file for details

## 🆘 Troubleshooting

### Common Issues
- **Permission Errors**: Ensure Azure CLI is authenticated with proper permissions
- **Slow Performance**: Check network connectivity and Azure API status
- **No Results**: Verify resource names and subscription access

### Debug Mode
```bash
# Enable verbose logging
export AZURE_CLI_DISABLE_CONNECTION_VERIFICATION=1
./azrf.exe secret my-secret
```

## 🔗 Related Projects

- [Azure CLI](https://github.com/Azure/azure-cli)
- [Azure Resource Graph](https://docs.microsoft.com/en-us/azure/governance/resource-graph/)
- [Azure SDK for Go](https://github.com/Azure/azure-sdk-for-go)

---

**Made with ❤️ for Azure DevOps engineers** 
