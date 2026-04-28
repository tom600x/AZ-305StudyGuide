# Azure Migrations — Deep-Dive Study Guide for AZ-305

## Why Migrations Matter on AZ-305
The exam tests your ability to:
- Recommend migration strategies (Rehost/Refactor/Rearchitect/Rebuild/Replace)
- Select the right Azure Migrate tools for different scenarios
- Choose Database Migration Service for database migrations
- Select data transfer tools based on data volume and connectivity
- Understand the Cloud Adoption Framework (CAF) migration methodology

---

## 1. Cloud Adoption Framework (CAF) — Migration

### CAF Lifecycle Overview

```
Strategy → Plan → Ready → Adopt → Govern → Manage
                          ↑
                      Migrate (sub-phase of Adopt)
```

### CAF Migration Phases

| Phase | Description |
|---|---|
| **Assess** | Discover workloads, analyze dependencies, right-size, calculate TCO |
| **Deploy** | Replicate workloads to Azure, test in Azure |
| **Release** | Cut over traffic to Azure, decommission on-premises |

### The 6 R's of Migration (Rationalization)

| Strategy | Name | Description | Tools | Example |
|---|---|---|---|---|
| **Rehost** | Lift-and-Shift | Move as-is to IaaS VMs | Azure Migrate | Move SQL Server VM to Azure VM |
| **Refactor** | Move to PaaS | Minor changes to use PaaS | App Service Migration Assistant | ASP.NET app → App Service |
| **Rearchitect** | Modify architecture | Change code significantly for cloud | Dev team + ADF | Monolith → microservices |
| **Rebuild** | Rewrite | Build from scratch on cloud-native | Dev team | Legacy app → Azure Functions |
| **Replace** | Use SaaS | Replace with SaaS solution | N/A | CRM → Dynamics 365 |
| **Retire** | Decommission | No longer needed | N/A | Unused legacy apps |

> **Exam tip:** The 6 R's are often tested with scenario questions. Recognize: minimal effort = Rehost; use PaaS with minor code changes = Refactor; major code changes = Rearchitect.

---

## 2. Azure Migrate

### What It Is
Azure Migrate is the **central hub for discovery, assessment, and migration** of on-premises servers, databases, web apps, and virtual desktops to Azure.

### Azure Migrate Tools

| Tool | Purpose |
|---|---|
| **Azure Migrate: Discovery and Assessment** | Agentless discovery of VMs, dependencies, performance data |
| **Azure Migrate: Server Migration** | Replicate and migrate VMs to Azure |
| **Database Assessment (Database Migration Guide)** | Assess SQL Server databases for Azure readiness |
| **Web App Assessment** | Assess IIS web apps for App Service migration |
| **Azure VMware Solution Assessment** | Assess VMware VMs for AVS |

### Assessment Types

| Assessment Type | Description |
|---|---|
| **Azure VM Assessment** | Right-size on-premises VMs for Azure VMs |
| **Azure VMware Solution (AVS)** | Assess for running VMs as-is in AVS |
| **Azure App Service** | Assess IIS web apps for App Service migration |
| **Azure SQL** | Assess on-premises SQL Server for SQL DB, SQL MI, or SQL on VM |

### Sizing Approaches

| Approach | Description | Use When |
|---|---|---|
| **As on-premises** | Size Azure VMs to match on-premises config | Predictable, consistent loads |
| **Performance-based** | Right-size based on actual CPU/memory utilization data | Want to right-size, reduce cost |

### Dependency Analysis

| Type | Description |
|---|---|
| **Agentless** | Uses vCenter APIs to collect data, no agent needed | Quick assessment, minimal setup |
| **Agent-based** | Install Microsoft Monitoring Agent (MMA) + Dependency agent on each VM | Detailed process-level mapping |

> **Exam tip:** Agentless dependency analysis does not require installing agents — it uses hypervisor-level data collection. Agent-based gives more detail (process names, port numbers).

### Azure Migrate Appliance
- Lightweight virtual appliance deployed on-premises
- Performs continuous discovery and assessment
- Supports VMware, Hyper-V, and physical servers

---

## 3. Azure Database Migration Service (DMS)

### What It Is
Azure Database Migration Service automates **database migration** from on-premises sources to Azure database targets with minimal downtime.

### Migration Modes

| Mode | Description | Downtime |
|---|---|---|
| **Online** | Continuous sync — data replicated while source stays live | Minimal (cutover only) |
| **Offline** | One-time snapshot migration | Downtime during migration |

> **Exam tip:** Online migration = minimal downtime, requires more setup. Offline migration = simpler, but requires downtime window.

### Supported Sources and Targets

| Source | Azure Target |
|---|---|
| SQL Server (on-premises) | Azure SQL Database, Azure SQL MI |
| MySQL (on-premises) | Azure Database for MySQL |
| PostgreSQL (on-premises) | Azure Database for PostgreSQL |
| MongoDB | Azure Cosmos DB API for MongoDB |
| Oracle | Azure Database for PostgreSQL |

### DMS vs DIY Migration

