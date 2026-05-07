# Azure Storage — Deep-Dive Study Guide for AZ-305

## Why Storage Matters on AZ-305
The exam tests your ability to **choose the right storage service and the right SKU** based on:
- Performance requirements (IOPS, throughput, latency)
- Data type (blobs, files, tables, queues, disks)
- Access patterns (hot/cool/archive, random vs sequential)
- Redundancy and compliance requirements
- Cost constraints

---

## 1. Azure Storage Account Types

### General Purpose v2 (GPv2) — Default Choice
- Supports **Blob, File, Queue, Table, and Data Lake Storage Gen2**
- All access tiers: Hot, Cool, Cold, Archive
- Zone-redundant, geo-redundant options
- **Choose when:** You need a versatile account for most workloads

### General Purpose v1 (GPv1) — Legacy
- Lacks access tiering (no Cool/Archive)
- Slightly lower transaction cost but no modern features
- **Avoid for new designs** — migrate to GPv2

### BlockBlobStorage (Premium)
- Optimized for **high transaction rates** with small blobs
- Low latency (~1ms)
- No Cool/Archive tiering
- **Choose when:** IoT telemetry, log pipelines, event streams needing fast writes

### FileStorage (Premium)
- Premium performance for **Azure Files only**
- Supports **NFS 4.1 and SMB 3.x**
- SSD-backed
- **Choose when:** SAP, Oracle, or other enterprise apps requiring high IOPS file shares

### BlobStorage (Legacy)
- Only Block and Append blobs (no Page blobs, no Files)
- Replaced by GPv2 — **avoid for new designs**

---

## 2. Azure Blob Storage

### Blob Types

| Type | Best For |
|---|---|
| **Block Blob** | Documents, images, video, backups — most common |
| **Append Blob** | Log files, audit streams (append-only operations) |
| **Page Blob** | VM OS/data disks (random read/write, 512-byte pages) |

### Access Tiers

| Tier | Latency | Storage Cost | Access Cost | Minimum Duration |
|---|---|---|---|---|
| **Hot** | Milliseconds | Highest | Lowest | None |
| **Cool** | Milliseconds | Lower | Higher | 30 days |
| **Cold** | Milliseconds | Lower than Cool | Higher than Cool | 90 days |
| **Archive** | Hours (rehydrate) | Lowest | Highest | 180 days |

**Key exam scenarios:**
- **Frequently accessed data** → Hot
- **Backups kept 30–90 days** → Cool or Cold
- **Long-term compliance archives** → Archive
- **Early deletion penalty** → Charges apply if deleted before minimum duration

### Lifecycle Management
- Automatically transition blobs between tiers or delete based on age/last-modified
- Policies run daily, defined in JSON rules
- **Exam tip:** Use lifecycle policies to minimize costs without manual intervention

### Immutable Storage — Write Once, Read Many (WORM)
- **Time-based retention:** Blob locked for a defined period
- **Legal hold:** Indefinite hold for litigation
- **Choose when:** Financial records, HIPAA, SEC 17a-4 compliance

---

## 3. Azure Files

### Protocols
| Protocol | Use Case |
|---|---|
| **SMB 3.x** | Windows, Linux (with Kerberos), cloud-only or hybrid |
| **NFS 4.1** | Linux-only, premium tier required, no auth (IP-level) |
| **REST** | Any platform via HTTP/S |

### Performance Tiers

| Tier | Storage Account | IOPS | Throughput | Use Case |
|---|---|---|---|---|
| **Transaction Optimized** | GPv2 | Up to 10k | 300 MB/s | General shares, moderate I/O |
| **Hot** | GPv2 | Up to 10k | 300 MB/s | Frequently accessed shares |
| **Cool** | GPv2 | Up to 10k | 300 MB/s | Infrequently accessed, lower cost |
| **Premium** | FileStorage | Up to 100k+ | Scales with provisioned GiB | SAP, databases, low-latency apps |

### Azure File Sync
- Extends on-premises file servers to Azure
- Cloud tiering: rarely accessed files stored in Azure, hot files cached locally
- **Choose when:** Hybrid file share replacement or branch office consolidation

---

## 4. Azure Disk Storage (Managed Disks)

### Disk Types (SKUs)

