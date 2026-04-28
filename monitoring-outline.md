# Monitoring & Management — AZ-305 Study Outline

> **Status:** Outline — expand as needed
> **Exam Weight:** Part of ~25% Identity/Governance domain

---

## Key Topics to Study

### 1. Azure Monitor
- Central platform for **metrics, logs, alerts, dashboards**
- Data sources: Azure resources, OS (agents), applications, custom

#### Metrics
- Numerical time-series data, stored 93 days
- Near-real-time, low latency
- Used for autoscale rules and alert conditions

#### Logs (Azure Monitor Logs / Log Analytics)
- Stored in **Log Analytics workspace** (Azure Data Explorer under the hood)
- Query with **Kusto Query Language (KQL)** — exam may show basic KQL
- Retention: 30 days default, up to 730 days (longer via archive)

### 2. Log Analytics Workspace
- Central repository for logs from multiple sources
- One workspace can aggregate: VMs, AKS, App Service, Activity Log, Security events
- **Workspace design:** Centralized (one workspace) vs decentralized (per team/region)
  - **Exam tip:** Centralized recommended unless compliance requires data isolation

### 3. Azure Monitor Agents
| Agent | Platform | Data | Status |
|---|---|---|---|
| **Azure Monitor Agent (AMA)** | Windows/Linux | Metrics + Logs | Current, preferred |
| **MMA/OMS Agent** | Windows/Linux | Logs only | Legacy, being retired |
| **Diagnostics Extension (WAD/LAD)** | Windows/Linux | Metrics to Storage | Legacy |

### 4. Alerts
- **Metric alerts:** Fast, near-real-time, stateless or stateful
- **Log alerts:** Based on KQL query results, minimum 1-min frequency
- **Activity log alerts:** Triggered by control-plane events (VM deleted, policy assigned)
- **Action groups:** Reusable notification + action sets (email, SMS, webhook, Logic App, runbook, ITSM)

### 5. Application Insights
- Application Performance Monitoring (APM) for web apps
- Tracks: requests, dependencies, exceptions, custom events, traces
- **Smart detection:** Auto-detect anomalies in failure rate, performance
- **Application Map:** Visual dependency topology
- **Live Metrics:** Real-time stream (useful for deployments)
- **Sampling:** Adaptive (default) or fixed-rate — reduces data volume

### 6. Azure Workbooks
- Interactive reports combining metrics, logs, parameters
- Use for custom dashboards, operational reports, DR runbooks

### 7. Azure Advisor
- Personalized best-practice recommendations
- Categories: Cost, Security, Reliability, Performance, Operational Excellence
- Integrates with Defender for Cloud for security recommendations

### 8. Cost Management
- **Cost Analysis:** Visualize spend by resource, tag, subscription
- **Budgets:** Alert at threshold % of spending
- **Azure Reservations:** Commit to 1–3 years for significant discounts
- **Azure Hybrid Benefit:** Use existing Windows Server / SQL Server licenses
- **Tags:** Required for cost allocation — enforce via Azure Policy

### 9. Azure Service Health
- **Azure Status:** Global outage page
- **Service Health:** Personalized view of incidents affecting your subscription/region
- **Resource Health:** Per-resource health status and history
- Configure alerts for planned maintenance and incidents

---

## SLA Quick Reference — Tier Edge Cases

Exam questions frequently test whether you know which tier gains or loses SLA.

| Service | Tier / Config | SLA | Key Note |
|---|---|---|---|
| **APIM** | Consumption | 99.95% | Serverless, no dedicated infra |
| **APIM** | Developer | **No SLA** | Never use in production |
| **APIM** | Basic / Standard / Premium | 99.95% | Premium adds multi-region |
| **Azure Cache for Redis** | Basic | **No SLA** | Dev/test only |
| **Azure Cache for Redis** | Standard | 99.9% | Primary + replica |
| **Azure Cache for Redis** | Premium | 99.9% + zone redundancy option | Adds persistence, clustering |
| **App Service** | Free / Shared | **No SLA** | Not for production |
| **App Service** | Basic and above | 99.95% | Basic has no autoscale |
| **Azure SQL Database** | General Purpose (single) | 99.99% | |
| **Azure SQL Database** | Business Critical (single) | 99.99% | Adds readable secondary |
| **Azure SQL Database** | Hyperscale | 99.99% | |
| **Azure Functions** | Consumption | 99.95% | |
| **Azure Functions** | Premium | 99.95% | Pre-warmed, VNet |
| **VMs** | Single VM with Premium SSD | 99.9% | |
| **VMs** | Availability Set | 99.95% | Same datacenter, different fault domains |
| **VMs** | Availability Zones | 99.99% | Different datacenters in region |
| **Storage Account** | LRS | 99.9% read / 99.9% write | |
| **Storage Account** | ZRS | 99.99% read / 99.9% write | |
| **Storage Account** | GRS/RA-GRS | 99.9% / 99.99% read | RA-GRS adds secondary read |
| **AKS** | Free tier | **No SLA** | Dev/test clusters |
| **AKS** | Standard tier | 99.5% API server | Paid tier with SLA |
| **AKS** | Premium tier | 99.95% API server | Adds longer support |
| **Azure Kubernetes Service** | + Availability Zones | 99.9% nodes | Node pool spread |
| **Event Hubs** | Basic | 99.95% | |
| **Event Hubs** | Standard | 99.95% | Adds consumer groups |
| **Event Hubs** | Premium / Dedicated | 99.95% | Adds zone redundancy |
| **Service Bus** | Basic | 99.9% | No topics |
| **Service Bus** | Standard / Premium | 99.9% | Premium adds VNet, zone redundancy |

> **Exam tip:** Any **Developer** or **Free/Shared/Basic** tier (APIM Developer, Redis Basic, App Service Free/Shared, AKS Free) typically means **no production SLA**. Upgrading to the next tier is the fix.

---

## Exam Tips
- **Log Analytics workspace** is the backbone of most monitoring scenarios
- Know **AMA** is the current agent; MMA is legacy
- **Application Insights** is the answer for application-level monitoring (not VM metrics)
- **Action groups** are reusable — one group can be attached to many alerts
- Tags + Azure Policy = cost governance at scale
- Understand the difference between **Metrics** (fast, numeric) and **Logs** (rich, queryable)