| Approach | Use When |
|---|---|
| Azure DMS | Automated, validated migration with minimal downtime |
| Backup/Restore | Simple offline migration, full control |
| Log Shipping | Manual Change Data Capture (CDC) for SQL Server → SQL MI |
| SQL Server Migration Assistant (SSMA) | Schema + data migration for non-SQL sources (Oracle, MySQL) |

---

## 4. Server Migration with Azure Migrate

### VMware VMs
- Agentless replication using vCenter snapshot-based replication
- Up to 500 VMs per appliance with agentless migration
- Agent-based replication uses Mobility Service agent

### Hyper-V VMs
- Uses Hyper-V Replication Provider (installed on Hyper-V host)
- Replicates directly to Azure Recovery Services vault

### Physical Servers
- Uses Mobility Service agent installed on each server
- Works for bare metal, other hypervisors (Xen, Kernel-based Virtual Machine (KVM)), cloud VMs

### Migration Process (General)
1. **Discover** — Deploy appliance, discover VMs
2. **Assess** — Analyze readiness, right-size, estimate cost
3. **Replicate** — Set up replication, initial sync
4. **Test Migration** — Test failover to validate (recommended step)
5. **Migrate** — Cutover, stop replication

---

## 5. Unstructured Data Migration

For migrating large amounts of files, blobs, and object storage.

### Azure Storage Mover
- Managed service specifically for migrating files and blob data to Azure Storage
- Supports: NFS shares, SMB shares, Azure Files, Azure Blob Storage
- Agent-based (deploy agents on-premises)
- Provides job tracking and migration progress

### AzCopy
- Command-line tool for copying data to/from Azure Storage
- Supports Blob, Files, Data Lake Gen2
- Incremental sync mode
- Best for: Scripted migrations, DevOps automation, smaller datasets

### Azure Data Box — Offline Transfer Options

| Device | Capacity | Use Case |
|---|---|---|
| **Data Box Disk** | Up to 35 TB (5 disks × 8 TB) | Small-medium offline transfers |
| **Data Box** | 80 TB usable | Medium-large offline transfers |
| **Data Box Heavy** | 770 TB usable | Very large (hundreds of TB to PB) |

> **Decision Rule:** Use Data Box when uploading would take **more than 7–14 days** over your internet connection, or when connectivity is unreliable or unavailable.

### Transfer Method Decision Tree

```
Need to move data to Azure?
├── Data < 40 TB AND good internet connectivity
│   ├── Files/blobs → AzCopy or Azure Storage Mover
│   └── Databases → DMS or backup/restore
└── Data > 40 TB OR poor internet connectivity
    ├── < 35 TB → Data Box Disk
    ├── 35–80 TB → Data Box
    └── > 80 TB → Data Box Heavy
```

---

## 6. Web App Migration

### App Service Migration Assistant
- Tool to assess and migrate IIS web apps to Azure App Service
- Analyzes app compatibility, generates ARM templates
- Handles some automatic remediation of common issues

### Containerization
- Azure Migrate: App Containerization tool
- Containerizes Java or ASP.NET apps into Docker images
- Deploys to App Service for Containers or AKS
- No code changes required for basic containerization

---

## 7. Well-Known Migration Patterns

### SQL Server to Azure SQL MI (Lift-and-Shift)
- Nearly 100% SQL Server feature compatibility
- Use when: Need SQL Agent, cross-database queries, CLR, linked servers
- Migration: DMS online migration or backup to Azure Blob + restore

### SQL Server to Azure SQL Database (Refactor)
- Remove dependencies on SQL Server-specific features
- Gain: Automatic tuning, serverless, built-in HA, elastic pools
- Migration: DMS or BACPAC export/import

### On-Premises VMs to AKS (Rearchitect)
1. Containerize with Azure Migrate App Containerization
2. Deploy to AKS with Helm charts
3. Migrate databases separately (DMS)

---

## 8. Exam Scenario Cheat Sheet

| Scenario | Answer |
|---|---|
| Discover and assess 500 on-premises VMs | Azure Migrate appliance + Discovery and Assessment tool |
| Migrate VMware VMs to Azure with minimal changes | Azure Migrate Server Migration (agentless replication) |
| Move SQL Server to Azure with minimal downtime | Azure DMS online migration to Azure SQL MI |
| Migrate 200 TB of files, no reliable internet | Azure Data Box |
| Migrate 20 TB of files over good internet connection | AzCopy or Azure Storage Mover |
| Migrate legacy app to Dynamics 365 | Replace (6 R's) |
| Containerize IIS app, no code changes, deploy to AKS | Azure Migrate App Containerization |
| Understand dependency relationships before migration | Azure Migrate agentless dependency analysis |
| Migrate MongoDB to Azure | Azure DMS → Cosmos DB API for MongoDB |
| On-premises app using SQL Agent, linked servers | Migrate to Azure SQL Managed Instance |
| Assess migration cost and right-sizing | Azure Migrate performance-based assessment |
| Migrate MySQL database, near-zero downtime | Azure DMS online migration to Azure DB for MySQL |
