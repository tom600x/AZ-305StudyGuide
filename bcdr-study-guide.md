# Business Continuity & Disaster Recovery — AZ-305 Study Guide

> **Exam Weight:** ~15%

---

## Key Concepts: RTO and RPO

| Term | Definition | Impact |
|---|---|---|
| **RPO** (Recovery Point Objective) | Max acceptable data loss (time) | Determines backup frequency |
| **RTO** (Recovery Time Objective) | Max acceptable downtime | Determines recovery mechanism |

**Exam tip:** Low RPO → more frequent backups / replication. Low RTO → hot standby / active-active.

---

## Key Topics to Study

### 1. Azure Backup

#### Vault Types

| Vault | Supports | Key Difference |
|---|---|---|
| **Recovery Services Vault (RSV)** | Backup + Site Recovery (ASR) | Original vault type — required for ASR |
| **Backup Vault** | Newer datasources (Disks, Blobs, PostgreSQL) | Newer, lighter vault — does NOT support ASR |

> **Exam tip:** If the question mentions Azure Site Recovery (ASR), the answer requires a **Recovery Services Vault**, not a Backup Vault.

#### Supported Workloads

| Workload | Vault | Agent / Method |
|---|---|---|
| Azure VMs | RSV or Backup Vault | Azure Backup extension (agentless) |
| SQL Server on Azure VMs | RSV | SQL workload extension |
| SAP HANA on Azure VMs | RSV | HANA backup extension |
| Azure Files | RSV | Snapshot-based |
| On-premises Windows files/folders | RSV | MARS (Microsoft Azure Recovery Services) agent |
| On-premises VMs (VMware/Hyper-V) | RSV | MABS or DPM |
| Azure Blobs | Backup Vault | Operational backup (soft delete layer) |
| Azure Disks | Backup Vault | Snapshot-based |
| Azure Database for PostgreSQL | Backup Vault | Long-term retention backups |

#### Key Features

| Feature | Detail |
|---|---|
| **Soft delete** | Backed-up data retained 14 days after deletion — protects against accidental/ransomware deletion |
| **Cross-region restore** | Restore VM backup to paired region for DR or compliance testing |
| **Backup policies** | Define frequency (daily/weekly) and retention (daily/weekly/monthly/yearly) |
| **Immutable vault** | Prevents modification or deletion of backup data — protects against insider threats |
| **Multi-user authorization (MUA)** | Requires secondary approver for critical backup operations |

#### Backup Tiers (Azure VM Backup)

| Tier | Where Stored | Retention | Access Speed |
|---|---|---|---|
| **Snapshot tier** | VM disk snapshot (local) | Per policy | Instant restore |
| **Vault-standard tier** | Recovery Services Vault | Per policy (up to 99 years) | Standard |
| **Archive tier** | Cold storage | Long-term retention (LTR) | Hours to restore |

### 2. Azure Site Recovery (ASR)

ASR replicates workloads continuously to a secondary location so failover can be completed in minutes.

#### Replication Scenarios

| Source | Target | Agent Required |
|---|---|---|
| **Azure VM** (Region A) | Azure VM (Region B) | No — ASR mobility extension installed automatically |
| **VMware VM** (on-prem) | Azure VM | Mobility service agent + Configuration/Process server |
| **Hyper-V VM** (on-prem) | Azure VM | Azure Site Recovery Provider on Hyper-V host |
| **Physical server** (Windows/Linux) | Azure VM | Mobility service agent |

#### Key Metrics
- **RPO:** As low as 30 seconds (Azure-to-Azure replication)
- **RTO:** Typically 1–2 hours depending on recovery plan complexity

#### Recovery Plans
- Define an **ordered sequence of failover groups** — fail over in waves
- Add manual actions (e.g., wait for DNS update) or **Azure Automation runbooks** between steps
- **Test failover:** Validate DR plan in an isolated VNet without impacting production replication
- **Planned failover:** Zero data loss — synchronizes before cutting over (e.g., maintenance, migration)
- **Unplanned failover:** Immediate — minimal data loss based on last recovery point

