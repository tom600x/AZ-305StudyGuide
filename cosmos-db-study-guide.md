# Azure Cosmos DB — Deep-Dive Study Guide for AZ-305

## Why Cosmos DB Matters on AZ-305
The exam tests your ability to:
- Choose the correct Cosmos DB API for the use case
- Select the right consistency level (and explain the trade-offs)
- Design partition keys for scale
- Choose between provisioned throughput, autoscale, and serverless
- Protect data and design for high availability / DR

---

## 1. Cosmos DB Overview

Azure Cosmos DB is a **fully managed, globally distributed, multi-model NoSQL database**. Key characteristics:
- **SLA guarantees:** 99.999% availability, <10ms reads at P99, <10ms writes at P99
- **Turnkey global distribution:** Add/remove regions without downtime or re-deployment
- **Multi-API support:** Multiple API surfaces on the same engine
- **Schema-agnostic:** No fixed schema, auto-indexed by default

---

## 2. Cosmos DB APIs — Choose the Right One

| API | Best For | Protocol |
|---|---|---|
| **NoSQL (Core)** | New cloud-native apps, JSON documents | Native Cosmos SDK, SQL-like query |
| **MongoDB** | Migrating existing MongoDB apps | MongoDB wire protocol |
| **Cassandra** | Migrating Apache Cassandra apps | Cassandra Query Language (CQL) |
| **Gremlin** | Graph data — social networks, recommendations, fraud detection | Gremlin (Apache TinkerPop) |
| **Table** | Migrating Azure Table Storage apps needing global distribution | OData/REST (Azure Table SDK compatible) |
| **PostgreSQL (Citus)** | Distributed PostgreSQL, analytical + transactional | PostgreSQL wire protocol |

### API Decision Guide
```
Existing MongoDB → API for MongoDB
Existing Cassandra → API for Cassandra
Existing Azure Table Storage → API for Table
Graph relationships (social, fraud, knowledge) → API for Gremlin
New app, maximum features, best Cosmos performance → API for NoSQL (Core)
Distributed relational + Cosmos scale → API for PostgreSQL
```

> **Exam tip:** You cannot change the API after account creation. The API for NoSQL gives access to all Cosmos DB-specific features (e.g., change feed, multi-region writes with custom conflict resolution).

---

## 3. Consistency Levels — Critical Exam Topic

Cosmos DB offers **5 consistency levels** in a spectrum from strongest to weakest:

| Level | Description | Read Throughput | Write Latency | Use Case |
|---|---|---|---|---|
| **Strong** | Linearizable reads — always see the latest committed write | Lowest (~50% vs eventual) | Highest (2× RTT for multi-region) | Financial transactions, inventory control |
| **Bounded Staleness** | Reads lag behind writes by at most K versions or T seconds | High | Low | Global reads where slight staleness is acceptable, strong consistency in region |
| **Session** | Within a session: read-your-own-writes, monotonic reads/writes | High | Low | **Most common** — user-specific data (shopping carts, user profiles) |
| **Consistent Prefix** | Reads never see out-of-order writes | High | Low | Social media feeds, event streams where order matters but not immediacy |
| **Eventual** | No ordering guarantees, converges eventually | Highest (2× vs bounded staleness) | Lowest | Non-critical aggregations, like counts, social likes |

### Key Facts for Exam
- **Default consistency:** Session (set at account level, can override per request)
- **Strong consistency** with multiple regions: write latency = 2 × RTT between farthest regions
- Strong consistency is **not available** for multi-region write accounts
- Relaxed consistency levels (Session, Consistent Prefix, Eventual) give ~2× read throughput vs Strong/Bounded Staleness
- You can **tighten** consistency per read request but never **relax** below the account default

---

## 4. Throughput Models — Provisioned vs Serverless vs Autoscale

### Provisioned Throughput (Manual)
- You set exact RU/s (Request Units per second) — billed hourly
- Can be set at **database level** (shared) or **container level** (dedicated)
- Minimum: 400 RU/s per container
- **Choose when:** Predictable, steady workloads

