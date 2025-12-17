**AI cleanup phase (Terraformer output → production-ready Terraform)** using **cagent + Terraform MCP Server** on **Windows 11**.

# Terraformer AI Cleanup Guide (Windows 11) — cagent + Terraform MCP Server

This guide starts **after** you already completed Terraformer infra scan and you have generated Terraform `.tf` files (and maybe `.tfstate`) from Azure.

Goal:
- Take Terraformer output (messy auto-generated Terraform)
- Run AI-based cleanup (cagent + Terraform MCP Server)
- Produce a **clean, production-ready Terraform codebase** that passes `terraform validate`

---

## 0) Prerequisites (Must Have)

### A) Tools
- Windows 11 (PowerShell)
- Terraform installed (your Terraform is OK)
- Docker Desktop installed + running (recommended)
- Azure CLI installed (optional, only needed later for import)
- Git (optional)

### B) Folder Convention (same as your setup)
We will use:
- Working folder: `C:\terraformer-ai-cleanup`
- Input folder: `C:\terraformer-ai-cleanup\input`
- Output folder: `C:\terraformer-ai-cleanup\output`

---

## 1) Create working folders

Open PowerShell and run:

```powershell
mkdir C:\terraformer-ai-cleanup -Force
mkdir C:\terraformer-ai-cleanup\input -Force
mkdir C:\terraformer-ai-cleanup\output -Force

---

## 2) Put Terraformer output into `input`

Copy your Terraformer-generated files into:

`C:\terraformer-ai-cleanup\input`

Typical files:

* `*.tf` (resource_group.tf, virtual_network.tf, subnet.tf, etc.)
* `provider.tf` (if available)
* `terraform.tfstate` (if Terraformer produced it)

Check:

```powershell
dir C:\terraformer-ai-cleanup\input
```

---

## 3) Install cagent on Windows (Beginner Steps)

### Option A (Recommended): Download Release Binary

1. Open cagent GitHub releases (from your senior’s repo link)
2. Download **Windows amd64** binary (usually named like `cagent-windows-amd64.exe`)

### Put it in a tools folder

```powershell
mkdir C:\tools\cagent -Force
```

Move the EXE into:

* `C:\tools\cagent\cagent.exe`

You can rename it easily:

```powershell
Rename-Item "C:\tools\cagent\<downloaded-file-name>.exe" "cagent.exe"
```

### Add to PATH (so you can run `cagent` from anywhere)

Run this in PowerShell:

```powershell
$old = [Environment]::GetEnvironmentVariable("Path", "User")
$new = $old + ";C:\tools\cagent"
[Environment]::SetEnvironmentVariable("Path", $new, "User")
```

Close PowerShell and open again, then test:

```powershell
cagent --help
```

> Note: Some cagent builds don’t support `--version`. So use `cagent --help` as validation.

---

## 4) Run Terraform MCP Server (Docker recommended)

Why we need this:

* Terraform MCP server helps the AI understand Terraform registry/provider behaviors and generate better refactoring.

### Run MCP server (stdio mode) using Docker

You do NOT need to run it separately if your cagent YAML starts it as a tool,
but Docker-based setup is most stable.

Test Docker is running:

```powershell
docker ps
```

Optional: pull image (faster later)

```powershell
docker pull hashicorp/terraform-mcp-server:0.3.3
```

---

## 5) Create cagent workflow YAML

Create a YAML file:
`C:\terraformer-ai-cleanup\terraform-cleanup.yaml`

PowerShell command to create empty file:

```powershell
New-Item -ItemType File -Path "C:\terraformer-ai-cleanup\terraform-cleanup.yaml" -Force
```

Now open it:

```powershell
notepad C:\terraformer-ai-cleanup\terraform-cleanup.yaml
```

### Sample YAML (template)

> This is a simple structure. Your org may already have a standard YAML.
> If your senior provided a YAML, use that.

Minimum example (root agent + MCP tool):

```yaml
version: "2"

models:
  openai:
    provider: openai
    model: gpt-5-mini