#### Replication Policy
- Retention of recovery points: default 24 hours (up to 15 days for Azure-to-Azure)
- App-consistent snapshot frequency: captures in-memory state (VSS on Windows)
- Crash-consistent vs app-consistent recovery points — exam may test the difference

### 3. High Availability Patterns

| Pattern | RTO | RPO | Cost |
|---|---|---|---|
| **Availability Sets** | Minutes | Seconds | Low |
| **Availability Zones** | Seconds | Near-zero | Medium |
| **Active-Active (multi-region)** | Seconds | Near-zero | High |
| **Active-Passive (ASR failover)** | Minutes–Hours | Minutes | Medium |

### 4. Azure SQL BCDR

| Feature | RPO | RTO | Notes |
|---|---|---|---|
| **Active Geo-Replication** | ~5 seconds | Manual failover | Up to 4 readable secondaries; manual failover only |
| **Auto-Failover Groups** | ~5 seconds | Automatic (minutes) | Group-level endpoint; automatic or manual failover; preferred over geo-replication for HA |
| **Point-in-time restore (PITR)** | Varies | Minutes–hours | Restores to any point within 7–35 days; new database created |
| **Long-term retention (LTR)** | N/A | Minutes–hours | Weekly/monthly/yearly backups in Blob Storage, up to 10 years |
| **Geo-restore** | Up to 1 hour | Minutes–hours | Restore from geo-redundant backup to any region |

> **Exam tip:** **Auto-failover groups** are preferred over active geo-replication when you need automatic failover and a consistent connection string (listener endpoint). Geo-replication requires manually updating connection strings after failover.

> **Exam tip:** PITR creates a **new database** — it does not overwrite the original. Restoration time depends on database size.

### 5. Cosmos DB BCDR

| Configuration | RPO | RTO | Notes |
|---|---|---|---|
| **Single region** | Up to hours | Manual restore | No replication; vulnerable to regional outage |
| **Multi-region reads (single-write)** | ~15 min (manual failover) | ~15 min | Microsoft-managed automatic failover by priority list |
| **Multi-region writes** | Near-zero | Near-zero | Writes succeed in any region; conflict resolution required |

- **Automatic failover:** Cosmos DB triggers failover to highest-priority secondary if primary goes down
- **Manual failover:** Available for planned maintenance — you control the timing
- **Priority list:** Set the failover priority order across your replica regions
- **Consistency levels affect RPO:** Strong consistency = higher latency; Eventual = lower latency but more potential data divergence

> **Exam tip:** **Multi-region writes** (also called multi-master) achieves near-zero RPO and RTO but requires a conflict resolution policy. If the scenario says "write availability even during regional failure" — choose multi-region writes.

### 6. Storage BCDR

#### Redundancy Options for DR

| Redundancy | Copies | Locations | Failover |
|---|---|---|---|
| **LRS** (Locally Redundant) | 3 | Same datacenter | None — single point of failure |
| **ZRS** (Zone Redundant) | 3 | 3 availability zones | None — survives datacenter failure within region |
| **GRS** (Geo-Redundant) | 6 | Primary + paired secondary region | Microsoft-managed failover |
| **GZRS** (Geo-Zone Redundant) | 6 | 3 zones primary + secondary region | Microsoft-managed failover |
| **RA-GRS** | 6 | Primary + secondary (read-only) | Read from secondary during primary degradation |
| **RA-GZRS** | 6 | 3 zones primary + read secondary | Combines ZRS resilience with geo read access |

#### Failover Behavior
- **Microsoft-managed failover:** Microsoft triggers only in large-scale disasters with estimated recovery > 1 hour
- **Customer-managed failover (preview):** You trigger manually — may result in data loss (recovery point lag)
- After failover, the storage account becomes LRS in the new region; re-enable geo-replication manually

> **Exam tip:** **RA-GRS / RA-GZRS** provides read access to the secondary endpoint at all times, not just during failover — useful when the application can route read traffic to the secondary to reduce latency or tolerate primary degradation.

