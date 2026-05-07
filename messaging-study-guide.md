# Azure Messaging & Integration — Deep-Dive Study Guide for AZ-305

## Why Messaging Matters on AZ-305
The exam regularly tests your ability to pick the right messaging service based on:
- Message size, ordering, and delivery guarantees
- Throughput requirements (events/sec vs messages/sec)
- Pull vs push patterns
- Decoupling vs streaming vs broadcast scenarios

---

## 1. Quick Service Selection

```
High-volume event streaming / telemetry / log ingestion?
  └── Azure Event Hubs

Enterprise messaging, guaranteed delivery, ordering, dead-lettering?
  └── Azure Service Bus (Queues or Topics)

Simple async decoupling, large backlog, basic queue?
  └── Azure Queue Storage

Real-time event routing / reactive architecture?
  └── Azure Event Grid
```

---

## 2. Azure Event Hubs

### What It Is
- **Big data streaming platform** and event ingestion service
- Designed for **millions of events per second** with low latency
- Think: Kafka-compatible log pipeline, not a traditional message queue
- Consumers read from a persistent **event log** (not destructive dequeue)

### Key Concepts

| Concept | Description |
|---|---|
| **Namespace** | Container for Event Hubs (like a server) |
| **Event Hub** | A single stream/topic within a namespace |
| **Partition** | Ordered lane within an Event Hub — messages in one partition are ordered |
| **Consumer Group** | Independent view of the stream — multiple apps read the same data independently |
| **Offset** | Position in a partition — consumers track their own offset |
| **Retention** | 1–90 days (Standard: up to 7 days; Premium/Dedicated: up to 90 days) |

### Throughput Units (TU) / Processing Units (PU)

| SKU | Scaling Unit | Ingress | Egress | Features |
|---|---|---|---|---|
| **Basic** | Throughput Units (manual) | 1 MB/s or 1k events/s per TU | 2 MB/s per TU | 1 consumer group, 1-day retention |
| **Standard** | Throughput Units (manual/auto-inflate) | 1 MB/s per TU | 2 MB/s per TU | 20 consumer groups, 7-day retention, Capture |
| **Premium** | Processing Units (auto-scale) | Higher, predictable | Higher | Schema Registry, 90-day retention, isolated compute |
| **Dedicated** | Capacity Units (dedicated cluster) | Highest | Highest | Single-tenant, 90-day retention, highest compliance |

### Event Hubs Capture
- Automatically writes event stream to **Azure Blob Storage or Data Lake Storage Gen2**
- Format: Apache Avro
- **Choose when:** You need both real-time processing and durable long-term storage/replay

### Event Hubs for Kafka
- Drop-in Kafka endpoint (protocol compatible)
- Migrate Kafka apps to Azure without code changes
- Available on Standard, Premium, Dedicated

### Partitions and Ordering
- Events in the **same partition** are strictly ordered
- Events across partitions are **not ordered**
- Partition count is fixed at creation (cannot change on Standard; can increase on Premium/Dedicated)
- **Exam tip:** If you need order for a subset of events, use a partition key to route related events to the same partition

---

## 3. Azure Service Bus

### What It Is
- **Enterprise message broker** — reliable, ordered, transactional messaging
- FIFO guarantees, duplicate detection, dead-letter queue, sessions
- Used for decoupling business-critical application components

### Service Bus vs Event Hubs

| Feature | Service Bus | Event Hubs |
|---|---|---|
| Pattern | Pull (consumers poll) | Pull (consumers poll offset) |
| Message model | Destructive dequeue (lock → process → complete) | Non-destructive log (offset tracking) |
| Ordering | Per-session FIFO | Per-partition ordering |
| Dead-lettering | Yes | No |
| Max message size | 256 KB (Standard), 100 MB (Premium) | 1 MB (Standard), 1 MB–20 MB (Premium) |
| Retention | Until consumed (up to 14 days) | Time-based (up to 90 days) |
| Throughput | Thousands/sec | Millions/sec |
| **Use for** | Orders, workflows, commands | Telemetry, logs, event streams |

### Queues vs Topics/Subscriptions

| | Queue | Topic + Subscriptions |
|---|---|---|
| Consumers | One consumer per message (competing consumers) | Multiple consumers each get a copy |
| Pattern | Point-to-point | Publish-subscribe (fan-out) |
| **Choose when** | Task distribution, work items | Event notification to multiple systems |

### Key Features

| Feature | Description |
|---|---|
| **Dead-letter queue (DLQ)** | Messages that can't be processed are moved here for inspection |
| **Message sessions** | Group related messages, ensure ordered processing per session (e.g., per order ID) |
| **Duplicate detection** | Discards re-sent messages within a detection window |
| **Message lock** | Consumer locks a message while processing; expires if not completed (another consumer picks it up) |
| **Scheduled messages** | Deliver a message at a future time |
| **Transactions** | Atomic operations across multiple messages |
| **Auto-forward** | Chain queues/topics for complex routing |

### SKUs

| SKU | Max Message Size | Namespaces | Features |
|---|---|---|---|
| **Basic** | 256 KB | Shared | Queues only, no topics |
| **Standard** | 256 KB | Shared | Queues + Topics, basic features |
| **Premium** | 100 MB | Dedicated (isolated) | All features, VNet integration, Private Endpoint, zone redundancy |

**Exam tip:** **Premium** is required for Private Endpoints, VNet integration, and large messages.

---

