# Terraformer Azure Infrastructure Export – End-to-End Working Guide

Environment Used: Windows 11 + Ubuntu (WSL)  
Purpose: Reverse-engineer an existing Azure subscription into Terraform code using Terraformer and prepare it for AI-based cleanup.

This document is written so that **any new team member (including beginners)** can follow the same steps and reach the same output.

---

# STAGE 1: INITIAL SETUP & CONTEXT (FOUNDATION)

This stage covers system setup, tool installation, and Azure authentication.

---

## 1. Prerequisites

### System Requirements
- OS: Windows 11
- Stable internet connection
- Read-only access to Azure subscription

### Tools Required
- Azure CLI
- Terraform
- Terraformer
- Docker (optional – required only for AI cleanup in later stages)
- Ubuntu via WSL (used for stable execution)

---

## 2. Verify System & Terraform Installation

### Check Operating System
```powershell
systeminfo | findstr /B /C:"OS Name"
````

### Check Terraform Installation

```powershell
terraform --version
```

Terraform v1.x is sufficient for Terraformer export.

---

## 3. Install Terraformer (Windows)

### Create Terraformer Directory

```powershell
mkdir C:\terraformer
```

### Download Terraformer Binary

Download from:

```
https://github.com/GoogleCloudPlatform/terraformer/releases
```

Select:

```
terraformer-all-windows-amd64.exe
```

### Rename & Move Binary

```powershell
rename terraformer-all-windows-amd64.exe terraformer.exe
move terraformer.exe C:\terraformer\
```

### (Optional) Add Terraformer to PATH

Add this path to **System Environment Variables → PATH**:

```
C:\terraformer
```

### Verify Terraformer Installation

```powershell
terraformer --version
```

---

## 4. Azure Authentication

### Login to Azure

```bash
az login
```

### List Available Subscriptions

```bash
az account list --output table
```

### Confirm Active Subscription

```bash
az account show
```

Ensure the correct **Tenant ID** and **Subscription ID** are selected before proceeding.

---

# STAGE 2: TERRAFORMER EXECUTION & VERIFIED WORKING PROCESS

This stage documents **only the steps that actually worked end-to-end**.

---

## 5. Azure Subscription Inventory (What Exists in Portal)

To identify all resource types under the subscription:

```bash
az resource list --query "[].type" -o tsv | sort -u > azure_types.txt
nano azure_types.txt
```

### Azure resource types found

```text
Microsoft.Compute/sshPublicKeys
Microsoft.ContainerRegistry/registries
Microsoft.ContainerRegistry/registries/webhooks
Microsoft.DevCenter/devcenters
Microsoft.Insights/components
Microsoft.Insights/dataCollectionRules
Microsoft.KeyVault/vaults
Microsoft.Logic/workflows
Microsoft.Network/networkInterfaces
Microsoft.Network/networkSecurityGroups
Microsoft.Network/networkWatchers
Microsoft.Network/publicIPAddresses
Microsoft.Network/virtualNetworks
Microsoft.OperationalInsights/workspaces
Microsoft.Sql/servers
Microsoft.Sql/servers/databases
Microsoft.Storage/storageAccounts
Microsoft.Web/connections
Microsoft.Web/serverFarms
Microsoft.Web/sites
Microsoft.Web/staticSites
microsoft.alertsmanagement/smartDetectorAlertRules
microsoft.insights/actiongroups
microsoft.visualstudio/account
```

This represents **everything visible in Azure portal under the subscription**.

---

## 6. Terraform Working Directory Setup (MANDATORY)

Terraformer failed earlier with:

```text
exec: no command
```

This was fixed **only after running Terraformer from a proper Terraform working directory**.

### Create Terraform working directory

```bash
mkdir -p ~/tf-azure
cd ~/tf-azure
```

### Create versions.tf

```bash
cat > versions.tf <<'EOF'
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.56.0"
    }
  }
}
EOF
```

### Create provider.tf

```bash
cat > provider.tf <<'EOF'
provider "azurerm" {
  features {}
}
EOF
```

### Initialize Terraform

```bash
terraform init
```

Terraformer was executed **only from this directory**.

---

## 7. Terraformer-Supported Resources (From This Subscription)

Terraformer does not import everything automatically.
Only supported resources must be explicitly provided.

### Supported mappings

| Azure Resource Type                             | Terraformer Resource       |
| ----------------------------------------------- | -------------------------- |
| Microsoft.Compute/sshPublicKeys                 | ssh_public_key             |
| Microsoft.ContainerRegistry/registries          | container_registry         |
| Microsoft.ContainerRegistry/registries/webhooks | container_registry_webhook |
| Microsoft.Network/networkInterfaces             | network_interface          |
| Microsoft.Network/networkSecurityGroups         | network_security_group     |
| Microsoft.Network/networkWatchers               | network_watcher            |
| Microsoft.Network/publicIPAddresses             | public_ip                  |
| Microsoft.Network/virtualNetworks               | virtual_network            |
| Microsoft.Storage/storageAccounts               | storage_account            |
| Microsoft.Web/serverFarms + sites               | app_service                |
| Microsoft.Sql/servers + databases               | database (logical group)   |

---

## 8. Final Terraformer Command That Worked

Executed from `~/tf-azure`:

```bash
set -o pipefail