> **Exam tip:** **ZRS** is recommended over LRS for most production workloads — zone-redundant at same price point as LRS in supported regions.

### 7. Azure Backup Center

#### What It Is
Azure Backup Center provides a **unified management console** for all Azure Backup and Azure Site Recovery operations across subscriptions, resource groups, vaults, and regions — without navigating individual Recovery Services vaults.

#### Key Capabilities

| Capability | Description |
|---|---|
| **Unified inventory** | View all protected items, vaults, and policies across subscriptions in one place |
| **Governance** | Monitor backup compliance — identify unprotected workloads, policy non-compliance |
| **Monitoring & Alerts** | Aggregated backup jobs, alerts, and anomalies across all vaults |
| **Backup reports** | Azure Monitor Workbooks-based reports on storage, jobs, and trends |
| **At-scale operations** | Trigger backup/restore, update policies across multiple items simultaneously |
| **Datasource-centric view** | View backup status from the resource's perspective, not the vault's |

#### Supported Datasources
Azure VMs, SQL on Azure VMs, SAP HANA on Azure VMs, Azure Files, Azure Blobs, Azure Disks, Azure Database for PostgreSQL

#### Backup Center vs Recovery Services Vault

| | Recovery Services Vault | Backup Center |
|---|---|---|
| **Scope** | Single vault | Cross-vault, cross-subscription |
| **Use for** | Configure backup policies, perform restores | Monitor compliance, governance at scale |
| **Cost** | No additional cost (vault resource) | No additional cost (management layer) |

> **Exam tip:** When a scenario asks for **centralized backup monitoring and governance across multiple subscriptions or vaults** — the answer is **Azure Backup Center**, not individual Recovery Services vaults.

---

## DR Strategy Patterns

| Pattern | Description | RTO | RPO | Cost | When to Use |
|---|---|---|---|---|---|
| **Backup & Restore** | Restore from backup after failure | Hours | Hours | Low | Non-critical, cost-sensitive workloads |
| **Pilot Light** | Core infrastructure running in DR region, minimal resources | Minutes–Hours | Minutes | Low-Medium | Systems that can't afford full standby |
| **Warm Standby** | Scaled-down but running replica in DR region | Minutes | Seconds–Minutes | Medium | Important workloads with moderate RTO requirements |
| **Active-Active** | Full deployment running in multiple regions simultaneously | Seconds | Near-zero | High | Mission-critical workloads, zero downtime requirement |
| **Active-Passive (ASR)** | Secondary is powered off; ASR replicates continuously | Minutes–Hours | Minutes | Medium | Disaster recovery for VMs without always-on cost |

### Mapping Patterns to Azure Services

| Pattern | Typical Azure Implementation |
|---|---|
| **Backup & Restore** | Azure Backup (RSV) + geo-restore |
| **Pilot Light** | ASR replication, manual failover; minimal secondary infra |
| **Warm Standby** | ASR + scaled-down secondary App Service / VMs in DR region |
| **Active-Active** | Traffic Manager / Front Door routing across two full deployments |
| **Active-Passive** | ASR with recovery plans; Azure SQL auto-failover groups |

---

## Exam Scenario Cheat Sheet