### Autoscale Throughput
- Set a **maximum** RU/s — scales from 10% to 100% of max automatically
- Only pays for RUs actually consumed (billed at highest hourly RU/s in each hour window)
- Minimum max: 1,000 RU/s (automatically scales down to 100 RU/s when idle)
- **Choose when:** Variable workloads with unpredictable spikes; want hands-off scaling

### Serverless
- No provisioned capacity — pay per RU consumed per request
- Not globally distributable (single region only)
- Best for dev/test, sporadic workloads
- **Choose when:** Small apps, prototype, very low and unpredictable traffic

### Throughput Comparison

| Model | Billing | Multi-region | Best For |
|---|---|---|---|
| Provisioned | Fixed hourly | Yes | High, predictable traffic |
| Autoscale | Per max RU/s scaled to | Yes | Variable traffic with spikes |
| Serverless | Per RU consumed | No (single region) | Dev/test, sporadic |

> **Request Unit (RU) cost factors:** Item size, item complexity (nested objects), index policy, query patterns. A 1 KB item read costs ~1 RU; a write costs ~5 RUs.

---

## 5. Partitioning

### Logical Partitions
- A group of items with the same **partition key value**
- Max size: **20 GB** per logical partition
- All items with the same partition key share a logical partition

### Physical Partitions
- Cosmos DB maps logical partitions to physical partitions automatically
- Each physical partition: up to 10,000 RU/s and 50 GB storage
- Partitions split automatically as data grows (no downtime)

### Choosing a Partition Key
A good partition key must:
1. Have **high cardinality** (many unique values)
2. Distribute reads and writes **evenly** (avoid hot partitions)
3. Be included in most of your queries (avoids cross-partition queries)

| Good Partition Keys | Bad Partition Keys |
|---|---|
| `userId`, `deviceId`, `customerId` | `status` (only a few values) |
| `orderId` for order history | `country` (skewed distribution) |
| Composite (tenantId + date) | Boolean fields |

### Cross-Partition Queries
- Queries without the partition key fan out to all partitions
- More expensive in RUs, slower
- Design your data model and partition key to minimize cross-partition queries

---

## 6. Global Distribution

### Adding Regions
- Can add/remove regions at any time with zero downtime
- All data replicated to all configured regions
- **RU cost:** `R × N` total RUs globally (R = provisioned per region, N = number of regions)

### Single-Region vs Multi-Region Writes

| | Single-Region Writes | Multi-Region Writes |
|---|---|---|
| Write latency | Lowest in write region only | Low in all regions |
| Read latency | Low in all regions | Low in all regions |
| Strong consistency | Supported | Not supported |
| Conflict resolution | Not needed | Required (LWW or custom) |
| Cost | Lower | Higher |

### Conflict Resolution (Multi-Region Writes)
- **Last-Write-Wins (LWW):** Uses timestamp or custom property — default
- **Custom (merge procedures):** Application-defined conflict resolution logic
- Conflicts are stored in the conflicts feed for custom resolution

### Service-Managed Failover
- Cosmos DB automatically fails over to the next region in priority list
- Enable via account settings
- **Manual failover** is also available for DR drills

---

## 7. Data Protection

### Encryption
- All data encrypted at rest (AES-256) and in transit (TLS)
- Customer-managed keys (CMK) supported via Azure Key Vault Managed HSM

### Backup
- **Continuous backup (default):** Restore to any point within 7–30 days (configurable up to 30 days)
- **Periodic backup (legacy):** Full backups every 1–24 hours, 2 copies retained for 8–1,168 hours
- Continuous backup supports self-service restore
- Backups stored in geo-redundant storage

### Network Security
- VNet service endpoints and Private Endpoints
- IP firewall rules
- Defender for Cosmos DB — threat detection, anomaly alerts

---

## 8. Cosmos DB Change Feed

- An ordered, append-only log of all changes (inserts and updates) to a container
- Enables event-driven patterns, real-time analytics, Change Data Capture (CDC)
- **Not supported:** Deletes (workaround: soft delete with TTL)
- Consumed via: Azure Functions trigger, Change Feed Processor SDK, Azure Stream Analytics
- **Use cases:** Real-time dashboards, downstream sync to data warehouse, event sourcing

---

## 9. Time to Live (TTL)

