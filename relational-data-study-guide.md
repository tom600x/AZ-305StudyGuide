# Azure Relational Data — Deep-Dive Study Guide for AZ-305

## Why Relational Data Matters on AZ-305
The exam tests your ability to:
- Choose between Azure SQL Database, SQL Managed Instance, and SQL Server on VMs
- Select the correct purchasing model (DTU vs vCore) and service tier
- Design for scalability (elastic pools, read scale-out, Hyperscale)
- Protect data (TDE, Dynamic Data Masking, Always Encrypted, backups)
- Design for high availability and disaster recovery

---

## 1. Azure SQL Service Options — Which One to Choose

| Service | Description | Best For |
|---|---|---|
| **Azure SQL Database** | Fully managed PaaS, single database or elastic pool | New cloud-native apps, SaaS, variable workloads |
| **Azure SQL Managed Instance** | PaaS with near 100% SQL Server compatibility | Lift-and-shift from on-premises SQL Server |
| **SQL Server on Azure VM (IaaS)** | Full control, OS-level access | Requires OS access, unsupported features, specific SQL versions |
| **Azure Database for PostgreSQL** | Managed PostgreSQL | Open-source PostgreSQL workloads |
| **Azure Database for MySQL** | Managed MySQL | Open-source MySQL, LAMP stack, WordPress |

### Decision Tree
```
Need OS-level access or unsupported SQL features?
  YES → SQL Server on Azure VM

Migrating on-premises SQL Server with minimal code changes?
  YES, complex dependencies → SQL Managed Instance
  YES, simple migration → SQL Database

Building new cloud-native app?
  YES → Azure SQL Database

Open-source preference?
  PostgreSQL → Azure Database for PostgreSQL (Flexible Server)
  MySQL     → Azure Database for MySQL (Flexible Server)
```

---

## 2. Azure SQL Database — Purchasing Models

### DTU-Based Model (Legacy, Simpler)
- **DTU = Database Transaction Unit** — bundled blend of CPU, memory, read, and write
- Fixed resource ratios, easier to understand
- Service tiers:

| Tier | DTUs | Storage | Use Case |
|---|---|---|---|
| **Basic** | 5 DTUs | Up to 2 GB | Dev/test, low traffic |
| **Standard (S0–S12)** | 10–3,000 DTUs | Up to 1 TB | General business workloads |
| **Premium (P1–P15)** | 125–4,000 DTUs | Up to 4 TB | High IOPS, low latency, business critical |

- **DTU to vCore rule:** 100 Standard DTUs ≈ 1 vCore GP; 125 Premium DTUs ≈ 1 vCore BC

### vCore-Based Model (Recommended for New Deployments)
- Separate control over CPU cores, memory, and storage
- Enables Azure Hybrid Benefit (use on-premises SQL Server licenses)
- Two compute tiers:
  - **Provisioned:** Fixed compute, billed per hour
  - **Serverless:** Auto-scale compute based on demand, pause when idle — billed per second

| Service Tier | HA Replicas | Read Scale-Out | Use Case |
|---|---|---|---|
| **General Purpose** | 1 (Log-based HA) | No | Most business workloads, balanced price/performance |
| **Business Critical** | 3 (Always On AG) | Yes (free read replica) | High IOPS, low latency, in-memory Online Transaction Processing (OLTP) |
| **Hyperscale** | 1–5 named replicas | Yes (up to 30 replicas) | Large databases (100 TB+), fast backup/restore |

### Purchasing Model Comparison

| Criteria | DTU | vCore |
|---|---|---|
| License portability (Hybrid Benefit) | No | Yes — up to 55% savings |
| Fine-grained compute control | No | Yes |
| Serverless option | No | Yes |
| Transparent resource view | No (bundled) | Yes |
| **Exam advice** | Legacy/simple scenarios | All new deployments |

---

## 3. Azure SQL Database — Scalability