| Type | Max IOPS | Max Throughput | Max Size | Use Case |
|---|---|---|---|---|
| **Ultra Disk** | 160,000 | 2,000 MB/s | 65,536 GiB | SAP HANA, top-tier SQL, real-time analytics |
| **Premium SSD v2** | 80,000 | 1,200 MB/s | 65,536 GiB | Business-critical apps, lower cost than Ultra |
| **Premium SSD v1** | 20,000 | 900 MB/s | 32,767 GiB | SQL Server, enterprise apps, VM OS disks |
| **Standard SSD** | 6,000 | 750 MB/s | 32,767 GiB | Web servers, dev/test, lightly used apps |
| **Standard HDD** | 2,000 | 500 MB/s | 32,767 GiB | Backup, non-critical, archive |

### Key Decision Factors
- **Ultra Disk / Premium SSD v2:** Sub-millisecond latency, configurable IOPS/throughput without resizing
- **Premium SSD v1:** Predictable performance, widest VM compatibility
- **Standard SSD:** Consistent latency for non-critical workloads
- **Standard HDD:** Lowest cost, tolerate high latency

### Exam Tips
- Ultra Disk requires supported VM series and is **zone-specific**
- Premium SSD v2 can **adjust IOPS/throughput independently** — no downtime
- **Shared disks** (Premium SSD, Ultra) enable Windows Server Failover Clustering (WSFC)
- OS disk → typically Premium SSD; Data disk → depends on IOPS requirement

---

## 5. Azure Queue Storage

- Simple message queue, up to **64 KB per message**, **500 TB** total
- Messages survive up to **7 days**
- **Choose when:** Decoupling components in a simple architecture, or when you don't need ordering or dead-lettering
- **vs. Service Bus Queues:** Service Bus offers ordering, dead-letter, sessions, topics — use for enterprise messaging

---

## 6. Azure Table Storage

- NoSQL key-value store (PartitionKey + RowKey)
- Cheap, massively scalable, schemaless
- No secondary indexes, no stored procedures
- **Choose when:** Simple lookups, telemetry metadata, semi-structured data at low cost
- **vs. Cosmos DB Table API:** Same data model but Cosmos adds global distribution, SLAs, multiple consistency levels, secondary indexes

---

## 7. Redundancy SKUs (Critical for Exam)

| SKU | Full Name | Copies | Zone-Aware | Region |
|---|---|---|---|---|
| **LRS** | Locally Redundant Storage | 3 | No | Single datacenter |
| **ZRS** | Zone-Redundant Storage | 3 | Yes | 3 AZs in one region |
| **GRS** | Geo-Redundant Storage | 6 | No | Primary (3 LRS) + Secondary (3 LRS) |
| **GZRS** | Geo-Zone-Redundant Storage | 6 | Yes in primary | Primary (3 ZRS) + Secondary (3 LRS) |
| **RA-GRS** | Read-Access GRS | 6 | No | GRS + read from secondary |
| **RA-GZRS** | Read-Access GZRS | 6 | Yes | GZRS + read from secondary |

### How to Choose Redundancy

```
Is data loss across zones acceptable?
  YES → LRS (cheapest)
  NO  → ZRS

Is regional outage unacceptable?
  YES → GRS or GZRS
  
Do you need reads during a regional failover?
  YES → RA-GRS or RA-GZRS
  
Do you need zone protection in the primary region too?
  YES → GZRS or RA-GZRS (best protection)
```

**Exam scenarios:**
- **Compliance requiring data stays in one region** → LRS or ZRS
- **RPO < 15 min, survive region failure** → GZRS
- **Read workloads during DR** → RA-GRS or RA-GZRS
- **Lowest cost for dev/test** → LRS

---

## 8. Security & Access

### Authorization Methods

| Method | Best For |
|---|---|
| **Azure Role-Based Access Control (RBAC)** | Fine-grained, identity-based, auditable — preferred |
| **Shared Access Signature (SAS)** | Delegated, time-limited access for external users |
| **Storage Account Keys** | Full access, avoid in production — use managed identity instead |
| **Anonymous public access** | Static websites, public blobs only |

### SAS Types
- **Service SAS:** Access to one service (blob, file, queue, table)
- **Account SAS:** Access to one or more services
- **User Delegation SAS:** Backed by Azure AD — most secure form of SAS

### Encryption
- **At rest:** AES-256, enabled by default (Microsoft-managed keys)
- **Customer-managed keys (CMK):** Key Vault integration, required for compliance
- **Infrastructure encryption:** Double encryption at hardware layer
- **In transit:** TLS 1.2+ enforced (can require minimum TLS version)

