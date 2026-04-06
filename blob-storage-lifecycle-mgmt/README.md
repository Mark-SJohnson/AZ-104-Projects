# Azure Blob Storage & Lifecycle Management
### AZ-104 Hands-On Project | Storage Domain

![Azure](https://img.shields.io/badge/Azure-Blob_Storage-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![Lifecycle](https://img.shields.io/badge/Azure-Lifecycle_Management-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![SAS](https://img.shields.io/badge/Azure-SAS_Tokens-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner--Intermediate-yellow?style=flat)
![Time](https://img.shields.io/badge/Est._Time-2--3_hours-green?style=flat)
![Cost](https://img.shields.io/badge/Cost-~$1--2-brightgreen?style=flat)

---

## Overview

This project covers core Azure Storage administration skills including blob container management, access tier configuration, automated lifecycle policies, versioning, SAS token-based access delegation, network security controls, and diagnostic logging. It simulates a data pipeline using three containers representing different stages of data processing.

**Domain covered:** Implement & Manage Storage (AZ-104 — 15–20% of exam weight)

---

## What I Built

- A General Purpose v2 storage account with LRS redundancy
- Three blob containers with different access levels simulating a data pipeline
- Blob-level access tier changes across Hot, Cool, and Archive tiers
- Lifecycle management policy automating tier transitions and deletion by age
- Blob versioning with soft delete for data protection and recovery
- An Azure File Share simulating a cloud-based SMB share
- SAS token with scoped read-only access and validated expiry behavior
- Storage account firewall restricting access to a known IP range
- Diagnostic logging forwarding storage telemetry to a Log Analytics workspace

---

## Architecture

```
Storage Account: storagedemoyourname (Standard LRS | General Purpose v2)
│
├── Containers
│   ├── raw-data          (Private)       ← Ingested/unprocessed data
│   ├── processed-data    (Private)       ← Transformed/cleaned data
│   └── public-assets     (Blob Public)   ← Publicly readable static assets
│
├── File Shares
│   └── fileshare-demo    (5GB quota)     ← SMB share for hybrid access
│
└── Configuration
    ├── Versioning: Enabled
    ├── Soft Delete (Blobs): 7 days
    ├── Soft Delete (Containers): 7 days
    ├── Lifecycle Policy: Hot → Cool → Archive → Delete
    ├── Firewall: Selected networks (known IP only)
    └── Diagnostics: StorageRead, StorageWrite, StorageDelete → Log Analytics
```

---

## Resources Created

| Resource | Type | Purpose |
|---|---|---|
| storagedemoyourname | Storage Account (GPv2) | Primary storage resource |
| raw-data | Blob Container (Private) | Raw ingested data |
| processed-data | Blob Container (Private) | Processed/transformed data |
| public-assets | Blob Container (Blob Public) | Publicly accessible static assets |
| fileshare-demo | Azure File Share (5GB) | Cloud SMB share |
| Lifecycle Policy | Management Policy | Automated tier transitions |
| Log Analytics Workspace | Monitoring | Diagnostic log destination |

---

## Blob Access Tiers

| Tier | Storage Cost | Access Cost | Minimum Duration | Use Case |
|---|---|---|---|---|
| Hot | Highest | Lowest | None | Frequently accessed data |
| Cool | Lower | Higher | 30 days | Infrequently accessed data |
| Cold | Lower still | Higher still | 90 days | Rarely accessed data |
| Archive | Lowest | Highest + rehydration | 180 days | Long-term archival |

> Blobs deleted before their minimum duration are charged for the remaining days.

---

## Lifecycle Management Policy

Automated policy applied to the `raw-data` container:

```json
{
  "rules": [
    {
      "name": "tiering-and-expiry",
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["raw-data/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
            "delete": { "daysAfterModificationGreaterThan": 365 }
          }
        }
      }
    }
  ]
}
```

**What this does:**
- Moves blobs to Cool tier after 30 days of no modification
- Moves blobs to Archive tier after 90 days
- Deletes blobs after 365 days
- Runs automatically with no manual intervention required after setup

---

## SAS Token Access Control

Generated a Shared Access Signature (SAS) token demonstrating scoped, time-limited access delegation:

| Setting | Value |
|---|---|
| Permissions | Read only |
| Expiry | 1 hour from generation |
| Scope | Single blob |
| Auth method | SAS token (no account key shared) |

**Validation performed:**
- Accessed blob via SAS URL in browser while token was valid ✅
- Attempted access after token expiry — authorization error returned ✅

---

## Versioning Validation

With versioning enabled, uploaded the same file twice with different content:

| Version | Is Current | Notes |
|---|---|---|
| Version 1 | No | Original upload — preserved automatically by Azure |
| Version 2 | Yes | Overwrite upload — becomes the current version |

- Downloaded Version 1 to confirm original content was preserved ✅
- Confirmed soft delete retains deleted blobs for 7 days before permanent removal ✅

---

## Steps Completed

- [x] Created resource group `rg-storage-demo`
- [x] Created General Purpose v2 storage account with LRS redundancy
- [x] Created three blob containers with appropriate access levels
- [x] Uploaded test files and changed individual blob access tiers
- [x] Configured lifecycle management policy for automated tiering
- [x] Enabled blob versioning and soft delete
- [x] Tested versioning by uploading same file twice with different content
- [x] Created Azure File Share `fileshare-demo` with 5GB quota
- [x] Generated SAS token with read-only access and validated expiry
- [x] Configured storage account firewall to restrict access by IP
- [x] Enabled diagnostic logging to Log Analytics workspace
- [x] Reviewed storage metrics (Transactions, Ingress, Egress)
- [x] Deleted resource group to complete teardown

---

## Key Concepts Demonstrated

**Lifecycle policies automate cost optimization** — instead of manually moving blobs between tiers, rules run on a schedule and transition blobs based on age. This is essential for managing storage at scale.

**Archive blobs cannot be read directly** — they must be rehydrated to Hot or Cool tier first, which can take several hours. Early deletion before the 180-day minimum incurs a penalty charge.

**SAS tokens vs account keys** — sharing an account key gives full access to the entire storage account. SAS tokens scope access to specific resources, permissions, and time windows. Always use SAS for external access delegation.

**Versioning adds billing overhead** — each version is stored and billed as a separate blob. In production, lifecycle policies should include version cleanup rules to control costs.

**Soft delete and versioning work together** — soft delete protects against accidental deletion, versioning protects against accidental overwrites. Both should be enabled in production environments.

**Storage firewall adds a network layer** — restricting access to known IPs or VNets prevents unauthorized access even if credentials are compromised.

---

## Technologies Used

- Microsoft Azure Portal
- Azure Blob Storage (General Purpose v2)
- Azure File Shares
- Azure Lifecycle Management
- Shared Access Signatures (SAS)
- Azure Monitor / Log Analytics
- Azure CLI

---

## CLI Reference

```bash
# Create storage account
az storage account create \
  --name storagedemoyourname \
  --resource-group rg-storage-demo \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot

# Create blob container
az storage container create \
  --name raw-data \
  --account-name storagedemoyourname \
  --public-access off

# Upload a file
az storage blob upload \
  --account-name storagedemoyourname \
  --container-name raw-data \
  --name testfile.txt \
  --file ./testfile.txt

# Change blob access tier
az storage blob set-tier \
  --account-name storagedemoyourname \
  --container-name raw-data \
  --name testfile.txt \
  --tier Cool

# Generate SAS token
az storage blob generate-sas \
  --account-name storagedemoyourname \
  --container-name raw-data \
  --name testfile.txt \
  --permissions r \
  --expiry 2026-01-01T00:00:00Z

# List all blobs in a container
az storage blob list \
  --account-name storagedemoyourname \
  --container-name raw-data \
  --output table
```

---

## Related AZ-104 Exam Objectives

- Configure Azure Blob Storage
- Configure storage tiers
- Configure blob lifecycle management
- Configure Azure Files and Azure Blob Storage
- Create and configure a storage account
- Configure stored access policies and SAS

---

*Part of a 5-project AZ-104 hands-on portfolio*