## 4. Azure Queue Storage

### What It Is
- Simple, cheap HTTP-based queue built into a Storage Account
- Up to **64 KB per message**, up to **500 TB** total queue size
- Messages visible after a configurable **visibility timeout** (default 30s)
- Messages expire after up to **7 days**

### Queue Storage vs Service Bus Queues

| Feature | Queue Storage | Service Bus Queue |
|---|---|---|
| Max message size | 64 KB | 256 KB (Standard), 100 MB (Premium) |
| Max queue size | 500 TB | 1–80 GB |
| Ordering | Best-effort (FIFO not guaranteed) | FIFO with sessions |
| Dead-letter | No | Yes |
| Duplicate detection | No | Yes |
| Transactions | No | Yes |
| Auth | Storage Account Key, Shared Access Signature (SAS), Role-Based Access Control (RBAC) | SAS, RBAC, Managed Identity |
| Cost | Very low | Higher |
| **Choose when** | Simple decoupling, massive backlog, audit logs | Reliable delivery, ordering, enterprise integration |

---

## 5. Azure Event Grid

### What It Is
- **Reactive event routing** — publishes discrete events to subscribers
- Near-real-time, push-based (HTTP webhooks, Azure Functions, Service Bus, Event Hubs, Queue Storage)
- Built-in integration with many Azure services as **event sources** (Blob Storage, Resource Groups, Subscriptions, etc.)

### Event Grid vs Event Hubs vs Service Bus

| | Event Grid | Event Hubs | Service Bus |
|---|---|---|---|
| Pattern | Push (reactive) | Pull (streaming) | Pull (queue/topic) |
| Volume | Millions of events/sec | Millions of events/sec | Thousands/sec |
| Message size | 1 MB | 1 MB+ | 256 KB–100 MB |
| Retention | 24 hours (retry) | Days–90 days | Until consumed |
| Ordering | No guarantee | Per partition | Per session |
| **Use for** | React to Azure resource events, serverless triggers | Stream telemetry, log data | Reliable commands, workflow steps |

### Common Event Grid Sources
- Blob Storage → trigger on blob create/delete
- Azure Container Registry → image push/delete
- Resource Groups → resource created/deleted
- Azure Subscriptions → policy violations, security alerts
- Custom topics → your own application events

---

## 6. Exam Scenario Cheat Sheet

| Scenario | Answer |
|---|---|
| Ingest 1M IoT sensor events/sec, replay later | Event Hubs (Standard/Premium) + Capture |
| Process orders one at a time, guaranteed FIFO per customer | Service Bus Queue with **Sessions** |
| Send notifications to 5 different systems on each new order | Service Bus **Topic** with 5 subscriptions |
| Messages must not be lost if consumer crashes | Service Bus (lock + dead-letter) |
| Simple async queue, messages < 64 KB, millions in backlog | Azure Queue Storage |
| Trigger Azure Function when a blob is uploaded | Event Grid (Blob Storage event source) |
| Kafka app migration to Azure | Event Hubs (Kafka endpoint) |
| Audit all dequeue operations | Queue Storage (logs to Azure Monitor) |
| Enterprise messaging with VNet private endpoint | Service Bus **Premium** |
| Multiple teams need to independently replay same stream | Event Hubs (consumer groups) |
| Real-time dashboard + long-term archive of same stream | Event Hubs + Capture to ADLS Gen2 |
| Route events to different handlers by event type | Event Grid with event filters |

---

## 7. Key Limits

| Resource | Limit |
|---|---|
| Event Hubs max partitions (Standard) | 32 |
| Event Hubs max retention (Premium/Dedicated) | 90 days |
| Event Hubs message size (Standard) | 1 MB |
| Service Bus max message size (Premium) | 100 MB |
| Service Bus queue/topic size (Premium) | Up to 80 GB |
| Queue Storage max message size | 64 KB |
| Queue Storage max TTL | 7 days |
| Event Grid retry period | 24 hours |

---

## 8. Messaging Exam Traps

### 1. Choosing Event Hubs for enterprise command messaging
- **Trap:** Event Hubs handles very high throughput, so it looks like the scalable default
- **Better default:** Service Bus for commands, workflows, sessions, dead-lettering, and guaranteed delivery features

### 2. Choosing Service Bus for telemetry streaming
- **Trap:** It is reliable and full-featured
- **Better default:** Event Hubs for massive ingestion, replay, partitions, and consumer groups

### 3. Choosing Queue Storage when advanced messaging semantics are required
- **Trap:** It is simple and cheap
- **Better default:** Service Bus when you need FIFO with sessions, duplicate detection, transactions, or dead-letter queues

### 4. Choosing Event Grid for queued work processing
- **Trap:** Event Grid reacts quickly to events, so it seems suitable for all event patterns
- **Better default:** Event Grid is for reactive event routing, not durable queued work processing

### 5. Ignoring the difference between stream replay and message handling
- **Trap:** Both move events between systems
- **Better default:** Event Hubs for replayable event streams; Service Bus for individual message processing and acknowledgment

### Rapid Elimination Rules

| Requirement | Eliminate First |
|---|---|
| Millions of telemetry events per second | Service Bus-first answers |
| FIFO per customer or ordered workflow steps | Event Hubs-only answers |
| Durable queue with dead-letter support | Queue Storage-only answers |
| React to a blob created event | Service Bus-first answers |
| Kafka protocol compatibility | Non-Event Hubs answers |
