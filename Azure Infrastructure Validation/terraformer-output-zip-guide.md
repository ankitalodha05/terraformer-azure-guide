# Terraformer Output – Directory & ZIP Creation Guide

## Purpose

This document shows:

* Where the Terraformer output is located
* How to navigate to it
* How to show the output files
* How to create a ZIP file
* How to verify the ZIP before sharing

## ✅ What we are zipping

Terraformer output directory:

```
/root/tf-scan-subscription-no-db/azurerm
```

This contains:

* `.tf` files
* `terraform.tfstate`
* provider files
* resource definitions

## 1. Navigate to Terraformer Output Directory

### Command

```bash
cd /root/tf-scan-subscription-no-db
```

### Purpose

Move to the parent directory containing Terraformer output.

---

## 2. Show Terraformer Output Folder

### Command

```bash
ls -la azurerm
```

### Purpose

Displays Terraformer-generated Azure Terraform files and folders.

---

## 3. Show Terraformer Files to Reviewer

### Command

```bash
find azurerm -type f | head -20
```

### Purpose

Lists sample Terraformer-generated files for verification.

---

## 4. Create ZIP of Terraformer Output

### Command

```bash
zip -r terraformer-output-azurerm.zip azurerm
```

### Purpose

Creates a ZIP archive of the complete Terraformer output directory.

---

## 5. Verify ZIP File Creation

### Command

```bash
ls -lh terraformer-output-azurerm.zip
```

### Purpose

Confirms ZIP file size and successful creation.

---

## 6. (Optional) View ZIP Contents

### Command

```bash
unzip -l terraformer-output-azurerm.zip | head -20
```

### Purpose

Shows files included inside the ZIP without extracting it.

---

## Final Output File

```
terraformer-output-azurerm.zip
```

This file contains the full Terraformer-generated Azure infrastructure output and is ready to be shared for review.
