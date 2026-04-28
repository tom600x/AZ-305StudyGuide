# Application Architecture, Deployment & Integration — Deep-Dive Study Guide for AZ-305

## Why Application Architecture Matters on AZ-305
The exam tests your ability to:
- Design caching strategies (Azure Cache for Redis)
- Choose the right API Management tier for the scenario
- Select between App Configuration and Key Vault
- Design CI/CD pipelines and deployment strategies
- Choose between microservices platforms (AKS, Container Apps, Service Fabric)

---

## 1. Azure API Management (APIM)

### What It Is
Azure API Management is a **gateway, portal, and management layer** for publishing, securing, and monitoring APIs. Sits in front of backend APIs and applies policies.

### APIM Component Architecture

```
Consumers (apps, mobile, partners)
         ↓
  [Developer Portal]  ← Discover & subscribe to APIs
         ↓
  [API Gateway]       ← Apply policies, auth, rate limiting
         ↓
  [Backend Services]  ← Your actual APIs, Functions, Logic Apps
```

### APIM Service Tiers

| Tier | SLA | Scale | Features | Use Case |
|---|---|---|---|---|
| **Consumption** | 99.95% | Serverless, auto-scale | No VNet, no dedicated capacity | Dev/test, low traffic, event-driven |
| **Developer** | No SLA | 1 unit | Full features, not for prod | Development and testing only |
| **Basic** | 99.95% | 2 units | Basic features | Low-traffic prod, simple scenarios |
| **Standard** | 99.95% | 4 units | User groups, caching | Mid-traffic production |
| **Premium** | 99.95% | Up to 31 units | Multi-region, VNet integration | Enterprise, multi-region, high scale |

> **Exam tips:**
> - Need VNet integration → **Premium** or **Developer**
> - Need multi-region deployment → **Premium** only
> - Cost optimization, sporadic traffic → **Consumption**

### APIM Policies
Policies apply transformations to API requests and responses:

| Policy Type | When Applied | Examples |
|---|---|---|
| **Inbound** | Before request forwarded to backend | Auth validation, rate limiting, request transformation |
| **Backend** | Before backend call | URL rewrite, retry policy, caching lookup |
| **Outbound** | After response from backend | Response transformation, header addition |
| **On-Error** | On exception | Error response formatting |

**Key Policy Examples:**
- `rate-limit-by-key` — throttle calls per subscription/IP
- `validate-jwt` — validate OAuth 2.0/JWT tokens
- `cache-lookup` / `cache-store` — response caching
- `set-header` — add/modify headers
- `rewrite-uri` — change backend URL
- `mock-response` — return static response (testing)

### Policy Structure — Edge Cases

Policy XML has four sections. `<base />` runs policies inherited from a higher scope (global → product → API → operation):

```xml
<policies>
  <inbound>
    <base />              <!-- run parent-scope inbound policies first -->
    <rate-limit-by-key calls="10" renewal-period="60"
      counter-key="@(context.Subscription.Id)" />
  </inbound>
  <backend>
    <base />
  </backend>
  <outbound>
    <base />
    <set-header name="X-Processed-By" exists-action="override">
      <value>APIM</value>
    </set-header>
  </outbound>
  <on-error>
    <base />
    <return-response>
      <set-status code="500" reason="Internal Error" />
    </return-response>
  </on-error>
</policies>
```

> **Exam tip:** Omitting `<base />` **skips all parent-scope policies** at that point — this is a common exam distractor. Always include `<base />` unless you intentionally want to block inheritance.

### Policy Scenario Quick-Reference

| Scenario | Policy & Key Attributes |
|---|---|
| Rate limit: 10 calls/min per subscription | `rate-limit-by-key calls="10" renewal-period="60" counter-key="@(context.Subscription.Id)"` |
| Rate limit per caller IP | `rate-limit-by-key counter-key="@(context.Request.IpAddress)"` |
| Validate Bearer JWT, check audience claim | `validate-jwt header-name="Authorization" require-expiration-time="true"` |
| Block all IPs except known range | `ip-filter action="allow"` with `<address-range from="..." to="..." />` |
| Return 429 automatically on rate exceed | Built-in — `rate-limit` / `rate-limit-by-key` throw HTTP 429 |
| Cache GET responses 5 minutes | `cache-lookup` (inbound) + `cache-store duration="300"` (outbound) |
| Strip sensitive response header | `set-header name="X-Internal-Token" exists-action="delete"` (outbound) |
| Transform JSON body to XML for legacy backend | `json-to-xml` policy (inbound) |
| Forward custom correlation ID to backend | `set-header name="x-correlation-id"` with `@(context.RequestId)` (inbound) |
| Return mock/static response without hitting backend | `mock-response` (inbound — short-circuits pipeline) |