---

## 9. Data Lake Storage Gen2

- Built on **Blob Storage** with hierarchical namespace enabled
- Supports **Portable Operating System Interface (POSIX)-style ACLs**
- Optimized for analytics (Hadoop, Spark, Synapse, Databricks)
- **Choose when:** Big data analytics, data lakehouse architectures
- **vs. Blob Storage without Hierarchical Namespace (HNS):** HNS enables true directory semantics and atomic rename

---

## 10. Storage Decision Tree

```
What are you storing?

├── Unstructured files (docs, images, video, backups)
│   └── Blob Storage (GPv2) → tier: Hot/Cool/Archive
│
├── File shares (SMB/NFS)
│   ├── Standard → Azure Files (GPv2)
│   └── High IOPS / enterprise → Azure Files Premium (FileStorage)
│
├── VM disks
│   ├── OS disk → Premium SSD v1
│   ├── Data disk (moderate) → Premium SSD v1 or Standard SSD
│   └── Data disk (extreme IOPS) → Ultra Disk or Premium SSD v2
│
├── Messages / queues
│   ├── Simple decoupling → Queue Storage
│   └── Enterprise (ordering, sessions, topics) → Service Bus
│
├── NoSQL key-value
│   ├── Simple / cheap → Table Storage
│   └── Global, SLA, advanced queries → Cosmos DB Table API
│
└── Big data analytics
    └── Data Lake Storage Gen2
```

---

## 11. Exam Scenario Cheat Sheet

| Scenario | Answer |
|---|---|
| Archive regulatory data for 7 years, lowest cost | Blob Archive tier + lifecycle policy + WORM |
| Lift-and-shift Windows file server to Azure, hybrid | Azure Files + Azure File Sync |
| Linux app needs NFS share with low latency | Azure Files Premium (NFS 4.1) |
| App writes millions of small log entries per second | BlockBlobStorage (Premium) + Append Blobs |
| SQL Server on Azure VM, needs consistent IOPS | Premium SSD v1 (or v2 for configurable IOPS) |
| Protect against datacenter failure, not region failure | ZRS |
| Active-active geo-redundant reads | RA-GZRS |
| Store Spark/Databricks analytics data | Data Lake Storage Gen2 |
| External partner needs 24hr access to specific blob | User Delegation SAS |
| Compliance: data must stay in one country | LRS or ZRS (no geo-replication) |

---

## 12. Key Service Limits to Know

| Resource | Limit |
|---|---|
| Max storage account capacity | 5 PiB |
| Max blob size (block blob) | ~190.7 TiB |
| Max single file share size | 100 TiB (Premium), 100 TiB (Standard with large file enabled) |
| Max queue message size | 64 KB |
| Archive rehydration time | Up to 15 hours (standard priority) or 1 hour (high priority) |
| SAS token max expiry | No hard limit, but best practice ≤ 1 hour |

---

## 13. Storage Exam Traps

### 1. Choosing geo-redundancy when data residency forbids cross-region copies
- **Trap:** GRS and GZRS sound safer, so they look like the best default
- **Better default:** LRS or ZRS when data must remain in a single region or country

### 2. Choosing Archive tier for data that must be accessed quickly
- **Trap:** Archive is cheapest
- **Better default:** Cool or Hot when retrieval time matters; Archive has rehydration delay and retrieval cost

### 3. Choosing Azure Files when the scenario is really object storage or analytics lake storage
- **Trap:** Files sounds like a general-purpose storage answer
- **Better default:** Blob Storage or ADLS Gen2 depending on object semantics and analytics needs

### 4. Confusing SAS types and over-granting access
- **Trap:** Any SAS token seems good enough
- **Better default:** Prefer User Delegation SAS when possible and scope permissions tightly

### 5. Choosing Service Endpoints when private IP-based access is required
- **Trap:** Service Endpoints secure access to storage from a VNet
- **Better default:** Private Endpoint when the requirement explicitly says no public endpoint or private IP access

### Rapid Elimination Rules

| Requirement | Eliminate First |
|---|---|
| Data must stay in one geography | GRS and GZRS answers |
| Lowest cost but retrieval can wait hours | Hot-tier answers |
| NFS file share for Linux app | Blob-only answers |
| Spark or lake analytics data | Azure Files answers |
| External partner needs temporary blob access | Account key sharing answers |
