# AZ-305: Designing Microsoft Azure Infrastructure Solutions — Study Guide Index

## Exam Overview

| | |
|---|---|
| **Exam** | AZ-305 |
| **Role** | Azure Solutions Architect Expert |
| **Prerequisite** | AZ-104 (recommended) |
| **Format** | 40–60 questions — case studies, single/multi-select, drag-and-drop |
| **Passing Score** | 700 / 1000 |
| **Duration** | 120 minutes |
| **Renewal** | Every year (free online assessment) |

---

## Exam Domain Weights (2024–2026)

| Domain | Weight | Key Services Tested |
|---|---|---|
| Design infrastructure solutions | ~35% | Networking, VMs, AKS, App Service, APIM, Migrations |
| Design identity, governance & monitoring solutions | ~25% | Entra ID, RBAC, Policy, Key Vault, Monitor, Sentinel |
| Design data storage solutions | ~25% | Storage, SQL, Cosmos DB, ADF, Synapse, Data Lake |
| Design business continuity solutions | ~15% | Backup, ASR, HA patterns, geo-replication, RTO/RPO |

---

## Study Guides

### Infrastructure Solutions (~35%)
- [Azure Networking — VNets, NSG, Firewall, VPN, ExpressRoute, Load Balancing, DNS, Private Endpoints](./networking-study-guide.md) ⭐ **Start Here**
- [Compute Solutions — VMs, VMSS, App Service, AKS, ACI, Functions, Azure Arc](./compute-study-guide.md)
- [Application Architecture — APIM, Redis, App Config, CI/CD, Containers, Logic Apps](./application-architecture-study-guide.md)
- [Migrations — Azure Migrate, DMS, CAF, Data Box, Data Transfer Options](./migrations-study-guide.md)

### Data Storage Solutions (~25%)
- [Azure Storage — Blob, Files, Queues, Tables, SKUs, Tiers, Replication](./storage-study-guide.md) ⭐ **Start Here**
- [Relational Data — Azure SQL, SQL Managed Instance, PostgreSQL, MySQL](./relational-data-study-guide.md)
- [Azure Cosmos DB — APIs, Consistency, Partitioning, Throughput, BCDR](./cosmos-db-study-guide.md)
- [Data Integration — ADF, Synapse Analytics, Databricks, Stream Analytics](./data-integration-study-guide.md)
- [Azure Messaging — Event Hubs, Service Bus, Queue Storage, Event Grid](./messaging-study-guide.md)

### Identity, Governance & Monitoring (~25%)
- [Identity, Governance & Monitoring — Entra ID, RBAC, Policy, Key Vault, Monitor, Sentinel](./identity-governance-study-guide.md) ⭐ **Start Here**
- [Monitoring & Management — Azure Monitor, Log Analytics, Alerts, App Insights, Cost Management, SLA Tiers](./monitoring-study-guide.md)

### Business Continuity Solutions (~15%)
- [Business Continuity & DR — Backup, ASR, HA Patterns, SQL/Cosmos/Storage BCDR, Backup Center](./bcdr-study-guide.md)

---

## Recommended Study Order

```
Week 1 — Networking & Storage (highest exam density)
  → networking-study-guide.md
  → storage-study-guide.md

Week 2 — Identity & Compute
  → identity-governance-study-guide.md
  → compute-study-guide.md

Week 3 — Data & Messaging
  → relational-data-study-guide.md
  → cosmos-db-study-guide.md
  → messaging-study-guide.md

Week 4 — Architecture, Integration & BCDR
  → application-architecture-study-guide.md
  → data-integration-study-guide.md
  → bcdr-study-guide.md

Week 5 — Fill Gaps
  → monitoring-study-guide.md
  → migrations-study-guide.md
  → 2–3 full practice exams
```

---

## Topic Coverage Matrix