OUT="$HOME/tf-scan-subscription-no-db"
rm -rf "$OUT" && mkdir -p "$OUT"

terraformer import azure \
  --resources=resource_group,virtual_network,network_security_group,network_interface,public_ip,network_watcher,storage_account,container_registry,container_registry_webhook,app_service,ssh_public_key \
  --path-output="$OUT" --compact 2>&1 | tee "$OUT/full.log"

echo "EXIT=$?"
```

Result:

```text
EXIT=0
```

---

## 9. Terraformer Output (Verified)

### Generated folders

```text
azurerm/
├── app_service
├── network_interface
├── network_security_group
├── network_watcher
├── public_ip
├── resource_group
├── ssh_public_key
├── storage_account
└── virtual_network
```

### Terraform resources generated

```text
azurerm_app_service
azurerm_network_interface
azurerm_network_security_group
azurerm_network_security_rule
azurerm_network_watcher
azurerm_public_ip
azurerm_resource_group
azurerm_ssh_public_key
azurerm_storage_account
azurerm_virtual_network
```

---

## 10. Resources Not Imported by Terraformer

### Not exported by Terraformer

```text
Microsoft.KeyVault/vaults
Microsoft.Insights/components
Microsoft.OperationalInsights/workspaces
Microsoft.Insights/dataCollectionRules
microsoft.insights/actiongroups
microsoft.alertsmanagement/smartDetectorAlertRules
Microsoft.Logic/workflows
Microsoft.Web/staticSites
Microsoft.Web/connections
Microsoft.DevCenter/devcenters
microsoft.visualstudio/account
```

These may be Terraform-supported, but **Terraformer does not export them**.

---

## 11. SQL Limitation (Terraformer Crash)

When SQL was included:

```text
Microsoft.Sql/servers
Microsoft.Sql/servers/databases
```

Terraformer crashed with:

```text
panic: unknown resource type azurerm_sql_active_directory_administrator
```

SQL was excluded due to this Terraformer limitation.

---

## 12. AI Cleanup Workspace Verification

### Locate AI cleanup folder

```bash
find $HOME -maxdepth 3 -type d -name tf-ai-cleanup
```

Output:

```text
/root/tf-ai-cleanup
```

### Navigate to workspace

```bash
cd /root/tf-ai-cleanup
```

### Verify structure

```bash
tree . -L 3
```

This confirmed:

* `input/` → raw Terraformer code
* `output/` → AI-cleaned output
* `shared/` → provider and versions baseline

---

## 13. Final Outcome

* Azure subscription inventory captured
* Terraformer successfully imported all supported resources
* Unsupported and crashing resources documented
* AI cleanup workspace verified and ready

This document reflects **only what actually worked in practice**.