| Scenario | Answer | Key Detail |
|---|---|---|
| VM goes down in one region, need to recover in DR region | Azure Site Recovery | Continuous replication, ordered failover plans |
| Need to test DR without impacting production | ASR test failover | Isolated VNet, no production impact |
| SQL database needs automatic failover with same connection string | Auto-failover groups | Listener endpoint survives failover |
| Need to read SQL data from secondary for reporting | Active geo-replication | Readable secondaries |
| Restore SQL to a specific point 10 days ago | PITR (Point-in-Time Restore) | Up to 35 days; creates new DB |
| Keep SQL backups for 5 years for compliance | Long-term retention (LTR) | Up to 10 years in Blob Storage |
| Cosmos DB needs near-zero data loss on regional failure | Multi-region writes | Writes accepted in any region |
| Cosmos DB needs automatic failover, single write region | Multi-region reads + automatic failover | Priority list-driven failover |
| Storage account secondary needs to be readable at all times | RA-GRS or RA-GZRS | Read access to secondary endpoint always |
| Monitor backup compliance across all subscriptions | Azure Backup Center | Cross-subscription governance |
| VM backup deleted accidentally, need to recover | Soft delete | 14-day retention after deletion |
| Protect backup data from ransomware / insider deletion | Immutable vault + MUA | Cannot be modified or deleted |
| Highly available VMs in same datacenter | Availability Sets | 99.95% SLA, same datacenter |
| Highly available VMs across datacenters in one region | Availability Zones | 99.99% SLA, separate physical zones |
| Entire region fails, need automatic SQL failover | Auto-failover groups | Automatic cross-region failover |

---

## Exam Tips
- **Backup ≠ DR** — backup restores data; DR restores operations
- **Always identify RPO/RTO requirements first** before choosing a solution
- Auto-failover groups (SQL) provide seamless failover with a consistent endpoint
- Availability Zones ≠ regions — AZs protect against datacenter failure, not regional failure
- Site Recovery is the go-to answer for VM-level DR across regions
- **Centralized backup monitoring across subscriptions** = Azure Backup Center

---

## Gap-Closing Review Priorities

The most common AZ-305 mistakes in this domain come from mixing up backup, high availability, and disaster recovery.

### High-Yield Decision Rules

| Requirement | Preferred Direction | Common Mistake |
|---|---|---|
| Low downtime in one region | High availability design | Choosing backup when the app must stay online |
| Regional outage protection | Disaster recovery design | Choosing Availability Zones, which do not protect against region failure |
| Recover deleted or corrupted data | Backup and restore | Choosing geo-replication alone |
| Same app endpoint after SQL failover | Auto-failover groups | Choosing active geo-replication without considering connection string handling |
| Test DR safely | ASR test failover or service-specific test process | Skipping validation entirely |

### Recommendations

1. Start every BCDR question by identifying the required **RPO** and **RTO**.
2. Separate data protection from application continuity. The exam often expects both.
3. Review zone redundancy and geo-redundancy together so you do not confuse datacenter failure with regional failure.
4. Practice answering what happens to the application endpoint, not just the data.

### Practice Prompts

1. A database must survive accidental deletion and support point-in-time restore. Why is backup required even if geo-replication exists?
2. A two-region app must fail over quickly with minimal operational steps. Which parts need active-active, and which can remain active-passive?
3. A workload requires near-zero data loss across regions. Why might multi-region writes be required for Cosmos DB instead of single-write-region failover?

---

## BCDR Exam Traps

### 1. Treating backup as disaster recovery
- **Trap:** Backup protects data, so it feels like it solves outages too
- **Better default:** Backup restores data; DR restores service availability and operations

### 2. Choosing Availability Zones for regional disaster protection
- **Trap:** Zones improve availability, so they seem like enough for every failure case
- **Better default:** Zones protect against datacenter failure inside one region, not full regional outages

### 3. Choosing geo-replication when point-in-time restore is required
- **Trap:** Geo-replication sounds like complete protection
- **Better default:** Restore scenarios still require backup/PITR or LTR depending on retention needs

### 4. Ignoring application endpoint behavior after failover
- **Trap:** The data layer survives, so the architecture appears complete
- **Better default:** Check whether the app also needs listener endpoints, DNS failover, or global traffic routing

### 5. Skipping DR testing in the design
- **Trap:** Replication looks sufficient on paper
- **Better default:** AZ-305 often expects ASR test failover or another safe validation path

### Rapid Elimination Rules

| Requirement | Eliminate First |
|---|---|
| Recover deleted data or corruption | Geo-replication-only answers |
| Regional outage protection | Availability Zones-only answers |
| Same SQL connection string after failover | Active geo-replication-only answers |
| VM-level cross-region recovery | Backup-only answers |
| Need to validate DR safely | Designs with no test failover path |
