this document works when you have already work on terraformer and working now for second time.


It includes : Azure login, subscription setup, Terraformer execution, Windows fixes, and demo notes.

---

````md
# Terraformer + Azure CLI Setup (Windows | PowerShell)

This document explains how to set up Azure authentication, configure subscription context, and run **Terraformer** successfully on **Windows (PowerShell)**.

---

## 1. Prerequisites

Ensure the following tools are installed:

- Azure CLI  
- Terraform  
- Terraformer  

Verify installation:

```powershell
az version
terraform -version
terraformer --version
````

---

## 2. Azure Login

Login to Azure using Azure CLI:

```powershell
az login
```

* Browser will open
* Login with **company Azure account**
* Close browser after successful login

This creates a **temporary authenticated session** (no secrets stored).

---

## 3. List Available Subscriptions

```powershell
az account list --output table
```

Note down the correct **Subscription ID** you want to work on.

---

## 4. Set Active Subscription

Set the subscription explicitly (VERY IMPORTANT):

```powershell
az account set --subscription <SUBSCRIPTION_ID>
```

Example:

```powershell
az account set --subscription 12345678-aaaa-bbbb-cccc-123456789abc
```

---

## 5. Verify Subscription Context

```powershell
az account show --output table
```

Confirm:

* Subscription ID
* Subscription Name
* Logged-in User

---

## 6. (Optional but Recommended) Export ARM Subscription ID

This helps Terraform/Terraformer detect Azure context properly.

### PowerShell (Windows)

```powershell
$env:ARM_SUBSCRIPTION_ID="12345678-aaaa-bbbb-cccc-123456789abc"
```

---

## 7. Sanity Check Azure Access

Run a harmless read-only command:

```powershell
az group list --output table
```

If resource groups are listed → Azure access is working ✅

---

## 8. Terraformer Command (PowerShell Syntax)

⚠️ **Important**
PowerShell does NOT support Linux `\` line continuation.
Use **single line** or **backtick (`)**.

### Correct Terraformer Import Command

```powershell
terraformer import azure --resources=virtual_network,subnet --resource-group=my-rg
```

OR (with backticks):

```powershell
terraformer import azure `
  --resources=virtual_network,subnet `
  --resource-group=my-rg
```

---

## 9. Authentication Logs (Expected Output)

Terraformer tries multiple auth methods and finally uses Azure CLI:

```text
Using Obtaining a token from the Azure CLI for Authentication
```

This confirms **Azure CLI authentication is working correctly**.

---

## 10. Windows Plugin Directory Error (Common Issue)

### Error Seen

```text
open \.terraform.d/plugins/windows_amd64: The system cannot find the path specified.
```

### Reason

* Terraformer expects Terraform plugin directories
* On Windows, `HOME` variable may not be set correctly

---

## 11. Fix: Configure Terraform Plugin Directory (Windows)

### Step 1: Set HOME variable

```powershell
$env:HOME = $env:USERPROFILE
```

### Step 2: Create plugin directory

```powershell
New-Item -ItemType Directory -Force -Path "$env:HOME\.terraform.d\plugins\windows_amd64" | Out-Null
```

### Step 3: Set Terraform plugin cache (recommended)

```powershell
$env:TF_PLUGIN_CACHE_DIR = "$env:HOME\.terraform.d\plugins\windows_amd64"
```

### Step 4: Verify directory exists

```powershell
Test-Path "$env:HOME\.terraform.d\plugins\windows_amd64"
```

Expected output:

```text
True
```

---

## 12. (If Required) Download Azure Provider Manually

If Terraformer still complains about provider plugins:

### Create a temporary folder

```powershell
mkdir C:\terraform-provider-init
cd C:\terraform-provider-init
```

### Create provider config

```powershell
@'
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}
'@ | Set-Content .\main.tf
```

### Initialize Terraform

```powershell
terraform init
```

This downloads Azure provider plugins.

---

## 13. Re-run Terraformer

Go back to your working directory and run:

```powershell
terraformer import azure --resources=virtual_network,subnet --resource-group=my-rg
```

---

## 14. Output Location

Terraformer generates output under:

```
./generated/azurerm/
```

Contains:

* `providers.tf`
* Resource `.tf` files
* `terraform.tfstate`

---

## 15. Demo Explanation (One-liner)

> “I authenticated using Azure CLI, scoped the correct subscription, and used Terraformer to reverse-engineer existing Azure infrastructure into Terraform code. This is read-only and safe.”

---

## 16. Safety Notes

* ❌ No resources are created
* ❌ No changes made to Azure
* ✔️ Read-only API calls
* ✔️ Suitable for demos and audits

---

## 17. Useful Verification Commands

```powershell
az account show
terraformer --version
terraform -version
```

---

## End of Document