agents:
  root:
    model: openai
    description: "Clean and refactor Terraformer output to production-ready Terraform"
    instruction: |
      You are an expert Terraform engineer.
      Clean the Terraform files from ./input and write cleaned output to ./output.
      Ensure output passes terraform fmt and terraform validate.
      Do not run apply.
    toolsets:
      - type: filesystem
      - type: mcp
        command: docker
        args:
          ["run","-i","--rm","hashicorp/terraform-mcp-server:0.3.3"]
```

Save the file.

---

## 6) Set AI API Key (Required)

cagent needs an API key depending on the model you use.

### For OpenAI

In PowerShell:

```powershell
setx OPENAI_API_KEY "YOUR_KEY_HERE"
```

Close PowerShell and open again (important), then verify:

```powershell
echo $env:OPENAI_API_KEY
```

> If you are using OpenRouter, you still set it in `OPENAI_API_KEY` as per many OpenAI-compatible tools.

---

## 7) Run AI Cleanup

Go to the cleanup folder:

```powershell
cd C:\terraformer-ai-cleanup
```

Run:

```powershell
cagent run .\terraform-cleanup.yaml
```

If the tool opens an interactive prompt and asks you to type something, type:

* `start`

After completion, your cleaned code should be in:
`C:\terraformer-ai-cleanup\output`

Check:

```powershell
dir C:\terraformer-ai-cleanup\output
```

---

## 8) Validate cleaned Terraform output (Must Do)

Go to output folder:

```powershell
cd C:\terraformer-ai-cleanup\output
```

Run:

```powershell
terraform fmt
terraform validate
```

Expected success:

```text
Success! The configuration is valid.
```

---

## 9) Common Errors & Fixes (Real-world)

### Error A: Invalid multi-line string in variables.tf

Example:

* `description = "....` spanning multiple lines

Fix:

* Make description single-line OR use heredoc.
  Simplest fix is single-line description.

---

### Error B: Unsupported argument in subnet.tf

Example error:

```text
Unsupported argument: private_endpoint_network_policies_enabled
```

Fix:
Replace:

```hcl
private_endpoint_network_policies_enabled = true/false
```

With supported form:

```hcl
private_endpoint_network_policies = "Enabled"
private_endpoint_network_policies = "Disabled"
```

Mapping:

* `true`  → `"Enabled"`
* `false` → `"Disabled"`

Then run again:

```powershell
terraform fmt
terraform validate
```

---

### Error C: terraform.tfstate missing in output folder

This is not a failure.

Many cleanup workflows intentionally produce **state-less** cleaned Terraform.
State is recreated later using `terraform import`.

So your output may contain:

* Clean `.tf` files ✅
* No `terraform.tfstate` ✅ (expected)

---

## 10) Package output for seniors / internal tool testing

Create zip:

```powershell
Compress-Archive `
  -Path C:\terraformer-ai-cleanup\output\* `
  -DestinationPath C:\terraformer-ai-cleanup\terraformer-clean-output.zip `
  -Force
```

Deliver:

* `terraformer-clean-output.zip`

---

## 11) What you can safely claim as “Completed”

* Terraformer infra scan completed (previous phase)
* AI cleanup + refactoring completed
* `terraform validate` succeeded
* Output packaged as zip and ready for:

  * Import (state recreation)
  * Internal tool testing
  * Hierarchy/report generation

Note:

* No apply performed
* Imports will be done only after confirmation

---

## 12) Optional next steps (ONLY if asked by senior)

### A) Initialize providers (optional)

```powershell
terraform init
```

### B) Import to recreate state (approval required)

Usually flow:

* az login
* terraform init
* terraform import ...
* terraform plan (review)
* terraform apply (only with approval)

---

## Quick Proofs to show seniors

1. Validation proof:

* Screenshot of:

  ```powershell
  terraform validate
  ```

  output:
  `Success! The configuration is valid.`

2. Output proof:

* Screenshot of:

  ```powershell
  dir C:\terraformer-ai-cleanup\output
  ```

3. Zip proof:

* Screenshot of:

  ```powershell
  dir C:\terraformer-ai-cleanup\terraformer-clean-output.zip
  ```

---

```

---

If you want, I can also generate a **second file** like `ai-cleanup-runbook-short.md` (1-page quick steps) for daily usage.
::contentReference[oaicite:0]{index=0}
```