### Self-Hosted Gateway
- Deploy APIM gateway component on-premises or in other clouds
- Connects back to APIM control plane in Azure
- Requires **Premium** tier for production
- Use when: APIs hosted on-premises, edge gateway needed, data sovereignty requirements

---

## 2. Azure Cache for Redis

### What It Is
Azure Cache for Redis is a **managed in-memory key-value store** based on the Redis open-source project. Used for caching, session management, and pub/sub messaging.

### Service Tiers

| Tier | Max Memory | Clustering | Geo-Replication | Zone Redundancy | Use Case |
|---|---|---|---|---|---|
| **Basic C0–C6** | 250 MB – 53 GB | No | No | No | Dev/test, no HA |
| **Standard C0–C6** | 250 MB – 53 GB | No | No | No | Production, primary + replica |
| **Premium P1–P5** | 6 GB – 120 GB | Yes (up to 10 shards) | Yes | Yes | Enterprise, large datasets, persistence |
| **Enterprise** | Up to 1.5 TB | Yes | Yes | Yes | Highest performance, Redis modules |
| **Enterprise Flash** | Up to 1.5 TB (NVMe) | Yes | Yes | Yes | Cost-effective for large datasets |

> **Exam tips:**
> - Need clustering (sharding for scale) → **Premium** or Enterprise
> - Need geo-replication (multi-region active-passive) → **Premium** or Enterprise
> - Need Redis persistence (RDB/AOF) → **Premium** or Enterprise
> - Dev/test, single instance → **Basic**; HA (primary + replica) → **Standard** minimum

### Persistence Options (Premium+)

| Option | Description | Use Case |
|---|---|---|
| **Redis Database (RDB)** | Point-in-time snapshot at configurable intervals | Lower overhead, acceptable data loss on failure |
| **Append Only File (AOF)** | Log every write operation | Near-zero data loss, higher disk I/O |

### Common Use Cases

| Use Case | Implementation |
|---|---|
| **Cache-aside pattern** | App checks Redis first; on miss, loads from DB and stores in Redis |
| **Session state** | Store user session data; fast reads, TTL-based expiry |
| **Pub/Sub messaging** | Redis channels for lightweight messaging (not durable) |
| **Rate limiting** | Atomic increment counters per key with TTL |
| **Leaderboards** | Redis sorted sets for ranked data |

### Redis vs Service Bus vs Event Hubs

| | Redis Pub/Sub | Service Bus | Event Hubs |
|---|---|---|---|
| Durability | No | Yes | Yes |
| Message ordering | No | Optional (sessions) | Per partition |
| Consumer groups | No | Yes (subscriptions) | Yes |
| Scale | Vertical (sharding) | High | Massive |
| Use Case | Lightweight notifications | Reliable messaging | Telemetry streaming |

---

## 3. Azure App Configuration

### What It Is
Azure App Configuration is a **centralized key-value store** for application configuration settings and feature flags. Separate from secrets (which belong in Key Vault).

### Key Features

| Feature | Description |
|---|---|
| **Key-value pairs** | Hierarchical keys (e.g., `AppName:Setting:Value`) |
| **Feature flags** | Toggle features on/off without redeployment |
| **Labels** | Version or environment-specific config (e.g., prod, dev) |
| **Snapshots** | Immutable point-in-time capture of configuration |
| **Automatic refresh** | Apps detect and reload config changes without restart |

### App Configuration vs Key Vault

| | App Configuration | Azure Key Vault |
|---|---|---|
| **Stores** | Config settings, feature flags | Secrets, keys, certificates |
| **Encryption** | Encrypted at rest | HSM-protected |
| **Access** | App config SDK, REST | Key Vault SDK, REST |
| **Use Together** | Reference Key Vault secrets from App Config | Store secrets in Key Vault |
| **Pricing** | Free + Standard tier | Per operation |

> **Rule of thumb:** Config that is not sensitive → App Configuration. Sensitive data (passwords, connection strings, certificates) → Key Vault.