### Elastic Pools
- Pool of shared eDTU or vCore resources across multiple databases
- Databases draw from the pool — idle databases free resources for active ones
- **Choose when:** Multiple databases with unpredictable, variable workloads (SaaS tenancy pattern)
- **Avoid when:** Databases consistently run at high utilization simultaneously

### Read Scale-Out (Business Critical & Hyperscale)
- Route read-only queries to a secondary replica using `ApplicationIntent=ReadOnly` in connection string
- No extra cost (replica is always maintained for HA)
- **Choose when:** Heavy reporting queries that would impact primary

### Hyperscale
- Separates compute from storage using a distributed storage architecture
- Up to 100 TB database size
- Near-instant backup (snapshot-based) and fast restore (minutes not hours)
- Supports named replicas for read scale-out
- **Choose when:** Very large databases, need fast scaling, unpredictable storage growth

### Serverless Compute Tier
- Auto-pauses after a configurable inactivity period (minimum 1 hour)
- Auto-resumes on next connection (with a small cold-start delay)
- Billed per second of compute used (not per hour)
- **Choose when:** Development databases, intermittent workloads, variable demand with low average use

---

## 4. Azure SQL Managed Instance

### Key Differences vs SQL Database

| Feature | SQL Database | SQL Managed Instance |
|---|---|---|
| SQL Agent | No | Yes |
| Cross-database queries | Limited | Yes |
| Linked servers | No | Yes |
| CLR | No | Yes |
| Service Broker | No | Yes |
| SSRS/SSIS/SSAS | No | SSIS only |
| VNet deployment | Optional (PE) | Required (always in VNet) |
| Migration from on-premises | Re-architect needed | Near-zero changes |

### Service Tiers (SQL MI)
| Tier | Storage | IOPS | Use Case |
|---|---|---|---|
| **General Purpose** | Remote storage (Azure Premium) | 7,500 max | Most migration workloads |
| **Business Critical** | Local SSD | 40,000+ max | Highest IOPS, in-memory OLTP, built-in read replica |

### Deployment Options
- **Instance pool:** Pre-provision compute for multiple small MIs — faster provisioning, cost efficient
- Deployed inside a dedicated subnet in a VNet (requires at minimum /27 subnet)

---

## 5. Azure Database for PostgreSQL

### Deployment Options

| Option | Description | Use Case |
|---|---|---|
| **Flexible Server** | Current, recommended — full control, AZ support, stop/start | All new workloads |
| **Single Server** | Legacy — being retired | Avoid for new deployments |

### Key Features
- Compute tiers: Burstable (B-series), General Purpose, Memory Optimized
- High availability: Zone-redundant HA (standby in different AZ), same-zone HA
- Read replicas: Up to 5, cross-region supported
- pgBouncer built-in connection pooling
- **Microsoft Entra authentication** — passwordless, recommended
- Point-in-time restore: 7–35 days

---

## 6. Azure Database for MySQL

### Deployment Options
- **Flexible Server** only — Single Server retired in September 2024
- Zone-redundant HA, same-zone HA
- Read replicas: Up to 5
- Built-in connection pooling (ProxySQL via Azure portal)
- Point-in-time restore: 1–35 days
- **Choose when:** WordPress, Magento, LAMP stack, or any MySQL-dependent app

---

## 7. Data Protection

### Transparent Data Encryption (TDE)
- Encrypts data at rest (data files, log files, backups)
- Enabled by default on all Azure SQL services
- **Service-managed keys:** Default, Microsoft manages keys
- **Customer-managed keys (CMK):** Use Key Vault — required for compliance scenarios where you control keys

### Dynamic Data Masking (DDM)
- Obfuscates sensitive data in query results for non-privileged users
- No change to stored data — masking applied at query layer
- **Use case:** Developers/support see masked credit card numbers, SSNs

