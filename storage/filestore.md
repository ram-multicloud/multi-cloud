# Azure File Storage

## Overview

Azure offers four managed file storage services, each designed for different workloads and performance requirements.

| Service | Best For | Protocol | Latency |
|---|---|---|---|
| Azure Files | General cloud file shares | SMB, NFS, REST | Low (ms) |
| Azure File Sync | Hybrid on-prem + cloud | SMB (local) | Local cache speed |
| Azure NetApp Files | Enterprise NAS, SAP HANA | NFS, SMB | Sub-millisecond |
| Azure Managed Lustre | HPC, AI/ML training | Lustre (POSIX) | Sub-millisecond |

---

## Azure Files

A fully managed cloud file share service accessible via **SMB**, **NFS**, and **REST** protocols.

### a. Classic File Share (Standard)

HDD-backed storage with three cost tiers:

- **Transaction Optimized** — high transaction workloads
- **Hot** — frequently accessed files
- **Cool** — backup and archival

**Key specs:**
- Max share size: 100 TiB
- Max IOPS: 10,000
- Storage type: HDD
- Protocols: SMB 2.1/3.x, NFS 4.1, REST

**Use cases:**
- Lift-and-shift of legacy applications
- Shared configuration files for containerized apps
- Dev/test environments
- Home directories for users

### b. Premium File Share

SSD-backed storage with provisioned billing:

- Max share size: 100 TiB
- Max IOPS: 100,000
- Storage type: SSD
- Protocols: SMB 3.x, NFS 4.1

**Use cases:**
- Latency-sensitive databases
- Virtual Desktop Infrastructure (VDI)
- High-performance applications

### Common Features (Both Share Types)

| Feature | Details |
|---|---|
| Authentication | AD DS, Azure AD DS, Azure AD Kerberos, storage key |
| Encryption at rest | AES-256 |
| Encryption in transit | SMB 3.x |
| Snapshots | Point-in-time read-only copies |
| Soft delete | Protect shares from accidental deletion |

### CLI — Create a File Share

```bash
# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group myRG \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2

# Get storage key
KEY=$(az storage account keys list \
  --account-name mystorageaccount \
  --resource-group myRG \
  --query "[0].value" -o tsv)

# Create standard file share
az storage share create \
  --name myfileshare \
  --account-name mystorageaccount \
  --account-key "$KEY" \
  --quota 100

# Create premium file share
az storage share-rm create \
  --name premiumshare \
  --storage-account mystorageaccount \
  --enabled-protocol SMB \
  --quota 100
```

---

## Azure File Sync

Extends Azure Files by **caching file shares on Windows Servers on-premises** — a hybrid bridge between cloud and local.

### Architecture

```
On-Premises                          Azure Cloud
┌──────────────┐    Sync        ┌──────────────────┐
│Windows Server│◄──────────────►│  Azure File Share │
│  (hot data)  │                │  (full dataset)   │
└──────────────┘                └──────────────────┘
  Local cache only
```

### Key Components

| Component | Description |
|---|---|
| Storage Sync Service | Top-level Azure resource |
| Sync Group | Defines topology (1 cloud + N server endpoints) |
| Registered Server | Windows Server enrolled in sync |
| Server Endpoint | A local path on the Windows Server |
| Cloud Endpoint | The Azure File Share |
| Cloud Tiering | Stubs cold files locally; recalls on demand |

### Cloud Tiering

- Rarely accessed files replaced by **lightweight pointer stubs** on the server
- Files recalled **transparently** when a user opens them
- Full directory namespace remains visible even for stubbed files
- Configurable by free space policy or date policy

### Use Cases

- Modernize file servers without changing user workflows
- Multi-site file share replication (branch offices)
- Backup and disaster recovery for on-prem file servers
- Branch office caching of centralized data

### CLI — Setup File Sync

```bash
# Create Storage Sync Service
az storagesync create \
  --name myStorageSyncService \
  --resource-group myRG \
  --location eastus

# Create Sync Group
az storagesync sync-group create \
  --storage-sync-service myStorageSyncService \
  --resource-group myRG \
  --name mySyncGroup

# Create Cloud Endpoint
az storagesync sync-group cloud-endpoint create \
  --storage-sync-service myStorageSyncService \
  --sync-group-name mySyncGroup \
  --resource-group myRG \
  --name myCloudEndpoint \
  --storage-account-resource-id /subscriptions/.../storageAccounts/mystorageaccount \
  --azure-file-share-name myfileshare
```

---

## Azure NetApp Files (ANF)

An enterprise-grade, high-performance NAS service built on **NetApp ONTAP** technology, fully managed by Azure. Runs on bare-metal NetApp hardware — not emulated.

### Architecture

```
NetApp Account
└── Capacity Pool (e.g., 4 TiB, Ultra tier)
     ├── Volume 1 — NFS v4.1 — 1 TiB
     ├── Volume 2 — SMB 3.x — 500 GiB
     └── Volume 3 — Dual protocol — 2 TiB
```

### Performance Tiers

