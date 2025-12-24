# Terraformer Azure Infrastructure Export – End-to-End Guide

## Purpose of This Document

This guide documents the complete end-to-end process followed to reverse-engineer an existing **Azure subscription** into Terraform code using **Terraformer**.
It is written so that **any new team member (including beginners)** can follow the same steps and reach the same output.

---

## Prerequisites

### System Requirements

* OS: **Windows 11**
* Stable internet connection
* Read-only access to Azure subscription

### Tools Required

* **Azure CLI**
* **Terraform**
* **Terraformer**
* **Docker** (optional – required only for AI cleanup in later phases)

---

## Step 1: Verify System & Terraform Installation

### Check Operating System

```powershell
systeminfo | findstr /B /C:"OS Name"
```

### Check Terraform Installation

```powershell
terraform --version
```

> Terraform v1.x is sufficient for Terraformer export. Upgrade is optional.

---

## Step 2: Install Terraformer (Windows)

### Create Terraformer Directory

```powershell
mkdir C:\terraformer
```

### Download Terraformer Binary

Download the Windows binary from:

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

Add the following path to **System Environment Variables → PATH**:

```
C:\terraformer
```

### Verify Terraformer Installation

```powershell
terraformer --version
```

---

## Step 3: Azure Authentication

### Login to Azure

```powershell
az login
```

### List Available Subscriptions

```powershell
az account list --output table
```

### Confirm Active Subscription

```powershell
az account show
```

> Ensure correct **Tenant ID** and **Subscription ID** are selected before proceeding.

---

## Step 4: Create Terraformer Working Directory

```powershell
mkdir C:\terraformer-work
cd C:\terraformer-work
```

This directory is used to store Terraformer-generated files.

---

## Step 5: Run Terraformer for Azure

### Initial Validation – Import Resource Groups

```powershell
terraformer import azure --resources=resource_group
```

Terraformer automatically uses the Azure CLI authentication context.

### Import Additional Infrastructure Resources

Example command:

```powershell
terraformer import azure --resources=virtual_network,subnet,network_security_group,network_interface,public_ip,storage_account
```

> ⚠️ Terraformer runs in **read-only mode**. No Azure resources are modified.

---

## Step 6: Validate Terraformer Output

Terraformer generates a `generated/` directory.

### Typical Files Generated

* `resource_group.tf`
* `virtual_network.tf`
* `subnet.tf`
* `network_security_group.tf`
* `network_security_rule.tf`
* `network_interface.tf`
* `public_ip.tf`
* `storage_account.tf`
* `provider.tf`
* `variables.tf`
* `outputs.tf`
* `terraform.tfstate`

This confirms a **successful Terraformer export**.

---

## Step 7: Consolidate Output for Reporting / Cleanup

### Create Report Input Directory

```powershell
mkdir C:\terraformer-report-input
```

### Copy All Terraform Files

```powershell
Get-ChildItem C:\terraformer-work\generated -Recurse -Filter *.tf |
Copy-Item -Destination C:\terraformer-report-input
```

### Copy Terraform State File

```powershell
Get-ChildItem C:\terraformer-work\generated -Recurse -Filter terraform.tfstate |
Copy-Item -Destination C:\terraformer-report-input
```

### Verify Contents

```powershell
dir C:\terraformer-report-input
```

This folder now contains **all required inputs** for:

* Infrastructure hierarchy visualization
* Report generation
* AI-based Terraform cleanup workflows

---

## Step 8: (Optional) Prepare AI Cleanup Workspace

> This step aligns with the reference article on AI-based Terraformer cleanup.

### Create Workspace Structure

```powershell
mkdir C:\terraformer-ai-cleanup
mkdir C:\terraformer-ai-cleanup\input
mkdir C:\terraformer-ai-cleanup\output
```

### Copy Input Files

```powershell
copy C:\terraformer-report-input\* C:\terraformer-ai-cleanup\input\
```

This structure matches the expected `./input` and `./output` layout for AI-agent workflows.

---

## Final Status

### Completed ✅

* Terraformer installed and verified
* Azure authentication confirmed
* Infrastructure successfully exported
* Terraform `.tf` files and `terraform.tfstate` generated
* Consolidated input bundle prepared

### Pending ⏳

* AI-based Terraform cleanup & refactoring (Docker cagent + Terraform MCP), or
* Upload to internal reporting / visualization platform

---

## Notes for New Users

* Terraformer export is **safe and read-only**
* Generated Terraform is **not production-ready by default**
* Cleanup and refactoring are expected next steps
* Always review output before using in CI/CD pipelines

---

## Conclusion

Terraformer provides a fast and reliable way to capture existing Azure infrastructure as Terraform code. This guide ensures a repeatable and auditable process for future users and teams.