### Always Encrypted
- Client-side encryption — data encrypted in the application, never plaintext in the database engine
- Database administrator cannot read the data
- **Use case:** Extremely sensitive data where DBA access must be prevented (medical records, financial data)

### Row-Level Security (RLS)
- Restrict which rows a user can access using predicates
- Implemented as security policies in T-SQL
- **Use case:** Multi-tenant SaaS where each customer sees only their data

### Microsoft Defender for SQL
- Detects anomalous database activities (SQL injection, unusual access patterns)
- Vulnerability Assessment — scans for misconfigurations
- Available for SQL Database, SQL MI, SQL on VMs

---

## 8. High Availability (Built-in)

### General Purpose / Standard
- Uses Azure Premium Storage with 3 replicas
- Log-based HA — compute can fail over to another node, storage persists
- RTO: typically 20–30 seconds
- No read scale-out

### Business Critical
- Always On Availability Group — 3 synchronous replicas + 1 readable secondary
- In-memory data (local SSD), ultra-low latency
- RTO: ~5–10 seconds
- Built-in free read replica

### Hyperscale
- Page servers and log service are distributed independently
- Compute failover in ~60 seconds
- Storage is always available (not tied to compute)

---

## 9. Disaster Recovery

### Active Geo-Replication (SQL Database only)
- Up to 4 readable secondary replicas in different regions
- Manual failover (no automatic)
- Each secondary is a full read-only database
- **Use when:** Custom failover logic or multiple read replicas needed

### Auto-Failover Groups
- Layer on top of geo-replication
- Single read-write and read-only listener endpoint (DNS-based, no client reconfiguration)
- Automatic or manual failover
- Covers SQL Database (single or elastic pool) and SQL Managed Instance
- **Use when:** Transparent failover with no connection string changes

### Point-In-Time Restore (PITR)
- Every 5–12 minutes transaction log backup
- Restore to any point within the retention period
- Retention: 7 days (Basic), up to 35 days (Standard/Premium/vCore)

### Long-Term Retention (LTR)
- Weekly, monthly, or yearly backup copies stored in Azure Blob Storage (RA-GRS)
- Up to 10 years
- Required for regulatory compliance (HIPAA, SOC, etc.)

---

## 10. Exam Scenario Cheat Sheet

| Scenario | Answer |
|---|---|
| Lift-and-shift SQL Server with SQL Agent, linked servers | SQL Managed Instance |
| New SaaS app, 100 small databases with variable load | SQL Database + Elastic Pool |
| Very large database 50 TB+, fast backup needed | SQL Database Hyperscale |
| Highest IOPS SQL workload, in-memory OLTP | SQL Database or SQL MI Business Critical tier |
| Dev database needs to pause overnight to save cost | SQL Database vCore Serverless |
| Existing SQL Server licenses, reduce Azure cost | vCore model + Azure Hybrid Benefit |
| DBA must not see encrypted data | Always Encrypted |
| Support team should see masked credit card numbers | Dynamic Data Masking |
| Automatic failover, single connection string, multi-region | Auto-Failover Groups |
| Multiple read replicas, custom failover logic | Active Geo-Replication |
| Backup retained for 7 years for compliance | Long-Term Retention (LTR) |
| Open-source MySQL migration to Azure | Azure Database for MySQL Flexible Server |
| PostgreSQL app needs passwordless auth | Flexible Server + Microsoft Entra auth |

---

## 11. Key Limits to Know

| Resource | Limit |
|---|---|
| SQL Database max size (vCore GP) | 4 TB |
| SQL Database max size (Hyperscale) | 100 TB |
| SQL MI max size | 16 TB |
| Elastic pool max databases (vCore) | 500 |
| Active geo-replication max secondaries | 4 |
| Auto-failover group regions | 1 primary + 1 secondary |
| PITR max retention | 35 days |
| LTR max retention | 10 years |
| PostgreSQL read replicas | 5 |
| MySQL read replicas | 5 |