| Tier | Throughput per TiB | Use Case |
|---|---|---|
| Standard | 16 MiB/s | General workloads, file archiving |
| Premium | 64 MiB/s | Databases, ERP, Oracle |
| Ultra | 128 MiB/s | SAP HANA, HPC, VDI at scale |

### Key Features

- **Protocols:** NFS v3/v4.1, SMB 3.x, dual-protocol (both simultaneously)
- **Latency:** Sub-millisecond on bare-metal hardware
- **Snapshots:** Instant point-in-time copies (space-efficient)
- **SnapMirror:** Cross-region volume replication
- **Volume cloning:** Instant writable clones for dev/test
- **SAP HANA certified:** Meets SAP performance requirements

### Use Cases

- SAP HANA and SAP S/4HANA workloads
- High-performance databases (Oracle, SQL Server)
- HPC simulations and rendering
- VDI at scale (persistent desktops)
- Analytics pipelines requiring fast NFS

### CLI — Create a NetApp Volume

```bash
# Create NetApp account
az netappfiles account create \
  --resource-group myRG \
  --location eastus \
  --name myNetAppAccount

# Create capacity pool
az netappfiles pool create \
  --resource-group myRG \
  --location eastus \
  --account-name myNetAppAccount \
  --name myPool \
  --size 4 \
  --service-level Ultra

# Create volume
az netappfiles volume create \
  --resource-group myRG \
  --location eastus \
  --account-name myNetAppAccount \
  --pool-name myPool \
  --name myVolume \
  --service-level Ultra \
  --usage-threshold 1099511627776 \
  --protocol-types NFSv3 \
  --vnet myVNet \
  --subnet mySubnet \
  --file-path myvolumepath
```

---

## Azure Managed Lustre

A fully managed **parallel file system** based on open-source Lustre — purpose-built for massive-scale HPC and AI/ML workloads.

### How Lustre Works

Lustre distributes data across many **Object Storage Targets (OSTs)** simultaneously. Hundreds of compute nodes can read and write in parallel, so aggregate throughput scales with the number of OSTs.

### Architecture

```
GPU / HPC Cluster (clients)
        │ Lustre client (POSIX)
        ▼
┌───────────────────────────────┐
│   Managed Lustre File System   │
│   (parallel OSTs)             │
└──────────────┬────────────────┘
               │ import / export
               ▼
       Azure Blob Storage
```

### Key Features

- **Throughput:** Scales from 500 MB/s up to tens of GB/s
- **Protocol:** Native Lustre client (POSIX-compliant)
- **Blob integration:** Import data from Blob before job; export results after
- **Ephemeral by design:** Spin up for a job, tear down after
- **POSIX compliant:** Existing HPC apps work with zero code changes
- **Min size:** 48 TiB (increments of 48 TiB)

### SKUs

| SKU | MB/s per TiB |
|---|---|
| AMLFS-Durable-Premium-40 | 40 |
| AMLFS-Durable-Premium-125 | 125 |
| AMLFS-Durable-Premium-250 | 250 |
| AMLFS-Durable-Premium-500 | 500 |

### Use Cases

- AI/ML model training (fast checkpoint reads/writes for GPU clusters)
- Genomics and life sciences data processing
- Seismic processing (oil and gas simulations)
- CFD/FEA simulations (aerospace, automotive HPC)
- Rendering farms (visual effects pipelines)

### CLI — Create Managed Lustre

```bash
az amlfs create \
  --name myLustreFS \
  --resource-group myRG \
  --location eastus \
  --sku AMLFS-Durable-Premium-250 \
  --storage-capacity-tib 48 \
  --subnet-id /subscriptions/.../subnets/mySubnet \
  --maintenance-window \
    dayOfWeek=Saturday timeOfDayUtc=22:00
```

---

## Side-by-Side Comparison

| | Azure Files | File Sync | NetApp Files | Managed Lustre |
|---|---|---|---|---|
| **Best for** | General file shares | Hybrid on-prem+cloud | Enterprise NAS / SAP | HPC / AI-ML |
| **Protocol** | SMB, NFS, REST | SMB | NFS, SMB | Lustre (POSIX) |
| **Latency** | Low (ms) | Local cache speed | Sub-ms | Sub-ms |
| **Max throughput** | ~10 GB/s | Server-limited | ~4.5 GB/s/vol | Tens of GB/s |
| **On-prem integration** | Mount as drive | Native agent sync | Via VNet | Cloud-native |
| **Managed** | Fully | Agent-based | Fully | Fully |
| **Min size** | 1 GiB | N/A | 4 TiB (pool) | 48 TiB |
| **Cost tier** | Low | Low + server cost | High | Medium |

---

## Quick Decision Guide

```
Need a simple cloud file share (SMB/NFS)?
  └── Azure Files — Standard (classic share)

Need low-latency SSD-backed share for apps?
  └── Azure Files — Premium

Still using on-prem Windows file servers?
  └── Azure File Sync (hybrid caching)

Running SAP HANA, Oracle, or enterprise NAS?
  └── Azure NetApp Files

Training AI/ML models or running HPC jobs?
  └── Azure Managed Lustre
```
