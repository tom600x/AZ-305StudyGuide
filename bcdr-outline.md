# Business Continuity & Disaster Recovery — AZ-305 Study Outline

> **Status:** Outline — expand as needed
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
- **Azure Backup vault vs Recovery Services vault** — Recovery Services vault supports both backup and Site Recovery
- Workloads: Azure VMs, SQL on VMs, SAP HANA, Azure Files, on-premises (MARS agent)
- Backup policies: frequency, retention, cross-region restore
- Soft delete — 14-day protection against accidental deletion
- Cross-region restore — restore to paired region for compliance/DR

### 2. Azure Site Recovery (ASR)
- Replicates VMs (Azure-to-Azure, on-premises-to-Azure, VMware/Hyper-V)
- **RPO:** as low as 30 seconds (Azure-to-Azure)
- **RTO:** typically 1–2 hours depending on recovery plan
- Recovery plans — ordered failover groups, manual actions, Azure Automation runbooks
- Test failover — validate DR without impacting production

### 3. High Availability Patterns

| Pattern | RTO | RPO | Cost |
|---|---|---|---|
| **Availability Sets** | Minutes | Seconds | Low |
| **Availability Zones** | Seconds | Near-zero | Medium |
| **Active-Active (multi-region)** | Seconds | Near-zero | High |
| **Active-Passive (ASR failover)** | Minutes–Hours | Minutes | Medium |

### 4. Azure SQL BCDR
- **Geo-replication:** Up to 4 readable secondaries, manual failover
- **Auto-failover groups:** Automatic failover, custom DNS endpoint, group-level management
- **Point-in-time restore (PITR):** 7–35 days retention
- **Long-term retention (LTR):** Up to 10 years in Blob Storage

### 5. Cosmos DB BCDR
- Multi-region writes → near-zero RPO
- Automatic failover with priority list
- Single-region writes → manual failover

### 6. Storage BCDR
- GRS/GZRS → Microsoft-managed failover to secondary
- Customer-managed failover → manual, data loss possible (recovery point varies)
- RA-GRS/RA-GZRS → read from secondary while primary is degraded

---

## Exam Tips
- **Backup ≠ DR** — backup restores data; DR restores operations
- **Always identify RPO/RTO requirements first** before choosing a solution
- Auto-failover groups (SQL) provide seamless failover with a consistent endpoint
- Availability Zones ≠ regions — AZs protect against datacenter failure, not regional failure
- Site Recovery is the go-to answer for VM-level DR across regions