| Topic | File |
|---|---|
| VNets, NSGs, Firewall, VPN, ExpressRoute | [networking-study-guide.md](./networking-study-guide.md) |
| Load Balancer, App Gateway, Front Door, Traffic Manager | [networking-study-guide.md](./networking-study-guide.md) |
| Private Endpoints, Service Endpoints, Bastion | [networking-study-guide.md](./networking-study-guide.md) |
| Virtual WAN, DDoS, Routing | [networking-study-guide.md](./networking-study-guide.md) |
| VMs, VMSS, Spot, Reserved, Dedicated Host | [compute-study-guide.md](./compute-study-guide.md) |
| App Service, ASE, Deployment Slots | [compute-study-guide.md](./compute-study-guide.md) |
| AKS, ACI, Azure Functions | [compute-study-guide.md](./compute-study-guide.md) |
| Azure Arc (multi-cloud/on-prem management) | [compute-study-guide.md](./compute-study-guide.md) |
| APIM tiers, policies, self-hosted gateway | [application-architecture-study-guide.md](./application-architecture-study-guide.md) |
| Azure Cache for Redis, tiers, persistence | [application-architecture-study-guide.md](./application-architecture-study-guide.md) |
| App Configuration, Key Vault (config use) | [application-architecture-study-guide.md](./application-architecture-study-guide.md) |
| CI/CD, IaC (Bicep, Terraform), deployment strategies | [application-architecture-study-guide.md](./application-architecture-study-guide.md) |
| Container Apps, AKS vs Container Apps | [application-architecture-study-guide.md](./application-architecture-study-guide.md) |
| Azure Logic Apps, connectors, B2B/EDI | [application-architecture-study-guide.md](./application-architecture-study-guide.md) |
| Azure Migrate, DMS, CAF, Data Box | [migrations-study-guide.md](./migrations-study-guide.md) |
| Blob, Files, Queues, Tables | [storage-study-guide.md](./storage-study-guide.md) |
| Storage tiers (Hot/Cool/Cold/Archive) | [storage-study-guide.md](./storage-study-guide.md) |
| Storage replication (LRS/ZRS/GRS/RA-GRS) | [storage-study-guide.md](./storage-study-guide.md) |
| Azure SQL Database, elastic pools, DTU vs vCore | [relational-data-study-guide.md](./relational-data-study-guide.md) |
| SQL Managed Instance, SQL Server on VM | [relational-data-study-guide.md](./relational-data-study-guide.md) |
| PostgreSQL Flexible Server, MySQL | [relational-data-study-guide.md](./relational-data-study-guide.md) |
| Cosmos DB APIs, consistency levels | [cosmos-db-study-guide.md](./cosmos-db-study-guide.md) |
| Cosmos DB partitioning, RU/s, autoscale | [cosmos-db-study-guide.md](./cosmos-db-study-guide.md) |
| ADF pipelines, Synapse, Databricks | [data-integration-study-guide.md](./data-integration-study-guide.md) |
| Stream Analytics, real-time pipelines | [data-integration-study-guide.md](./data-integration-study-guide.md) |
| Event Hubs, Service Bus, Event Grid | [messaging-study-guide.md](./messaging-study-guide.md) |
| Queue Storage vs Service Bus | [messaging-study-guide.md](./messaging-study-guide.md) |
| Entra ID, Conditional Access, MFA, PIM | [identity-governance-study-guide.md](./identity-governance-study-guide.md) |
| RBAC, Azure Policy, Blueprints, Management Groups | [identity-governance-study-guide.md](./identity-governance-study-guide.md) |
| Key Vault (secrets, keys, certs, HSM) | [identity-governance-study-guide.md](./identity-governance-study-guide.md) |
| Microsoft Sentinel (SIEM/SOAR) | [identity-governance-study-guide.md](./identity-governance-study-guide.md) |
| Azure Monitor, Log Analytics, KQL | [monitoring-study-guide.md](./monitoring-study-guide.md) |
| Alerts, Action Groups, Application Insights | [monitoring-study-guide.md](./monitoring-study-guide.md) |
| Cost Management, Advisor, Service Health | [monitoring-study-guide.md](./monitoring-study-guide.md) |
| SLA tier edge cases across all services | [monitoring-study-guide.md](./monitoring-study-guide.md) |
| Azure Backup, Recovery Services vault | [bcdr-study-guide.md](./bcdr-study-guide.md) |
| Azure Site Recovery (ASR), RTO/RPO | [bcdr-study-guide.md](./bcdr-study-guide.md) |
| HA patterns (Availability Sets, Zones, Active-Active) | [bcdr-study-guide.md](./bcdr-study-guide.md) |
| SQL BCDR (geo-replication, auto-failover groups) | [bcdr-study-guide.md](./bcdr-study-guide.md) |
| Cosmos DB, Storage, BCDR options | [bcdr-study-guide.md](./bcdr-study-guide.md) |
| Azure Backup Center (multi-vault governance) | [bcdr-study-guide.md](./bcdr-study-guide.md) |

---

## Quick Tips
- Focus on **why** to choose a service, not just **what** it does.
- Every case study has a set of requirements — map them to services.
- Know the difference between **HA**, **DR**, and **Backup** — the exam tests all three.
- Understand **cost trade-offs** — cheaper options always have a catch.

---

## Probability of Passing on the First Attempt

| Study Approach | Estimated Pass Probability |
|---|---|
| These guides alone, no hands-on experience | ~30–40% |
| These guides + 2–3 full practice exams | ~55–65% |
| These guides + practice exams + some hands-on Azure experience | ~70–80% |
| The above + 1–2 years designing Azure solutions professionally | ~85%+ |

**What these guides cover well:**
- Service comparison tables and "choose when" decisions — the core of AZ-305 scenario questions
- Tier/SKU trade-offs, limits, and SLA edge cases
- Decision trees mapping requirements to services

**What these guides cannot replace:**
- **Practice exams** — Required. The exam uses long scenario questions; recognizing the pattern takes repetition. Use [Microsoft's free practice assessment](https://learn.microsoft.com/en-us/credentials/certifications/azure-solutions-architect/) or MeasureUp.
- **Hands-on experience** — AZ-305 tests *design judgment*, not memorization. Time in the Azure portal building solutions builds the intuition the exam probes.
- **The official study guide** — Cross-reference with the [Microsoft Learn AZ-305 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-305) to confirm full topic coverage.

> Microsoft's publicly reported first-attempt pass rate for AZ-305 candidates who self-study with mixed resources is approximately **60–65%**.