- Automatically delete items after a specified number of seconds
- Set at container level (default TTL) or per item (item-level TTL)
- TTL = -1 means items never expire
- **Use cases:** Session data, temporary caches, ephemeral event data

---

## 10. Cosmos DB vs Azure Table Storage — Comparison

| Feature | Cosmos DB (Table API) | Azure Table Storage |
|---|---|---|
| Global distribution | Yes, multi-region | No |
| Consistency levels | 5 options | Eventual only |
| Throughput SLA | Yes | No dedicated throughput |
| Latency SLA | <10ms at P99 | No SLA |
| Cost | Higher | Lower |
| Migration | Table API is wire-compatible | N/A |

> **Exam tip:** If the scenario has existing Azure Table Storage data and needs global distribution, low latency SLAs, or stronger consistency → migrate to Cosmos DB Table API.

---

## 11. High Availability

- **99.999% availability SLA** with multi-region writes configuration
- **99.99% availability SLA** with single-region writes + one secondary
- Zone redundancy available per region (within-region HA)
- Combination of multi-region + zone redundancy = maximum HA

---

## 12. Exam Scenario Cheat Sheet

| Scenario | Answer |
|---|---|
| Existing MongoDB app, move to Azure with minimal changes | Cosmos DB API for MongoDB |
| Social graph — connections, recommendations | Cosmos DB API for Gremlin |
| Shopping cart per user, read-your-own-writes | Cosmos DB Session consistency |
| Global app, need same data everywhere, no stale reads | Cosmos DB Strong consistency (single-region write) |
| Global app, highest availability, tolerate slight staleness | Cosmos DB Bounded Staleness |
| IoT events — order matters, not necessarily real-time | Consistent Prefix |
| Social likes counter, approximate aggregate OK | Eventual consistency |
| New app, variable traffic with spikes | Cosmos DB with Autoscale throughput |
| Dev/test, sporadic traffic, cost-sensitive | Cosmos DB Serverless |
| Multi-tenant SaaS, isolate tenant data | Partition key = `tenantId` |
| Event-driven architecture, react to Cosmos DB changes | Change Feed + Azure Functions |
| Azure Table Storage needs global distribution | Cosmos DB Table API |
| Data must be deleted automatically after 24 hours | TTL = 86400 |

---

## 13. Key Limits

| Resource | Limit |
|---|---|
| Logical partition size | 20 GB |
| Max RUs per physical partition | 10,000 RU/s |
| Provisioned minimum | 400 RU/s per container |
| Autoscale minimum max setting | 1,000 RU/s |
| Max regions (distribution) | No limit |
| PITR retention (continuous backup) | Up to 30 days |
| Max item size | 2 MB |

---

## 14. Cosmos DB Exam Traps

### 1. Choosing Strong consistency for every global workload
- **Trap:** Strong consistency sounds safest
- **Better default:** Use the weakest consistency that still meets the business requirement, often Session or Bounded Staleness

### 2. Picking a bad partition key because it matches a business identifier
- **Trap:** Tenant or user IDs look natural without checking cardinality and access patterns
- **Better default:** Choose a high-cardinality key that spreads RU usage and aligns with common queries

### 3. Choosing serverless for sustained high-throughput production workloads
- **Trap:** Serverless looks cheapest
- **Better default:** Autoscale or provisioned throughput for predictable or heavy sustained workloads

### 4. Treating multi-region reads as near-zero data loss protection
- **Trap:** Multi-region sounds resilient enough
- **Better default:** Near-zero data loss across regions often points to multi-region writes and the right consistency model

### 5. Using Cosmos DB when relational requirements dominate
- **Trap:** Cosmos DB is globally distributed and highly available
- **Better default:** Re-check whether the scenario actually needs relational joins, transactions, or SQL Server compatibility

### Rapid Elimination Rules

| Requirement | Eliminate First |
|---|---|
| Existing MongoDB app with minimal changes | API for NoSQL answers |
| Global scale with flexible consistency | SQL-first answers |
| Sustained high RU demand | Serverless answers |
| Need evenly distributed throughput | Low-cardinality partition keys |
| Automatic item expiry | Designs without TTL |