### Integration Points
- **App Service:** Mount App Configuration as environment variables
- **AKS:** Kubernetes provider for App Configuration
- **Azure Functions:** Configuration provider
- **Feature flags:** Microsoft.FeatureManagement .NET library

---

## 4. Automated Deployment

### CI/CD Approaches

| Tool | Description | Best For |
|---|---|---|
| **Azure DevOps Pipelines** | Full CI/CD: YAML or classic pipelines | Enterprise, complex multi-stage |
| **GitHub Actions** | Event-driven workflows integrated with GitHub | GitHub repos, OSS, modern teams |
| **Azure Deployment Center** | Simplified portal-based setup | Quick setup for App Service, AKS |

### Infrastructure as Code (IaC)

| Tool | Format | Azure-Native? | State Management |
|---|---|---|---|
| **Bicep** | Declarative DSL | Yes | Azure Resource Manager |
| **ARM Templates** | JSON | Yes | Azure Resource Manager |
| **Terraform** | HCL (HashiCorp) | No (multi-cloud) | State file (local/remote) |
| **Pulumi** | General-purpose languages | No (multi-cloud) | State managed by Pulumi |

> **Exam tip:** Bicep is the **recommended Azure-native IaC** approach. It compiles to ARM templates. Terraform for multi-cloud scenarios.

### Deployment Strategies for App Service

| Strategy | Description | Benefit |
|---|---|---|
| **Deployment Slots** | Separate slot (e.g., staging) with swap to production | Zero-downtime, instant rollback |
| **Blue-Green** | Two identical environments, traffic switch | Zero downtime, easy rollback |
| **Canary** | Gradually route traffic to new version | Risk reduction, validate incrementally |
| **Rolling** | Update instances one at a time | No extra infrastructure needed |

**Deployment Slot Details:**
- Available in Standard, Premium, Isolated App Service tiers
- Slot-specific settings can be "sticky" (not swapped)
- Swap warms up new version before taking traffic
- Supports auto-swap on successful deployment

---

## 5. Microservices & Container Platforms

### Platform Comparison

| Service | Manages | Control | Use Case |
|---|---|---|---|
| **Azure Kubernetes Service (AKS)** | Containers (pods, nodes) | Full | Complex microservices, custom networking, ML |
| **Azure Container Apps** | Containers (abstracts Kubernetes) | Simplified | Event-driven microservices, no Kubernetes expertise |
| **Azure App Service** | Web apps, APIs | Minimal | Traditional web apps, APIs, simple scaling |
| **Azure Container Instances (ACI)** | Single container or groups | None | Dev/test, batch jobs, simple containerized tasks |
| **Azure Functions** | Functions | None | Event-driven, serverless compute |

### AKS vs Container Apps

| Feature | AKS | Container Apps |
|---|---|---|
| Kubernetes expertise needed | Yes | No |
| Custom networking / Ingress | Full control | Managed Dapr + Envoy |
| Auto-scaling | Kubernetes Event-Driven Autoscaling (KEDA), Horizontal Pod Autoscaler (HPA), Cluster Autoscaler | Built-in KEDA |
| Cost | Pay for nodes | Pay per vCPU/memory second |
| Sidecar pattern | Manual | Built-in (Dapr) |

---

## 6. API Design & Integration Patterns

### API Gateway Pattern
- APIM in front of all backend services
- Single entry point, unified auth, rate limiting, logging
- Decouples client from backend service changes

### Backends for Frontends (BFF)
- Separate API gateway per client type (mobile, web, partner)
- Each BFF optimizes for its client's needs
- Can use APIM with multiple APIs/products

### Event-Driven Architecture
- Loose coupling via events instead of direct API calls
- Event Grid (reactive, pub-sub), Event Hubs (streaming), Service Bus (reliable queuing)
- See [messaging-study-guide.md](messaging-study-guide.md) for full details

---

## 7. Azure Logic Apps

### What It Is
Azure Logic Apps is a **low-code workflow automation platform** for integrating apps, data, services, and systems. Uses a visual designer with 400+ prebuilt connectors (Salesforce, SAP, Office 365, ServiceNow, etc.).

### Hosting Plans

| Plan | Description | Scaling | VNet Integration | Use Case |
|---|---|---|---|---|
| **Consumption** | Serverless, per-execution billing | Auto-scale, fully managed | No | Simple integrations, sporadic workloads, cost-sensitive |
| **Standard** | Single-tenant, runs on App Service | Dedicated or elastic | Yes (full VNet) | Enterprise, private endpoints, on-premises connectivity |

> **Exam tip:** If the scenario requires VNet integration, private endpoints, or on-premises connectivity for Logic Apps — the answer is **Standard** plan, not Consumption.

### Key Concepts

| Concept | Description |
|---|---|
| **Trigger** | What starts the workflow (HTTP request, schedule, event from connector) |
| **Action** | A step in the workflow (call API, send email, transform data) |
| **Connector** | Prebuilt integration to an external service (managed or custom) |
| **Managed Identity** | Authenticate to Azure services without storing credentials |

### Connectors

| Type | Description | Examples |
|---|---|---|
| **Built-in** | Native runtime connectors, faster, lower cost | HTTP, Service Bus, Event Hubs, Azure Functions |
| **Managed (Standard)** | Microsoft-hosted, shared infrastructure | Office 365, Salesforce, SAP, SQL Server |
| **Enterprise** | High-volume, mission-critical | SAP, IBM MQ, IBM 3270 |
| **On-premises** | Requires On-Premises Data Gateway | SQL Server on-prem, SharePoint on-prem, file system |

### Logic Apps vs Azure Functions vs Azure Data Factory

| | Logic Apps | Azure Functions | Azure Data Factory |
|---|---|---|---|
| Primary use | Workflow orchestration, B2B integration | Event-driven compute, custom code | Data movement and ETL pipelines |
| Coding required | Low-code (visual designer) | Code-first (C#, Python, JS, etc.) | Low-code + code for transforms |
| Connectors | 400+ prebuilt | Manual HTTP calls or bindings | 90+ data store connectors |
| Long-running workflows | Yes (stateful) | Not natively (Durable Functions for this) | Yes (pipeline runs) |
| B2B/EDI support | Yes (AS2, X12, EDIFACT) | No | No |
| **Choose when** | Enterprise integration, B2B, many SaaS systems | Custom logic, compute-intensive, event processing | Data pipelines, ETL/ELT, data lake ingestion |

> **Exam tip:** When a scenario involves integrating enterprise SaaS systems (e.g., Salesforce → SAP), automating business processes, or B2B EDI messaging — Logic Apps is the answer. When the scenario requires custom code or compute — use Functions. When it involves bulk data movement between stores — use ADF.

### Common Patterns

| Pattern | Logic Apps Feature |
|---|---|
| Scheduled data sync | Recurrence trigger |
| React to new blob, process and route | Azure Blob Storage trigger → condition → actions |
| B2B partner messaging (EDI) | Integration Account + AS2/X12/EDIFACT connectors |
| Call a Function when Logic App can't handle the logic | Azure Functions action inside a Logic App |
| Retry on failure | Built-in retry policies on each action |

---

## 8. Exam Scenario Cheat Sheet

| Scenario | Answer |
|---|---|
| API gateway for internal + external APIs, need VNet | APIM Premium tier |
| Throttle API calls to 100 per minute per user | APIM rate-limit-by-key policy |
| On-premises API needs APIM protection | APIM Self-Hosted Gateway |
| Cache API responses for 5 minutes | APIM cache-lookup + cache-store policy |
| Store feature flags, toggle without redeployment | Azure App Configuration with feature flags |
| Store database password for app | Azure Key Vault (not App Configuration) |
| Session state for web app, low latency | Azure Cache for Redis Standard |
| Redis needs to survive region failure | Redis Premium with geo-replication |
| Redis needs 100 GB+ with clustering | Redis Premium P4/P5 |
| Zero-downtime App Service deployment | Deployment slots + swap |
| IaC in Azure, native support | Bicep |
| Multi-cloud IaC | Terraform |
| Microservices, need full Kubernetes control | AKS |
| Microservices, team doesn't know Kubernetes | Azure Container Apps |
| Dev/test, run single container quickly | Azure Container Instances |
| Integrate Salesforce + SAP, no custom code | Azure Logic Apps (Consumption) |
| B2B EDI messaging with trading partner | Azure Logic Apps + Integration Account |
| Logic App needs private endpoint, VNet integration | Logic Apps Standard plan |
| Automate business workflow, 400+ SaaS connectors | Azure Logic Apps |
| Logic App needs custom compute logic | Azure Functions action within Logic App |
